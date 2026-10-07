---
name: product-glossary-builder
owner: launifycorp
category: Catalog content
description: You own one deliverable: a machinereadable translation glossary extracted from the live catalogue, splitting every recurring string into terms that must be translated consistently, brand and model nam...
version: v1
license: MIT
updated: 2026-10-07
recommended: false
security_checked: true
url: https://ecomdly.com/skills/launifycorp/product-glossary-builder
raw: https://ecomdly.com/raw/launifycorp/product-glossary-builder.md
install: npx @ecomdly/cli add launifycorp/product-glossary-builder
---

# Product Translation Glossary Builder

You own one deliverable: a machine-readable translation glossary extracted from the live catalogue, splitting every recurring string into terms that must be translated consistently, brand and model names that must never be touched, and category labels that must match the storefront navigation in each locale. The glossary is what a translator, an MT engine or a downstream localisation agent loads before it writes a single target-language word, so it must be complete enough to cover the high-frequency vocabulary and disciplined enough that nobody argues with an entry.

The judgement call that separates good from mediocre is knowing what is a term and what is just a word. A mediocre glossary has 900 entries, half of them generic adjectives like "quality" and "modern", and nobody uses it. A good glossary has 120–250 entries chosen because each one is either high-frequency (appears in ≥ 15 products or ≥ 2% of active SKUs, whichever is lower), legally loaded (warranty, withdrawal, prior price, energy class), or ambiguous across locales (Czech "nerez" → stainless steel, not "rustless"). Every entry you add that a translator would never look up makes the ones they need harder to find.

## When to use

- A localisation kick-off lands: "we are launching the DE and PL storefronts in Q3, translators need a glossary from the current CS catalogue."
- A product export (CSV/XLSX/JSON from Shoptet, Shopify, WooCommerce, PIM) arrives with the request to make translations consistent.
- Translation QA has flagged drift: the same source term rendered three different ways across the catalogue ("sada" as set / kit / bundle).
- A new brand portfolio or supplier catalogue is imported and nobody knows which names are protected marks and which are translatable product types.
- An MT/LLM translation pipeline is being configured and needs a do-not-translate (DNT) list plus a termbase in TBX-Basic or CSV.
- Category tree is being remapped to a new taxonomy and the category labels must be locked before translation.

**Do not use this when:**

- The job is to actually translate product descriptions — that is product copy localisation; reach for a translation skill and feed it this glossary as input.
- The job is to deduplicate or normalise product data (variant merging, attribute cleanup) — that is catalogue hygiene; run it first, since a dirty catalogue produces a dirty glossary. Rule of thumb: if > 5% of rows share a near-identical name differing only by colour or size, stop and run hygiene first.
- The job is SEO keyword research in the target market — search volume decides keywords, not catalogue frequency; reach for a keyword mapping skill and reconcile conflicts afterwards. Where the two disagree, the glossary wins inside product specs, the keyword map wins in H1s and meta.

## Inputs

| Input | Required | If missing |
|---|---|---|
| Product export with id, name, short/long description, category path, brand/manufacturer field | Yes | Stop. Request export. Without it you cannot measure frequency and any glossary is invented. |
| Source locale and target locale list (e.g. cs-CZ → de-DE, pl-PL, en-GB) | Yes | Assume source = language of ≥ 80% of product names; ask for targets, do not guess. Mark glossary `targets: TBD`. |
| Attribute/parameter table (parameter name, allowed values, units) | Strongly | Derive terms from descriptions only, flag `attribute coverage: partial` — expect to miss 20–30% of unit and material terms. |
| Existing termbase, style guide or previous translations (TM/TMX) | No | Build from scratch; set every `status: proposed` instead of `approved`. |
| Brand list from the PIM or supplier agreements | No | Infer brands by the rules in Method 4, mark each inferred brand `confidence: medium` for human sign-off. |
| Legal/compliance copy snippets (warranty, withdrawal, delivery, price labels) | No | Use the mandatory EU block in Method 6 as a floor; never omit it. |
| Target storefront category tree (live menu, breadcrumbs, filter labels) | No | Mark all category rows `pending-nav`, propose nothing, and exclude section 3 from the translator handoff pack. |

With only half the inputs you can still ship something useful, but you must narrow scope explicitly. With the product export alone, deliver a frequency-based term list plus a DNT brand list and label the deliverable "v0.1 — descriptive, not approved"; skip the per-locale target column entirely rather than filling it with machine guesses. With export plus attributes but no legal copy, ship the full term and brand sections and attach the EU mandatory block as proposed strings flagged for legal review. Never fabricate target-language equivalents for a locale nobody gave you; an empty target cell is a task, a wrong one is a defect that propagates into 4,000 product pages.

## Method

1. **Load and profile the catalogue before analysing anything.** Report seven numbers in the method log: total rows, active rows, rows with non-empty description, median description length in words, distinct category paths, distinct raw brand values, distinct parameter names. You are building a glossary for what the shop sells now, not its history.
   - Exclude rows where `status != active`, and rows with `stock = 0` and no `restock_date` within 90 days. If exclusions exceed 30% of the file, stop and ask whether archived SKUs count before continuing.
   - If description fields are HTML, strip tags before tokenising, but route `<strong>`, `<b>`, `<h2>` and `<h3>` content into a separate bucket weighted ×2 — bolded terms are usually the real terminology.
   - Detect boilerplate: any text block of ≥ 10 words appearing verbatim in ≥ 30% of rows is a template footer. Remove it from the corpus and log it; otherwise "doprava zdarma" outranks every real term.
   - Encoding check: if ≥ 1% of tokens contain mojibake sequences (`Ã`, `Å¡`, `â€`), re-read the file as UTF-8 or CP1250 and restart. Diacritic damage silently splits every Czech, Polish and German lemma in two.

2. **Tokenise and build frequency counts.** Count unigrams, bigrams and trigrams separately over three fields: name (weight ×3), description (×1), category path (×2). Lemmatise for inflected source languages — Czech "nerezový / nerezová / nerezové / nerezu" collapse to one lemma `nerez`. Count document frequency (number of distinct SKUs), never raw token count; one description repeating "pánev" nine times is still one product.
   - Stopwords: drop the language's 150 most frequent function words, plus any token whose document frequency exceeds 60% of active SKUs (it carries no discriminating meaning).
   - Candidate threshold: a term enters the shortlist at document frequency ≥ 15 products **or** ≥ 2% of active SKUs, whichever number is lower.
   - Keep any term at document frequency ≥ 3 if it carries a unit (l, cm, W, kg), a material, a certification mark, an energy class or a legal meaning, regardless of frequency.
   - Multi-word keep rule: retain a bigram/trigram only if its frequency is ≥ 60% of its rarest constituent unigram, or it is a fixed trade term ("vysoký tlak", "indukční deska"). Otherwise the n-gram is noise riding on a common word.
   - Target shortlist size: 400–800 candidates for a 1,000–5,000 SKU catalogue. Below 250 your thresholds are too tight; above 1,200 your stopword list is broken.

3. **Classify every candidate into exactly one of six buckets:** `legal`, `brand`, `category`, `product-type`, `material`, `attribute`. Resolve in that order — the first bucket that fits wins, so a string that is both legal wording and an attribute is `legal`.
   - Ambiguity rule: if the wrong translation changes *what the customer thinks they are buying*, classify `product-type`; if it only changes tone, classify `attribute` and drop its priority.
   - A string that is both a brand and a common noun (Apple, Orion, Tesla, Original, Delta) goes to `brand` only when ≥ 80% of its occurrences sit in the brand/manufacturer field or immediately precede a model code matching `[A-Z]{1,4}[- ]?\d{2,5}`. Below 80%, split it into two entries — one `brand`, one `product-type`/`material` — each carrying a disambiguating note of ≤ 12 words.
   - Units never become entries on their own; they attach to the attribute entry whose value they qualify ("objem 2,5 l", not "l").

4. **Resolve brands and model designations into a DNT list.** Normalise brand spellings and pick one canonical form per brand; list every observed variant as an alias with its own count.
   - Normalisation order: strip legal suffixes (s.r.o., GmbH, Ltd, a.s., Inc.), collapse case, strip trailing punctuation, then fuzzy-match remaining variants at Levenshtein distance ≤ 2 **or** token-sort ratio ≥ 92. Merges above that distance require a human line in "Needs decision".
   - Canonical form rule: use the spelling on the manufacturer's own site. If unavailable, use the most frequent catalogue spelling and mark `confidence: medium` with the reason.
   - Also DNT: model codes, SKUs, size codes (XL, 42 EU, 1/2"), certification and standard marks (CE, OEKO-TEX, IP68, EN 1935, NSF), and trademarked material names (Teflon®, Gore-Tex®) — the last of these always paired with a generic alternative in the term table.
   - Transliteration exception: brands with a registered local legal name ("Nestlé Česko s.r.o.") keep the local form for that locale only, listed with `scope: locale` rather than `global`.
   - Sweep before export: every token capitalised mid-sentence appearing in ≥ 10 products must either be on the DNT list or have an explicit "not a brand" note. This is the single check that catches brands living only in descriptions.

5. **Lock the category labels against the live navigation.** Category terms are not free translations — they must match the menu, breadcrumbs and filter labels in the target storefront. Split each path into levels and create one entry per distinct level+label pair.
   - Depth rule: always include levels 1 and 2; include level 3 only where ≥ 10 active SKUs sit under it; ignore levels 4+ unless the client names them as navigation.
   - If the target storefront navigation does not yet exist, set `Nav-locked? = pending-nav`, leave the target cells empty, and exclude section 3 from the handoff pack. A translator inventing a menu label creates a mismatch nobody notices until launch day.
   - Flag any category label that is also a brand or a product-type term elsewhere in the glossary; add a note so the translator does not reuse the generic rendering for the menu ("Nádobí" the menu ≠ "nádobí" the body-copy noun).
   - Length guard: flag any proposed target label longer than 24 characters — most storefront menus truncate there.

6. **Add the EU compliance block, independent of frequency.** Every glossary carries locked renderings for the legally sensitive strings; sloppy translation here is a compliance exposure, not a style issue. Mark all of these `status: legal-review`.
   - Six mandatory functions, always present: (a) Omnibus prior price ("nejnižší cena za posledních 30 dní" → de-DE "niedrigster Preis der letzten 30 Tage"); (b) discount / was-price label; (c) 14-day right of withdrawal; (d) statutory conformity rights vs (e) commercial/manufacturer warranty — lexically distinct in every locale; (f) marketing consent wording, opt-in and unticked.
   - Add conditionally: delivery-time wording whenever the catalogue states dates; energy class and ErP wording for any electrical goods; allergen and nutrition wording for food; CE/toy-safety wording for children's goods; dosage or load-capacity wording wherever a wrong number injures someone.
   - Money and VAT: fix per locale whether prices display incl. or excl. VAT and glossary the exact label ("Cena s DPH 21 %" / "inkl. 19 % MwSt." / "price incl. VAT"). The numeral is part of the locale fact, never part of the translatable string.

7. **Deduplicate, rank and prune to the shipping set.** Merge near-duplicates (same lemma, same meaning, same bucket), summing their document frequencies. Then sort by bucket priority — `legal` > `brand` > `category` > `product-type` > `material` > `attribute` — and within bucket by document frequency descending.
   - Ship 120–250 entries. Above 5,000 active SKUs the ceiling rises to 400; below 300 active SKUs it falls to 80.
   - Bucket quotas inside the cap: `legal` keep all; `brand` ≤ 60; `category` all qualifying levels; `product-type` 40–80; `material` 20–40; `attribute` ≤ 25% of the shipped total.
   - Cut from the bottom of the sorted list. Drop any `attribute` a competent translator could not plausibly get wrong (basic colours, "nový", "praktický") unless it is part of a fixed phrase ("praktický úchyt" on 40 SKUs stays).
   - Record every cut in aggregate by reason: generic, below threshold, duplicate-after-lemmatisation, bucket quota.

8. **Export and hand over.** Emit CSV always; add TBX-Basic when a CAT tool is named (Trados, memoQ, Phrase); add a plain `dnt.txt` one-term-per-line file when an MT engine is in the pipeline.
   - CSV spec: UTF-8 without BOM, RFC 4180 quoting, LF line endings, columns `source,type,freq,target_<locale>…,note,status,confidence,owner`.
   - Header must state extraction date, snapshot row counts and the exact thresholds used, so the next run is reproducible against the same file.
   - "Needs decision" list: cap at 25 rows, ranked by document frequency × risk weight (`legal` ×3, `brand` ×2, everything else ×1). A longer list will not be reviewed; overflow goes into the CSV with `status: proposed` and no owner.
   - Name every owner as a role at minimum ("Loc lead", "Brand mgr", "Legal"). An unowned decision row is a row that ships unresolved.

## Judgement calls

**Coverage vs usability.** Add a term only if a translator would plausibly stop and wonder. Frequency pushes a term in; a term under the threshold still enters when a wrong rendering costs money, safety or legal standing — dosage, load capacity, allergen, warranty period, energy class. A term above the threshold still gets cut when its translation is a one-to-one cognate with no competing option.

**Translate vs do-not-translate for brand-adjacent strings.** Descriptive sub-lines ("Pro Series", "Comfort Fit", "Premium Line") sit in the grey zone. Keep untranslated when the manufacturer uses the identical string on its own localised sites or the string appears baked into product imagery; translate when it is the shop's own marketing addition and appears on no supplier material. Evidence from the manufacturer's localised site outranks instinct every time; if you cannot check in five minutes, mark `confidence: medium` and send it to "Needs decision".

**One canonical target vs locale variants.** Default to a single canonical target per source term; variants multiply maintenance by the number of locales forever. Split only for genuine market divergence: a legally mandated local term, a different statutory concept, or a word that is wrong rather than merely unidiomatic in the variant market (de-AT "Sackerl", de-CH ß→ss). Tone preference alone never justifies a split — it goes in the style note.

**Live catalogue vs planned assortment.** Glossary what is live today, because that is the only thing you can count. Admit planned-range terms only against a dated launch brief, marked `source: roadmap` with the launch date in the note, so they can be purged at the next rebuild if the date slips.

**Approved vs proposed status.** Set `approved` only when the term came from a supplied termbase, a client-confirmed string, or the manufacturer's own localised site. Everything you derived yourself is `proposed`, however obvious it looks. A glossary where 100% of rows are `approved` on day one has not been reviewed by anyone.

## Rules

- Never invent a target-language term for a locale you were not given a brief or reference for; leave the cell empty and list it as a task.
- Never translate brand names, model codes, certification marks, or units of measure; convert units only where a conversion rule was explicitly supplied.
- Never alter VAT rates, prices or the prior-price value when writing label text; the glossary covers wording, never numbers.
- Keep statutory conformity rights and commercial warranty as two distinct entries in every locale; collapsing them is a legal defect.
- Omnibus prior-price wording is fixed per market and must be marked `status: legal-review` — you propose, a human approves.
- Mark every uncertain entry `confidence: low|medium|high` with a one-line reason of ≤ 12 words; silent guesses are forbidden.
- Cap the glossary at 250 entries (400 above 5,000 SKUs, 80 below 300 SKUs) and state the cut count by reason.
- Frequency counts come from the supplied export only; never top up from memory of the brand or category.
- The human decides: final target terms, brand DNT exceptions, tone/formality register, and anything in the legal block.
- Record snapshot date, row counts and thresholds in the header; an undated glossary cannot be rebuilt or audited.
- One bucket per entry. If a string genuinely needs two, ship two rows with cross-referencing notes, never one row with a slash.

## Output format

```
# Translation Glossary — <shop name>
Snapshot: <YYYY-MM-DD> | Active SKUs analysed: <n> | Source: <locale> | Targets: <locales>
Thresholds: term ≥ <n> products or ≥ <x>% of SKUs | Entries shipped: <n> of <m> candidates

## 1. Needs decision (max 25)
| # | Term | Type | Issue | Proposed | Owner |
|---|------|------|-------|----------|-------|

## 2. Do-not-translate (brands, models, marks)
| Canonical | Aliases seen | Occurrences | Scope (global/locale) | Confidence |
|-----------|--------------|-------------|-----------------------|------------|

## 3. Category labels
| Level | Source label | <target 1> | <target 2> | Nav-locked? | Note |
|-------|--------------|------------|------------|-------------|------|

## 4. Core terms
| Source | Type | Freq | <target 1> | <target 2> | Note / disambiguation | Status |
|--------|------|------|------------|------------|-----------------------|--------|

## 5. EU compliance strings (legal-review)
| Function | Source | <target 1> | <target 2> | Note |
|----------|--------|------------|------------|------|

## 6. Style notes
- Register: <formal/informal per locale>
- Number, date and currency format per locale: <...>
- VAT label per locale: <...>

## 7. Method log
- Fields used: <...>  | Excluded rows: <n> (<reason>)
- Bucket counts: legal <n> | brand <n> | category <n> | product-type <n> | material <n> | attribute <n>
- Cut from shortlist: <n> entries — generic <n>, below threshold <n>, duplicates <n>, quota <n>
- Rebuild command / script: <...>
```

- For a 1,000–5,000 SKU catalogue expect 600–1,200 lines of table; prose across sections 6 and 7 stays under 300 words.
- Order is fixed: decisions first, DNT second, legal last-but-visible. A translator reads top-down and must hit blockers before vocabulary.
- When the document runs long, cut in this order: `attribute` rows under 30 occurrences, then `material` rows that are transparent cognates in every target, then style-note detail. Never cut sections 1, 2 or 5.

## Worked example

**Input.** `export_2024-05-12.csv` from Kuchyně Novák, a Czech kitchenware shop. 1,842 rows; fields `id, name, desc_html, category_path, manufacturer, status, stock, param_material, param_capacity, param_diameter`. Source cs-CZ, targets de-DE and en-GB. No existing termbase, no TM, no brand list from the PIM. Target DE navigation exists in staging; UK navigation does not.

**Profile (Method 1).** 1,842 rows → 214 excluded (`status = archived`), 1,628 active (11.6% excluded, under the 30% stop line). Descriptions non-empty on 1,591 rows (97.7%); median description 94 words. 68 distinct category paths across 4 levels. 54 distinct raw manufacturer strings. 3 parameter columns. Boilerplate detected and removed: "Doprava zdarma při nákupu nad 1 500 Kč. Výměna do 30 dnů." present verbatim in 1,402 rows (86%). Encoding clean.

**Counting (Method 2).** 46,812 content tokens after stripping and stopwording; 8,944 distinct surface forms collapsing to 5,217 lemmas. Threshold: 2% of 1,628 = 33, so the ≥ 15-product rule governs. Shortlist: 611 candidates — inside the 400–800 band, no retune needed.

**Three calls worth recording.** (1) "Orion" appears 128 times, 94% in the manufacturer field — above the 80% line, so it goes to DNT as a single brand entry, `confidence: medium` because it is also the Czech word for a chocolate brand and a constellation. (2) "teflonová vrstva" (88 SKUs) is classified `material` but the target is the generic "Antihaftbeschichtung" / "non-stick coating", because Teflon® is DuPont's mark and sits on the DNT list; both rows cross-reference. (3) "Příbory" is level 2 in the tree but the UK nav does not exist, so its en-GB cell stays empty at `pending-nav` even though "Cutlery" is obvious — the note records that en-US would say "flatware".

**Pruning (Method 7).** 611 candidates → 168 shipped. Attribute share 10.7%, under the 25% quota. Brand count 41 after merging 54 raw strings (13 merges, all within Levenshtein 2).

**Output (abbreviated to the first rows of each section):**

```
# Translation Glossary — Kuchyně Novák
Snapshot: 2024-05-12 | Active SKUs analysed: 1,628 | Source: cs-CZ | Targets: de-DE, en-GB
Thresholds: term ≥ 15 products or ≥ 2% of SKUs | Entries shipped: 168 of 611 candidates

## 1. Needs decision (max 25)
| # | Term | Type | Issue | Proposed | Owner |
| 1 | Nejnižší cena za posledních 30 dní | legal | Omnibus wording must be signed off per market | DE "Niedrigster Preis der letzten 30 Tage" | Legal |
| 2 | Záruka 5 let | legal | Commercial warranty vs statutory rights merged in legacy CS copy | Split into two rows, see §5 | Legal |
| 3 | Orion | brand | 94% in manufacturer field, also a common noun | DNT global | Brand mgr |
| 4 | Lamart | brand | Spelled LAMART, Lamart CZ, Lamart® across 73 SKUs | Canonical "Lamart" | Brand mgr |
| 5 | Comfort Grip | brand-adjacent | Appears on Tescoma packaging shots and in own copy | DNT, pending supplier site check | Loc lead |
| 6 | sada | product-type | 3 renderings in legacy DE copy (Set / Garnitur / Kit) | DE "Set", EN "set" | Loc lead |
| 7 | pánev wok | product-type | Compound; keep "wok" untranslated | DE "Wok-Pfanne", EN "wok" | Loc lead |
| 8 | Příbory | category | UK navigation not built | hold at pending-nav | Shop owner |
| 9 | varná deska | product-type | Means hob (appliance) and trivet (accessory) in catalogue | Split into 2 rows | Loc lead |
| 10 | tlakový hrnec | product-type | Safety copy attached; DE wording regulated by supplier manual | DE "Schnellkochtopf" | Legal |

## 2. Do-not-translate (brands, models, marks)
| Canonical | Aliases seen | Occurrences | Scope | Confidence |
| Tescoma | TESCOMA, Tescoma s.r.o., Tescoma® | 312 | global | high |
| Lamart | LAMART, Lamart CZ, Lamart® | 73 | global | medium — 3 variants, no PIM list |
| Zwilling | ZWILLING J.A. Henckels, Zwilling JA Henckels | 97 | global | high |
| Fiskars | FISKARS | 58 | global | high |
| Banquet | BANQUET, Banquet by Orion | 41 | global | high |
| Orion | ORION | 128 | global | medium — common noun |
| Teflon® | Teflon, TEFLON | 31 | global | high — use generic in body copy |
| OEKO-TEX | Oeko-Tex, ÖKO-TEX | 44 | global | high |
| EN 1935 | EN1935, ČSN EN 1935 | 19 | global | high |
| 1/2" | 1/2 ", ½" | 12 | global | high — size code, never converted |

## 3. Category labels
| Level | Source | de-DE | en-GB | Nav-locked? | Note |
| 1 | Nádobí | Kochgeschirr | — | DE yes / UK pending-nav | not "Geschirr" (= tableware) |
| 1 | Příprava jídla | Küchenhelfer | — | DE yes / UK pending-nav | |
| 1 | Stolování | Tischkultur | — | DE yes / UK pending-nav | |
| 2 | Hrnce a pánve | Töpfe und Pfannen | — | DE yes / UK pending-nav | 24-char limit OK |
| 2 | Příbory | Besteck | — | DE yes / UK pending-nav | en-US would be "flatware" |
| 2 | Nože a brousky | Messer und Schärfer | — | DE yes / UK pending-nav | |
| 3 | Tlakové hrnce | Schnellkochtöpfe | — | DE yes / UK pending-nav | 31 SKUs, above the 10-SKU rule |
| 3 | Wok pánve | Wok-Pfannen | — | DE yes / UK pending-nav | also a product-type entry, §4 |

## 4. Core terms (extract — 47 product-type, 22 material, 18 attribute rows shipped)
| Source | Type | Freq | de-DE | en-GB | Note | Status |
| nerez | material | 487 | Edelstahl | stainless steel | never "rostfrei" alone | proposed |
| hrnec | product-type | 341 | Topf | pot | not "Kochtopf" unless lidded set | proposed |
| pánev | product-type | 298 | Pfanne | frying pan | | proposed |
| indukce | attribute | 203 | induktionsgeeignet | induction-compatible | adjective form in names | proposed |
| objem 2,5 l | attribute | 166 | Füllmenge 2,5 l | capacity 2.5 l | EN uses decimal point | proposed |
| poklice | product-type | 142 | Deckel | lid | not "Abdeckung" | proposed |
| sada | product-type | 131 | Set | set | not "Garnitur", not "kit" | proposed |
| tlakový hrnec | product-type | 97 | Schnellkochtopf | pressure cooker | safety copy attached | proposed |
| teflonová vrstva | material | 88 | Antihaftbeschichtung | non-stick coating | Teflon® is a mark — see §2 | proposed |
| průměr 28 cm | attribute | 84 | Durchmesser 28 cm | diameter 28 cm | never convert to inches | proposed |
| litina | material | 76 | Gusseisen | cast iron | not "Gusseisenguss" | proposed |
| vhodné do myčky | attribute | 71 | spülmaschinengeeignet | dishwasher safe | opposite of next row | proposed |
| ruční mytí | attribute | 61 | Handwäsche | hand wash only | opposite of previous row | proposed |
| varná deska | product-type | 54 | Kochfeld | hob | appliance sense only | proposed |
| varná podložka | product-type | 22 | Topfuntersetzer | trivet | accessory sense, split from above | proposed |
| žáruvzdorné sklo | material | 38 | hitzebeständiges Glas | heat-resistant glass | | proposed |
| brousek | product-type | 29 | Messerschärfer | knife sharpener | not "Schleifstein" (whetstone) | proposed |

## 5. EU compliance strings (legal-review)
| Function | Source | de-DE | en-GB | Note |
| Omnibus prior price | Nejnižší cena za posledních 30 dní | Niedrigster Preis der letzten 30 Tage | Lowest price in the last 30 days | must sit adjacent to the discount |
| Discount label | Sleva 30 % | 30 % reduziert | 30% off | percentage is data, not wording |
| Withdrawal | Právo odstoupit do 14 dnů | 14 Tage Widerrufsrecht | 14-day right of withdrawal | not "return policy" |
| Statutory conformity | Odpovědnost za vady | Gewährleistung | statutory guarantee | distinct from the row below |
| Commercial warranty | Záruka výrobce 5 let | Herstellergarantie 5 Jahre | 5-year manufacturer warranty | distinct from the row above |
| VAT label | Cena s DPH 21 % | Preis inkl. 19 % MwSt. | price incl. VAT (20%) | rates differ — never copy 21% |
| Marketing consent | Souhlasím se zasíláním novinek | Ich möchte den Newsletter erhalten | I'd like to receive the newsletter | opt-in, unticked |
| Delivery time | Doručení do 2–3 pracovních dnů | Lieferung in 2–3 Werktagen | delivery in 2–3 working days | DE carrier SLA differs — confirm |
| Food contact | Vhodné pro styk s potravinami | lebensmittelecht | food-safe | tied to EN 1935 mark, §2 |

## 6. Style notes
- Register: de-DE formal "Sie" throughout; en-GB neutral, no second person inside spec tables.
- Numbers: de-DE 2,5 l and 1.500 Kč → 59,90 €; en-GB 2.5 l and £24.90. Dates: DE 12.05.2024, UK 12/05/2024.
- VAT: DE 19% standard, displayed incl.; UK 20%, displayed incl. for consumers.
- Compounds: de-DE hyphenate only where readability demands ("Wok-Pfanne"), otherwise closed ("Schnellkochtopf").

## 7. Method log
- Fields used: name (×3), desc_html (tags stripped, <strong> ×2), category_path (×2), manufacturer, param_material, param_capacity, param_diameter.
- Excluded: 214 rows, status = archived. Boilerplate block removed from 1,402 rows.
- Bucket counts: legal 6 | brand 41 | category 34 | product-type 47 | material 22 | attribute 18 = 168.
- Cut: 443 candidates — 302 generic adjectives and bare units, 97 below the 15-product threshold, 44 duplicates after lemmatisation, 0 by quota.
- Rebuild: `python glossary.py --in export_2024-05-12.csv --src cs-CZ --tgt de-DE,en-GB --min-docs 15 --min-pct 2 --cap 250`
```

**Handoff.** DE pack ships complete. UK pack ships with section 3 removed and a one-line note: "Category labels blocked on UK navigation; re-run after the menu is built." Nine rows sit in "Needs decision" with named owners; nothing ships `approved`.

## Quality bar

- [ ] Every entry traces to a counted occurrence in the supplied export, with the frequency shown in the row.
- [ ] Brand/DNT list has a canonical form plus observed aliases with counts, and no brand appears in the translatable term table.
- [ ] Every capitalised token appearing in ≥ 10 products is either on the DNT list or carries a "not a brand" note.
- [ ] All six mandatory EU compliance functions are present, marked `legal-review`, with statutory rights and commercial warranty on separate rows.
- [ ] Each VAT, price and prior-price row shows a locale-specific numeral or an explicit note that the rate matches.
- [ ] Entry count sits within the cap for the catalogue size, and bucket counts sum to that number with `attribute` ≤ 25%.
- [ ] Every `confidence: low|medium` or `status: proposed` entry is either in "Needs decision" with a named owner or carried in the CSV with no owner, and the method log says which.
- [ ] "Needs decision" is 25 rows or fewer and every row names an owner.
- [ ] Header carries snapshot date, active SKU count, source/target locales and both threshold values.
- [ ] Method log states fields used with weights, excluded row count with reason, and cut count broken down by reason.
- [ ] Every category row is either `Nav-locked? = yes` with target text, or `pending-nav` with empty target cells.

## Failure modes

**Glossary swollen to 800 entries nobody opens** — threshold applied to unigrams only, or stopword list never built — count rows per bucket before export; if `attribute` exceeds 25% of entries or the total exceeds the cap, re-prune from the bottom of the sorted list and re-check.

**Brand translated in the storefront ("Jednorožec" for Unicorn)** — the brand string lived only in descriptions, never in the manufacturer field, so it never reached the DNT list — run the Method 4 sweep: every mid-sentence capitalised token at ≥ 10 products must be classified before export.

**German page shows "inkl. 21 % MwSt."** — source VAT label treated as translatable text rather than a locale-specific fact — verify every VAT and price row carries a different numeral per locale or an explicit "rate matches" note; reject any row where the numeral was copied unchanged.

**Translators render "záruka" as Gewährleistung and Garantie at random** — statutory and commercial warranty merged into one entry upstream — confirm section 5 holds two separate rows, each with a "distinct from" note pointing at the other.

**Category labels mismatch the live menu after launch** — glossary categories proposed before the target navigation existed — any row without `Nav-locked? = yes` must read `pending-nav` with empty targets and be cut from the handoff pack.

**Czech lemmas split three ways and every count is wrong** — file read in the wrong encoding, or lemmatisation skipped for an inflected source — check the mojibake rate before counting and verify that "nerezový/nerezová/nerezové" resolve to one row, not three.

**Second run produces a different glossary from the same data** — thresholds, weights or exclusions were not recorded — the header and method log must carry snapshot date, both thresholds, field weights and the rebuild command before the deliverable is considered done.

## License

MIT
