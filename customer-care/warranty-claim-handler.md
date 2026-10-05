---
name: warranty-claim-handler
owner: helpdeskly
category: Customer care
description: Triages consumer claims for faulty goods under the EU legal guarantee (2019/771) with country-correct deadlines and burden of proof, then drafts the acknowledgement, evidence request and decision letter for the owner to approve.
version: v1
license: MIT
updated: 2026-10-05
recommended: false
security_checked: true
url: https://ecomdly.com/skills/helpdeskly/warranty-claim-handler
raw: https://ecomdly.com/raw/helpdeskly/warranty-claim-handler.md
install: npx @ecomdly/cli add helpdeskly/warranty-claim-handler
---

# Warranty claim handler

Produces, for each consumer complaint about faulty goods, a claim card (channel, delivery date, liability and presumption windows, statutory handling deadline, remedy requested) and three drafts for a human to approve: the acknowledgement, the request for evidence and the decision letter. The naive approach treats the claim like a store policy: "warranty is 12 months", "send it back in the original box at your cost", "we cannot prove it was faulty on delivery, so rejected". Each of those breaks the legal guarantee, which is the seller's statutory liability for lack of conformity under Directive (EU) 2019/771, separate from any commercial warranty and from the 14-day withdrawal. **The key insight: most claims are decided by dates before anyone opens the parcel.** The delivery date fixes whether the seller is liable at all and who must prove what; the claim date starts a national handling clock that, once missed, hands the consumer stronger remedies. The agent gets the dates and the evidence right; the owner makes the decision.

## When to use
- A customer reports that a product stopped working, broke, or does not match its description weeks or months after delivery.
- A claim arrives through email, the contact form, a marketplace message or a public review and needs a compliant first answer.
- An inspection or service report has come back and the decision letter must be written.
- Weekly, to list open claims by days left on the statutory clock.

## When not to use
- The customer wants to return goods that work (change of mind, wrong size within 14 days): that is a withdrawal. Reply with `order-status-reply-drafter`; analyse patterns with `returns-reason-analyzer`.
- Monthly defect rates by SKU, supplier or batch: `returns-reason-analyzer`.
- Parcel not yet delivered, or lost: `order-status-reply-drafter`.
- The public side of a review that mentions a defect: `review-response-drafter` drafts the reply; this skill handles the claim itself.
- Marketplace orders where the marketplace runs the claim process and it affects account metrics: `marketplace-account-health-triage`.
- Business buyers (B2B): consumer rules do not apply; ask the owner for the contract terms.

## Inputs

| input | definition | typical source |
|---|---|---|
| Claim message | customer's own words, channel, timestamp of receipt | mailbox, contact form, Shoptet Reklamace module, helpdesk |
| Order record | order ID, buyer type (consumer or business), SKU, price paid, shipping country | store admin export |
| Delivery date | date the consumer took physical possession, not the dispatch date | carrier proof of delivery |
| Country rules | liability period, presumption period, notification duty, handling deadline, required confirmations, deadline counting rule | this skill for CZ and SK; the store's counsel for other countries |
| Claim policy | who decides, return address, pickup carrier, internal buffer before the statutory deadline, ADR body named in the store's terms | owner, terms and conditions |
| Inspection result | technician or supplier report with photos, if already available | service partner, warehouse |

Optional: earlier claims and repairs on the same order; commercial guarantee terms (záruční list) if the store or producer gives one; serial number and batch; the customer's photos or video.

Missing input → stop and ask. Never assume a delivery date, a country rule or a deadline from an "industry standard".

## Best practices

### Classify the channel first
1. **Legal guarantee, withdrawal, transit damage and commercial warranty are four different things.** Withdrawal is 14 days without giving a reason (2011/83/EU Art. 9); the legal guarantee covers lack of conformity at delivery (2019/771 Art. 10); transit damage is the trader's risk until the consumer or their nominee, other than the carrier, takes physical possession (2011/83/EU Art. 20); a commercial guarantee is an extra promise on top (2019/771 Art. 17). Mixing them gives the customer the wrong rights and the owner the wrong cost line.
2. **Transit damage is the store's problem with its carrier.** Never send the consumer to the carrier when the store chose the carrier; record it as transit damage and route to operations.
3. **Inside 14 days, ask which right the customer uses.** A defective item within the withdrawal period can be either; the drafts must not choose for them.
4. **A commercial warranty never shortens the legal guarantee.** Its statement must say the consumer keeps the free statutory remedies (2019/771 Art. 17(2)(a); CZ OZ § 2174a). Never answer "your warranty expired" when the legal period is still running.

### Dates decide liability and burden of proof
5. **Liability period: a defect present at delivery that becomes apparent within two years of delivery** (Art. 10(1)); member states may set longer periods (Art. 10(3)). CZ: two years from taking over (OZ § 2165(1)). SK: two years from delivery, extended once by 12 months after the first repair (OZ § 619(1), (4)).
6. **Presumption: one year in the Directive, two allowed.** A defect that becomes apparent within one year of delivery is presumed to have existed at delivery unless proved otherwise or incompatible with the nature of the goods or defect (Art. 11(1)); states may use two years (Art. 11(2)). CZ uses one year (OZ § 2161(5)). SK applies the presumption to the whole liability period (OZ § 620(1)). Inside the window, "you cannot prove it was faulty at delivery" is not a reason; only evidence of another cause is.
7. **Late notification is a national rule, not a default.** The Directive allows states to require notice within at least two months of discovery (Art. 12). SK requires it (OZ § 621(3)). CZ courts grant the right even without notice "bez zbytečného odkladu" (OZ § 2165(3)), so never reject a Czech claim for lateness inside the period.
8. **Wear and misuse are not defects, but the store proves them.** CZ excludes wear from normal use and defects the buyer caused (OZ § 2167); the Directive leaves consumer contribution to national law (Art. 13(7)). A rejection needs an inspection finding, not a guess.

### Remedy hierarchy
9. **Repair or replacement first, at the consumer's choice,** unless the chosen one is impossible or disproportionate compared with the other (Art. 13(2)); the seller may refuse both only if both are impossible or disproportionate (Art. 13(3)).
10. **Price reduction or termination only in the Art. 13(4) cases:** remedy not completed or refused, defect reappears, defect serious enough to justify it immediately, or the seller will clearly not fix it in reasonable time. No termination for a minor defect, with the burden on the seller (Art. 13(5)); CZ presumes a defect is not insignificant (OZ § 2171(3)). A reduction is proportionate to the drop in value (Art. 15).
11. **Free of charge, reasonable time, no significant inconvenience** (Art. 14(1)); the seller takes back replaced goods at its expense (Art. 14(2)), and in CZ and SK bears the cost of taking the item over (OZ § 2170(2); SK OZ § 623(7)). Never ask the consumer to pay postage for a claim.
12. **On termination, refund upon receipt of the goods or proof they were sent** (Art. 16(3)); CZ: without undue delay (OZ § 2171(4)).

### National procedure (verified for CZ and SK only)
13. **Czechia (zákon o ochraně spotřebitele § 19).** At the claim, a written confirmation with the date, content, requested remedy and contact details (§ 19(2)); the claim, including removal of the defect, handled and the consumer informed within 30 days unless a longer period is agreed (§ 19(3)); after that the consumer may withdraw or demand a reasonable discount (§ 19(4)); a confirmation of date and method of handling, or a written reasoning for a rejection (§ 19(5)).
14. **Slovakia (Občiansky zákonník §§ 619–624).** Written confirmation without delay stating the period for removal, at most 30 days from notification unless an objective reason outside the seller's control justifies longer, with the burden on the seller (§ 622(3), § 623(4)); before removing the defect, inform the buyer of the repair-or-replacement choice and the 12-month extension (§ 623(2)); reasons for a refusal in writing (§ 622(4)).
15. **Counting:** in both countries a period in days starts the day after the event, and a last day on a weekend or holiday moves to the next working day (CZ OZ § 605(1), § 607; SK OZ § 122(1), (3)). Plan to calendar day 30 minus the owner's buffer; do not rely on the shift.
16. **Every other country is an input.** Do not apply Czech or Slovak rules to a German or Polish customer.

## Process
1. **Read the claim and the order.** Confirm buyer type and shipping country; get the delivery date from carrier proof.
2. **Classify the channel** (rules 1–3). If withdrawal or transit damage, stop and route; if unclear inside 14 days, draft the clarifying question.
3. **Compute the dates:** days since delivery, liability end, presumption end, claim date, statutory deadline, internal target (deadline minus buffer).
4. **Record the requested remedy** in the customer's words; if none, the acknowledgement asks for it.
5. **Draft the acknowledgement** with the country's confirmation fields, pickup or return instructions at the store's cost, and no statement on liability.
6. **Draft the evidence request** only for what decides the case: description, when it appeared, photos or video, serial number. The order record already proves the purchase.
7. **After inspection, draft the decision letter** for the owner's chosen outcome: remedy and date, reduction with the calculation, termination and refund timing, or rejection with the inspection finding and the ADR body from the store's terms.
8. **Hand the claim card and drafts to the owner** and add the claim to the open-claims list sorted by days left.
9. **Run the quality checklist.**

## Pitfalls and edge cases
- **Dispatch date used instead of delivery date** shortens every window by the transit time; use carrier proof.
- **"No fault found"**: the device may fail intermittently. Inside the presumption window this is not proof of misuse; ask the owner whether to retest, replace or reject with the report attached.
- **Repeat claim on the same item**: a defect reappearing after repair opens price reduction or termination (Art. 13(4)(b)); check earlier claims.
- **Second-hand goods**: CZ allows agreeing a liability period down to one year (OZ § 2168), SK down to one year (OZ § 619(3)); only if the order shows such an agreement.
- **Goods with digital elements and installed goods** carry extra rules (Art. 10(2), Art. 14(3)); flag for the owner.
- **Marketplace orders** may run under the marketplace's claim flow; keep its deadline and the statutory one, use the shorter.
- **Personal data**: photos may show homes or faces; store them with the claim, never in reports.

## Rules
- Draft only. The agent never sends a message, accepts or rejects a claim, promises a refund, books a pickup or issues a credit without the owner's explicit decision.
- Never state a deadline, presumption or period that is not verified above or supplied by counsel for that country.
- Never ask the consumer to pay for postage, inspection or repair of a claim within the legal guarantee.
- Never present a commercial warranty as replacing the legal guarantee.
- A rejection draft always contains the evidence it relies on; without inspection evidence, there is no rejection draft.

## Output format
```
CLAIM CARD · <claim ID> · order <ID> · <country> · consumer: yes/no
Channel: legal guarantee | withdrawal | transit damage | commercial warranty | unclear (question drafted)
Product: <SKU, name> · price paid <amount>
Delivered: <date> (source: <carrier proof>) · claim received: <date> · day <n> since delivery
Liability until: <date> · presumption until: <date> → burden on <seller|consumer>
Notification rule: <none | within 2 months of discovery: discovered <date>, met yes/no>
Statutory deadline: <date> (<rule>) · internal target: <date>
Requested remedy: <repair|replacement|reduction|termination|not stated>
Evidence held: <list> · missing: <list>
Owner decision needed: <question>

DRAFT 1 · Acknowledgement
DRAFT 2 · Evidence request
DRAFT 3 · Decision letter (<outcome chosen by owner>)
```

## Worked example
Illustrative dates and amounts. Czech store; two claims received.

```
CLAIM CARD · RK-2026-118 · order 40712 · CZ · consumer: yes
Channel: legal guarantee
Product: espresso machine EM-2 · price paid CZK 8 990
Delivered: 2026-02-16 (carrier proof) · claim received: 2026-10-05 · day 231 since delivery
Liability until: 2028-02-16 · presumption until: 2027-02-16 → burden on seller
Notification rule: none (CZ OZ § 2165(3))
Statutory deadline: 2026-11-04 (ZOS § 19(3)) · internal target: 2026-10-30 (buffer 5 days)
Requested remedy: replacement
Evidence held: customer video of pump not priming · missing: serial number
Owner decision needed: replace from stock, or repair if the supplier confirms it is faster for the customer?

CLAIM CARD · RK-2026-119 · order 38120 · SK · consumer: yes
Channel: legal guarantee
Product: kettle KT-1 · price paid EUR 49.00
Delivered: 2025-01-20 · defect discovered: 2026-09-15 · claim received: 2026-09-29 · day 617 since delivery
Liability until: 2027-01-20 · presumption until: 2027-01-20 (SK OZ § 620(1)) → burden on seller
Notification rule: within 2 months of discovery: met (14 days)
Statutory deadline: removal within 30 days → 2026-10-29 (SK OZ § 622(3), § 623(4))
Requested remedy: not stated → acknowledgement asks, and states the repair/replacement choice and 12-month extension
```

Check: 2026-02-16 to 2026-10-05 is 12 + 31 + 30 + 31 + 30 + 31 + 31 + 30 + 5 = 231 days, inside one year. Day 1 of the CZ deadline is 2026-10-06; 26 days to 2026-10-31 plus 4 days gives 2026-11-04, a Wednesday. For SK, 2026-09-29 + 30 days = 2026-10-29, a Thursday. The kettle is 617 days old: in Czechia it would be outside the one-year presumption, in Slovakia the seller still carries the burden.

## Quality checklist
- Channel classified; withdrawals and transit damage routed away, not decided here.
- Delivery date from carrier proof; all windows computed from it.
- Country rules come from this skill (CZ, SK) or counsel; no Czech rule applied abroad.
- Acknowledgement contains every confirmation field the country requires.
- No cost to the consumer in any draft; no "warranty expired" inside the legal period.
- Rejection drafts cite inspection evidence; none exists without it.
- Every draft is marked for owner approval; nothing was sent.

## Sources
- EUR-Lex, Directive (EU) 2019/771 on contracts for the sale of goods, Art. 10–17: https://eur-lex.europa.eu/eli/dir/2019/771/oj
- EUR-Lex, Directive 2011/83/EU on consumer rights, Art. 9 and 20: https://eur-lex.europa.eu/eli/dir/2011/83/oj
- Zákony pro lidi, zákon č. 634/1992 Sb., o ochraně spotřebitele, § 19: https://www.zakonyprolidi.cz/cs/1992-634
- Zákony pro lidi, zákon č. 89/2012 Sb., občanský zákoník, § 605, 607, 2161, 2165, 2167–2171, 2174a: https://www.zakonyprolidi.cz/cs/2012-89
- Zákony pre ľudí, zákon č. 40/1964 Zb., Občiansky zákonník, § 122, 619–624: https://www.zakonypreludi.sk/zz/1964-40

## License
MIT
