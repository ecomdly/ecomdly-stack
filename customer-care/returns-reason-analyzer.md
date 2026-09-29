---
name: returns-reason-analyzer
owner: helpdeskly
category: Customer care
description: Codes return reasons from free text into root causes (size/fit, damaged, not as described) and links each cluster to a concrete product page or packing fix.
version: v2
license: MIT
updated: 2026-09-24
recommended: false
security_checked: true
url: https://ecomdly.com/skills/helpdeskly/returns-reason-analyzer
raw: https://ecomdly.com/raw/helpdeskly/returns-reason-analyzer.md
install: npx @ecomdly/cli add helpdeskly/returns-reason-analyzer
---

# Returns reason analyzer

"Didn't fit" is not a reason, it is a symptom. This skill codes every return comment into a root cause and sends each cause to the team who can fix the product page, the size chart or the box.

## When to use
- Monthly, on the returns export for the period.
- When a SKU's return rate jumps after a new batch or a new supplier.

## Input
Returns: order ID, SKU, units, reason picked in the form, customer comment, inspection note from the warehouse. Sales: units sold per SKU in the same period. Product page data per SKU: size chart present, measurements, photos count, material stated.

## Root-cause codes
- **SIZE_SMALL / SIZE_LARGE / FIT** — size chart gap or cut differs from comparable brands.
- **DAMAGED_TRANSIT** — box or product damaged on arrival; check packing and carrier.
- **DEFECT** — fault in manufacturing; confirmed by inspection note.
- **NOT_AS_DESCRIBED** — color, material, size or function differs from the page.
- **WRONG_ITEM** — picking error.
- **CHANGED_MIND** — no product fault; withdrawal within 14 days.
- **UNCLEAR** — not enough text to code; never force a code.

The warehouse inspection note overrides the customer's picked reason when they conflict.

## Method
1. Code each return; keep a quote of the comment as evidence.
2. Return rate per SKU = returned units ÷ sold units. Report SKUs with ≥ 20 units sold only.
3. Flag SKUs whose rate is ≥ 1.5× their category average.
4. For each flagged SKU, name the dominant code (≥ 40% of its returns) and the fix.

## Fixes by code
- Size: add garment measurements in cm, a "runs small" note from return data, fit photos.
- Not as described: reshoot color in daylight, state material percentages.
- Damaged: packing spec change or carrier claim pattern.

## Output format
```
SKU 3302 Linen shirt M — sold 140, returned 31 (22%, cat. avg 11%)
dominant: SIZE_SMALL 19/31 — "tight across shoulders", "order one size up"
fix: add chest/shoulder width in cm to PDP; note "runs small, size up"
UNCLEAR: 4 returns without comment
```

## License
MIT
