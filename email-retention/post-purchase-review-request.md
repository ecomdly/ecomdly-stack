---
name: post-purchase-review-request
owner: inboxcart
category: Email & retention
description: Drafts review-request mails timed after delivery, not purchase, asking every customer the same way — no incentives for positive reviews, no review gating.
version: v2
license: MIT
updated: 2026-09-18
recommended: true
security_checked: true
url: https://ecomdly.com/skills/inboxcart/post-purchase-review-request
raw: https://ecomdly.com/raw/inboxcart/post-purchase-review-request.md
install: npx @ecomdly/cli add inboxcart/post-purchase-review-request
---

# Post-purchase review request

A review request that arrives before the parcel gets ignored or earns a "haven't received it yet" one-star. This skill times requests after delivery, keeps them short, and stays inside review platform and consumer protection rules.

## When to use
- Setting up or fixing a review flow in an ESP (Klaviyo, Mailchimp, Ecomail) or a review platform (Trustpilot, Google Customer Reviews, Yotpo, Judge.me).
- Review volume is low or reviews mention delivery problems the request itself caused.

## Input
Order data with the delivery event (carrier "delivered" status or estimated delivery date), product type, the review platform and its rules, the store's voice guide, and the support address.

## Timing
Count from delivery, not from purchase:
- Consumables and cosmetics: 10–14 days after delivery (they need to try it).
- Apparel and shoes: 5–7 days (after the return window decision is mostly made).
- Electronics and furniture: 14–21 days.
- No delivery event available: estimated delivery + 5 days, and say that the timing is estimated.
- One reminder at most, 7 days later, only to non-responders. Suppress both if a return or support ticket is open.

## Rules
- Ask every customer, the same way. No review gating: never route happy customers to the public platform and unhappy ones to a private form.
- No incentive tied to the review's content or rating. If the store offers anything for a review, it must go to every reviewer regardless of rating, and must be disclosed; many platforms (Google, Amazon) ban incentives completely, so check the input platform rules and flag.
- Never write, suggest or pre-fill review text, and never invent ratings or quotes.
- Name the product and show its image; one tap to a star rating if the platform supports it.
- Body under 80 words, one CTA, a line on how to reach support if something went wrong.

## Output format
```
Trigger: delivered + 7 days (apparel) · reminder +7 days · suppress if return open
Subject: How are the Speedcross 6 working out?
Body:
Hi {{ first_name }},
your {{ item_name }} arrived last week. Once you've had them on a trail or two,
would you tell other runners how they fit and grip? It takes a minute.
[Rate {{ item_name }}] -> {{ review_url }}
Something wrong with the order? Reply here and we'll sort it out.
Tags used: {{ first_name }}, {{ item_name }}, {{ review_url }}
Policy flags: none (no incentive offered)
```

## License
MIT
