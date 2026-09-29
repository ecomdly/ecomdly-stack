---
name: order-status-reply-drafter
owner: helpdeskly
category: Customer care
description: Drafts "where is my order" replies from real order and carrier tracking data, states delays plainly, and never promises a delivery date the data does not support.
version: v2
license: MIT
updated: 2026-09-08
recommended: false
security_checked: true
url: https://ecomdly.com/skills/helpdeskly/order-status-reply-drafter
raw: https://ecomdly.com/raw/helpdeskly/order-status-reply-drafter.md
install: npx @ecomdly/cli add helpdeskly/order-status-reply-drafter
---

# Order status reply drafter

Most support tickets ask one question: where is it. This skill answers it from the order system and the carrier, in the customer's language, and says so honestly when the answer is "late".

## When to use
- On incoming tickets about order status, delays, split shipments or cancellations.
- In bulk, when a carrier incident delays many parcels at once.

## Input
The ticket text, the order record (number, date, items, payment status, shipping method, promised dispatch window), the fulfilment status per item, and the carrier tracking events with timestamps. The store's policies on delays, cancellations and returns.

## Reading the data
1. Find the latest carrier event and its age. No event for 48 hours after the label was created counts as "not yet collected".
2. Compare with the promised window. Late = dispatched after the window, or delivery estimate past the promise.
3. Split shipments: report each parcel separately.

## Reply rules
- Every fact in the reply comes from the data. If tracking is missing, say the parcel status is being checked with the carrier, and send the ticket to an agent.
- Give a delivery estimate only if the carrier provides one; quote it as the carrier's estimate.
- When late, say so in the first sentence, give the reason if known, and the next step.
- If the customer wants to cancel an undelivered order, state the options the store policy allows. For a delivered order from an EU consumer, state the right of withdrawal: 14 days from receipt of the goods, no reason needed, refund of the payment including the standard delivery cost within 14 days of the withdrawal notice. The store may wait with the refund until the goods are back or proof of sending arrives.
- Exceptions (custom-made goods, opened hygiene-sealed items) only if the policy lists them and they apply.
- No blame on the customer, no invented goodwill vouchers.

## Output format
```
Order 20461 | status: late | parcel 1 of 2 delivered, parcel 2 in transit

Hello Petra,
your order is running late, sorry about that. The first parcel was delivered on 23 September. The second left our warehouse on 24 September and the carrier (Zásilkovna) estimates delivery on 27 September. Tracking: Z 4410 2287 91.
If you would rather cancel the second parcel, reply and we will refund it.

flags: none
```

## License
MIT
