---
name: winback-segment-planner
owner: inboxcart
category: Email & retention
description: Segments lapsed customers by RFM and their own purchase cycle, then plans a win-back sequence per segment with honest offers taken from the store's real policy.
version: v2
license: MIT
updated: 2026-09-24
recommended: false
security_checked: true
url: https://ecomdly.com/skills/inboxcart/winback-segment-planner
raw: https://ecomdly.com/raw/inboxcart/winback-segment-planner.md
install: npx @ecomdly/cli add inboxcart/winback-segment-planner
---

# Win-back segment planner

"Everyone who hasn't bought in 90 days" mixes coffee buyers who are two months late with sofa buyers who are right on schedule. This skill defines lapse per customer, groups them by value, and plans a sequence each group deserves.

## When to use
- Repeat purchase rate is falling, or the lapsed list is large and gets one generic mail.
- Before a discount campaign, to decide who needs an offer and who does not.

## Input
Order export: customer id, order date, order value, product categories, returns. Marketing consent status. The store's real offer policy (what discounts exist, margin limits) and the voice guide.

## Method
1. **Purchase cycle.** Per category, the median days between first and second order. Customers with one order use their category's median.
2. **Lapse.** A customer is lapsed when days since last order > 1.5 × their expected cycle; lost at > 3 ×. Customers inside their cycle are excluded, whatever the calendar says.
3. **RFM.** Score recency, frequency and monetary value 1–5 by quintile over customers with at least one order in the last 24 months.
4. **Segments.**
   - *Lapsed champions* (F ≥ 4, M ≥ 4): personal note, new arrivals in their category, early access if it exists. Discount only if the policy allows and as a last step.
   - *Lapsed regulars* (F 2–3): replenishment reminder with the product they bought, then the category's best seller.
   - *One-time buyers*: ask what went wrong (one-question survey), then a relevant second product.
   - *Lost* (> 3 × cycle): one final mail with a clear opt-out, then move to a sunset list to protect deliverability.
5. **Sequence.** 2–3 mails per segment, 5–7 days apart; stop on purchase, unsubscribe or open support ticket.

## Rules
- Only customers with valid marketing consent. Transactional data does not imply consent.
- Offers and end dates only from the policy in the input. Never invent scarcity or deadlines.
- Exclude customers whose last order was fully returned or disputed; flag them for support instead.
- Report segment sizes and the revenue they represented, so the effort matches the value.

## Output format
```
Cycle (median): coffee 34 d · equipment 210 d
| Segment            | Customers | Last-12m revenue | Sequence                        | Offer            |
| Lapsed champions   | 312       | EUR 48 900       | Note → new arrivals → survey    | Early access     |
| Lapsed regulars    | 1 840     | EUR 61 300       | Replenish → best seller         | None             |
| One-time buyers    | 4 205     | EUR 92 100       | Survey → 2nd product            | 10% (policy A)   |
| Lost               | 2 960     | EUR 35 400       | Final mail → sunset list        | None             |
Excluded: 184 no consent · 37 open disputes
```

## License
MIT
