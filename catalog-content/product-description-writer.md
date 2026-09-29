---
name: product-description-writer
owner: tonecheck
category: Catalog content
description: Writes product page descriptions from the spec sheet in the store's voice — benefit first, facts only from the data, missing specs flagged instead of guessed.
version: v2
license: MIT
updated: 2026-08-29
recommended: false
security_checked: true
url: https://ecomdly.com/skills/tonecheck/product-description-writer
raw: https://ecomdly.com/raw/tonecheck/product-description-writer.md
install: npx @ecomdly/cli add tonecheck/product-description-writer
---

# Product description writer

A description has one job: answer the question the shopper would otherwise email you about. This skill turns a spec sheet into copy that does that, in your voice, without a single invented fact.

## When to use
- New products arriving with only a supplier spec sheet or a one-line title.
- Rewriting thin or duplicated manufacturer copy (the same text on 40 other shops ranks for nobody).

## Input
Per product: title, brand, category, attributes (material, dimensions, weight, capacity, compatibility, care), what is in the box, warranty, and any certificates with their numbers. The store's voice guide and a list of banned claims. Optional: top support questions for the category.

## Structure
1. **Opening line (≤ 25 words):** what it is and who it is for, in plain words.
2. **Why it matters (2–3 sentences):** translate the two attributes that decide the purchase into use. "600 g" becomes "light enough to carry in one hand", but only if 600 g is in the data.
3. **Specs list:** every attribute present, units as given, one per line.
4. **In the box / care / warranty:** only if provided.

Length targets: consumables 60–100 words, apparel 80–130, electronics and furniture 120–200 plus specs.

## Rules
- Every factual statement must trace to an input field. If it does not, cut it.
- Missing a spec the category needs (size chart for apparel, dimensions for furniture, compatibility for accessories)? Write the rest and add it to `missing`.
- Regulated claims ("waterproof", "hypoallergenic", "organic", "BPA-free", health or eco claims) only with the matching attribute or certificate in the data. Otherwise flag.
- No superlatives you cannot prove ("best", "#1"), no competitor names, no invented reviews or testimonials.
- Keep brand casing and the voice guide's person (we/you) and spelling (US/UK).
- Do not copy the supplier text: rewrite, and keep sentences under 20 words.

## Output format
```
id: 2231
title: Hario V60 Ceramic Dripper 02, White
description: |
  A ceramic pour-over dripper for one to four cups, for anyone who wants
  more control over their morning coffee than a machine gives.
  The spiral ribs and single large hole let you set the flow with your pour...
specs:
  - Material: Ceramic (Arita-yaki)
  - Size: 02 (1–4 cups)
missing: [dimensions, dishwasher-safe]
flags: ["'keeps coffee hot' not supported by data - removed"]
```

## License
MIT
