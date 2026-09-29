---
name: feed-custom-label-planner
owner: adsledger
category: Product feeds
description: Designs custom_label_0–4 so Shopping and PMax can bid by margin, performance and season, with rules drawn only from data the store actually has.
version: v2
license: MIT
updated: 2026-09-02
recommended: false
security_checked: true
url: https://ecomdly.com/skills/adsledger/feed-custom-label-planner
raw: https://ecomdly.com/raw/adsledger/feed-custom-label-planner.md
install: npx @ecomdly/cli add adsledger/feed-custom-label-planner
---

# Feed custom label planner

Custom labels are the only way to tell Google that two products at the same price are not worth the same. This skill plans the five slots so each one maps to a bidding or budget decision.

## When to use
- Before restructuring Shopping campaigns or PMax listing groups.
- When every product sits in one "All products" group and ROAS hides big spread.

## Input
Per product: id, price, cost (or margin %), category, 90-day clicks, conversions, conversion value, stock level, and season or launch date if known. The store's target ROAS or margin goal.

## Default plan
- **custom_label_0 – margin band:** `m-high` (≥ 50%), `m-mid` (30–50%), `m-low` (< 30%). Thresholds come from the store's own margin distribution; use terciles if no goal is given.
- **custom_label_1 – performance:** `hero` (top 20% of conversion value), `sidekick` (converts, below ROAS target), `villain` (≥ 100 clicks, 0 conversions), `zombie` (< 10 impressions / 30 days).
- **custom_label_2 – price bucket:** 3–5 buckets cut at natural price points (e.g. <25, 25–60, 60–150, 150+).
- **custom_label_3 – season / lifecycle:** `new` (< 30 days), `seasonal-ss`, `seasonal-aw`, `evergreen`, `clearance`.
- **custom_label_4 – stock:** `low` (< 5 units), `ok`, reserved for promos.

## Rules
- Each label must drive one decision (separate asset/listing group, bid, or exclusion). A label nobody acts on is dropped.
- Max 1 000 unique values per label; keep values short, lowercase, stable.
- Performance labels need a minimum sample: under 30 clicks the product stays `unrated`.
- If cost data is missing, label_0 is left empty and flagged; never estimate margin.
- Output is a plan and a mapping file for review; the agent does not edit the feed or supplemental sheet itself.

## Output format
```
| label | values | rule | used for |
| custom_label_0 | m-high/m-mid/m-low | margin ≥50 / 30–50 / <30 | tROAS per group |
| custom_label_1 | hero/sidekick/villain/zombie/unrated | 90d value, clicks ≥30 | split zombies to own group |
Coverage: 2 310 products; label_0 on 1 980 (330 missing cost); 412 villains, 640 zombies.
```

## License
MIT
