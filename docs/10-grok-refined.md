Summary of the Journey
You started with a broad vision: Nepal Earth — an AI discovery engine for the long-tail of Nepal (preference-based matching across destinations, geospatial knowledge graph as moat, map + agent, later biodiversity and B2B tourism intelligence).
Analysis showed the vision is strong and timely, but too wide for a solo engineer. The risk of building infrastructure for months before learning whether travellers care was high.
You then proposed a sharp wedge: Annapurna-only trek finder with preference sliders, ranked results, evidence explanations, and explicit ML models from day one.
Further refinement (the latest input) improved it significantly:
	•	Treat Annapurna as the first vertical slice of Nepal Earth, not a separate product.
	•	Start with a transparent weighted scoring model (not fancy ML).
	•	Make difficulty explainable and evidence-based early.
	•	Treat incomplete teahouse data as the seed of a proprietary data moat.
	•	Attack distribution via high-intent SEO/content.
	•	Expand region-by-region and capability-by-capability only after real usage validates the core loop.
Clear conclusion: Build a complete, excellent product for one high-intent problem (“Which Annapurna trek is right for me?”). Architect it so every piece of data and ranking logic can later power the broader Nepal Earth vision.

Product Requirements Document
Product Name (MVP): Find Your Trek Working Title / Future Brand: Nepal Earth — Find Your Trek Version: 1.0 (Annapurna Wedge) Date: August 2026 Owner: Solo founder / engineer
1. Vision
Help independent travellers choose the right trek in Nepal with confidence, evidence, and clarity — starting with Annapurna.
Long-term: Become the geographic intelligence layer for discovering and planning experiences across Nepal (Nepal Earth).
2. Problem Statement
Travellers who want to trek in Nepal (especially Annapurna) face decision paralysis:
	•	Dozens of options (ABC, Circuit variants, Mardi Himal, Khopra Ridge, Poon Hill, etc.)
	•	Conflicting blog advice
	•	Hard to match personal constraints (days, fitness, altitude tolerance, crowds, scenery, culture, budget)
	•	Difficulty and crowding claims are rarely transparent or evidence-based
Existing tools are either generic itinerary generators, static maps, or agency sales pages. None deliver ranked, preference-matched, evidence-backed recommendations with an interactive map and agent that can answer follow-up questions grounded in data.
3. Goals (MVP – 4 weeks)
	•	Launch a working product that takes user preferences → returns 5 ranked Annapurna treks with match scores, transparent “why this matches”, and basic itinerary.
	•	Every factual claim must be traceable to a deterministic data source (no hallucinated numbers or teahouse names).
	•	Establish the core data model and ranking approach that can later expand to other regions and experience types.
	•	Validate real demand: do strangers actually use it and find the recommendations useful?
4. Target Users (MVP)
Primary: Independent international trekkers (and experienced domestic travellers) researching Annapurna treks who have already decided on Nepal but not yet chosen a specific route.
Secondary (later): People exploring “which trek in Nepal” more broadly.
5. Scope
In scope (Phase 1 – Annapurna MVP)
	•	Preference input via sliders / controls
	•	Ranking of 5–10 curated Annapurna treks
	•	Match score + dimension breakdown
	•	Evidence-backed “Why this trek” explanations
	•	Interactive map with trail, elevation, key features
	•	Basic 3–5 day sample itinerary
	•	Agent that answers grounded questions about the recommended trek
	•	GPX export and elevation profile
	•	Transparent scoring (inspectable weights)
Out of scope for MVP
	•	All of Nepal or other regions
	•	Bookings, payments, guide marketplace
	•	Fancy learning-to-rank models (start with weighted scoring)
	•	Precise real-time crowding numbers
	•	Biodiversity / wildlife mode
	•	Offline mobile app
	•	User accounts / saved trips (nice-to-have if time allows)
6. Core User Experience
Primary flow
	1	Land on “Find Your Trek”
	2	Adjust preferences:
	◦	Difficulty / fitness
	◦	Number of days
	◦	Mountain scenery importance
	◦	Culture / villages
	◦	Nature / forest / wildlife
	◦	Crowd preference
	◦	Altitude tolerance
	◦	(Optional) Budget range
	3	Click “Find my trek”
	4	See ranked cards (top 5) with:
	◦	Match %
	◦	Key dimension scores
	◦	Short “Why this matches you” paragraph (evidence-linked)
	5	Click a trek → detailed view:
	◦	Full map (trail + elevation + layers)
	◦	Elevation profile
	◦	Day-by-day outline
	◦	Evidence for scores
	◦	Agent chat: “Where is the steepest section?”, “What if I remove one day?”, “Quieter alternatives?”
	6	Export GPX or save itinerary
7. Ranking & Scoring Approach (MVP)
Start with transparent weighted scoring (v0) — not black-box ML.
Example structure (weights tunable):
match_score = 
  0.25 × scenery_fit +
  0.20 × difficulty_fit +
  0.15 × duration_fit +
  0.15 × crowd_fit +
  0.10 × culture_fit +
  0.10 × nature_fit +
  0.05 × accessibility_fit
Every component score must be inspectable and explainable.
Difficulty model (build early, keep deterministic & explainable) Derived from:
	•	Total elevation gain
	•	Maximum altitude
	•	Average & steepest gradient
	•	Daily distance
	•	Consecutive hard days
	•	Altitude gain rate
Show the breakdown: “This is moderate because…”
Crowding Call it “Crowd estimate” initially. Use available signals (popularity, accommodation density, season, public stats, OSM activity). Be transparent about uncertainty. Improve later with user reports.
Progression path for ranking Rules / weighted scoring → learning-to-rank (once you have click/save/complete data) → personalised ranking.
8. Data Architecture Principles
	•	Geospatial foundation first (trail segments, elevation, terrain, settlements, protected areas).
	•	Trek-level feature vector built on top of the geospatial layer.
	•	Every claim the agent makes must resolve to a source (database field, computed feature, or curated fact with citation).
	•	Teahouse / accommodation data starts incomplete → treat as opportunity:
	◦	OSM baseline
	◦	Manual verification for the initial 5–10 treks
	◦	Traveller submission / reporting loop later
	◦	This becomes proprietary data over time.
9. Technical Notes (High-level)
	•	Backend: PostGIS (or equivalent) for spatial features + feature store for trek vectors.
	•	Ranking: deterministic scoring engine first.
	•	Frontend: clean map (MapLibre or similar) + preference UI + results cards.
	•	Agent: LLM with strict grounding / tool use / citations only. Mandatory eval set of ~20 questions with known correct numbers.
	•	Export: GPX + elevation profile.
10. Phased Roadmap
Phase 1 (Weeks 1–4) – Annapurna MVP “Find Your Trek” for Annapurna. Ship and validate.
Phase 2 Deepen Annapurna data (fuller trail network, better accommodation, viewpoints, seasonality, crowd signals, user contributions).
Phase 3 Add second region (e.g. Langtang). Product becomes “Find Your Nepal Trek”. Test whether ranking generalises.
Phase 4 Biodiversity / ecosystem layer (“forests + wildlife, not just mountain views”).
Phase 5+ Expand experience types (hiking, wildlife, cultural journeys, road trips) → full Nepal Earth discovery engine + eventual B2B tourism intelligence.
11. Distribution Strategy (Critical)
Do not rely on “build it and they will come”.
Primary channel: high-intent SEO + content
	•	“Annapurna trek finder”
	•	“Which Annapurna trek is right for me”
	•	“Mardi Himal vs Khopra Ridge”
	•	“Best 5-day Annapurna trek”
	•	“Quiet Annapurna treks”
	•	“Annapurna trek difficulty comparison”
Each page embeds or links to the interactive tool.
Secondary: trekker communities, Reddit, Facebook groups, partnerships with content creators, soft outreach to NTB / community networks later.
12. Success Metrics (First 30–60 days after launch)
	•	Preference form → results completion rate
	•	% of users who mark recommendations as “new / useful / better than blogs”
	•	Click-through to detailed trek view
	•	GPX / itinerary export rate
	•	Return visits
	•	Qualitative feedback on explanation quality and score trustworthiness
	•	Number of treks saved or shared
North-star signal: Strangers say some version of “This is exactly how I wanted to choose a trek.”
13. Key Risks & Mitigations
Risk
Mitigation
Incomplete teahouse data
Curate rigorously for initial treks; open contribution loop early
LLM hallucination
Strict grounding + mandatory eval set; never invent numbers or names
Distribution
High-intent content/SEO from day one
Scope creep
Ruthlessly stay Annapurna-only until validation
Weak differentiation
Transparent, evidence-backed scores + interactive map + grounded agent
14. Definition of Done (MVP)
	•	User can set preferences and receive ranked Annapurna treks with match scores.
	•	Every score and claim has visible evidence.
	•	Map + elevation + basic itinerary work.
	•	Agent answers grounded questions without fabricating facts.
	•	Eval set passes (no hallucinated numbers).
	•	Publicly accessible and usable by strangers.
	•	Architecture supports future expansion to more regions and layers.

Final Direction
Build Find Your Trek (Annapurna) as a complete, polished product for one clear problem. Architect every data layer, ranking component, and map interaction so it becomes the first vertical of Nepal Earth.
Do not build a smaller version of the big vision. Build one excellent product that proves people want evidence-based discovery — then earn the right to expand.
This is the strongest solo-founder path that preserves the high-upside long-term vision.
