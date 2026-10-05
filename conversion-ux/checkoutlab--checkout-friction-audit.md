---
name: checkout-friction-audit
owner: checkoutlab
category: Conversion & UX
description: Audits the checkout from cart to withdrawal flow using GA4 step data and a mobile guest walkthrough, ranks friction by severity and reach, and lists EU must-fixes such as the order button and pre-ticked extras.
version: v3
license: MIT
updated: 2026-09-29
recommended: false
security_checked: true
url: https://ecomdly.com/skills/checkoutlab/checkout-friction-audit
raw: https://ecomdly.com/raw/checkoutlab/checkout-friction-audit.md
install: npx @ecomdly/cli add checkoutlab/checkout-friction-audit
---

# Checkout friction audit

Produces a ranked table of checkout findings, from the cart to the order confirmation and the withdrawal flow, each with step, evidence, severity, reach, effort and a concrete fix, plus a separate list of EU legal must-fixes. The naive audit lists best practices and ticks boxes; the useful one starts from where the step-level data says people leave, tests the flow as a first-time guest on a phone, and distinguishes friction (costs conversion) from non-compliance (costs conversion and exposes the store), because the two get different owners and different urgency.

## When to use
- Cart-to-purchase or `begin_checkout`-to-`purchase` rate has fallen against the store's own history, or differs sharply by device, browser or payment method.
- Before and after a checkout redesign, a platform migration (for example to Shoptet, Upgates or Shopify), or adding a payment or shipping provider.
- As the EU compliance check of the ordering process (order button, pre-contract information, withdrawal function).

## When not to use
- Visitors do not reach the cart: that is a product page problem, use `product-page-cro-review`.
- You need the funnel numbers themselves or to find the leaking step across the whole site: run `ga4-funnel-analyst` first.
- Purchases in GA4 do not match the shop backend: use `revenue-discrepancy-reconciler`; do not diagnose friction from broken data.
- Recovering abandoners by e-mail: `abandoned-cart-sequence`.

## Inputs
Required:
- A walkthrough of every step on mobile (390 px) and desktop: screenshots or a screen recording from cart to thank-you page, as a guest, with at least one error state (submit an empty form, an invalid postcode).
- Field list per step: label, required or optional, input type, `autocomplete` value if the HTML is available.
- Shipping methods with cost, free-shipping threshold, delivery estimate; payment methods with any fee.
- Market(s) served and currency.

Optional:
- GA4 step events, last 28–90 days, split by device category: users with `view_cart`, `begin_checkout`, `add_shipping_info`, `add_payment_info`, `purchase`; also `add_payment_info` broken down by the `payment_type` parameter and `add_shipping_info` by `shipping_tier` if they are sent. Source: GA4 Explorations, funnel exploration, open funnel off.
- Payment provider decline and 3-D Secure abandonment reports (from the gateway dashboard).
- Customer-service tickets about ordering, and the store's own history of checkout conversion.

If step events are missing or are not all implemented, say so and continue with a qualitative audit; reach becomes an estimate labelled "est.". If `purchase` is double-counted or missing, stop the quantitative part and hand off to `ga4-ecommerce-event-auditor`.

## Best practices

### Measure before judging
1. Step conversion = users at step n+1 / users at step n, per device. The step with the largest absolute user loss, not the lowest rate, is usually where the fix pays. Compare to the store's own previous period and between devices; external "average checkout rates" vary too much by vertical to set targets.
2. A step that cannot be seen in the data (single-page checkout firing only `begin_checkout` and `purchase`) is audited qualitatively; recommend instrumenting `add_shipping_info` and `add_payment_info` as a finding.

### Costs and delivery (the most-cited abandonment reason)
3. Baymard Institute's abandonment survey (US shoppers, excluding "just browsing") lists extra costs too high, slow delivery, not trusting the site with card details, forced account creation and a too long or complicated checkout among the top reasons. It is a US sample and a survey of stated reasons; use it to decide what to look for, not as a prediction for this store.
4. Shipping cost, or an exact estimate by country/postcode, is visible in the cart before any personal data is asked. Revealing it on the payment step is High severity.
5. Free-shipping threshold with the remaining amount ("190 Kč to free shipping") in the cart, computed on the same basis the checkout uses (with or without discount, VAT).
6. Each shipping method shows a delivery date or range and the cost, not only the carrier logo. Pickup-point methods (Zásilkovna/Packeta, PPL, DPD, Balíkovna in CZ) need a working map or search on mobile; test selecting a point.
7. Payment surcharges: in the EU, charging consumers a fee for cards covered by the Interchange Fee Regulation is prohibited under PSD2 (Directive (EU) 2015/2366, Art. 62(4)), and any payment fee must not exceed the trader's cost (Consumer Rights Directive 2011/83/EU, Art. 19). Flag card surcharges; flag cash-on-delivery fees for the owner to confirm they reflect cost.

### Account and flow
8. Guest checkout is offered and at least as visible as login. Account creation is offered on the thank-you page with a password field only, because the data is already known.
9. Login for returning customers does not wipe the cart or send the user back to the start.
10. No step exists only to confirm data already entered, unless it is the final order review. Back navigation keeps all entered data; test it with the browser back button.
11. Express wallets (Apple Pay, Google Pay, PayPal) placed in the cart can skip the address form entirely; check they pass the shipping address to the shop so shipping cost is correct.

### Forms
12. Baymard reports that an ideal checkout can be as short as 7–8 form fields (12–14 form elements), against a much higher average in their benchmark of US sites. Use it as a direction: count fields shown by default and justify each one. Usual surplus: separate billing address shown by default, "Address line 2" open, company and VAT ID fields for B2C buyers (put behind a "buying for a company" checkbox; in CZ these are IČO and DIČ), title/salutation, required phone with no stated reason, "confirm e-mail".
13. Billing = shipping by default via a pre-checked checkbox; this is a convenience default, not an extra paid option, so it is not affected by the pre-ticked box ban in point 20.
14. Correct HTML `autocomplete` tokens so browsers and password managers fill the form: `email`, `given-name`, `family-name` (or `name`), `tel`, `street-address` or `address-line1`, `postal-code`, `address-level2` (city), `country`, `organization`, and on card fields `cc-name`, `cc-number`, `cc-exp`, `cc-csc`. Mobile keyboards: `type="email"`, `type="tel"`, `inputmode="numeric"` for postcode and card number.
15. Inline validation fires when the field loses focus, not on each keystroke; messages say how to fix ("Postcode has 5 digits, e.g. 110 00"), and the page scrolls to the first error on submit. Accept common formats (spaces in postcodes and phone numbers, +420 prefix) instead of rejecting them.
16. Address lookup or autocomplete by postcode/street speeds entry, but must allow manual entry when the lookup fails.

### Payment and trust
17. Local methods for the market. For Czech stores that typically includes card, instant bank transfer, and often cash on delivery and pay-later options; confirm with the store's actual payment mix rather than assuming.
18. 3-D Secure (Strong Customer Authentication under PSD2) is expected on card payments; test that returning from the bank app lands on the order confirmation, not an empty cart. High decline or 3DS abandonment rates in the gateway report are a finding for the payment provider.
19. Order total including all costs repeated next to the final button. Coupon field collapsed to a text link so it does not prompt a code search.

### EU legal requirements in the ordering process (report as Compliance)
20. No pre-ticked boxes for extra paid items (insurance, gift wrap, donations, priority shipping): express consent is required for any payment beyond the main obligation, and consent inferred from a default option does not count (CRD Art. 22). A pre-ticked newsletter box is a GDPR consent problem.
21. Accepted payment means and any delivery restrictions (countries, product types) shown clearly at the latest at the beginning of the ordering process (CRD Art. 8(3)).
22. Directly before the order is placed: main characteristics of the goods, total price including taxes and all delivery costs, duration and conditions of any subscription, shown clearly and prominently (CRD Art. 8(2)).
23. The final button is labelled "order with obligation to pay" or an equally unambiguous formulation (CRD Art. 8(2)); "Continue", "Submit" or "Confirm" do not qualify. If this is not met, the consumer is not bound by the order.
24. Withdrawal function: since 19 June 2026, traders concluding distance contracts through an online interface must provide a withdrawal function labelled "withdraw from contract here" or an equally unambiguous formulation, prominently placed and available throughout the withdrawal period, with a confirmation step ("confirm withdrawal") and an acknowledgement of receipt on a durable medium with date and time (CRD Art. 11a, inserted by Directive (EU) 2023/2673). Check the national transposition's wording for the store's market. It is not in the checkout itself but belongs in this audit because the ordering and withdrawal processes are judged together.
25. Accessibility: the European Accessibility Act (Directive (EU) 2019/882) applies to e-commerce services from 28 June 2025, with an exemption for microenterprises providing services. Keyboard operability, visible labels (not placeholders only), error messages announced to screen readers and sufficient contrast are the common checkout failures.

## Process
1. Collect step data per device. Compute step conversion and absolute loss per step. Mark the top two steps by absolute loss.
2. Walk the flow as a first-time guest on mobile, then desktop, then one returning-customer path. Record every screen, field and message. Trigger at least one validation error per form step.
3. Run checks 4–25 per step. Each check: pass, fail (with evidence) or "not verified".
4. For each fail, assign severity: 3 = blocks or misleads completion, or legal non-compliance; 2 = adds effort or doubt on the main path; 1 = polish or affects a small segment.
5. Reach = share of checkouts that hit the issue: 100 % if on the main path; otherwise from data (for example share choosing pickup points, share of mobile users) or an estimate labelled "est.".
6. Priority score = severity × reach (as a fraction). Order by score, then by effort (S before M before L). Compliance items are listed first regardless of score.
7. Write each fix as an action a developer can do without further research. For findings in the top-loss steps, add the metric to watch (that step's conversion rate, by device).
8. List everything not visible in the input under "Not verified".

## Pitfalls and edge cases
- Auditing only desktop, or only as a logged-in user with saved addresses: most friction appears for first-time mobile guests.
- Treating a low `add_payment_info` → `purchase` rate as UX: it is often payment declines, 3DS failures, or bank-transfer orders that are counted as purchases only later. Check the gateway report and how bank-transfer orders fire `purchase`.
- Cross-border: VAT and duties for non-EU destinations, and country-specific postcode formats. A Czech-only postcode validator blocks Slovak orders.
- Cash on delivery counted as "purchase" but later refused: not checkout friction; mention it only if the data mixes the two.
- Removing the phone field when the carrier requires it for delivery notifications; keep it and state why ("the courier will text you").
- Free-shipping threshold computed before discount in the cart but after discount in checkout: the shopper sees shipping appear.
- Consent banner or chat widget covering the pay button on small screens.
- Recommending removing the order review step on high-value or B2B orders, where it prevents errors.

## Rules
- Read-only. The agent does not place real orders unless the owner provides a test mode or test products, and never enters real card data.
- Report only what the input shows; everything else is "not verified".
- Reach numbers from data carry their source; estimates are labelled "est.".
- No invented conversion uplift. State which step metric should move.
- Legal items state the rule and source and are flagged for the owner (and their counsel) to confirm against national transposition; the agent does not give legal opinions.
- Changes to payment or shipping configuration are proposals for a human.

## Output format
```
Checkout audit: <store> | <date> | Markets: <list> | Devices tested: <list> | Path: guest + returning
Step data (<period>, source): view_cart -> begin_checkout <x %> | -> add_shipping_info <x %> | -> add_payment_info <x %> | -> purchase <x %>  (mobile / desktop)
Largest absolute loss: <step> (<n> users)

COMPLIANCE
| # | Step | Finding | Rule | Evidence | Fix |

FRICTION (ranked by severity x reach, then effort)
| # | Step | Finding | Evidence | Sev | Reach | Score | Effort | Fix | Watch metric |

Not verified: <list>
Instrumentation gaps: <list>
```

## Worked example
Example input: CZ fashion store, last 28 days, mobile only. `begin_checkout` 3,000 users, `add_shipping_info` 2,100, `add_payment_info` 1,500, `purchase` 1,050. Step rates: 70 %, 71.4 %, 70 %. Absolute loss: 900, 600, 450; largest at begin_checkout → shipping. Walkthrough shows shipping price first displayed at the shipping step, 13 default fields including company, IČO, DIČ and "confirm e-mail", final button "Pokračovat" ("Continue"), and a pre-ticked "Pojištění zásilky 29 Kč" (parcel insurance).

```
Checkout audit: example-shop.cz | 2026-09-29 | Markets: CZ | Devices tested: mobile 390px | Path: guest
Step data (28 days, GA4): begin_checkout -> add_shipping_info 70.0 % | -> add_payment_info 71.4 % | -> purchase 70.0 % (mobile)
Largest absolute loss: begin_checkout -> add_shipping_info (900 users)

COMPLIANCE
| 1 | Review | Final button reads "Pokračovat" | CRD Art. 8(2) | screenshot 6 | Label "Objednat s povinností platby" or equally unambiguous |
| 2 | Shipping | Parcel insurance 29 Kč pre-ticked | CRD Art. 22 | screenshot 4 | Default unticked |

FRICTION
| # | Step | Finding | Evidence | Sev | Reach | Score | Effort | Fix | Watch metric |
| 3 | Cart | Shipping cost first shown at shipping step | screenshots 1-4 | 3 | 100 % | 3.0 | S | Show cheapest method price and free-shipping remainder in cart | begin_checkout -> add_shipping_info |
| 4 | Details | 13 fields; company, IČO, DIČ, confirm e-mail shown by default | field list | 2 | 100 % | 2.0 | S | Hide company fields behind "Nakupuji na firmu"; drop confirm e-mail | begin_checkout -> add_shipping_info |
| 5 | Shipping | Pickup-point map does not scroll on mobile | recording 0:42 | 3 | 55 % (share choosing pickup, GA4 shipping_tier) | 1.65 | M | Replace with searchable list + map | add_shipping_info -> add_payment_info |

Not verified: 3-D Secure return path; withdrawal function on account pages; accessibility with screen reader
Instrumentation gaps: payment_type not sent with add_payment_info
```

## Quality checklist
- Step rates recomputed from user counts; the "largest absolute loss" matches the numbers.
- Every finding has evidence (screenshot, field list, data) and a fix a developer can act on.
- Compliance items cite the article and are separate from friction items.
- Scores = severity × reach, sorted correctly; estimated reach labelled "est.".
- Mobile guest path was covered; error states were triggered or listed as not verified.
- No uplift predictions, no benchmarks without named source.
- "Not verified" and instrumentation gaps listed.

## Sources
- Consumer Rights Directive 2011/83/EU (Art. 6, 8, 19, 22): https://eur-lex.europa.eu/eli/dir/2011/83/oj
- Directive (EU) 2023/2673 inserting Art. 11a (withdrawal function): https://eur-lex.europa.eu/eli/dir/2023/2673/oj
- PSD2, Directive (EU) 2015/2366, Art. 62: https://eur-lex.europa.eu/eli/dir/2015/2366/oj
- European Accessibility Act, Directive (EU) 2019/882: https://eur-lex.europa.eu/eli/dir/2019/882/oj
- Baymard Institute, cart abandonment reasons and form-field benchmark: https://baymard.com/lists/cart-abandonment-rate
- HTML autofill tokens (WHATWG): https://html.spec.whatwg.org/multipage/form-control-infrastructure.html#autofill
- GA4 ecommerce events (`begin_checkout`, `add_shipping_info`, `add_payment_info`, `purchase`): https://developers.google.com/analytics/devguides/collection/ga4/ecommerce

## License
MIT
