---
name: merchant-center-disapproval-fixer
owner: cartlift
category: Product feeds
description: Turns a Google Merchant Center issue export into a fix plan sorted by lost revenue: root cause per issue (feed, site, account setting or policy), owner, and whether a review is needed. For store owners and PPC specialists.
version: v4
license: MIT
updated: 2026-09-29
recommended: true
security_checked: false
url: https://ecomdly.com/skills/cartlift/merchant-center-disapproval-fixer
raw: https://ecomdly.com/raw/cartlift/merchant-center-disapproval-fixer.md
install: npx @ecomdly/cli add cartlift/merchant-center-disapproval-fixer
---

# Merchant Center disapproval fixer

Produces a prioritised fix plan for Google Merchant Center item and account issues: each issue group gets the attribute to change, the system that owns it, the fix type, and the order to do it in. The naive approach sorts by item count, so the team spends a morning on 400 zero-click accessories while the bestseller stays dark, or "fixes" a price mismatch by editing the feed to a price the site does not show, which turns a product issue into a misrepresentation risk. This skill sorts by what the issue costs, traces every issue back to its source of truth, and separates data fixes from policy decisions that need a human.

## When to use

- Diagnostics or the "Needs attention" tab shows a jump in disapproved or limited items.
- An account-level warning email arrived and the warning period is running.
- Weekly hygiene on the item-issues export, before anyone edits the feed by hand.

## When not to use

- The account is suspended for misrepresentation or another egregious policy. That is an appeal with evidence about the business, not a feed fix; this skill can only list what the appeal must address.
- The problem is identifiers only (invalid GTIN, `identifier_exists` conflicts). Use `gtin-identifier-auditor`, which does the check-digit and prefix work properly.
- The goal is better titles and descriptions for performance, not fixing issues. Use `product-feed-optimizer`.
- Heureka, Zboží.cz or Idealo errors. Use `comparison-feed-mapper`.

## Inputs

Required:

- **Issue export.** Merchant Center > Products > Needs attention > download (CSV) of all affected products, or the per-issue download from the Issue Details Page. Needed columns: item/offer ID, issue name or code, affected attribute, status (disapproved / warning / limited), affected destinations (Shopping ads, free listings, etc.) and countries.
- **Feed rows** for the affected IDs, from the primary data source (the file or platform export Merchant Center fetches), plus any supplemental sources and feed rules. A fix applied to the primary feed is overwritten if a supplemental source or rule sets the same attribute.
- **Performance per item**, last 30 days: clicks and conversion value. Google Ads > Products report (segment by item ID), or Merchant Center performance reports. Use the Issue Details Page's estimated lost clicks when present.

Optional but high-value:

- Landing page HTML or a crawl of `link` URLs capturing HTTP status, final URL after redirects, visible price, visible availability, and the `Offer` JSON-LD (`price`, `priceCurrency`, `availability`).
- Account shipping settings (services, countries, rates, delivery times) and return policy settings.
- Whether automatic item updates (Automations) are on for price, availability and condition.

If performance data is missing, sort by item count and label the sort as a fallback. If feed rows are missing, you can group and describe issues but cannot name the fix per item; say so. If no crawl is available, mismatch findings are "reported by Google, not re-verified".

## Best practices

### Triage order

1. **Account-level before item-level.** Account warnings come with a fixed warning period, per Google 7 or 28 calendar days depending on the violation; if the issue is not fixed and no review is requested, the account is reviewed at the end of the period and suspended if issues remain. An account suspension disables every product, so any account-level item jumps the queue regardless of item-level value.
2. **Disapproved before warning/limited.** Google: disapproved products stop showing; products with warnings keep showing with possibly limited performance, and unresolved warnings can lead to disapprovals.
3. **Within each tier, sort by cost:** `priority_value = Σ(30d conversion value of affected items)`, tie-break on clicks. For items with no history (new products), use Google's lost-click estimate if present, otherwise count them separately; zero history is not zero value for a launch.
4. **Destination and country matter.** An issue that blocks Shopping ads in the main market outranks one that limits free listings in a secondary country with the same item count.

### Fix the source of truth, never the symptom

5. **Price mismatch.** Google verifies the data source against the landing page and structured data. Establish which of three values is correct: feed price, visible page price, JSON-LD `offers.price`. Fix the wrong system, not the feed by default. Usual causes: a site sale without `sale_price` in the feed, a daily fetch lagging a price change, net-of-VAT export, JSON-LD showing another variant's price.
6. **Sale price rules.** When a product is on sale, keep `price` as the regular price and add `sale_price`; the page must show both prices and checkout only the sale price. Add `sale_price_effective_date` in ISO 8601 with time and timezone (`2026-10-01T00:00+0200/2026-10-08T23:59+0200`); without a timezone Google defaults to UTC, which in the EU means the sale starts or ends 1–2 hours off from the site. Without the date attribute the sale price applies immediately and stays until removed.
7. **Availability mismatch.** Feed, page and checkout must agree; Google disapproves if the page or checkout shows out of stock while the feed says `in_stock`, and also the reverse. Do not use `out_of_stock` to hide an item that is actually for sale; use `excluded_destination` or `pause` (pause stops ads for up to 14 days). For `preorder` and `backorder`, `availability_date` is required and the date should be visible on the page. Discontinued products are removed from the feed, not set to out of stock.
8. **Automatic item updates** read the page (structured data first, then Google's extractors) and can correct price, sale price, availability and condition. Google states they fix sporadic problems for a small share of products, may not work when prices change more than once per day, may disapprove instead of update, and may stop if mismatches are too numerous. Recommend turning them on as a safety net, never as the fix for a systemic export problem. Recommend turning price updates off if the page shows multiple strikethrough prices.
9. **Preemptive item disapproval (PID).** When Google sees a pattern of price or availability mismatches, it may disapprove products likely to violate before checking each one; a review is needed to clear it. Treat a sudden wave of mismatch disapprovals across unrelated products as a systemic feed-timing issue, fix the pipeline, then request review.

### Other common issue families

10. **Missing shipping.** Fix at account level (shipping settings per country) unless a product genuinely needs an override via the `shipping` attribute (bulky, fragile). Item-level shipping overrides account settings for that item, so a stale item-level value silently wins.
11. **Images.** `image_link` must show the product without promotional overlays, watermarks, logos, or borders, and must not be a placeholder. Size: Google's spec states at least 500 × 500 px, announced as the requirement for all products from 31 January 2027; Google recommends 1500 × 1500 px or larger, no more than 64 megapixels and 16 MB. Move lifestyle shots with text to `additional_image_link` only if they are also clean; the overlay rule applies to all images Google shows. Merchant Center offers automatic image improvements to remove promotional overlays; it is a stopgap, the clean packshot is the fix. When replacing an image, use a new URL: Google says a new URL is typically recrawled within 24–72 hours, the same URL can take up to 6 weeks.
12. **Landing page issues.** `link` must resolve to the specific product page (not a category, search or homepage), be crawlable (robots.txt must not block Googlebot or Googlebot-Image), load without login or blocking interstitials, and show title, price, availability and a buy button. Report the HTTP status and final URL. Geo-redirects that send Googlebot to another country store are a common hidden cause.
13. **Identifiers.** Invalid GTIN, restricted-range GTIN, or `identifier_exists=false` on a product Google believes has a GTIN. Hand off to `gtin-identifier-auditor`; do not attempt GTIN fixes here.
14. **Policy disapprovals** (restricted products, health claims, counterfeit, adult, dangerous products, misrepresentation). These need a human decision: remove the item, change the product page claim, or appeal with evidence. Bulk appeals are available for some issues, but Google warns that appealing a set containing real violations can fail and limit further appeals; the violating items must be removed first.

### Review mechanics

15. Editing data through the normal upload triggers automatic re-review of the affected products. Manual review requests or appeals are for when you disagree or when Google requires one (PID, account warnings). Google states crawls after a review request usually finish within 24–48 hours and account reviews typically take about 7 business days; set expectations accordingly.
16. If the store uses a platform integration (Shopify, Shoptet, WooCommerce plug-in), the fix and sometimes the review request happen in that platform. Name the platform setting, not a Merchant Center field the team cannot edit.

## Process

1. **Load and normalise.** Join the issue export to feed rows on offer ID and to performance data on item ID. Flag IDs that are in the issue export but missing from the feed (deleted products still in Merchant Center, or ID format drift between feed and Ads such as case changes or prefixes).
2. **Split account vs item.** List account-level issues first with the warning deadline date (absolute date), required action and owner.
3. **Group item issues** by issue name, attribute and status. One row per group.
4. **Score:** for each group compute items, 30d clicks, 30d conversion value, and share of total account conversion value. Sort per the triage order above.
5. **Diagnose each group.** Pick 1–3 example items. For mismatches, compare feed vs page vs JSON-LD for the example and state which system is wrong. Decision rule: if the page and JSON-LD agree and the feed differs, the feed export is wrong; if the page and feed agree and JSON-LD differs, the theme markup is wrong; if the page differs from both, check for a running promotion or a caching layer.
6. **Name the fix** per group: attribute, correct value source, owning system (ERP, e-shop admin, theme, feed tool, Merchant Center setting), fix type (feed edit, site edit, account setting, policy decision), and whether a review request is needed after the fix.
7. **Check override layers.** For each proposed feed edit, confirm no supplemental source or feed rule overrides that attribute.
8. **Write the plan** in the output format, including missing data and what could not be verified.

## Pitfalls and edge cases

- **Timezone and fetch timing.** A daily feed fetch at 02:00 and a sale that starts at 00:00 produce two hours of mismatches every campaign. Fix with `sale_price_effective_date`, a second scheduled fetch, or Content API/Merchant API updates, not by editing prices by hand.
- **Variant pages.** One URL for all sizes, with the default variant priced differently from the feed variant. Link each variant to a URL that preselects it (URL parameter), otherwise price and availability checks compare the wrong variant.
- **Rounding.** Google rounds to two decimals; a page showing 1 299 Kč and a feed showing 1298.99 CZK is a mismatch.
- **Multi-country feeds.** The same item can be approved in one country and disapproved in another for policy reasons; group by country when the issue is country-specific.
- **Do not delete to hide.** Deleting a disapproved bestseller from the feed removes the warning and the revenue. Only discontinued or policy-violating items should be removed.

## Rules

- Read-only. The output is a plan for a human. Never submit feed changes, request reviews, file appeals or change account settings without explicit confirmation from the owner for that specific action.
- Never set a feed price or availability to a value the landing page does not show. The page and checkout are what the customer sees; the feed follows them.
- Never guess a GTIN, price, stock level or policy outcome. Unknowns are listed as "needs owner input".
- Policy items are listed separately and marked "policy decision"; the agent does not decide whether a product is compliant, it describes the rule and the evidence.
- If the export lacks performance data, say so on the first line of the output and label the sort as "by item count".

## Output format

```text
Merchant Center fix plan: <store>, export dated <YYYY-MM-DD>, <n> products affected
Sort basis: 30d conversion value (fallback: item count, if performance data missing)

ACCOUNT-LEVEL
| issue | status | deadline | required action | owner |

ITEM-LEVEL (sorted by priority)
| # | issue | status | destinations/countries | items | 30d clicks | 30d value | root cause (example ID) | fix | owner | type | review needed |

POLICY DECISIONS (human)
| issue | items | rule | options | evidence needed |

CHECKS BEFORE APPLYING
- supplemental sources / rules overriding: <list or none>
- automatic item updates: price <on/off>, availability <on/off>

MISSING DATA / NOT VERIFIED
- <item or input>: <what is missing and effect on the plan>
```

## Worked example

Example data. Store: Czech outdoor shop, 3 140 products, CZK, feed fetched daily at 03:00. Export dated 2026-09-28, 212 affected items. 30d account conversion value 1 240 000 CZK.

Diagnosis highlights: the 38 price-mismatch items all belong to an "Autumn -15 %" promotion that started 2026-09-26 at 00:00 on the site; the feed has no `sale_price`, so the feed still shows full price. JSON-LD on the example page (SKU-4411) shows 2 550 CZK, page 2 550 CZK, feed 3 000 CZK: the feed is wrong. The 112 image issues are all from one supplier whose images carry a logo watermark.

```text
Merchant Center fix plan: example-outdoor.cz, export dated 2026-09-28, 212 products affected
Sort basis: 30d conversion value

ACCOUNT-LEVEL
| issue | status | deadline | required action | owner |
| none | | | | |

ITEM-LEVEL (sorted by priority)
| # | issue | status | destinations/countries | items | 30d clicks | 30d value | root cause (example ID) | fix | owner | type | review needed |
| 1 | Mismatched price | disapproved | Shopping ads, free listings / CZ | 38 | 4 210 | 186 000 CZK | promo live on site, no sale_price in feed (SKU-4411: feed 3000, page 2550) | export sale_price + sale_price_effective_date 2026-09-26T00:00+0200/2026-10-12T23:59+0200 | e-shop admin feed export | feed edit | no, auto re-review |
| 2 | Missing shipping (SK) | disapproved | Shopping ads / SK | 56 | 640 | 41 000 CZK | SK feed target added, no SK shipping service | add SK shipping service in account settings | marketing | account setting | no |
| 3 | Promotional overlay on image | disapproved | all / CZ | 112 | 690 | 22 500 CZK | supplier images with logo (SKU-7710) | request clean packshots, publish under new image URLs | content | feed edit | no |
| 4 | Restricted: health claim | disapproved | all / CZ | 6 | 150 | 3 100 CZK | "cures joint pain" on page (SKU-9020) | see policy decisions | owner | policy | yes, after page change |

POLICY DECISIONS (human)
| issue | items | rule | options | evidence needed |
| unapproved health claim | 6 | healthcare and medicines policy | remove claim from page and description, then request review; or remove items | final page text |

CHECKS BEFORE APPLYING
- supplemental sources / rules overriding: rule "price = price * 1" on source "CZ main"; harmless, but confirm it does not drop sale_price
- automatic item updates: price on, availability on

MISSING DATA / NOT VERIFIED
- promo end date: taken from the site banner, confirm with owner
- crawl only for example IDs; other 37 price items assumed same cause
```

Share check: item 1 is 186 000 / 1 240 000 = 15 % of 30d value, which is why it leads despite having a third of item 3's count.

## Quality checklist

- Account-level issues listed first, with absolute deadline dates.
- Sort basis stated; fallback labelled if used.
- Every mismatch fix names which of feed, page and JSON-LD is wrong, with an example ID and values.
- No fix sets a feed value the page does not show.
- Sale fixes include `sale_price_effective_date` with a timezone.
- Override layers (supplemental sources, rules, platform integration) checked for every feed edit.
- Policy items separated and not decided by the agent.
- Totals add up: group item counts sum to the affected total or the difference is explained.
- Missing data and unverified assumptions listed.

## Sources

- https://support.google.com/merchants/answer/12153802 (Issues in Merchant Center: product vs account issues, PID, review timing)
- https://support.google.com/merchants/answer/6150244 (Editorial and professional requirements; warning periods, appeals, bulk appeals)
- https://support.google.com/merchants/answer/3246284 (Automatic item updates)
- https://support.google.com/merchants/answer/4752265 (Landing page requirements)
- https://support.google.com/merchants/answer/6324448 (availability attribute)
- https://support.google.com/merchants/answer/6324471 (sale_price)
- https://support.google.com/merchants/answer/6324460 (sale_price_effective_date)
- https://support.google.com/merchants/answer/6324350 (image_link)
- https://support.google.com/merchants/answer/6324484 (shipping attribute)
- https://support.google.com/merchants/answer/6150127 (Misrepresentation policy)
- https://support.google.com/merchants/answer/7052112 (Product data specification)

## License
MIT
