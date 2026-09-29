---
name: break-even-roas-calculator
owner: marginmath
category: Google Shopping
description: Computes break-even and target ROAS from contribution margin after COGS, fees, fulfilment and returns, using only the store's numbers and showing every step.
version: v3
license: MIT
updated: 2026-09-12
recommended: true
security_checked: true
url: https://ecomdly.com/skills/marginmath/break-even-roas-calculator
raw: https://ecomdly.com/raw/marginmath/break-even-roas-calculator.md
install: npx @ecomdly/cli add marginmath/break-even-roas-calculator
---

# Break-even ROAS calculator

A 400% ROAS target copied from a blog can lose money on every order. Break-even ROAS is 1 divided by contribution margin, and this skill builds that margin honestly.

## When to use
- Before setting tROAS on Shopping or PMax.
- When a category's margin, fees or return rate changes.

## Input
Average order value (and whether it includes VAT and shipping), COGS per order, payment fees, shipping and pick/pack cost, shipping charged to customer, return rate, cost per return, share of returned COGS that can be resold. Ideally per category.

## Method
1. **Match the basis.** ROAS = conversion value ÷ ad cost. Use revenue on the same basis as the conversion value sent to Google Ads (usually ex VAT, before returns). A mismatch here moves the answer more than anything else.
2. **Contribution per order** = revenue after returns − COGS net of resold returns − payment fees − fulfilment − return handling.
3. **Contribution margin** = contribution ÷ the revenue basis from step 1.
4. **Break-even ROAS** = 1 ÷ contribution margin.
5. **Target ROAS** for a profit share p of revenue = 1 ÷ (margin − p).

## Worked example
AOV 80.00 ex VAT; COGS 32.00; fees 2.9% + 0.30 = 2.62; shipping + packing 8.50; returns 12%, 75% of returned stock resold, return shipping 6.00.
- Revenue after returns: 80 × 0.88 = 70.40
- COGS net: 32 − (0.12 × 32 × 0.75) = 29.12
- Return shipping: 0.12 × 6 = 0.72
- Contribution: 70.40 − 29.12 − 2.62 − 8.50 − 0.72 = 29.44
- Margin: 29.44 ÷ 80 = 36.8% → **break-even ROAS 2.72 (272%)**
- Keeping 10% profit: 1 ÷ (0.368 − 0.10) = **3.73 (373%)**

## Rules
- Every input comes from the store. If one is missing, stop at that line and ask; do not plug in an "industry average".
- Show the arithmetic, not only the result.
- If margin ≤ p, say that no ROAS target reaches the goal at this cost structure.
- The output is advice; the agent does not change bid targets.

## Output format
```
| category | margin | break-even ROAS | tROAS (10% profit) |
| Footwear | 36.8% | 2.72 | 3.73 |
| Socks | 52.1% | 1.92 | 2.37 |
Basis: ex VAT, before returns (matches Ads conversion value).
```

## License
MIT
