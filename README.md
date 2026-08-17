# Nepal Earth — Strategy & Feasibility Analysis

**Interactive Geospatial Intelligence Platform for Nepal**

This repository is the *strategy layer* for the **Nepal Earth** product idea — a critical,
evidence-based analysis of the idea's market, feasibility, and defensibility, written from
four seats: product strategist, startup analyst, market researcher, and technical architect.

> **Verdict (one line): `Build` (round 1) → `Validate First` (before commercial round 2).**
> The idea has evolved from "geospatial intelligence" into a sharper product: a **district-level
> atlas (demographics + biodiversity + tourism)** with an **agentic explainer panel**, free in
> round 1, commercial later. That refinement gives it a real — if niche — business tail.
>
> *Earlier verdict (for the original blueprint):* `Portfolio Only` — see doc 01.

---

## What this is

The source blueprint (a "Geospatial Intelligence Project Blueprint" dated 14 Aug 2026)
proposed fusing Nepal's public GIS data — National Geoportal, ICIMOD, NTNC, OpenStreetMap —
into a single PostGIS + raster pipeline with a 3D WebGL front end and derived analytics
(land-cover "time machine", forest structure, terrain profiles, spatial query).

This repo stress-tests that idea against real competitors, real pricing, and real market
evidence, and answers the two questions the blueprint left open:

1. **Is there a business here?** (Short answer: not as framed.)
2. **How do you find a moat — and is one even available?** (This is `docs/moat.md`.)

---

## Document index

| Doc | What it answers |
|---|---|
| [`docs/01-critical-analysis.md`](docs/01-critical-analysis.md) | Full idea / market / feasibility report. Problem, users, demand, competitors, differentiation, business model, tech, complexity, scalability, legal, CAC, defensibility, risks, scores, verdict. |
| [`docs/02-moat-and-defensibility.md`](docs/02-moat-and-defensibility.md) | **How to find a moat** — moat taxonomy, why "aggregation" isn't one, and the concrete paths to a real moat (or the honest conclusion that the moat is the *skill*, not the product). |
| [`docs/03-competitor-teardown.md`](docs/03-competitor-teardown.md) | Competitor-by-competitor analysis with researched pricing and the free-substitute problem. |
| [`docs/04-suggestions-and-roadmap.md`](docs/04-suggestions-and-roadmap.md) | Concrete improvement suggestions: product, positioning, technical, and a phased roadmap. |
| [`docs/05-validation-plan.md`](docs/05-validation-plan.md) | The cheapest way to validate — or kill — the idea before writing more code. |
| [`docs/06-refined-vision.md`](docs/06-refined-vision.md) | **The refined idea** — district-level tourism + demographics + biodiversity atlas with an agentic explainer. Critical analysis, prospective, and multi-technical-mind spec. |
| [`docs/07-app-ideas.md`](docs/07-app-ideas.md) | **5 concrete app ideas** with clear build paths — Trek Copilot, District Showcase, Diaspora Explorer, Wildlife Atlas, full Nepal Earth. |
| [`docs/09-startup-vision.md`](docs/09-startup-vision.md) | **Critical analysis of doc 08 + the bigger-vision startup** — destination-intelligence platform, portfolio-first, ML built in. |
| [`docs/09b-solo-refined.md`](docs/09b-solo-refined.md) | **Critical review of doc 09 + refined solo-startup plan** |
| [`docs/10-grok-refined.md`](docs/10-grok-refined.md) | **"Find Your Trek" PRD** — Grok-refined product spec for the Annapurna trek-discovery wedge. |
| [`docs/11-map-stack-tutorial.md`](docs/11-map-stack-tutorial.md) | **Hands-on map-stack tutorial** — 7-layer progressive guide (GDAL → PostGIS → tippecanoe/PMTiles → titiler/COG → MapLibre → FastAPI → React). |
| [`docs/12-technical-architecture.md`](docs/12-technical-architecture.md) | **Technical architecture research** — verified data sources, stack decisions, build-vs-buy, cost model ($15–130/mo), phased build plan. |

---

## Scorecard (out of 10)

*Doc 01 (original blueprint) → Doc 06 (refined vision):*

| Dimension | Original | Refined |
|---|---|---|
| Market potential | 3 | **5** |
| Business feasibility | 2 | **4** |
| Technical feasibility | 8 | 8 |
| Competitive position | 2 | **4** |
| Portfolio value | 9 | 9 |

**Verdict: `Build` (round 1) → `Validate First` (before commercial round 2).**

The one defensible business wedge is the **tourism-promotion / discovery attention** play
(guesthouses, trekking agencies, municipalities paying for visibility), powered by the
**agentic explainer** as the differentiator. Validate it with 5–10 business interviews before
building any monetization.
