---
name: review-response-drafter
owner: helpdeskly
category: Customer care
description: Triages new reviews from Google Business Profile, Heureka, Trustpilot and the store's site by severity and drafts one-standard public replies plus internal follow-ups, policy-checked against platform rules and EU review law.
version: v1
license: MIT
updated: 2026-10-05
recommended: false
security_checked: true
url: https://ecomdly.com/skills/helpdeskly/review-response-drafter
raw: https://ecomdly.com/raw/helpdeskly/review-response-drafter.md
install: npx @ecomdly/cli add helpdeskly/review-response-drafter
---

# Review response drafter

Produces, for every new review in the period, a severity tier, a public reply draft and an internal follow-up (owner, action, linked order or claim), plus a short list of recurring themes. The naive approach thanks the five-star reviews, argues with the one-star ones and offers a voucher "if you update your review". That last move is prohibited by Google and Trustpilot, and on the store's own site, selective handling of reviews can amount to misrepresenting consumer reviews under the EU Omnibus rules. Arguing in public, or quoting an order to prove a point, leaks personal data and convinces nobody. **The key insight: the public reply is written for the next shopper, not the reviewer, so every review gets the same standard of reply regardless of its stars, and the problem itself is solved offline in the follow-up.** Whether the reviewer later changes the review is their decision alone.

## When to use
- Daily or weekly, on reviews that are new since the last run, across all review platforms the store uses.
- A one- or two-star review mentions a defect, a missing parcel, a refund or a safety problem.
- A spike of reviews on one theme, or a review that looks fake or off-topic.
- Before a store starts replying to reviews, to set the reply standard.

## When not to use
- Asking customers for reviews (timing, wording, incentives): `post-purchase-review-request`.
- The defect claim behind a review: `warranty-claim-handler`. The order status behind a "never arrived" review: `order-status-reply-drafter`.
- Root causes of return and defect themes across SKUs: `returns-reason-analyzer`.
- Review markup and star snippets on product pages: `product-schema-validator`.
- Marketplace seller feedback that affects account metrics (Amazon, Allegro, Kaufland): `marketplace-account-health-triage`.

## Inputs

| input | definition | typical source |
|---|---|---|
| Reviews | platform, review ID or link, date, rating, text, reviewer display name, any existing reply | Google Business Profile, Heureka administration, Trustpilot Business, store review app export |
| Order and claim lookup | whether the reviewer can be matched to an order, its status, open claim or return | store admin, helpdesk |
| Reply standard | signature (name or team), tone, length limit, languages, public contact channel for offline follow-up | owner |
| Escalation map | who owns safety, legal, delivery, product, support issues, and their response targets | owner |
| Moderation policy (own site) | which reviews the store publishes, how it checks they come from buyers, and how this is disclosed | owner, review app settings |

Optional: last period's themes for trend comparison; known incidents (carrier outage, recalled batch) to recognise clusters.

Missing input → stop and ask. Never invent a response-time target, a contact address or a fact about the order.

## Best practices

### One reply standard for every review
1. **Same structure for every rating:** thanks, the specific point the reviewer made, a fact or correction if there is one, the offline route for anything unresolved, signature. Varying effort by rating teaches readers the store only cares about complaints, and on the own site, handling negative reviews differently drifts towards the selective treatment that recital 49 of Directive 2019/2161 gives as an example of what traders must not do (publishing only positive reviews and deleting the negative ones).
2. **Short, polite, relevant.** Google's tips: be nice and professional, keep it short, reply when there is new or relevant information, and do not offer deals or promotions in replies. Trustpilot removes replies with personal information, threats or aggressive language.
3. **Admit what went wrong, do not accept what did not.** Google advises admitting mistakes but not taking responsibility for things outside your control; state facts the next shopper can check (policy, process), not a rebuttal of the reviewer.
4. **Reply in the review's language** and sign with the name or team from the reply standard; Google suggests signing with a name or initials.

### Never trade anything for a review
5. **Never ask a reviewer to change or remove a review, in public or in the follow-up.** Trustpilot forbids pressuring reviewers "to write, edit, or delete a review"; reviewers own their reviews and can edit or delete them themselves. Google notes customers can still change their review after reading a reply; that is their choice.
6. **No incentive tied to a review.** Google prohibits offering payment, discounts or free goods in exchange for posting, revising or removing a review; Trustpilot lists discounts, promo codes, prize draws, refunds and freebies connected with leaving a review. A refund or replacement owed under a claim is given because it is owed, never framed as linked to the review.
7. **Do not report reviews because they are negative.** Google: do not report a review just because you disagree with it; Trustpilot asks businesses to flag five-star reviews for the same reasons as one-star ones. Heureka does not delete reviews at a shop's request; suspected competitor reviews go to Heureka support for checking.

### Keep personal data out of public replies
8. **Nothing the reviewer did not publish.** No order numbers, products bought, dates, addresses, phone numbers, payment or health details, and no confirming or denying that the person ordered. GDPR data minimisation (Art. 5(1)(c)) applies to what the store discloses; Google says never share the reviewer's private information.
9. **Resolution goes offline.** The reply names a channel (support email, phone, contact form) from the reply standard; the follow-up ticket carries the order match.

### Store's own review section (EU Omnibus)
10. **Do not claim reviews are verified unless reasonable and proportionate checks exist** (Directive 2005/29/EC Annex I point 23b), and disclose whether and how the store checks that reviews come from buyers (Art. 7(6)).
11. **Do not misrepresent reviews** (Annex I point 23c). Recital 49 of Directive (EU) 2019/2161 gives publishing only positive reviews and deleting negative ones as an example. Moderation rules (abuse, personal data, off-topic) must apply identically to every rating; the agent flags any review it would hold back and names the rule.

### Severity triage
12. **S1 — safety, legal, data or threats:** injury, fire, allergic reaction, alleged fraud, a data leak, threats, or a reviewer posting someone's personal data. Owner immediately, at the escalation map's target; the public reply waits for the owner's wording.
13. **S2 — unresolved customer problem:** defect, non-delivery, refund pending, rude contact. Follow-up to support with the order match; route to `warranty-claim-handler` or `order-status-reply-drafter`.
14. **S3 — negative but resolved or opinion:** price, taste, delivery speed already resolved. Reply only; theme tagged.
15. **S4 — positive or neutral:** reply with the same standard; theme tagged.
16. **Policy flag (any tier):** looks fake, off-topic, promotional, or contains personal data. Recommend whether to use the platform's report tool and on which listed policy ground; the owner decides.

## Process
1. **Collect** new reviews since the last run per platform; skip ones already replied to unless the review was edited after the reply.
2. **Match** each to an order or claim where possible; record the match only in the internal follow-up.
3. **Tier** each review (rules 12–16) and tag one primary theme (product quality, sizing, delivery, packaging, support, price, other).
4. **Draft the reply** with the one standard; check length, language, personal data and the incentive and removal rules sentence by sentence.
5. **Draft the follow-up**: owner, action, link to the order or claim, due date from the escalation map.
6. **Summarise themes** with counts by platform and tier; send clusters (one SKU, one carrier) to `returns-reason-analyzer` or operations.
7. **Hand the batch to the owner** for approval. Nothing is posted until approved.
8. **Run the quality checklist.**

## Pitfalls and edge cases
- **Platform moderation**: Google may take up to 30 days to review a reply before it appears; do not post a duplicate.
- **Heureka replies** are added in the Heureka administration with "Reagovat"; Heureka asks not to use disallowed characters such as emoji.
- **Reviewer is not matchable**: say so neutrally ("we could not find this, please contact us at ..."), never "you are not our customer".
- **Edited reviews**: a reviewer who updates a review after resolution may get a short acknowledgement; never a thank-you for "changing the rating".
- **Employee or competitor reviews**: Google removes conflict-of-interest content; the store must not submit or commission reviews of itself either (Annex I point 23c; recital 49 of 2019/2161).
- **Mixed reviews**: tier by the most severe part; a five-star review mentioning a burn is S1.
- **Legal threats or mentions of the ČOI or SOI**: S1, no public reply until the owner decides.

## Rules
- Draft only. The agent never posts, edits or deletes a reply, reports a review, contacts a reviewer, or offers anything without the owner's approval.
- Never ask, hint or arrange for a review to be changed or removed; never link any benefit to a review.
- No personal data or order details in public replies.
- Never hide, delay or soften the handling of a review because of its rating.
- Policy and legal statements are limited to the sources below; anything else goes to the owner.

## Output format
```
REVIEW REPLIES · <store> · reviews <from>–<to> · prepared <date>
Totals: <n> reviews · S1 <n> · S2 <n> · S3 <n> · S4 <n> · policy flags <n>
By platform: Google <n> · Heureka <n> · Trustpilot <n> · own site <n>

<review ID> · <platform> · <date> · <stars>★ · tier <S1-4> · theme <theme> · lang <cs|sk|en|...>
  Review (excerpt): "<text>"
  Public reply draft: "<reply>"
  Checks: length <n>/<limit> · personal data none · incentive/removal ask none
  Follow-up (internal): owner <role> · action <action> · link <order/claim> · due <date>
  Policy flag: <none | ground + recommended report>

Themes: <theme> <n> (<platforms>) · cluster <SKU/carrier> → <routed to>
Awaiting owner approval: <n> replies, <n> follow-ups
```

## Worked example
Illustrative reviews and counts, Czech home-goods store, week 2026-09-28 to 2026-10-04; three of the nine reviews shown; replies shown in English, drafted in Czech in practice.

```
REVIEW REPLIES · DomaDobře · reviews 2026-09-28–2026-10-04 · prepared 2026-10-05
Totals: 9 reviews · S1 1 · S2 2 · S3 2 · S4 4 · policy flags 1
By platform: Google 3 · Heureka 4 · Trustpilot 1 · own site 1

G-77 · Google · 2026-10-01 · 1★ · tier S2 · theme product quality · lang cs
  Review (excerpt): "Kettle stopped heating after a month, nobody answers."
  Public reply draft: "Thank you for letting us know, and we are sorry the kettle let you down so soon.
  We want to put this right: please write to us at <support email> and we will take it from there.
  — Jana, customer care"
  Checks: length 40/80 words · personal data none · incentive/removal ask none
  Follow-up (internal): owner support · action open claim · link order 40955 / claim RK-2026-121 · due 2026-10-06
H-12 · Heureka · 2026-09-30 · 5★ · tier S1 · theme product quality
  Review (excerpt): "Great delivery, but the lamp's plug got hot and smelled burnt."
  Public reply draft: held for owner wording (safety)
  Follow-up (internal): owner product safety · action contact buyer, quarantine batch check · due 2026-10-05
T-03 · Trustpilot · 2026-10-02 · 1★ · tier S3 · theme other · policy flag: names a staff member's phone number
  Public reply draft: standard structure, not shown; it does not repeat the number
  Follow-up (internal): owner decides on report under "personal information"

Themes: product quality 3 (Google, Heureka) · delivery 2 · support 2 · price 1 · other 1
Awaiting owner approval: 8 reply drafts, 1 reply held for owner wording, 4 follow-ups
```

Check: tiers 1 + 2 + 2 + 4 = 9; platforms 3 + 4 + 1 + 1 = 9; themes 3 + 2 + 2 + 1 + 1 = 9. Replies awaiting approval: 9 reviews minus the S1 held for owner wording = 8. Follow-ups: S1 1 + S2 2 + policy flag 1 = 4.

## Quality checklist
- Every review got a reply draft built on the same structure, regardless of rating (S1 held only for owner wording).
- No reply asks for, hints at or thanks for a changed or removed review; no benefit linked to a review.
- No order numbers, product purchased, dates, contact details or health information in any public reply.
- Each S1 and S2 has an internal follow-up with owner and due date from the escalation map.
- Policy flags name the platform's listed ground; none is based only on a low rating.
- Own-site moderation applies the same rules to all ratings and "verified" appears only where checks exist.
- Counts by tier, platform and theme add up to the total.

## Sources
- Google Business Profile Help, Manage customer reviews: https://support.google.com/business/answer/3474050
- Google Business Profile Help, Tips to get more reviews: https://support.google.com/business/answer/3474122
- Google Business Profile Help, Report inappropriate reviews on your Business Profile: https://support.google.com/business/answer/4596773
- Google Maps, Prohibited and restricted content (user contributed content policy): https://support.google.com/contributionpolicy/answer/7400114
- Trustpilot, Guidelines for Businesses: https://corporate.trustpilot.com/legal/for-businesses/guidelines-for-businesses
- Heureka.cz, Jak reagovat na recenze zákazníků?: https://sluzby.heureka.cz/napoveda/recenze-reakce/
- EUR-Lex, Directive 2005/29/EC (Unfair Commercial Practices), Art. 7(6) and Annex I points 23b and 23c as amended: https://eur-lex.europa.eu/eli/dir/2005/29/oj
- EUR-Lex, Directive (EU) 2019/2161 (Omnibus), Art. 3 and recitals 47–49: https://eur-lex.europa.eu/eli/dir/2019/2161/oj
- EUR-Lex, Regulation (EU) 2016/679 (GDPR), Art. 5(1)(c): https://eur-lex.europa.eu/eli/reg/2016/679/oj

## License
MIT
