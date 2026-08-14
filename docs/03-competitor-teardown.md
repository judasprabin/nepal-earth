# 03 — Competitor Teardown & Pricing

*Competitive facts verified against live sources on 14 Aug 2026 unless marked "approx."*

The blueprint's §2 lists **data sources** as if they were the competitive field. They are not.
The competitive field is made of **products** that already solve pieces of the problem — and
critically, the strongest ones are **free**. This document is the product-level teardown the
blueprint was missing.

---

## 1. The competitive map

```
                     FREE (no revenue)                     PAID (revenue exists)
                 ┌──────────────────────────────┐   ┌──────────────────────────┐
  Consumer       │ Google Earth / Earth Engine   │   │ PeakVisor (~$30/yr)      │
  / prosumer     │ Global Forest Watch           │   │ AllTrails (~$36/yr)      │
                 │ SERVIR / ICIMOD viewers       │   │ Gaia GPS / Komoot        │
                 │ Nepal National Geoportal      │   │                          │
                 └──────────────────────────────┘   └──────────────────────────┘
                 ┌──────────────────────────────┐   ┌──────────────────────────┐
  Enterprise     │ (none free)                  │   │ Esri ArcGIS (~$100–850/  │
  / B2B          │                              │   │   user/yr)               │
                 │                              │   │ Sentinel Hub (credits)   │
                 │                              │   │ Planet / Maxar (imagery) │
                 └──────────────────────────────┘   └──────────────────────────┘
```

**The structural problem:** Nepal Earth is positioned in the *free* consumer quadrant, where
the incumbents are free and well-funded, while the *paid* quadrants (B2B) are owned by Esri
and require sales teams. There is no obvious empty quadrant for "a solo-built Nepal-only map."

---

## 2. Competitor-by-competitor

### 2.1 Global Forest Watch (WRI) — **the most direct, and most dangerous, competitor**
- **What it is:** a free global platform for forest monitoring: tree-cover loss/gain
  time-series, deforestation & fire alerts, custom analysis, downloadable data.
- **Price:** **Free** (funded by WRI / donors / foundations).
- **Nepal Earth overlap:** *kills the flagship feature.* The blueprint's "Landscape Time
  Machine" (land-cover change 2000–2022) is precisely what GFW already does for all of Nepal,
  with a bigger team, more data sources, and a known brand.
- **Verified:** live site exposes "Tree Cover Loss Analysis," "Deforestation Alerts,"
  dashboards, and monitoring tools at globalforestwatch.org.

### 2.2 Google Earth Engine / Google Earth
- **What it is:** planetary-scale satellite imagery + analysis platform; Timelapse (a literal
  global "time machine"), 3D terrain in Google Earth.
- **Price:** **Free** for research/non-commercial; paid for commercial compute.
- **Nepal Earth overlap:** kills "time slider" and "3D terrain" for any casual/technical user.
- **Implication:** a Nepal-only time slider cannot win on novelty against a global Timelapse.

### 2.3 PeakVisor
- **What it is:** 3D mountain maps, peak identification, terrain/elevation profiles, offline
  maps, hiking trails — exactly the "Mountain Profile" + 3D consumer terrain features.
- **Price:** **Freemium; PRO tier** — historically ~$30/yr or ~$90 lifetime (approx.; site
  confirms a "PeakVisor Lifetime PRO" tier exists).
- **Nepal Earth overlap:** high, for the consumer mountain-exploration angle.
- **Verified:** site confirms 3D maps, peak ID, offline, and a PRO tier.

### 2.4 AllTrails / Gaia GPS / Komoot
- **What it is:** trail discovery, routing, elevation profiles, offline maps.
- **Price:** **Freemium** — AllTrails+ ~$36/yr (approx., well-documented); Gaia/Komoot similar.
- **Nepal Earth overlap:** kills "Trail Intelligence" for the consumer trekking audience.
- **Implication:** these have mobile apps + offline + community content; a web-only Nepal
  trail feature cannot compete on distribution.

### 2.5 Esri ArcGIS Online + Living Atlas
- **What it is:** the dominant enterprise GIS platform; Living Atlas includes Nepal datasets;
  Story Maps for narrative; hosted services for dashboards.
- **Price:** **~$100–850/user/yr** depending on tier (Creator/Professional) (approx.,
  well-documented).
- **Nepal Earth overlap:** owns the *paying* B2B market (hydropower, planning, gov). Buyers
  here do not leave Esri for a MapLibre app.

### 2.6 Sentinel Hub / EO Browser
- **What it is:** on-demand satellite imagery processing and visualization (Sentinel, Landsat),
  cloud-based, with APIs.
- **Price:** **Freemium** (trial credits) → commercial credit packages.
- **Nepal Earth overlap:** subsumes the raster-analytics layer for any technical user.

### 2.7 SERVIR / ICIMOD portals & Nepal National Geoportal
- **What they are:** the *source data itself*, exposed with their own viewers and APIs.
- **Price:** **Free.**
- **Nepal Earth overlap:** the "competitor" is the data source. ICIMOD/SERVIR could ship a
  better viewer than a solo operator, and they already own the data pipeline.
- **Implication:** any advantage must come from *derived, proprietary* analytics, not from
  re-serving their layers.

---

## 3. The free-substitute problem, stated plainly

Nepal Earth's entire value proposition — "see land-cover change over time, explore terrain,
inspect forests" — can be experienced today, for free, across GFW + Google Earth + PeakVisor's
free tier. **A product whose core value is available for free has no pricing power.**

This is why the scorecard lands at **Competitive position 2/10** and **Business feasibility
2/10**. The only way out is to build something the free tools *cannot* produce: a **validated,
ground-truthed risk model** (see `docs/02-moat-and-defensibility.md` §5).

---

## 4. Competitive gaps (where free tools actually fall short)

Being honest about the flip side — these are the genuine gaps a new entrant *could* exploit:

1. **No tool fuses Nepal's *authoritative* layers** (geoportal + ICIMOD + NTNC) into one
   queryable, attributable view with provenance. GFW is forest-only; Google Earth is global
   and shallow on Nepal-specific institutional layers.
2. **No Nepal-specific validated hazard layer** (landslide/flood exposure) exists as a
   commercial product. This is the real whitespace.
3. **Attribution/trust is under-served** — the blueprint's "data provenance drawer" is
   genuinely rare and valuable, especially for institutional/INGO audiences.

These gaps are real but narrow. Gap 2 is the only one with a business; gaps 1 and 3 are
portfolio/credibility features, not revenue engines.
