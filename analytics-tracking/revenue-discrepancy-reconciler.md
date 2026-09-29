---
name: revenue-discrepancy-reconciler
owner: shopmetric
category: Analytics & tracking
description: Explains why backend, GA4 and Google Ads revenue differ with a bridge of causes — tax, shipping, refunds, consent, attribution, time zones — from supplied numbers only.
version: v2
license: MIT
updated: 2026-09-25
recommended: false
security_checked: true
url: https://ecomdly.com/skills/shopmetric/revenue-discrepancy-reconciler
raw: https://ecomdly.com/raw/shopmetric/revenue-discrepancy-reconciler.md
install: npx @ecomdly/cli add shopmetric/revenue-discrepancy-reconciler
---

# Revenue discrepancy reconciler

"GA4 says 41k, Ads says 47k, the shop says 52k" is normal. This skill turns the gap into a bridge of named causes, so what is left unexplained is small enough to investigate.

## When to use
- When a stakeholder asks which revenue number is right.
- Monthly, before reporting ROAS to anyone.

## Input
For the same date range: backend orders (id, created time, status, total, tax, shipping, discounts, refunds), GA4 purchase events (transaction_id, value, tax, shipping, date), and Google Ads conversions and conversion value by conversion date and by click date. Property and account time zones and currencies. Consent mode setup.

## The bridge, in this order
1. **Definitions.** Backend gross incl. VAT and shipping vs GA4 `value` (often ex tax/shipping) vs Ads value (whatever the tag sends). Restate all three on one basis first.
2. **Cancellations, refunds, test orders.** Backend may net them; GA4 only if `refund` events are sent; Ads only via adjustments.
3. **Time zones and date cut.** Store in UTC, GA4 in Europe/Prague, Ads in another zone moves orders across the day boundary.
4. **Ads counts by click date, GA4 by event date.** A purchase on 2 Sep from a 28 Aug click sits in August in Ads (default columns).
5. **Attribution scope.** Ads counts conversions it can claim in its window, including view-through and cross-device; GA4 attributes across all channels; backend has all orders, including ones no tool attributes.
6. **Consent mode and blockers.** Denied consent removes events from GA4 observed data; Ads may add modeled conversions. Report both effects separately.
7. **Duplicates and gaps.** Repeated or empty `transaction_id`, payment-gateway redirects losing the session, missing currency.
8. **Currency conversion** at different rates or days.

## Rules
- Each bridge line needs a number from the input. Where data is missing, the line says "not quantified" and why.
- Never pick one source as "correct"; say which one to use for which question (finance: backend; channel mix: GA4; bidding: Ads).
- Read-only toward all accounts.

## Output format
```
Backend gross, Sep 1–30          52 340
 − VAT and shipping              −9 870
 − cancelled / refunded          −2 110
 = backend net comparable        40 360
GA4 purchase value               37 920   gap −2 440 (−6.0%)
   consent/blockers (est. from order match 93.9%)  ~ −2 300
   unexplained                   −140
Ads (click date, incl. modeled)  43 150   higher: view-through + modeled, not quantified
```

## License
MIT
