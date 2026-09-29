---
name: merchant-center-disapproval-fixer
owner: cartlift
category: Product feeds
description: Triages Merchant Center disapprovals by reason and impact, maps each to the attribute or page fix, and proposes feed changes only for a human to approve.
version: v3
license: MIT
updated: 2026-09-09
recommended: true
security_checked: true
url: https://ecomdly.com/skills/cartlift/merchant-center-disapproval-fixer
raw: https://ecomdly.com/raw/cartlift/merchant-center-disapproval-fixer.md
install: npx @ecomdly/cli add cartlift/merchant-center-disapproval-fixer
---

# Merchant Center disapproval fixer

A disapproval list sorted by count sends you after 400 low-value SKUs while the bestseller stays dark. This skill sorts by what the disapproval costs, then names the exact fix.

## When to use
- When the Diagnostics tab shows a jump in disapproved or limited items.
- Weekly, on the item-issues export, before touching the feed.

## Input
The Merchant Center item-issues export (item id, issue code, attribute, severity, affected destinations), the feed rows for those items, and clicks or revenue per item for the last 30 days. Optional: the landing page HTML or a crawl of price and availability.

## Common reasons and their fix
- **Mismatched value (price / availability):** the feed says 49.00 in stock, the page says 54.00 or out of stock. Fix the source of truth, then check structured data (`offers.price`, `offers.availability`) matches. Never "fix" by changing the feed to a price the page does not show.
- **Missing or invalid GTIN / limited performance due to missing identifiers:** see `gtin-identifier-auditor`; set `identifier_exists = no` only for genuinely unbranded or custom items.
- **Missing shipping / tax:** account-level shipping settings or the `shipping` attribute; affected countries listed per item.
- **Promotional overlay or watermark on image, image too small:** `image_link` must be a clean product shot, ≥ 100×100 (apparel ≥ 250×250); move lifestyle shots to `additional_image_link`.
- **Unsupported content / restricted product (policy):** list separately. These need a human decision or an appeal, not a feed edit.
- **Landing page not working / desktop or mobile page not available:** 4xx, 5xx or geo-redirect on `link`. Report the status code seen.

## Process
1. Group issues by issue code and severity (disapproved first, then demoted).
2. For each group, sum 30-day clicks and revenue of affected items. Sort by that.
3. Per group, name the attribute to change, the source system that owns it, and one example item.
4. Mark each fix as feed edit, site edit, account setting, or policy review.

## Rules
- Read-only. Proposed changes go in a table for a human; nothing is submitted to Merchant Center or the feed without explicit confirmation.
- If the item-issues export lacks performance data, say so and sort by item count instead, labelled as such.
- Never guess a GTIN, price or policy outcome.

## Output format
```
| issue | items | 30d clicks | fix | owner | type |
| Mismatched price | 38 | 4 210 | sale price not in feed; add sale_price + dates | ERP export | feed edit |
| Image promotional overlay | 112 | 690 | use packshot as image_link | content | feed edit |
| Restricted: supplements claim | 6 | 150 | appeal or remove "cures" wording | owner | policy review |
Missing data: revenue per item not provided; clicks used.
```

## License
MIT
