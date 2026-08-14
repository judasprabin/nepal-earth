# 04 — Suggestions & Roadmap

Concrete, ranked improvements. These assume the `Portfolio Only` verdict — i.e. build it, but
build it *well* and for the right reason. A separate note covers the one business pivot.

---

## 1. Positioning fixes (highest leverage, zero code)

1. **Stop calling it a "product" / "startup."** Call it what it is: *a production-grade
   geospatial data platform, built publicly.* The framing "digital twin explorer" invites the
   comparison you lose (Google Earth). The framing "data platform" invites the comparison you
   win (engineer credibility).
2. **Lead every demo with one derived insight, not the layer catalog.** Open with
   *"Kaski district lost X ha of forest 2000–2022 — here's where and here's the class
   transition."* That single change moves you from "map viewer" to "analysis product" in a
   recruiter's mind.
3. **Make provenance the headline feature.** The "data provenance drawer" is the rarest and
   most impressive thing in the blueprint. Put it front and center — it is the differentiator
   against every free tool (none of them show you *why* to trust the map).

---

## 2. Scope cuts (make it finishable)

The blueprint is realistic but over-scoped for a solo sprint. Cut, in this order:

| Cut | Why |
|---|---|
| ❌ ML models (phase 6) | Gimmick risk until the platform proves itself |
| ❌ Knowledge graph (Neo4j/semantic) | Relational tables suffice; revisit only if traversal becomes a feature |
| ❌ Kubernetes / Terraform | Infrastructure theater at Nepal scale; $50–100/mo serverless suffices |
| ❌ Free-form AI agent | Structured spatial filters first (the blueprint already says this) |
| ⚠️ Real-time weather/gauge feeds | Explicitly non-essential; add via API later |

**Keep:** 3D/hillshade terrain, rivers, NLCMS 2000–2022 slider, forest metrics, protected
areas, OSM context, click-to-inspect, elevation profile, provenance, compare mode.

---

## 3. Technical suggestions

1. **Prove the data pipeline first (Phase 0, but rigorous).** The single highest-value
   artifact is a reproducible ingestion pipeline with a dataset registry (source → license →
   checksum → CRS → refresh). This is what separates you from "someone who made a map."
2. **COGs + vector tiles from day one.** Never send raw geometries or full rasters to the
   browser. This is the difference between a demo that stutters and one that feels production.
3. **Keep render path and analytics path separate** (the blueprint is right). Cache feature
   profiles in Redis; run heavy raster algebra asynchronously.
4. **Use PostGIS `GiST`/`SP-GiST` indexes** and precompute zonal stats/proximity rather than
   computing on click.
5. **Pin every dependency and version your processing code** — "reproducible" is a portfolio
   word, but only if it's actually true.

---

## 4. Portfolio-narrative suggestions

The blueprint's closing narrative is strong. Tighten it to three beats:

1. **"I fused government, institutional, and open geodata** (National Geoportal, ICIMOD, NTNC,
   OSM) into one normalized spatial model."
2. **"I turned raster/vector data into queryable spatial intelligence** — land-cover
   transitions, forest structure, terrain profiles — with full provenance."
3. **"I exposed it through an interactive time-aware 3D web application."**

That maps one-to-one to the skills a geo/EO ML role tests: data engineering → spatial
analytics → front-end/visualization.

---

## 5. Roadmap (refined from the blueprint's §17)

| Phase | Focus | Output | Effort |
|---|---|---|---|
| 0 | Data proof: license checks + sample layers + registry | Reproducible ingestion script | 2–3 days |
| 1 | PostGIS + FastAPI + React + one map | Working map with layers + click-to-inspect | 1–2 weeks |
| 2 | NLCMS time slider + change panel | "Landscape Time Machine" (the centerpiece) | 1–2 weeks |
| 3 | Forest metrics + terrain/elevation profile | Forest profile + 3D terrain | 1 week |
| 4 | Provenance + compare mode + spatial query | The differentiators | 1 week |
| 5 | *(optional)* one ML model on top | Validated model + uncertainty viz | 2+ weeks |

**Total: ~4–8 weeks** for a strong individual. Ship after phase 4; ML only if the platform is
already demonstrably useful.

---

## 6. If you ever pivot to a business (the risk-model wedge)

The only defensible business path is the **validated Nepal landslide/flood exposure product**
(see `docs/02-moat-and-defensibility.md` §5). If you go there, the roadmap inverts:

1. **First:** acquire/build a ground-truthed event inventory (the scarce asset).
2. **Then:** train + validate a risk model against held-out events (benchmark accuracy bar,
   not a demo).
3. **Only then:** wrap it in a map/API and sell to insurers/INGOs.

That is a *different* project with a `Validate First` gate before build — see
`docs/05-validation-plan.md` §4.
