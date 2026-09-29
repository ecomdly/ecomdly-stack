---
name: clearance-markdown-planner
owner: marginmath
category: Pricing & merchandising
description: Plans staged markdowns from weeks of cover and sell-through, never below the margin floor you set, with EU 30-day lowest-price references on every reduction.
version: v2
license: MIT
updated: 2026-09-15
recommended: true
security_checked: true
url: https://ecomdly.com/skills/marginmath/clearance-markdown-planner
raw: https://ecomdly.com/raw/marginmath/clearance-markdown-planner.md
install: npx @ecomdly/cli add marginmath/clearance-markdown-planner
---

# Clearance markdown planner

Markdowns that come too late cost margin twice: once on the discount, once on the storage. This skill schedules them from stock and sales data, step by step, and stops at your floor.

## When to use
- End of season, or when a SKU's stock outlasts its selling window.
- Monthly on slow movers, before stock ages into write-off.

## Input
Per SKU: units on hand, units sold per week for the last 8 weeks, current price, landed cost, the lowest price charged in the last 30 days, and the exit date (end of season or last day you want to hold stock). The store's margin floor (e.g. gross margin ≥ 10%, or price ≥ landed cost).

## Formulas
- **Weekly rate** = average units sold over the last 4 weeks (use 8 if sales are lumpy).
- **Weeks of cover (WOC)** = units on hand ÷ weekly rate.
- **Sell-through** = units sold ÷ (units sold + units on hand), since season start.
- **Weeks left** = weeks until the exit date.

## Cadence
1. WOC ≤ weeks left → no markdown; recheck in 2 weeks.
2. WOC between 1× and 2× weeks left → first step 20–30%.
3. WOC above 2× weeks left, or sell-through under 40% at mid-season → first step 30–40%.
4. Review every 2 weeks. If the weekly rate did not rise by at least half, take the next step (typically +15–20 points).
5. Stop when price would fall below the floor. Report the SKU for bundle, outlet or return-to-vendor instead.

## Rules
- Floor price = landed cost ÷ (1 − floor margin). Never propose a price under it.
- Every reduction shows the lowest price of the prior 30 days as its reference (EU Omnibus rule). If that price is missing, flag it.
- No invented demand uplift. Expected outcome is stated as the weekly rate needed, not a forecast.

## Output format
```
SKU 4471 Trail jacket M — on hand 180, rate 9/wk, WOC 20, weeks left 8
step 1 (wk 36): 2 490 → 1 790 Kč (−28%), 30-day low 2 490, floor 1 240
needed rate: 22/wk to clear by exit; recheck wk 38
SKU 4490 Fleece S — WOC 41, at floor after step 2 → outlet or bundle
```

## License
MIT
