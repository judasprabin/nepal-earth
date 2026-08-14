# 02 — How to Find a Moat (and whether one exists here)

This is the document the blueprint was missing. It answers three questions in order:

1. What is a moat, and how do you *find* one (the general method)?
2. Does **Nepal Earth** have one as currently framed? (No.)
3. What would a real moat look like here — and what's the honest fallback if none exists?

---

## 1. What a moat actually is

A moat is a structural, durable advantage that lets you earn returns competitors cannot
copy away. It is **not** a feature, not a "better UX," and not first-mover status. The two
reference frameworks worth internalizing:

**Buffett / Morningstar taxonomy (five classic moats):**
1. **Intangible assets** — patents, brands, regulatory licenses.
2. **Switching costs** — expensive/high-friction to leave you.
3. **Network effects** — more users make the product more valuable to every user.
4. **Cost advantage** — structural (scale, process, location) not tactical.
5. **Efficient scale** — a niche big enough for one player but not two.

**Hamilton Helmer's "7 Powers"** (more precise for tech):
1. **Scale economies** — declining unit cost with volume.
2. **Network economies** — value rises with users.
3. **Counter-positioning** — a new model the incumbent *won't* copy because it would
   cannibalize their economics.
4. **Switching costs** — customer lock-in.
5. **Branding** — trust/preference that reduces CAC and supports pricing.
6. **Cornered resource** — exclusive access to a scarce input (data, talent, channel).
7. **Process power** — proprietary methods/culture that are hard to replicate.

### The general method for finding a moat

A moat is *found*, not invented in a slide. The method is:

1. **Name the incumbents** who already serve your customer (not the data sources — the
   products). *If no incumbent exists, you either found whitespace or — more likely — found
   a market that doesn't exist yet.*
2. **Ask what they cannot copy** without destroying their own economics. That's
   counter-positioning. Everything else (features, price, UI) they can copy in a sprint.
3. **Look for a cornered resource** you could actually own: exclusive data, an exclusive
   channel, an exclusive distribution relationship, a regulatory right, or a compounding
   dataset that gets *better* every time someone uses it.
4. **Test for the flywheel.** A real moat compounds: usage → data → better product → more
   usage. A fake moat is static: it looks impressive on day one and decays after.
5. **Be honest about the null case.** Most ideas have **no moat**, and the correct strategic
   move is not to pretend — it's to pick a different game (build for portfolio/skill, or pick
   a niche where one of the five powers is obtainable).

---

## 2. Applying the test to Nepal Earth (as framed)

Run each power against the blueprint's actual claims:

| Power | Nepal Earth's claim | Verdict |
|---|---|---|
| Intangible assets | None (no patent, brand, or license) | ❌ |
| Switching costs | None — users can reopen GFW tomorrow | ❌ |
| Network effects | None — a viewer has no cross-user value | ❌ |
| Cost advantage | None — worse, the *data* is free for everyone | ❌ |
| Efficient scale | No — the niche is served by free tools | ❌ |
| Scale economies | No volume-linked cost decline of note | ❌ |
| Counter-positioning | No — nothing an incumbent would refuse to copy | ❌ |
| Cornered resource | **Claimed implicitly**: "normalized spatial model + knowledge graph" | ❌ — it's *public* data + standard raster ops; not scarce |
| Branding | None | ❌ |
| Process power | None (yet) | ❌ |

**The core error in the blueprint:** it treats "fusing public data into a coherent model" as
a defensible asset. But a moat requires **scarcity**. Public data fused with standard
operations is not scarce — it is *replicable by definition*. ICIMOD, SERVIR, or any competent
GIS shop could rebuild the core in weeks. The "knowledge graph" (§14 of the blueprint) is a
relational table the blueprint itself admits doesn't even need Neo4j.

> **Rule: if your "moat" can be rebuilt by reading your own public blueprint, it is not a moat.**

---

## 3. Where a real moat *could* exist here

There are exactly **three** places a durable advantage could theoretically live in this
domain. Only one is realistic for a solo operator.

### 3.1 Proprietary derived data (the cornered-resource path) — **most realistic**

The raw layers are public. But **validated, ground-truthed risk models are not**.

- A **landslide-susceptibility model** or **flood-exposure model** for Nepal, trained on
  field-verified event inventories, rainfall, slope, land cover, and road proximity, and
  *validated against observed events*, is a **cornered resource**: it cannot be copied from
  the public layers because the *ground truth* (the labelled event data) is the scarce part,
  and it gets *better* the more events you ingest.
- This is the flywheel: each flood/landslide event you ingest improves the model, which
  improves the product, which attracts more data partners.
- **Caveat:** this is a *different product* (a risk-intelligence API), not a map explorer.
  It is what the blueprint's "landslide susceptibility" and "flood exposure" ML rows are
  really pointing at, and it is the only wedge with a genuine moat.

### 3.2 Distribution lock-in (the switching-cost / channel path) — **less likely**

If you became the *default embedded map* inside a specific workflow — e.g. the tool every
Nepali trekking agency uses to generate permits, or the dashboard every hydropower
consultant attaches to bids — you'd have switching costs. But that requires winning an
embedded-distribution deal first, which is a *sales* problem, not a build problem, and not
where a solo engineer starts.

### 3.3 Brand as "the Nepal data authority" — **possible but slow**

Becoming the trusted, cited source for Nepal geospatial change ("what Global Forest Watch is
to forests, Nepal Earth is to Nepal") is a real moat — **brand + cornered provenance** — but
it takes years of public, reproducible, high-quality output and an audience. It's a
*byproduct* of doing the portfolio work well, not a plan you can execute in a sprint.

---

## 4. The honest fallback: the moat is the *skill*, not the product

For a backend/ML engineer, the strategically correct framing is:

> **The durable asset is not "Nepal Earth the product." It is "Nepal Earth the demonstrable
> capability"** — proof that you can build a production-grade geospatial data platform
> (PostGIS, raster pipelines, 3D viz, ML) end to end.

That is a cornered resource of a different kind: it is *you*. It can't be copied, it
compounds with every project, and it converts directly into differentiated positioning in
the EO/geo-ML hiring and consulting markets. The product is the *evidence*; the skill is
the moat.

This is not a consolation framing — it is the correct conclusion of the moat test. When a
product has no product-moat available, the rational move is to treat it as an investment in
a *personal* moat, which is exactly what the blueprint's "portfolio narrative" section was
reaching for.

---

## 5. The one defensible business path (if you pursue one at all)

If the goal shifts from portfolio to *business*, there is exactly one path with a real moat,
and it is a pivot:

**A validated Nepal landslide/flood exposure risk product.**

- **Customer:** insurers, reinsurers, INGOs (World Bank, UNDP, IFRC), hydropower developers,
  road planners.
- **The moat:** a ground-truthed, validated risk surface that compounds with every observed
  event. Nobody has a *current, validated, Nepal-wide* version of this as a commercial API.
- **The catch:** this is an ML + data-collection problem with a benchmark-accuracy bar, not a
  front-end problem. It needs its own `Validate First` cycle (see
  `docs/05-validation-plan.md` §4) before any build investment.

*Everything else* — the consumer map, the "digital twin explorer," the time machine — is
portfolio material, not business material.

---

## 6. Summary

| Question | Answer |
|---|---|
| Does Nepal Earth have a moat as framed? | **No.** Aggregation of public data is replicable, not scarce. |
| Could it ever have one? | **Yes, but only via the risk-model pivot** (cornered resource: ground-truthed data). |
| What's the moat *today*? | **The skill and portfolio value** — the personal moat, which is real and durable. |
| Strategic conclusion | Build for portfolio + skill; if you want a business, pivot to the risk model and validate first. |
