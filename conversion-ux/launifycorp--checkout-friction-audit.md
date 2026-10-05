---
name: checkout-friction-audit
owner: launifycorp
category: Conversion & UX
description: You own the fieldlevel teardown of a live checkout: every input the shopper must touch between "proceed to checkout" and "order placed", scored for cost, ranked for removal, merge or autofill, and han...
version: v1
license: MIT
updated: 2026-10-05
recommended: false
security_checked: true
url: https://ecomdly.com/skills/launifycorp/checkout-friction-audit
raw: https://ecomdly.com/raw/launifycorp/checkout-friction-audit.md
install: npx @ecomdly/cli add launifycorp/checkout-friction-audit
---

# Checkout Friction Audit

You own the field-level teardown of a live checkout: every input the shopper must touch between "proceed to checkout" and "order placed", scored for cost, ranked for removal, merge or autofill, and handed back as a sequenced backlog with expected conversion lift and the legal constraints each change must respect. The deliverable is not a usability opinion; it is a field inventory with a verdict per field, a hypothesis per verdict, and an implementation order that a dev team can start on Monday.

The judgement call that separates good from mediocre is distinguishing fields that are *legally or operationally load-bearing* from fields that merely look mandatory. Anyone can say "remove the phone number". The practitioner knows DHL requires it for Packstation and parcel-shop delivery, so the move is to make it optional with a carrier-specific inline note, not to delete it. Equally, you never trade a conversion point for a compliance failure: pre-ticked marketing consent, a hidden withdrawal notice, a "Pay now" button where § 312j Abs 3 BGB demands "Zahlungspflichtig bestellen", or a shipping cost revealed only after the payment step will win an A/B test and lose a regulator. When a cut is legally constrained, name the rule, cite the article, and propose the compliant alternative.

## When to use

- A client sends a checkout URL or Figma flow and asks "why is our checkout conversion only 38%?"
- A funnel export shows a step-level drop of ≥10 percentage points between two checkout steps and someone needs to know which fields cause it.
- A migration to a new checkout (Shopify Plus checkout extensibility, commercetools, custom headless) needs pre/post field parity: every field in the old flow mapped to keep/drop/moved, before cutover.
- A new market launch (DE, FR, NL, PL) requires adding fields — company VAT ID, house number separate from street, pickup point selection — and you must stop field creep.
- A payment provider change (adding Klarna, iDEAL, Bancontact) risks duplicating address or email capture inside the PSP's own sheet.
- Mobile checkout completion sits below 55% of the desktop rate and the hypothesis is input burden.

**Do not use this when:**
- The problem is upstream of checkout — cart abandonment driven by shipping cost or delivery date. That is a *cart and shipping-promise audit*; reach for the cost-transparency teardown instead.
- The problem is payment failure or 3DS drop-off (authorisation rate below 92%, or `begin_checkout`→`add_payment_info` healthy but `purchase` broken). That is a *payment authorisation audit*; pull PSP decline codes and SCA exemption rates, not field counts.
- Someone wants copy rewriting or trust-badge placement. That is a *checkout messaging review*; this skill only touches copy where it blocks or mis-labels an input.

## Inputs

| Input | Required | If missing |
|---|---|---|
| Live checkout URL or staging access, guest path included, with a test card that completes a real order | Yes | Request a full screen recording of a complete guest purchase on mobile and desktop at 1× speed with the keyboard visible; do not audit from screenshots |
| Step-level funnel data (sessions entering/exiting each step, last 28 days, split mobile/desktop) | Yes | Ask for GA4 `begin_checkout` → `add_shipping_info` → `add_payment_info` → `purchase` counts; if unavailable, mark every lift estimate "unquantified hypothesis" and rank by Cost × field count only |
| Field-level analytics (time-in-field, correction rate, error rate per field) | No | Proxy with three timed test orders per device, recording per-field seconds with a stopwatch and transcribing every validation error string; label the numbers "observed, n=3", not "measured" |
| Operational field requirements (carrier, ERP/WMS, fraud, tax) with a named owner | Yes | Flag every candidate removal "pending ops confirmation", keep it out of tier 1, and list the owner by name in §8 |
| Market list and VAT setup (B2C/B2B, OSS registration, countries served, rates applied) | Yes | Assume B2C-only in the single declared country at the standard rate; write the assumption in the Summary line, not a footnote |
| Current consent configuration (marketing opt-in, CMP, account creation) | No | Inspect the DOM for `checked` attributes on consent inputs and record every one as a finding regardless |

With roughly half the inputs you still run the audit, but you change its shape. Without funnel data you produce a *ranked field inventory* instead of a lift-estimated backlog: keep the verdicts and compliance findings, delete every percentage, and add a measurement plan naming the exact events and parameters to instrument before anyone builds. Without ops confirmation, split the backlog into "ship now" — purely presentational changes: `autocomplete` tokens, `inputmode`, validation timing, label wording, field order — and "confirm then ship", meaning anything that stops storing a value. Never fill an input gap by inventing an operational reason; write "no confirmed consumer — owner: ops" and move on.

## Method

1. **Walk the flow end to end and build the raw field inventory.** Complete two real guest purchases — one mobile at 375×812, one desktop at 1440×900 — plus one logged-in purchase if accounts exist, and one B2B purchase if a VAT-ID path exists. Record every input in DOM order with eleven columns: step, label, element type, required/optional, default value, `autocomplete` token, `inputmode`, validation trigger (keystroke / blur / submit), error copy verbatim, Cost, Necessity.
   - Count one entry per input element, not per visual row: "First name / Last name" is two fields; a split card form is four (number, expiry, CVC, name); a native `<select>` is one field but scores Cost ≥3 on mobile.
   - Conditional fields (company VAT ID behind an "I'm a business" toggle) are recorded with the condition stated and counted at 0.3 weight in the totals.
   - Capture the field set inside PSP iframes and wallet sheets too; a Klarna sheet that re-asks for the address is a duplicate-capture finding even though it is not your DOM.

2. **Measure the baseline burden.** Compute, for the mobile run: total required fields, total taps (every focus, keyboard switch, dropdown open and option select counts as one), keystrokes, and time from first input focus to the order-confirmation paint, averaged over three orders.
   - Benchmarks for B2C guest checkout in a single EU market: ≤12 required fields and ≤90 s on mobile is healthy; 13–17 required fields is bloated and named in the Summary; ≥18 is a red-flag finding and must be the Summary headline.
   - Keyboard switches are counted separately: more than two switches between alpha and numeric keypads across the address block is a defect, fixed with `inputmode="numeric"` on postcode and house number.
   - If the total exceeds 20 because of a B2B path, recompute against the B2C path alone and state both numbers ("21 required on the B2C path; 24 with the VAT-ID block").

3. **Score each field on cost and necessity.** Cost 1–5 from typing effort, observed error rate, label ambiguity and keyboard switches: 1 = one tap, no typing (pre-filled country); 3 = 6–12 characters typed; 5 = free-text over 12 characters, or a select with >10 options, or an observed error rate above 10%. Necessity 1–5 from evidence only: 5 = a named law, carrier API field, PSP requirement or ERP NOT-NULL column; 4 = a named system reads it and degrades without it; 3 = a system reads it and tolerates nulls; 2 = stored, no reader identified; 1 = no consumer at all.
   - Verdict matrix, applied mechanically: Necessity ≤2 → **Cut**. Necessity 3–4 with Cost ≥3 → **Autofill or derive**. Necessity 3–4 with Cost ≤2 → **Merge or make optional**. Necessity 5 → **Keep, reduce cost in place**.
   - A field ops "would like to have" but cannot tie to a system ("how did you hear about us?") scores Necessity 1 and is Cut, with a note offering it as a one-click post-purchase survey on the thank-you page where it costs zero checkout taps.
   - Never score Necessity 4 or 5 without writing the system or rule name in the Note column. An unnamed Necessity 5 is a Necessity 2 the client has not checked.

4. **Run the compliance pass before you rank anything.** Every verdict touching price display, consent or contract formation passes through this gate; a Cut here is downgraded to Keep and logged as a rejected candidate with the reason.
   - Total price including VAT, shipping and all unavoidable surcharges must be visible on the same screen as the order button, above the fold on 375px. The button must carry payment-obligation wording — "Zahlungspflichtig bestellen", "Kaufen", "Order with obligation to pay" (§ 312j Abs 3 BGB, Art. 8(2) CRD). "Pay now", "Continue", "Complete order" fail.
   - Marketing consent must be a separate, unticked opt-in, never bundled with terms acceptance (GDPR Art. 4(11), Art. 7(2)). Account creation must not be a condition of purchase. A pre-ticked box is a defect of severity high, never a conversion asset.
   - The 14-day withdrawal right and the model withdrawal form must be linked before the order button (CRD Art. 6(1)(h)). For digital content delivered immediately, the explicit consent plus acknowledgement of losing the withdrawal right (CRD Art. 16(m)) is a required checkbox and is never a removal candidate.
   - Any promotional price in the order summary carries the 30-day lowest-price reference (Omnibus Directive, Art. 6a Price Indication Directive; § 11 PAngV in DE). Do not simplify the summary by dropping it.
   - Record each finding with severity: **high** = regulator-actionable or contract-invalidating; **medium** = incorrect disclosure that a diligent shopper would notice (a cent-level VAT error); **low** = wording or placement that is defensible but weak.

5. **Fix money math before fixing fields.** Hand-reconcile one mixed-rate test order to the cent and compare with the rendered summary.
   - Work from gross B2C prices: `net = round(gross / (1 + rate), 2)` per line, VAT as the difference; sum line VAT, never re-derive VAT from a rounded order total. €29.99 at 19% = €25.20 net, €4.79 VAT.
   - Shipping carries VAT at the rate of the goods it delivers; in a mixed-rate basket apportion shipping gross by line net value, then compute VAT on each apportioned part.
   - Discounts reduce the gross line and therefore the VAT: a 10% code on €29.99 gives €26.99 gross, €22.68 net, €4.31 VAT.
   - For B2B intra-EU with a VAT ID validated against VIES, the summary switches to net prices with a reverse-charge note (Art. 196 VAT Directive). If the checkout cannot validate in real time, the VAT-ID field stays and is never proposed for removal.
   - A discrepancy of €0.01 is a medium finding, not a rounding curiosity: it means the summing order is wrong and will scale on larger baskets.

6. **Apply the reduction techniques in fixed priority order.** For every Autofill/Merge verdict, name the technique, not the goal.
   - Order: (a) **derive** — city and province from postcode lookup (DE/AT/NL/PL 1:1 enough to prefill, always editable); (b) **native autofill** — correct tokens (`email`, `given-name`, `family-name`, `address-line1`, `postal-code`, `address-level2`, `tel`, `cc-number`, `cc-exp`, `cc-csc`) plus `inputmode="numeric"` on postcode, house number and card fields; (c) **address lookup** on a single search input with a visible "Enter address manually" escape hatch; (d) **merge** into one input only where the data model and carrier accept it; (e) **defer** to post-purchase.
   - Never merge street and house number for DE/AT/NL/CZ: DHL, PostNL and GLS reject or mis-sort the parse. Keep a separate, narrow house-number input with `inputmode="numeric"` and a suffix-tolerant pattern for NL (`12-A`, `12 bis`).
   - Never collapse billing and shipping silently. Use a pre-checked "Billing address same as delivery" that reveals the second block on uncheck — this removes 6–7 required fields from the default path without losing the data.
   - Validation fires on blur, never on keystroke; error copy names the fix ("Postcode must be 5 digits"), not the fault ("Invalid input").

7. **Rank into a sequenced backlog.** Score each recommendation ICE-style: Impact = midpoint of the estimated relative lift range (compliance fixes take Impact 0 and are ranked above everything by severity), Confidence 1–5 by evidence strength (5 = client's own data or a defect fix; 3 = benchmark range applied to a comparable field; 1 = intuition), Effort in developer-days to one decimal. Sort by `Impact × Confidence / Effort`.
   - Ranking order is fixed: all high-severity compliance fixes first, then medium-severity, then conversion items by ICE score descending.
   - Lift benchmarks, used only as order-of-magnitude and always quoted as a range: removing one required field on mobile ≈ 0.5–1.5% relative lift on checkout completion; correcting `autocomplete` tokens across the whole address block ≈ 2–4%; collapsing an always-visible billing block ≈ 2–3%; express wallet above the form ≈ 5–10%; moving validation from keystroke to blur ≈ 0.5–1%.
   - Anything with Confidence ≤2 **or** Effort ≥3 days goes to the test-first tier with a named hypothesis, primary metric and computed sample size — not to the build list.
   - Cap tier 1 at eight rows, and make rank 1 completable in under one developer-day so the team ships something in week one.

8. **Write the measurement plan.** For each tier-1 change: metric, segment, instrumentation gap, decision threshold, written before handover.
   - Primary metric: checkout completion rate = sessions with `purchase` ÷ sessions with `begin_checkout`, segmented mobile/desktop. Secondary: per-field error rate, time-to-complete, and — for anything touching the address block — carrier delivery-failure rate over the following 30 days.
   - Sample size: `n per variant = 2 × 7.85 × p(1−p) / d²` at 95%/80%. At p = 0.38 and a 1.5pp absolute target, that is ~16,400 sessions per variant.
   - Minimum run: 14 days covering two full weekends, or the computed sample, whichever is longer. Never call a result before both.
   - Defect fixes (wrong token, keystroke validation, missing button wording) ship without a test and are judged on error-rate telemetry 7 days after release.

## Judgement calls

**Cutting a field vs making it optional** — cut when no downstream system reads the value and it has no retention value; make it optional when exactly one system reads it and tolerates nulls. What tips it: whether a null breaks an automated process. If the WMS rejects orders with a null phone number, the field stays required and you reduce its cost instead (`inputmode="tel"`, `autocomplete="tel"`, no formatting mask). If a null only degrades an SMS notification, it goes optional with a one-line benefit note ("For delivery updates — optional"), and you expect 55–70% of shoppers to still fill it.

**Native autofill vs a third-party address lookup** — default to native autofill plus postcode-derived city: 0.5–2 developer-days, no per-query fee, no vendor. Choose a lookup API (Loqate, Google Places, PostNL) when delivery failures exceed 1.5% of orders, or in markets with structurally messy addressing: UK flats, IE Eircode, NL house-number suffixes, PL ulica/numer. What tips it: cost per failed delivery. Above €8 per failure, an API at ~€0.02/query pays back at 10,000 orders/month; below that, fix the tokens and stop.

**Express wallets above the form vs a cleaner single-column form** — put Apple Pay / Google Pay / PayPal above the form when mobile exceeds 60% of checkout sessions and guest orders exceed 70%, since the wallet skips the entire address block. Keep the top of the form clean when average basket exceeds €250 or B2B invoicing exceeds 20% of revenue, where wallet metadata maps poorly to the ERP. What tips it: whether the wallet-supplied address passes your validation unedited — test ten wallet orders; if more than one needs manual correction, fix the address mapping before promoting the wallet.

**Shipping directly vs A/B testing** — ship directly any defect fix: wrong or missing `autocomplete` token, validation firing on keystroke, missing payment-obligation wording, pre-ticked consent, VAT arithmetic errors. Test anything that stops storing data the business currently keeps, and anything with Impact midpoint ≥2%. What tips it: traffic. Below 1,000 checkout sessions per week you cannot resolve a 2% relative effect inside a quarter, so ship defect fixes, measure per-field error rates, and run no split tests at all.

## Rules

- Never invent a reason a field exists. With no confirming owner, write "no confirmed consumer — owner: ops" and mark the verdict provisional.
- Never recommend removing, hiding or deferring: the VAT-inclusive total, the shipping cost, the payment-obligation button wording, the withdrawal information, the terms link, or the digital-content withdrawal-waiver checkbox.
- Never recommend a pre-ticked box, a bundled consent, or forced account creation as a conversion tactic, even when it would raise the number.
- Never present an estimated lift as a measured one. Every number not taken from the client's own data is written as a range, carries the word "estimated", and states its assumption inline.
- Respect data minimisation (GDPR Art. 5(1)(c)): never propose adding a field for analytics or personalisation value alone.
- Keep VAT arithmetic at line level, rounded to two decimals per line, summed upward; show gross prices to consumers in all EU B2C contexts.
- Cap tier 1 at eight items and make rank 1 under one developer-day.
- The decision to stop collecting operationally useful data (phone, company, delivery notes) belongs to the client's named operations owner; you supply the trade-off, the recommendation and the deadline for a decision.
- Mark every uncertainty inline as `[assumption: …]`, never in a footnote.
- Audit the guest path as the primary path. If guest checkout does not exist, that is finding number one and outranks every field-level item.

## Output format

```
# Checkout Friction Audit — <store> — <date>

## Summary
- Baseline: <N> required fields, <M> steps, <T>s mobile completion, <X>% checkout completion rate (<source, date range>)
- Headline: <one sentence naming the single biggest cost>
- Compliance defects found: <count> (see §4)
- Estimated combined lift from tier 1: <range>% relative [assumption: …]

## 1. Field inventory
| # | Step | Label | Type | Req | autocomplete | inputmode | Validation | Cost 1-5 | Necessity 1-5 | Verdict | Note |
|---|------|-------|------|-----|--------------|-----------|------------|----------|---------------|---------|------|

## 2. Verdict detail
### Cut
- <field> — no consumer identified — owner to confirm: <name/role> — decision due <date>
### Autofill or derive
- <field> — technique — effort in days
### Merge or make optional
- <field> — proposed shape — expected fill rate
### Keep, reduce cost in place
- <field> — rule or system requiring it — cost reduction

## 3. Money math check
- Test order: <items, gross, VAT rate(s)>
- Hand-computed per line: net <>, VAT <>, shipping apportionment <>, total <>
- Displayed: <> — reconciles / discrepancy of <> — cause
- Prior-price (Omnibus) display on discounted lines: present / missing

## 4. Compliance findings
| # | Finding | Where | Rule touched | Severity | Fix | Effort |
|---|---------|-------|--------------|----------|-----|--------|

## 5. Ranked backlog
| Rank | Change | Est. impact | Confidence | Effort (d) | ICE | Owner |
|------|--------|-------------|------------|------------|-----|-------|

## 6. Test first
- <change> — hypothesis — primary metric — sample per variant — run length

## 7. Measurement plan
- Primary metric, segments, instrumentation gaps, decision thresholds, review date

## 8. Assumptions and open questions
- [assumption: …]
- Open: <question> — owner — due
```

- Target 900–1,400 words plus tables; the field inventory table may run long and is never trimmed.
- Order is fixed: Summary first so a stakeholder can stop after 20 lines; compliance before backlog so no ranked item contradicts a finding.
- Over length, cut in this order: §6 prose (keep the rows), then Keep-verdict notes in §2, then desktop walkthrough timings. Never cut §3 or §4.

## Worked example

**Input:** Nordvelt Home, DE/AT/NL homeware store on Shopify Plus (checkout extensibility), guest checkout enabled, B2C with an optional company field. GA4, 1–28 Sep: 41,200 `begin_checkout`, 15,650 `purchase` → 38.0% completion; mobile 31.4% (25,544 sessions), desktop 49.1% (15,656 sessions). Average basket €96. Ops owner Lena Brandt (Head of Fulfilment) confirms DHL requires a phone number only for Packstation and parcel-shop orders (11% of German orders) and that the WMS accepts null phone. CRM owner Tobias Reil confirms "How did you hear about us?" is not mapped to any Klaviyo or ERP field. Three timed mobile test orders averaged 142 s, 47 taps, 6 alpha/numeric keyboard switches.

**Output (extract, §§1–8):**

**Summary** — Baseline: 21 required fields, 3 steps, 142 s mobile completion, 38.0% checkout completion (GA4, 1–28 Sep; mobile 31.4%, desktop 49.1%). Headline: an always-visible billing block and a required source dropdown add 9 required fields and 14 mobile taps that no confirmed system reads — 21 required fields is above the ≥18 red-flag line. Compliance defects: 4 (two high, two medium). Estimated combined tier-1 lift: 6–11% relative [assumption: 0.5–1.5% relative per required mobile field removed; benchmarks applied to Nordvelt's 62% mobile mix].

**§1 Field inventory (extract of 25 rows)**

| # | Step | Label | Type | Req | autocomplete | inputmode | Validation | Cost | Nec | Verdict | Note |
|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | 1 Contact | Email | email | Y | (none) | (none) | keystroke | 3 | 5 | Keep, reduce cost | Order confirmation + contract. Add `autocomplete="email"`, `inputmode="email"`, validate on blur |
| 2 | 1 | "Send me offers" | checkbox | N | — | — | — | 1 | 1 | Keep, untick | **Pre-ticked** — compliance finding C1 |
| 3 | 1 | How did you hear about us? | select (11 options) | Y | (none) | — | submit | 4 | 1 | Cut | No CRM or ERP mapping — confirmed Tobias Reil, 14 Sep. Offer as thank-you-page survey |
| 4 | 2 Delivery | Country | select | Y | `country-name` | — | — | 1 | 5 | Keep, reduce cost | Tax + carrier routing. Default from IP, keep editable |
| 5 | 2 | First name | text | Y | (none) | — | blur | 2 | 4 | Keep, reduce cost | DHL label field. Add `given-name` |
| 6 | 2 | Last name | text | Y | (none) | — | blur | 2 | 4 | Keep, reduce cost | DHL label field. Add `family-name` |
| 7 | 2 | Company | text | N | `organization` | — | — | 1 | 3 | Make optional (hide) | 4.2% fill rate. Move behind "Deliver to a business" toggle; weight 0.3 |
| 8 | 2 | Street | text | Y | `address-line1` | — | blur | 3 | 5 | Keep, reduce cost | Carrier-required. Address lookup in tier 2 |
| 9 | 2 | House number | text | Y | (none) | text | blur | 2 | 5 | Keep, reduce cost | **Never merge with street** — DHL/PostNL parse. Add `inputmode="numeric"`, allow NL suffix `12-A` |
| 10 | 2 | Address line 2 | text | N | `address-line2` | — | — | 2 | 2 | Cut | 3.1% fill rate; NL suffixes handled by field 9 |
| 11 | 2 | Postcode | text | Y | (none) | text | keystroke | 2 | 5 | Keep, reduce cost | Add `postal-code`, `inputmode="numeric"` — removes 2 of 6 keyboard switches |
| 12 | 2 | City | text | Y | (none) | — | blur | 2 | 4 | Derive | Prefill from DE/AT/NL postcode lookup, remain editable |
| 13 | 2 | Phone | tel | Y | `tel` | tel | blur | 3 | 3 | Make optional | DHL needs it for Packstation/parcel-shop only (11% of DE orders); WMS accepts null — Lena Brandt, 12 Sep |
| 14 | 2 | Delivery notes | textarea | N | — | — | — | 2 | 2 | Cut | Not printed on the label, not read by WMS |
| 15–21 | 2 | Billing block: first, last, street, house no., postcode, city, country | mixed | Y | partial | — | blur | 5 | 4 | Merge | PSP requires a billing address but accepts a copy. Collapse behind pre-checked "Billing address same as delivery" |
| 22 | 3 Payment | Card number | text | Y | `cc-number` | numeric | blur | 4 | 5 | Keep, reduce cost | Shopify Payments |
| 23 | 3 | Expiry | text | Y | `cc-exp` | numeric | blur | 2 | 5 | Keep | — |
| 24 | 3 | CVC | text | Y | `cc-csc` | numeric | blur | 2 | 5 | Keep | — |
| 25 | 3 | Name on card | text | Y | `cc-name` | — | blur | 2 | 2 | Cut | Not required by Shopify Payments; not used in AVS for DE/AT/NL |

Required-field count after tier 1: 11 (email, country, first, last, street, house no., postcode, city prefilled, card number, expiry, CVC) — inside the ≤12 benchmark. Estimated mobile completion 75–85 s [assumption: 2.8 s saved per removed field at observed typing speed, plus 11 s saved on the billing block].

**§2 Verdict detail (extract)** — *Cut:* "How did you hear about us?" — no CRM mapping — owner Tobias Reil, decision due 20 Sep; relocate to the thank-you page as a 5-option single-tap survey. Address line 2 — 3.1% fill, no carrier consumer — owner Lena Brandt. Delivery notes — not printed on labels — owner Lena Brandt. Name on card — not used by Shopify Payments — no owner needed. *Derive:* City — DE/AT/NL postcode→city table (8,200 DE, 1,300 AT, 4,100 NL entries), prefill on 5-digit (DE/AT) or 4-digit+2-letter (NL) entry, field stays editable — 2.0 d. *Merge:* Billing block → pre-checked "Billing address same as delivery", second block revealed on uncheck; expected uncheck rate 6–9% based on the current 7.4% of orders where billing ≠ shipping — 1.0 d. *Keep, reduce cost:* House number — DHL label parse — narrow 80px field, `inputmode="numeric"`, NL suffix pattern.

**§3 Money math check** — Test order #NV-10427: 1× Alva table lamp €89.00 gross (19%), 1× "Nordisk Køkken" cookbook €24.00 gross (7%), shipping €4.90. Lamp: net €74.79, VAT €14.21. Cookbook: net €22.43, VAT €1.57. Goods net €97.22; apportionment 76.93% / 23.07%. Shipping 19% share €3.77 → net €3.17, VAT €0.60; 7% share €1.13 → net €1.06, VAT €0.07. Hand totals: gross €117.90, net €101.45, VAT €16.45. Displayed: total €117.90 — reconciles; "incl. VAT €16.44" — **discrepancy of €0.01**, caused by VAT re-derived from the rounded order total (117.90 − 117.90/1.19 blended) instead of summed per line. Prior-price display on the Autumn promotion lines: **missing**.

**§4 Compliance findings**

| # | Finding | Where | Rule touched | Severity | Fix | Effort |
|---|---|---|---|---|---|---|
| C1 | Marketing opt-in pre-ticked and placed directly under the terms sentence | Step 1 | GDPR Art. 4(11), Art. 7(2); § 7 UWG | High | Unticked by default, separated from terms acceptance by its own block | 0.25 d |
| C2 | Order button reads "Jetzt bezahlen" | Step 3 | § 312j Abs 3 BGB; CRD Art. 8(2) | High | "Zahlungspflichtig bestellen" (DE/AT), "Bestelling met betalingsverplichting" (NL) | 0.25 d |
| C3 | Autumn promotion shows "€89,00 — statt €129,00" with no 30-day lowest price | Order summary, all steps | Omnibus / Art. 6a PID; § 11 PAngV | High | Add "Niedrigster Preis der letzten 30 Tage: €99,00" per discounted line | 1.0 d |
| C4 | VAT line understates by €0.01 on mixed-rate baskets | Order summary | § 14 UStG (correct VAT disclosure) | Medium | Sum per-line VAT; stop re-deriving from the order total | 0.5 d |

**§5 Ranked backlog**

| Rank | Change | Est. impact | Conf | Effort (d) | ICE | Owner |
|---|---|---|---|---|---|---|
| 1 | C2: payment-obligation button wording, 3 locales | compliance (high) | 5 | 0.25 | — | Marek Dubois (FE) |
| 2 | C1: untick and unbundle marketing consent | compliance (high) | 5 | 0.25 | — | Marek Dubois |
| 3 | C3: Omnibus prior-price line on discounted items | compliance (high) | 5 | 1.0 | — | Sofie Haugen (BE) |
| 4 | Correct `autocomplete`/`inputmode` across email, name, postcode, house number; move validation to blur | est. 2–4% rel. | 4 | 0.5 | 24.0 | Marek Dubois |
| 5 | Cut source dropdown; move to thank-you page | est. 1–1.5% rel. | 4 | 0.5 | 10.0 | Tobias Reil |
| 6 | Collapse billing behind pre-checked "same as delivery" | est. 2–3% rel. | 4 | 1.0 | 10.0 | Marek Dubois |
| 7 | Cut address line 2, delivery notes, name on card | est. 1.5–2.5% rel. | 3 | 0.5 | 12.0 | Lena Brandt (sign-off) |
| 8 | Phone optional with "Nur für Packstation-Lieferung nötig" note | est. 0.5–1.5% rel. | 4 | 0.5 | 8.0 | Lena Brandt |

C4 (€0.01 VAT) and postcode→city derivation (est. 1–2%, conf 3, 2.0 d, ICE 2.25) sit in tier 2 — tier 1 is capped at eight rows.

**§6 Test first** — Apple Pay / Google Pay above the form. Hypothesis: wallet payment skips the 11-field address block for the 62% mobile cohort, lifting mobile completion from 31.4% to 34–37%. Primary metric: mobile checkout completion. Confidence 2 (no wallet baseline at Nordvelt; benchmark-only). Effort 3.0 d. Sample: 16,400 sessions per variant at p = 0.314, 1.5pp absolute MDE; at 912 mobile sessions/day split 50/50, that is 36 days — run 42 days to cover six weekends. Guard metric: share of wallet orders needing manual address correction, threshold 10%.

**§7 Measurement plan** — Primary: `purchase` ÷ `begin_checkout`, segmented mobile/desktop, reviewed 7 and 28 days after each release. Secondary: per-field error rate (instrumentation gap — no `form_error` event exists today; add with `field_id` and `error_text` parameters, 0.5 d, must ship before rank 4), mobile time-to-complete, and DHL delivery-failure rate over 30 days for ranks 6–8. Decision thresholds: keep any change whose segment-level completion rate is not below baseline minus 0.5pp after 14 days; roll back rank 8 if delivery failures exceed 1.5% of DE orders. Review date: 28 Oct with Lena Brandt and Marek Dubois.

**§8 Assumptions and open questions** — [assumption: field-removal lift of 0.5–1.5% relative per required mobile field, applied to Nordvelt's 62% mobile mix]. [assumption: billing ≠ shipping on 7.4% of orders, from the Sep order export; not verified against 12-month seasonality]. Open: does Klaviyo need the phone number for the abandoned-cart SMS flow? — owner Tobias Reil — due 20 Sep. Open: AT and NL button wording sign-off — owner external counsel — due 25 Sep.

## Quality bar

- [ ] Every input element in the live guest checkout appears as a row in §1, each with a Cost and a Necessity score; the row count matches the field count stated in the Summary.
- [ ] Every Necessity 4 or 5 names the law, carrier, PSP or system in its Note column.
- [ ] §4 appears above §5, and no §5 row proposes removing or deferring anything §4 lists as required.
- [ ] §3 shows one test order reconciled line by line, with each VAT rate stated and the displayed-versus-computed difference given to the cent.
- [ ] Every impact figure in §5 is a range, the word "estimated" appears with it, and its assumption is written in §8.
- [ ] §5 has eight rows or fewer; each has a named owner and an effort in days to one decimal; rank 1 is under 1.0 d.
- [ ] Every operational claim not confirmed by a named person carries `[assumption: …]` or "owner: ops" with a due date.
- [ ] The Summary states the baseline required-field count against the ≤12 / 13–17 / ≥18 benchmark.
- [ ] §7 states a sample size per variant and a run length in days for every tested change.

## Failure modes

**Backlog full of "simplify the form" items** — the audit was run from screenshots instead of real test orders, so no field-level cost was observed — check that §1 carries a measured time-to-complete, a tap count, and a Cost score on every row; if any is blank, the walkthrough was skipped.

**A recommendation that would breach consumer law** — compliance was treated as a review step after ranking rather than a gate before it — check that every Cut verdict touching price, consent, terms or withdrawal has an explicit §4 outcome recorded, including rejected candidates and the reason.

**Delivery failures rise after shipping the backlog** — street and house number were merged, or city was derived without an editable override, breaking carrier address parsing in DE/AT/NL — check that the house-number field survived as a separate input and that every derived field is still user-editable; verify against ten real labels post-release.

**Lift estimates presented as forecasts and then missed** — benchmark ranges were quoted as point values and confidence was never applied — check that each §5 row shows a range and a confidence 1–5, and that everything at confidence ≤2 or effort ≥3 d sits in §6 with a sample size.

**Client ships nothing** — the backlog listed 25 undifferentiated items with no owners or effort estimates — check that §5 is capped at eight rows, every row has a named human, and rank 1 is under one developer-day.

**Tests called early and the wrong change kept** — a 3-day read on a 2% effect looked significant — check that §7 states a computed sample per variant and a minimum 14-day run covering two weekends, and that no decision was taken before both were met.

## License

MIT
