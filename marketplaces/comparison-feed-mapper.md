---
name: comparison-feed-mapper
owner: listwise
category: Marketplaces
description: Maps a store catalog to Heureka.cz/.sk and Zboží.cz XML and idealo offer data element by element, including DELIVERY_DATE, carrier IDs and prior-price rules, and reports every row that would be rejected or badly paired. For Czech and EU e-shops.
version: v4
license: MIT
updated: 2026-09-29
recommended: false
security_checked: true
url: https://ecomdly.com/skills/listwise/comparison-feed-mapper
raw: https://ecomdly.com/raw/listwise/comparison-feed-mapper.md
install: npx @ecomdly/cli add listwise/comparison-feed-mapper
---

# Comparison feed mapper

Produces element-by-element mappings from a store catalog to comparison-shopping feeds: Heureka.cz / Heureka.sk XML, Zboží.cz XML (Seznam Nákupy), and idealo offer data, plus a row-level report of what will be rejected, shown incorrectly, or paired badly. These channels rank and pair what they can parse: a product name with shop text, a delivery time the site does not show, or a variant without its own URL ends up unpaired, hidden in price comparison, or banned. The naive approach reuses the Google feed with renamed tags; it fails because the channels differ on the meaning of delivery time, how variants are grouped, category taxonomy, carrier IDs and discount display rules. This skill maps each element to the channel's own specification, derives delivery values from real data, and reports every row it cannot fill honestly.

## When to use

- Setting up a Heureka, Zboží.cz or idealo feed, or migrating e-shop platforms (Shoptet, Upgates, WooCommerce, Shopify) where the built-in export needs checking.
- Heureka or Zboží reports unpaired products, feed errors, or wrong availability.
- Adding a market (Heureka.sk, idealo) to an existing CZ setup.

## When not to use

- Google Merchant Center feeds: `product-feed-optimizer`, `merchant-center-disapproval-fixer`.
- Only identifier problems: `gtin-identifier-auditor` first, then map.
- Marketplaces where you sell through the platform (Allegro, Kaufland, Amazon): `marketplace-listing-adapter`.

## Inputs

Required, one row per sellable variant:

- Internal ID, product name, manufacturer, product line/model, variant attributes (size, colour, capacity), EAN, manufacturer part number, price incl. VAT per market currency, VAT rate, product URL (per variant if available), image URLs, stock quantity, supplier lead time in days, category path, variant group ID, condition (new, used, open box, refurbished).
- Delivery methods per market: carrier, service (home delivery or pick-up point), price prepaid, price cash on delivery (or "COD not offered"), exclusions (oversized items, carriers that cannot take some products).
- Order handling: dispatch cut-off time, working days, how the site phrases availability on the product page.

Sources: the platform's product export, shipping settings, and a sample of product pages.

Optional: existing feeds and the channel's error reports (Heureka admin feed diagnostics, Zboží/Sklik feed diagnostics and validator), channel category lists (Heureka category tree, Zboží `categories.json`), price history for the last 30 days (for discount elements), energy label data (EPREL registration numbers).

If lead time, delivery prices or EAN are missing, the element is left out and the row reported; the channel shows "info in shop" or fails pairing, which is better than a wrong claim.

## Best practices

### Shared rules across channels

1. **Stable IDs.** Heureka `ITEM_ID` and Zboží `ITEM_ID`: up to 36 characters (Heureka) of `a-z A-Z 0-9 _ -`, no diacritics, unique per offer and never reused. Heureka uses it to recognise the product for reviews and the availability feed even when the URL changes. Zboží: keep it when the name or URL changes, but change it when the offer becomes a different product (including a different colour or size), so it unpairs from the wrong product card.
2. **One offer per variant.** Heureka: a T-shirt in 5 sizes and 10 colours is 50 `SHOPITEM`s. Each needs its own URL; Heureka accepts a `#fragment` per variant when variants share one page (`.../product#43`). Zboží: every offer including variants must have a unique URL without diacritics or spaces.
3. **Price = what the customer pays on the page.** Final price incl. VAT and statutory fees, per piece, no quantity discounts, max two decimals, no dot as thousands separator. If a voucher element is sent, the price rules differ per channel (see below).
4. **Pairing names.** Pairing depends on manufacturer + line + model + distinguishing variant attributes in the name and on EAN. Shop text ("free delivery", "gift", "novinka") never goes in `PRODUCTNAME`.
5. **Images.** Clean product image, no watermark, shop logo, discount badge or promotional text. Stable URL per image; when the image changes, change the URL (both Heureka and Zboží keep the old image otherwise).
6. **Only sellable items.** Heureka: the export must not contain unsellable, unavailable or sold-out items; long-term unavailable items with unknown lead time must not be in the feed.

### Heureka.cz / Heureka.sk

7. **Required and key elements.** Required: `ITEM_ID`, `PRODUCTNAME`, `PRICE_VAT`, `URL`. Key for quality: `CATEGORYTEXT`, `DELIVERY`, `DELIVERY_DATE`, `IMGURL`, `ITEMGROUP_ID`. Tags must be upper case; lower-case tags are ignored. UTF-8.
8. **`PRODUCTNAME`** max 200 characters: manufacturer, line and product number, and every variant parameter (colour, size, flavour, quantity). `PRODUCT` may add a short note (for example pick-up location). `MANUFACTURER` is used for filters, not pairing, so the manufacturer must also be in `PRODUCTNAME`.
9. **`CATEGORYTEXT`**: full path, ideally in Heureka's tree, starting `Heureka.cz | ...` (`Heureka.sk | ...` for Slovakia).
10. **`DELIVERY_DATE`** = days from payment (for cash on delivery, from order) to dispatch, a single integer, no ranges. Display: 0 = in stock; 1–3 = within 3 days; 4–7 = within a week; 8–14 = within 2 weeks; 15–30 = within a month; 31+ = more than a month; empty = "info in shop". A date (`YYYY-MM-DD`) only for new products on pre-order. Derive it: `0` if stock > 0 and dispatch within 24 h; else supplier lead time in days plus handling; empty if unknown.
11. **`DELIVERY`** repeats per carrier: `DELIVERY_ID` from Heureka's fixed list only (for example `CESKA_POSTA`, `PPL`, `DPD`, `ZASILKOVNA`, `BALIKOVNA_DEPOTAPI`, `ALZABOX`, `VLASTNI_PREPRAVA`), `DELIVERY_PRICE` incl. VAT prepaid, `DELIVERY_PRICE_COD` incl. COD fee (repeat the price if equal; omit if no COD). Each carrier once per item; listing the same carrier twice hides both. Max 100 different deliveries in the feed. Feed delivery overrides delivery prices set in the Heureka admin. With several services of one carrier, give the cheapest that delivers to the home anywhere in the country. Own transport only if nationwide.
12. **`ITEMGROUP_ID`**: the current Heureka spec reserves it for clothing and footwear variants differing only in size (with some workwear categories excluded). Do not use it to group colours or other variant types on Heureka; confirm in the admin if in doubt.
13. **`EAN`**: EAN-13; required only for books, textbooks, maps and guides, films, music and comics, but it is the strongest pairing signal everywhere. Never internal numbers.
14. **Images**: URL without spaces or diacritics, max 255 characters; formats webp, jpeg, png, bmp; max 4,096 × 4,096 px and 2,000 kB; recommended at least 175 × 175 px; no transparent background.
15. **Voucher and gifts**: with `SALES_VOUCHER`, `PRICE_VAT` must be the price after the voucher, the voucher must be usable by anyone without conditions, and removed when expired. `GIFT` only for a real item or voucher for a thing, given with this product unconditionally. `ITEM_TYPE` = `bazar` for used, refurbished, open-box, returned, display or second-quality items; they must not be sent as new.
16. **Heureka Marketplace**: prices rounded to whole crowns and full descriptions required; watermarked images lose the top position.
17. **Operations**: max 500,000 sellable items in one feed URL (splitting feeds does not raise the limit); fetched every 2 hours in PPC mode, every 4 hours in FREE mode; send `Last-Modified`; gzip accepted. Changing URLs, product names or categories unpairs products and re-pairing takes about 4 working days, so batch such changes. A separate availability XML (updated every 10 minutes) is available for fast-changing stock.
18. **Heureka.sk**: same structure, prices in EUR, Slovak category tree and carrier list (for example `SLOVENSKA_POSTA`). Build a separate feed with SK prices and SK delivery; never reuse CZK values.

### Zboží.cz (Seznam Nákupy)

19. **Root and required elements.** `<SHOP xmlns="http://www.zbozi.cz/ns/offer/1.0">` exactly. Required: `PRODUCTNAME`, `DESCRIPTION`, `URL`, `PRICE_VAT`, `DELIVERY_DATE`, `IMGURL`, and `EPREL_ID` in categories with EU energy labelling.
20. **`PRODUCTNAME`**: recommended 70, max 255 characters; template `MANUFACTURER LINE PRODUCT_DESIGNATION PRODUCT_CODE COLOUR OTHER`; the actual manufacturer of a compatible part, not the brand it fits. `PRODUCT` (shown in search, 64 characters displayed): no slogans, superlatives, emoticons, repeated words, "..." or more than one exclamation mark.
21. **`DESCRIPTION`**: plain text in Czech, recommended 250–1,024 characters, main benefits first; no shop advertising, repeated keywords, emoticons, gift information (use `EXTRA_MESSAGE`).
22. **`DELIVERY_DATE`**: days from order to dispatch as integer, or `YYYY-MM-DD` launch date for pre-orders, `-1` for unknown. Display bands: 0 in stock, 1–3, 4–7, 8+. Used/bazaar items must be `0`. Note the difference from Heureka, which counts from payment for prepaid orders.
23. **`DELIVERY`**: `DELIVERY_ID` from the Zboží list (carriers and pick-up networks such as `ZASILKOVNA`, `CESKA_POSTA_BALIKOVNA`, `PPL_PARCELSHOP`, `ONLINE`), `DELIVERY_PRICE` prepaid and `DELIVERY_PRICE_COD`; a method without a price is not shown; omit COD where it is not possible and omit carriers that cannot take the item (oversized goods). No weight-based pricing. The IDs differ from Heureka's (for example Balíkovna is `CESKA_POSTA_BALIKOVNA` on Zboží and `BALIKOVNA_DEPOTAPI` on Heureka); map per channel.
24. **`CATEGORYTEXT`**: full path from Zboží's category tree (`categories.json` / `categories.csv`), `|` separated, max 500 characters, up to 10 values per offer.
25. **`PRICE_BEFORE_DISCOUNT`** is the lowest price in the 30 days before the discount started. Zboží shows the discount only if the current price is below it, the discount is not older than 30 days, and the discount is 5–90 %. This mirrors the EU "prior price" rule (Directive 98/6/EC Art. 6a as amended by Directive 2019/2161). Never send the list price or RRP here; compute from price history or leave out.
26. **Other elements**: `EAN` (8, 12, 13 or 14 digits), `PRODUCTNO` (manufacturer part number), `ITEMGROUP_ID` (max 36 characters; variants of the same manufacturer, line and price level differing in one parameter), `IMGURL_ALTERNATIVE` (first 20 used), `CONDITION` (`new`, `open_box`, `used`, `refurbished`) with `CONDITION_DESC` and `WARRANTY` required for non-new, `MAX_CPC` and `MAX_CPC_SEARCH` (1–1,000 Kč), `VISIBILITY` 0 to hide, `CUSTOM_LABEL_0` / `CUSTOM_LABEL_1` (max 250 characters) for Sklik product groups.
27. **Images**: JPEG, PNG or WebP; recommended at least 425 × 440 px; bazaar offers at least 800 × 600 px; no watermarks, shop logos or slogans.

### idealo

28. **Verified reference is idealo's Partner Web Service 2.0 offer model.** Fields required for price comparison there: `sku`, `title`, `price`, `formerPrice`, `url`, `paymentCosts`, `deliveryCosts` (and merchant name and ID for marketplaces). Also important: `brand` (idealo says it matters for mapping to its catalog), `eans`, `hans` (manufacturer article numbers), `categoryPath` (the shop's own), `imageUrls` (first = main image), `delivery` (precise delivery time text; vague phrases such as "ready for shipment" or "presumably" are not allowed), `basePrice` where unit-price labelling is mandatory, `packagingUnit` for multipacks, `energyLabels` where required.
29. **Formats**: prices as `^\d{1,9}\.\d{2}$` (for example `177.99`); `sku` without spaces; variant URLs parameterised to the exact variant; `deliveryCosts` keyed by idealo's carrier list (`DHL`, `DPD`, `GLS`, `HERMES`, `UPS`, `SPEDITION`, and others); `paymentCosts` lists every supported payment method with its cost (0.00 allowed).
30. **CSV feeds**: idealo also accepts file feeds. The column names in the CSV template can differ from the API field names; confirm them in the idealo business account before generating the file.

## Process

1. **Scope**: channels and markets, feed mode (Heureka PPC/FREE), existing feeds and error reports.
2. **Inventory the data**: completeness per source field (EAN, lead time, images, delivery prices), variant model (separate URLs or one page), condition types.
3. **Map categories**: build a table store category → Heureka category → Zboží category → idealo `categoryPath`. Unmapped categories go to the report; never guess leaf categories for whole branches.
4. **Map elements per channel** using the rules above; build `PRODUCTNAME` from attributes.
5. **Derive delivery**: `DELIVERY_DATE` per channel from stock and lead time; `DELIVERY` blocks from the shipping matrix with each channel's IDs; exclude carriers per item where they cannot carry it.
6. **Discount elements**: compute the 30-day lowest price for Zboží `PRICE_BEFORE_DISCOUNT`; check voucher rules if vouchers are used.
7. **Validate**: XML well-formed, UTF-8, entities escaped (`&amp;` or CDATA), upper-case tags, uniqueness of IDs and URLs, lengths, number formats; run the channel validators where available.
8. **Report**: counts per channel of mapped, excluded and warned rows, with reasons; sample `SHOPITEM`s.

## Pitfalls and edge cases

- **Site vs feed availability.** If the product page says "skladem u dodavatele" (at supplier) and the feed sends `0`, it must match what the channel allows for that value; Heureka lists allowed page wording per value.
- **Same carrier twice on Heureka** (for example PPL home and PPL pick-up both as `PPL`) hides delivery for the item. Pick-up services have their own IDs.
- **Free delivery thresholds** cannot be expressed per order value; send the price that applies to that single item.
- **Heureka `ITEMGROUP_ID` misuse** on non-apparel, or Zboží variants lacking the parameter in the name, lead to wrong grouping.
- **Unit prices** (flooring, tiles): Zboží requires clear indication whether the price is per m², pack or other unit; idealo requires `basePrice` where unit-price rules apply.

## Rules

- Never invent EANs, lead times, delivery prices, prior prices or categories. Missing → element omitted and row reported.
- Feed price must equal the landing page price incl. VAT, per piece, in the channel's currency.
- `DELIVERY_DATE` must reflect what the page shows and what the store actually ships; do not send `0` for items not in stock.
- Never put shop promotions in names, descriptions or images; gifts and vouchers go only in their elements, only when they meet the channel's conditions.
- Read-only. Output is a mapping, sample XML and a report; the agent does not publish feeds or change channel settings. Bids (`HEUREKA_CPC`, `MAX_CPC`) are proposals for a human.

## Output format

```text
Feed mapping report: <store>, <YYYY-MM-DD>
Channels: <Heureka.cz | Heureka.sk | Zboží.cz | idealo.de>

Element map
| source field | Heureka | Zboží | idealo | transform / rule |

Category map
| store category | Heureka CATEGORYTEXT | Zboží CATEGORYTEXT | idealo categoryPath | status |

Delivery map
| store method | Heureka DELIVERY_ID | Zboží DELIVERY_ID | idealo deliveryCosts key | prepaid | COD | exclusions |

Row report
| channel | rows mapped | excluded | warnings | top reasons |

Rows needing owner input
| id | channel | missing / issue | effect |

Sample SHOPITEM per channel (XML)
```

## Worked example

Example data: Czech sports shop, 412 sellable variants, CZK, carriers PPL home (99 Kč, COD 129 Kč) and Zásilkovna pick-up (69 Kč, COD 99 Kč). Salomon Speedcross 6 GTX black, size 43, stock 3, EAN 8591234567893 (example value), was 3 990 Kč for the past 30 days, now 3 590 Kč.

Heureka:

```xml
<SHOPITEM>
  <ITEM_ID>SAL-SPX6-BLK-43</ITEM_ID>
  <PRODUCTNAME>Salomon Speedcross 6 GTX Black 43</PRODUCTNAME>
  <PRODUCT>Salomon Speedcross 6 GTX Black 43</PRODUCT>
  <URL>https://example-sport.cz/salomon-speedcross-6-gtx-black#43</URL>
  <IMGURL>https://example-sport.cz/img/spx6-black-1.jpg</IMGURL>
  <PRICE_VAT>3590</PRICE_VAT>
  <MANUFACTURER>Salomon</MANUFACTURER>
  <CATEGORYTEXT>Heureka.cz | Oblečení a móda | Obuv | Pánská obuv | Pánské běžecké boty</CATEGORYTEXT>
  <EAN>8591234567893</EAN>
  <PARAM><PARAM_NAME>Velikost</PARAM_NAME><VAL>43</VAL></PARAM>
  <DELIVERY_DATE>0</DELIVERY_DATE>
  <DELIVERY><DELIVERY_ID>PPL</DELIVERY_ID><DELIVERY_PRICE>99</DELIVERY_PRICE><DELIVERY_PRICE_COD>129</DELIVERY_PRICE_COD></DELIVERY>
  <DELIVERY><DELIVERY_ID>ZASILKOVNA</DELIVERY_ID><DELIVERY_PRICE>69</DELIVERY_PRICE><DELIVERY_PRICE_COD>99</DELIVERY_PRICE_COD></DELIVERY>
  <ITEMGROUP_ID>SALSPX6BLK</ITEMGROUP_ID>
</SHOPITEM>
```

Zboží.cz (inside `<SHOP xmlns="http://www.zbozi.cz/ns/offer/1.0">`):

```xml
<SHOPITEM>
  <ITEM_ID>SAL-SPX6-BLK-43</ITEM_ID>
  <PRODUCTNAME>Salomon Speedcross 6 GTX černé 43</PRODUCTNAME>
  <DESCRIPTION>Trailová běžecká obuv s membránou Gore-Tex a hrubým vzorkem podrážky pro bláto a sypký terén.</DESCRIPTION>
  <URL>https://example-sport.cz/salomon-speedcross-6-gtx-black?size=43</URL>
  <IMGURL>https://example-sport.cz/img/spx6-black-1.jpg</IMGURL>
  <PRICE_VAT>3590</PRICE_VAT>
  <PRICE_BEFORE_DISCOUNT>3990</PRICE_BEFORE_DISCOUNT>
  <DELIVERY_DATE>0</DELIVERY_DATE>
  <DELIVERY><DELIVERY_ID>PPL</DELIVERY_ID><DELIVERY_PRICE>99</DELIVERY_PRICE><DELIVERY_PRICE_COD>129</DELIVERY_PRICE_COD></DELIVERY>
  <DELIVERY><DELIVERY_ID>ZASILKOVNA</DELIVERY_ID><DELIVERY_PRICE>69</DELIVERY_PRICE><DELIVERY_PRICE_COD>99</DELIVERY_PRICE_COD></DELIVERY>
  <MANUFACTURER>Salomon</MANUFACTURER>
  <EAN>8591234567893</EAN>
  <ITEMGROUP_ID>SALSPX6BLK</ITEMGROUP_ID>
  <PARAM><PARAM_NAME>velikost</PARAM_NAME><VAL>43</VAL></PARAM>
</SHOPITEM>
```

Discount check: (3 990 − 3 590) / 3 990 = 10.0 %, within 5–90 %, sale started 6 days ago, so Zboží may show it. Category paths above are illustrative; take the exact path from each channel's current tree.

```text
Row report
| channel | rows mapped | excluded | warnings | top reasons |
| Heureka.cz | 394 | 18 | 23 | 12 excluded: supplier lead time unknown and stock 0; 6 excluded: discontinued; 23 warnings: no EAN |
| Zboží.cz | 388 | 24 | 23 | as Heureka + 6 in category "Doplňky" without Zboží category match |
Rows needing owner input
| SAL-SPX6-GRY-44 | both | lead time missing, stock 0 | excluded until lead time supplied |
```

## Quality checklist

- Every required element per channel present or the row excluded with a reason.
- IDs and URLs unique; ITEM_ID pattern and length valid; no reused IDs.
- `DELIVERY_DATE` derived from stock and lead time, consistent with the product page wording.
- Each `DELIVERY_ID` from the correct channel list; no carrier twice per Heureka item; COD omitted where not offered.
- Prices incl. VAT, per piece, correct currency per market, two decimals max.
- `PRICE_BEFORE_DISCOUNT` computed from 30-day history, not list price.
- Heureka `ITEMGROUP_ID` used only for apparel/footwear size variants.
- XML well-formed, UTF-8, upper-case tags, entities escaped, Zboží namespace present.
- Report counts add up to total sellable rows per channel.

## Sources

- https://sluzby.heureka.cz/napoveda/xml-feed/ (Heureka.cz XML specification: elements, DELIVERY_ID list, DELIVERY_DATE table, limits)
- https://sluzby.heureka.sk/napoveda/xml-feed/ (Heureka.sk XML specification)
- https://napoveda.sklik.cz/reklamy/xml-feed/specifikace/ (Zboží.cz XML feed specification, Seznam Nákupy)
- https://www.zbozi.cz/static/categories.json (Zboží.cz category tree)
- https://idealo.github.io/partner-web-service/docs/v2/ (idealo Partner Web Service 2.0 offer fields)
- https://eur-lex.europa.eu/eli/dir/2019/2161/oj (Directive (EU) 2019/2161, prior price rule in Art. 6a of Directive 98/6/EC)

## License
MIT
