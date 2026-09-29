---
name: product-feed-optimizer
owner: cartlift
category: Product feeds
description: Rewrites Google Shopping feed titles and descriptions from verified attributes, policy-clean and marked as AI-generated via structured_title, and proposes missing attribute fills in a reviewable CSV. For store owners and feed managers.
version: v3
license: MIT
updated: 2026-09-29
recommended: false
security_checked: true
url: https://ecomdly.com/skills/cartlift/product-feed-optimizer
raw: https://ecomdly.com/raw/cartlift/product-feed-optimizer.md
install: npx @ecomdly/cli add cartlift/product-feed-optimizer
---

# Product feed optimizer

Produces rewritten feed titles and descriptions, plus fill-in proposals for structured attributes, as a reviewable CSV with a reason and a flag per row. Shopping and free listings have no keywords: Google matches the query against your product data, and the title carries most of that weight, so the naive approach of pasting the web H1 or stuffing keywords either hides the attribute people search for or trips editorial policy. The other trap is new: Google requires titles and descriptions written with generative AI to be submitted through `structured_title` / `structured_description` marked as AI-generated. This skill builds titles from verified attributes in the order searchers use them, keeps them policy-clean, and tags AI-written text correctly.

## When to use

- A feed export (CSV, TSV, XML) needs better titles and descriptions before or after it reaches Merchant Center.
- Products with impressions but low click-through, or products with very few impressions in a category where competitors show, weekly.
- After onboarding a supplier whose titles are internal codes ("TC-88 BLK 43").
- When Merchant Center flags missing apparel attributes (`color`, `size`, `gender`, `age_group`) or title and description warnings.

## When not to use

- Disapprovals for price, availability, images or policy: `merchant-center-disapproval-fixer`.
- GTIN, MPN or `identifier_exists` problems: `gtin-identifier-auditor`.
- Extracting missing attributes from unstructured supplier text: `product-attribute-extractor` first, then this skill.
- Web product-page copy (the page, not the feed): `product-description-writer`.
- Heureka or Zboží.cz naming rules differ (pairing-driven): `comparison-feed-mapper`.

## Inputs

Required per product (one row per variant):

- `id`, `item_group_id`, current `title`, current `description`, `brand`, `google_product_category`, `product_type`, `link`.
- Variant attributes present in the data: `color`, `size`, `material`, `pattern`, `gender`, `age_group`, capacity, dimensions, pack size, model number.
- The target country and language of the feed.

Required from the store:

- Brand rules: banned words, trademark casing, claims policy (for example "no medical claims"), own-brand name.

Optional, strongly recommended:

- Performance per item, last 30–90 days: impressions, clicks, CTR, conversions (Google Ads > Products, segment by item ID). Lets you prioritise and later measure.
- Search terms from Shopping or Performance Max search-term insights, to learn the words people actually use (`shopping-search-terms-miner`).
- The landing page H1 per product.

If attributes are missing, do not recover them from the old title's guesswork; either leave the slot empty and flag the row, or hand off to `product-attribute-extractor`. If no performance data exists, process by category, largest first.

## Best practices

### Title

1. **Hard limits.** `title` is 1–150 characters; longer titles are truncated with a warning. Google says users usually notice only the first 70 or fewer characters depending on screen, and recommends putting the most important details first and using the space. Aim for the decisive attributes within the first 70 characters and a complete title up to 150.
2. **Order by how people search in the category.** These patterns are practitioner conventions, not Google rules; confirm against the store's search terms:
   - Apparel and footwear: Brand + gender + product type + model/line + material or key feature + color + size.
   - Electronics: Brand + product line + model + product type + key spec (capacity, screen size) + color.
   - Home and garden: Brand + product type + material + dimensions + color.
   - Consumables and cosmetics: Brand + product line + product type + variant (flavour, shade, scent) + quantity or pack size.
   - Spare parts and accessories: Brand of the part + product type + "for" + compatible model + part number. Brand is the part's maker, never the OEM it fits.
   Rule: if the brand is unknown to shoppers (own brand, niche importer), move product type ahead of brand; if shoppers search the brand, it goes first.
3. **Variants must be distinguishable.** Google requires each variant's distinguishing details in the title (for example color, size) and the same values in the matching attributes. For design elements with no attribute, such as flavour, add them to the title.
4. **Editorial rules (disapproval risk).** No promotional text: Google lists price, sale price, sale dates, shipping, delivery date, other time-related information and your company name as not allowed in titles. No capitals for emphasis (abbreviations, units and brand casing are fine), no gimmicky symbols or HTML, no foreign words unless well understood, correct grammar. No keyword lists or repeated words.
5. **Match the landing page product, not its wording.** Google requires the title to describe the product on the landing page; it does not need to be identical to the H1. Flag rows where the title names a different product, variant or quantity than the page, not rows that are merely worded differently.
6. **Use real units and the market's language.** "500 ml", "43 EU", "2 TB" in the feed language. Accented letters are allowed. Do not mix alphabets.

### AI-written text

7. **Structured attributes for generative AI.** Google: for titles created using generative AI, use `structured_title`; for AI-written descriptions, use `structured_description`; set `digital_source_type` to `trained_algorithmic_media`. `content` has the same limits as the plain attribute (150 characters for title, 5 000 for description). Every title or description this skill writes is AI-generated text, so the output must say so and be delivered in that form, even after a human edits it lightly, unless the owner decides otherwise and records that decision.
8. **Google may also rewrite.** Google can customise titles in Shopping ads (Product Data Customization); Google's title page describes opting out through the account manager or support. If the owner needs exact titles, note this.

### Description

9. **Limits and placement.** `description` is 1–5 000 characters; Google advises listing the most important details in the first 160–500 characters. Write 300–1 000 characters for most products: what it is, who or what it is for, the specs that decide the purchase (size, material, compatibility, capacity), care or use notes.
10. **Not allowed in descriptions** (per Google's description spec): promotional text including price, sale dates, shipping, delivery date, company name; links to your store or other sites; comparisons with other products; details of accessories or similar products; business history or policies; category paths; capitals for emphasis. Escape XML properly (`&amp;`, `&quot;`) and do not double-encode.

### Attributes that also match queries

11. **Fill structured attributes, not only the title.** `color`, `size`, `gender`, `age_group`, `material`, `pattern` are used for filtering and matching. Apparel needs `color`, `size`, `gender` and `age_group` when targeting Brazil, France, Germany, Japan, the UK or the US; fill them for other markets too because filters use them. `color` must be a word, not a number or hex code (max 40 characters per color, 100 total); one-size items use `one_size` or equivalents Google lists.
12. **`product_type`** is your own category path, up to 750 characters; submit up to 5, but only the first is used to organise bidding and reporting in Shopping campaigns. Make it granular and stable because product groups depend on it.
13. **`google_product_category`**: one predefined category, as ID or full path, not both. Wrong category can change which attributes Google requires.
14. **`product_highlight`** (2 to 100 highlights, up to 150 characters each) and **`product_detail`** (up to 100 details) are optional places for specs that do not fit the title. Only fill them from verified data.

### Workflow discipline

15. **Change in batches and measure.** Rewrite one category or supplier at a time and compare CTR and impressions for 2–4 weeks against an untouched comparable group (`ab-test-readout` for the readout). Seasonal swings are larger than most title effects.
16. **Keep a rollback column.** Always output the old value next to the new one.

## Process

1. **Profile the feed.** For each category: row count, share of rows with each attribute filled, average title length, share of titles with promotional words, caps runs, or repeated words. This tells you where the gain is.
2. **Prioritise.** Order rows by impressions (or by category size if no performance data). Start with high-impression, low-CTR items and with items that have almost no impressions despite being in stock.
3. **Choose the pattern** per category (rule 2); adjust with search-term evidence if available.
4. **Build the title** from attributes only: fill slots left to right, skip empty ones, join with spaces or " - " consistently, apply brand casing and banned-word rules, cut optional trailing slots if over 150 characters. Never cut the variant attributes.
5. **Validate the title:** length ≤ 150; decisive attributes within the first 70 characters; no promotional or time words, company name, or all-caps words other than known abbreviations; no duplicate words; matches the landing page product.
6. **Write the description** from verified attributes and supplier copy; validate against rule 10; check it does not contradict the title.
7. **Propose attribute fills** only where the value exists in the data (for example `color` present in a variant option but missing in the feed column).
8. **Flag** every row with reasons: `missing_attr:<name>`, `h1_mismatch`, `claim_review`, `truncated_optional_slot`, `ai_generated`.
9. **Output** CSV plus a short summary of changes by category.

## Pitfalls and edge cases

- **Color and size inside the title but not in attributes** (or the other way round) causes variant confusion. Keep them in both.
- **Supplier model codes.** Keep a model number only if shoppers search it (electronics, spare parts). Internal SKUs never go in titles.
- **Bundles and multipacks.** State the count ("3-Pack", "Set of 4") and match `multipack` / `is_bundle`.
- **Health, cosmetics, supplements.** Claims like "cures", "treats", "anti-ageing guaranteed" trigger policy review; flag `claim_review` and do not write them.
- **Own brand equal to the store name.** Keep it in `brand`, leave it out of the title: Google bars the company name in titles, and its own store-brand example title omits the brand.
- **Brand in the title vs `brand`.** They must agree; "Salomon" in the title with `brand` = store name is inconsistent.
- **Feed rules and supplemental sources** can override your new title. Check which source wins before handing over.
- **Czech and Slovak grammar.** Adjective agreement with gender and number ("Pánská trekingová obuv", "Pánské boty") and declension of colour words; keep the feed's language natural, not an English word order transplanted.

## Rules

- Attributes come from the data, never from the old title's wording or from assumptions. A missing color stays missing and is flagged.
- Never add claims (waterproof, organic, certified, medical effects) that are not in verified product data.
- No promotional text, prices, shipping, dates or company name in titles or descriptions.
- Mark every generated text as AI-generated and deliver it for `structured_title` / `structured_description` with `digital_source_type` = `trained_algorithmic_media`.
- Read-only. Output is a proposal file; the agent does not upload to Merchant Center, the store or the feed tool.
- Keep brand casing, trademarks and banned-word lists exactly as the store gives them.

## Output format

CSV, UTF-8, one row per item; the summary follows as plain text.

```text
id,old_title,new_title,new_title_len,old_description_len,new_description,attribute_fills,flags,digital_source_type
<id>,"<old>","<new>",<n>,<n>,"<new description>","color=Black; gender=male","missing_attr:material|ai_generated",trained_algorithmic_media

Summary
- rows processed: <n> (categories: <list>)
- titles changed: <n>; avg length <old> -> <new>
- rows with missing attributes: <n> (top: <attr>: <n>)
- rows flagged for claim review: <n>
- H1 mismatches: <n>
- Deliver as structured_title / structured_description (AI-generated).
```

## Worked example

Example data, CZ store feed in English for a UK target, two rows.

Input row 8841: title `SPEEDCROSS 6 GTX - BEST PRICE!!`, brand Salomon, gender male, color Black, size 43 (EU), material Gore-Tex membrane, product_type `Shoes > Running > Trail`. Row 8842: title `Hiker socks merino`, brand Trail Store (own brand), size L, color Grey, pack 3, material merino wool, gender missing, landing H1 "Hiker Socks 3-pack".

```text
id,old_title,new_title,new_title_len,old_description_len,new_description,attribute_fills,flags,digital_source_type
8841,"SPEEDCROSS 6 GTX - BEST PRICE!!","Salomon Men's Speedcross 6 GTX Trail Running Shoes Gore-Tex Black EU 43",71,0,"Trail running shoe with a Gore-Tex membrane that keeps feet dry on wet ground and a lugged sole for grip in mud and loose terrain. Quicklace system tightens with one pull. Men's fit, EU size 43, black.","",ai_generated,trained_algorithmic_media
8842,"Hiker socks merino","Merino Wool Hiking Socks Crew 3-Pack Grey L",43,0,"Crew-length hiking socks in merino wool. Pack of three pairs, grey, size L.","multipack=3","missing_attr:gender|ai_generated",trained_algorithmic_media

Summary
- rows processed: 2 (categories: footwear, socks)
- titles changed: 2; avg length 25 -> 57
- rows with missing attributes: 1 (top: gender: 1)
- rows flagged for claim review: 0
- H1 mismatches: 0 ("Hiker Socks 3-pack" and the new title describe the same product)
- Deliver as structured_title / structured_description (AI-generated).
```

Why: "BEST PRICE!!" and all caps are promotional and emphasis; removed. For 8842 the own brand is also the store name, so it stays in `brand` but not in the title (company name is not allowed in titles) and product type leads. "Crew" came from the supplier data; the length word was not invented. Gender is missing, so the title does not say "Men's".

## Quality checklist

- Every title ≤ 150 characters; decisive attributes within the first 70.
- No promotional words, prices, dates, shipping, company name or emphasis caps in any title or description.
- Every value in a new title exists in the row's attributes or verified data.
- Variant attributes present in both title and attribute columns.
- Descriptions contain no links, comparisons, policies or category paths.
- Old values kept for rollback.
- AI-generated flag and `digital_source_type` set on every rewritten row.
- Rows with missing data flagged, not filled.
- Summary numbers match the CSV.

## Sources

- https://support.google.com/merchants/answer/6324415 (title and structured_title)
- https://support.google.com/merchants/answer/6324468 (description and structured_description)
- https://support.google.com/merchants/answer/7052112 (Product data specification: color, size, gender, age_group, product_highlight, product_detail)
- https://support.google.com/merchants/answer/6324406 (product_type)
- https://support.google.com/merchants/answer/6324436 (google_product_category)
- https://support.google.com/merchants/answer/6324507 (item_group_id)
- https://support.google.com/merchants/answer/4752265 (landing page requirements)

## License
MIT
