---
name: revenue-discrepancy-reconciler
owner: shopmetric
category: Analytics & tracking
description: Reconciles backend, GA4 and Google Ads revenue with an order-level join and a line-by-line bridge (VAT and shipping, time zones, consent, gateway loss, test orders, click vs conversion date) for store owners and analysts.
version: v3
license: MIT
updated: 2026-09-29
recommended: false
security_checked: true
url: https://ecomdly.com/skills/shopmetric/revenue-discrepancy-reconciler
raw: https://ecomdly.com/raw/shopmetric/revenue-discrepancy-reconciler.md
install: npx @ecomdly/cli add shopmetric/revenue-discrepancy-reconciler
---

# Revenue discrepancy reconciler

Produces a revenue bridge that walks from the backend (shop/ERP) total to the GA4 total and then sets the Google Ads number beside it. Each line in the bridge is a named cause with a number, so only a small unexplained remainder is left to investigate. The naive approach compares three monthly totals and blames "tracking". That fails because the three systems measure different things. The backend records every order at its gross value. GA4 records `purchase` events from browsers that loaded the tag and allowed it, usually excluding tax and shipping. Google Ads records only the conversions it can attribute to an ad interaction, dated by the click. Most of the gap is definitions, not data loss. An order-level join (backend order id to GA4 `transaction_id`) is the only way to separate the two.

## When to use
- Someone asks "which revenue number is right?", or the gap between tools grows.
- Monthly, before ROAS or channel revenue goes into a report or budget decision.
- After a checkout, payment-gateway, consent banner or tag change, and before value-based bidding goes live.

## When not to use
- The events themselves are suspected to be malformed (missing `currency`, string values, empty `transaction_id`). Run `ga4-ecommerce-event-auditor` first. A bridge built on broken events explains nothing.
- The question is which channel deserves credit. That is attribution modelling, not reconciliation.
- Accounting revenue recognition. This skill explains analytics gaps only.

## Inputs
Required, all for the same date range and stated time zone:
1. **Backend orders**, one row per order: `order_id`, `created_at` with time zone, `status` (paid, cancelled, refunded, test), `channel` (web, phone, admin, marketplace), `currency`, `total_gross`, `tax`, `shipping_gross`, `order_level_discount`, `refunded_amount`, `refund_date`, `payment_method`. Source: order export from Shoptet, Upgates, Shopify, WooCommerce or the ERP.
2. **GA4 purchases per transaction.** Best: BigQuery export rows with `event_name = 'purchase'`: `ecommerce.transaction_id`, `event_timestamp`, `ecommerce.purchase_revenue`, `tax_value`, `shipping_value`, raw `value` and `currency` params, `user_pseudo_id`. Fallback: a free-form Exploration with `Transaction ID` × `Purchase revenue`, noting whether it was sampled or thresholded.
3. **Google Ads**, segmented by `Conversion action`: `Conversions` and `Conv. value` (click date) plus the `(by conv. time)` versions. Also each purchase action's settings: source (Ads tag, GA4 import, offline import), counting, click-through, engaged-view and view-through windows, primary or secondary, enhanced conversions on or off.
4. **Settings**: GA4 property time zone and currency, Ads account time zone and currency, backend time zone, consent mode type (none, basic, advanced) and CMP consent rate if available.

Optional: GA4 `refund` events, Ads conversion adjustment uploads, payment gateway logs (to check who reached the thank-you page), list of staging or test hosts.

A missing input keeps its bridge line, marked "not quantified" with the reason; never substitute a typical percentage. With monthly totals only, the bridge can be structural (definitions, date cut) but not order-level; ask for the transaction export.

## Best practices

### Put everything on one basis before comparing
1. **Define the comparable amount explicitly.** GA4's documented convention is `value` = Σ(`price` × `quantity`) of the items, with `shipping` and `tax` excluded and sent as separate parameters. Item `price` is the discounted unit price. Backend totals are usually gross, including VAT and shipping. Restate the backend on the GA4 basis (merchandise, ex VAT, after item discounts) before any other step, because VAT alone is a large share of gross (at a 21% rate, 21/121 ≈ 17% of the gross amount).
2. **Find out what the tag actually sends; do not assume.** Many shop platforms send `value` including VAT, or including shipping, or after the order-level coupon. Compare `value` against backend fields for 10–20 matched orders and state the tag's real formula. Every later step depends on it.
3. **Know which conversion source feeds Ads.** If the Ads purchase action is imported from GA4, its value has the GA4 definition but Ads attribution. If it is a Google Ads tag, the value is whatever that tag sends, which may differ from GA4 on the same site. Two tags with two value formulas is a common finding.

### Order of the bridge (fixed; each step works on the output of the previous)
4. **Definitions** (tax, shipping, order-level discounts, currency of the value).
5. **Scope of orders**: remove channels no web tag can see (phone, admin, marketplace, POS) and test orders. These are real revenue that analytics is not designed to capture.
6. **Date cut and time zone**: convert every timestamp to the GA4 property time zone, then cut the period. A backend export in UTC against a GA4 property in Europe/Prague moves the first one or two hours of the month across the boundary. A change to the GA4 property time zone applies only from that point forward, so check the change history.
7. **Status timing**: GA4 records the purchase when the event fires. The backend's "net" may already exclude orders cancelled later. Compare on orders as placed, and handle cancellations and refunds as a separate line.
8. **Order-level join** of backend `order_id` to GA4 `transaction_id` (normalise case, whitespace, prefixes such as `#`). It splits the rest into three sets. **Matched, value differs**: definitional drift (coupons, price basis, currency).
9. **Backend-only (no GA4 event)**: consent denial, ad or tracking blockers, the payment gateway not returning the buyer to the thank-you page, the tag not firing on some payment methods, or JavaScript errors. Group by `payment_method` and device first. A cluster on one gateway is a tagging bug, not consent.
10. **GA4-only (no backend order)**: test orders from staging using the production measurement ID, orders deleted in the backend, duplicate or fake ids, or orders from another shop sharing the property.

### Duplicates and identity
11. GA4 deduplicates `purchase` events with the same transaction ID from the same user, on web streams only (not app streams). The same ID from a different user (thank-you page reopened on another device, server-side call with another client ID) is not deduplicated. An empty `transaction_id` is a trap: Google states that all purchase events with `transaction_id=""` are deduplicated together, so revenue is undercounted, not doubled. BigQuery holds the raw events, so count `COUNT(*)` against `COUNT(DISTINCT transaction_id)` there. Do not rely on report totals.
12. In Google Ads, a purchase action should count "Every" conversion, and duplicate protection relies on the transaction ID being passed to the tag. Confirm in the action settings and the tag configuration.

### Attribution and dates (Ads vs GA4 cannot be bridged order by order)
13. **Click date vs conversion date.** The standard Ads `Conversions` column reports on the date of the ad interaction. The `(by conv. time)` columns report on the date the conversion happened. Use the "by conv. time" columns for any comparison with GA4 or the backend. Click-date numbers for recent periods keep growing while the window runs, so state the pull date.
14. **Windows.** For a new Ads conversion action the defaults are a 30-day click-through window, a 3-day engaged-view window and a 1-day view-through window, all configurable. Read the actual settings. GA4's key-event lookback defaults are 30 days for acquisition key events and 90 days for all other key events (options 30/60/90). Different windows mean different claims on the same order.
15. **Scope of credit.** Ads can claim view-through and engaged-view conversions and cross-device conversions, and it includes modeled conversions where consent or browser limits prevent observation. GA4 distributes credit across all channels using its reporting attribution model (data-driven or last click). The backend credits nobody. So the "Google Ads" row in GA4 is expected to be lower than Ads' own number. Summing platform-reported revenue across Google, Meta and Heureka overstates the total, because each platform claims the same orders.
16. **Primary vs secondary actions.** The `Conversions` column includes only primary actions in the goals used for bidding. `All conv.` includes secondary actions and view-through. State which column was used.

### Consent mode and modeling
17. With consent mode, denied `analytics_storage` means the GA4 event is not stored with identifiers. In basic mode, no event is sent at all. In advanced mode a cookieless ping is sent. GA4 behavioural modeling can then estimate users and sessions, but only if the property meets its prerequisites. Google lists at least 1,000 events per day with `analytics_storage='denied'` for at least 7 days, and at least 1,000 daily users with `'granted'` for at least 7 of the previous 28 days. Even then, eligibility is not guaranteed. Modeled data appears only under the Blended reporting identity.
18. Separate **observed** from **modeled** in the bridge. Order-level matching uses observed events only. Ads totals may include modeled conversions that cannot be matched to any order. Report that as its own line and never net it against the consent loss.
19. A backend-only share close to the CMP's denial rate among buyers suggests consent, but it does not prove it. If the backend-only orders cluster on one payment method or browser, a technical cause is more likely.

### Refunds, cancellations and currency
20. GA4 reduces revenue only if `refund` events are sent (with `transaction_id` and items). Ads reduces conversion value only if conversion adjustments (retract or restate) are uploaded. Google Ads Help sets time limits on adjustments, so confirm the current window in "About conversion adjustments". The backend reflects refunds on the refund date. Show refunds as their own bridge line in each system and state which date they are dated by.
21. GA4 converts non-reporting-currency values using the exchange rate of the day before the transaction. Ads and the backend may use other rates. On multi-currency shops (CZK/EUR), reconcile each currency separately first, then convert once with a stated rate.

### Which number for which question
22. Finance, VAT and margin: backend. Channel mix and on-site behaviour: GA4 (with the consent gap stated). Bid optimisation inside Google Ads: Ads, because it is the number the bidding system learns from. Say this in every output. Never declare one source "correct".

## Process
1. **Collect settings**: time zones, currencies, consent mode type, Ads conversion source, counting and windows. If one is unknown, ask before computing.
2. **Normalise the backend.** Convert `created_at` to the GA4 property time zone, filter to the period, and compute `comparable = total_gross − tax − shipping_gross − order_level_discount` (adjust to the tag's real formula found in best practice 2). Flag `status = test` and non-web channels.
3. **Normalise GA4.** From BigQuery, take all purchase events in the period. Compute `events`, `distinct_ids`, `empty_ids` (`transaction_id` null or `''` or constants such as `undefined`, `0`, `(not set)`) and `value_sum`.
4. **Join** on the normalised id. Produce four sets: matched, backend-only, GA4-only, and unmatchable (GA4 events with empty ids).
5. **Quantify each bridge line**, in the order Definitions, Scope, Date cut, Backend-only, then GA4-only, using the formulas below:
   - `definition_delta = Σ(ga4_value − backend_comparable)` over matched orders
   - `match_rate = matched_orders ÷ backend_web_orders`
   - `backend_only_value = Σ backend_comparable` over backend-only orders, then split by `payment_method` and device
   - `ga4_only_value = Σ ga4_value` over GA4-only ids, then split by hostname or page location where available
   - `unexplained = GA4_total − (backend_comparable_web + Σ bridge lines)`
6. **Classify backend-only orders.** A payment method with a match rate well below the others is a probable tagging or redirect issue. The rest is labelled "not observed (consistent with consent denial or blockers)", never plainly "consent".
7. **Ads section.** Report Ads value by conv. time, the click-date value beside it (difference = "date basis"), and GA4 revenue attributed to Google Ads under the property's model. List the structural reasons Ads is higher (best practices 13–16) and quantify only what Ads exposes as columns; `View-through conv.` is a count, so its value stays "not quantified".
8. **Decide the verdict.** If `|unexplained| ÷ GA4_total` is small enough that no decision would change, close the bridge. Otherwise list the next investigation step for the largest unexplained line. Ask the user what tolerance they work to. Do not invent one.
9. Write the output, then run the quality checklist.

## Pitfalls and edge cases
- **Off-site payment redirects** (bank buttons, 3-D Secure, buy now pay later): buyers who close the tab after paying never reach the thank-you page. This looks like consent loss but clusters on one payment method.
- **Browser tag plus Measurement Protocol purchase**: double-counts unless only one sends `purchase`.
- **Cash on delivery** refused weeks later: GA4 keeps the purchase; it belongs in the status-timing line.
- **Edited orders**: staff change items after purchase; GA4 keeps the original value, so the order lands in matched value differences.
- **Mixed-VAT carts**: use the per-order `tax` field, never gross ÷ one rate.
- **GA4 data thresholds and sampling** in Explorations can hide rows. BigQuery is not thresholded.

## Rules
- Read-only. Do not change tags, GTM containers, consent settings, conversion actions or ad accounts. Proposed fixes go to a human, who applies them.
- Every bridge line carries a number from the inputs, or "not quantified" with the reason. Never fill a line with an industry percentage.
- Do not net modeled Ads conversions against observed GA4 losses. They are different quantities.
- Customer data (emails, names) from order exports is used only for matching and never appears in the output.

## Output format
```
Revenue reconciliation — <period> (<time zone>)
Basis: <e.g. merchandise ex VAT, after item discounts, excl. shipping> · currency <CUR>
Data pulled: <date> · GA4 source: <BigQuery | Exploration (sampled? thresholded?)>

## Bridge: backend → GA4
| line | orders | value | note |
| Backend, all orders, gross | n | x | <source export> |
| − tax and shipping | | x | per-order fields |
| − non-web channels / test orders | n | x | phone, admin, marketplace, test |
| = backend comparable (web) | n | x | |
| − date cut / time zone | n | x | or "none: all in <tz>" |
| − backend-only, clustered on <payment method> | n | x | probable tagging/redirect |
| − backend-only, not observed | n | x | consistent with consent/blockers; CMP denial <x% or unknown> |
| ± matched value differences | n | x | <cause, e.g. order-level coupon in value> |
| + GA4-only ids | n | x | <cause> |
| = GA4 purchase revenue | n | x | |
| unexplained | | x (<y%> of GA4) | |
Match rate: <matched> / <backend web orders> = <z%>

## Google Ads (not bridgeable order by order)
Conv. value by conv. time: x · by click date: x (date basis difference)
GA4 revenue attributed to Google Ads (<model>, <lookback>): x
Why Ads is higher: <view-through n (value not quantified) | modeled | windows | model>

## Which number for which question
Finance: backend <x> · Channel mix: GA4 <x> · Bidding: Ads <x>

## Next steps (for a human to approve)
1. <largest unexplained or fixable line, owner, how to confirm>
```

## Worked example
Illustrative numbers for a Czech shop, September 2026. All systems are cut to Europe/Prague and amounts are in CZK. The tag sends `value` = Σ(price × quantity) ex VAT, without deducting order-level coupons.

Backend: 1,000 orders, 1,900,000 merchandise ex VAT after item discounts; 12 phone/admin orders (24,000) and 2 test orders (3,000) are removed. The join to GA4 gives 905 matched, 81 backend-only, 16 GA4-only and 0 empty ids. Of the backend-only orders, 14 sit on one bank-transfer gateway (match rate 71% against 93% for card); the other 67 show no cluster. GA4-only ids come from the staging host (10) and from orders deleted in the backend as fraud (6).

```
| Backend comparable (web) | 986 | 1 873 000 | |
| − backend-only, bank-transfer gateway | 14 | −27 000 | redirect loss, probable |
| − backend-only, not observed | 67 | −126 000 | 6.8% of web orders; CMP denial rate unknown |
| + matched value differences | 905 | +6 900 | order-level coupon not deducted |
| + GA4-only: staging host | 10 | +19 000 | |
| + GA4-only: deleted fraud orders | 6 | +11 800 | |
| = GA4 purchase revenue | 921 | 1 758 300 | |
| unexplained | | +600 (0.03%) | |
Match rate: 905 / 986 = 91.8%
```
The Ads purchase action (a Google Ads tag using the same value formula) shows 1,212,000 by conversion time and 1,160,000 by click date. GA4 attributes 902,000 to Google Ads under data-driven attribution with a 90-day lookback. Ads is higher by design (view-through, cross-device, modeled); not quantified in value.

Next steps: (1) developer checks the thank-you redirect for the bank-transfer gateway; (2) exclude the staging host from the production data stream or give staging its own measurement ID; (3) decide whether `value` should deduct order-level coupons, and apply the same rule in the Ads tag.

## Quality checklist
- The time zone, currency and value basis are stated at the top, and all systems were cut the same way.
- Every bridge line has a number or "not quantified" with a reason, and the lines add up exactly to the GA4 total plus the unexplained remainder.
- Order counts add up: matched + backend-only = backend web orders, and matched + GA4-only = GA4 distinct ids.
- The Ads comparison uses "by conv. time" columns, and the column set (`Conversions` vs `All conv.`) is named.
- Modeled and observed data are never netted against each other.
- No customer PII in the output. No action is taken on any account.
- The "which number for which question" block is present.

## Sources
- GA4 recommended events reference (value, currency, tax, shipping, price, discount): https://developers.google.com/analytics/devguides/collection/ga4/reference/events
- GA4 measure ecommerce (refunds, items array limits): https://developers.google.com/analytics/devguides/collection/ga4/ecommerce
- [GA4] Minimize duplicate key events with transaction IDs: https://support.google.com/analytics/answer/12313109
- [GA4] Currency reference: https://support.google.com/analytics/answer/9796179
- [GA4] Behavioral modeling for consent mode: https://support.google.com/analytics/answer/11161109
- [GA4] Select attribution settings: https://support.google.com/analytics/answer/10597962
- [GA4] About data thresholds: https://support.google.com/analytics/answer/9383630
- Google Ads, About conversion windows: https://support.google.com/google-ads/answer/3123169
- Google Ads, Understand your conversion tracking data (by conv. time): https://support.google.com/google-ads/answer/6270625
- Google Ads, Data discrepancies: https://support.google.com/google-ads/answer/7457111
- Google Ads, About conversion adjustments: https://support.google.com/google-ads/answer/7686447

## License
MIT
