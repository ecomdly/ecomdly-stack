---
name: product-page-cro-review
owner: checkoutlab
category: Conversion & UX
description: Reviews a product detail page element by element — images, price, variants, delivery, returns, reviews — and writes testable fixes, never invented product claims.
version: v2
license: MIT
updated: 2026-09-21
recommended: false
security_checked: true
url: https://ecomdly.com/skills/checkoutlab/product-page-cro-review
raw: https://ecomdly.com/raw/checkoutlab/product-page-cro-review.md
install: npx @ecomdly/cli add checkoutlab/product-page-cro-review
---

# Product page CRO review

Most product pages lose the sale on a missing answer, not on button colour. This review checks whether the page answers what a shopper needs before adding to cart, and turns every gap into a test you can run.

## When to use
- Product pages with high traffic and add-to-cart rate well below the category average.
- Template changes: review one representative page per template, not every product.

## Input
Page URL or screenshots (mobile and desktop), product data, the store's delivery and returns policy, review data if any, and if available GA4 metrics per page: views, add_to_cart rate, scroll depth.

## Checks, in mobile scroll order
1. **Above the fold:** product name, price with any unit price, primary image, variant selector and add-to-cart all visible without scrolling on a 390 px wide screen.
2. **Images:** at least 5 for considered purchases, zoomable, in-context or scale shot, every color variant shown when selected.
3. **Variants:** out-of-stock options visible but marked, not hidden; size guide linked next to the size selector; the chosen variant updates price, image and stock.
4. **Price and costs:** total cost clarity. Shipping cost or free threshold and delivery date near the button.
5. **Returns and warranty:** one line near the button with the real terms, linked to the policy.
6. **Description:** key specs scannable, the top two purchase questions answered (sizing, compatibility, materials, care).
7. **Social proof:** review count and average near the title if reviews exist; reviews filterable; negative reviews shown. Missing reviews are a finding, never something to fill.
8. **Trust and help:** payment methods, contact or chat for questions, stock information only if it is real data.
9. **Performance:** LCP under 2.5 s on mobile, images sized, no layout shift when variants load.

## Rules
- Report what is on the page; anything you cannot see in the input is "not verified".
- Recommendations may reorganise or reword existing facts, never add product facts, badges, stock counts or reviews that are not in the data.
- Each fix is a hypothesis with a metric: "Because X, changing Y will raise add_to_cart rate".

## Output format
```
Page: /p/speedcross-6-gtx (template: footwear)
Score: 6/9 checks passed
1. [High] Size guide not linked at the size selector
   Hypothesis: shoppers unsure of fit leave to search; linking the guide raises add_to_cart rate.
   Metric: add_to_cart rate, returns for size reasons
2. [Medium] Delivery date missing near button - policy says 2-3 days, show it
3. [Low] Only 3 images, no sole close-up
Not verified: LCP (no performance data)
```

## License
MIT
