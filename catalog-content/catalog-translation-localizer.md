---
name: catalog-translation-localizer
owner: tonecheck
category: Catalog content
description: Translates a catalog export in resumable batches with a mandatory glossary, protected brand and placeholder tokens, locale number formats and per-row QA flags, so content managers know exactly which rows need a human.
version: v4
license: MIT
updated: 2026-09-29
recommended: true
security_checked: true
url: https://ecomdly.com/skills/tonecheck/catalog-translation-localizer
raw: https://ecomdly.com/raw/tonecheck/catalog-translation-localizer.md
install: npx @ecomdly/cli add tonecheck/catalog-translation-localizer
---

# Catalog translation localizer

Translates and localises a full catalog export in resumable batches and returns target-language rows plus a per-row flag list of what still needs a human. Raw machine translation gets most words right and fails where it costs money: inconsistent terminology across thousands of SKUs, translated brand and model names, broken placeholders and HTML, silently converted sizes, numbers that change between source and target, and legally regulated wording (textile fibre names, cosmetics ingredients, safety warnings) rendered freely. This skill treats the glossary, the do-not-translate list and the source numbers as hard constraints, and verifies every batch mechanically before it counts as done.

## When to use
- Launching an existing catalog in a new language market (for example `cs-CZ → sk-SK`, `cs-CZ → de-DE`, `en-GB → pl-PL`).
- Keeping a translated catalog in sync: only SKUs created or changed since the last run.
- Preparing localized text for a marketplace listing before `marketplace-listing-adapter` fits it to channel limits.

## When not to use
- Writing new persuasive copy in the target language: use `product-description-writer` with the target locale.
- Category page text: `category-page-copy-writer`.
- Fixing feed attributes or disapprovals: `product-feed-optimizer`, `merchant-center-disapproval-fixer`.
- Legal texts (terms, privacy, withdrawal information): these need a qualified translator and legal review, not this skill.

## Inputs
Required:
- Catalog export (CSV, XLSX or XML) with a stable `id` per row and the fields to translate: title, short and long description, attribute names and values, variant option names, SEO title, meta description, image alt text. Shoptet, Upgates and Shopify all export these; say which export if the user asks how.
- Source and target locale as BCP 47 tags (`cs-CZ → de-AT`, not "German"). Austrian and Swiss German differ in vocabulary and, for Switzerland, in the use of "ss" instead of "ß" and in number format.
- Field length limits per output field, from the destination: feed title (Google Merchant Center `title` 1–150 characters), meta title (practical limit set by the store's SEO owner), platform field limits.

Strongly recommended (ask for them; if missing, create drafts and flag):
- Glossary: source term → approved target term, optionally per category and with part of speech and gender. If none exists, build a draft from the 200 most frequent category nouns and attribute values, return it for approval, and run no more than one pilot batch before it is approved.
- Do-not-translate (DNT) list: brands, product lines, model numbers, trademarks, INCI names, standard designations (EN ISO 20345, IP67), SKU codes.
- Size charts per brand/category with the target size system, if sizes are to be converted.
- Style rules for the target market: formal or informal address (Sie/du, vy/ty), capitalisation of titles, preferred quote marks.

State file: `progress.json` in the working directory, created on the first run.

## Best practices

### Terminology control
1. The glossary is mandatory, not advisory. If the source term appears and the approved target term is absent from the output, the row fails QA. Why: consistency is what shoppers and filters see; "Wanderschuh" on 300 products and "Trekkingschuh" on 200 splits the facets and on-site search.
2. Attribute values (colour, material, size labels) are translated once as a value map and reused, never re-translated per row. Filters in the target store must receive exactly the same string for the same value.
3. Glossary versioning: every change bumps `glossary_version`. Rows translated under an older version are listed for re-check, not silently mixed.
4. Inflecting languages (Czech, Slovak, Polish, German): the glossary entry is the lemma. The output may inflect it; QA checks the stem, not the exact string, and a human reviews stems that did not match.

### Do-not-translate and protected tokens
5. Before translating, mask protected spans: DNT terms, HTML tags and attributes, placeholders (`{name}`, `{{ price }}`, `%s`, `%1$s`, `[shipping]`, Shoptet/Shopify Liquid tags), URLs, e-mail addresses, SKU and model codes, numbers with units. Translate the text around the masks, then unmask. Why: models translate attribute values inside tags, reorder placeholders and "correct" model numbers.
6. Placeholder order may change in the target language only if the placeholders are named; positional ones (`%s`) keep their order.

### Regulated wording
7. Textile fibre names must use the names listed in Annex I of Regulation (EU) 1007/2011 in the language of the target member state, with percentages by weight; the composition must be visible before purchase online (Art. 16). "Merino wool" becomes the regulation's target-language name for wool, not a free translation, and trade names such as "Tencel" do not replace the fibre name (Lyocell).
8. Cosmetics ingredient lists use common ingredient names (INCI) under Regulation (EC) 1223/2009 Art. 19(1)(g); keep them untranslated. Translate the surrounding usage and warnings.
9. Safety warnings and safety information shown in the online offer must be in a language easily understood by consumers as determined by the member state (GPSR, Regulation (EU) 2023/988, Art. 19). Translate them literally, never shorten them to meet a length limit, and flag every row containing a warning for human review.
10. Food, supplements, toys, electrical goods and chemicals carry sector labelling rules. If the catalog contains them, mark mandatory-information fields for professional review rather than finalising them.

### Numbers, units, sizes, prices
11. Every number in the source must appear in the target, either unchanged or as a documented conversion. This is the single most valuable automatic check.
12. Units: EU markets use metric; do not convert between metric units for style. For markets that expect imperial (en-US, partly en-GB for some goods), convert with fixed factors (1 in = 2.54 cm, 1 lb = 0.45359237 kg), round sensibly, and keep the original in brackets for technical products: `30 cm (11.8 in)`.
13. Sizes: convert only with the store's or brand's size chart. No chart means no conversion: keep the source system label (`EU 43`, `UK 9`) and flag. Generic conversion tables are wrong for enough brands to cause returns.
14. Prices and currency: never convert amounts. Prices in the target market come from the target price list (VAT differs by country). Only format numbers found in text, using CLDR locale data (for example a library's `Intl.NumberFormat`) instead of hand rules. CLDR output for 1,299: `de-DE` "1.299,00 €", `cs-CZ` "1 299,00 Kč", `sk-SK` "1 299,00 €", `pl-PL` "1299,00 zł" (Polish does not group four-digit numbers), `fr-FR` "1 299,00 €" with a narrow no-break space. Spaces in these are no-break spaces; keep them.
15. Dates, decimal separators and quotation marks follow the target convention („…" in German and Czech, «…» in French). Check numeric strings that could be dates (`03/04`) and do not reorder them unless the source is unambiguous.

### Length limits
16. Measure in characters of the final string (and bytes where the destination measures bytes). German and Polish output is often longer than English or Czech source; plan title order so that shortening removes the least important tail.
17. Shorten by removing in this order: repeated category words, marketing adjectives, secondary attributes. Never drop brand, product type, the variant attribute or a safety warning. Every shortened row is flagged `shortened`.

### Resumability
18. `progress.json` holds `{source_file_hash, locale_pair, glossary_version, batch_size, last_completed_batch, done_ids_count, total, failed_ids[]}`. A batch is written to the output and only then marked complete, so an interruption repeats at most one batch.
19. Delta runs: compare a hash of each row's source fields with the hash stored at translation time; re-translate only changed rows. A changed source hash on a previously approved row resets its status to "needs review".
20. Batch by category (50–200 rows), so glossary slices and context stay consistent within a batch.

## Process
1. Validate inputs: locale tags, required columns present, `id` unique. Report duplicates and empty source fields before starting.
2. Load or create `progress.json`. If `source_file_hash` or `locale_pair` differs from the stored one, stop and ask whether this is a new run or a delta run.
3. If no approved glossary: build the draft glossary and value maps, translate one pilot batch, return both, and stop for approval.
4. For each batch: mask protected tokens → translate field by field, applying glossary and value maps → unmask → localise numbers, units, dates and quotes → apply length limits → run the per-batch QA (below) → write rows → update `progress.json` → report `done / total`.
5. Per-batch QA, each failing row gets a flag, the batch is still written:
   - Placeholder and tag count equal to source; tags well-formed.
   - Number set check: numbers in source = numbers in target after documented conversions.
   - DNT tokens present byte for byte.
   - Glossary: source term present → approved target stem present.
   - Length within limit.
   - Leftover source-language words not on the DNT list (detect with a source-language stopword list).
   - No new claims: target sentences that have no counterpart in the source (added benefits, "ideal for", certifications) are flagged.
6. After the last batch, run whole-catalog QA: the same attribute value translated two ways, the same title in two rows (duplicates after translation), glossary conflicts.
7. Return the review queue sorted by risk: safety and regulated text first, then numbers, placeholders, glossary, length.

## Pitfalls and edge cases
- Brand names that are ordinary words ("Tatra", "Nature", "Alpine"): only the DNT list tells them apart; if unsure, flag.
- Czech to Slovak looks trivial and is where false friends and untranslated leftovers survive. QA the leftover-word check strictly.
- Gendered and plural agreement in variant names generated from templates (`{colour} {product}`): the concatenation may be ungrammatical in the target language; flag templated rows for review.
- HTML entities (`&nbsp;`, `&amp;`) and escaped quotes in CSV: preserve encoding; round-trip one row through the store's import before running the full catalog.
- SEO fields: translate intent, not words. A literal translation of a keyword that nobody searches in the target market is a finding for the SEO owner, not something to invent keyword data for.
- Unit strings inside model names (`Bottle 750`) are not quantities; the DNT list or a pattern rule must protect them.
- Sizes like `One size`, `Uni`, `S/M` have market-specific conventions; put them in the value map.
- The source contains an error (wrong number, typo in a spec): translate faithfully and flag the suspected error; do not correct it silently.

## Rules
- Read-only towards the store: output is a file for a human to review and import. Never upload to the store, feed or marketplace.
- Never add claims, benefits, certifications or specifications not in the source. Ambiguous source text is flagged, not guessed.
- Never convert prices or sizes without the target price list or a size chart.
- Glossary conflicts are listed, not resolved by the agent.
- Rows with safety warnings or regulated information always require human sign-off.
- Resume from `progress.json`; never restart a run that has completed batches unless the user asks.

## Output format
```
progress: <done> / <total> rows (batch <n> of <m>, glossary v<k>, <source> -> <target>)
id,locale,field,target_text,flags
<id>,<locale>,title,"<text>","<flag;flag>"
...
BATCH QA <n>: rows <r> | placeholder errors <x> | number mismatches <x> | DNT misses <x> | glossary misses <x> | over-length <x> (shortened <y>) | leftovers <x> | new-claim suspects <x>
REVIEW QUEUE (risk order): <id> <field> <flag> <short reason>
```

Flag vocabulary: `placeholder_mismatch`, `number_mismatch`, `dnt_changed`, `glossary_miss`, `glossary_conflict`, `shortened`, `over_length`, `leftover_source`, `size_not_converted`, `regulated_text`, `safety_warning`, `new_claim_suspect`, `source_suspect`, `template_grammar`.

## Worked example
Example: 1,200 rows, `cs-CZ → de-DE`, batch size 100, glossary v2 approved, DNT list includes "Merino Pro", "EN ISO 20345". The run was interrupted after batch 7; `progress.json` shows `last_completed_batch: 7, done_ids_count: 700`. The rerun starts at batch 8.

Source row SKU-2311 title: `Ponožky Merino Pro, 80 % merino vlna, 20 % polyamid, vel. 43–46, 3 páry` (feed title limit 150).

```
progress: 800 / 1200 rows (batch 8 of 12, glossary v2, cs-CZ -> de-DE)
id,locale,field,target_text,flags
SKU-2311,de-DE,title,"Merino Pro Socken, 80 % Wolle (Merino), 20 % Polyamid, Gr. 43–46, 3 Paar","regulated_text"
SKU-2312,de-DE,title,"Merino Pro Socken, 3 Paar, Größe US 9","size_not_converted"
SKU-2340,de-DE,description,"<p>Mit {shipping_note} …</p>",""
BATCH QA 8: rows 100 | placeholder errors 0 | number mismatches 1 | DNT misses 0 | glossary misses 2 | over-length 3 (shortened 3) | leftovers 1 | new-claim suspects 0
REVIEW QUEUE (risk order): SKU-2355 description safety_warning "Nicht für Kinder unter 3 Jahren" check wording | SKU-2319 description number_mismatch source "2 roky záruka", target lacks 2 | SKU-2311 title regulated_text confirm fibre name per Reg. 1007/2011
```
The number check caught SKU-2319 where "2 years warranty" was dropped; SKU-2312 keeps the US size because no brand size chart was supplied.

## Quality checklist
- `progress.json` updated after each written batch; `done / total` matches rows in the output.
- Every row passed or carries flags; batch QA counts equal the number of flags per type.
- No price amount changed; no size converted without a chart.
- DNT tokens byte-identical; placeholders and tags equal in count and well-formed.
- Regulated and safety rows are in the review queue.
- Glossary version recorded on the output; conflicts listed.
- Output encoding UTF-8, no-break spaces preserved in number formats.

## Sources
- Textile Fibre Names Regulation (EU) 1007/2011 (Annex I names, Art. 16): https://eur-lex.europa.eu/eli/reg/2011/1007/oj
- Cosmetics Regulation (EC) 1223/2009, Art. 19: https://eur-lex.europa.eu/eli/reg/2009/1223/oj
- General Product Safety Regulation (EU) 2023/988, Art. 19: https://eur-lex.europa.eu/eli/reg/2023/988/oj
- Google Merchant Center, title attribute (1–150 characters): https://support.google.com/merchants/answer/6324415
- Unicode CLDR locale data: https://cldr.unicode.org/
- BCP 47 language tags: https://www.rfc-editor.org/info/bcp47

## License
MIT
