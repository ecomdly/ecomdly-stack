---
name: abandoned-cart-sequence
owner: inboxcart
category: Email & retention
description: Designs a three-mail abandoned cart flow for an ESP with an EU consent check, exit and frequency rules, a margin-tested discount only in the last mail, and a holdout to measure real recovery.
version: v3
license: MIT
updated: 2026-09-29
recommended: false
security_checked: false
url: https://ecomdly.com/skills/inboxcart/abandoned-cart-sequence
raw: https://ecomdly.com/raw/inboxcart/abandoned-cart-sequence.md
install: npx @ecomdly/cli add inboxcart/abandoned-cart-sequence
---

# Abandoned cart sequence

Produces a three-mail abandoned cart flow (templates with merge tags, or per-cart drafts for human review) plus the entry, exit and suppression logic an ESP needs to run it. The naive version fails in three places: it mails people the store has no legal basis to mail, it leads with a discount and teaches customers to abandon on purpose, and it measures itself by ESP-attributed revenue, which counts buyers who would have come back anyway. Each mail here has one job — remind, help, decide — and none of them pretends.

## When to use
- Building or rebuilding a cart flow in an ESP (Klaviyo, Mailchimp, Ecomail, SmartEmailing, Shopify Email) with merge tags.
- Drafting per-cart mails when the store wants a human to approve each send.
- Auditing an existing flow that "works" but relies on a standing discount or has never been tested against a holdout.

## When not to use
- Browse abandonment (viewed a product, no cart). The legal basis is weaker and the content differs; do not reuse this flow.
- Transactional mails (order confirmation, payment failed on a placed order, shipping). Those are service messages; see `order-status-reply-drafter`.
- Lapsed customers with no recent cart. Use `winback-segment-planner`.
- Diagnosing why carts are abandoned in the first place. Use `checkout-friction-audit`; a mail flow cannot fix a broken shipping step.

## Inputs
Required:
- **Cart payload fields available in the ESP**: item name, variant (size/colour), image URL, unit price incl. VAT at time of send, quantity, cart URL that restores the cart, currency, locale.
- **Consent data per contact**: marketing consent flag and its source and timestamp, plus whether the contact is an existing customer who was offered an opt-out at the time the address was collected. If the ESP only has "subscribed yes/no", ask where the flag comes from before designing entry rules.
- **Discount policy**: which codes exist, for whom, minimum order, end dates, and the store's gross margin by category (or a single blended margin). If missing, the flow ships with no discount at all.
- **Voice guide** and **support contact** (address or reply-to that reaches a human).
- **Shipping rules**: free-shipping threshold, carriers, delivery promise.

Optional:
- Top product FAQs or pre-sale support tickets per category (feeds mail 2).
- Stock levels (only then may a mail mention stock).
- Existing flow performance export: sends, clicks, orders per mail, and whether a holdout exists.

If the consent source is unknown, output the templates but mark the entry rule `BLOCKED: legal basis unconfirmed` and do not propose a go-live.

## Best practices

### Legal basis comes before copy
1. **Treat a cart reminder as direct marketing.** It promotes a purchase the person has not made. Under the ePrivacy Directive 2002/58/EC Art. 13(1), marketing e-mail to natural persons needs prior consent. Build the flow as marketing unless the store's counsel has documented otherwise for its market.
2. **The soft opt-in is narrower than most stores assume.** Art. 13(2) lets a business mail its own *customers* whose address it obtained "in the context of the sale of a product or a service", for its *own similar products*, and only if they were clearly given a free, easy way to object at collection and in every message. Whether an unfinished checkout counts as "in the context of the sale" is decided by national law and regulators (Czechia transposes this in § 7 of Act No. 480/2004 Coll.). The UK ICO reads its equivalent as covering "negotiations for a sale" but says plain browsing is not enough. Decision rule: contacts with recorded marketing consent → eligible. Existing customers who were shown an opt-out at collection and did not use it → eligible if counsel confirms the soft opt-in for the market. Guests with no order and no consent → not eligible until counsel signs off in writing.
3. **The opt-out at collection must exist at collection.** The ICO states that putting the opt-out in the order confirmation is too late. Check the checkout: is there an unticked marketing box or a clear opt-out next to the e-mail field?
4. **Do not capture e-mails that were typed but not submitted** unless counsel approves; many regulators consider this collection without transparency. Flag any ESP setting that does it.
5. **Cart tracking depends on cookie consent.** Identifying a returning visitor and reading the cart via an ESP script is storage/access on the device (ePrivacy Art. 5(3)) and needs consent where it is not strictly necessary. If the visitor refused, the flow will not trigger for them. That is correct behaviour, not a bug to work around.
6. **Every mail identifies the sender and carries a working opt-out** (Art. 13(4)). For Gmail, marketing mail must support one-click unsubscribe (List-Unsubscribe header, RFC 8058) and show a visible unsubscribe link. An objection to direct marketing is absolute under GDPR Art. 21(3): suppress immediately across all flows, not only this one.

### Timing and frequency
7. **Three mails, spaced by purpose, not by habit.** Mail 1 at about 1 hour (the decision is still warm), mail 2 at about 24 hours (time to have hit a real objection), mail 3 at about 72 hours (last reminder or the only offer). These are starting points to test, not proven optima; do not quote conversion figures for them.
8. **Exit on any purchase, not only this cart.** Check orders from all channels (web, phone, marketplace if the ID matches) before each send. A customer who bought by phone and then gets "still thinking?" files a complaint.
9. **Re-entry cap.** One cart flow per contact per 14 days (store may choose 7–30). Without a cap, habitual cart-builders receive a mail every visit, which approaches the "persistent and unwanted solicitations" prohibited by the Unfair Commercial Practices Directive, Annex I point 26.
10. **Priority over other flows.** If a contact is in a cart flow, pause browse-abandonment and win-back sends; send at most one marketing mail per day across flows.
11. **Minimum cart value.** Skip carts below the cost of the effort only if the store sets a threshold; do not invent one.

### Content
12. **Dynamic content reflects the cart at send time.** Show current price and stock, not the price at abandonment. If the price rose, show the new price; if an item went out of stock, drop it from the mail and suppress the flow if nothing is left.
13. **Mail 2 answers a real objection.** Pull the top two questions for the product type from FAQs or support tickets (sizing, delivery time, returns, compatibility). State the return right accurately: 14 days from receipt for EU distance sales (Directive 2011/83/EU Art. 9) plus any longer store policy.
14. **Free-shipping nudges only when true.** "You are CZK 210 from free shipping" is computed from the live cart and the live threshold, or omitted.
15. **No invented urgency.** A countdown, "only today" or "almost gone" must be backed by a real end date or a stock number. Falsely stating that a product or terms are available only for a very limited time is a blacklisted practice (UCPD Annex I point 7).
16. **Short and single-purpose.** Subject under 50 characters, body under 120 words, one CTA back to the restored cart. No fake personal sender ("Jana from the team") unless Jana exists and replies.

### Discount discipline
17. **No discount in mails 1 and 2.** An offer in the first mail pays people who were coming back anyway, and a predictable offer trains repeat abandonment.
18. **A discount in mail 3 must pass a margin check.** Contribution after discount = price × (1 − d) − COGS − shipping cost borne − payment fee. Maximum discount d_max = gross margin % − the store's minimum contribution % per order. If margin data is missing, no discount.
19. **Omnibus price rule.** If the mail announces a price reduction on the product ("was / now"), it must show the prior price = the lowest price applied in at least the 30 days before the reduction (Directive 98/6/EC Art. 6a, inserted by Directive 2019/2161). A personal code with no reference price is a different presentation; confirm national guidance before relying on that.
20. **Single-use codes with a real expiry,** limited to first-time carts if the policy says so, and excluded from carts that already contain discounted items if the policy stacks badly.

### Measurement
21. **Hold out.** Keep a random 10% of eligible carts out of the flow (or at least out of mail 3 when testing the discount). Incremental recovery = conversion rate of mailed carts − conversion rate of holdout carts, over the same 7-day window. ESP-attributed revenue is not incrementality.
22. **Watch complaint signals.** Keep Gmail Postmaster spam rate below 0.10% and never at 0.30% or above (Google sender guidelines). A cart flow that pushes the domain over this harms every other mail the store sends.

## Process
1. **Confirm legal basis.** Map each consent source to eligible / conditional / not eligible per rules 1–5. Write the entry filter. If any source is unknown, mark `BLOCKED`.
2. **Define trigger and data.** Trigger = cart created or updated, checkout not completed, contact identified, 60 minutes without activity. List merge tags the ESP actually exposes; if a needed field (restored cart URL, variant) is missing, flag it rather than faking it.
3. **Write exit and suppression conditions**: order placed (any channel), unsubscribed or objected, bounced, open support ticket, all items out of stock, re-entry cap hit, contact is staff/test.
4. **Draft mail 1 (Remind).** Product name in the subject, image, one reason to buy taken from the product description, link back. No offer.
5. **Draft mail 2 (Help).** Two objections with factual answers, a line to reach a human, the return right.
6. **Decide mail 3.** If policy has an eligible discount and d ≤ d_max: state it with its real end date and code terms. Otherwise: a plain last reminder with the date the cart will be cleared, if the store actually clears carts; if it does not, say nothing about clearing.
7. **Add the holdout and the metrics** (incremental recovery, unsubscribe rate per mail, spam complaints).
8. **Run the quality checklist** and list every open question for the human approver.

## Pitfalls and edge cases
- **Logged-out returning visitor on a shared device**: the cart may belong to someone else in the household. Never show the full address or order history in the mail.
- **B2B carts** (company ID filled): consent rules for legal persons differ nationally (Art. 13(5)); handle as a separate segment only after confirmation.
- **Preorders and back-in-stock items**: mail 1 must state the expected dispatch date from the product data, or omit dates.
- **Price drops during the flow**: never show the old higher price as a strikethrough unless it complies with the 30-day prior price rule.
- **Payment failure after order placement** is not an abandoned cart; it goes to a transactional flow.
- **Multi-language stores**: send in the locale the cart was built in, not the contact's first-ever locale.
- **Apple Mail Privacy Protection** pre-fetches images, so opens are unreliable. Do not use "opened mail 1" as a branch condition; use clicks or site activity.
- **Heavy repeat abandoners** who always wait for mail 3: if the holdout shows no lift from the discount for them, drop the offer for contacts with more than one prior discounted recovery.

## Rules
- Read-only. Produce templates, rules and drafts; never activate a flow, create discount codes or change ESP settings.
- Offers, end dates, stock figures and shipping thresholds come only from the inputs. Missing means omitted, never estimated.
- Never write urgency, scarcity or social proof ("12 people are looking") without a data source.
- No guilt, no "we noticed you left", no fake personal sender.
- A discount, a change to the entry filter's consent logic, and go-live each require explicit human sign-off; legal-basis questions go to the store's counsel.
- Merge tags in `{{ }}`; list every tag used so the ESP mapping can be checked.

## Output format
```
FLOW: Abandoned cart · store {{ store_name }} · locale {{ locale }}
Legal basis: <eligible sources> | conditional: <sources + what counsel must confirm> | excluded: <sources>
Entry: cart updated, no order, contact identified, 60 min idle, consent filter = <rule>
Exit: order any channel · unsubscribe/objection · bounce · open ticket · all items OOS · re-entry < 14 d
Holdout: 10% random, measured on 7-day conversion
Discount check: margin <x>% − floor <y>% = d_max <z>% · proposed <d>% · <PASS/FAIL/NONE>

### Mail 1 · +1 h · Remind
Subject: <≤ 50 chars>
Body: <≤ 120 words>
CTA: <label> → {{ cart_url }}

### Mail 2 · +24 h · Help
Subject:
Body:
CTA:

### Mail 3 · +72 h · Decide
Subject:
Body:
CTA:

Tags used: {{ ... }}
Open questions for approver:
- ...
```

## Worked example
Illustrative store: Czech running shop, blended gross margin 38%, minimum contribution per order 25%, free shipping from CZK 1 500, policy allows a 10% single-use code for first-time customers valid 48 hours. Checkout has an unticked newsletter box; existing customers were shown an opt-out on the account form.

```
FLOW: Abandoned cart · store BěžecPro · locale cs → drafted in English for review
Legal basis: newsletter consent = eligible | conditional: existing customers via soft opt-in (counsel to confirm for CZ, § 7 Act 480/2004) | excluded: guests without consent
Entry: cart updated, no order, contact identified, 60 min idle, consent filter = consent OR (customer AND soft_opt_in_confirmed)
Exit: order any channel · unsubscribe/objection · bounce · open ticket · all items OOS · re-entry < 14 d
Holdout: 10% random, measured on 7-day conversion
Discount check: margin 38% − floor 25% = d_max 13% · proposed 10% · PASS (first-time customers only)

### Mail 1 · +1 h · Remind
Subject: Your Speedcross 6 is still in the cart
Body: Hi {{ first_name }}, the Salomon Speedcross 6 in {{ item_variant }} is waiting in your cart at {{ item_price }}. Its deep lugs are built for mud and wet trails. Your cart is saved, pick up where you left off.
CTA: Back to cart → {{ cart_url }}

### Mail 2 · +24 h · Help
Subject: Sizing the Speedcross 6
Body: Two questions we get most: fit and returns. The Speedcross 6 runs narrow; if you are between sizes, most customers go half a size up (size chart: {{ size_chart_url }}). You have 14 days from delivery to return it without giving a reason. Questions? Reply to this mail and a person from our shop will answer.
CTA: Back to cart → {{ cart_url }}

### Mail 3 · +72 h · Decide
Subject: 10% off your first order, until Friday
Body: If the Speedcross 6 is still on your list: code {{ coupon_code }} takes 10% off your first order until {{ coupon_expiry }}. One use, not combinable with sale items.
CTA: Use my code → {{ cart_url }}

Tags used: {{ first_name }}, {{ item_variant }}, {{ item_price }}, {{ cart_url }}, {{ size_chart_url }}, {{ coupon_code }}, {{ coupon_expiry }}
Open questions for approver:
- Counsel confirmation of soft opt-in for cart mails in CZ.
- Is "half a size up" supported by return data? (check with returns-reason-analyzer; otherwise remove)
- Mail 3 for returning customers: no code, plain reminder.
```

## Quality checklist
- Entry filter names a legal basis for every contact source; unknown sources are excluded or `BLOCKED`.
- Every mail has sender identity, visible unsubscribe, and one-click unsubscribe is noted for the ESP.
- Exit checks orders from all channels; re-entry cap and cross-flow priority are set.
- No discount before mail 3; any discount passes the d_max check and has a real expiry from policy.
- Any "was/now" price shows the 30-day lowest prior price.
- No urgency, stock or social-proof claim without a data source.
- Subjects ≤ 50 characters, bodies ≤ 120 words, one CTA each; all tags listed.
- A holdout and incremental metric are defined.
- Factual claims in mail 2 (sizing, returns, delivery) are traceable to inputs.

## Sources
- Directive 2002/58/EC (ePrivacy), Art. 5(3) and Art. 13: https://eur-lex.europa.eu/eli/dir/2002/58/oj
- ICO, How do we comply with the PECR electronic mail marketing rules: https://ico.org.uk/for-organisations/direct-marketing-and-privacy-and-electronic-communications/guidance-on-direct-marketing-using-electronic-mail/how-do-we-comply-with-the-pecr-electronic-mail-marketing-rules/
- Regulation (EU) 2016/679 (GDPR), Art. 21: https://eur-lex.europa.eu/eli/reg/2016/679/oj
- Directive 2005/29/EC (Unfair Commercial Practices), Annex I points 7 and 26: https://eur-lex.europa.eu/eli/dir/2005/29/oj
- Directive (EU) 2019/2161 (Omnibus), inserting Art. 6a into Directive 98/6/EC: https://eur-lex.europa.eu/eli/dir/2019/2161/oj
- Directive 2011/83/EU (Consumer Rights), Art. 9: https://eur-lex.europa.eu/eli/dir/2011/83/oj
- Google, Email sender guidelines: https://support.google.com/a/answer/81126
- RFC 8058, One-Click unsubscribe: https://www.rfc-editor.org/rfc/rfc8058

## License
MIT
