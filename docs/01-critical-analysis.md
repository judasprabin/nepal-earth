# 01 — Critical Analysis: Idea, Market & Feasibility

**Nepal Earth** — Interactive Geospatial Intelligence Platform for Nepal
*Prepared from four seats: product strategist, startup analyst, market researcher, technical architect.*
*Analysis date: 14 Aug 2026.*

---

## 0. Executive verdict

> **`Portfolio Only`** — build it as a production-grade geospatial data platform and portfolio
> centerpiece. Do **not** treat it as a startup: there is no demonstrated buyer, no moat,
> free substitutes, and a legal gray zone around redistributed government data.

The blueprint is a **9/10 engineering document** and a **2/10 business document**. Every
technical instinct is sound; every commercial claim is unvalidated. This asymmetry is the
single most important fact about the idea, and the rest of this report explains why.

---

## 1. What the idea actually is

The project fuses Nepal's *already-public* geospatial data into one normalized spatial model
and exposes it through a 3D WebGL map with derived analytics.

**Facts (verified against live sources):**

- The data is real and mostly free. Nepal's National Geoportal exposes a GeoServer with 31
  layers (hydrography, land cover, contours, buildings, transport, local units); ICIMOD
  exposes ArcGIS REST services (land cover 2000–2022, forest type, physiography, admin);
  NTNC exposes 114 vector layers; OpenStreetMap's Geofabrik Nepal extract is current to
  July 2026.
- Licensing is mostly permissive: ICIMOD datasets are CC BY 4.0; OSM is ODbL.
- Every *component* the product is built from already exists, separately, for free.

**Assumption (unvalidated):** that fusing them into "one explorable system" is a moat.
It is not, by itself — aggregation is replicable in weeks by anyone who reads the blueprint.

---

## 2. Problem & target users

The blueprint never names a user with a **job to be done and a budget**. It lists six
"potential markets" as a wish-list, not a customer. Forcing the question:

| Segment | Real problem? | Would they pay? | Verdict |
|---|---|---|---|
| Trekking / tourism | Yes — route planning, elevation, permit context | Weak. PeakVisor/AllTrails already serve this, with mobile apps | Not the wedge |
| Conservation NGOs (WWF, NTNC, WCN) | Yes — change monitoring | No budget; Global Forest Watch is free | No revenue |
| Hydropower / planning consultants | Yes — terrain & accessibility | Maybe, but they buy Esri/ArcGIS | Incumbent-locked |
| Disaster / insurance | Yes — landslide & flood exposure | **Strongest** — but needs validated ML, not a map viewer | The only real wedge |
| Government / NGO analytics | Yes — dashboards | Donor-funded, procurement-heavy, slow | Long-tail |
| "General consumer curiosity" | No | Zero willingness to pay | None |

**Critical finding:** the only user who is *guaranteed* to exist is **a hiring manager
looking at a portfolio**. That is the one user the idea serves today, and it serves them
well. Everything else is speculation.

---

## 3. Market demand & size

- **Nepal trekking/tourism:** ~1.2M tourist arrivals (2024, post-COVID recovery), several
  hundred thousand trekkers across Annapurna/Everest/Langtang. A real but *small, seasonal,
  price-sensitive* consumer market already captured by global apps.
- **Nepal GIS/geospatial B2B:** tiny. A handful of hydropower consultants, INGOs, and
  government programs. Esri + donor-funded custom apps absorb most of it. No evidence of
  unmet commercial demand for a "Nepal digital twin."
- **Global EO/GIS:** $10B+ and growing, but owned by Esri, Planet, Maxar, Google. A solo
  operator does not enter this market with a MapLibre front end.

**Conclusion:** no evidence of a commercially meaningful, addressable, *unserved* market
for a Nepal-only map explorer. The blueprint provides **zero market evidence** — no customer
interviews, no willingness-to-pay tests, no pricing research. This is the single biggest gap.

---

## 4. Competitors & alternatives

The blueprint lists *data sources* as if they were the competitive field. The real
competitors are *products* (full teardown in `docs/03-competitor-teardown.md`):

| Competitor | Threat | Price |
|---|---|---|
| Global Forest Watch (WRI) | **Kills the "Landscape Time Machine"** — land-cover change, globally, already | Free |
| Google Earth Engine / Earth | Kills "time slider" + "3D terrain" for casual use | Free (research) |
| PeakVisor | Kills "Mountain Profile" + 3D consumer terrain | Freemium, ~$30/yr |
| AllTrails / Gaia / Komoot | Kills "Trail Intelligence" | Freemium, ~$36/yr |
| Esri ArcGIS + Living Atlas | Owns the paying B2B market | ~$100–850/user/yr |
| Sentinel Hub / EO Browser | Subsumes raster analytics for technical users | Freemium + credits |
| SERVIR / ICIMOD portals | The *source data itself*, with its own viewers | Free |
| Nepal National Geoportal | Government catalogue | Free |

**Structural problem:** the differentiation is "bring them together + derive analytics," but
(a) GFW already does the flagship temporal-analytics feature for free, globally; (b) consumer
incumbents have mobile apps, offline mode, and decade-long content moats; and (c) the paying
B2B buyers are locked into Esri.

---

## 5. Differentiation (claimed vs. real)

- **Claimed:** "intelligence layer" that connects terrain, rivers, forests, land cover,
  protected areas, trails, settlements, and time-series change into one explorable system.
- **Real:** a nice unified viewer + a few derived raster metrics (zonal stats, transitions,
  elevation profiles). All standard operations on public data.
- **Conclusion:** differentiation by *aggregation* is not durable. Differentiation must come
  from *proprietary derived data* (a validated risk model), *exclusive access*, or *brand* —
  none of which the blueprint establishes.

---

## 6. Business model & monetization

The blueprint itself concedes "the consumer map is unlikely to be the strongest
monetization path." The alternatives it lists are all B2B and unpriced.

- **Realistic:**
  1. Free public tool as **lead-gen into consulting** (plausible for a solo engineer).
  2. A niche **ML risk API** sold to insurers/INGOs (the disaster-exposure wedge).
- **Unrealistic:**
  - Consumer subscriptions (PeakVisor/AllTrails own it).
  - Government procurement (2+ year sales cycles, RFPs, no solo-operator path).

There is **no validated revenue**. The most credible "monetization" is *portfolio → job /
consulting*, which is not a business model — it is a career asset.

---

## 7. Technical & data requirements (where the blueprint is strongest)

- **Stack** (PostGIS + FastAPI + React/MapLibre + COG rasters + Redis + K8s CronJobs) is the
  correct modern geospatial architecture.
- **Data model, API design, performance strategy, ingestion pipeline** are all sound and
  production-grade in intent.
- **Correct instincts:** rasters-in-object-storage + COGs; vector tiles over raw geometries;
  render-vs-analytics path separation; license/attribution registry; "unknown instead of
  invented" uncertainty discipline.
- **Correct cautions:** ODbL share-alike; Geoportal/NTNC redistribution terms; topology
  cleaning for river networks; "don't imply official hazard ratings."

**Development complexity: 6–7/10.** Achievable for a competent ML/backend engineer. The hard
20% is the NLCMS 22-year raster change pipeline, river topology, and 3D terrain tuning. This
is a **4–8 week focused build**, and the phased roadmap is realistic.

**Scalability: not a concern.** Nepal-scale data fits on a single PostGIS node and a few
hundred GB of COGs. The K8s/Terraform ceremony is portfolio-signaling, not necessity — a
$50–100/month Cloud Run + Cloud SQL + GCS setup serves the whole thing.

---

## 8. Legal risks

- **ODbL share-alike (OSM):** redistributing derived OSM data can force your product to be
  ODbL-licensed too. The blueprint flags it; the implication (your "intelligence layer" could
  be legally forced open) is under-weighted.
- **National Geoportal & NTNC:** no clear redistribution terms. "Viewable online ≠
  redistributable." A public product mirroring these layers is legally gray.
- **Misrepresentation:** a derived "risk score" misread as official hazard guidance in a
  disaster-prone country is a genuine liability. The "labelled modelled index" mitigation is
  right but only works if you stay disciplined.
- **Data provenance** is the single most impressive (and most defensible) thing in the
  blueprint — keep it.

---

## 9. Customer acquisition & defensibility

- **CAC:** no answer, because there is no customer. For consumer play, CAC against
  PeakVisor/AllTrails' brand + app-store distribution is unsustainably high for a Nepal-only
  product. For B2B, it's a long-tail sales slog.
- **Defensibility: low.** The moat claim is "normalized spatial model + derived analytics +
  knowledge graph," but the data is public, the analytics are standard raster ops, and the
  knowledge graph is a relational table (the blueprint itself says no Neo4j needed). No data
  exclusivity, no network effect, no switching cost. Full analysis in
  `docs/02-moat-and-defensibility.md`.

---

## 10. Risks, weaknesses, opportunities, unknowns

**Biggest risks**
1. **Free substitutes** — GFW does the flagship time-series feature for free, globally.
2. **No buyer** — no one is demonstrated to pay for this.
3. **Aggregation ≠ moat** — replicable.
4. **Legal gray zone** on redistributed gov/NTNC layers.
5. **Scope creep into ML** before the spatial core is proven.

**Weaknesses**
- Thin market analysis; no competitor *product* research; no pricing; no WTP evidence.
- Over-indexed on WebGL "wow" vs. a real job-to-be-done.
- K8s/Terraform overkill = infrastructure theater for a solo build.

**Opportunities**
1. **The portfolio is genuinely exceptional** — geospatial data engineering + PostGIS +
   raster processing + 3D viz + ML is a rare, in-demand, demonstrable skill set.
2. **The disaster-exposure wedge** (landslide susceptibility, flood exposure) is a real,
   under-served, fundable problem in Nepal if pursued as a separate, validated ML product.

**Unknowns (must resolve before any business claim)**
- Does any trekking agency, NGO, or hydropower consultant actually want this?
- What are Geoportal/NTNC's actual redistribution terms?
- Is there any sponsor (INGO/donor) who'd fund a Nepal geospatial dashboard?

---

## 11. Scorecard

| Dimension | Score | Rationale |
|---|---|---|
| Market potential | **3 / 10** | Real segments exist but tiny, fragmented, mostly free/incumbent-served |
| Business feasibility | **2 / 10** | No demonstrated buyer, free substitutes, no pricing/WTP evidence |
| Technical feasibility | **8 / 10** | Data real & accessible; stack standard; author has the skills |
| Competitive position | **2 / 10** | Free incumbents (GFW, PeakVisor, Earth Engine) cover most features |
| Portfolio value | **9 / 10** | Exceptional, rare, demonstrable skill stack for EO/geo ML hiring |

**Weighted read:** 8–9 as an engineering artifact; 2–3 as a business.

---

## 12. Verdict

**`Portfolio Only`.**

Build it — the blueprint is a strong plan, the build is within capability, and as a portfolio
piece it will outperform almost anything else a backend/ML engineer could ship. But do **not**
treat it as a startup. The only path from portfolio to business is a pivot to the
**disaster/insurance exposure model**, which is a different product needing its own
`Validate First` cycle.

*Next: `docs/02-moat-and-defensibility.md` — how to actually find a moat.*
