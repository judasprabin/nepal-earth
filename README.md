# Nepal Earth — Strategy & Feasibility Analysis

**Interactive Geospatial Intelligence Platform for Nepal**

This repository is the *strategy layer* for the **Nepal Earth** product idea — a critical,
evidence-based analysis of the idea's market, feasibility, and defensibility, written from
four seats: product strategist, startup analyst, market researcher, and technical architect.

> **Verdict (one line): `Portfolio Only`** — build it as a production-grade geospatial data
> platform and portfolio centerpiece, not as a startup. The blueprint is an excellent
> *engineering* document and a weak *business* document.

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

---

## Scorecard (out of 10)

| Dimension | Score |
|---|---|
| Market potential | **3** |
| Business feasibility | **2** |
| Technical feasibility | **8** |
| Competitive position | **2** |
| Portfolio value | **9** |

**Verdict: `Portfolio Only`**

The one defensible business wedge — a *validated* landslide/flood exposure risk product for
Nepal's disaster-prone landscape — is a different product that needs its own
`Validate First` cycle (see `docs/02-moat-and-defensibility.md` §5 and
`docs/05-validation-plan.md` §4).
