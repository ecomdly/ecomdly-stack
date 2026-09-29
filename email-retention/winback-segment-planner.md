---
name: winback-segment-planner
owner: inboxcart
category: Email & retention
description: Segments lapsed customers by their own purchase cycle and net-of-returns RFM, plans a sequence and honest offer per segment with a holdout and break-even check, and sunsets unresponsive contacts.
version: v3
license: MIT
updated: 2026-09-29
recommended: false
security_checked: true
url: https://ecomdly.com/skills/inboxcart/winback-segment-planner
raw: https://ecomdly.com/raw/inboxcart/winback-segment-planner.md
install: npx @ecomdly/cli add inboxcart/winback-segment-planner
---

# Win-back segment planner

Produces a win-back plan: who is actually lapsed, how much they were worth, which sequence and offer each segment gets, who is suppressed, and when to stop mailing. "Everyone who hasn't bought in 90 days" mixes coffee buyers who are two months late with sofa buyers who are right on schedule, then sends both the same discount. This skill defines lapse per customer from the store's own purchase cycles, scores value with RFM net of returns, spends discounts only where a holdout can show they add orders, and sunsets the unresponsive to protect deliverability.

## When to use
- Repeat purchase rate is falling, or a large lapsed list gets one generic mail.
- Before a discount campaign, to decide who needs an offer and who does not.
- Deliverability is slipping and the store keeps mailing people who have not engaged for years.

## When not to use
- Customers with an open cart right now: `abandoned-cart-sequence` has priority.
- Fewer than roughly 200 customers with two or more orders: cycles and quintiles will be noise. Use a simple "last order > X days" rule set by the owner and say so.
- Measuring the campaign afterwards: use `ab-test-readout` for the holdout comparison.

## Inputs
Required:
- **Order export, at least 24 months**: customer ID, order ID, order date, net order value (after discounts, excl. VAT if possible), line items with category, returned/refunded amounts per order, order status. Shopify: Orders export; Shoptet: Objednávky export (CSV/XML); or the ESP's synced order data.
- **Consent per contact**: marketing consent flag, source, timestamp, and whether the address came from a purchase with an opt-out offered (soft opt-in). Unsubscribe and objection lists.
- **Offer policy**: which incentives exist, for whom, margin floor, end dates.
- **Voice guide.**

Optional:
- Gross margin by category (turns monetary value into contribution and enables the discount check).
- Engagement data: last click date, last site visit (opens are unreliable, see pitfalls).
- Support data: open tickets, disputes, chargebacks.
- Product usage data for consumables (pack size in days).

If returns data is missing, say monetary scores are gross and may over-rank heavy returners. If consent source is missing, the plan is `BLOCKED` for sending.

## Best practices

### Defining lapse
1. **Lapse is relative to the customer's cycle, not the calendar.** Expected cycle = the customer's own median inter-purchase gap if they have 3 or more orders; otherwise the median gap between consecutive orders in their main category (the category with most spend). For consumables with a known pack duration, the pack duration is a better prior than history.
2. **Use percentiles of the category's gap distribution for thresholds.** Lapsed when days since last order exceed the category's 75th-percentile gap; lost beyond the 90th. Fallback when fewer than 100 gaps exist in a category: lapsed > 1.5 × expected cycle, lost > 3 ×. Percentiles adapt to skewed distributions; fixed multipliers do not.
3. **Beware survivorship.** Gaps are measured only on customers who came back, so they understate how long lapsed customers stay away. That is fine for "is this customer late", not for predicting return probability.
4. **One-time buyers have no gap of their own.** Use the category median gap between first and second order; they form their own segment because their behaviour differs from repeaters.

### Scoring value
5. **RFM over customers with an order in the last 24 months.** Recency = days since last order divided by expected cycle (cycle-normalised, so a furniture buyer at 200 days is not "old"). Frequency = orders in 24 months. Monetary = net revenue after returns, or contribution if margins exist. Score each 1–5 by quintile; with heavy ties in frequency (most customers have 1 order), use fixed bands (1, 2, 3–4, 5–7, 8+) instead of quintiles.
6. **Net of returns and cancellations.** A customer who ordered EUR 2 000 and returned EUR 1 600 is not a champion.

### Legal basis and deliverability
7. **Only contacts with a valid marketing basis.** Marketing consent, or soft opt-in under ePrivacy Directive Art. 13(2): existing customers, contact obtained in the context of a sale, marketing of the store's own similar products, opt-out offered at collection and in every mail. Transactional history alone does not create consent. Soft opt-in does not stretch to unrelated product lines.
8. **Objection is final.** A customer who objected to direct marketing must not be processed for it (GDPR Art. 21(3)); keep them on a suppression list, do not delete the suppression record.
9. **Old consent is a risk, not a licence.** GDPR sets no fixed expiry, but consent must still reflect what the person expects; mailing someone after years of silence invites complaints. Check the national regulator's guidance and the store's retention policy for how long it keeps inactive marketing contacts.
10. **Protect the domain.** Gmail requires spam rates below 0.3% and recommends below 0.1% in Postmaster Tools; long-dormant addresses produce complaints and spam-trap hits. Send the lost segment last, in small batches, and stop if complaints rise.

### Offers
11. **Offer ladder: relevance before price.** Step 1: no offer (new arrivals, replenishment reminder, the product they bought). Step 2: non-price value that exists in policy (free shipping, early access, extended returns). Step 3: a discount, only for segments where it can pay back.
12. **Break-even check for discounts.** Incremental reactivations needed = (expected redemptions × average discount amount) ÷ contribution per reactivated order. Redemptions include people who would have bought anyway; that is why the discount must beat a holdout, not zero.
13. **Holdout per segment.** Keep 10% of each segment unmailed (or on the no-offer variant). Incremental reactivation = reactivation rate treated − reactivation rate holdout, same 60-day window. Without it, the plan cannot tell whether it worked.
14. **Honest offers only.** End dates and conditions come from the policy. Falsely claiming a limited-time offer to force a decision is blacklisted under the Unfair Commercial Practices Directive (Annex I point 7). Any "was/now" price must show the lowest price of the prior 30 days (Directive 98/6/EC Art. 6a).

### Sequencing and sunsetting
15. **2–3 mails per segment, 5–7 days apart**, stop on purchase (any channel), unsubscribe, bounce, or open support ticket.
16. **Measure engagement by clicks and visits, not opens.** Apple Mail Privacy Protection loads images automatically, so opens overstate engagement.
17. **Sunset explicitly.** Lost segment: one final mail that says plainly they will stop receiving marketing unless they click to stay, then move non-clickers to a sunset list excluded from campaigns. Re-admit only on a new purchase or a new opt-in.

## Process
1. **Clean orders**: drop cancelled and test orders, subtract refunds, merge duplicate customer IDs on normalised e-mail. Report how many were merged.
2. **Compute gaps and expected cycle** per customer and per category; publish the category table (median, P75, P90, number of gaps).
3. **Classify status**: active (inside cycle), lapsed, lost, one-time (split into "inside expected second-order window" and "past it").
4. **Score RFM** on the cycle-normalised recency, frequency bands and net monetary.
5. **Apply exclusions**: no marketing basis, objection/unsubscribe, open ticket, open dispute or chargeback, last order fully returned (route to support review), staff/test, B2B accounts unless separately handled.
6. **Assign segments**:
   - *Lapsed champions* (F ≥ 4 and M ≥ 4, lapsed): personal note from a real person, new arrivals in their category, early access if it exists; discount only as step 3 and only if the break-even check passes.
   - *Lapsed regulars* (F 2–3, lapsed): replenishment or "time for a new one" reminder with the product they bought, then the category's best seller.
   - *One-time buyers past window*: one-question survey ("what stopped you from ordering again?"), then one relevant second product.
   - *Lost* (beyond P90): one sunset mail, then the sunset list.
7. **Size each segment**: customers, last-24-month net revenue, contribution if available.
8. **Run the discount break-even** for any segment that gets one; show the numbers.
9. **Draft sequences** per segment with merge tags and the holdout split.
10. **Run the quality checklist** and list approvals needed.

## Pitfalls and edge cases
- **Seasonal buyers** (Christmas only, garden in spring): their gap is ~365 days; do not call them lapsed in July. Detect customers whose orders cluster in the same months and time their mail before the season.
- **Durable goods**: a sofa buyer is not lapsed, they are done. Cross-sell accessories or care products, or exclude.
- **Gift buyers**: the product category is not their taste; use the survey rather than "more of the same".
- **Guest checkouts with different e-mails** split one person into several customers; frequency is understated.
- **Customers lapsed because of a bad experience** (late delivery, defect in their last order): check support history first; an apology and fix beats a coupon.
- **Price-sensitive regulars who only buy on discount**: if their history is all discounted orders, a no-offer mail will look like failure; that is information, not a reason to discount everyone.
- **Small segments** (under ~100 customers) cannot support a meaningful holdout; say the result will be directional.

## Rules
- Read-only: produce segments, counts, sequences and recommendations; never upload lists, create segments in the ESP, generate codes or send.
- Never invent offers, deadlines or scarcity; offers come only from the policy input.
- Never contact customers without a recorded marketing basis; excluded counts are always reported.
- Customers with open disputes, chargebacks or fully returned last orders go to support, not marketing.
- Discounts, the sunset rule and any change to suppression lists need human sign-off.
- Missing inputs are flagged with their effect on the result, never estimated.

## Output format
```
WIN-BACK PLAN · {{ store_name }} · data <from>–<to> · customers scored <n>
Cycle table (days): category · median · P75 (lapsed) · P90 (lost) · gaps
Status: active <n> · lapsed <n> · lost <n> · one-time past window <n>

| Segment | Customers | Net rev. 24m | Contribution | Sequence | Offer | Holdout |

Discount check (per segment with an offer):
  redemptions <n> × avg discount <amount> = cost <amount>
  contribution per order <amount> → break-even incremental orders <n>
Excluded: no basis <n> · objected <n> · open ticket <n> · dispute/chargeback <n> · fully returned <n>
Sunset: <rule>, moves <n> contacts after final mail
Assumptions / missing data: <list>
Needs approval: <list>
```

## Worked example
Illustrative coffee and equipment shop, 24 months of data, 11 200 customers scored.

```
WIN-BACK PLAN · KávaDomů · data 2024-09-01–2026-08-31 · customers scored 11 200
Cycle table (days): coffee · median 34 · P75 52 · P90 95 · 6 410 gaps
                    equipment · median 210 · P75 330 · P90 540 · 380 gaps
Status: active 2 150 · lapsed 2 152 · lost 2 960 · one-time past window 3 757 · excluded 181

| Segment               | Customers | Net rev. 24m | Contribution | Sequence                          | Offer              | Holdout |
| Lapsed champions      | 312       | EUR 48 900   | EUR 17 100   | Note → new arrivals → early access| Early access only  | 10%     |
| Lapsed regulars       | 1 840     | EUR 61 300   | EUR 21 500   | Replenish → best seller           | None               | 10%     |
| One-time past window  | 3 757     | EUR 92 100   | EUR 29 500   | Survey → 2nd product → code       | 10% (policy A)     | 10%     |
| Lost                  | 2 960     | EUR 35 400   | EUR 11 300   | Sunset mail → sunset list         | None               | none    |

Discount check (One-time past window):
  redemptions 190 × avg discount EUR 3.10 = cost EUR 589
  contribution per order EUR 9.40 → break-even incremental orders 63
  Holdout must show ≥ 63 more orders than expected from 3 381 treated at holdout rate; otherwise drop the code.
Excluded: no basis 128 · objected 16 · open ticket 9 · dispute/chargeback 5 · fully returned 23
Sunset: no click in final mail within 14 days → sunset list, moves up to 2 960 contacts
Assumptions / missing data: margins by category from owner's estimate (38% coffee, 22% equipment); opens not used.
Needs approval: policy A code for one-time buyers; sunset of 2 960 contacts; lost segment sent in batches of 500.
```

## Quality checklist
- Lapse uses per-customer or per-category cycles; the cycle table with gap counts is shown.
- Monetary value is net of returns (or flagged as gross).
- Every sent contact has a recorded marketing basis; exclusions are counted by reason.
- Each segment has a holdout or a stated reason it cannot.
- Any discount has a break-even calculation and comes from policy with its real end date.
- Engagement rules use clicks/visits, not opens.
- Sunset rule is explicit; suppression records are kept, not deleted.
- Seasonal and durable-goods customers were checked before being called lapsed.
- Numbers add up: segment counts + active + excluded = customers scored.

## Sources
- Directive 2002/58/EC (ePrivacy), Art. 13: https://eur-lex.europa.eu/eli/dir/2002/58/oj
- Regulation (EU) 2016/679 (GDPR), Art. 21: https://eur-lex.europa.eu/eli/reg/2016/679/oj
- ICO, Electronic mail marketing rules (soft opt-in): https://ico.org.uk/for-organisations/direct-marketing-and-privacy-and-electronic-communications/guidance-on-direct-marketing-using-electronic-mail/how-do-we-comply-with-the-pecr-electronic-mail-marketing-rules/
- Directive 2005/29/EC (Unfair Commercial Practices), Annex I: https://eur-lex.europa.eu/eli/dir/2005/29/oj
- Directive (EU) 2019/2161, Art. 2 inserting Art. 6a into Directive 98/6/EC: https://eur-lex.europa.eu/eli/dir/2019/2161/oj
- Google, Email sender guidelines: https://support.google.com/a/answer/81126

## License
MIT
