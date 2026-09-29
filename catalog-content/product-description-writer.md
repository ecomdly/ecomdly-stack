---
name: product-description-writer
owner: tonecheck
category: Catalog content
description: Writes product descriptions and spec lists from supplier data in the store's brand voice, ordering facts by what decides the purchase for each product type, listing missing specs and flagging unsupported or EU-restricted claims.
version: v3
license: MIT
updated: 2026-09-29
recommended: false
security_checked: false
url: https://ecomdly.com/skills/tonecheck/product-description-writer
raw: https://ecomdly.com/raw/tonecheck/product-description-writer.md
install: npx @ecomdly/cli add tonecheck/product-description-writer
---

# Product description writer

Produces a product description, a clean specs list and a `missing`/`flags` report for each product, from the supplier spec sheet and the store's voice guide, without a single invented fact. A description has one job: answer the question the shopper would otherwise email you about, or return the product over. The naive approach (paraphrase the supplier text, add adjectives) fails three ways: the same text on forty other shops gives Google no reason to prefer this page, adjectives answer no question, and paraphrasing tends to harden vague supplier wording into claims ("water-resistant" becomes "waterproof", "recyclable packaging" becomes "eco-friendly product") that are false or, in the EU, prohibited. This skill writes from attributes, in order of what decides the purchase for that product type, and refuses what the data does not support.

## When to use
- New products arriving with only a supplier spec sheet, a feed row or a one-line title.
- Rewriting thin or duplicated manufacturer copy.
- Rewriting after returns data shows shoppers misunderstood the product (wrong size, wrong compatibility).
- Batch runs over a category, with a human reviewing a sample and every flagged item.

## When not to use
- Category pages: `category-page-copy-writer`.
- Translating existing descriptions: `catalog-translation-localizer`.
- Extracting attributes from messy supplier text into structured fields first: `product-attribute-extractor`, then this skill.
- Marketplace-specific listing formats (character limits, bullet slots): `marketplace-listing-adapter`.
- Products with no attributes beyond a title: write nothing, return the `missing` list.

## Inputs

Required, per product:
- `id`, `title`, `brand`, `category` (store category path).
- **Attributes** as key/value with units: material or composition, dimensions, weight, capacity/volume, compatibility, power/battery, size range and size chart reference, care instructions, what is in the box, warranty (length, who provides it), country of origin if the store shows it.
- **Certificates and test results** with identifiers (standard and number, e.g. "EN ISO 20345:2022 S3", "IP67", "EU Ecolabel licence no."), or the statement that there are none.
- **Supplier text** (to mine facts from, never to copy).

Required, per store:
- **Voice guide file**: person and address (we/you; in Czech or German, informal or formal address), spelling variant (US/UK), sentence-length preference, banned words, brand-term casing, 2–3 approved example descriptions.
- **Banned-claims list** from the owner (legal or brand reasons).

Optional:
- Top support questions and returns reasons for the category (ask the helpdesk for the last 90 days).
- Search Console queries landing on the product or its category.
- Product safety information: manufacturer name, postal and electronic address, warnings.

If the voice guide is missing, write in neutral second person, short sentences, and state `voice: default (no guide)`. If a required attribute for the product type is missing, write the rest and list it under `missing` (see the matrix below). Never fill a gap from general knowledge of the brand or model, even when "everyone knows" the spec: model years and regional versions differ.

## Best practices

### Lead with what decides the purchase
1. **Order facts by product type, not by the supplier's order.** Working matrix (adapt with the store's returns and support data, which override it):
   - Apparel and footwear: fit and sizing (size range, how it runs only if evidence exists), fibre composition, use/season, care. Must-have: composition, size chart reference.
   - Electronics and accessories: what it works with (compatibility, standards, connectors), the one performance spec buyers compare (capacity, output, resolution), power, in the box. Must-have: compatibility, power/battery.
   - Furniture and large items: external dimensions (W × D × H), weight, load capacity, materials, assembly required or not, delivery form. Must-have: dimensions.
   - Consumables (food, coffee, supplements, cleaning): quantity and unit, how to use, ingredients/allergens (food), shelf life if given. Must-have: quantity; for food, the mandatory food information the store shows elsewhere.
   - Cosmetics: product type and skin/hair type as stated by the manufacturer, volume, how to use, full ingredient list (INCI) shown on the page. Must-have: volume, ingredients.
   - Tools and equipment: the task it does, key performance spec, power source, safety standard, included accessories.
2. **Opening sentence (≤ 25 words): what it is and who or what it is for**, in the words shoppers use. Product type + the one attribute that separates it from its siblings in the same category. Not the brand story.
3. **Translate the two deciding attributes into use**, one sentence each, only from the data. "600 g" → "light enough to carry in one hand" only if 600 g is in the data and the comparison is self-evident. Do not invent comparisons with other products ("30% lighter") unless both values are in the input.
4. **Answer the top support question in the body** if the data answers it (e.g. "Does it fit the 02 filter papers?" → state compatible papers if given). If the data does not answer it, add it to `missing`: that gap is what generates the emails.
5. **Specs list: every attribute present, one per line, units as given**, labelled consistently across the catalog (the same label for the same attribute in every product). Convert units only if the voice guide asks, and keep the original in brackets.
6. **In the box / care / warranty** as short lines, only if provided. Warranty: state length and provider exactly as given; in the EU the statutory legal guarantee exists regardless, so never describe a commercial warranty as if it replaced it; flag wording that could read that way.

### Length by product type
7. Working defaults for body copy, excluding the specs list: consumables 60–100 words; apparel and footwear 80–130; cosmetics 80–130; electronics, tools and furniture 120–200. These are store-practice defaults, not a ranking rule. Go shorter when the data is thin; never pad to reach the range. Go longer only for complex or expensive items where the support data shows questions the extra text answers.

### Voice from the guide
8. **Extract the voice rules before writing**: person and address form, sentence length, reading level, allowed and banned words, how the brand refers to itself, punctuation habits (dashes, exclamation marks), number formats. Write them into a short checklist in the output header so the reviewer can check them.
9. **Imitate the approved examples' structure, not their sentences.** Reusing their phrases across products creates template sameness.
10. **Keep sentences under 20 words** unless the guide says otherwise; one idea per sentence; no stacked adjectives.

### Claims: what needs evidence and what is banned
11. **Every factual statement traces to an input field.** If it does not, cut it and log it in `flags`.
12. **Environmental claims (EU, applicable from 27 September 2026 under Directive (EU) 2024/825 amending the Unfair Commercial Practices Directive)**: generic claims such as "eco-friendly", "green", "environmentally friendly", "sustainable" are prohibited unless the trader can demonstrate recognised excellent environmental performance relevant to the claim; a claim whose specification is given in clear and prominent terms on the same medium is not generic. So write "outer made of 100% recycled polyester (per supplier certificate <no.>)" instead of "eco-friendly jacket", and only with the certificate or data present. Also prohibited: sustainability labels not based on a certification scheme or established by public authorities; claims of neutral, reduced or positive greenhouse-gas impact based on offsetting; environmental claims about the whole product when true only of one aspect (packaging vs product).
13. **Legal requirements are not features.** Presenting something required by law for all products of the category in the EU as a distinctive advantage is a blacklisted practice under the same directive. "CE marked" or "complies with EU toy safety rules" can appear in specs, never as a selling point.
14. **Regulated claims need the matching evidence in the data, else flag**: "waterproof" (and the rating: IPX7, a hydrostatic head, a standard), "hypoallergenic", "dermatologically tested", "organic" (certification body/code), "BPA-free", "antibacterial" (biocidal claims have their own EU rules), "medical", "orthopedic". Food health and nutrition claims in the EU are limited to authorised claims (Regulation (EC) No 1924/2006); cosmetics claims must meet the common criteria of Regulation (EU) No 655/2013 (truthfulness, evidential support, honesty, fairness). Any health, therapeutic or disease-related wording on a non-medical product: flag, never soften.
15. **Durability, repairability and performance claims** ("lasts for years", "indestructible", "never needs sharpening") need test data or a stated guarantee in the input; otherwise cut.
16. **Superlatives and comparisons**: no "best", "#1", "most comfortable", no competitor names, no invented reviews, ratings or "customers love".
17. **Textile composition** must be stated with fibre names and percentages as the supplier/label gives them (the EU Textile Regulation (EU) No 1007/2011 governs fibre names); never shorten "95% cotton, 5% elastane" to "cotton".

### EU product-safety information on the listing
18. Under the General Product Safety Regulation (EU) 2023/988, Article 19, an online offer must show the manufacturer's name, postal and electronic address (or the EU responsible person's, if the manufacturer is outside the EU), product identification (including an image and type/batch identifiers) and warnings or safety information in a language easily understood where sold. Many platforms show this in a separate block. Check whether the store's template covers it; if warnings exist on the packaging but not in the input, add `safety info` to `missing`. Do not write warnings yourself.

### Uniqueness and scale
19. **Do not reuse supplier sentences**; rewrite from the attributes. Do not generate near-identical descriptions across variants: a colour variant of the same product shares one description (the variant attribute changes in the specs), or the variants are grouped on one page.
20. **Batch runs**: Google's spam policies define scaled content abuse as many pages generated to manipulate rankings with little value to users, regardless of how the content is created. Every description must carry product-specific facts; if a batch run produces descriptions that differ only in the product name, stop and return the list of products lacking attributes instead.

## Process
1. **Load the voice guide and banned-claims list**; write the voice checklist.
2. **Normalise attributes**: units, labels, duplicates; separate facts from supplier marketing phrases (keep facts, discard phrases).
3. **Classify the product type** and check must-have attributes (rule 1). Missing must-haves → `missing`, continue.
4. **Pick the two deciding attributes**: the ones that most separate this product from siblings in the category and match top support questions or queries. If there is no support/query data, use the matrix order.
5. **Draft**: opening sentence, 2–3 use sentences, answer to the top question, specs list, in-the-box/care/warranty lines.
6. **Claim scan**: highlight every adjective and claim; trace each to an input field; apply rules 12–17. Cut or flag.
7. **Voice and length check** against the checklist and the length range for the type.
8. **Duplication check**: compare against the supplier text and against other descriptions in the batch; rewrite any sentence that matches.
9. **Return** the output block per product; batch runs also return a summary of `missing` by attribute so the owner can fix data at source.

## Pitfalls and edge cases
- **Supplier "water-resistant" vs "waterproof"**: keep the supplier's term and the rating; never upgrade it.
- **Bundles and sets**: list each item with quantity; do not merge specs.
- **Variant-dependent specs** (weight per size, capacity per model): state per variant or give the range with its source; never quote one variant's value for all.
- **Refurbished or used goods**: condition and what was tested/replaced must be stated; never describe as new.
- **Imported spec sheets in another language**: translate attributes, but keep standard names and certificate numbers verbatim.
- **Numbers in the title that conflict with attributes** (title says 1 L, attribute says 900 ml): stop and flag; do not choose.
- **Age-graded products (toys)**: age warnings come from the manufacturer; missing = `missing: safety info`.
- **Energy-labelled appliances**: the energy class and label must be shown under EU energy labelling rules; if absent from input, add to `missing`, do not guess the class.

## Rules
- Never invent a fact, number, certification, comparison, review or warning.
- Missing data goes to `missing`; removed or unsupported claims go to `flags` with the reason.
- Regulated, health, environmental and legal wording is flagged for owner review; the skill does not give legal advice.
- Output is a draft for human approval; never publish to the store or feed.
- Follow the voice guide; keep brand casing exactly.
- Do not copy supplier text.

## Output format
```yaml
voice: <person/address, spelling, sentence length, banned words applied | default (no guide)>
id: <id>
title: <title as given>
type: <product type from matrix>
description: |
  <opening sentence>
  <use sentences>
  <answer to top question, if supported>
specs:
  - <Label>: <value with unit>
in_the_box: [<item>, ...]        # omit if not provided
care: <text>                      # omit if not provided
warranty: <length, provider>      # omit if not provided
word_count: <n> (target <range>)
missing: [<attribute>, ...]
flags:
  - "<claim or issue> - <action taken or owner decision needed>"
sources_used: [<input fields used>]
```

## Worked example
Example data, not a real listing. Input for id 2231: title "Hario V60 Ceramic Dripper 02, White"; brand Hario; material ceramic (Arita-yaki); size 02, 1–4 cups; features spiral ribs, single large hole; country of origin Japan; supplier text includes "keeps coffee hot" and "eco-friendly alternative to capsules". No dimensions, no dishwasher information, no filter-paper compatibility field. Top support question for drippers (helpdesk, 90 days): "Which filter papers fit?" (14 tickets). Voice guide: "you", UK spelling, sentences ≤ 18 words, no exclamation marks.

```yaml
voice: you, UK spelling, sentences <= 18 words, no exclamation marks
id: 2231
title: Hario V60 Ceramic Dripper 02, White
type: tools and equipment (manual brewer)
description: |
  A ceramic pour-over dripper in size 02, for brewing one to four cups by hand.
  The spiral ribs and single large hole leave the flow rate to your pouring.
  It is made of Arita-yaki ceramic in Japan.
specs:
  - Material: Ceramic (Arita-yaki)
  - Size: 02 (1-4 cups)
  - Colour: White
  - Country of origin: Japan
word_count: 37 (target 120-200)
missing: [dimensions, dishwasher-safe, compatible filter papers]
flags:
  - "'keeps coffee hot' - not supported by data, removed"
  - "'eco-friendly alternative to capsules' - generic environmental claim, prohibited without recognised excellent environmental performance; removed"
  - "Below target length - data is thin; add filter-paper compatibility (14 support tickets ask) before publishing"
sources_used: [title, brand, material, size, features, country_of_origin]
```

## Quality checklist
- Every sentence traces to an input field listed in `sources_used`.
- Must-have attributes for the product type checked; gaps in `missing`.
- No generic environmental claim, legal requirement presented as a feature, unsupported regulated claim, superlative, competitor name or review.
- Supplier terms not upgraded (water-resistant stays water-resistant).
- Voice checklist applied; sentence length and person match.
- Length within the type range, or the shortfall explained in `flags`.
- No sentence copied from supplier text or duplicated across the batch.
- GPSR safety/manufacturer information presence checked.

## Sources
- https://eur-lex.europa.eu/eli/dir/2024/825/oj
- https://commission.europa.eu/document/download/3c257883-bb2a-4dd9-a6dc-501d587bb34f_en?filename=faq-empowerting-consumers-gtd.pdf
- https://eur-lex.europa.eu/eli/dir/2005/29/oj
- https://eur-lex.europa.eu/eli/reg/2023/988/oj
- https://eur-lex.europa.eu/eli/reg/2006/1924/oj
- https://eur-lex.europa.eu/eli/reg/2013/655/oj
- https://eur-lex.europa.eu/eli/reg/2011/1007/oj
- https://developers.google.com/search/docs/essentials/spam-policies

## License
MIT
