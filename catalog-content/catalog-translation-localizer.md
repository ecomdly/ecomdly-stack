---
name: catalog-translation-localizer
owner: tonecheck
category: Catalog content
description: Translates thousands of SKUs in resumable batches with a glossary and do-not-translate list, localises units, sizes and currency, then runs a QA pass before import.
version: v3
license: MIT
updated: 2026-09-16
recommended: true
security_checked: true
url: https://ecomdly.com/skills/tonecheck/catalog-translation-localizer
raw: https://ecomdly.com/raw/tonecheck/catalog-translation-localizer.md
install: npx @ecomdly/cli add tonecheck/catalog-translation-localizer
---

# Catalog translation localizer

Machine translation gets a catalog 80% there; the last 20% is terminology, sizes and numbers. This skill works through a full export in batches, keeps your terms consistent across 4 000 products, and tells you exactly what still needs a human.

## When to use
- Launching a store in a new market (DE, FR, PL, CZ...) from an existing catalog export.
- Keeping a translated catalog in sync: only new or changed SKUs since the last run.

## Input
- Catalog export (CSV/XML) with `id`, title, description, attributes, variant names, SEO title/meta.
- Source and target locale, e.g. `en-GB → de-DE`.
- **Glossary:** source term → approved target term (per category if needed).
- **Do-not-translate list:** brand names, product lines, model numbers, trademarks, ingredient INCI names.
- Target market rules: units (metric/imperial), size system (EU/UK/US), currency and decimal format.
- Field length limits (e.g. feed title 150, meta title 60).

## Batch workflow
1. Split the export into batches of 50–200 SKUs, grouped by category so the glossary stays in context.
2. Before each batch, read `progress.json` (`{last_id, done, total, glossary_version}`); skip what is done. A rerun resumes, never restarts.
3. Translate field by field. Glossary terms are mandatory; do-not-translate tokens are copied byte for byte.
4. Localise, do not just translate:
   - Units: convert only when the market expects it, keep the original in brackets for technical products: `30 cm (11.8 in)`.
   - Sizes: map with the store's size chart only. No chart, no conversion, flag it.
   - Prices and currency: never convert amounts; prices come from the target store's price list. Only format (`1.299,00 €`).
   - Dates, decimal separators and quotes in the target convention.
5. After each batch, write the output rows and update `progress.json`, then report `2 480 / 4 000`.

## QA pass (per batch, then whole catalog)
- Placeholders, HTML tags and merge tags intact and equal in count to the source.
- Every number in the source appears in the target (after unit conversion).
- Glossary hits: source term present → approved target term present. Conflicts listed, not resolved silently.
- Length limits respected; over-length rows shortened and flagged.
- Untranslated leftovers (source-language words not on the do-not-translate list).
- Never add claims or benefits the source does not contain; ambiguous source text is flagged, not guessed.

## Output format
```
progress: 2480 / 4000 (batch 25 of 40, glossary v3)
id,locale,title,description,flags
SKU-1188,de-DE,"Merino-Wandersocken Crew, 3er-Pack, Grau, Gr. 43-46","...",""
SKU-1189,de-DE,"Trailrunning-Schuh Speedcross 6 GTX","...","size chart missing: US 9 not converted"
QA: 200 rows · 0 placeholder errors · 3 glossary conflicts · 2 over-length titles
```

## License
MIT
