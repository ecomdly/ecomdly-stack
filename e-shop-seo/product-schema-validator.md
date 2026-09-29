---
name: product-schema-validator
owner: rankcraft
category: E-shop SEO
description: Audits a product page's JSON-LD against Google's merchant listing and product snippet rules and against the visible price, stock, ratings and written shipping/return policy, then lists errors, gaps and confirmed fixes.
version: v3
license: MIT
updated: 2026-09-29
recommended: true
security_checked: true
url: https://ecomdly.com/skills/rankcraft/product-schema-validator
raw: https://ecomdly.com/raw/rankcraft/product-schema-validator.md
install: npx @ecomdly/cli add rankcraft/product-schema-validator
---

# Product schema validator

Produces a field-by-field verdict on a product page's structured data: does it meet Google's requirements for merchant listings and product snippets, and does every marked-up value match what a shopper sees on the page and what the store's written policies say. The naive check stops at "the Rich Results Test passes". That test validates syntax; it cannot see that the JSON-LD says 3 490 Kč while the page shows 3 990 Kč, or that the marked-up return window is 30 days while the terms page says 14. Those mismatches cost eligibility, trigger Merchant Center mismatches and mislead shoppers.

## When to use
- After any change to the product template, theme, review widget, pricing app or structured-data plugin.
- When Search Console's Merchant listings or Product snippets enhancement reports show new errors, warnings, or a drop in valid items.
- When Merchant Center reports price or availability mismatches between feed, landing page and markup.
- Before a sale period, to confirm strikethrough prices and sale dates are encoded correctly.

## When not to use
- Category or brand listing pages: Google's product rich results support pages focused on a single product (or its variants). Report them out of scope.
- Editorial review pages about products the site does not sell: product snippets only, no merchant-listing checks.
- Feed-only problems (disapprovals, GTIN validity across the catalog): hand off to `merchant-center-disapproval-fixer` or `gtin-identifier-auditor`.

## Inputs

Required:
- **Rendered page per URL**: HTML after JavaScript, e.g. "View tested page" in the Rich Results Test or URL Inspection (raw source misses injected markup). Extract every JSON-LD block plus any Microdata/RDFa.
- **Visible values per URL** on the default variant: name, active price and currency, crossed-out price, availability text, selected variant, rating value and count shown, number of readable reviews, shipping cost and delivery estimate, return terms.
- **Written policies**: the shipping page and return/withdrawal policy page (URL and text), with the date read.

Optional but valuable:
- Search Console Merchant listings / Product snippets exports (issue, URL, last crawled).
- Merchant Center mismatch issues and any shipping/return settings configured there.
- Organization-level markup (home or policy page), where shipping and return policies can live.
- A sample of 10–30 URLs across templates: simple, variant, out-of-stock, sale, pre-order, bundle.

If visible values are missing, do not infer them from the markup. Mark the affected checks `NOT CHECKED - visible value not provided` and ask for a screenshot or the rendered HTML. If the written policy is missing, every shipping and return value in the markup is `GAP - cannot verify against policy`.

## Best practices

### Eligibility: which feature, which rules
1. **Know the two targets.** Product snippets need `Product.name` plus at least one of `review`, `aggregateRating` or `offers`. Merchant listings need `name`, `image` and an `offers` of type `Offer` (not `AggregateOffer`), because the merchant must be the seller. Validate against both and report which each URL qualifies for.
2. **Price must be greater than zero for merchant listings** (Google's explicit difference from product snippets). A `0` or missing price on "price on request" or out-of-stock templates is an ERROR.
3. **`priceCurrency` is a three-letter ISO 4217 code** (`CZK`, `EUR`), never a symbol or "Kč". One currency per URL: Google asks for a distinct URL per currency. 
4. **`price` is a number, not a formatted string.** `3990` or `3990.00`, not `"3 990 Kč"` or `"3.990,00"`. Space and comma separators (Czech, German formats) silently corrupt prices.
5. **Active price precedence.** If both `offers.price` and `offers.priceSpecification` encode an active price, Google uses `offers.price` and ignores the specification. If they differ, report it.

### Sale and member prices
6. **Strikethrough price**: the original price goes in a `UnitPriceSpecification` with `priceType` = `https://schema.org/StrikethroughPrice`; the active (sale) price carries no `priceType`. A `priceType` on the active price is an error. `ListPrice` is only accepted for a transition period; flag it as "migrate".
7. **Sale duration**: `validFrom` for start, `validThrough` or `priceValidUntil` for end, ISO 8601 with time and timezone (`2026-11-27T00:00:00+01:00`). Start must be ≤ end. `priceValidUntil` does not apply inside a `PriceSpecification`.
8. **`priceValidUntil` in the past can suppress the listing.** Many templates hard-code a date one year ahead or a stale date. Flag any past date as ERROR and any date not tied to a real price end as WARN ("remove unless the price truly ends").
9. **EU price-reduction consistency.** In the EU, an announced price reduction must reference the lowest price in the prior 30 days (Directive 98/6/EC as amended by the Omnibus Directive). This skill does not compute that price, but if the marked-up strikethrough differs from the "was" price shown on the page, report it: the markup must equal what the page announces, and the page figure is the one a regulator will read.
10. **Member prices** use `validForMemberTier` and must not also carry `priceType`; a specification with both is ignored by Google. A member price also requires a regular active price on the Offer.

### Availability and condition
11. **One `availability` value, as a schema.org enum** (`https://schema.org/InStock`, `OutOfStock`, `BackOrder`, `PreOrder`, `LimitedAvailability`, etc.; short names also accepted). Map the store's stock states explicitly: "on order in 5–7 days" is `BackOrder`, not `InStock`.
12. **Availability must track real stock.** Cached HTML that says InStock after sell-out is the most common Merchant Center mismatch. Compare the markup against the visible stock label on the same fetch.
13. `itemCondition` (`NewCondition`, `UsedCondition`, `RefurbishedCondition`): missing on a used or refurbished item is a WARN.

### Identity and images
14. **Identifiers**: `gtin`/`gtin13` etc. in numeric form, `sku` without whitespace, `brand.name`. A GTIN in markup that differs from the feed GTIN for the same item is an ERROR; hand the catalog-wide check to `gtin-identifier-auditor`.
15. **Images** must be crawlable and indexable, represent the product, and Google recommends multiple high-resolution images (at least 50 000 pixels, width × height) in 16:9, 4:3 and 1:1. A placeholder or lazy-load stub URL is an ERROR.

### Variants
16. **Variants need distinct, preselectable URLs.** Google's variant guidance: each variant should be reachable by a URL that preselects it (path segment or query parameter). Use `ProductGroup` with `productGroupID`, `variesBy` (e.g. `https://schema.org/size`) and `hasVariant`, or variants pointing back with `isVariantOf`/`inProductGroupWithID`. The group name is generic; variant names include the differentiator.
17. **Price and availability per variant.** A single Offer with the lowest variant price on a page defaulting to a more expensive variant is a visible-price mismatch.
18. **Canonical for optional variant parameters**: Google recommends the URL without the optional variant parameter as canonical. A variant URL canonicalising to a different product is an ERROR.

### Ratings and reviews
19. **AggregateRating needs `ratingValue` and at least one of `ratingCount` or `reviewCount`.** `bestRating` defaults to 5 and `worstRating` to 1; a 10-point or 100-point scale without `bestRating` is silently wrong.
20. **Marked-up reviews must be readily available on the page.** Google's structured data policies prohibit marking up content not visible to readers; if multiple reviews are marked up, all reviews visible on the page should be marked up. A `reviewCount` of 212 against 48 readable reviews is an ERROR unless the page states and links to the full set.
21. **Ratings must come from users.** Aggregated ratings imported from other websites, or ratings curated by staff, are not eligible. A review author must be a valid name (a promo line like "50% off" is not), under 100 characters.
22. **Pros/cons markup is for editorial review pages only**; on a merchant product page flag it "no effect".

### Shipping and returns
23. **Prefer Organization-level policies.** Google recommends a global shipping policy (`hasShippingService` under `Organization`) and a global return policy (`hasMerchantReturnPolicy` under `Organization`), using offer-level markup only to override for specific products. Offer-level supports only a subset of the organization-level properties.
24. **Precedence is not "markup wins".** For both returns and shipping Google applies: Content API for Shopping settings, then settings in Merchant Center or Search Console, then product-level markup, then organization-level markup. If Merchant Center has shipping or return settings, markup changes will not change what is shown; say so instead of recommending markup edits.
25. **OfferShippingDetails, if used, needs**: `shippingDestination.addressCountry` (ISO 3166-1 alpha-2), `shippingRate` with `currency` equal to the offer currency and `value` or `maxValue` (0 = free), and one `deliveryTime` with `handlingTime` and `transitTime`, each a `QuantitativeValue` with whole-number `minValue`, `maxValue` and `unitCode` `DAY` or `d`. One `shippingRate` per `OfferShippingDetails`; multiple services mean multiple `shippingDetails` entries. `addressRegion` is only supported for a few countries (US, Australia, Japan at the time of writing); for CZ/SK use country only.
26. **MerchantReturnPolicy needs** `applicableCountry` (up to 50 codes) and `returnPolicyCategory` (`MerchantReturnFiniteReturnWindow`, `MerchantReturnNotPermitted`, `MerchantReturnUnlimitedWindow`); `merchantReturnDays` is required for a finite window. `returnFees`: `FreeReturn` and `ReturnFeesCustomerResponsibility` must not carry `returnShippingFeesAmount`; `ReturnShippingFees` should. At organization level, `merchantReturnLink` is an alternative to the country/category pair.
27. **EU withdrawal period is a floor, not the marked-up value.** EU consumers have a 14-day withdrawal right for distance contracts; a store may offer more. The markup must state the store's actual window from its written policy. `merchantReturnDays: 14` on a store whose policy says 30 is an error in the store's disfavour; `30` on a policy that says 14 is a misleading claim.
28. **Delivery estimates are commitments.** `handlingTime` and `transitTime` come from the written policy or the checkout's displayed estimate, never from typical carrier times. If the policy says "usually 2–4 working days" without splitting handling and transit, report GAP and ask; do not split it yourself.

### Implementation
29. **Server-render Product markup** in the initial HTML. Google warns that JavaScript-generated Product markup can make Shopping crawls less frequent and less reliable for fast-changing price and availability.
30. **One Product entity per page.** Theme plus plugin duplicates (one with a stale price) are a classic mismatch source; report every block.

## Process
1. **Inventory.** For each URL, list every structured-data block (format, `@type`, source if identifiable: theme, plugin, review app). If more than one Product entity exists, mark `DUPLICATE` and continue validating each.
2. **Classify the page.** Single product, variant group, listing page, or editorial. Listing or editorial pages exit with the matching note.
3. **Required-field pass.** Check properties from Best practices 1–4, 11, 16, 19, 25, 26. Missing required = ERROR; missing recommended = WARN only if it affects a feature the store wants (shipping/returns display, identifiers).
4. **Type and format pass.** Numbers are numbers, enums are schema.org values, dates are ISO 8601, codes are ISO 4217 / ISO 3166-1, `unitCode` is `DAY`/`d`.
5. **Visible-page pass.** For each of: name, active price, currency, strikethrough price, availability, rating value, rating/review count, readable reviews, shipping cost, delivery estimate, return window: compare markup to visible value. Rule: any difference in price, currency, availability, rating value or count is ERROR. Rounding (`3990` vs `3 990,00 Kč`) is OK.
6. **Policy pass.** Compare shipping and return markup to the written policy text. Any value not found in the policy is GAP; any contradiction is ERROR.
7. **Precedence check.** If Merchant Center settings or Content API shipping/return settings exist, note that they override markup and compare against those instead.
8. **Variant pass.** For variant groups, open at least two variant URLs; check each preselects its variant and that its price and availability match its Offer.
9. **Search Console reconciliation.** Map each reported issue to a finding; unmatched issues go to "unexplained" with URL and last-crawl date.
10. **Propose corrected JSON-LD** only for fields with a confirmed value. Fields without a confirmed value stay out of the proposal and appear in the GAP list.
11. **Prioritise**: ERRORs affecting all URLs of a template first, then per-URL ERRORs, then WARNs, then GAPs.

## Pitfalls and edge cases
- **Price including vs excluding VAT.** B2C pages in the EU show gross prices; a B2B template may mark up net. The marked-up price must be the one displayed to the consumer.
- **Per-unit pricing.** Goods sold by weight/volume/length can use `referenceQuantity` inside a `UnitPriceSpecification`; the active price stays the pack price. Do not replace the pack price with the unit price.
- **Out-of-stock pages with price 0 or no Offer.** Keep the last selling price with `OutOfStock` if the page still shows it; removing the Offer removes merchant-listing eligibility.
- **Bundles**: a bundle is its own Product; component GTINs do not belong on it.
- **Review widgets loaded in an iframe** from a review platform: the stars may be visible, but the review text may not be on the page. If only the aggregate is shown, marking up individual reviews is not supported by the page.
- **Store-wide ratings** (shop reviews such as Heureka shop ratings) placed on product pages as product `aggregateRating`: that is a rating of the store, not the product. ERROR.
- **Stale cache after price change**: re-fetch before reporting; note the fetch timestamp on every price finding.
- **Rich Results Test passes, report still errors**: the report reflects the last crawl; check "last crawled" before concluding the fix failed.

## Rules
- Read-only. Never edit the theme, plugin, feed or Merchant Center settings; output proposals for a human or developer to implement.
- Never invent a value. Shipping rates, delivery days, return windows, fees, GTINs and ratings come from the page, the written policy or the store's systems. Unknown = GAP.
- Never propose marking up reviews, ratings or prices the page does not show.
- Never recommend changing the store's actual policy to match the markup; recommend changing the markup to match the policy, and flag the policy question separately for the owner.
- State the fetch date and source (rendered vs raw HTML) for every URL.
- Legal points (withdrawal period, price-reduction rules) are flagged for the owner to confirm; this skill does not give legal advice.

## Output format
```text
Product schema audit - <store> - fetched <YYYY-MM-DD HH:MM TZ> - source: rendered HTML
Scope: <n> URLs, templates: <list>
Precedence: Merchant Center shipping settings <present/absent>; return settings <present/absent>

<url>  [type: single | variant group | out of scope]  eligible: merchant listings <yes/no>, product snippets <yes/no>
ERROR  <property path>  markup <value> ≠ <source> <value>
WARN   <property path>  <issue and effect>
GAP    <property path>  <what is missing from which source>
OK     <property path or group>  <short confirmation>

Template-level findings (affect all URLs of template <name>):
1. <finding> - fix: <what to change>

Proposed JSON-LD changes (confirmed values only):
<property path>: <old> -> <new>  (source: <page/policy/Merchant Center>)

Search Console issues not explained by findings:
- <issue> <url> last crawled <date>

Questions for the store owner:
- <question>
```

## Worked example
Example data, not a real store. Shoe shop, CZ market, two URLs, fetched 2026-09-28 10:15 CEST. Written return policy: 30 days, free return via Zásilkovna drop-off. Shipping page: CZ delivery 89 Kč, free from 2 000 Kč; "dispatched next working day, delivered in 1–2 working days". Merchant Center: no shipping or return settings.

```text
Product schema audit - example-shoes.cz - fetched 2026-09-28 10:15 CEST - source: rendered HTML
Scope: 2 URLs, templates: product
Precedence: Merchant Center shipping settings absent; return settings absent

/p/speedcross-6-gtx-black  [type: variant group]  eligible: merchant listings yes, product snippets yes
ERROR  offers.price  markup 3490 ≠ visible 3990 Kč (3490 is the size 38 variant; page defaults to 42)
ERROR  aggregateRating.reviewCount  markup 212 ≠ page 48 readable reviews (212 includes shop ratings)
ERROR  offers.priceValidUntil  2025-12-31 is in the past - remove, no sale running
WARN   image  one 800x600 image (480 000 px, 4:3 only) - add 1:1 and 16:9
GAP    shippingDetails.deliveryTime  policy says "next working day" dispatch and "1-2 working days" delivery; confirm handling 0-1 / transit 1-2 before marking up
OK     hasMerchantReturnPolicy  CZ, finite window, 30 days, FreeReturn, ReturnByMail - matches policy

/p/trail-sock-merino  [type: single]  eligible: merchant listings no, product snippets no (offer invalid, no rating)
ERROR  offers.price  "249 Kč" is a string with currency symbol - use 249
ERROR  offers.priceCurrency  "Kč" - use CZK
OK     availability  InStock = page "Skladem"

Template-level findings (affect all URLs of template product):
1. Theme and review plugin both output Product JSON-LD - keep the theme block, disable plugin Product output, keep its AggregateRating only after count fix.

Proposed JSON-LD changes (confirmed values only):
offers.price (speedcross, variant 42): 3490 -> 3990  (source: page)
offers.priceCurrency (trail-sock): "Kč" -> "CZK"  (source: page)

Search Console issues not explained by findings: none

Questions for the store owner:
- Is the 89 Kč CZ rate the same for all carriers shown at checkout?
```

## Quality checklist
- Every ERROR names both values and the source of the visible or policy value.
- No proposed value lacks a source; unknowns are GAP.
- Each URL has an eligibility verdict for merchant listings and product snippets.
- Merchant Center / Content API precedence was checked before recommending shipping or return markup edits.
- Duplicate Product entities are reported.
- Variant URLs were opened, not assumed.
- Fetch timestamp and rendered-vs-raw source stated.
- Listing and editorial pages were excluded with a reason, not audited as products.

## Sources
- https://developers.google.com/search/docs/appearance/structured-data/merchant-listing
- https://developers.google.com/search/docs/appearance/structured-data/product-snippet
- https://developers.google.com/search/docs/appearance/structured-data/product-variants
- https://developers.google.com/search/docs/appearance/structured-data/review-snippet
- https://developers.google.com/search/docs/appearance/structured-data/return-policy
- https://developers.google.com/search/docs/appearance/structured-data/shipping-policy
- https://developers.google.com/search/docs/appearance/structured-data/sd-policies
- https://developers.google.com/search/docs/specialty/ecommerce/designing-a-url-structure-for-ecommerce-sites
- https://schema.org/Product
- https://schema.org/MerchantReturnPolicy

## License
MIT
