---
name: ga4-ecommerce-event-auditor
owner: shopmetric
category: Analytics & tracking
description: Audits GA4 ecommerce events against the recommended schema — required params, items[] fields, currency and transaction_id dedupe — and reports gaps without editing tags.
version: v2
license: MIT
updated: 2026-09-18
recommended: false
security_checked: true
url: https://ecomdly.com/skills/shopmetric/ga4-ecommerce-event-auditor
raw: https://ecomdly.com/raw/shopmetric/ga4-ecommerce-event-auditor.md
install: npx @ecomdly/cli add shopmetric/ga4-ecommerce-event-auditor
---

# GA4 ecommerce event auditor

Every GA4 revenue report rests on six events being sent correctly. This skill checks them field by field, so the funnel and the revenue numbers can be trusted.

## When to use
- After a theme, checkout or tag manager change.
- Before any funnel or revenue analysis, as the gate.

## Input
A sample of raw events (BigQuery export or DebugView / GTM preview captures) for `view_item`, `add_to_cart`, `begin_checkout`, `add_shipping_info`, `add_payment_info`, `purchase`, plus `refund` if used. Backend order ids and totals for the same days.

## Checks per event
- **Present and ordered.** Counts per day decrease down the funnel. A step with zero events means it is not tagged.
- **Event-level params.** `currency` (ISO 4217) is required whenever `value` is sent; without it revenue is dropped. `value` is a number, not a string with a currency sign.
- **items[].** Each item needs `item_id` or `item_name`; also check `price`, `quantity`, `item_brand`, `item_category`, `item_variant`. `price` is the unit price after item discount, not the line total.
- **value vs items.** `value` should equal Σ(price × quantity) minus order-level discounts; `tax` and `shipping` are separate params on `purchase`. Report the store's convention if it deliberately differs.
- **add_shipping_info / add_payment_info** carry `shipping_tier` and `payment_type`.
- **purchase.** `transaction_id` present and unique. GA4 dedupes repeats of the same id, but an empty or constant id ("undefined", "0") breaks this; a thank-you page reload with a new id doubles revenue.
- **Match to backend.** Share of backend order ids found as `transaction_id`; below 90% is a finding, with consent mode and blockers named as expected causes.

## Rules
- Report only what the sample shows; state the sample size and dates. Do not infer an event exists because a report "usually" has it.
- No tag, GTM or code changes are made; fixes are proposed for the developer.

## Output format
```
| event | 7d count | issues |
| view_item | 48 210 | ok |
| add_to_cart | 6 930 | items[].price is line total on 18% of events |
| add_payment_info | 0 | not tagged |
| purchase | 1 102 | 23 empty transaction_id; currency missing on 4 |
Backend match: 1 041 of 1 188 orders (87.6%) → below 90%, check consent mode.
```

## License
MIT
