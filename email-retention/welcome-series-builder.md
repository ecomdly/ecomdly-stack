---
name: welcome-series-builder
owner: inboxcart
category: Email & retention
description: Designs a welcome flow from signup to first purchase with provable EU consent, double opt-in, exit rules, a margin-tested discount decision, Gmail/Yahoo sender checks and a holdout. Drafts only, never sends.
version: v1
license: MIT
updated: 2026-10-05
recommended: false
security_checked: true
url: https://ecomdly.com/skills/inboxcart/welcome-series-builder
raw: https://ecomdly.com/raw/inboxcart/welcome-series-builder.md
install: npx @ecomdly/cli add inboxcart/welcome-series-builder
---

# Welcome series builder

Produces a ready-to-build welcome flow for an ESP (Klaviyo, Ecomail, Mailchimp, SmartEmailing, Brevo and similar): the consent capture and the record that proves it, the confirmation step, three to five drafted e-mails with timing, exit and suppression rules, a decision on whether to offer a signup discount backed by margin arithmetic, a deliverability checklist for a young list, and a holdout design that measures what the flow adds. The naive welcome series promises 10% off in a popup, sends five mails in five days to everyone who typed an address, and reports "revenue attributed to the flow"; that counts buyers who would have purchased anyway, gives margin to all of them, and can collect addresses the store cannot prove consented. **The key insight:** a welcome flow is judged by incremental contribution per signup against a randomised holdout, and every mail in it rests on a consent record the store can produce on request, because under GDPR the burden of proving consent is on the store, not on the subscriber.

## When to use

- Launching e-mail marketing, migrating ESPs, or rebuilding a signup popup or footer form.
- The welcome flow "makes money" in the ESP report but nobody knows what it adds, or the signup discount is being questioned.
- Spam complaints, bounces or Gmail/Yahoo deliverability problems on a young list.
- Entering a new EU market whose consent rules or language differ.

## When not to use

- Shoppers who started checkout but did not finish: `abandoned-cart-sequence`.
- Customers who bought before and lapsed: `winback-segment-planner`.
- Asking buyers for reviews after delivery: `post-purchase-review-request`.
- Deciding the discount depth for a store-wide promotion rather than a signup offer: `promotion-profit-check`.
- Reading out the holdout test once data is in: `ab-test-readout`.

## Inputs

| input | definition | typical source |
|---|---|---|
| Signup points | every form or checkbox that adds an address: popup, footer, account, checkout | site, ESP forms list |
| Consent wording and form versions | exact text shown, pre-ticked or not, link to privacy notice | site, CMS history |
| Recipient countries | where subscribers are, for national rules | ESP, order data |
| Signups per month, confirmed and unconfirmed | volume and confirmation rate | ESP |
| First-purchase rate and lag of subscribers | share buying within 30/60 days of signup, by source | ESP joined to backend orders |
| First-order AOV and contribution margin | ex VAT, after returns, before ad cost | backend, `break-even-roas-calculator` inputs |
| Sending domain setup | SPF, DKIM, DMARC records, From domain, ESP's one-click unsubscribe support | DNS, ESP |
| Spam rate and domain reputation | Google Postmaster Tools, ESP complaint report | Postmaster Tools, ESP |

Optional: brand voice guide; product categories and bestsellers for content; existing flow performance; competitor signup offers the owner wants considered.

If an input is missing, stop and ask. Never use an industry average open rate, conversion rate or "typical" welcome uplift.

## Best practices

### Consent you can prove

1. **Consent is the basis for non-customers.** ePrivacy Directive 2002/58/EC Art. 13(1) allows direct-marketing e-mail only with prior consent. The Art. 13(2) exception (the "soft opt-in") covers only contact details obtained from a customer in the context of a sale, used for the store's own similar products, with a clear, free and easy chance to object at collection and in every message. A newsletter signup without a purchase is not a sale, so a welcome flow for leads needs consent.
2. **Consent must be provable and as easy to withdraw as to give.** GDPR Art. 4(11) requires a freely given, specific, informed and unambiguous indication by a statement or clear affirmative action; Art. 7(1) puts the burden of demonstrating it on the controller; Art. 7(3) requires withdrawal to be as easy as giving. So no pre-ticked boxes, no bundling with terms, and store a record per subscriber: timestamp, signup point, form version with the exact wording, confirmation timestamp, and the source of any later change.
3. **National law implements Art. 13 and can differ.** In Czechia, zákon 480/2004 Sb. § 7(2) requires prior consent, § 7(3) carries the customer exception, and § 7(4)(a) requires the mail to be clearly marked as a commercial communication. In Germany, § 7(2) UWG requires prior express consent and § 7(3) sets the existing-customer conditions. For other countries, make the national rule an input and route it to the store's counsel.
4. **Double opt-in is a practice, not an EU-wide legal requirement.** Neither GDPR nor the ePrivacy Directive prescribes it, but it is the most practical proof that the address owner consented, and it keeps typos, bots and spam traps off the list. In Germany it is widely treated as the standard way to document e-mail consent; for German recipients use it by default and let counsel confirm. Keep the confirmation mail free of advertising, because it goes out before consent is complete.

### Flow design

5. **Mail 1 is sent on confirmation, not on a schedule.** It delivers anything promised at signup and nothing else critical; later mails carry the brand story, bestsellers or category guidance, social proof from real reviews, and practical facts (delivery, returns, payment).
6. **Exit rules beat timing.** Exit on first purchase (check the order-sync lag of the ESP, and suppress buyers who ordered as guests with the same address), on unsubscribe, on hard bounce and on spam complaint. A mail sent after the subscriber bought is the most common welcome-flow error.
7. **Space mails by the store's own first-purchase lag.** If most subscribers who buy do so within N days, put the decision mails inside N and stop soon after; drafted timings are a starting point to test, not a rule.
8. **One welcome per person.** Suppress existing customers and subscribers already inside another flow (`abandoned-cart-sequence` takes priority while a cart is open), so the same person does not get two offers on one day.

### The discount decision

9. **A signup discount pays only if it creates more contribution than it gives away.** With first-order contribution CM and discount d (ex VAT, on the order), the discounted arm must reach first-purchase rate ≥ rate_without × CM ÷ (CM − d). Below that, the extra orders do not cover the margin handed to buyers who would have bought anyway.
10. **Test it, do not assume it.** Run no-code vs code arms inside the flow; alternatives worth testing are free shipping (cost = carrier cost on those orders) or no incentive with stronger content.
11. **Codes leak.** Single-use codes per subscriber, an expiry date, and exclusion of already-discounted items keep the cost to the arithmetic above.

### Deliverability for a young list

12. **Meet the Gmail and Yahoo sender requirements before volume grows.** Gmail defines a bulk sender as close to 5,000 or more messages to personal Gmail accounts in 24 hours, and that status is permanent. Bulk senders need SPF and DKIM, a DMARC policy (p=none is accepted) with the From domain aligned, one-click unsubscribe for marketing mail plus a visible unsubscribe link, and a spam rate in Postmaster Tools below 0.30% (Google recommends below 0.10%). Yahoo asks to honour unsubscribes within 2 days; Gmail recommends 48 hours.
13. **One-click means RFC 8058.** The message carries `List-Unsubscribe` with an HTTPS URI and `List-Unsubscribe-Post: List-Unsubscribe=One-Click`, both covered by a valid DKIM signature, and the endpoint must not redirect. Check that the ESP sets these on flow mails, not only on campaigns.
14. **Never import bought or scraped lists into the flow.** They fail the consent proof and raise spam complaints that hurt every later mail.

### Measure against a holdout

15. **Randomise at signup.** A fixed share (sized with `ab-test-readout`) receives only the confirmation and anything promised at signup, nothing else. Primary metric: first-purchase contribution per signup within the window the store's lag supports, net of returns. Guardrails: unsubscribe and spam-complaint rate per mail.

## Process

1. **Inventory signup points** and their wording; flag pre-ticked boxes, bundled consent and forms without a privacy-notice link.
2. **Set the legal basis per segment and country:** consent for leads; soft opt-in only for checkout customers who were offered an opt-out; national rule as input.
3. **Define the consent record** fields and where the ESP stores them; decide single or double opt-in per country (rule 4).
4. **Pull first-purchase rate, lag and contribution** for subscribers by signup source.
5. **Draft the flow:** trigger, 3–5 mails with subject, preview text, purpose and body copy, timing from step 4, exit and suppression rules.
6. **Run the discount arithmetic** (rule 9) and propose arms.
7. **Check deliverability** (rules 12–13) with DNS records and a test mail's headers.
8. **Specify the holdout** and the readout date; hand the plan and drafts to the owner for build and approval.

## Pitfalls and edge cases

- **Promised discount in the holdout.** If the popup promises a code, the holdout still gets it; only the follow-up mails are withheld.
- **Checkout checkbox subscribers.** Buyers enter as customers, not leads; route them to post-purchase flows, not the "first purchase" welcome.
- **Unconfirmed double opt-in addresses** cannot be mailed marketing; do not resend the confirmation repeatedly.
- **Attribution in the ESP** counts any purchase after a click or open; it is not incrementality.
- **Shared or role addresses** (info@, objednavky@) complain more; flag them, do not drop them silently.
- **Language.** Send in the storefront language of signup; Czech mail to Slovak subscribers is acceptable only if the owner decides so.

## Rules

- The agent drafts copy, settings and the plan; it never sends, schedules, activates a flow, imports contacts or changes DNS on its own.
- No pre-ticked consent, no bought lists, no marketing in the confirmation mail.
- Legal statements are limited to the verified rules above; anything country-specific goes to the store's counsel.
- Every rate, AOV and margin comes from the store's data; missing ones are asked for.

## Output format

```
WELCOME SERIES PLAN · <store> · countries <list> · prepared <date>
Legal basis: leads = consent (<single|double> opt-in) · checkout customers = <consent|soft opt-in>
Consent record: <fields> · stored in <ESP field/tag>
Signup points: <point> — wording "<text>" — issues <list>
Flow: trigger <event> · exits <purchase, unsubscribe, bounce, complaint> · suppress <list>
  Mail <n> · <timing> · subject "<x>" · preview "<x>" · purpose <x> · draft <body>
Discount: CM <x>, d <x>, rate without <x>% → break-even rate with code <x>% · arms <list>
Deliverability: SPF <ok>, DKIM <ok>, DMARC <policy, aligned>, one-click <ok>, spam rate <x>%
Holdout: <share>% · metric <contribution per signup, <n> days> · guardrails <x> · readout <date>
Missing inputs: <list>
```

## Worked example

Illustrative numbers only. Czech home-goods store, first order AOV EUR 50.00 ex VAT, contribution after returns 40% = 20.00. Proposed code: 10% off, d = 5.00, so CM with code = 15.00.

```
Break-even rate with code = rate without × 20.00 ÷ 15.00 = rate without × 1.333
Test, 4 000 signups randomised:
  Holdout   400 · 16 first orders · 4.0% · contribution/signup 0.04 × 20 = 0.80
  No code 1 800 · 108 · 6.0% · 0.06 × 20 = 1.20
  Code    1 800 · 135 · 7.5% · 0.075 × 15 = 1.125
Break-even for code arm: 6.0% × 1.333 = 8.0% > 7.5% observed
```

The flow without a code adds 1.20 − 0.80 = 0.40 per signup over the holdout. The code arm sells 25% more first orders (135 ÷ 108) yet earns 0.075 less per signup than no code, because it falls short of the 8.0% break-even. With 16 orders in the holdout the interval is wide; confirm with `ab-test-readout` before acting.

## Quality checklist

- Every signup point has unticked, specific wording and a stored consent record.
- Soft opt-in used only for customers from a sale, for similar own products, with an opt-out at collection and in each mail.
- Country rules are inputs; Czech and German rules are not presented as EU-wide.
- Exit on purchase works with the ESP's sync lag and guest orders.
- Discount justified by the break-even rate, not by attributed revenue.
- SPF, DKIM, DMARC, RFC 8058 headers and spam rate checked.
- Holdout defined before launch; nothing sent or activated by the agent.

## Sources

- EUR-Lex, Regulation (EU) 2016/679 (GDPR), Art. 4(11), Art. 7(1) and 7(3): https://eur-lex.europa.eu/eli/reg/2016/679/oj
- EUR-Lex, Directive 2002/58/EC (ePrivacy), Art. 13 as amended by Directive 2009/136/EC: https://eur-lex.europa.eu/eli/dir/2002/58/oj
- e-Sbírka, Zákon č. 480/2004 Sb., o některých službách informační společnosti, § 7: https://www.e-sbirka.cz/sb/2004/480
- Bundesministerium der Justiz, Gesetz gegen den unlauteren Wettbewerb (UWG), § 7: https://www.gesetze-im-internet.de/uwg_2004/__7.html
- Google Workspace Admin Help, Email sender guidelines: https://support.google.com/a/answer/81126
- Google Workspace Admin Help, Email sender guidelines FAQ: https://support.google.com/a/answer/14229414
- Yahoo Sender Hub, Sender best practices: https://senders.yahooinc.com/best-practices/
- IETF, RFC 8058, Signaling One-Click Functionality for List Email Headers: https://www.rfc-editor.org/rfc/rfc8058

## License
MIT
