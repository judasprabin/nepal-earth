# 07 — Agentic Map + Travel-Promotion App Ideas (with clear build paths)

*Concrete, buildable product ideas. Each has a named user, a specific agentic layer, data
sources, a stack, a phased build path with time estimates, monetization, and a validation
first-move. Grounded in the data verified in this repo (GBIF, OSM, Nepal Tourism Board,
Census 2021 / Nepal in Data).*

---

## 0. The core loop (what all of these share)

```
      MAP (district-level, layered)  ←  the surface
              │  click / question
              ▼
   AGENTIC EXPLAINER (grounded copilot)  ←  the moat
              │  "here's what's here, and how to experience it"
              ▼
   TRAVEL PROMOTION (routes / treks / stays)  ←  the value
              │  discovery attention
              ▼
   MONETIZATION (business pays for visibility)  ←  round 2
```

Every idea below is a different **wedge** into this loop. They share one foundation (§4), so
building one gets you ~70% of the way to the others.

---

## 1. The idea menu (ranked)

| # | Idea | Wedge | Build speed | Portfolio value | Commercial ceiling | Difficulty |
|---|---|---|---|---|---|---|
| A | **Trek Copilot** — agentic trekking planner | Consumer (trekking) | ⚡ Fast | High | Medium | Medium |
| B | **District Showcase** — tourism-board-as-a-service | B2B (municipalities) | ⚡ Fast | Medium | **Highest (clear $)** | Low |
| C | **Diaspora Home Explorer** | Nepali diaspora | Medium | High | Low-Medium | Medium |
| D | **Wildlife Atlas** — ecotourism copilot | Ecotourism niche | Medium | High | Low | Medium |
| E | **Nepal Earth Atlas** — full 3-layer flagship | Umbrella (all) | Slow | **Highest** | Medium | High |

**Recommendation (§3): build the shared foundation + Idea A (Trek Copilot) first.** It is
the fastest path to a "wow" demo, it exercises the agentic layer hardest (rich grounding),
and it directly feeds Idea B's monetization later.

---

## 2. Each idea, with a clear path

---

### Idea A — **Trek Copilot** (agentic trekking planner)

**One-liner:** "Tell me your days, fitness, and region — I'll plan your Nepal trek: route,
teahouses, permits, elevation profile, and what to see."

**User & job:** independent trekkers (the fastest-growing segment vs. guided groups) who are
overwhelmed by Nepal's fragmented trek info.

**The agentic layer (the point of the app):**
- Conversational planner: *"I have 12 days, moderate fitness, want the Annapurna region."*
- **Deterministic grounding:** real trail network (OSM), elevation gain/loss (DEM), teahouse
  spacing, permit rules (TIMS/ACAP/Sagarmatha), seasonal window.
- **Tools v1:** `find_treks`, `elevation_profile`, `plan_days`, `teahouses_on_route`,
  `permit_checklist`, `seasonal_advice`.
- **Output:** a day-by-day itinerary with a rendered elevation profile, cited sources, and
  export (GPX + PDF).

**Data:**
| Data | Source | License |
|---|---|---|
| Trails | OSM `route=hiking`, `highway=path` | ODbL |
| Elevation | SRTM/Copernicus DEM (COG) | Public |
| Teahouses/lodges | OSM `tourism=*` + manual enrichment | ODbL |
| Permit rules | ACAP / TIMS / DNPWC | Public text (RAG) |
| Trek descriptions | Wikipedia + Wikivoyage + curated blogs | CC BY-SA (RAG) |

**Stack:** FastAPI + PostGIS/Postgres + Redis · React/MapLibre + deck.gl (elevation profile) ·
LLM (Haiku/4o-mini class) with structured tool-calling · RAG over Wikivoyage/permit docs.

**Clear path (phases):**
| Phase | Build | Time |
|---|---|---|
| 1 | Gazetteer + load Annapurna trail network + DEM elevation profiles | 3–4 days |
| 2 | Teahouse POI layer + spacing model ("next lodge in X km") | 2–3 days |
| 3 | RAG corpus (Wikivoyage + permit PDFs) + tool-calling planner agent | 4–5 days |
| 4 | Day-by-day itinerary UI + elevation chart + GPX export | 3–4 days |
| 5 | Eval set (50 questions) + groundedness/citation scoring | 2 days |
| **Total** | | **~3 weeks** |

**Monetization:** teahouse/lodge featured placement; trekking-agency affiliate; premium
"detailed route + offline export" (one-time or subscription).

**Validation first-move:** prototype for **one region (Annapurna)**; show 10 independent
trekkers; ask *"would you plan your trek here instead of a guide/blog?"* — plus interview 5
teahouse owners on paying for placement.

---

### Idea B — **District Showcase** (tourism-board-as-a-service)

**One-liner:** "Give every Nepali district a rich, auto-generated, agentic landing page that
promotes its attractions, routes, and stays."

**User & job:** the 77 district municipalities / local tourism committees who want to attract
tourists but have no budget or skill for a modern web presence.

**The agentic layer:**
- **Auto-generates** the district page from data (census highlights + OSM POIs + Wikivoyage
  + photos).
- **Embedded copilot** ("ask about X district") that municipalities paste into their own site.
- **Maintenance agent** that refreshes listings monthly and flags stale content.

**Data:** Census 2021 (demographic highlights) + OSM POIs + Wikivoyage/Wikipedia + CC photos
(Wikimedia Commons).

**Stack:** same foundation · SSG (Next.js) for per-district pages · the copilot as an
embeddable widget/iframe · CMS-lite for municipality edits.

**Clear path:**
| Phase | Build | Time |
|---|---|---|
| 1 | One district page auto-generated (pick a demo district) | 2–3 days |
| 2 | Embeddable copilot widget (iframe + API key) | 3–4 days |
| 3 | Template across 77 districts + SEO (77 × `/district/kaski`) | 3–4 days |
| 4 | Municipal "claim & edit" flow + refresh pipeline | 3–4 days |
| **Total** | | **~2–3 weeks** |

**Monetization (clearest of all):** municipalities pay a small annual fee (~$50–200/yr) for a
claimed, maintained, rich district page + analytics. 77 districts × modest fee = real, if
small, recurring revenue. *This is the most honest "commercial later" in the set.*

**Validation first-move:** build **one** district page, then pitch 3–5 local municipalities /
tourism committees; ask if they'd pay $100/yr for it. This is a $0, one-week test with a
hard dollar answer.

---

### Idea C — **Diaspora Home Explorer**

**One-liner:** "Explore your home district — people, places, wildlife, and what's changed —
and plan your trip home."

**User & job:** the ~4M+ Nepali diaspora (and the large internal-migrant population) who want
to reconnect and plan visits home. High emotional intent, underserved, and *not* served by
global trekking apps.

**The agentic layer:** a "home district" copilot — *"show me my village, my district's
people, and what's new since I left."* Grounded in census + OSM + Wikivoyage + news.

**Data:** Census 2021 + OSM + Wikivoyage + (later) diaspora remittance/return-visit content.

**Stack:** same foundation · authentication (save "home district") · the copilot tuned for
personal/emotional queries.

**Clear path:**
| Phase | Build | Time |
|---|---|---|
| 1 | "Set home district" + demographic profile view | 3–4 days |
| 2 | Diaspora copilot (census + OSM + Wikivoyage grounding) | 4–5 days |
| 3 | Shareable "home district" cards (viral loop) | 2–3 days |
| **Total** | | **~2 weeks** |

**Monetization:** affiliate (flights, remittance partners, local stays); sponsored "return
home" content. Weaker direct revenue, strong organic reach.

**Validation:** post the "home district" card in one diaspora community (Reddit/FB group);
measure shares. Cost: one day.

---

### Idea D — **Wildlife Atlas** (ecotourism copilot)

**One-liner:** "What wildlife can I see in the Terai in March — and which lodge should I book
to see it?"

**User & job:** ecotourists (birdwatching, wildlife treks, safari) and the growing
sustainable-travel segment.

**The agentic layer:** grounded wildlife Q&A — species lists, seasonality, protected areas,
lodge proximity — from **GBIF (~1.98M records, verified)** + IUCN + WDPA.

**Data:** GBIF occurrences + IUCN threat status + WDPA protected areas + OSM lodges.

**Stack:** same foundation · GBIF precompute (district species summaries + seasonality) ·
IUCN API · deck.gl heatmaps.

**Clear path:**
| Phase | Build | Time |
|---|---|---|
| 1 | GBIF → district species summary tables + IUCN status | 3–4 days |
| 2 | Species heatmaps + protected-area overlays | 2–3 days |
| 3 | Wildlife copilot (seasonality + "where to see") | 3–4 days |
| **Total** | | **~2 weeks** |

**Monetization:** ecotourism lodge placement; conservation-org sponsorship. Low but
distinctive — the layer that makes the whole product feel *alive* and *authoritative*.

**Validation:** pitch to 3–5 ecotourism lodges/guides (e.g. Chitwan/Bardia) — do they want a
"wildlife discovery" listing?

---

### Idea E — **Nepal Earth Atlas** (the full 3-layer flagship)

**One-liner:** the umbrella — demographics + biodiversity + tourism for all 77 districts, with
the agentic copilot as the front door.

**User & job:** everyone (tourists, diaspora, researchers, institutions). This is the
**portfolio flagship** and the eventual home of ideas A–D as *tabs*.

**The agentic layer:** the full spatial copilot (six tools from doc 06) — the front door to
every other idea.

**Data:** all of the above, unified by the canonical gazetteer.

**Clear path:** this is the *end state*, not a first build. It is reached by building the
foundation (§4) then layering A → C → D → B, each adding a tab to the atlas.

---

## 3. Recommendation — build THIS first

**Build the shared foundation (§4) + Idea A (Trek Copilot), for the Annapurna region only.**

Why this order:

1. **Trekking is Nepal's signature product** — highest-intent tourists, clearest "wow."
2. **The agentic layer shines hardest here** — elevation profiles, teahouse spacing, and
   permit rules are *deterministic*, so the copilot is grounded and impressive from day one
   (no gimmick risk).
3. **It's the fastest path to a portfolio centerpiece** (~3 weeks) that also happens to
   validate the commercial wedge.
4. **It feeds Idea B directly** — the same trek/teahouse/stay data powers the District
   Showcase monetization later.

**The round-1 goal is not revenue.** It is: (a) one region, working end-to-end, (b) an eval
set proving the copilot is *grounded* (cited, no hallucinations), and (c) 10 real trekkers
who say "I'd use this." Revenue is round 2 (Idea B).

---

## 4. The shared foundation (build once, reuse everywhere)

Every idea rides on the same four pieces. Build these **first**, before any idea:

1. **Canonical gazetteer** — one join table for districts/places across CBS, ICIMOD, OSM
   (names are inconsistent: "Kaski" / "Kaski District" / "NP-4"). Everything keys off this.
2. **Spatial base** — PostGIS + Postgres, vector tiles (MVT), COG rasters (DEM), Redis cache.
3. **Agentic grounding layer** — deterministic spatial lookups (`what's at lat,lng?`),
   structured tool-calling, citation-aware LLM synthesis. This is the reusable moat.
4. **Provenance + refresh pipeline** — the dataset registry (source/license/date) from doc
   06, plus monthly OSM / quarterly census+biodiversity refreshes.

**Foundation build time: ~1 week.** After that, each idea is a 2–3 week increment, not a
from-scratch build.

---

## 5. Assumptions to watch (don't let them hide)

- **Assumption:** tourists prefer a conversational planner over a guide/blog. *(Test: Idea A
  validation.)*
- **Assumption:** municipalities will pay for a district page. *(Test: Idea B validation —
  the hardest-dollar test in this set.)*
- **Assumption:** the copilot can be kept grounded + < $0.01/session at free-traffic scale.
  *(Test: the eval set in Idea A phase 5.)*

Each assumption has a named, cheap test — run them before investing past round 1.
