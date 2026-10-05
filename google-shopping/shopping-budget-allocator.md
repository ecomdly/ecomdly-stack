---
name: shopping-budget-allocator
owner: adsledger
category: Google Shopping
description: Reallocates a fixed monthly budget across Shopping and Performance Max campaigns by margin-based break-even, lost IS to budget vs rank and marginal return on matured data, as stepwise moves for the owner to approve.
version: v1
license: MIT
updated: 2026-10-05
recommended: false
security_checked: true
url: https://ecomdly.com/skills/adsledger/shopping-budget-allocator
raw: https://ecomdly.com/raw/adsledger/shopping-budget-allocator.md
install: npx @ecomdly/cli add adsledger/shopping-budget-allocator
---

# Shopping budget allocator

Produces a stepwise budget reallocation proposal for a fixed monthly Google Ads budget spread across Standard Shopping and Performance Max campaigns: which campaign gives up how much daily budget, which receives it, the expected change in contribution, and when to check before the next step. The naive approach ranks campaigns by average ROAS over the last 30 days and moves money to the top. That fails three ways: the last days are missing conversions that have not been reported yet, average ROAS says nothing about what the next euro buys, and a campaign that loses impressions to rank, not to budget, cannot spend extra money well. The key insight: move budget only where the budget is the binding constraint, and judge each move by marginal ROAS on matured data against that campaign's own margin-based break-even, not by average ROAS against an account-wide target.

## When to use

- At the monthly budget review, or when the owner fixes a new monthly total.
- When one campaign shows "Limited by budget" while another underspends or runs below break-even.
- After `break-even-roas-calculator` changed a campaign's break-even, so the allocation built on old targets is wrong.
- Before peak season, to decide where an unchanged total should sit.

## When not to use

- Computing break-even ROAS or tROAS per campaign: `break-even-roas-calculator`. This skill consumes its output.
- Restructuring products between campaigns or labels: `feed-custom-label-planner`.
- Asset group, signal or product-overlap problems inside a PMax campaign: `pmax-asset-group-reviewer`. Rank-limited campaigns usually land there.
- Ads revenue that does not match store revenue: `revenue-discrepancy-reconciler`. Run it first if the gap is large; allocation on inflated value moves money the wrong way.
- Wasted spend on irrelevant queries: `shopping-search-terms-miner`.

## Inputs

| input | definition | typical source |
|---|---|---|
| Monthly budget (B) | the fixed total the owner will pay for these campaigns in the month | owner |
| Campaign list | name, type (Standard Shopping or PMax), bid strategy, tROAS if set, daily budget, shared budget membership, total-budget use | Google Ads campaigns view |
| Break-even ROAS and margin m per campaign | on the same value basis as the conversion action | `break-even-roas-calculator` |
| Cost and conversion value per campaign by day | at least 8 weeks, by click date | Google Ads report |
| Days to conversion | share of conversion value reported N days after the click | Google Ads segment Conversions > Days to conversion |
| Search IS, Search lost IS (budget), Search lost IS (rank) | per campaign, same matured window | Google Ads campaign columns |
| Budget simulator points | cost and value estimates at other budget levels, where shown | Budget column simulator icon |
| Bid strategy status and change history | Learning, Limited by budget; dates of target, budget and composition changes | campaigns view, change history |

Optional: stock constraints or clearance deadlines per campaign; seasonality from last year's same weeks; a store-level minimum spend agreed with the owner for brand or new-product campaigns.

If a required input is missing, stop at that step and ask. Never fill a break-even, a conversion lag or a marginal ROAS with an industry average.

## Best practices

### Measure on matured data

1. **Exclude the conversion-lag window.** Google Ads attributes conversions to the click date and can report them up to 90 days later, so recent days show full cost with partial value. Use the store's Days to conversion data: end the analysis window at the lag by which the store's chosen share of value (for example 90%) has arrived. Google's general advice is a report end date at least 30 days ago, or more with a longer window.
2. **Use the same window for every metric.** Cost, value, impression share and simulator inputs from different periods cannot be compared.

### Is budget the constraint?

3. **Lost IS (budget) says money would buy more auctions; lost IS (rank) says it would not.** Google defines Search lost IS (budget) as the share of time ads were not shown due to insufficient budget (campaign level only), and lost IS (rank) as the share lost to poor Ad Rank. A campaign with high rank loss and almost no budget loss should get a bid, target or asset fix, not more budget.
4. **A campaign that underspends its budget frees nothing by being cut.** Lowering an unused daily budget changes the plan on paper, not the spend. Report it as headroom, not as a funding source.
5. **For PMax, check what the columns show.** Google calculates PMax impression share from Search and Shopping impressions; if lost IS columns are empty for a campaign, record "not available" instead of reading zero.

### Marginal, not average

6. **Judge a move by marginal ROAS:** Δ value ÷ Δ cost between two budget levels. Google's Performance Max reporting help points to optimizing for marginal ROI instead of average ROI. Average ROAS mixes the cheapest orders with the last ones bought, so it overstates what an added slice of budget returns.
7. **Sources of marginal ROAS, in order:** the budget simulator; then the campaign's own history at different spend levels (matured weeks, same season, no target change in between). Say which one each estimate uses. Google's simulators use the last 7 days, may be unavailable when the campaign hit or nearly hit its daily budget at least once in those 7 days, and the campaign bid simulator is unavailable with shared budgets.
8. **Decision rule per move.** Take budget from campaigns where marginal ROAS of the last slice is below break-even. Give it to campaigns limited by budget whose marginal ROAS on the added slice is above break-even, best first by Δ contribution = Δ value × m − Δ cost. Above break-even but below tROAS is a profit-share question for the owner; show it, do not decide it.

### Budget mechanics

9. **Daily budgets from the monthly total.** Google may spend up to 2× the average daily budget on a given day, but over the month charges no more than 30.4× the average daily budget. Set Σ daily budgets ≤ B ÷ 30.4, and do not flag a single 2× day as overspend.
10. **Shared budgets move money by themselves.** Shared budgets exist for Search, Shopping, Display and Video, not PMax. Inside one, Google reallocates between member campaigns, so allocate to the shared budget as a unit.
11. **Campaign total budgets follow different rules.** They pace over a fixed flight, have no 2× daily limit, and overdelivery is credited against the total. Keep them out of the daily arithmetic and list them separately.
12. **Change in steps and wait.** After a bid strategy setting or composition change the status may show Learning; Google says calibration takes a few conversion cycles (1–2 typically). Google's status list names new strategies, setting changes and composition changes as Learning triggers, not budget changes, so read the status column after each step instead of assuming either way. Make one move per campaign per step and re-measure only after the conversion lag plus at least one conversion cycle. Step size is the owner's choice, stated in the proposal.

## Process

1. **Fix the window:** from Days to conversion, pick the lag L where the store's chosen share of value has arrived; analysis window = 28 days (or longer) ending L days before today.
2. **Per campaign, compute** cost, value, average ROAS, contribution = value × m − cost, Search lost IS (budget) and (rank), and budget utilisation (average daily cost ÷ daily budget).
3. **Classify:** budget-limited (lost IS budget material, utilisation near 100%), rank-limited, not constrained, or below break-even on average.
4. **Estimate marginal ROAS** for each candidate slice from the simulator or matured history; label the source.
5. **Build step 1:** donors (marginal below break-even, or headroom), recipients (budget-limited, marginal above break-even), ranked by Δ contribution. Check Σ daily ≤ B ÷ 30.4.
6. **Flag non-budget actions:** rank-limited campaigns to `pmax-asset-group-reviewer` or target review; campaigns below break-even on average to a tROAS review with the owner.
7. **Set the check date** = step date + L + one conversion cycle, and the metrics to read then.
8. **Write the proposal** for the owner's approval.

## Pitfalls and edge cases

- **Value basis mismatch.** Break-even on ex-VAT value against conversion value incl. VAT misstates every comparison; confirm the basis from the calculator's header.
- **Seasonality.** A matured window from a quiet month underestimates marginal ROAS before a peak; compare with last year's same weeks if available.
- **Clearance and stock.** A campaign below break-even may be the owner's choice to clear stock; flag, do not cut automatically.
- **Out of stock.** A campaign that lost products to stock-outs looks rank- or demand-limited; check the product count served.
- **Small campaigns.** A few conversions per window give noisy ROAS; widen the window or pool with similar campaigns and say so.
- **Brand and PMax cannibalisation.** PMax may take brand traffic; a rising PMax ROAS can be borrowed demand. Note it, and route to `pmax-asset-group-reviewer`.
- **Mid-month changes.** Changing budgets mid-month changes the month's average daily budget; read the actual charge limit in the account rather than recomputing it.

## Rules

- The agent proposes; it never changes budgets, targets, bid strategies or campaign status itself. The owner approves and applies each step.
- Every break-even, margin and lag comes from the store's data or `break-even-roas-calculator`; missing inputs stop the step with a request.
- State the source of every marginal ROAS (simulator or history, with dates).
- One step at a time; the next step is computed only from data matured after the previous one.
- Σ daily budgets × 30.4 must not exceed the monthly budget B.

## Output format

```
BUDGET ALLOCATION · <account> · month <yyyy-mm> · B = <amount> · max Σ daily = B ÷ 30.4 = <x>
Window: <from>–<to> (lag L = <n> days, <x>% of value matured) · value basis: <ex|incl VAT>

| campaign | type | daily | util. | cost | value | ROAS | BE ROAS | lost IS budget | lost IS rank | class |
|---|---|---|---|---|---|---|---|---|---|---|

Step <n> proposal
  <campaign>: <old> → <new> daily · slice marginal ROAS <x> (source <simulator|history dates>) · Δ cost <x> · Δ value <x> · Δ contribution <x>
  Σ daily after step: <x> (≤ <max>) · expected Δ contribution per <n> days: <x>
Non-budget actions: <campaign> → <route>
Check on: <date> · read: cost, value, lost IS budget/rank, status
Owner decisions needed: <list>
```

## Worked example

Illustrative numbers only. Czech store, EUR, value ex VAT. B = 9,000 → max Σ daily = 9,000 ÷ 30.4 = 296.05. Store's Days to conversion: 90% of value within 14 days, so the window is the 28 days ending 14 days ago. Break-even from `break-even-roas-calculator`.

```
| campaign | type | daily | cost | value | ROAS | m | BE ROAS | lost IS budget | lost IS rank | class |
| PMax – Footwear | PMax | 150 | 4,150 | 14,525.00 | 3.50 | 42.4% | 2.36 | 18% | 22% | budget-limited |
| Shopping – Accessories | Shopping | 80 | 1,960 | 4,312.00 | 2.20 | 52.1% | 1.92 | 2% | 61% | rank-limited, headroom |
| PMax – Clearance | PMax | 66 | 1,848 | 3,880.80 | 2.10 | 40.0% | 2.50 | 25% | 9% | below break-even |

Step 1
  PMax – Footwear: 150 → 178 · simulator +28/day: Δ cost +784, Δ value +2,587 · marginal 3.30
      Δ contribution 2,587 × 0.424 − 784 = +312.89
  PMax – Clearance: 66 → 46 · simulator −20/day: Δ cost −560, Δ value −1,008 · marginal of cut slice 1.80
      Δ contribution −(1,008 × 0.400 − 560) = +156.80
  Shopping – Accessories: 80 → 72 · headroom only (avg spend 70/day), Δ cost ≈ 0
  Σ daily after step: 178 + 72 + 46 = 296 (≤ 296.05) · expected Δ contribution per 28 days: +469.69
Non-budget actions: Accessories (61% lost to rank) → target and product review; Clearance average below break-even → owner decides tROAS or stock goal
```

Check: 4,150 × 3.50 = 14,525; 1,960 × 2.20 = 4,312; 1,848 × 2.10 = 3,880.80. 1 ÷ 0.424 = 2.36; 1 ÷ 0.521 = 1.92; 1 ÷ 0.40 = 2.50. 784 × 3.30 = 2,587.2; 1,008 ÷ 560 = 1.80. 312.89 + 156.80 = 469.69. Before: 150 + 80 + 66 = 296. Clearance contribution before the step: 3,880.80 × 0.40 − 1,848 = −295.68, which is why it is a donor even though its lost IS (budget) is 25%.

## Quality checklist

- Window excludes the conversion-lag period; lag taken from the store's own data.
- Every campaign has a break-even on the same value basis as its conversion value.
- Budget-limited and rank-limited campaigns separated; no budget added to a rank-limited one.
- Every marginal ROAS has a stated source and date range.
- Σ daily budgets ≤ B ÷ 30.4; shared and total budgets handled separately.
- Δ contribution arithmetic shown per move.
- Check date set after lag plus a conversion cycle.
- Proposal only; nothing applied by the agent.

## Sources

- Google Ads Help, About overdelivery and your average daily budget (2× daily, 30.4× monthly): https://support.google.com/google-ads/answer/2375423
- Google Ads Help, Get impression share data: https://support.google.com/google-ads/answer/7103314
- Google Ads Help, About impression share: https://support.google.com/google-ads/answer/2497703
- Google Ads Help, Estimate your results with bid, budget and target simulators: https://support.google.com/google-ads/answer/2470105
- Google Ads Help, About shared budgets: https://support.google.com/google-ads/answer/10487241
- Google Ads Help, Campaign total budgets FAQs: https://support.google.com/google-ads/answer/15137812
- Google Ads Help, Duration of the learning period for campaigns: https://support.google.com/google-ads/answer/13020501
- Google Ads Help, Find out how long it takes for your customers to convert: https://support.google.com/google-ads/answer/6239119
- Google Ads Help, About asset group reporting for Performance Max (marginal vs average ROI link): https://support.google.com/google-ads/answer/13872527

## License
MIT
