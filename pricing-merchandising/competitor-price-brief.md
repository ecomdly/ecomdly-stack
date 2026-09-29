---
name: competitor-price-brief
owner: marginmath
category: Pricing & merchandising
description: Compares your prices with competitor data you supply, matched by EAN, and recommends moves that respect your margin floor. Uses only provided or public data.
version: v2
license: MIT
updated: 2026-09-22
recommended: false
security_checked: true
url: https://ecomdly.com/skills/marginmath/competitor-price-brief
raw: https://ecomdly.com/raw/marginmath/competitor-price-brief.md
install: npx @ecomdly/cli add marginmath/competitor-price-brief
---

# Competitor price brief

Price gaps only matter where shoppers compare. This skill lines up your offers against the competitor prices you bring, finds where you are out of the market, and where you are giving margin away for nothing.

## When to use
- Weekly, on a competitor export from Heureka/Zboží shop listings, a price-monitoring tool, or a CSV you keep.
- Before a promotion, to check which SKUs actually need it.

## Input
Your SKUs: EAN, price incl. VAT, shipping price, landed cost, stock, margin floor. Competitor rows: shop name, EAN, price, shipping price, availability, date observed. Work only with data the user provides or pages that are public; never log into competitor accounts or pull prices from behind a login.

## Method
1. Match on EAN. Match by name only when the user allows it, and mark those as `name-match`.
2. Compare **total price** (item + cheapest shipping) unless the user says otherwise.
3. Drop competitor rows older than 7 days or marked out of stock.
4. Per SKU compute your position: gap to the cheapest in-stock offer and to the median.
5. Classify:
   - **Out of market** — more than 5% above the cheapest and above the median.
   - **Leaving money** — cheapest by more than 5% versus the next offer.
   - **In band** — everything else.
6. Recommend a price only where the gap is actionable. Never below floor = landed cost ÷ (1 − floor margin).

## Rules
- State the observation date for each competitor price.
- No price is inferred for a competitor that has no row.
- Always state the full shipping cost you compared against.

## Output format
```
EAN 8594012345678 Merino socks 3-pack
you 449 + 79 = 528 | cheapest ShopA 399 + 69 = 468 (2026-09-21) | median 510
status: out of market (+12.8%) | suggest 419 (floor 312) → 498 total
EAN 8594012349999 Trail bottle 750 ml
you 289 + 79 | next offer 349 + 79 | status: leaving money, room to 329
```

## License
MIT
