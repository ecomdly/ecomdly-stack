---
name: product-schema-validator
owner: rankcraft
category: E-shop SEO
description: Validates Product, Offer, AggregateRating, shipping and return-policy JSON-LD against Google rich result requirements and the visible page, and reports every mismatch.
version: v2
license: MIT
updated: 2026-09-25
recommended: true
security_checked: true
url: https://ecomdly.com/skills/rankcraft/product-schema-validator
raw: https://ecomdly.com/raw/rankcraft/product-schema-validator.md
install: npx @ecomdly/cli add rankcraft/product-schema-validator
---

# Product schema validator

Rich results fail quietly: the markup parses, the stars never show. This skill checks product JSON-LD field by field against Google's requirements and against what the page actually says.

## When to use
- After a template change on product pages.
- When Search Console's Merchant listings or Product snippets report shows errors or warnings.

## Input
Page HTML or its JSON-LD blocks, plus the visible values: name, price, currency, availability, rating and review count, shipping and return terms shown on the page. The store's written shipping and return policy.

## Required checks
- **Product:** `name` present. For product snippets, one of `review`, `aggregateRating` or `offers`.
- **Offer (merchant listings):** `price` greater than 0, `priceCurrency` as ISO 4217, `availability` as a schema.org URL (`https://schema.org/InStock`). Variants as separate offers or `ProductGroup` with `hasVariant`.
- **AggregateRating:** `ratingValue` plus `reviewCount` or `ratingCount`; ratings must be visible on the page and come from real customer reviews.
- **Recommended:** `gtin`/`gtin13`, `brand`, `image`, `sku`, `priceValidUntil` only when the price truly ends.
- **shippingDetails:** `OfferShippingDetails` with `shippingRate`, `shippingDestination.addressCountry`, and `deliveryTime` with `handlingTime` and `transitTime` in days (`unitCode` `DAY`).
- **hasMerchantReturnPolicy:** `applicableCountry`, `returnPolicyCategory` and, for a finite window, `merchantReturnDays`; `returnMethod` and `returnFees` when known.

## Consistency rules
- Markup must equal the visible page: same price, currency, availability and rating count. A mismatch is an error, not a warning.
- Shipping and return values come from the store's written policy. If the policy is missing, report the gap; never fill in days or fees.
- Do not add review markup for reviews the page does not show.

## Output format
```
/p/speedcross-6-gtx-black
ERROR  offers.price 3490 ≠ visible 3990 Kč
ERROR  aggregateRating.reviewCount 212, page shows 48 reviews
WARN   shippingDetails missing deliveryTime.transitTime
OK     hasMerchantReturnPolicy: CZ, finite window, 30 days, free by mail
GAP    return fees not stated in written policy
```

## License
MIT
