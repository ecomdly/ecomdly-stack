---
name: comparison-feed-mapper
owner: listwise
category: Marketplaces
description: Maps a store catalog to Heureka, Zboží.cz and Idealo XML/CSV feeds with correct elements, categories and delivery days, and reports rows it cannot fill from real data.
version: v3
license: MIT
updated: 2026-09-11
recommended: false
security_checked: true
url: https://ecomdly.com/skills/listwise/comparison-feed-mapper
raw: https://ecomdly.com/raw/listwise/comparison-feed-mapper.md
install: npx @ecomdly/cli add listwise/comparison-feed-mapper
---

# Comparison feed mapper

Heureka and Zboží.cz rank what they can parse. This skill turns your catalog into a clean SHOPITEM feed, element by element, and tells you which rows will be paired badly or rejected.

## When to use
- Setting up a new comparison-shopping feed, or migrating e-shop platforms.
- When Heureka or Zboží reports unpaired products or feed errors.

## Input
Catalog export: ID, name, manufacturer, EAN, price incl. VAT, URL, image URLs, stock quantity, supplier lead time, category path, variant group, attributes. The store's delivery methods with prices (including cash on delivery).

## Element map (SHOP > SHOPITEM)
- `ITEM_ID` — stable internal ID, max 36 characters, letters, digits, `_` and `-` only. Never reuse an ID for a different product.
- `PRODUCTNAME` — exact product name for pairing: Manufacturer · Product line · Model · Variant. No shop text.
- `PRODUCT` — may add a short distinguishing phrase; same product otherwise.
- `CATEGORYTEXT` — the channel's own taxonomy: Heureka starts with `Heureka.cz | ...`, Zboží uses its tree with ` | ` separators. Map your category, do not paste it.
- `PRICE_VAT`, `URL`, `IMGURL`, `IMGURL_ALTERNATIVE`, `EAN`, `MANUFACTURER`, `ITEMGROUP_ID` for variants.
- `DELIVERY_DATE` — days until dispatch: `0` in stock; otherwise supplier lead time from data.
- `DELIVERY` blocks with `DELIVERY_ID`, `DELIVERY_PRICE`, `DELIVERY_PRICE_COD`.
- `PARAM` with `PARAM_NAME` / `VAL` for size, color and other attributes that separate variants.

Idealo takes CSV: sku, brand, title, categoryPath, price, deliveryTime, deliveryCosts, eans, url, imageUrls.

## Rules
- Never invent EANs, lead times or delivery prices. A row without them is reported, not guessed.
- One URL per variant when variants have separate prices or stock.
- Prices in the feed must equal the price on the landing page, including VAT.

## Output format
```xml
<SHOPITEM>
  <ITEM_ID>SAL-SPX6-BLK-43</ITEM_ID>
  <PRODUCTNAME>Salomon Speedcross 6 GTX Black 43</PRODUCTNAME>
  <CATEGORYTEXT>Heureka.cz | Sport | Běh | Běžecká obuv</CATEGORYTEXT>
  <PRICE_VAT>3990</PRICE_VAT>
  <DELIVERY_DATE>0</DELIVERY_DATE>
  <PARAM><PARAM_NAME>Velikost</PARAM_NAME><VAL>43</VAL></PARAM>
</SHOPITEM>
```
Report: `412 rows mapped; 18 missing EAN; 6 without lead time (DELIVERY_DATE left out).`

## License
MIT
