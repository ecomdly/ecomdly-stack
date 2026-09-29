---
name: post-purchase-review-request
owner: inboxcart
category: Email & retention
description: Plans a post-delivery review request flow that asks every buyer the same way, with timing by product type, no gating, and incentive rules checked against Google, Trustpilot and EU review law.
version: v3
license: MIT
updated: 2026-09-29
recommended: true
security_checked: true
url: https://ecomdly.com/skills/inboxcart/post-purchase-review-request
raw: https://ecomdly.com/raw/inboxcart/post-purchase-review-request.md
install: npx @ecomdly/cli add inboxcart/post-purchase-review-request
---

# Post-purchase review request

Produces a review-request flow (trigger, suppression, one request, at most one reminder, copy with merge tags) and a policy check against the review platforms the store uses. The naive flow fails twice: it fires on purchase, so the request lands before the parcel and earns "haven't received it yet" ratings; and it "protects" the rating by skipping unhappy customers or routing them to a private form, which is review gating, prohibited by Google and Trustpilot and a misleading practice under EU law. The core of this skill is asking every buyer, the same way, at the moment they can judge the product.

## When to use
- Setting up or fixing a review flow in an ESP (Klaviyo, Mailchimp, Ecomail) or a review platform (Trustpilot, Yotpo, Judge.me, Heureka Ověřeno zákazníky, Google Customer Reviews).
- Review volume is low, reviews mention delivery problems the request itself caused, or someone proposed an incentive for reviews.
- Before submitting product reviews to Google Merchant Center Product Ratings.

## When not to use
- Replying to or moderating reviews already posted (a different task; never delete negative reviews to improve an average).
- Delivery-problem follow-ups. Those are service messages; use `order-status-reply-drafter`.
- Analysing why products get returned; use `returns-reason-analyzer`.

## Inputs
Required:
- **Order and fulfilment data**: order ID, order date, line items (name, SKU, image URL, category), carrier "delivered" event with timestamp, or the estimated delivery date if no event exists.
- **Review destinations**: which platforms the store uses (store-level vs product-level) and whether any incentive exists today.
- **Consent flags** for the order's e-mail address (marketing consent, soft opt-in eligibility) and the country of the store.
- **Returns and support status per order**: open return, open ticket, resolved date.
- **Voice guide and support contact.**

Optional:
- Product-type usage time (how long before a customer can judge it), if the store knows better than the defaults below.
- Existing flow export: requests sent, reviews collected, star distribution before/after.

If the delivered event is missing for most orders, fall back to estimated delivery + 5 days and label timing as estimated. If the platform list is missing, stop and ask; the rules differ materially.

## Best practices

### Platform and legal rules (check each platform in use)
1. **Google Maps / Business Profile reviews: no incentives at all.** Google does not allow merchants to offer "payment, discounts, free goods and/or services" in exchange for a review, or for revising or removing a negative one, and does not allow merchants to "discourage or prohibit negative reviews, or selectively solicit positive reviews". Incentivised reviews are removed and the profile can be restricted.
2. **Google Merchant Center Product Ratings: incentives only if sentiment-independent and disclosed.** Google may show reviews obtained through incentives (gift cards, discounts), but the incentive cannot depend on the sentiment, and each such review must carry `<is_incentivized_review>` in the reviews feed. Only reviews "honestly solicited from customers who made a purchase" may be submitted; employee and conflict-of-interest reviews are removed. Eligibility requires at least 50 reviews across all products; confirm current thresholds in Product Ratings eligibility.
3. **Google Customer Reviews (store ratings)**: the opt-in must be shown to all customers, consistently on the order confirmation page, and Google itself mails the survey after delivery. Do not send a second store-rating request that competes with it; the store's own flow should ask about the product.
4. **Trustpilot: no incentives, no selective inviting.** Its business guidelines prohibit incentives of any kind connected with writing or editing reviews (discounts, promo codes, prize draw entries, refunds, freebies) and being selective: inviting only customers known to be happy, **timing invitations for a stage of the journey only happy customers reach**, and pre-screening people by sending happy ones to review and unhappy ones to contact support.
5. **EU unfair commercial practices law (as amended by Directive 2019/2161).** If the store shows reviews, whether and how it checks that they come from real buyers is material information that must be disclosed (UCPD Art. 7(6)). Claiming reviews are from verified buyers without reasonable and proportionate checks is blacklisted (Annex I point 23b), as is submitting or commissioning false reviews or misrepresenting reviews, which the recitals illustrate with publishing only positive reviews and deleting negative ones (point 23c).
6. **US stores**: the FTC rule on consumer reviews (16 CFR Part 465) prohibits incentives conditioned, expressly or implicitly, on a particular sentiment, and misrepresenting that displayed reviews are all or most reviews when negative ones were suppressed.
7. **Consequence for design**: the safe default everywhere is no incentive. If the store insists on one for on-site product reviews, it goes to every reviewer regardless of rating, is disclosed in the request and on the review, is flagged `is_incentivized_review` for Google, and is never used on Google Maps or Trustpilot invitations.

### Consent
8. **Treat the request as marketing unless counsel says otherwise.** A review request is not needed to perform the contract, and several regulators treat it as marketing. Send to contacts with marketing consent or a confirmed soft opt-in (ePrivacy Directive Art. 13(2)); include an unsubscribe link. Contacts who objected to marketing (GDPR Art. 21(3)) get nothing.
9. **Platform-sent invitations** (Heureka Ověřeno zákazníky, Google Customer Reviews, Trustpilot automatic invites) involve passing the buyer's e-mail to that platform. Confirm the store's privacy notice covers it; flag if unclear.

### Timing
10. **Count from delivery, not purchase.** Defaults by product type, tune with the store's knowledge:
    - Consumables and cosmetics: 10–14 days after delivery (they need several uses).
    - Apparel and shoes: 5–7 days (worn at least once; still within the 14-day withdrawal period, which is fine: the customer judges fit either way).
    - Electronics, furniture, appliances: 14–21 days (set up and used).
    - Gifts and seasonal items: consider asking about the ordering and delivery experience, not the product.
11. **One reminder at most**, 7 days after the first request, only to non-responders. More is nagging and inflates unsubscribes.
12. **Pause, do not exclude.** While a return, complaint or support ticket is open, hold the request; after it is resolved, send the standard request 7 days later. Permanently dropping everyone who complained is exactly the selective timing Trustpilot prohibits and skews the average upward in a way consumers are not told about.
13. **One request per order**, and no more than one review request per customer per 30 days across orders (store may choose another cap).

### Content
14. **Same message for everyone.** No "Did you love it? Yes → public review, No → tell us privately" split, and no star-click in the mail that routes 4–5 stars to the public page and 1–3 to a private form. A support line is allowed when it is the same for every recipient and is not offered as an alternative to reviewing.
15. **Name the product and show its image.** For multi-item orders, ask about up to three items, picking non-returned items with the highest price first; or link to a multi-item form if the platform supports it.
16. **Never write, suggest or pre-fill review text or ratings**, including "AI-suggested" phrasing. Do not ask for specific content (Google prohibits requesting specific content).
17. **Short**: body under 80 words, one CTA, the support line, the unsubscribe link.
18. **Verified label only when verified.** Only mark reviews "verified purchase" when they are matched to an order; state on the site how verification works (Art. 7(6)).

## Process
1. **Inventory destinations.** For each platform in use, record: incentives allowed? selective timing rules? who sends the invitation (store or platform)? Mark conflicts (for example, a planned discount that would breach Trustpilot's rules).
2. **Check consent coverage** and privacy-notice coverage for platform-sent invitations. Mark `BLOCKED` if the basis is unknown.
3. **Set the trigger** per product type: `delivered_at + N days`; fallback `estimated_delivery + 5 days` labelled estimated.
4. **Set suppression and pause logic**: unsubscribed/objected → never; open return/ticket → pause, resume `resolved_at + 7 days`; order cancelled or fully refunded before delivery → none; staff/test orders → none.
5. **De-duplicate with platform invites** so a customer does not get the store's request, a Google survey and a Heureka questionnaire in the same week. Stagger: platform survey first (it is store-level), product request later.
6. **Draft the request and reminder** in the store's voice and the order's language.
7. **Write the policy flags block** and the measurement plan: request-to-review rate per product type, star distribution compared with the pre-change period, share of reviews mentioning delivery issues.
8. **Run the quality checklist.**

## Pitfalls and edge cases
- **Split shipments**: trigger from the delivery of the parcel containing the reviewed item, not the first parcel.
- **Pickup points**: "delivered to pickup point" is not "collected". Use the collection event if the carrier provides it.
- **Delivery problems resolved by a replacement**: time from the replacement's delivery.
- **Subscriptions**: ask once after the first or second delivery, not every cycle.
- **Negative review arrives**: respond publicly and factually, fix the issue privately; never offer anything in exchange for editing or removing it (prohibited by Google and Trustpilot, and by the FTC rule in the US).
- **Syndicated reviews** (brand network, marketplace): check that the source platform's rules and the Google feed's incentive flag carry over.
- **Employee and friends' reviews**: exclude staff e-mail domains from the flow.
- **Heureka and other comparison sites** run their own post-purchase questionnaires; check the store's contract for their current rules rather than assuming Google's.

## Rules
- Read-only: produce the flow design, copy and policy flags; never activate flows, send invitations, or edit reviews.
- Never gate, never pre-fill, never invent ratings, quotes or review counts.
- Any incentive proposal is flagged per platform and needs human and, where relevant, legal sign-off.
- Missing delivery data is labelled as estimated timing, never presented as exact.
- Missing platform rules or consent basis: ask, do not assume.

## Output format
```
REVIEW FLOW · {{ store_name }} · platforms: <list>
Policy check:
| Platform | Incentive allowed | Selective timing ban | Sender | Status |
Trigger: delivered + <N> d (<product type>) · fallback estimated + 5 d [ESTIMATED]
Reminder: +7 d, non-responders only
Pause: open return/ticket → resume resolved + 7 d
Never send: unsubscribed/objected · cancelled before delivery · staff/test
Frequency cap: 1 request / customer / 30 d

Subject: <≤ 50 chars>
Body:
<≤ 80 words>
[<CTA>] -> {{ review_url }}
<support line>
<unsubscribe>

Tags used: {{ ... }}
Policy flags: <none | list>
Open questions: <list>
```

## Worked example
Illustrative: Czech outdoor shop, Trustpilot for store reviews, Judge.me for product reviews submitted to Google Product Ratings, Google Customer Reviews opt-in on the thank-you page. Marketing proposed "CZK 100 voucher for every 5-star review".

```
REVIEW FLOW · TrailShop · platforms: Trustpilot, Judge.me → Google Product Ratings, Google Customer Reviews
Policy check:
| Platform                 | Incentive allowed                 | Selective timing ban | Sender   | Status  |
| Trustpilot               | No                                | Yes                  | Platform | OK      |
| Google Product Ratings   | Yes, if not sentiment-dependent + flagged | Honest solicitation | Store | CONFLICT |
| Google Customer Reviews  | n/a (Google survey)               | Opt-in to all        | Google   | OK      |
Trigger: delivered + 6 d (apparel/shoes) · fallback estimated + 5 d [ESTIMATED]
Reminder: +7 d, non-responders only
Pause: open return/ticket → resume resolved + 7 d
Never send: unsubscribed/objected · cancelled before delivery · staff/test
Frequency cap: 1 request / customer / 30 d

Subject: How are the Speedcross 6 working out?
Body:
Hi {{ first_name }},
your {{ item_name }} arrived last week. Once you have had them on a trail or two,
would you tell other runners how they fit and grip? It takes a minute.
[Rate {{ item_name }}] -> {{ review_url }}
Something wrong with the order? Reply here and we will sort it out.
{{ unsubscribe_link }}

Tags used: {{ first_name }}, {{ item_name }}, {{ review_url }}, {{ unsubscribe_link }}
Policy flags:
- Proposed "voucher for 5-star reviews" is sentiment-conditional: breaches Google Product Ratings policy, Trustpilot guidelines, and would be misleading under UCPD. Not used.
- If any voucher is kept, it must go to every product reviewer regardless of rating, be disclosed, and be flagged is_incentivized_review; never on Trustpilot.
Open questions: Does the privacy notice cover passing e-mails to Trustpilot? Is the carrier "collected" event available for pickup points?
```

## Quality checklist
- Every platform in use has its incentive and selectivity rules recorded; conflicts are flagged, not resolved silently.
- Trigger counts from delivery (or is labelled estimated); one reminder maximum.
- Unhappy customers are paused and resumed, never permanently excluded; no routing by sentiment anywhere in the mail or landing page.
- No pre-filled text, suggested phrasing, or invented ratings.
- Consent basis is stated; unsubscribe present.
- Body ≤ 80 words, one CTA, support line identical for all recipients.
- Duplicate asks across store, Google and comparison-site surveys are staggered.

## Sources
- Google Maps user contributed content policy, Fake engagement: https://support.google.com/contributionpolicy/answer/7400114
- Google Business Profile, Tips to get more reviews: https://support.google.com/business/answer/3474122
- Google Merchant Center, Product Ratings policies: https://support.google.com/merchants/answer/6098512
- Google Merchant Center, Product Ratings eligibility: https://support.google.com/merchants/answer/14549080
- Google Merchant Center, Google Customer Reviews opt-in and survey: https://support.google.com/merchants/answer/14629305
- Trustpilot, Guidelines for businesses: https://legal.trustpilot.com/for-businesses/guidelines-for-businesses
- Directive (EU) 2019/2161, amending Directive 2005/29/EC (Art. 7(6), Annex I points 23b and 23c): https://eur-lex.europa.eu/eli/dir/2019/2161/oj
- FTC, Consumer Reviews and Testimonials Rule, 16 CFR Part 465: https://www.ftc.gov/legal-library/browse/federal-register-notices/16-cfr-part-465-trade-regulation-rule-use-consumer-reviews-testimonials-final-rule
- Directive 2002/58/EC (ePrivacy), Art. 13: https://eur-lex.europa.eu/eli/dir/2002/58/oj

## License
MIT
