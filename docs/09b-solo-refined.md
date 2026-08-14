# 09b — Critical Review of "Find My Nepal" & the Refined Solo-Startup Plan

*Critical analysis of `docs/09-openai-idea-2.md`, the missing execution layer, and a
concrete startup plan that solves the cold-start problem with ML and computable data.*
*Analysis date: 14 Aug 2026.*

---

## 0. TL;DR

Doc 09-idea-2 is a **vision doc**, not an execution plan. Its strengths are real — "long-tail
destination discovery," the geospatial engine as moat, and the flywheel loop. Its gap is
fatal to a solo builder: **it assumes the content exists.** It doesn't.

The fix is not to add content. It's to **remove the content dependency entirely** — build a
product whose entire value comes from *computable* facts (elevation, forest, canopy, trails,
teahouses, biodiversity), scored by ML, and narrated by an agent over auto-generated
evidence. No manual curation. No "destination profiles." No cold-start.

**Verdict:** `Build` — with the refined plan below, starting with ONE region (Annapurna),
ONE activity (teahouse trekking), and ONE question ("what kind of trek fits me?"), powered
by ML from day one.

---

# PART A — CRITICAL ANALYSIS OF DOC 09-idea-2

## A1. What's genuinely strong (and should survive)

| Insight | Why it's right |
|---|---|
| "Don't compete on Everest. Help people discover the long tail." | Nepal Tourism Board is actively pushing under-promoted destinations. This is a validated government goal, plus overtourism in Everest/Annapurna is real. |
| "The geospatial engine is the moat; the LLM isn't." | The single most important sentence in the doc. Correct. |
| "Discovery → Recommendation → Planning, not booking." | Right for a solo builder. Avoids marketplace complexity. |
| The flywheel loop (traveler → visits → data improves → better recommendations) | A real, rare network effect for a solo product. |
| "Find My Nepal" — one screen, sliders, 5 results | Correct MVP scope instinct. |
| The "Nepal by ecosystem" taxonomy | Genuinely novel vs. any existing travel app. |
| Ranking scores (solo feasibility 8, uniqueness 9, business 8, technical 10) | Honestly assessed, not inflated. |

## A2. The fatal gap: the content cold-start

The doc describes a product where a user enters constraints and receives 5 ranked
destinations, each with:

> Match score · Why it matches · Map · Things to do · Nature · Culture · Best season ·
> Difficulty · How to get there · 3-day/5-day itinerary

**Every one of these requires data that does not exist in a structured, queryable form
today.** "Things to do," "Culture," "Best season" and "How to get there" are **content** —
someone must write them, and that someone can't be a solo ML engineer at launch.

The doc never answers: *where does the per-destination structured data come from?*

## A3. What else is missing or under-specified

1. **The 6-month timeline is optimistic** for a part-time solo builder with other projects
   (Saathi, scan-service). Months 4–6 (scoring engine + agent + itinerary generation) are
   each 4–6 week undertakings, not 4 weeks. Realistic timeline: 8–12 months part-time.

2. **The scoring engine is underspecified.** How does "Mountains 92, Forest 96, Wildlife 84"
   get computed? If it's hand-labeled for each destination → content problem again. If it's
   computed → ML/feature-engineering problem the doc doesn't address.

3. **Monetization is "later" with no trigger.** "Affiliate revenue" for Nepal hotels is a
   high-traffic, low-margin game. The site would need serious organic traffic before
   affiliate revenue is material. The doc doesn't estimate when that happens.

4. **No ML.** The user's core differentiator is ML engineering. A feature-weighted scoring
   engine is not ML; it's a spreadsheet. The doc describes a product that an ML engineer
   *can* build, not a product that *needs* an ML engineer.

5. **The "Nepal by ecosystem" concept is strong but has the exact same content cold-start**
   as the destination profiles. Ecosystems need maps, descriptions, biodiversity summaries,
   representative places — all content.

6. **The flywheel loop is real but slow.** It requires users to visit, return, and contribute
   data before the graph compounds. But the app needs a useful baseline *before* users will
   engage. This chicken-and-egg is not acknowledged.

## A4. The one-sentence critique

> The doc describes the product at maturity but doesn't solve the *first user's first visit* —
> which is the only problem that matters for a solo builder.

---

# PART B — THE REFINED PLAN (the "really good to start" version)

## B1. The fix: remove the content dependency entirely

The core insight: **computable geospatial facts do not need content curation.** Every number
below is a spatial query, not something someone wrote:

- Elevation range → DEM query
- Forest cover / canopy % → ICIMOD raster query
- Trail count and length → OSM vector query
- Teahouse count and spacing → OSM + enrichment
- Protected-area overlap → spatial intersection
- Species count nearby → GBIF precompute
- Seasonality → derived from elevation + climate zone
- Distance from trailhead / city → routing query
- Crowding score → teahouse density + trail popularity model

These are **free at runtime**, **never go stale without a pipeline**, and **don't need a
content writer.** They are the "why this trek" evidence the doc imagines, but generated
deterministically, then narrated by an LLM — not curated by hand.

The product shifts from:
> "Content-first: build beautiful destination profiles, then rank them."

To:
> **"Data-first: compute every fact, score every place, let the agent narrate the evidence.
> No curation. No profiles. Just math."**

## B2. The refined product: "Find Your Trek" (round 1)

**Scope (narrow enough to finish, rich enough to impress):**

- **One region:** Annapurna Conservation Area + surroundings
- **One activity:** teahouse trekking (the dominant trekking mode; ~30 treks/routes)
- **One core question:** *"What kind of trek are you looking for?"*

**The three killer features (all data-first, no content):**

1. **Match score (ML-powered).** Train a ranking model on computable features — elevation
   range, canopy cover %, trail length/difficulty, teahouse density, protected-area overlap,
   GBIF species count nearby, distance-from-road, seasonality window. No hand labels. The
   model learns to map user-preference sliders → ranked treks.

2. **"Why this trek" — auto-generated evidence.** Every trek card displays *computed* facts
   (elevation profile, forest type + canopy %, teahouses/km, proximity to protected areas,
   nearby species). An LLM narrates these facts into plain English, with citations to the
   source layer (ICIMOD, OSM, GBIF). No static descriptions.

3. **Auto-generated itinerary.** The agent splits a trek route into day stages using teahouse
   spacing + elevation gain constraints, and renders it with an elevation profile + GPX
   export. Deterministic; LLM only for narration.

**The UI (one screen, dead simple):**

```
           FIND YOUR TREK IN ANNAPURNA

   How many days?              [  5—12  ]
   Difficulty?                 [ Easy — Strenuous ]
   🏔 Mountains                [ ████████░░ 75 ]
   🌲 Nature / forest          [ █████████░ 90 ]
   🐅 Wildlife possibility     [ ████░░░░░░ 40 ]
   👥 Avoid crowds             [ ██████████ 95 ]
   🏘 Village / culture        [ ██████░░░░ 60 ]

                [ FIND MY TREK ]

   ─────────────────────────────────────────────
   🥇 Khopra Ridge Trek                               94% match
      ▸ 6 days, Moderate, in/+near Annapurna Sanctuary
      🏔 2,000–3,660m  🌲 72% canopy  🐅 58 species nearby
      🥾 12 trails  🏘 4 villages  👥 Low crowds  🛏 8 teahouses
      [Why this trek?] [3-day plan] [Show on map]

   🥈 Mardi Himal Trek                                88% match
      ...
```

## B3. The data sources (all verified, all live)

| Data | Source | What it powers |
|---|---|---|
| Elevation / terrain | Copernicus DEM / SRTM (COG) | Elevation range, profile, slope |
| Forest type, canopy cover %, canopy height | ICIMOD Forest Type MapServer | 🌲 Nature score, forest evidence |
| Trails, teahouses, villages | OSM (Geofabrik Nepal extract) | Trail count, teahouse spacing, village proximity |
| Protected areas | WDPA / ICIMOD / NTNC | Protected-area overlap |
| Species occurrences | GBIF API (~1.98M records) | 🐅 Wildlife score, species count nearby |
| Route descriptions | Wikipedia + Wikivoyage (RAG corpus) | LLM-narrated plain-English explanations |
| Admin boundaries | ICIMOD NepalAdmin | District/region labeling |
| Seasonality | Derived from elevation + climate zone | Best-season indicator |

## B4. The ML (built in from day one, not "later")

| # | Model | Inputs | Output | Technique | What it replaces |
|---|---|---|---|---|---|
| 1 | **Trek ranker** | Computed features per trek × user slider values | Ranked treks with match scores | Feature-weighted scoring v1 → learning-to-rank v2 | Hand-labeled "Nepal Scores" |
| 2 | **Difficulty model** | Elevation gain, distance, trail density, surface (OSM), segment slope distribution | Difficulty score (Easy → Strenuous) | Regression on labeled routes | Subjective difficulty guesses |
| 3 | **Crowding model** | Teahouse density, trail connectivity, distance-from-road, season | Per-trek crowding score | Heuristic v1 → ML with traffic data v2 | No answer today |

**Why this is your edge:** every competing travel app either (a) has hand-curated scores
(subjective, stale) or (b) has no scoring at all. A product where the scores are
**computed, verifiable, and improve with data** is genuinely defensible and can only be built
by someone with geo + ML skills.

## B5. The cold-start solution (how it actually works on day one)

| The doc's unanswered question | The fix |
|---|---|
| "Who writes 50+ destination profiles?" | **Nobody.** Every fact is a spatial query; narration is LLM + citations. |
| "Where do 'Things to do,' 'Culture,' 'Best season' come from?" | **LLM over Wikipedia/Wikivoyage.** Auto-generated from a retrievable corpus, clearly attributed. |
| "Where do photos come from?" | **Don't need them at launch.** Elevation profiles + forest-canopy heatmaps + trail maps *are* the visual identity. The "photo" IS the data viz. |
| "How does the scoring engine work with zero user data?" | **Cold-start with computable features.** Start with a feature-weighted engine; upgrade to learning-to-rank when user feedback (clicks, itineraries) accumulates. |

**The launch truth:** the product is NOT beautiful at launch. It won't have photos. It won't
have "Top 10 Things to Do." What it WILL have — and what no competitor has — is **computed,
verifiable, data-backed evidence for every claim**, and an agent that narrates *why* a trek
matches you with full provenance. That is a credible, differentiated, solo-buildable launch.

## B6. The exact first sprint (4 weeks, part-time)

| Week | Build | Output |
|---|---|---|
| 1 | Load Annapurna terrain, trails, teahouses, forest/canopy, protected areas, admin into PostGIS. Build the computable-feature pipeline: for each ~30 Annapurna treks, compute elevation range, canopy cover %, trail length, teahouse count, protected-area overlap, species count nearby. | A `trek_features` table with all computed fields. |
| 2 | Build the ranking engine v1 (feature-weighted scoring) + the "Find Your Trek" UI: sliders → scores → 5 ranked trek cards with computed evidence. | Working UI with real, computed scores. |
| 3 | Add the agent: deterministic spatial grounding (PostGIS) + LLM narration (prompted to narrate computed facts in plain English, with citations). Add "Why this trek?" and the auto-generated 3-day itinerary. | Full agentic loop working end-to-end. |
| 4 | Polish: elevation profiles (deck.gl), GPX export, shareable trek-card URLs. Eval set: 20 questions scored for factual accuracy (no hallucinated numbers). | Launch-ready. |

## B7. What comes after (months 5–12)

- **Add Everest / Langtang / Manaslu** regions (same pipeline, new datasets).
- **Upgrade the ranker to learning-to-rank** when user click data accumulates.
- **Add the crowdsourcing flywheel:** trekkers who complete a route can upload a review +
  trail-condition photo → the graph improves → better recommendations.
- **Monetize:** (a) teahouse featured listings, (b) "premium itinerary export" (GPX + offline
  PDF + weather integration), (c) guide/porter referral, (d) DMO/municipality demographic
  analytics.
- **Add the biodiversity layer** ("Wild Nepal" — species distribution models, habitat
  suitability; doc 09b's ML model #4).

## B8. The moat (restated, improved)

1. **Cornered resource:** the *cleaned, enriched, continuously refreshed* Nepal geospatial
   knowledge graph. It is public data made *proprietary* through normalization, enrichment,
   and the refresh pipeline.
2. **ML models that improve with data:** the trek ranker, difficulty model, and crowding
   model get better as users interact. This is a compounding asset, not a static dataset.
3. **The impossibility of cold-start replication.** A competitor would need to (a) build
   the same data pipeline, (b) normalize across 5+ inconsistent sources, (c) build the
   computable-feature engine, and (d) train the ML models. That's months of specialized
   geo/ML work that the user already has.

---

# PART C — HONEST LIMITS (a solo builder must acknowledge these)

1. **The TAM is small.** Nepal trekking is niche, even globally. This is a lifestyle/indie
   business, not venture-scale.
2. **Teahouse data is incomplete in OSM.** You will need to enrich it — either by crawling
   trekking blogs, buying a dataset from a trekking agency, or crowdsourcing. This is the
   single biggest data gap and it won't solve itself.
3. **The LLM narration requires guardrails.** The agent must never fabricate a teahouse
   name, a trail length, or a difficulty claim. Every narrated fact must be traceable to a
   deterministic source. The eval set (week 4) is mandatory, not optional.
4. **User acquisition is the unsolved second half.** The product works — but who sees it?
   SEO for "Annapurna trek finder" is competitive. Distribution (social, diaspora
   communities, trekking forums) needs a plan beyond the build.

---

# PART D — VERDICT

**`Build`** — with the refined plan above. The 4-week Annapurna sprint is the smallest
possible unit of work that delivers a differentiated, ML-powered, genuinely useful product.
It eliminates the content cold-start, puts ML at the center, and produces a portfolio
artifact that also validates the commercial wedge.

The doc 09-idea-2's *vision* is correct. This plan is the *execution layer* it was missing.