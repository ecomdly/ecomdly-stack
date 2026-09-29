---
name: order-status-reply-drafter
owner: helpdeskly
category: Customer care
description: Drafts where-is-my-order replies from order and carrier data, with per-parcel status, only carrier-backed dates, and accurate EU delivery, withdrawal and warranty statements for a human agent to send.
version: v3
license: MIT
updated: 2026-09-29
recommended: false
security_checked: false
url: https://ecomdly.com/skills/helpdeskly/order-status-reply-drafter
raw: https://ecomdly.com/raw/helpdeskly/order-status-reply-drafter.md
install: npx @ecomdly/cli add helpdeskly/order-status-reply-drafter
---

# Order status reply drafter

Drafts replies to "where is my order" tickets from the order system and carrier tracking, in the customer's language, and says so plainly when the answer is "late". The naive reply fails by guessing: "it should arrive tomorrow" when the carrier gave no estimate, "please contact the carrier" when the parcel is legally still the store's responsibility, or a vague "we'll look into it" that generates a second ticket. Every fact in the reply here is traceable to a data field, dates are quoted only when the carrier or the order record supports them, and the customer's statutory options (delivery deadline, withdrawal, conformity) are stated accurately or not at all.

## When to use
- Incoming tickets about order status, delays, split shipments, "tracking says delivered but I have nothing", cancellations of undelivered orders.
- In bulk, when a carrier incident or a warehouse backlog delays many parcels at once.

## When not to use
- Tickets about a product defect or complaint on a delivered item beyond a first acknowledgement: route to the warranty/claims process (a short reply template is included below for triage only).
- Review requests or marketing follow-ups: see `post-purchase-review-request`.
- Analysing why returns happen: see `returns-reason-analyzer`.

## Inputs
Required:
- **Ticket**: text, sender e-mail, language, channel.
- **Order record**: number, date and time of order, items and quantities, payment method and status, shipping method (standard or paid express), delivery promise shown at checkout (dispatch window or delivery date), shipping address country.
- **Fulfilment per item**: picked / packed / dispatched, parcel ID, carrier, label-created time.
- **Carrier tracking events** per parcel with timestamps and time zone, and any carrier delivery estimate.
- **Store policies**: delays, cancellations, returns (including any period longer than the legal 14 days), who pays return shipping, and the location of the online withdrawal function.

Optional:
- Carrier incident notices (depot closure, strike, weather).
- Proof of delivery (signature, photo, pickup-point collection record).
- Previous tickets on the same order.

If the sender's e-mail does not match the order's e-mail, do not disclose order details; draft an identity-check reply. If tracking data is missing, say the status is being checked with the carrier and route the ticket to an agent.

## Best practices

### Reading the data
1. **Latest event and its age decide the status.** Normalise carrier events to: label created → collected → in transit → out for delivery → delivered / ready at pickup point → collected by customer; plus exception states (address problem, delivery failed, returned to sender, damaged). Convert timestamps to the customer's time zone.
2. **Label without movement.** No carrier scan 48 hours (working days) after the label was created means "not yet collected" — the parcel is still with the store, not the carrier. Say so and escalate internally; do not blame the carrier.
3. **Late is defined against the promise.** Late = not dispatched within the promised window, or carrier estimate past the promised delivery date. Without a stated promise, the legal default applies: delivery without undue delay and not later than 30 days after the contract was concluded (Directive 2011/83/EU Art. 18(1)).
4. **Split shipments**: report each parcel separately with its own status; never average them into "partially shipped".
5. **Payment-dependent orders**: unpaid bank transfers are not dispatched; say what is missing (payment not yet received) rather than "processing".

### Dates and promises
6. **Quote only dates the data supports.** Give a delivery date only when the carrier provides an estimate, and attribute it ("the carrier estimates Monday 28 September"). Never add a buffer-less guess of your own, never say "tomorrow" on day-of-week logic.
7. **Use absolute dates** with the weekday, in the customer's language format.
8. **When late, say it in the first sentence**, then the reason if known from the data, then the next concrete step and when the customer will hear back.

### Statutory rights (EU consumers; check national law for additions)
9. **Missed delivery deadline** (Art. 18(2)): if the agreed time or the 30-day limit passes, the consumer may set an additional appropriate period; if the store misses that too, the consumer may terminate. If the customer told the store before ordering that the date was essential, or the store refuses to deliver, they may terminate immediately. On termination the store reimburses all sums without undue delay (Art. 18(3)). Do not argue that the customer must wait longer than the law requires.
10. **Risk stays with the store until the customer has the goods** (Art. 20): loss or damage in transit is the store's problem, not the customer's. "Delivered but not received" is therefore handled by the store with the carrier; do not tell the customer to file the carrier claim themselves.
11. **Right of withdrawal** (Art. 9, 13, 14):
    - 14 days, no reason needed, counted from the day the customer (or a third party they named, other than the carrier) takes physical possession; for several items in one order delivered separately, from receipt of the **last** item. The customer may also withdraw before delivery.
    - If the store did not inform the customer of the right of withdrawal, the period extends by up to 12 months (Art. 10).
    - The store refunds all payments, including the standard delivery cost, without undue delay and within 14 days of being informed; it need not refund the extra cost of a more expensive delivery the customer chose (Art. 13(2)). Same payment method unless agreed otherwise.
    - The store may withhold the refund until it receives the goods or proof they were sent back, whichever is earlier (Art. 13(3)).
    - The customer sends the goods back within 14 days of withdrawing and pays the direct return cost only if the store said so before purchase (Art. 14(1)); they are liable only for loss of value from handling beyond what is needed to establish the nature, characteristics and functioning (Art. 14(2)).
    - For contracts concluded through an online interface, the store must also provide an online withdrawal function labelled "withdraw from contract here" or equivalent (Art. 11a, inserted by Directive (EU) 2023/2673, applicable from 19 June 2026). Point to it if the store has it; if the store has none, flag this internally.
12. **Exceptions** (Art. 16) only if the store's policy lists them and they apply to this item: goods made to the consumer's specifications or clearly personalised; goods liable to deteriorate or expire rapidly; sealed goods not suitable for return for health or hygiene reasons once unsealed after delivery; sealed audio, video or software unsealed after delivery. An opened but unsealed-by-design item is not an exception.
13. **Faulty or not as described** is a conformity claim, not a withdrawal (Directive (EU) 2019/771): the seller is liable for at least two years from delivery, defects appearing in the first year are presumed to have existed at delivery (two years in some countries), and repair or replacement must be free of charge and within a reasonable time. In a status reply, acknowledge, ask for photos and route to the claims process; do not decide the claim.

### Tone and content
14. **No blame on the customer**, no invented goodwill vouchers, no apology spiral. One apology when the store is late, then facts.
15. **One reply answers all open questions** in the ticket, in the ticket's language and the store's form of address (for example formal "vy" in Czech unless the voice guide says otherwise).
16. **Do not disclose more than needed**: no full address, no other customers' data, no internal notes.

## Process
1. **Verify identity**: sender e-mail matches order e-mail (or the ticket comes from the logged-in account). If not, draft an identity-check reply and stop.
2. **Classify the question**: status, late, delivered-not-received, damaged on arrival, cancel before dispatch, cancel after dispatch, withdrawal after delivery, address change, other.
3. **Build the per-parcel timeline** and determine status per rules 1–5.
4. **Compare with the promise** and set `status` = on time / late / not yet collected / delivered / exception / unknown.
5. **Pick the reply pattern**:
   - On time: status + carrier estimate (if any) + tracking link.
   - Late: say late first + reason from data + next step + when they hear back; if past the 30-day or agreed deadline, state that they may cancel and receive a refund.
   - Not yet collected: say it has not left the store yet + internal escalation + when they hear back.
   - Delivered, not received: ask to check pickup point, neighbours, household, mailbox, with the delivery time and location from the carrier; open a carrier investigation on the store's side; say the store will resolve it.
   - Damaged on arrival: ask for photos of box and item; route to claims; state it is the store's responsibility.
   - Cancel before dispatch: follow store policy; if the order has shipped, explain the withdrawal route instead.
   - Withdrawal after delivery: confirm, point to the withdrawal function or form, state return steps and refund timing per rule 11.
6. **Draft** in the customer's language with absolute dates and attributed estimates.
7. **Set flags** for the human agent: escalate, refund decision needed, carrier investigation opened, policy exception, missing data.
8. **Bulk mode**: group affected orders by carrier incident; draft one template with per-order merge fields (order number, parcel ID, last event, carrier estimate); never promise a common date the tracking does not show for each parcel.

## Pitfalls and edge cases
- **"Delivered" to a pickup point** is not collected; the withdrawal clock starts when the customer collects.
- **Tracking time zones**: a 23:30 UTC event may be the next day locally.
- **Carrier "estimated delivery" that has already passed**: say the carrier's estimate has passed and the store has asked the carrier for an update; do not repeat the stale date.
- **Returned to sender** (address problem, not collected): check whether the customer refused it (possible withdrawal) or never had the chance; propose reshipment or refund per policy.
- **Cash on delivery refused**: not a withdrawal by itself; policy decides whether costs are claimed; flag for a human.
- **Address change after dispatch**: only the carrier can redirect; do not promise it.
- **Partial cancellation of a split order**: refund covers the cancelled items and, if nothing else remains to ship, the proportionate standard delivery cost; flag for a human to compute.
- **Non-EU customers or B2B orders**: consumer-law statements above do not apply automatically; use store policy only.
- **Pre-orders**: the promise is the pre-order date on the order, not the standard dispatch window.

## Rules
- Draft only; a human agent sends. Never trigger refunds, cancellations, carrier claims or vouchers.
- Every fact comes from the ticket, order, fulfilment or tracking data; unknowns are stated as "being checked" and flagged.
- Never quote a delivery date the carrier or order record does not provide; attribute every estimate.
- Legal statements are limited to the rules above; anything else country-specific is flagged for the agent.
- No disclosure to a sender whose identity does not match the order.
- No blame on the customer; no invented compensation.

## Output format
```
Order <number> | status: <on time / late / not yet collected / delivered / exception / unknown> | <parcel summary>
Timeline: <parcel> <last event> <date time TZ> · promise <date/window> · carrier estimate <date or none>

<Reply in customer's language>

flags: <none | escalate · refund decision · carrier investigation · policy exception · missing data: ...>
facts used: <field → value, one per line>
```

## Worked example
Illustrative: Czech store, order placed 18 September 2026, promise "dispatch within 2 working days", two parcels. Customer Petra writes on 25 September: "Where is the rest of my order? I only got the shoes."

```
Order 20461 | status: late | parcel 1 of 2 delivered, parcel 2 in transit
Timeline: P1 (Zásilkovna Z 4410 2287 80) delivered 23 Sep 2026 14:12 CEST
          P2 (Zásilkovna Z 4410 2287 91) label 22 Sep, collected 24 Sep 09:40 CEST, in transit
          promise: dispatch by 22 Sep (2 working days) · P2 dispatched 24 Sep → late
          carrier estimate P2: 28 Sep 2026

Hello Petra,
the second part of your order is running late, sorry about that. The shoes were
delivered on Wednesday 23 September. The second parcel with the jacket left our
warehouse on Thursday 24 September, two days later than we promised, and the carrier
(Zásilkovna) estimates delivery on Monday 28 September. Tracking: Z 4410 2287 91.
If you would rather not wait, reply and we will cancel the jacket and refund it.
Your 14 days to return anything without giving a reason start when you receive
the jacket, the last item of the order.

flags: none
facts used:
  order_date → 2026-09-18
  promise → dispatch within 2 working days (by 2026-09-22)
  P1 last event → delivered 2026-09-23 14:12 CEST
  P2 collected → 2026-09-24 09:40 CEST
  P2 carrier estimate → 2026-09-28
```


## Quality checklist
- Identity matched before any order detail is disclosed.
- Status is derived from the latest carrier event and the stated promise; each parcel is reported separately.
- Every date in the reply is in "facts used"; every estimate is attributed to the carrier.
- Weekdays checked against a calendar.
- If late, the first sentence says so; the next step and follow-up time are stated.
- Statutory statements match rules 9–13 exactly (withdrawal from the last item, standard delivery refunded, withholding rule, risk until possession); no country-specific claim without a flag.
- No blame, no invented voucher, no promise to redirect or deliver on a date the data does not show.
- Flags tell the agent exactly what decision is theirs.

## Sources
- Directive 2011/83/EU (Consumer Rights), Art. 9, 10, 13, 14, 16, 18, 20: https://eur-lex.europa.eu/eli/dir/2011/83/oj
- Directive (EU) 2023/2673, inserting Art. 11a (withdrawal function) into Directive 2011/83/EU: https://eur-lex.europa.eu/eli/dir/2023/2673/oj
- Directive (EU) 2019/771 (Sale of Goods), Art. 10, 11, 13, 14: https://eur-lex.europa.eu/eli/dir/2019/771/oj
- Your Europe, Returns: https://europa.eu/youreurope/citizens/consumers/shopping/returns/index_en.htm
- Your Europe, Shipping and delivery: https://europa.eu/youreurope/citizens/consumers/shopping/shipping-delivery/index_en.htm
- Your Europe, Guarantees and returns: https://europa.eu/youreurope/citizens/consumers/shopping/guarantees-returns/index_en.htm

## License
MIT
