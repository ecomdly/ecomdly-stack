---
name: checkout-friction-audit
owner: checkoutlab
category: Conversion & UX
description: Walks the checkout step by step against known friction points — forced accounts, late shipping costs, surplus form fields — and ranks fixes by impact and effort.
version: v2
license: MIT
updated: 2026-09-02
recommended: false
security_checked: true
url: https://ecomdly.com/skills/checkoutlab/checkout-friction-audit
raw: https://ecomdly.com/raw/checkoutlab/checkout-friction-audit.md
install: npx @ecomdly/cli add checkoutlab/checkout-friction-audit
---

# Checkout friction audit

Large-scale checkout research keeps finding the same causes of abandonment: extra costs shown too late, forced account creation, too many fields, and not trusting the site with a card. This skill checks your checkout for each of them and tells you what to fix first.

## When to use
- Checkout conversion (begin_checkout → purchase) below the store's own history or under ~45%.
- Before and after a checkout redesign or a platform migration.

## Input
Screenshots or a written walkthrough of every checkout step on desktop and mobile, the field list per step, shipping and payment options with costs, and if available GA4 step drop-off (begin_checkout, add_shipping_info, add_payment_info, purchase).

## Checklist
**Costs and delivery**
- Shipping cost or a reliable estimate visible in the cart, before any personal data is asked.
- Free-shipping threshold shown with the remaining amount.
- Delivery date or range per method, not just carrier names.
- Taxes and duties stated for cross-border orders.

**Account and flow**
- Guest checkout offered and at least as prominent as sign-in. Account creation offered after the order, with a password field only.
- Step count visible; no step that only confirms data already entered.
- Back navigation keeps entered data.

**Forms**
- Target 7–8 fields for a standard order. Common surplus: separate first/last name, "Address line 2" expanded by default, company, title, phone marked required with no reason given.
- Billing address = shipping by default (checkbox).
- Postcode-based address lookup or autocomplete; correct `autocomplete` attributes and mobile keyboards (numeric for card, tel for phone).
- Inline validation after the field loses focus, error messages that say how to fix it.

**Payment and trust**
- Wallets (Apple Pay, Google Pay) and local methods for the market.
- Card fields visually secured; total with all costs repeated next to the pay button.
- Coupon field collapsed to a link so it does not send people off to hunt for codes.

## Scoring
Each finding: severity (1–3) × reach (share of checkouts affected, from data or estimated and labelled as such) → priority. Effort as S/M/L. Report only what you observed; mark anything not visible in the input as "not verified".

## Output format
```
| # | Step     | Finding                                   | Sev | Reach | Effort | Fix |
| 1 | Cart     | Shipping cost first shown at step 3       | 3   | 100%  | S      | Show estimate by postcode in cart |
| 2 | Details  | Account required, no guest option         | 3   | ~60%  | M      | Add guest checkout, offer account after purchase |
| 3 | Details  | 14 fields, phone required without reason  | 2   | 100%  | S      | Cut to 8, explain phone use |
Not verified: 3-D Secure flow (no screenshots)
```

## License
MIT
