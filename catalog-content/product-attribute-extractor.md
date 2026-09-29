---
name: product-attribute-extractor
owner: cartlift
category: Catalog content
description: Pulls structured attributes (color, size, material, GTIN, dimensions) out of messy titles and descriptions into feed-ready columns, with a source and confidence per value.
version: v2
license: MIT
updated: 2026-09-10
recommended: false
security_checked: true
url: https://ecomdly.com/skills/cartlift/product-attribute-extractor
raw: https://ecomdly.com/raw/cartlift/product-attribute-extractor.md
install: npx @ecomdly/cli add cartlift/product-attribute-extractor
---

# Product attribute extractor

Filters, feeds and comparison tables all need clean columns, and most catalogs keep that data buried in free text. This skill extracts it, normalises it, and shows where every value came from.

## When to use
- Merchant Center warns about missing `color`, `size`, `material`, `gender` or `age_group`.
- Building faceted navigation on a catalog imported from suppliers.
- Before running a title optimizer, so it has attributes to work with.

## Input
Per product: id, title, description, existing attribute columns, variant names, category. The target schema (which attributes, which allowed values), plus the store's color and size vocabularies if they exist.

## Extraction order
1. Existing structured columns win. Do not overwrite a filled value; report disagreements.
2. Variant names (`Black / 43`) are the next most reliable source.
3. Title, then description, then spec tables inside the description.
4. Never infer from images, brand reputation or "typical" values. A t-shirt with no material stated has no material.

## Normalisation
- **Color:** map to the store's vocabulary (`anthracite` → `Grey`), keep the original as `color_label`. Multi-color: up to three, separated by `/`.
- **Size:** keep the system (`EU 43`, `UK 9`, `M`). Never convert between systems without a size chart.
- **Material:** composition with percentages when given (`80% merino wool, 20% polyamide`).
- **Dimensions and weight:** numeric value + unit in separate fields, metric unless the schema says otherwise.
- **GTIN:** digits only, validate the check digit; an invalid GTIN is flagged, never "fixed".

## Confidence
- `high` — structured column or variant name.
- `medium` — explicit phrase in the title or a spec table.
- `low` — mentioned in prose once, possibly ambiguous ("pairs well with black jeans" is not a color). Low values go to review, not into the feed.

## Output format
```
id,attribute,value,source,confidence,flag
8841,color,Black,variant,high,
8841,size,EU 43,variant,high,
8841,material,,,,"not stated anywhere"
8842,material,"80% merino wool, 20% polyamide",description,medium,
8843,gtin,4006381333931,column,high,"title says 4006381333932 - check"
```
Summary: rows processed, fill rate per attribute before and after, count of flags.

## License
MIT
