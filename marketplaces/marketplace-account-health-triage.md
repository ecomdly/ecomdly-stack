---
name: marketplace-account-health-triage
owner: listwise
category: Marketplaces
description: Turns Amazon Account Health, Allegro sales-quality and Kaufland performance exports into a risk-ranked fix plan: metric, window, slack in orders, roll-off date, root cause, fix and owner per issue.
version: v1
license: MIT
updated: 2026-10-05
recommended: false
security_checked: true
url: https://ecomdly.com/skills/listwise/marketplace-account-health-triage
raw: https://ecomdly.com/raw/listwise/marketplace-account-health-triage.md
install: npx @ecomdly/cli add listwise/marketplace-account-health-triage
---

# Marketplace account health triage

Produces a ranked fix plan from a seller's performance dashboards on Amazon (Account Health), Allegro (Jakość mojej sprzedaży) and Kaufland (seller performance KPIs): for each metric, its exact definition and window, the current value against the platform's own target, how many orders of slack are left, when the bad orders roll out of the window, the root cause found in the order data, and one fix with an owner. Reading the dashboard colours fails twice: a metric that is green can be one or two orders away from a breach on a small denominator, and a red metric often repairs itself in days as old orders leave a short window while a different, still-green metric is the one heading for a restriction. The key insight: each metric is a count of specific orders over a specific window, so risk is measured in orders and dates (slack, roll-off, inflow), and causes are found by reading the failing orders, where one root cause such as overselling across channels usually drives several metrics on several platforms at once.

## When to use
- When a platform sends a performance warning, a listing or category restriction, or the dashboard turns amber or red.
- Weekly for sellers close to a threshold, and before peak season or a jump in order volume.
- After changing carrier, warehouse, stock-sync tool or handling time.

## When not to use
- Whether a SKU or channel is worth keeping: `marketplace-profitability-check`.
- Fixing listing content or attributes flagged by a policy: `marketplace-listing-adapter`.
- Analysing return reasons in depth: `returns-reason-analyzer`.
- Drafting replies to individual buyers: `order-status-reply-drafter`; replying to reviews: `review-response-drafter`.

## Inputs

| input | definition | typical source |
|---|---|---|
| Performance export per platform | each metric's current value, window and the platform's target as shown | Amazon Account Health and its downloadable reports; Allegro Jakość mojej sprzedaży; Kaufland Seller Portal performance |
| Targets | the platform's own threshold per metric and the consequence named | the same dashboard or the platform's policy page |
| Order export | order ID, order date, channel, fulfilment method, promised and actual dispatch date, cancellation and who initiated it, carrier, tracking number, delivery date | Seller Central reports, Allegro and Kaufland order exports, OMS |
| Defect lists | negative feedback, A-to-z claims, chargebacks, Allegro discussions and ratings, Kaufland tickets, with order IDs | platform reports |
| Policy notices | listing, product-safety, IP or authenticity notices with ASIN, offer or EAN | Account Health, platform messages |
| Stock-sync setup | which system holds stock, sync interval, buffer per channel | OMS or integration settings |
| Owners | who runs warehouse, carrier contracts, integration, customer service, catalog | owner |

Optional: daily order volume forecast (to project inflow), holiday and carrier pick-up calendar, handling-time settings per channel.

If a target is not visible in the export, ask for a screenshot or the policy page; never use a threshold from memory or from another platform. Missing order-level data: report the metric and say the root cause cannot be shown.

## Best practices

### Read the metric exactly
1. **Definitions differ by platform; use the platform's.** Amazon's published Order Performance policy defines: Order Defect Rate as orders with negative feedback, an A-to-z Guarantee claim or a credit card chargeback, over a 60-day period, target under 1%; Cancellation Rate (pre-fulfilment) as seller-cancelled orders over 7 days, seller-fulfilled only, excluding cancellations the buyer requests through their Amazon account, target under 2.5%; Late Dispatch Rate as orders confirmed after the expected ship date over 10 and 30 days, merchant-fulfilled only, target under 4%; Valid Tracking Rate as shipments with valid tracking over 30 days, seller-fulfilled only, target above 95% per category. Confirm the current numbers on the dashboard; it is authoritative.
2. **Know which orders count.** On Amazon, ODR covers all orders, FBA included, while the cancellation, late dispatch and tracking metrics cover seller-fulfilled orders only. Negative feedback and chargebacks count by order date, not the date received; one- and two-star feedback is negative; fraud chargebacks do not count; A-to-z claims count when granted and debited, refunded after filing, cancelled or pending, but not when Amazon pays, the claim is denied or withdrawn.
3. **Kaufland measures per item and per calendar period.** Its seller-related cancellation rate and trackable-shipment rate use the last completed month; on-time delivery and ticket response time use the last calendar week, and tickets must be answered within 48 hours. Order units still unshipped 14 days after the latest delivery date are cancelled by Kaufland after a further seven days' notice.
4. **Allegro scores points, not pass/fail rates.** Jakość mojej sprzedaży converts measures (recommendations, satisfaction, unresolved discussions, response times, quick refunds, on-time and fast shipping, rules) into points over rolling 28- or 30-day windows. Take the measure list, the points and the level boundaries from the panel, because Allegro has changed them before.

### Measure risk in orders and dates
5. **Slack = orders left before breach.** For a "under x%" target with N orders in the window, the breach count is the smallest k with k ÷ N ≥ x. A green 0.9% on 1 800 orders can have two orders of slack.
6. **Roll-off date = order date + window length.** A red 7-day metric can clear in a week without action; a 60-day defect stays for two months. Rank by date of breach and date of recovery, not by colour.
7. **Projected inflow.** Divide the forecast orders in the coming window by the current failure rate of the cause; if the cause is not fixed, the metric will be breached again as soon as the old orders roll off.
8. **Severity by consequence.** Rank breaches of metrics whose stated consequence is loss of selling privileges above category restrictions, and those above fees or ranking effects.

### Fix the cause, not the metric
9. **Read the failing orders.** Group late, cancelled, untracked and defective orders by warehouse, carrier, channel, SKU, weekday and order hour. A cause shared by three quarters of the failures is the fix.
10. **Typical causes and owners.** Late dispatch: handling time shorter than the warehouse's real cut-off, weekend or holiday pick-ups, confirmation uploaded late by the integration (warehouse, IT). Seller cancellations: overselling because stock syncs between channels too slowly or without a buffer (IT, stock owner). Invalid tracking: carrier name or code not mapped to the platform's list, numbers uploaded before the carrier scans (IT, carrier). Defects: non-delivery, items not as described, refunds issued late (customer service, catalog, carrier). Policy notices: listing or document fixes (catalog, compliance).
11. **One fix, one owner, one check date.** The check date is when enough new orders have entered the window to show the effect.

## Process
1. **Collect** each platform's export and targets; record export date and window per metric.
2. **Compute** value, numerator and denominator per metric, and reconcile with the dashboard; a mismatch usually means the wrong order subset.
3. **Slack and roll-off** per metric (rules 5–6); projected value at the next roll-off with current inflow (rule 7).
4. **Root cause**: pull the failing orders and group them (rule 9); name the cause with its count.
5. **Merge causes across platforms** where the same cause appears (overselling, one carrier, one warehouse).
6. **Rank** issues: breached with privilege risk, near breach with privilege risk, category or listing restrictions, scores and fees.
7. **Write** fix, owner, check date and the evidence line per issue.

## Pitfalls and edge cases
- **Small denominators.** At 150 orders a week, one cancellation moves the rate by 0.67 points; show counts, not only percentages.
- **FBA and FBM mixed.** Do not divide seller-fulfilled failures by all orders.
- **Buyer-requested cancellations** coded as seller cancellations by the integration inflate the cancellation rate; check how the cancellation was submitted.
- **Peak season** inflows can mask or create breaches; project with the forecast, not last month.
- **Several Amazon stores or countries** in one export: check the marketplace column of each failing order before fixing the wrong country's process.
- **Disputed defects.** A claim won on appeal or feedback withdrawn by the buyer leaves the metric; Amazon notes removal can take up to 48 hours. Recompute slack after it drops out, not before.

## Rules
- Targets come from the dashboard or the platform's policy page, with the date read. Nothing from memory.
- Read-only: the agent does not submit appeals or plans of action, contact buyers, change handling times, cancel or edit orders, or change listings or stock settings. It drafts; the owner acts.
- No personal data of buyers in the output; refer to orders by ID.
- Policy and appeal decisions are for the owner and, where needed, counsel.

## Output format
```
Account health triage — <seller> — exports <dates> — prepared <date>

| rank | platform | metric (window) | value | target | slack (orders) | roll-off / breach date | cause | fix | owner | check date |
|---|---|---|---|---|---|---|---|---|---|---|

Shared causes: <cause> → <metrics and platforms>
Watch: <metrics green but with slack ≤ 3 orders>
Missing data: <list>
```

## Worked example
Illustrative numbers only. Czech seller on Amazon.de (FBA and FBM), Kaufland.cz and Allegro.cz; export 2026-10-01.

```
| rank | platform | metric (window) | value | target | slack | roll-off / breach | cause | fix | owner | check |
|---|---|---|---|---|---|---|---|---|---|---|
| 1 | Amazon.de | Cancellation rate (7 d) | 5/150 = 3.33% | < 2.5% | breached, 2 over | clears 7 d after last cancel if no new ones | 4 of 5 oversold: stock sync every 60 min, no buffer | sync on order event, buffer 1 unit on last 3 units | IT | 2026-10-09 |
| 2 | Amazon.de | Late dispatch (10 d) | 12/210 = 5.71% | < 4% | breached, 4 over | 4 late orders must leave window | 9 of 12 ordered Fri after 14:00, no Sat pick-up | handling time +1 day on Fridays or Sat pick-up | warehouse | 2026-10-12 |
| 3 | Amazon.de | Valid tracking, Home & Kitchen (30 d) | 386/410 = 94.15% | > 95% | breached | 30 d | 21 of 24 with carrier name not mapped | map carrier code in integration | IT | 2026-10-31 |
| 4 | Amazon.de | Order Defect Rate (60 d) | 16/1 850 = 0.86% | < 1% | 2 orders | breach at 19 defects | 6 A-to-z "not received", same carrier | claim tracking proof, review carrier | CS | 2026-11-30 |
| 5 | Kaufland.cz | Seller cancellation (Sept) | 4/380 = 1.05% | < 1% (confirm on dashboard) | breached | next month | oversold, same sync | as rank 1 | IT | 2026-11-01 |
```

Checks: 5 ÷ 150 = 3.33%; at 150 orders the limit is 3 (3 ÷ 150 = 2.0%, 4 ÷ 150 = 2.67%), so 2 over. 12 ÷ 210 = 5.71%; under 4% allows at most 8 (8 ÷ 210 = 3.81%, 9 ÷ 210 = 4.29%), so 4 late orders must roll off. 386 ÷ 410 = 94.15%. 16 ÷ 1 850 = 0.86%; 18 ÷ 1 850 = 0.97% and 19 ÷ 1 850 = 1.03%, so slack is 2 orders. 4 ÷ 380 = 1.05%. The 30-day late dispatch (22 ÷ 640 = 3.44%) is green but will fail too if Friday orders keep shipping late. Shared cause: overselling drives ranks 1 and 5, so one sync fix serves both platforms.

## Quality checklist
- Every metric has its definition, window, order subset and target from the platform.
- Values reconcile with the dashboard (numerator and denominator shown).
- Slack in orders and roll-off or breach date given for every metric near or over target.
- Each cause is backed by a count of failing orders.
- Shared causes merged across platforms.
- Every fix has one owner and a check date.
- No buyer personal data; no actions taken on the accounts.

## Sources
- Amazon Seller Central Help, Order Performance policy under Account Health (EU; ODR, Cancellation Rate, Late Dispatch Rate, Valid Tracking Rate, On-Time Delivery Rate). Readable after sign-in to Seller Central; the targets above were checked against Amazon's EU programme policy text of August 2025.
- Amazon, Order Defect Rate (feedback, A-to-z claims and chargebacks that count): https://m.media-amazon.com/images/G/02/rainier/help/legal/Order_Defect_Rate_EN_101121.pdf
- Kaufland Global Marketplace, KPIs for your delivery performance: https://www.kauflandglobalmarketplace.com/en/seller-university/your-performance/service-performance/kpis-for-your-delivery-performance/
- Kaufland Global Marketplace, Customer service KPIs: https://www.kauflandglobalmarketplace.com/en/seller-university/your-performance/service-performance/customer-service-kpis/
- Allegro, Wprowadziliśmy zmiany w panelu Jakość mojej sprzedaży (2024-02-15): https://allegro.pl/pomoc/aktualnosci/wprowadzilismy-zmiany-w-panelu-jakosc-mojej-sprzedazy-1nPO4araMfW

## License
MIT
