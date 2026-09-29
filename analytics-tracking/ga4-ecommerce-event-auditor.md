---
name: ga4-ecommerce-event-auditor
owner: shopmetric
category: Analytics & tracking
description: Audits GA4 ecommerce events against Google's schema (currency with value, transaction_id, items, value = price x quantity) and matches purchases to backend orders, returning a prioritised fix list for developers.
version: v3
license: MIT
updated: 2026-09-29
recommended: false
security_checked: true
url: https://ecomdly.com/skills/shopmetric/ga4-ecommerce-event-auditor
raw: https://ecomdly.com/raw/shopmetric/ga4-ecommerce-event-auditor.md
install: npx @ecomdly/cli add shopmetric/ga4-ecommerce-event-auditor
---

# GA4 ecommerce event auditor

Audits a store's GA4 ecommerce implementation event by event and parameter by parameter against Google's documented recommended-event schema. It then checks the `purchase` stream against backend orders and returns a pass/fail table with a prioritised fix list for the developer. The naive audit opens DebugView, sees `purchase` arrive and calls it done. That misses the failures that actually corrupt reports: a `value` without `currency`, which GA4 does not report as revenue; an empty `transaction_id`, which makes GA4 deduplicate every such purchase into one; line totals sent as unit prices; and events that fire twice on some templates and never on others. These show up only in a sample of real events and a join to the order book.

## When to use
- After any change to the theme, checkout, payment gateway, consent banner, GTM container or shop-platform GA4 plugin.
- As the gate before `ga4-funnel-analyst`, `revenue-discrepancy-reconciler` or any revenue-by-channel analysis.
- When GA4 revenue or purchase counts step-change without a business reason.
- Before GA4 key events are imported into Google Ads for value-based bidding.

## When not to use
- The question is only "why do the three tools disagree on revenue?" and events are known to be clean. Use `revenue-discrepancy-reconciler`.
- App (Firebase) implementations. Most of the schema applies, but deduplication and debugging differ. Say so and limit the audit to what the sample shows.
- Designing a tracking plan from scratch. This skill checks an existing implementation against the schema. It does not write tags.

## Inputs
Required:
1. **Event sample**, ideally the GA4 BigQuery export for 7 full days (`events_YYYYMMDD` tables). Needed columns: `event_name`, `event_date`, `event_timestamp`, `user_pseudo_id`, `event_params` (`ga_session_id`, `page_location`, `value`, `currency`, `transaction_id`, `coupon`, `shipping_tier`, `payment_type`), `ecommerce.*` (`transaction_id`, `purchase_revenue`, `tax_value`, `shipping_value`, `total_item_quantity`), `items[]` (all fields), `device.category`, `privacy_info.*`, `stream_id`.
   Fallback: GTM Preview or GA4 DebugView captures of one complete journey per template (category, product, cart, each checkout step, each payment method), plus Reports > Monetization > Ecommerce purchases for counts. State that a capture-based audit covers only the journeys captured.
2. **Backend orders** for the same days, in the GA4 property time zone: `order_id`, `created_at`, `channel`, `payment_method`, merchandise ex VAT after discounts, `tax`, `shipping`, `currency`.
3. **Implementation facts**: gtag.js, GTM or a platform plugin; server-side tagging or Measurement Protocol in use (yes/no); consent mode type (none, basic, advanced); GA4 property currency and time zone.

Optional: the tracking plan or dataLayer spec, CMP consent-rate report, list of staging hosts.

Missing inputs: without backend orders, skip the match section and say so. Without BigQuery, run the schema checks on captures and mark the counts-based checks (ordering, duplicates) as "not assessed".

## Best practices

### Schema rules that break reporting (severity: critical)
1. **`currency` whenever `value` is sent.** Google's event reference marks `currency` as required if `value` is set, in 3-letter ISO 4217 format. Without it, the event is not counted as revenue. Check every ecommerce event that carries `value`, not only `purchase`.
2. **`value` is a number, not a string**, with a dot decimal separator and no currency symbol or thousands separator. `"1 299,00 Kč"` is a string that GA4 cannot sum. In BigQuery, check which of `int_value`, `double_value` or `string_value` is populated.
3. **`transaction_id` on `purchase` and `refund`** is required. It must be unique per order, not empty, not a constant (`undefined`, `0`, `null`, `(not set)`), and free of personal data such as e-mail addresses. Google warns that all purchase events with an empty transaction ID are deduplicated together, and that reusing one ID for different transactions can significantly undercount.
4. **`items` array present on item events**, and each item has `item_id` or `item_name` (one is required). Without items, item reports and item-scoped funnels are empty.

### Value semantics (severity: high, because it corrupts revenue quietly)
5. **`value` = Σ(`price` × `quantity`)** over the items, and it does not include `shipping` or `tax`. Those go into their own parameters on `purchase` and `refund`. Test this on every purchase in the sample: `abs(value − Σ price×quantity) > 0.01` is a finding. If the store deliberately sends a different basis (for example after the order-level coupon, or incl. VAT), write down the basis and check that it is consistent, because downstream reconciliation depends on it.
6. **`price` is the discounted unit price.** When a discount applies, set `price` to the discounted unit price and put the unit discount into the item `discount` parameter. A common bug is `price` = line total (unit × quantity). Detect it where `quantity > 1` and `price` equals the backend line total.
7. **`quantity`** defaults to 1 if absent. A missing quantity on multi-unit lines understates item revenue and units.
8. **Consistent `item_id` across events.** The same product must use the same id in `view_item`, `add_to_cart` and `purchase`, and it should match the feed `id` if Merchant Center or Ads use the data. Parent vs variant id mixing breaks item-level funnels. Report the match rate of purchase item ids to view_item item ids.

### Coverage and order (severity: high)
9. **Every step exists.** The core sequence is `view_item_list`/`select_item` → `view_item` → `add_to_cart` → `view_cart` → `begin_checkout` → `add_shipping_info` → `add_payment_info` → `purchase`, with `remove_from_cart`, `refund`, `view_promotion`/`select_promotion` where relevant. A step with zero events is untagged, and GA4 funnel and checkout-journey reports will show it as a 100% drop.
10. **Counts fall down the funnel at session level.** Count distinct sessions (`user_pseudo_id` + `ga_session_id`) per step, not events. If a later step has more sessions than the step before it, the earlier one is under-tagged on some template, or the later one double-fires.
11. **`add_shipping_info` carries `shipping_tier` and `add_payment_info` carries `payment_type`**, with human-readable, stable values ("Zásilkovna pickup", "card"). Report the distinct values, because they are the most useful checkout segmentation available.
12. **Each payment method is covered.** Split purchase coverage by `payment_method` from the backend. Off-site gateways (bank buttons, 3-D Secure, buy now pay later) often lose the return to the thank-you page.

### Duplicates and the dataLayer (severity: high)
13. **Deduplication scope**: GA4 deduplicates purchases with the same transaction ID from the same user, and only on web streams. Duplicates across different users (a thank-you page opened on another device, server-side and browser events with different client IDs) stay in the data. Measure `COUNT(*) − COUNT(DISTINCT transaction_id)` in BigQuery.
14. **Clear the ecommerce object in GTM.** Google's ecommerce guide shows `dataLayer.push({ ecommerce: null })` before each ecommerce push, so items from a previous event are not merged into the next one. Symptom: `add_to_cart` events carrying the whole viewed list.
15. **One purchase source.** If both a browser tag and a server-side or Measurement Protocol purchase are active, confirm that only one sends `purchase`, or that both share the same client ID and transaction ID.

### Consent and completeness (severity: context)
16. The backend match rate cannot reach 100% under consent mode. With basic consent mode, nothing is sent for users who deny. With advanced mode, cookieless pings are sent, but they are not linked to a user. Compare the match rate with the CMP's analytics-consent rate among buyers when available, instead of a fixed threshold. A match rate well below the consent rate, or a drop against the previous audit, points to tagging.
17. Check the GA4 purchase events for staging or other hostnames (`page_location`). A test site sending to the production property inflates revenue.

### Limits worth knowing
18. The `items` array can hold up to 200 elements, and up to 27 custom item-scoped parameters in addition to the prescribed ones. Carts beyond that are truncated.
19. Event-level and item-level `coupon` are independent. Check that each is filled where the store uses it.

## Process
1. **Scope.** Record the property, stream(s), dates, sample source and size, implementation method and consent mode type. Filter out internal and staging hostnames and state how many events were removed.
2. **Coverage table.** For each ecommerce event, give the event count, session count, share of sessions with that event, and the templates or pages where it fires (top `page_location` patterns).
3. **Order check.** Compare consecutive steps at session level. Flag every step where `sessions(step n) > sessions(step n−1)`.
4. **Schema checks per event.** For each rule 1–8 and 11, compute the failing share = failing events ÷ events. Use this severity:
   - critical: rules 1, 2, 3, 4 (revenue or items missing)
   - high: rules 5, 6, 8, 9, 13, 15
   - medium: rules 7, 11, 19
5. **Purchase integrity.** Compute `events`, `distinct transaction_ids`, `empty or constant ids`, `repeat events on the same id` (split by same user vs different user), and the `value` sum check from rule 5.
6. **Backend match.** Normalise ids (trim, case, strip prefixes), then compute:
   - `match_rate = matched backend web orders ÷ backend web orders`
   - `value_agreement = share of matched orders where |GA4 value − backend comparable| ≤ 0.01`
   - `GA4-only ids`, split by hostname
   - match rate by `payment_method` and device. A method more than a few points below the others is a finding. Name it, but do not invent a threshold for the user.
7. **Prioritise.** Critical first, then by revenue affected (failing share × revenue of the event). Give each fix as: what to change, where, how to verify (DebugView check or BigQuery query).
8. **Output** the table and the fix list, then run the quality checklist.

## Pitfalls and edge cases
- **Platform plugins** (Shoptet, Upgates, Shopify apps) often send `value` including VAT or shipping. That is not wrong for reporting consistency, but it must be known and not "fixed" silently, because it would break year-over-year comparisons. Propose the change with its impact on history.
- **Multi-currency shops**: `currency` must be the currency of the transaction, not the property's. GA4 converts at the previous day's rate. Check that the currency switcher updates the dataLayer.
- **Single-page checkouts** fire `add_shipping_info` and `add_payment_info` at the moment the options are chosen. Some themes fire them on page load with default values, which makes the step look 100% completed.
- **Thank-you page as a GTM trigger on URL**: reloads and back-button visits re-fire it. The transaction ID limits the damage for the same user only.
- **Numeric `transaction_id` stored as number vs string**: order `001234` becomes `1234`. Normalise before joining.
- **`items[].price` of 0** for free gifts is valid. Do not flag it. `price` missing is a finding.
- **DebugView shows only debug devices**: absence in DebugView is not absence in production.
- **BigQuery daily tables** can be updated after the day ends, for late hits from apps and Measurement Protocol. Audit days that are at least 3 days old, or state that recent days may change.
- **Consent denied with advanced mode**: events may arrive without `user_pseudo_id`. Count them separately and do not label them as a tagging defect.

## Rules
- Read-only. No changes to GTM, gtag, the theme, plugins or GA4 settings. Output is a fix list for a developer, who deploys after a human approves.
- Report only what the sample shows, and state the sample source, dates and size. Never infer that an event exists because "it usually does".
- Every finding has a count and a denominator ("23 of 1,102 purchase events").
- Do not include personal data from the order export or from event parameters in the output. If a `transaction_id` or any parameter contains an e-mail address or name, report the pattern and count, not the values, and flag it as a privacy issue.
- If a check cannot be run on the available input, write "not assessed" with the reason.

## Output format
```
GA4 ecommerce audit — <property / stream> · <dates> (<time zone>)
Sample: <BigQuery | captures> · <n events> · implementation <GTM|gtag|plugin> · consent mode <none|basic|advanced>
Excluded: <n events from hosts ...>

## Coverage
| event | events | sessions | % of prev step (sessions) | status |
|---|---|---|---|---|
| view_item | n | n | | ok / finding |
...

## Findings (priority order)
| # | severity | event | rule | affected | fix | verify by |
|---|---|---|---|---|---|---|
| 1 | critical | purchase | transaction_id empty | 23 of 1 102 (2.1%) | ... | ... |

## Purchase vs backend
Backend web orders: n · matched: n (<x%>) · value agreement: <x%> of matched
By payment method: <method x%> · <method y%>
GA4-only ids: n (<hosts>)
Value basis actually sent: <formula>

Not assessed: <checks + reason>
```

## Worked example
Illustrative 7-day sample, 7–13 September 2026, from BigQuery. The implementation is a GTM web container with advanced consent mode, in CZK, Europe/Prague.

- Coverage: `view_item` 48,210 events; `add_to_cart` 6,930; `begin_checkout` 2,410; `add_shipping_info` 1,980; `add_payment_info` 0; `purchase` 1,102.
- Purchase integrity: 23 events have an empty `transaction_id`. The other 1,079 events carry 1,066 distinct ids, and 13 are repeats (9 from the same user, which GA4 deduplicates in reports, and 4 from different users, which it does not). 4 events have `value` but no `currency`.
- `add_to_cart`: on 1,247 of 6,930 events (18.0%), `quantity > 1` and `price` equals the line total.
- Backend: 1,188 web orders; 1,041 matched (87.6%); 25 GA4-only ids, all from `staging.` host. That gives 1,041 + 25 = 1,066 ids, which closes the arithmetic. By payment method, card is 91.2% matched and one bank-button gateway 64.0%.

```
| # | severity | event | rule | affected | fix | verify by |
| 1 | critical | purchase | transaction_id empty | 23 of 1 102 (2.1%) | read order id from dataLayer on the thank-you template used by bank-button returns | BigQuery: count empty ids = 0 for 3 days |
| 2 | critical | purchase | currency missing with value | 4 of 1 102 | set currency from order, not from session | DebugView on a EUR order |
| 3 | high | add_payment_info | event missing | 0 events | fire on payment method selection with payment_type | step sessions ≤ add_shipping_info |
| 4 | high | purchase | GA4-only ids from staging | 25 ids | separate measurement ID for staging | no staging host in export |
| 5 | high | add_to_cart | price = line total | 1 247 of 6 930 (18.0%) | send unit price; quantity separately | price × quantity = line total |
Backend match 87.6%; bank-button gateway 64.0% vs card 91.2% → check return URL of that gateway before attributing the gap to consent.
```

## Quality checklist
- Sample source, dates, size, time zone and exclusions are stated.
- Every ecommerce event in scope has a row, including zero-count events.
- Every finding has a count, a denominator, a severity, a fix and a verification step.
- The step order was checked on sessions, not on event counts.
- The purchase id arithmetic closes: distinct ids = matched + GA4-only, and events = distinct ids + repeats + empty.
- The consent context is stated, and the match rate is not judged against an invented benchmark.
- No personal data appears in the output. Anything not assessed is listed.

## Sources
- GA4 recommended events reference: https://developers.google.com/analytics/devguides/collection/ga4/reference/events
- GA4 measure ecommerce (items limits, refunds, clearing the ecommerce object): https://developers.google.com/analytics/devguides/collection/ga4/ecommerce
- [GA4] Minimize duplicate key events with transaction IDs: https://support.google.com/analytics/answer/12313109
- [GA4] Set up ecommerce events: https://support.google.com/analytics/answer/12200568
- [GA4] Currency reference: https://support.google.com/analytics/answer/9796179
- [GA4] Fix missing revenue data: https://support.google.com/analytics/answer/13800978
- [GA4] Behavioral modeling for consent mode: https://support.google.com/analytics/answer/11161109
- GA4 BigQuery export schema: https://support.google.com/analytics/answer/7029846

## License
MIT
