# Nepal Earth — Technical Architecture Research

**Scope:** "Build a Nepal map with layers and a cool UI" — the interactive geospatial
intelligence platform from the blueprint (`Nepal Earth | Geospatial Intelligence Project
Blueprint`, 14 Aug 2026).

**Status:** Data sources verified live against their endpoints (Aug 2026). This document is
the *how to build it* — stack, per-layer pipeline, rendering decisions, build-vs-buy, cost
model, and a phased build sequence.

---

## 0. Executive summary

| Decision | Recommendation | Why |
|---|---|---|
| Map renderer | **MapLibre GL JS v5** (WebGL2) | Free/BSD, terrain + hillshade + vector/raster tiles in one engine, `setStyle` for time-slicing |
| 3D terrain | **AWS Terrain Tiles (terrarium RGB)** — free, no pipeline | Verified live; MapLibre terrain consumes it directly |
| Overlay engine | deck.gl (only when >100k points/arcs) | MapLibre handles the MVP; deck.gl only for hexbins/flows |
| Raster (land-cover) | Mirror ICIMOD rasters → **COGs in GCS**, serve via **titiler** | Don't hammer a third-party MapServer; COG gives analytics + display from one file |
| Vector (OSM/admin/forest) | OSM PBF → **PostGIS** → **Martin** (Rust tile server) or **PMTiles** | One PostGIS source of truth; tiles generated on demand |
| Backend | **FastAPI** + **PostGIS** (Cloud SQL) + **Redis** | Python geo ecosystem (GDAL/Rasterio/Shapely), spatial SQL |
| Search | Self-hosted **gazetteer** in PostGIS (tsvector/trigram) | OSM Nominatim public is rate-limited; Photon is heavyweight |
| Hosting | **GCP**: Cloud Run (API + jobs) + Cloud SQL + GCS + CDN | Matches your GCP-native preference; $15–30/mo at MVP |
| Frontend shell | **React + TypeScript + Vite** | Your standard stack; MapLibre has no React dependency (imperative API) |

**The one sentence version:** a PostGIS/PostgreSQL spatial core + COGs for rasters + MapLibre
GL for rendering, where the "cool UI" comes from *precomputed, queryable layers* (terrain,
time-sliced land-cover, forest metrics, trails) rather than from any single exotic technology.

---

## 1. Verified data-source inventory (live checks, Aug 2026)

Every source below was hit directly with `curl` against a browser UA. HTTP status and payload
shape are real, not assumed.

| # | Source | Endpoint | Result | What it actually returns | Gotcha |
|---|---|---|---|---|---|
| 1 | ICIMOD Nepal REST folder | `geoapps.icimod.org/icimodarcgis/rest/services/Nepal?f=pjson` | ✅ HTTP 200 | JSON listing **11 MapServers**: AgricultureAtlas, BaseMap, GCF, Hydropower, KarnaliCropSuitability, KTM_Corona05Feb67, **Landcover**, **NepalAdmin**, **NepalForestType**, **Physiography**, Pure | ArcGIS REST (not OGC WMS); use `?f=pjson` and `export` endpoints |
| 2 | ICIMOD Landcover | `.../Nepal/Landcover/MapServer?f=pjson` | ✅ HTTP 200 | **22 annual raster layers**, `Landcover_2000`…`Landcover_2022`, WGS84 (EPSG:4326), full-Nepal extent | **2012 is missing** — the series skips from 2011 to 2013 |
| 3 | ICIMOD Landcover render | `.../Landcover/MapServer/export?bbox=…&size=512,256&f=image` | ✅ HTTP 200 | **Real 512×256 PNG** (35 KB) | You *can* render live, but don't depend on a 3rd-party server in prod — mirror it |
| 4 | ICIMOD NepalAdmin | `.../Nepal/NepalAdmin/MapServer?f=pjson` | ✅ HTTP 200 | 3 vector polygon layers: **Outline, Province, District** | Clean admin boundaries for free |
| 5 | ICIMOD Physiography | `.../Nepal/Physiography/MapServer?f=pjson` | ✅ HTTP 200 | 1 vector polygon: **Physiographic_Region** | High Himalaya → Terai |
| 6 | ICIMOD NepalForestType | `.../Nepal/NepalForestType/MapServer?f=pjson` | ✅ HTTP 200 | **ForestType**, **Tree Canopy Cover 2022 (%)**, **Tree Canopy Height 2022 (m)**, Patch Density, Largest Patch Index, GaPaNaPa | Raster + vector mix; canopy %/height are the crown jewels |
| 7 | Nepal National Geoportal GeoServer | `admin.nationalgeoportal.gov.np/geoserver/web/` | ✅ HTTP 200 (login page) | GeoServer admin UI; advertises WMS/WFS/WCS/WPS | **OGC endpoints return 400/exception** — per-layer auth likely needed; treat as secondary |
| 8 | Geofabrik Nepal OSM | `download.geofabrik.de/asia/nepal.html` | ✅ HTTP 200 | `nepal-latest.osm.pbf` = **411,966,673 bytes (~393 MB)**; also .shp.zip, .gpkg | ODbL — share-alike applies to derived DBs if redistributed |
| 9 | NTNC Geoportal | `geoportal.ntnc.org.np/` | ❌ **HTTP 000** (connection failed) | — | Down/unreachable from this network; nice-to-have, not core. Re-check before depending on it |
| 10 | AWS Terrain Tiles (Mapzen) | `s3.amazonaws.com/elevation-tiles-prod/terrarium/0/0/0.png` | ✅ HTTP 200 | **256×256 RGB PNG** (terrarium-encoded elevation) | Free global terrain, no key, no quota for reasonable use |
| 11 | Copernicus DEM GLO-30 | `registry.opendata.aws/copernicus-dem/` | ✅ HTTP 200 | Free **30 m** global DEM (AWS Open Data) | The raw DEM if you want custom hillshade/analytics beyond the terrain tiles |

### What this means

- **Core stack is fully unblocked** — land-cover time series, forest metrics, admin
  boundaries, physiography, and OSM detail all come from free, live sources.
- **Terrain is free and zero-pipeline** via AWS Terrain Tiles.
- **Three caveats to bake into the design:** (a) land-cover **2012 gap**, (b) National
  Geoportal OGC endpoints need per-layer auth confirmation, (c) NTNC is currently down.

---

## 2. Architecture overview

```
┌───────────────────────────  Frontend (React + TS + Vite)  ───────────────────────────┐
│  MapLibre GL JS (map, terrain, time-slicing, hillshade)                               │
│  └─ deck.gl (optional: hexbin/arc overlays)                                           │
│  └─ d3 / lightweight chart (elevation profiles, Sankey land-cover transitions)        │
└──────────────────────────────────────┬───────────────────────────────────────────────┘
                                       │ HTTPS (tiles + JSON API)
              ┌────────────────────────┴────────────────────────┐
              │                                                  │
   ┌──────────▼──────────┐                          ┌────────────▼────────────┐
   │  Tile / raster path  │                          │   Analytics API path    │
   │  (Cloud CDN + GCS)   │                          │   (Cloud Run / FastAPI) │
   │  - vector tiles      │                          │   - /search (gazetteer) │
   │  - land-cover COGs   │                          │   - /terrain/profile    │
   │  - hillshade         │                          │   - /landcover/change   │
   │  - terrain (AWS)     │                          │   - /forest/profile     │
   └──────────────────────┘                          │   - /proximity          │
                                                     └────────────┬────────────┘
                                                                  │
              ┌───────────────────────────────────────────────────▼──────────────┐
              │  PostGIS (Cloud SQL)  +  Redis cache  +  GCS (COGs / raw)        │
              │  Tables: admin_units, rivers, trails, peaks, places,             │
              │          protected_areas, forest_units, dataset_registry,        │
              │          landcover_stats, terrain_stats, derived_relations       │
              └───────────────────────────────────────────────────────────────────┘
                                                                  ▲
              ┌───────────────────────────────────────────────────┘
              │  Batch pipelines (Cloud Run Jobs, cron):
              │  1. fetch  → 2. validate → 3. standardise → 4. enrich → 5. tile → 6. publish
              │  (GDAL / Rasterio / osm2pgsql / tippecanoe)
```

Two cleanly separated paths: **render path** (tiles/COGs over CDN, no server compute per
request) and **analytics path** (spatial SQL behind FastAPI). The blueprint's §12 performance
strategy is correct here; this is the standard "don't run a spatial join on every mouse-move"
architecture.

---

## 3. Rendering stack decision (the "cool UI" engine)

| Criterion | MapLibre GL JS | deck.gl | CesiumJS | Three.js (raw) |
|---|---|---|---|---|
| License / cost | BSD / free | MIT / free | Apache-2.0 / free (ion token for assets) | MIT / free |
| Vector tiles | ✅ native | via loaders | limited | ❌ DIY |
| Raster + COG | ✅ native | via loaders | ✅ | ❌ DIY |
| **Terrain / 3D** | ✅ native (`terrain` source) | ❌ | ✅ best-in-class (true globe) | ✅ (full control) |
| Hillshade | ✅ native | ❌ | ✅ | DIY |
| Time-slicing (crossfade years) | ✅ `setStyle`/source swap | partial | ✅ | DIY |
| Large point/arc/hexbin | ⚠️ limited | ✅ best | ✅ | DIY |
| Learning curve / bundle | Low / ~200 KB gz | Medium / large | High / large | High / large |
| Nepal-only MVP fit | **Best** | overlay when needed | overkill (need ion assets) | only for bespoke globe |

**Verdict: MapLibre GL JS is the engine. deck.gl is a conditional add-on. Cesium and Three.js
are out of scope for MVP.**

Rationale: the blueprint's signature features (terrain, time slider, hillshade, feature
inspection) are all *first-class* in MapLibre with no external asset token (Cesium ion
requires one) and a fraction of the bundle size. Three.js/Cesium only earn their complexity
for a true rotating-globe "digital twin" hero mode, which is a Phase-4 luxury, not a must-have.

---

## 4. Terrain / DEM pipeline

**Decision: use free hosted terrain tiles; generate hillshade only if you want custom shading.**

1. **3D terrain + hillshade — buy, don't build.**
   - Point MapLibre `terrain` source at AWS Terrain Tiles (terrarium encoding):
     `https://s3.amazonaws.com/elevation-tiles-prod/terrarium/{z}/{x}/{y}.png`
   - Verified live (z=0 tile returns a real 256×256 RGB PNG). No key, no quota for a
     portfolio/demo product. This gives you 3D mountains + sky + fog for **$0 and zero
     pipeline code**.
2. **Custom elevation analytics — build (thin).**
   - For elevation profiles and "what's above 3,000 m here", download **Copernicus GLO-30**
     (30 m) tiles for Nepal's bounding box from AWS Open Data (~1–2 GB for Nepal).
   - Process with `gdaldem` / `rasterio`: reproject to EPSG:3857, build a VRT mosaic, then
     either (a) leave as COGs and sample on-demand with `rio-tiler`, or (b) precompute
     `terrain_stats` (min/max/mean/slope) per tile/administrative unit into PostGIS.
   - **Precompute > on-demand** for profile queries: a trail elevation profile is a single
     `ST_Value`-style raster sample along a line; cheap, but cache the result in Redis.

**Higher-res terrain (if trekking-grade detail matters later):** ALOS AW3D30 (free 30 m),
or commercial 5 m / 1 m. Not needed for the map MVP; the Annapurna trek wedge would revisit
this.

---

## 5. Raster pipeline — the "Landscape Time Machine" (land-cover 2000–2022)

This is the flagship feature and the most technically interesting layer. The naive approach
(proxy the ICIMOD `export` endpoint) works — I confirmed a live PNG render — but it's wrong
for production: it puts a foreign MapServer in your critical path, adds latency, and gives
you no analytics.

**Correct approach — mirror to COGs:**

1. **Download** the 22 annual GeoTIFFs from ICIMOD (dataset page, **CC BY 4.0** — attribution
   required, redistribution allowed with credit). If a direct GeoTIFF isn't exposed, fetch via
   the MapServer's `export`/`exportImage` at full extent, or the RDS download.
2. **Normalise** with `gdal_translate -of COG` (Cloud-Optimized GeoTIFF): internal tiling +
   overviews, EPSG:3857 (Web Mercator for tile alignment) or keep 4326 and let titiler
   reproject. Store in **GCS**.
   - 22 years × Nepal @ 30 m ≈ **2–4 GB total**. Trivial storage cost (~$0.05/mo).
3. **Serve** two ways, one asset:
   - **Display:** titiler (or a pre-generated raster-tile pyramid) serves XYZ tiles for the
     map. On slider move, swap the raster source / `beforeId` and crossfade.
   - **Analytics:** `rio-tiler`/`rasterio` computes class-transition matrices (2000 vs 2022)
     for an AOI, feeding the "change panel" and the Sankey diagram.
4. **Handle the 2012 gap explicitly.** The slider will render 2011 → 2013. Either (a) show a
   "no data for 2012" tick on the slider, or (b) interpolate/omit — but never silently fake a
   2012 layer. This is exactly the kind of data-integrity detail the blueprint's §20 trust
   model demands.

**COG over pre-tiled:** COGs are a single file per year that works for *both* display and
zonal statistics; pre-tiled pyramids double storage and don't give you analytics. This is the
standard modern answer (titiler + COG).

---

## 6. Vector pipeline — OSM detail + admin + forest

**6.1 OSM (roads, trails, rivers, places, buildings)**

- Download `nepal-latest.osm.pbf` (~393 MB, verified).
- **Two-stage:** load into PostGIS with `osm2pgsql` (flex output) filtering to the feature
  classes you need (`highway`, `waterway`, `place`, `natural=peak`, `building`, `leisure`,
  `tourism`, `boundary=protected_area`), then serve vector tiles.
- **Tile serving — pick Martin or PMTiles:**
  - **Martin** (Rust, PostGIS-native): generates tiles on the fly from SQL. Best when data
    refreshes and you want a live PostGIS → tile path.
  - **PMTiles** (single-file archive via `tippecanoe`): best when you want to precompute once
    and serve from GCS/CDN with zero server. Faster/cheaper at MVP.
  - **Recommendation: PMTiles at MVP** (static, CDN-served, ~free), **Martin later** if the
    gazetteer/refresh story needs live tiles.
- **ODbL obligation:** if you redistribute the *derived* tourism/trail database, share-alike
  applies. Mitigate by (a) keeping the OSM-derived layer clearly attributed and, (b) treating
  your *derived metrics* (elevation profiles, scores) as your own computation on top — but be
  honest that the underlying geometry is ODbL. This is the exact legal flag from the PRD
  review; it belongs in the data-provenance panel, not ignored.

**6.2 Admin boundaries + physiography (ICIMOD)**

- Pull `NepalAdmin` (Outline/Province/District) and `Physiography` as GeoJSON via the ArcGIS
  REST `query` endpoint (`where=1=1&outFields=*&f=geojson`), load into PostGIS. These are the
  navigation/aggregation skeleton.

**6.3 Forest (ICIMOD NepalForestType)**

- Vector polygons (Patch Density, Largest Patch Index, GaPaNaPa) → PostGIS `forest_units`.
- Raster (Canopy Cover %, Canopy Height m) → COGs alongside land-cover, so `forest/profile`
  can return canopy metrics for any clicked AOI via the same titiler path.

---

## 7. Backend — PostGIS schema + FastAPI API

### 7.1 PostGIS tables (from the blueprint §10, refined)

| Table | Geometry | Key fields |
|---|---|---|
| `admin_units` | Polygon | province, district, local_unit, source_id |
| `places` | Point/Polygon | name, type, admin_hierarchy, source_id |
| `peaks` | Point | name, elevation_m, prominence_m, source_id |
| `rivers` | LineString/MultiLineString | name, strahler_order (derived), source_id |
| `waterbodies` | Polygon | name, type, source_id |
| `trails` | LineString | name, route_type, surface, source_id |
| `protected_areas` | Polygon | name, designation, source_id |
| `forest_units` | Polygon | forest_type, canopy_cover, canopy_height, patch_density, lpi |
| `landcover_stats` | Polygon/Tile index | year, class, area_m2, source_version |
| `terrain_stats` | Polygon/Tile index | min_elev, max_elev, mean_elev, slope_stats |
| `dataset_registry` | — | source, license, temporal_range, processing_version, checksum |
| `derived_relations` | — | source_object, relation_type, target_object, confidence |

**Indexes:** GiST on every geometry column; btree on `source_id`, `year`, `class`. `derived_relations`
is the "knowledge graph" (river↔peak↔park↔trail) — plain relational rows, **no Neo4j needed**
(blueprint §14 is right; a graph DB is Phase-late or never).

### 7.2 API surface (FastAPI, from blueprint §11)

| Endpoint | Purpose |
|---|---|
| `GET /search?q=` | Gazetteer search (tsvector + trigram on names) |
| `GET /places/{id}` | Feature profile + provenance + computed metrics |
| `GET /map/layers` | Layer catalogue for the UI drawer |
| `GET /terrain/profile` | Elevation profile along a line (DEM sampling) |
| `GET /landcover/change` | Class transitions inside AOI (2000 vs 2022) |
| `GET /forest/profile` | Canopy + fragmentation summary |
| `GET /proximity` | Nearest / within-distance relationships |
| `POST /spatial/query` | Structured spatial filter (AOI + predicates + layer filters) |
| `GET /datasets/{id}/provenance` | Data lineage (source + license + version + refresh date) |

**Every endpoint returns a `provenance` block.** The blueprint's §20 trust model is a feature,
not a footnote — and it's the single cheapest thing that makes this look "real" rather than
"a pretty map."

---

## 8. Search / geocoding

**Decision: self-host a PostGIS gazetteer, don't call public Nominatim.**

- Public OSM Nominatim has a hard 1 req/s policy — unusable for a product.
- Build the gazetteer from the OSM extract: `places` + `peaks` + `rivers` + `trails` +
  `protected_areas` + `admin_units`, with a `tsvector` column + trigram index for fuzzy names.
- This is ~100 lines of SQL and gives instant, unlimited, Nepal-scoped search (better than
  global Nominatim for "Annapurna" vs "Annapurna South").

---

## 9. The "cool UI" — concrete spec

What actually makes a map feel modern (each mapped to a mechanism, not a vibe):

| Feature | Mechanism | Difficulty |
|---|---|---|
| 3D terrain + mountains | MapLibre `terrain` (AWS tiles) + `sky` + fog | Trivial |
| Hypsometric tint / hillshade | MapLibre hillshade layer + elevation ramp | Trivial |
| Land-cover **time slider** (2000→2022) | Swap COG raster source on slider input; crossfade via opacity | Easy |
| Feature **click-to-inspect** | `queryRenderedFeatures` → `GET /places/{id}` → glass panel | Easy |
| **Elevation profile** chart | `GET /terrain/profile` → d3 line chart | Medium |
| **Compare mode** (year A vs B, split/swipe) | Two synced maps or a swipe layer | Medium |
| Land-cover **transition Sankey** | `GET /landcover/change` → d3-sankey | Medium |
| **Animated fly-to** on search | MapLibre `flyTo` with easing | Trivial |
| Layer drawer (toggle + opacity) | React state → `setPaintProperty` | Trivial |
| URL state (shareable view) | Serialize `{layer, year, extent, selected}` to query string | Easy |
| Data provenance drawer | `GET /datasets/{id}/provenance` → modal | Easy |
| Spatial question builder | Structured filters → `POST /spatial/query` (no free-form LLM at MVP) | Medium |

**The through-line:** every "wow" is a *precomputed layer + a SQL/COG query*, not a heavy
runtime computation. That's why the map stays at 60 fps and the whole thing can run on Cloud
Run for pennies.

---

## 10. Build-vs-buy matrix

| Component | Decision | Rationale / cost |
|---|---|---|
| Map renderer | **Buy** (MapLibre GL, free) | No reason to build WebGL |
| Terrain tiles | **Buy** (AWS, free) | Zero pipeline, live now |
| DEM (for analytics) | **Buy** (Copernicus GLO-30, free) | 30 m is enough |
| Land-cover rasters | **Buy** (ICIMOD, CC BY 4.0) | Free, authoritative |
| Raster serving | **Build** (titiler, free OSS) | The analytics+display unified path is your IP |
| Vector tiles | **Build** (tippecanoe/PMTiles, free) | Self-host = zero per-request cost |
| PostGIS | **Buy** (Cloud SQL, ~$10–25/mo) | Managed spatial DB beats self-host |
| Search | **Build** (PostGIS gazetteer, free) | Unlimited + Nepal-scoped |
| Map tiles basemap | **Buy** (OSM raster or a free style) | Never compete on basemap (blueprint §2) |
| Auth / saved stories | **Buy** (Identity Platform) when Phase 7 | Not MVP |
| LLM / agent | **Buy** (Claude) — Phase 5+ only | Blueprint §13 "ML second" is correct |
| Hosting | **Buy** (Cloud Run/Cloud SQL/GCS) | GCP-native, serverless |

---

## 11. Cost model

| Resource | Spec | MVP $/mo | Growth $/mo |
|---|---|---|---|
| Cloud Run (API + titiler) | 1 vCPU / 512 MB, scale-to-zero | $0–3 | $10–30 |
| Cloud SQL PostGIS | db-f1-micro, no HA, SSD | ~$9–12 | $25–50 (f1-small + HA) |
| GCS (COGs + PMTiles + raw) | ~5 GB | ~$0.10 | ~$1 |
| CDN egress | <50 GB | $1–5 | $10–50 |
| Terrain tiles (AWS) | free | $0 | $0 |
| Vector tiles (self-host) | free | $0 | $0 |
| **Total** | | **~$15–30/mo** | **~$50–130/mo** |

**The entire MVP runs for roughly the cost of a restaurant meal per month**, because the heavy
assets (terrain, land-cover) are either free or self-hosted COGs served over CDN — there is no
per-request metered map API (Mapbox/Google would be $50–500/mo at the same traffic). This is
the single biggest cost argument for the self-hosted stack.

---

## 12. Implementation sequence (phased, with time)

| Phase | Deliverable | Est. | Depends on |
|---|---|---|---|
| **P0 — data proof** | Download 2 land-cover years + 1 admin layer; verify license/CRS; write `dataset_registry` + fetch script | 1–2 d | — |
| **P1 — map foundation** | PostGIS + FastAPI + React + MapLibre; Nepal basemap with admin/rivers/places; click-to-inspect | 3–5 d | P0 |
| **P2 — time machine** | Mirror 22 land-cover COGs to GCS; titiler serving; 2000–2022 slider + change panel (handle 2012 gap) | 3–4 d | P0 |
| **P3 — forest + terrain** | Forest COGs + `forest_units`; AWS terrain; hillshade; elevation-profile endpoint | 3–4 d | P1 |
| **P4 — spatial query + compare** | `POST /spatial/query`, split-screen compare, Sankey transitions | 3–4 d | P2 |
| **P5 — productise** | Auth, saved stories, URL sharing, observability, cost controls | 4–6 d | P1–P4 |
| *(later)* | One ML model (landslide susceptibility) with visible uncertainty | 1–2 wk | P4 |

**Total to a public-looking MVP: ~3–4 focused weeks** for a solo engineer, because the data is
free and the rendering is solved. The 2012 gap and ODbL attribution are handled in P2/P1
respectively, not as afterthoughts.

---

## 13. Risks & pitfalls

1. **Third-party MapServer instability.** Never proxy ICIMOD `export` in prod — mirror to COGs
   on first fetch, keep the source URL in `dataset_registry` for provenance (blueprint §12).
2. **Land-cover 2012 gap.** Design the slider to show the gap honestly; don't fake a 2012 layer.
3. **ODbL share-alike on OSM-derived layers.** Keep attribution visible; separate "derived
   metrics" (yours) from "underlying geometry" (ODbL). Reconcile before public redistribution.
4. **NTNC down (HTTP 000).** It's a nice-to-have enrichment; don't schedule work that depends
   on it. Re-check availability before Phase 3.
5. **National Geoportal OGC endpoints return exceptions.** Confirm per-layer auth/terms before
   relying on it as an authoritative source; ICIMOD covers most of the same ground.
6. **"Map looks good but does nothing."** The counter is mandatory feature inspection + derived
   analytics (P1's click-to-inspect is the anti-demo shield).
7. **Raster size blowup.** Always COG (internal tiles + overviews) + `rio-tiler` server-side
   clipping; never push raw GeoTIFFs to the browser.
8. **Premature ML.** Ship the spatial platform first; add one justified model with visible
   uncertainty later (blueprint §13's own rule).

---

*Data sources verified live August 2026. License checks: ICIMOD CC BY 4.0, OSM ODbL, Copernicus
DEM free (AWS Open Data), AWS Terrain Tiles free (Mapzen/terrarium).*
