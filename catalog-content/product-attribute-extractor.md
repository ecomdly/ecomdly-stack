---
name: product-attribute-extractor
owner: cartlift
category: Catalog content
description: Extracts colour, size, material, gender and other attributes from titles, variants and descriptions into Merchant Center-ready values, with source, evidence and confidence per value and a review queue for conflicts.
version: v3
license: MIT
updated: 2026-09-29
recommended: false
security_checked: true
url: https://ecomdly.com/skills/cartlift/product-attribute-extractor
raw: https://ecomdly.com/raw/cartlift/product-attribute-extractor.md
install: npx @ecomdly/cli add cartlift/product-attribute-extractor
---

# Product attribute extractor

Extracts structured attributes (colour, size, material, gender, age group, dimensions, weight, pattern, identifiers) from titles, descriptions, variant names and spec tables, normalises them to a target schema, and returns one row per value with its source, the exact evidence text and a confidence level. Filters, Merchant Center feeds, marketplace listings and comparison tables all need clean columns; most catalogs keep this data in free text. The naive approach asks a model to "fill in the attributes", which produces plausible values that are not in the data (every hoodie becomes "cotton"). This skill only records values that a quoted source states, keeps provenance so a human can audit any value in one look, and routes everything uncertain to review instead of the feed.

## When to use
- Merchant Center reports missing or invalid `color`, `size`, `gender`, `age_group`, `material` or `size_system` for apparel, or the feed lacks attributes for Shopping filters.
- Building or cleaning faceted navigation on a catalog imported from suppliers.
- Before `product-feed-optimizer`, `marketplace-listing-adapter` or `comparison-feed-mapper`, so they have structured values to work with.

## When not to use
- Identifier problems (missing, invalid or mismatched GTIN/MPN/brand) as the main issue: use `gtin-identifier-auditor`.
- Fixing a specific Merchant Center disapproval: `merchant-center-disapproval-fixer`.
- Designing which facets the site should have and how they are indexed: `faceted-navigation-seo-audit`.
- Writing descriptions: `product-description-writer`.

## Inputs
Required:
- Per product and per variant: `id`, `item_group_id` or parent id, title, description (HTML allowed), existing attribute columns, variant option names and values, category (store category and, if present, `google_product_category`).
- Target schema: the attribute list, for each attribute its type (enum, free text, number + unit), allowed values, and required/optional status. If the target is a Google feed, the schema is Merchant Center's product data specification; if a store filter, the store's filter values; if a marketplace, its category template.

Optional:
- Store vocabularies: colour map (supplier term → filter value), size vocabulary per category, material vocabulary.
- Brand size charts (needed for any size-system conversion).
- Supplier spec sheets or structured supplier feeds (often more reliable than the description).

How to get them: Shoptet and Upgates product CSV/XML export with parameters and variants; Shopify product CSV plus metafields export; for the Google target, the Merchant Center feed or the "Needs attention" product list download.

If the target schema is missing, ask for it. Without it the agent can still extract raw values with provenance, but it does not normalise, and says so in the summary.

## Best practices

### Source hierarchy and provenance
1. Order of trust: existing structured columns → variant option values → supplier structured data → spec table in the description → title → description prose. Why: structured fields were entered on purpose; prose is written to sell and mentions colours and materials that are not the product's ("pairs well with black jeans", "leather-look").
2. Never overwrite a filled structured value. When a lower source disagrees, keep the existing value and report the conflict with both evidence strings. A human decides.
3. Store the evidence: the exact substring and its field. A value without an evidence string is not allowed in the output.
4. Never infer from images, brand reputation, category norms or "typical" values. A t-shirt with no material stated has no material; a shoe with no gender stated has no gender, even if the model name sounds male.

### Variant level versus product level
5. Colour and size are variant-level attributes: extract them per variant, not from the parent title (a parent titled "T-shirt, black" may have five colours). Material and gender are usually product-level but check variants anyway (a leather and a textile version under one parent).
6. In Merchant Center each variant is a separate item sharing `item_group_id`; colour, size, material, pattern, age group and gender are the attributes that may differ between variants. Values must be consistent within a group except for the varying attributes.

### Normalisation rules
7. Colour: map to the store's vocabulary (`anthracite` → `Grey`) and keep the original as `color_label`. For Google: up to 3 colours, primary first, separated by "/" (for example `Black/White/Red`), no commas, 1–100 characters total and 1–40 per colour, no hex codes or numbers, no "multicolor" when the individual colours are known. Colour names that are marketing names ("Midnight", "Sand dune") map only if the store's colour map has them; otherwise flag.
8. Size: keep the size system with the value (`EU 43`, `UK 9`, `M`, `W32/L34`). Never convert between systems without the brand size chart. For Google, `size` is 1–100 characters, and `size_system` (AU, BR, CN, DE, EU, FR, IT, JP, MEX, UK, US) and `size_type` (regular, petite, maternity, big, tall, plus) are separate attributes. "One size" maps to the store's or channel's one-size value.
9. Material: composition with percentages by weight when given (`80% merino wool, 20% polyamide`); for Google, up to 3 materials separated by "/", 0–200 characters. Keep the full composition in a separate field for textile labelling, because EU textile composition rules require percentages and regulated fibre names (Regulation (EU) 1007/2011) and the "/" format drops them.
10. Gender and age group: extract only from explicit text ("dámské", "for men", "kids", "junior", category "Pánská obuv"). Map to Google values `male`, `female`, `unisex` and `newborn`, `infant`, `toddler`, `kids`, `adult`. A store category path is an acceptable source (confidence medium) because it was set by the merchant.
11. Dimensions and weight: numeric value and unit in separate fields, converted to the schema's unit with exact factors; keep the source string. Distinguish product dimensions from package dimensions; if the text does not say which, flag.
12. Numbers with ranges ("fits 38–42 cm") stay ranges; do not average.
13. Identifiers: GTIN digits only, lengths 8, 12, 13 or 14, check digit valid by the GS1 modulo-10 algorithm. An invalid GTIN is flagged, never "fixed" by recomputing the check digit. MPN is copied, not normalised beyond trimming.

### Confidence and routing
14. `high`: structured column, variant option, or supplier structured data.
15. `medium`: explicit statement in a spec table or title ("Material: 100% cotton", "... black, size 43"), or merchant category path for gender/age.
16. `low`: a single mention in prose, or any case where another reading exists. Low values never go to the feed; they go to the review queue.
17. Anything with a conflict is flagged regardless of confidence.

### Scope control
18. Extract only the attributes in the target schema plus identifiers. A long list of optional attributes that no filter or channel uses adds review work and no value.

## Process
1. Profile the input: rows, variants, categories; fill rate per target attribute before extraction (filled / applicable rows). Applicable means the attribute is relevant for the category (no `size` for a kettle).
2. For each product and variant, walk the source hierarchy; stop at the first source that gives an explicit value, but continue scanning lower sources to detect conflicts.
3. Normalise each value (rules 7–13). If a value cannot be mapped to an allowed enum value, keep the raw value and flag `unmapped`.
4. Assign confidence (14–16) and flags.
5. Validate: Google length limits, separators, enum values, GTIN checksum, group consistency within `item_group_id`.
6. Compute the after-fill rate counting only `high` and `medium` values without conflict flags; this is what could go to the feed after approval.
7. Build the review queue: conflicts first, then low confidence on required attributes, then unmapped values.
8. Summarise per attribute: before, after (approved-ready), in review, not stated.

## Pitfalls and edge cases
- "Black/White" in a variant name may be two variants joined by a separator in the export, or one two-colour product. Check whether the parent has separate variants.
- Negations and comparisons: "not suitable for children", "lighter than cotton", "leather-free". Pattern matching on "children" or "cotton" gives wrong values; low confidence at best.
- Materials of parts: "leather upper, rubber sole" is not "leather/rubber" for the whole product; keep part-level material if the schema supports it, otherwise flag.
- Units stuck to numbers without spaces (`43cm`, `1,5kg`) and decimal commas in Czech and German text: parse with the source locale.
- Model numbers containing digits that look like sizes (`Air 270`, `Speedcross 6`).
- Czech and Polish inflection: colour adjectives change with gender and case ("černý", "černá", "černé"). Normalise the lemma through the colour map.
- Bundles and sets: attributes per component differ; do not assign one colour to "3-pack assorted".
- Supplier text copied across products: the same material sentence on 400 SKUs may be a template error. If identical prose conflicts with product-specific structured data, trust the structured data and flag.

## Rules
- Read-only. Output is a proposal file; a human approves before anything is imported into the store or feed. Never push changes to Merchant Center, the store or a marketplace.
- Never invent a value. No evidence string, no value.
- Never overwrite an existing filled value; report the disagreement.
- Never "fix" identifiers; flag them.
- Low-confidence and conflicting values do not appear in the feed-ready output.
- If the target schema or a size chart is missing, say what that prevents and ask for it.

## Output format
```
id,item_group_id,attribute,value,raw_value,source_field,evidence,confidence,flag
...
SUMMARY
rows: <n> products / <m> variants | schema: <name/version>
| attribute | applicable | before | after (feed-ready) | in review | not stated |
flags: conflict <n> | low_confidence <n> | unmapped <n> | invalid_gtin <n> | group_inconsistent <n>
REVIEW QUEUE (priority order): <id> <attribute> <flag> <evidence>
```

## Worked example
Example input: 3 variants of one shoe (group 8840) and one sock product, target = Google Merchant Center feed plus the store colour map (`anthracite` → `Grey`).

```
id,item_group_id,attribute,value,raw_value,source_field,evidence,confidence,flag
8841,8840,color,Black,Black,variant_option,"Black / EU 43",high,
8841,8840,size,43,EU 43,variant_option,"Black / EU 43",high,
8841,8840,size_system,EU,EU 43,variant_option,"Black / EU 43",high,
8841,8840,gender,male,Pánská obuv,category,"Obuv > Pánská obuv",medium,
8842,8840,color,Grey,anthracite,variant_option,"anthracite / EU 44",high,
8840,8840,material,,,,,,"not stated: description says only 'pairs well with leather jackets'"
8850,,material,merino wool/polyamide,"80% merino wool, 20% polyamide",spec_table,"Složení: 80% merino wool, 20% polyamide",medium,
8850,,gtin,4006381333931,4006381333931,column_gtin,"4006381333931",high,"conflict: title contains 4006381333932 (invalid check digit)"
SUMMARY
rows: 2 products / 4 variants | schema: Google Merchant Center product data specification
| attribute | applicable | before | after (feed-ready) | in review | not stated |
| color     | 4 | 1 | 3 | 0 | 1 |
| size      | 3 | 0 | 3 | 0 | 0 |
| material  | 2 | 0 | 1 | 0 | 1 |
flags: conflict 1 | low_confidence 0 | unmapped 0 | invalid_gtin 1 | group_inconsistent 0
REVIEW QUEUE: 8850 gtin conflict (title vs column) | 8840 material not stated, ask supplier
```
(Only the rows needed to illustrate each case are shown; the third shoe variant follows the same pattern.) The full composition "80% merino wool, 20% polyamide" is kept in `raw_value` for the product page and textile labelling; the feed value uses the "/" format.

## Quality checklist
- Every non-empty value has a source field and a verbatim evidence string.
- No existing structured value was overwritten; all disagreements are flagged as conflicts.
- Google formats respected: "/" separators, max 3 colours and materials, length limits, enum values for gender, age group, size system and size type.
- GTINs validated by checksum; none altered.
- Feed-ready counts include only high/medium values without conflicts.
- Before/after fill rates computed over applicable rows only.
- Review queue ordered by impact (required attributes and conflicts first).

## Sources
- Google Merchant Center, color [color]: https://support.google.com/merchants/answer/6324487
- Google Merchant Center, size [size]: https://support.google.com/merchants/answer/6324492
- Google Merchant Center, material [material]: https://support.google.com/merchants/answer/6324410
- Google Merchant Center, product data specification: https://support.google.com/merchants/answer/7052112
- GS1 check digit calculation: https://www.gs1.org/services/how-calculate-check-digit-manually
- Textile Fibre Names Regulation (EU) 1007/2011: https://eur-lex.europa.eu/eli/reg/2011/1007/oj

## License
MIT
