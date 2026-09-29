---
name: product-page-cro-review
owner: checkoutlab
category: Conversion & UX
description: Reviews a product page template against the shopper's real purchase questions and EU price, review and safety-info rules, and returns ranked test hypotheses with metrics for store owners and CRO specialists.
version: v3
license: MIT
updated: 2026-09-29
recommended: false
security_checked: false
url: https://ecomdly.com/skills/checkoutlab/product-page-cro-review
raw: https://ecomdly.com/raw/checkoutlab/product-page-cro-review.md
install: npx @ecomdly/cli add checkoutlab/product-page-cro-review
---

# Product page CRO review

Produces a prioritised list of findings for one product page template, each written as a testable hypothesis with a metric and a severity. Most product pages lose the sale on a missing answer (fit, total cost, delivery date, returns, compatibility), not on button colour. The review must start from the questions a shopper of this category asks before adding to cart, check whether the page answers each one where the decision is made, and separate CRO findings from legal must-fixes, which are not A/B tests.

## When to use
- A product template (or a high-traffic product) has an add-to-cart rate well below comparable templates in the same store, or has dropped against its own history.
- A new or changed product template goes live: review one representative page per template, plus one edge-case page (many variants, out of stock, no reviews, on sale).
- Before planning an A/B test programme on product pages, to build a ranked hypothesis backlog.

## When not to use
- The drop is between cart and purchase: use `checkout-friction-audit`.
- You need to know where in the funnel users drop, not why: run `ga4-funnel-analyst` first.
- Structured data or rich result problems: use `product-schema-validator`.
- Writing the description itself: use `product-description-writer` after this review names the gaps.

## Inputs
Required:
- The page: live URL the agent can fetch, or full-length screenshots at mobile width (390 px CSS) and desktop (1440 px), including the state after a variant is selected and after add-to-cart.
- Product data for that item: price, variants, stock status, specifications, images count.
- The store's real delivery options (carriers, cost, cut-off time, lead time), returns policy and warranty terms.

Optional, each changes what can be concluded:
- GA4 per page or template, last 28–90 days: `view_item` users, `add_to_cart` users, add-to-cart rate = users with `add_to_cart` / users with `view_item`, split by device category. Export from GA4 Explorations (free form, dimension "Item name" or "Page path", metrics "Items viewed", "Items added to cart").
- Comparable template or category average from the same store (not an industry benchmark) as the baseline.
- Reviews export (count, average, text), returns reasons (hand off to `returns-reason-analyzer`), on-site search queries and customer-service questions about the product. These are the best source for "the top purchase questions".
- Field performance data: PageSpeed Insights / CrUX for the URL or origin.

If the page itself is missing, stop and ask; do not review from memory of the brand. If analytics are missing, the review still runs, but severity rests on judgement and every finding is labelled "no traffic data".

## Best practices

### Decide what the shopper must know first
1. Derive the top purchase questions per category before looking at the page. Apparel and footwear: fit, size system, material, returns for wrong size. Electronics: compatibility, included items, warranty. Furniture: dimensions, delivery to the room, assembly. Consumables: quantity, unit price, expiry. Rank them using customer-service tickets, on-site search, review text and return reasons when available. Why: a page can pass every generic check and still fail the one question that decides the category.
2. Check where each answer sits, not just whether it exists. An answer buried in a tab below the fold on mobile counts as partial. The decision is made at the variant selector and the add-to-cart button, so size guide, delivery date and returns summary belong there.

### Above the fold on mobile
3. At 390 px width the first screen shows: product name, price (with unit price where required), primary image, rating summary if reviews exist, and either the variant selector or the add-to-cart button. If add-to-cart is below the fold, check for a sticky add-to-cart bar; if neither, High severity for mobile-heavy stores.
4. Add-to-cart is the single primary action; wishlist, compare and instalment links are visually secondary.

### Images and media
5. Count is not the criterion; coverage is: the item from several angles, a close-up of material or detail that the category cares about (sole, seams, ports), scale or in-use shot, what is in the box. Each colour variant swaps the gallery. Zoom works on mobile by pinch. Missing coverage is a finding; never suggest adding images that do not exist, only shooting them.

### Variants
6. Unavailable variants stay visible and are marked (struck-through or "out of stock") rather than hidden, so the shopper learns that their size exists. Offer back-in-stock notification only if the store has that function.
7. Selecting a variant updates price, image, stock status and SKU without a full reload and without layout shift. Test this explicitly; it breaks often.
8. Size selector: the size system is labelled (EU 43, UK 9), the size guide link sits next to the selector and opens without leaving the page. If the size guide is generic (one chart for all brands), note it: brand-specific fit notes are the fix.

### Price, costs and legal price display
9. Total cost clarity near the button: shipping cost or free-shipping threshold, and a delivery date or range computed from the cut-off time ("order by 14:00, delivered Thu–Fri"). A carrier logo is not a delivery date.
10. Legal checks, reported separately as "Compliance", not as test ideas (EU sellers):
   - Price reductions: when a reduction is announced, the prior price shown must be the lowest price applied in the 30 days before the reduction (Price Indication Directive 98/6/EC, Art. 6a, inserted by the Omnibus Directive 2019/2161). A struck-through "was" price that is a recommended retail price or last week's price is a finding. Check the price history if available; otherwise mark "not verified".
   - Unit price (per kg, litre, metre, piece) for goods sold by quantity, per Directive 98/6/EC and its national transposition; member states may exempt some cases, so flag missing unit price as "check national rule" rather than a definite breach.
   - Reviews: if reviews are displayed, the page must say whether and how the trader ensures they come from customers who actually bought or used the product (UCPD 2005/29/EC Art. 7(6)). Claiming "verified reviews" without such checks is a blacklisted practice (UCPD Annex I, point 23b).
   - Urgency: countdown timers and "only 2 left" must be true. Falsely stating a product or price is available only for a very limited time is a blacklisted practice (UCPD Annex I, point 7). If the stock counter is not tied to real inventory, report it.
   - Safety information for distance sales: manufacturer name and postal and electronic address (and the EU responsible person when the manufacturer is outside the EU), product identification, and warnings or safety information must be shown in the online offer (GPSR, Regulation (EU) 2023/988, Art. 19).
   - Textiles: fibre composition must be visible before purchase, including online (Regulation (EU) 1007/2011, Art. 16).

### Returns, warranty and trust
11. One line near the button with the real terms ("30 days free returns, prepaid label") linked to the policy. The statutory 14-day withdrawal right is not a selling point on its own in the EU; state what exceeds it (longer period, free return shipping) if the policy allows.
12. Payment methods the market expects (for Czech stores typically card, bank transfer, cash on delivery; check the store's list) shown near the price or button, as small marks, not a wall of logos.
13. A visible way to ask a question (chat, phone, Q&A) for high-consideration items.

### Description and specifications
14. Specs as a scannable table (label: value), units stated, the attributes the shopper filters by present. The top two purchase questions answered in the first lines of the description, not in paragraph four.

### Social proof
15. Rating average and count next to the title, linking to reviews. Reviews filterable (by rating, by fit for apparel), negative reviews visible. A product with zero reviews gets a finding "no reviews: consider `post-purchase-review-request`", never fabricated or imported-from-elsewhere reviews.

### Performance
16. Core Web Vitals "good" thresholds at the 75th percentile of page loads: LCP at or under 2.5 s, INP at or under 200 ms, CLS at or under 0.1. Use field data (CrUX) when it exists; lab data (Lighthouse) is a diagnostic, not the verdict. The LCP element on product pages is usually the main image: check it is not lazy-loaded and is sized for mobile.

### Writing the finding
17. Every CRO finding is a hypothesis: "Because [observation + evidence], changing [element] will increase [metric] for [segment]". Metric is primary (add-to-cart rate) plus a guardrail (return rate, revenue per visitor). Findings the store should just fix (broken zoom, a wrong price on variant change, legal gaps) are labelled "Fix, no test".
18. Severity: High = blocks or misleads the decision for most visitors of the template, or legal exposure; Medium = answer exists but is hard to find, or affects one segment; Low = polish. Effort S/M/L. Rank by severity, then by reach (share of sessions affected, e.g. mobile share), then by effort.

## Process
1. Confirm scope: template name, URL(s), device split and traffic if available. If no data, note "no traffic data" once in the header.
2. Write the top purchase questions for the category (point 1), with their evidence source.
3. Walk the page on mobile in scroll order, then desktop. For each question record: answered / partially (where) / missing.
4. Run the checks in Best practices 3–16. For each, record pass, fail, or "not verified" when the input does not show it (for example no screenshot of the post-variant state).
5. Compute add-to-cart rate per device if data exists. If mobile rate is below desktop rate by more than the store's usual gap, prioritise mobile-only findings.
6. Split findings into Compliance, Fix (no test) and Test hypotheses.
7. Rank by severity, reach, effort. Keep at most 10 test hypotheses; more dilutes the backlog.
8. For each test hypothesis state the primary metric and a guardrail. If the template has too few add-to-cart events to detect a realistic change within a few weeks, recommend shipping low-risk fixes directly and comparing before/after periods, labelled as weaker evidence than a test; read any test that does run with `ab-test-readout`.

## Pitfalls and edge cases
- Reviewing the logged-in or cookie-accepted state only. Check what a first-time visitor sees, including the consent banner covering the button on mobile.
- Assuming a sale price is compliant because the store platform generated it. Platforms often show "compare at" prices that are not the 30-day lowest.
- Recommending "add urgency" or "add trust badges". Badges are recommended only when they represent something true (a real certification, a real guarantee).
- A/B testing legally required elements. They are not optional; ship them.
- Out-of-stock pages: the job is to keep the visitor (alternatives, notify me, expected date if known), not to maximise add-to-cart.
- Price shown excluding VAT on a B2C page: in the EU consumers must see the final price including taxes.
- Mobile add-to-cart rate being low can be a tracking problem; if `add_to_cart` fires inconsistently, hand off to `ga4-ecommerce-event-auditor` before drawing conclusions.

## Rules
- Read-only. Propose changes; a human approves and implements.
- Report only what is in the input. Anything not visible is "not verified", never assumed passed or failed.
- Recommendations may reorganise, move or reword facts that exist in the product data or policies. Never add product facts, stock counts, badges, certifications, reviews or delivery promises that are not in the data.
- No invented uplift estimates. Say which metric should move, not by how much.
- Legal points are flagged for the store owner to confirm with their counsel; state the rule and source, do not give a legal opinion on liability.

## Output format
```
Page: <URL> | Template: <name> | Reviewed: <date> | Devices: mobile 390px, desktop 1440px
Data: <GA4 period, view_item users, add_to_cart rate by device> | or "no traffic data"
Top purchase questions: 1) <question> [answered|partial: where|missing] (source: <tickets|search|reviews|judgement>)

COMPLIANCE (fix, not test)
C1. <finding> | Rule: <law, article> | Evidence: <what was seen> | Fix: <action>

FIX, NO TEST
F1. [High|Medium|Low] <finding> | Evidence | Fix | Effort S/M/L

TEST HYPOTHESES (ranked)
H1. [High] <finding>
    Because <observation + evidence>, changing <element> will increase <primary metric> for <segment>.
    Primary: <metric> | Guardrail: <metric> | Reach: <share of sessions> | Effort: <S/M/L>

Performance: LCP <value or n/a> | INP <value or n/a> | CLS <value or n/a> (source: CrUX|Lighthouse lab)
Not verified: <list>
```

## Worked example
Example input: footwear template, `/p/trail-runner-x`, GA4 last 28 days: 8,400 users with `view_item` (mobile 6,300, desktop 2,100); `add_to_cart` users mobile 252, desktop 147. Mobile add-to-cart rate = 252 / 6,300 = 4.0 %; desktop = 147 / 2,100 = 7.0 %. Returns export shows "too small" as the top reason. Price shows "2 490 Kč, was 3 290 Kč"; no price history supplied.

```
Page: /p/trail-runner-x | Template: footwear | Reviewed: 2026-09-29 | Devices: mobile 390px, desktop 1440px
Data: GA4 28 days, view_item 8,400 users; add_to_cart rate mobile 4.0 %, desktop 7.0 %
Top purchase questions: 1) Does it run small? [missing] (source: returns reasons) 2) Delivery before the weekend? [partial: in footer policy only] 3) Grip on wet rock? [answered: spec table]

COMPLIANCE (fix, not test)
C1. "Was 3 290 Kč" shown; 30-day lowest prior price not verifiable | Rule: Dir. 98/6/EC Art. 6a | Evidence: no price history in input | Fix: confirm the lowest price in the 30 days before the reduction and show that as the reference price

FIX, NO TEST
F1. [High] Selecting size 44 keeps the image of the black colourway after switching to blue | Evidence: screenshot 3 | Fix: bind gallery to colour variant | Effort S

TEST HYPOTHESES (ranked)
H1. [High] Size guide is a generic chart in a footer link; no fit note
    Because "too small" is the top return reason and the mobile add-to-cart rate is 3 points below desktop, adding a size-guide link and the brand fit note ("runs half a size small", from the supplier sheet) next to the size selector will increase mobile add-to-cart rate.
    Primary: add_to_cart rate (mobile) | Guardrail: return rate for size reasons | Reach: 75 % of sessions | Effort: S
H2. [Medium] Delivery date not near the button; policy says order by 14:00, dispatched same day, 1–2 days
    Because the answer exists only in the footer, showing "order by 14:00, delivered <date range>" under the button will increase add_to_cart rate.
    Primary: add_to_cart rate | Guardrail: delivery-related CS tickets | Reach: 100 % | Effort: M

Performance: LCP 3.1 s | INP 180 ms | CLS 0.02 (source: CrUX, origin-level, URL has too little data)
Not verified: review verification statement (no reviews section in screenshots); GPSR manufacturer details (spec tab not captured)
```

## Quality checklist
- Every finding cites what was seen (screenshot, element, data point); nothing asserted from general knowledge of the brand.
- Compliance, Fix and Test sections are separate; no legal requirement is proposed as an A/B test.
- Every test hypothesis has "Because … changing … will increase …", a primary metric and a guardrail.
- Rates are recomputed from the raw counts and match the header.
- No invented facts, badges, stock counts, reviews or uplift numbers.
- "Not verified" lists every check that the input could not show.
- At most 10 test hypotheses, ranked.

## Sources
- Price Indication Directive 98/6/EC, consolidated with Art. 6a: https://eur-lex.europa.eu/eli/dir/1998/6/oj
- Omnibus Directive (EU) 2019/2161: https://eur-lex.europa.eu/eli/dir/2019/2161/oj
- Unfair Commercial Practices Directive 2005/29/EC (Art. 7(6), Annex I points 7 and 23b): https://eur-lex.europa.eu/eli/dir/2005/29/oj
- General Product Safety Regulation (EU) 2023/988, Art. 19: https://eur-lex.europa.eu/eli/reg/2023/988/oj
- Textile Fibre Names Regulation (EU) 1007/2011, Art. 16: https://eur-lex.europa.eu/eli/reg/2011/1007/oj
- Web Vitals thresholds: https://web.dev/articles/vitals
- GA4 recommended ecommerce events: https://developers.google.com/analytics/devguides/collection/ga4/ecommerce

## License
MIT
