# Nepal Earth — Map Stack Tutorial

A hands-on, learn-by-building guide to the geospatial stack behind the Nepal Earth map
platform. This is the companion to `docs/10-technical-architecture.md` — that doc says
*what* to build and *why*; this one teaches you *how*, from zero.

**Who this is for:** a strong backend/ML engineer who is new to GIS and web maps. You already
know Python, SQL, APIs, and cloud. This tutorial fills the specific gap: *how do web maps,
geospatial data, and tiles actually work, and how do I build with them?*

**How to use it:** read §1–§2 once to build the mental model, then work through §3 in order.
Each layer ends with a **"prove it works"** command so you're never just reading — you're
building. The final §4 walks a Day-1 → Week-N path that mirrors the real project phases.

---

## 1. Prerequisites & setup (verified against your machine, Aug 2026)

### What's on your machine right now

| Tool | Status | Action needed |
|---|---|---|
| Homebrew 6.0.2 | ✅ | — |
| Node | ⚠️ **v10.14.1** | **Upgrade to v20 LTS** — Vite + MapLibre need Node 18+. v10 will fail immediately. |
| npm | ⚠️ 6.4.1 | Comes along with Node upgrade |
| Python | ⚠️ **3.14.5** | Too new — GDAL/rasterio ship wheels for 3.11/3.12. Pin a venv with `uv` |
| uv | ✅ | Use it for a pinned, isolated Python |
| GDAL / ogr2ogr / gdalinfo | ❌ | `brew install gdal` |
| tippecanoe | ❌ | `brew install tippecanoe` |
| PostGIS / psql | ❌ | `brew install postgresql@16 postgis` (or use Cloud SQL later) |
| Docker | ❌ | Optional — only if you want a one-command PostGIS container |
| gcloud | ❌ | `brew install --cask google-cloud-sdk` (when you deploy) |

### One-time setup (do this first)

```bash
# 1. Upgrade Node to LTS (this is the hard requirement)
brew install node@20
brew link --overwrite node@20
node --version   # must show v20.x

# 2. Install the GIS toolchain
brew install gdal tippecanoe

# 3. Pin a Python with working geospatial wheels (NOT 3.14)
uv python install 3.12
uv venv --python 3.12 ~/.venvs/nepal-earth
source ~/.venvs/nepal-earth/bin/activate
uv pip install rasterio shapely fiona geopandas rio-tiler fastapi uvicorn httpx

# 4. Start PostGIS (local dev — Homebrew)
brew services start postgresql@16
createdb nepal_earth
psql -d nepal_earth -c "CREATE EXTENSION IF NOT EXISTS postgis;"
```

> **Why pin Python 3.12?** `rasterio` and `fiona` compile against GDAL's C library and ship
> prebuilt wheels per Python version. At the time of writing, 3.14 wheels lag behind. `uv`
> makes a clean isolated env so your global Python isn't polluted by GIS deps.

---

## 2. The mental model — five concepts that explain all of web maps

You can learn every tool in this stack once you hold five ideas firmly.

### 2.1 There are two kinds of geospatial data: vector and raster

- **Vector** = discrete shapes with attributes. Points (a peak), lines (a river, a trail),
  polygons (a district, a park). Stored as **GeoJSON**, **Shapefile**, or rows in a database
  with a geometry column. Think "a list of named things."
- **Raster** = a grid of cells, each holding one number. Elevation, land-cover class, canopy
  cover %. Stored as **GeoTIFF**. Think "a spreadsheet stretched over the landscape."

```jsonc
// VECTOR (GeoJSON) — a point
{ "type": "Feature",
  "geometry": { "type": "Point", "coordinates": [83.98, 28.59] },  // lon, lat (NOT lat, lon)
  "properties": { "name": "Annapurna I", "elevation_m": 8091 } }
```

```bash
# RASTER (GeoTIFF) — inspect one
gdalinfo dem.tif   # shows size in pixels, resolution, CRS, bands, min/max
```

**The one rule that bites everyone:** coordinates are `[longitude, latitude]`, always in that
order (X, then Y). Nepal sits around `[84, 28]`, not `[28, 84]`.

### 2.2 Coordinate Reference Systems (CRS) — EPSG:4326 vs EPSG:3857

- **EPSG:4326** = WGS84, the "lon/lat on a sphere" that GPS uses. This is how the ICIMOD data
  arrives (verified: all ICIMOD MapServers report `wkid: 4326`).
- **EPSG:3857** = Web Mercator, the "flat square map" that every web tile service uses. It
  projects the globe onto a square so it tiles cleanly. This is what MapLibre and tile
  servers work in.

You will constantly transform between them: `4326` for storage/authoring, `3857` for tiling
and display. The tools do it for you, but you must know *which* CRS your data is in, or your
map renders in the middle of the ocean.

```bash
gdalinfo dem.tif | grep -i "coordinate system"   # always check CRS first
gdalwarp -t_srs EPSG:3857 in.tif out_3857.tif    # convert 4326 -> 3857
```

```sql
-- PostGIS: store 4326, transform on query
SELECT ST_Transform(geom, 3857) FROM admin_units WHERE name = 'Mustang';
```

### 2.3 Tiles — the XYZ scheme and zoom levels

A world map at full resolution is terabytes. The answer is **tiles**: chop the world into
256×256 px squares, organized by `zoom / x / y`.

- **Zoom 0** = one tile = the whole world.
- **Zoom 1** = 4 tiles, zoom 2 = 16, zoom z = 4^z tiles.
- Each zoom level **doubles** resolution. z=7 shows Nepal comfortably; z=13 shows a
  neighbourhood; z=15 shows buildings.

```text
https://s3.amazonaws.com/elevation-tiles-prod/terrarium/{z}/{x}/{y}.png
                                                          └── these three change per tile
```

The browser only ever downloads the ~20 tiles visible in your viewport at the current zoom.
That's the entire performance secret of web maps: **you never load the whole world, only the
view.**

### 2.4 Vector tiles vs raster tiles

- **Raster tiles** = pre-rendered PNGs. The server has already drawn the map. Simple and fast
  to serve, but you can't restyle them, and they're blurry when zoomed past their native
  resolution.
- **Vector tiles** = the actual geometry + attributes, encoded in a compact binary format
  (`.pbf`, wrapped in `.mbtiles` or **PMTiles**). The *browser* draws them with WebGL. You can
  restyle, hover, click, and toggle layers client-side — that's what makes a map feel
  interactive. This is what MapLibre consumes.

**Rule of thumb:** base terrain/satellite = raster; everything you want to interact with
(roads, rivers, trails, parks) = vector tiles.

### 2.5 Cloud-Optimized GeoTIFF (COG) — the raster superpower

A normal GeoTIFF must be fully downloaded before you can read any pixel. A **COG** is the
same data re-arranged so it has internal tiles + overviews + an index, letting a client
**read just the part it needs over HTTP**. One COG file serves both:

1. **Display** — a tile server reads the few tiles for your viewport.
2. **Analytics** — a client reads one region to compute a statistic.

This is why the architecture says "mirror ICIMOD land-cover to COGs" — one file, two uses.

```bash
gdal_translate -of COG landcover_2022.tif landcover_2022_cog.tif   # make a COG
```

---

## 3. The stack, layer by layer (what / why / prove-it-works)

The stack reads bottom-up: data is *processed* with GDAL, *stored* in PostGIS, *tiled* with
tippecanoe/Martin/titiler, *served* by FastAPI, and *rendered* by MapLibre.

```
MapLibre GL JS (render)  ←  FastAPI (query)  ←  PostGIS (store)
         ↑                      ↑                    ↑
   vector tiles (PMTiles)   raster tiles (titiler/COG)
         ↑                      ↑
        tippecanoe           gdal_translate
         ↑                      ↑
       GeoJSON              GeoTIFF / raw DEM
```

### Layer 0 — GDAL & Rasterio: the data workhorse

**What:** GDAL is the Swiss-army knife of raster/vector geodata — `gdalinfo`,
`gdal_translate`, `gdalwarp`, `gdaldem`, `ogr2ogr`. Rasterio is its Python binding.

**Why you need it:** every "make this data usable" step — reproject, clip, merge, convert to
COG, compute hillshade — is one GDAL command. It's the first tool you touch on any GIS job.

**Prove it works** (reproject a DEM and make a hillshade):

```bash
# inspect
gdalinfo dem.tif | grep -iE "size is|coordinate system|pixel size"

# reproject 4326 -> 3857 (Web Mercator, for tiles)
gdalwarp -t_srs EPSG:3857 -r bilinear dem.tif dem_3857.tif

# hillshade (sun from NW at 45° altitude — the standard cartographic look)
gdaldem hillshade dem_3857.tif hillshade.tif -z 2 -az 315 -alt 45

# make it a COG (internal tiles + overviews)
gdal_translate -of COG hillshade.tif hillshade_cog.tif
```

```python
# Rasterio: read a COG and inspect it (in Python)
import rasterio
with rasterio.open("dem_3857.tif") as ds:
    print(ds.crs, ds.res, ds.count)          # CRS, pixel size, band count
    print(ds.index(83.98, 28.59))            # row,col of a lon/lat
    # (in practice use rio-tiler for point sampling — see Layer 3)
```

### Layer 1 — PostGIS: the spatial database

**What:** PostgreSQL + the PostGIS extension = a relational database that *understands*
geometry and can answer "what's within 5 km of this river?"

**Why you need it:** the whole "feature profile" and "proximity" features are spatial SQL.
A GiST index makes `ST_DWithin` (distance) and `ST_Intersects` (overlap) fast on millions of
rows. You do *not* need a graph database — the blueprint's "knowledge graph" is just a
`derived_relations` table queried with spatial joins.

**Prove it works:**

```sql
CREATE EXTENSION IF NOT EXISTS postgis;

CREATE TABLE peaks (
  id SERIAL PRIMARY KEY,
  name TEXT,
  elevation_m INT,
  geom GEOMETRY(Point, 4326)      -- 4326 = lon/lat
);
CREATE INDEX peaks_gix ON peaks USING GIST (geom);

INSERT INTO peaks (name, elevation_m, geom) VALUES
  ('Annapurna I', 8091, ST_SetSRID(ST_MakePoint(83.82, 28.59), 4326)),
  ('Machapuchare', 6993, ST_SetSRID(ST_MakePoint(83.95, 28.49), 4326)),
  ('Dhaulagiri I', 8167, ST_SetSRID(ST_MakePoint(83.49, 28.69), 4326));

-- "peaks within 20 km of Machapuchare" — the query behind the Proximity feature
SELECT p.name, round(ST_Distance(p.geom::geography, m.geom::geography)/1000) AS km
FROM peaks p, peaks m
WHERE m.name = 'Machapuchare' AND p.name <> 'Machapuchare'
  AND ST_DWithin(p.geom::geography, m.geom::geography, 20000)
ORDER BY km;
```

The `::geography` cast makes `ST_Distance` return *metres on the real Earth*, not degrees —
always use it for distance queries. (`ST_Distance` on geometry with 4326 returns degrees;
`::geography` returns metres.)

### Layer 2 — tippecanoe & PMTiles: vector tiles

**What:** tippecanoe converts GeoJSON → vector tiles (`.pmtiles` or `.mbtiles`), generalizing
geometry per zoom so tiles stay small. **PMTiles** is the modern single-file format: one file
on a CDN serves every zoom level via HTTP range requests — no tile server needed.

**Why you need it:** this is how OSM detail (roads, trails, rivers, peaks) reaches the browser
as interactive layers. Self-hosted PMTiles = zero per-request cost and full styling control.

**Prove it works:**

```bash
# Convert GeoJSON features to a single PMTiles file
# -zg = "guess best max zoom from data density"; -Z6 = min zoom
tippecanoe -zg -Z6 -o nepal_pois.pmtiles \
  --drop-densest-as-needed --force \
  peaks.geojson trails.geojson rivers.geojson

# inspect the result
tippecanoe-decode nepal_pois.pmtiles 0 0 0   # decode one tile
```

Serve `nepal_pois.pmtiles` from any static host (GCS + CDN, or `npx pmtiles serve`) and point
MapLibre at it with a source. One file, all zoom levels, no backend.

### Layer 3 — titiler / rio-tiler: raster serving & analytics

**What:** `rio-tiler` reads COGs and returns tiles or statistics; **titiler** wraps it in a
FastAPI server so a COG in GCS becomes a live XYZ tile endpoint *and* an analytics endpoint.

**Why you need it:** the land-cover "time machine" (22 annual rasters) and forest canopy
layers are rasters. titiler turns one COG per year into both the display layer and the
"land-cover change in this area" calculator.

**Prove it works:**

```bash
uv pip install "titiler.application"
uvicorn titiler.application.main:app --port 8000
```

```text
# display — get a tile for zoom/x/y
GET http://localhost:8000/cog/tiles/WebMercatorQuad/{z}/{x}/{y}
  ?url=https://storage.googleapis.com/.../landcover_2022_cog.tif

# analytics — land-cover statistics for a bounding box
GET http://localhost:8000/cog/statistics
  ?url=https://storage.googleapis.com/.../landcover_2022_cog.tif
  &bbox=83,27,85,29
```

The same COG serves the map *and* the analytics panel — that's the COG superpower from §2.5,
and the reason we don't pre-render 22 years of raster tiles.

### Layer 4 — MapLibre GL JS: the renderer

**What:** the open-source WebGL2 map engine (the free successor to Mapbox GL). It consumes
vector tiles, raster tiles, terrain, and GeoJSON, and draws them with a declarative **style
spec** (sources + layers + expressions).

**Why you need it:** it is the thing the user actually sees. Every "cool UI" feature —
terrain, hillshade, time-slicing, click-to-inspect, fly-to — is MapLibre. Learn it well; it
replaces the entire Mapbox/Google pricing problem.

**Prove it works** (a complete Nepal map with terrain in one HTML file):

```html
<!doctype html>
<html>
<head>
  <link rel="stylesheet" href="https://unpkg.com/maplibre-gl@4/dist/maplibre-gl.css">
  <script src="https://unpkg.com/maplibre-gl@4/dist/maplibre-gl.js"></script>
  <style>body{margin:0}#map{position:absolute;inset:0}</style>
</head>
<body>
<div id="map"></div>
<script>
  const map = new maplibregl.Map({
    container: 'map',
    style: 'https://demotiles.maplibre.org/style.json',   // free base style
    center: [83.9, 28.4],   // Nepal (lon, lat)
    zoom: 7,
    maxPitch: 85
  });

  map.on('load', () => {
    // 3D terrain from free AWS tiles (terrarium-encoded elevation)
    map.addSource('dem', {
      type: 'raster-dem',
      url: 'https://s3.amazonaws.com/elevation-tiles-prod/terrarium/{z}/{x}/{y}.png',
      tileSize: 256, maxzoom: 15
    });
    map.setTerrain({ source: 'dem', exaggeration: 1.5 });

    // a GeoJSON layer — the Annapurna peaks
    map.addSource('peaks', {
      type: 'geojson',
      data: { type: 'FeatureCollection', features: [
        { type: 'Feature', properties: { name: 'Annapurna I', el: 8091 },
          geometry: { type: 'Point', coordinates: [83.82, 28.59] } },
        { type: 'Feature', properties: { name: 'Machapuchare', el: 6993 },
          geometry: { type: 'Point', coordinates: [83.95, 28.49] } }
      ]}
    });
    map.addLayer({
      id: 'peaks-circles', type: 'circle', source: 'peaks',
      paint: { 'circle-radius': 6, 'circle-color': '#ff5e5e' }
    });

    // click-to-inspect (the anti-"pretty map" feature)
    map.on('click', 'peaks-circles', (e) => {
      const f = e.features[0].properties;
      new maplibregl.Popup().setLngLat(e.lngLat)
        .setHTML(`<b>${f.name}</b><br>${f.el} m`).addTo(map);
    });
  });
</script>
</body>
</html>
```

Run `open nepal.html` — you now have a 3D Nepal map with clickable peaks in ~50 lines.

### Layer 5 — FastAPI: the analytics API

**What:** the Python web framework that exposes the *queries* — search, feature profiles,
elevation profiles, land-cover change, proximity.

**Why you need it:** MapLibre handles *rendering*; FastAPI handles *analysis*. The golden rule
from architecture §12: "render path (tiles, CDN) vs analytics path (SQL, FastAPI)" — never run
a spatial join on a mouse-move; run it behind an endpoint and cache it.

```python
# app.py
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

@app.get("/search")
async def search(q: str):
    # PostGIS trigram/tsvector search over places, peaks, rivers, trails
    ...

@app.get("/terrain/profile")
async def terrain_profile(line: str):
    # sample DEM along a LineString -> list of [distance_m, elevation_m]
    ...

@app.get("/proximity")
async def proximity(feature_id: int, distance_m: int):
    # features within distance_m of feature_id
    ...
```

### Layer 6 — React + TypeScript integration

MapLibre has no React dependency — it's an imperative API. The correct pattern is: **mount the
map once in a `useEffect`, keep the `map` object in a ref, and drive everything from React
state → `map.setPaintProperty` / `setStyle` / `flyTo`.**

```tsx
import { useRef, useEffect } from 'react';
import maplibregl from 'maplibre-gl';
import 'maplibre-gl/dist/maplibre-gl.css';

export function Map() {
  const container = useRef<HTMLDivElement>(null);
  const mapRef = useRef<maplibregl.Map>();

  useEffect(() => {
    if (!container.current || mapRef.current) return;
    mapRef.current = new maplibregl.Map({
      container: container.current,
      style: 'https://demotiles.maplibre.org/style.json',
      center: [83.9, 28.4], zoom: 7
    });
    mapRef.current.on('load', () => { /* add sources/layers */ });
    return () => mapRef.current?.remove();
  }, []);

  const flyTo = (lon: number, lat: number) =>
    mapRef.current?.flyTo({ center: [lon, lat], zoom: 11 });

  return <div ref={container} style={{ width: '100%', height: '100vh' }} />;
}
```

---

## 4. The "cool UI" recipes — concrete code for each flagship feature

Each recipe is the minimal mechanism; they compose into the full product.

### 4.1 Time slider (land-cover 2000→2022)

Swap the raster source's `url` on slider input. MapLibre re-fetches just the visible tiles.

```js
const years = [2000,2001,2002,2003,2004,2005,2006,2007,2008,2009,2010,2011,
               /* 2012 missing */ 2013,2014,2015,2016,2017,2018,2019,2020,2021,2022];

map.addSource('landcover', {
  type: 'raster', tileSize: 256,
  tiles: [`https://you/cog/tiles/WebMercatorQuad/{z}/{x}/{y}?url=.../lc_2000.tif`]
});
map.addLayer({
  id: 'landcover-layer', type: 'raster', source: 'landcover',
  paint: { 'raster-opacity': 0.7 }
});

slider.addEventListener('input', (e) => {
  const y = years[+e.target.value];
  const src = map.getSource('landcover');
  src.tiles = [`https://you/cog/tiles/WebMercatorQuad/{z}/{x}/{y}?url=.../lc_${y}.tif`];
  src.update();   // hot-swap the year — no full reload
});
```

### 4.2 Elevation profile (trail/route)

Sample the DEM along the route server-side, return `[[dist_m, elev_m], ...]`, draw with d3.

```python
# FastAPI endpoint: sample DEM along a GeoJSON LineString
import rasterio
from rasterio import warp

@app.get("/terrain/profile")
async def terrain_profile(line: str):
    # line = encoded or POSTed GeoJSON LineString
    # walk the line at ~50m intervals, sample DEM at each point
    # return [(distance_m, elevation_m), ...]
    ...
```

### 4.3 Compare mode (two years side-by-side)

Two synced `maplibregl.Map` instances: when one moves, update the other's center/zoom.

```js
[mapA, mapB].forEach(m =>
  m.on('move', () => {
    const other = m === mapA ? mapB : mapA;
    if (Math.abs(other.getZoom() - m.getZoom()) > 0.01)
      other.jumpTo({ center: m.getCenter(), zoom: m.getZoom() });
  })
);
```

### 4.4 Land-cover transition Sankey

`GET /landcover/change` returns a class→class transition matrix (computed with `rasterio`
zonal stats over two years); feed the matrix to `d3-sankey`.

### 4.5 Search with fly-to

PostGIS gazetteer returns `{name, type, lon, lat}` → `map.flyTo({ center, zoom: 11,
essential: true })`.

---

## 5. Learn-by-building path (mirrors the real phases)

| Step | Build | You learn | Time |
|---|---|---|---|
| **Day 1** | The Layer-4 HTML file: Nepal map + terrain + 3 clickable peaks | MapLibre init, sources, layers, terrain, events | 2–3 h |
| **Day 2** | Load real OSM Nepal: download PBF → tippecanoe → serve PMTiles → add trails/rivers layers | Vector tiles, PMTiles, styling | 3–4 h |
| **Day 3** | Load one ICIMOD land-cover year as a COG → titiler → raster layer | COG, titiler, raster sources | 3–4 h |
| **Day 4** | Add the 2000→2022 slider (hot-swap raster source) | Time-slicing, source.update | 2 h |
| **Day 5** | PostGIS: load admin boundaries + peaks; build `/search` and `/proximity` | PostGIS, spatial SQL, FastAPI | 4 h |
| **Day 6** | Click-to-inspect: `queryRenderedFeatures` → `/places/{id}` → panel | API integration, feature profiles | 3 h |
| **Day 7** | Elevation profile (DEM sampling) + a d3 chart | Raster sampling, charting | 3 h |
| **Week 2+** | Compare mode, Sankey, URL state, provenance drawer, auth, deploy to Cloud Run | Polish + productization | — |

Each step is independently demoable — you have a working artifact at the end of *every* day,
never a half-finished monolith.

---

## 6. Common pitfalls (the ones that will actually waste your day)

1. **`[lat, lon]` vs `[lon, lat]`.** GeoJSON and MapLibre both use `[lon, lat]`. Get it
   backwards and Nepal renders off the coast of Africa. Memorize: **X before Y**.
2. **CRS mismatch.** If tiles don't line up or features are in the wrong hemisphere, the
   first thing to check is `gdalinfo` / `ST_SRID`. The ICIMOD data is 4326; tiles are 3857.
3. **Node too old.** Your machine is on Node 10 — MapLibre/Vite will throw cryptic
   `SyntaxError` on modern syntax. Upgrade to Node 20 before anything else.
4. **Python 3.14 + rasterio.** Wheels lag new Python releases. Use `uv` to pin 3.12.
5. **Proxying ICIMOD `export` in production.** It works live (verified), but a foreign
   MapServer in your critical path is fragile. Mirror to COGs once, serve yourself.
6. **2012 land-cover gap.** The series skips 2012. Show the gap on the slider; never
   fabricate a 2012 layer.
7. **`ST_Distance` returning degrees.** Cast to `::geography` to get metres, or your
   "within 5 km" becomes "within 5 degrees" (≈ 555 km).
8. **Sending raw geometry to the browser.** OSM Nepal is ~393 MB. Always serve tiles
   (PMTiles/Martin), never raw GeoJSON, or the page crawls.
9. **ODbL on OSM.** If you redistribute an OSM-derived database, share-alike applies. Keep
   attribution visible and treat your *derived metrics* as your own computation — but don't
   hide the OSM geometry lineage.

---

## 7. Where to go next

- **MapLibre style spec** — the single most valuable reference: `maplibre.org/maplibre-style-spec/`
- **PostGIS docs** — `postgis.net/docs/` (the spatial SQL reference)
- **GDAL** — `gdal.org/programs/` (every command, exhaustively documented)
- **tippecanoe** — `github.com/mapbox/tippecanoe`
- **titiler** — `developmentseed.org/titiler/`
- **Copernicus DEM** — `registry.opendata.aws/copernicus-dem/`
- **AWS Terrain Tiles** — `registry.opendata.aws/terrain-tiles/`

When you're ready to go from "learned it" to "built it for real," the phased sequence in
`docs/10-technical-architecture.md` §12 is the checklist.

---

*Setup instructions verified against your actual machine (Node 10.14.1, Python 3.14.5,
Homebrew 6.0.2, no GIS tools) — August 2026. Data sources verified live in the architecture doc.*