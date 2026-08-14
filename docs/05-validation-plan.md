# 05 — Validation Plan (the cheapest way to test the idea)

The rule for a solo operator: **validate before you build, and spend the minimum that can
change a decision.** The blueprint skipped validation entirely. This fills that gap with two
tracks — one for the portfolio framing (what to do now), one for the business pivot (what to
do if you ever want revenue).

---

## 1. The principle

> The cheapest validation is one that can produce a **decision** — "build this" or "don't" —
> not a feature list. If a validation exercise can't kill the idea, it isn't a validation.

For Nepal Earth, the decision has already been made on the evidence available:
**`Portfolio Only`.** So the "validation" is not "should I build" (yes — for the portfolio),
it's "is there a business here" (almost certainly not — but here's how to confirm for a
weekend, not a quarter).

---

## 2. Track A — confirm "portfolio, not business" (a weekend, ~$0)

This is the test to run *now*, before writing more code:

1. **Build a static prototype of the time machine for ONE district** (Kaski or Solukhumbu),
   using public GFW/ICIMOD tiles. No backend, no PostGIS — a static page + a slider.
2. **Show it to 10 people across 3 segments:**
   - 3 trekking agencies / guides,
   - 3 conservation NGO people (WWF Nepal, NTNC, WCN),
   - 1–2 hydropower / planning consultants, 1–2 "interested friends."
3. **Ask exactly one question, and get a number:** *"What would you pay for this, and what
   would you actually do with it?"*

**Decision rule:**
- If **nobody names a use + a number** → confirmed: portfolio, not business. (Most likely.)
- If **the disaster/insurance segment** names a specific workflow (e.g. "we'd use this to
  screen road alignments for landslide risk") → you have the seed of the *other* product
  (Track B).

**Cost:** a weekend and some WhatsApp messages. This is the highest-leverage step in this
entire repo.

---

## 3. Track A follow-up — quantify willingness to pay (only if Track A signals)

If any segment bites, do not build the product. Build a **paid pilot offer** instead:

- A one-page "Nepal land-cover change report" for a single district, priced (say) $200–500,
  delivered as a PDF + one-hour call.
- Sell it to the 2–3 warmest leads from Track A.

**Decision rule:** if you cannot sell *one* report, you cannot sell a product. A pilot that
sells is the only honest market evidence; a pilot that doesn't is the cheapest possible
learning.

---

## 4. Track B — validate the risk-model pivot (if you ever want a business)

This is the only path with a real moat (see `docs/02-moat-and-defensibility.md` §5), and it
has its own gate. Do **not** build the platform for this. Validate in this order:

1. **Data gate (2 weeks):** can you assemble a ground-truthed inventory of landslide/flood
   events for Nepal (government disaster records, news archives, ICIMOD inventories, field
   reports)? **This is the scarce asset** — if you can't get ≥ a few hundred labelled events,
   stop. There is no product without it.
2. **Accuracy gate (3–4 weeks):** train a baseline model (slope + rainfall + land cover +
   distance-to-road/fault) and validate against a held-out set of events. Publish the AUC /
   precision-recall. If it's not *materially better* than a naive slope-only baseline, stop.
3. **Customer gate (2 weeks):** interview 5–10 buyers (reinsurers, INGO risk teams, hydropower
   developers). Confirm they (a) have budget, (b) have this problem *today*, (c) would pay for
   an API/scorecard.
4. **Only after 1–3 pass:** build the product.

**The bar is a benchmark, not a demo.** A "risk score" that isn't validated against observed
events is worse than nothing in a disaster-prone country — it's a liability (see
`docs/01-critical-analysis.md` §8).

---

## 5. What NOT to do

- ❌ Don't build the full platform to "test the market" — the platform is the expensive part
  and proves nothing about demand.
- ❌ Don't run a "would you use this?" survey with friends — politeness masquerades as demand.
  Always ask for **money or a named workflow**.
- ❌ Don't validate the *map* — the map is commodity. Validate the *insight* (the change
  metric, or the risk score) that only you can produce.
- ❌ Don't add ML before the spatial core is useful. The blueprint's own §13 rule is correct:
  ship the platform first, add one justified model later.

---

## 6. Summary

| Question | Cheapest test | Cost | Go / No-go signal |
|---|---|---|---|
| Is this a business? | Static 1-district prototype + 10 interviews | ~$0, 1 weekend | Named workflow + a number |
| Will anyone pay? | Sell one paid pilot report | ~$0, 2 weeks | One sale = proceed; none = stop |
| Is the risk-model wedge real? | Data gate → accuracy gate → customer gate | ~6–8 weeks part-time | Held-out AUC + buyer budget confirmed |

For the portfolio framing, **skip validation and build** — the portfolio value is already
established. For any business claim, run Track A first.
