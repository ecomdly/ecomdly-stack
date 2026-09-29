---
name: feed-custom-label-planner
owner: adsledger
category: Product feeds
description: Plans custom_label_0-4 from your own margins, VAT-correct break-even ROAS and sample-safe performance tiers, and outputs a mapping file for a supplemental feed plus the Shopping or PMax structure each label drives. For PPC specialists.
version: v3
license: MIT
updated: 2026-09-29
recommended: false
security_checked: true
url: https://ecomdly.com/skills/adsledger/feed-custom-label-planner
raw: https://ecomdly.com/raw/adsledger/feed-custom-label-planner.md
install: npx @ecomdly/cli add adsledger/feed-custom-label-planner
---

# Feed custom label planner

Produces a plan for the five custom label slots (`custom_label_0` to `custom_label_4`) and a mapping file (id → label values) ready for a supplemental data source, each label tied to a bidding, budget or exclusion decision. Custom labels are the only feed attributes whose meaning you define, so they are how you tell Google that two products at the same price are not worth the same. The naive plan copies a blog template ("hero / zombie") with fixed thresholds, uses gross prices with VAT to compute margin, and relabels products daily; the result is labels nobody acts on and products that flap between groups so bidding never learns. This skill derives thresholds from the store's own economics, requires a sample before judging performance, and makes every label map to a campaign structure.

## When to use

- Before restructuring Standard Shopping product groups or Performance Max listing groups.
- When all products sit in one "All products" group and account ROAS hides a wide spread by product.
- When margin varies a lot across the catalog and one tROAS target over- or under-bids big parts of it.
- Before a season or sale, to isolate seasonal, clearance or new products.

## When not to use

- Fewer than roughly 50 products or very low conversion volume: splitting starves every group of data. Recommend one group and better feed data instead.
- You need to compute the break-even or target ROAS itself: `break-even-roas-calculator` first, then this skill.
- You want to change titles or fix disapprovals: `product-feed-optimizer`, `merchant-center-disapproval-fixer`.
- Zboží.cz or Sklik labels: Zboží's feed has its own `CUSTOM_LABEL_0/1/3` with different semantics; handle in `comparison-feed-mapper`.

## Inputs

Required, per product (variant-level `id` as in Merchant Center):

- `id`, `item_group_id`, `price` and `sale_price` (as in feed, VAT included in EU), product category (`product_type` level 1–2), brand.
- Unit cost excluding VAT (purchase price plus any per-unit landed cost the store tracks).
- Google Ads performance, per item ID: impressions, clicks, cost, conversions, conversion value, for the last 90 days (Google Ads > Products report, or a Shopping/PMax item-level report). Note whether conversion value includes VAT and shipping; stores configure this differently.
- Current campaign structure: which campaigns/asset groups exist, their bidding strategy and targets.

Required from the owner:

- Target: break-even ROAS or target ROAS / POAS, and whether it is on revenue with or without VAT.

Optional:

- Stock quantity per item, launch date or first-available date, season tags, clearance list, return rate by category (`returns-reason-analyzer`).
- Existing custom label values (to avoid breaking live product groups).

Missing cost data: leave the margin label empty for those items, report coverage, never estimate margin from category averages unless the owner explicitly supplies the averages and accepts the label as "estimated".

## Best practices

### Platform facts

1. **Limits.** Five labels, `custom_label_0` to `custom_label_4`, one value each per product. Each value 1–100 characters. Up to 1,000 unique values per label, account-wide. Values are not case sensitive and not shown to customers. ASCII is recommended.
2. **Structure limits.** Standard Shopping: product groups can be subdivided up to 7 levels, up to 5,000 product groups per ad group. Performance Max: up to 1,000 listing groups per asset group; Google notes that very large numbers of listing groups are not best practice and recommends grouping with custom labels, then targeting the labels.
3. **Product groups are not self-updating.** Google: when you change attributes in the data source, the changes are not automatically applied to existing product group subdivisions; update the groups. A new label value appears under "Everything else" until someone adds it. Plan the values before launch and keep them stable.
4. **Delivery.** Add labels through a supplemental data source (for example a Google Sheet keyed by `id`) or feed rules, not by editing the e-shop's primary export. This keeps them independent of platform exports and easy to roll back. Check that no other source or rule already writes the same label.

### Design principles

5. **One label, one decision.** Each slot must answer a question that changes a bid, budget, target, or exclusion. If nobody will build a group or change a target on it, drop it. Write the decision next to the label in the plan.
6. **Stable slots for structure, volatile slots for tactics.** Put slow-changing facts (margin band, price bucket) in slots used for campaign or asset-group splits; put fast-changing states (stock, performance tier) in slots used for exclusions or bid adjustments. Moving a product between campaigns resets nothing in the product, but splits its data across groups; do that rarely.
7. **Use the store's economics, not generic thresholds.**
   - Net price: `price_net = price_gross / (1 + VAT rate)`. Use `sale_price` when active.
   - Margin: `margin = (price_net − unit_cost) / price_net`.
   - Break-even ROAS on net revenue: `BE_ROAS = 1 / margin`. A product at 25 % margin needs ROAS 4.0 to break even before other variable costs; at 50 % it needs 2.0.
   - Band products by the ROAS they need, which is what bidding reacts to: bands whose break-even ROAS differ enough to justify separate targets. If the owner gives no rule, cut at terciles of margin and report the cut points.
   - If conversion value in Google Ads includes VAT, compare it against `BE_ROAS × (1 + VAT)` or convert; mixing gross and net is the most common error in these plans.
8. **Performance labels need a sample.** A product with 12 clicks and zero conversions is not a loser; at a 2 % conversion rate, zero conversions in 12 clicks happens more often than not (0.98^12 ≈ 0.78). Rule: below a minimum number of clicks, label `unrated`. Derive the minimum from the account conversion rate: `min_clicks ≈ 3 / account_CVR` (about the clicks at which you would expect three conversions). At 2 % CVR that is 150 clicks. State the number used.
9. **Performance tiers relative to target, not to each other.** With enough clicks: `item_ROAS = conv_value / cost`. `top` if item ROAS ≥ target and conversion value in the top share the owner chooses (for example, items together making 50 % of value); `on_target` if ROAS ≥ target; `below_target` if ROAS < break-even; `no_conv` if clicks ≥ min_clicks and conversions = 0. `low_visibility` if impressions in the window are below a threshold the owner accepts (report the chosen value). Avoid cute names in client accounts; plain names survive handovers.
10. **Hysteresis.** Relabel performance on a fixed cadence (every 2–4 weeks), and move a product to a new tier only if it qualifies in two consecutive runs, or crosses the threshold by a margin (for example 10 %). This prevents weekly flapping.
11. **Seasonality and lifecycle.** `new` for products first available fewer than N days ago (the store's choice, commonly 30–60) so they get their own budget instead of competing with proven products; `seasonal_<season>` so budgets can be raised in season and cut after; `clearance` for items the owner wants sold at lower target ROAS (`clearance-markdown-planner`).
12. **Stock labels** only if the feed updates at least daily; otherwise the label lags reality. Typical use: exclude or lower bids on items with very low stock that would sell out from ad traffic alone.
13. **Price buckets** help when click cost is similar across a category but order value is not: cheap items cannot pay for the same CPC. Cut at natural price points in the store's distribution (quantiles), 3–5 buckets.

## Process

1. **Audit current state.** Existing labels, their values and usage in campaigns; current structure; coverage of cost data; share of items with any clicks in 90 days.
2. **Collect the decisions** the owner or PPC manager actually wants to make (for example "different tROAS by margin", "own budget for new products", "exclude items that never convert"). Map each decision to one slot. Maximum five; fewer is fine.
3. **Compute economics** per item: net price, margin, break-even ROAS; report distribution (min, terciles, max) and cut points.
4. **Compute performance** per item over 90 days; apply min-clicks rule; assign tiers against the target.
5. **Assign lifecycle, stock and price-bucket values** as planned.
6. **Check limits:** unique values per label ≤ 1,000 (should be single digits), value length ≤ 100, one value per item, lowercase ASCII, no spaces (use `_` or `-`).
7. **Simulate the structure:** count items, clicks, conversions and value per planned group. Any group with too little conversion volume for its bidding strategy should be merged; state the threshold used and where it comes from (the store's own history or the bid strategy's guidance in Google Ads).
8. **Write the plan and mapping file.** Include a change log versus existing labels and which product groups must be updated.

## Pitfalls and edge cases

- **Relabelling live groups.** Renaming a value that a product group targets moves those products into "Everything else". Keep old values until groups are rebuilt.
- **Variant-level vs parent-level.** Labels are per `id`. Compute margin per variant (sizes can have different costs); compute performance per `item_group_id` when variants split thin data, then apply the parent's tier to all variants, and say so.
- **Returns.** High-return categories overstate conversion value. If return data exists, discount value by category return rate before tiering.
- **Sale periods.** Margin during a sale is lower; compute with `sale_price` for items on sale and re-run when the sale ends.
- **PMax shares data across channels.** Item-level cost in PMax reports can include non-Shopping inventory; note the source of cost numbers.
- **Brand terms.** Items that sell mostly on brand searches look like top performers; that value might come anyway. Mention it when a top tier is dominated by the store's own brand.
- **Account-wide limit.** 1,000 unique values per label is account-wide; per-product values like SKU or exact price in a label will hit it.

## Rules

- Never estimate cost or margin. Missing cost → margin label empty and counted in coverage.
- Every label in the plan names the decision it drives; labels without a decision are removed from the plan.
- Performance tiers require the stated minimum sample; below it the value is `unrated`.
- State every threshold and its source (owner input, store distribution, or default chosen here).
- Read-only. The agent outputs a plan and a mapping file; it does not edit the feed, supplemental source, Merchant Center or campaigns.
- Changes to live product groups or campaign targets are listed for human sign-off, with the date the plan was computed.

## Output format

```text
Custom label plan: <store>, computed <YYYY-MM-DD>, data window <YYYY-MM-DD>..<YYYY-MM-DD>
Economics basis: VAT <rate>, conversion value <incl/excl> VAT, target ROAS <x> (<source>)

| slot | name | values | rule | decision it drives | refresh |

Thresholds used
- margin cut points: <a %>, <b %> (<source>)
- min clicks for rating: <n> (= 3 / CVR <x %>)
- other: <...>

Coverage
| slot | items labelled | items empty | reason |

Simulated structure
| group | items | 90d clicks | 90d conversions | 90d value | proposed target/action |

Changes needing sign-off
- <product group / asset group change>

Mapping file: id,custom_label_0,custom_label_1,custom_label_2,custom_label_3,custom_label_4
```

## Worked example

Example data. CZ store, 2,310 products, VAT 21 %, conversion value in Google Ads includes VAT, owner target: break-even plus 20 % buffer. Account CVR 1.8 %, so min clicks = 3 / 0.018 ≈ 167.

Margin example: jacket at 3,630 Kč gross → net 3,000 Kč; cost 1,800 Kč; margin = 1,200 / 3,000 = 40 %; BE_ROAS (net) = 2.5; on gross conversion value 2.5 × 1.21 ≈ 3.03; with 20 % buffer target ≈ 3.6 (gross).

```text
Custom label plan: example-outdoor.cz, computed 2026-09-29, data window 2026-07-01..2026-09-28
Economics basis: VAT 21 %, conversion value incl VAT, target = BE ROAS x 1.2 (owner)

| slot | name | values | rule | decision it drives | refresh |
| custom_label_0 | margin band | m_high / m_mid / m_low | margin >= 45 % / 30-45 % / < 30 % | separate PMax asset groups with tROAS 2.7 / 3.9 / 5.4 (gross) | monthly |
| custom_label_1 | performance | top / on_target / below_target / no_conv / unrated | clicks >= 167; ROAS vs target | exclude no_conv from main campaign, test in low-budget campaign | every 2 weeks, 2-run hysteresis |
| custom_label_2 | price bucket | p1 / p2 / p3 / p4 | < 500 / 500-1,500 / 1,500-4,000 / >= 4,000 Kc gross (store quartiles) | lower target for p4, exclude p1 from Search-heavy campaign | quarterly |
| custom_label_3 | lifecycle | new / seasonal_aw / clearance / evergreen | first available < 45 days / owner tag / owner list | own budget for new; clearance at BE ROAS | weekly |
| custom_label_4 | (unused) | | | no decision proposed | |

Thresholds used
- margin cut points: 30 %, 45 % (store terciles 29.6 %, 45.2 %, rounded)
- min clicks for rating: 167 (= 3 / CVR 1.8 %)
- lifecycle new: 45 days (owner)

Coverage
| slot | items labelled | items empty | reason |
| custom_label_0 | 1,980 | 330 | no cost in ERP export |
| custom_label_1 | 2,310 | 0 | 2,150 unrated (under 167 clicks); 160 rated |

Simulated structure
| group | items | 90d clicks | 90d conversions | 90d value | proposed target/action |
| m_high | 690 | 21,400 | 402 | 1,180,000 Kc | tROAS 2.7 |
| m_mid | 660 | 18,900 | 331 | 1,010,000 Kc | tROAS 3.9 |
| m_low | 630 | 9,800 | 172 | 640,000 Kc | tROAS 5.4 |
| no cost | 330 | 3,100 | 49 | 150,000 Kc | stays in m_mid until cost supplied |

Changes needing sign-off
- split PMax "All products" asset group into 3 listing-group-based asset groups by custom_label_0
- add exclusion of custom_label_1 = no_conv (18 items, 3,400 clicks, 17,000 Kc 90d cost, 0 conversions)
```

Target derivation for the bands (rounded up to one decimal): m_high uses the band's median margin 55 % → BE net 1.82 → gross 2.2 → ×1.2 = 2.64 ≈ 2.7; m_mid median 38 % → 2.63 → 3.18 → 3.82 ≈ 3.9; m_low median 27 % → 3.70 → 4.48 → 5.4.

## Quality checklist

- Each used slot names its decision; unused slots are stated as unused.
- VAT treatment consistent between margin, break-even ROAS and Google Ads conversion value.
- All thresholds listed with source; no generic numbers presented as rules.
- Min-clicks rule applied; `unrated` count reported.
- Unique values per label well under 1,000; values lowercase ASCII without spaces.
- Coverage adds up to total products per slot.
- Simulated groups have enough conversions for their bid strategy, or are merged.
- Existing live labels and product groups checked; renames listed for sign-off.

## Sources

- https://support.google.com/merchants/answer/6324473 (custom_label_0–4: length, 1,000 unique values, case-insensitive)
- https://support.google.com/google-ads/answer/6275317 (Shopping product groups: 7 levels, 5,000 groups per ad group, attribute changes not auto-applied)
- https://support.google.com/google-ads/answer/11596074 (Performance Max listing groups: 1,000 per asset group, use custom labels)
- https://support.google.com/merchants/answer/7052112 (Product data specification)

## License
MIT
