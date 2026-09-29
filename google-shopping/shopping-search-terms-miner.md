---
name: shopping-search-terms-miner
owner: adsledger
category: Google Shopping
description: Mines Shopping and Performance Max search terms into collision-tested negatives (match type and level), watch terms and winners to protect, using n-grams and a sample-size rule instead of guesswork; for PPC managers.
version: v4
license: MIT
updated: 2026-09-29
recommended: false
security_checked: true
url: https://ecomdly.com/skills/adsledger/shopping-search-terms-miner
raw: https://ecomdly.com/raw/adsledger/shopping-search-terms-miner.md
install: npx @ecomdly/cli add adsledger/shopping-search-terms-miner
---

# Shopping search terms miner

Turns the search terms report of Standard Shopping and Performance Max campaigns into three lists: negatives that are safe to add (with match type and level), terms and n-grams to watch, and converting terms worth protecting. Shopping has no keywords, so the search terms report is the only view of what the account pays for. The naive approach sorts single terms by cost and negates the top of the list. That misses most waste, which sits in hundreds of low-cost terms sharing one word ("repair", "used", "manual", a size the store does not stock), and it creates damage, because negative keywords do not match close variants and one careless phrase negative can block converting queries. This skill aggregates by n-gram, judges waste with a sample-size rule instead of a hunch, and tests every proposed negative against the converting terms before it is proposed.

## When to use

- Every 2 weeks on Standard Shopping with meaningful spend; monthly on PMax.
- After a feed title rewrite or a new product line, to see which queries changed.
- When Shopping CPC or cost grows without matching conversion value.

## When not to use

- Search campaigns with keywords: the logic is similar, but keyword match types and ad group routing add steps this skill does not cover.
- Brand-traffic policy (should PMax serve on the store's brand at all): that is a brand exclusion decision; see `pmax-asset-group-reviewer`.
- Deciding the ROAS target itself: `break-even-roas-calculator`.

## Inputs

Required:

1. **Search terms report**, 30–90 days, one row per search term × campaign (× ad group for Standard Shopping): search term, campaign, ad group, impressions, clicks, cost, conversions, conversion value. Source: Google Ads > Insights and reports > Search terms; for PMax choose the view "Search terms and landing pages for Performance Max" (data exists from March 2023). Download all rows, not the on-screen top.
2. **Campaign totals** for the same period (cost, conversions, value). Needed to measure how much cost the report does not show.
3. **Target**: tROAS or target CPA per campaign, and whether conversion value is ex or incl. VAT.
4. **Protected terms**: the store's own brand names, brands it sells, hero product names and model numbers.

Optional: existing negatives (campaign, ad group, shared lists, account-level list) with match types; catalog export (titles, brands, sizes, `product_type`) to test "we don't sell this"; Standard Shopping campaign priorities; search terms insights (themes) for PMax.

If conversion value is missing, judge by CPA and say so in the header. If the target is missing, ask; do not assume one.

## Best practices

### What the data can and cannot show

1. **The report is incomplete by design.** Google omits search terms without enough query activity for privacy reasons. Compute hidden cost = campaign cost − sum of cost across reported terms, and state it as a share. If a large share is hidden, the negative list can only fix the visible part; say so instead of implying full coverage. Search terms insights group low-volume queries into themes and "other queries" and can show where hidden cost goes.
2. **Shopping terms carry no keyword.** In the report, Shopping-matched terms have an empty keyword field and "Exact" in the match type column. That "Exact" does not mean exact match targeting; ignore it.
3. **Conversions lag.** Terms from the last days show clicks before their conversions. Exclude the last lag window (the store's typical time to purchase) from zero-conversion judgments.

### How negatives match (the part that causes damage)

4. **Negative match semantics differ from positive keywords.** Negative broad: blocks when the query contains all the negative's words in any order. Negative phrase: blocks when the query contains the words in the same order, with other words allowed around them. Negative exact: blocks only the identical query with no extra words.
5. **Negatives do not match close variants.** Google's example: negative broad `flowers` blocks "red flowers" but not "red flower". Add singular, plural and synonyms explicitly. Casing and misspellings are handled automatically, but **accented and unaccented forms are different negatives** (`cafe` vs `café`). For Czech, Slovak, German or Polish stores, add both forms: `bazar` and `bazár`, `levne` and `levné`.
6. **Word limit.** A negative that appears after the 16th word of a long query does not block it. Rare, but explains "why did this still show".
7. **Symbols.** `&`, accents and `*` are recognised; periods are ignored; `site:`, `OR` and a leading `-` are stripped. Some symbols (`, ! @ % ^ ( ) = { } ; ~ < > ? \ |`) are invalid and will be rejected; rewrite the term without them.
8. **Choose the match type by the unit of waste.**
   - A single word that is never commercial for this store (`repair`, `manual`, `jobs`, `rental`) → phrase negative of that word. Broad of a single word behaves the same, but phrase keeps multi-word negatives order-sensitive and predictable.
   - A multi-word concept that is only wasteful together (`size 50`, `for kids` in an adults-only store) → phrase.
   - One bad query whose words also appear in good queries (`nike air max 90 kids` when the store sells adult Air Max 90) → exact.
   - Broad multi-word negatives only when every combination of the words in any order is wrong for the store.

### Where to put a negative

9. **Account-level list** for terms that are wrong for the whole business (jobs, free, pdf, repair service). It applies to Search and Shopping inventory in Search, PMax, Shopping, App, Smart and Local campaigns, and is limited to 1,000 terms per account.
10. **Shared negative list** for terms wrong for a group of campaigns (all Shopping campaigns, but not the Search campaign that sells spare parts). Confirm current list limits in the account.
11. **Campaign or ad group** for terms wrong only for that product set. PMax negatives are campaign-level (or via lists) and apply only to Search and Shopping inventory, not to Display or YouTube placements.
12. **Query routing is not waste.** In Standard Shopping with High/Medium/Low priorities, negatives in the higher-priority campaign are used to push generic queries down to a lower bid campaign. Do not propose removing those, and do not flag them as blocking converters; read the existing structure before proposing anything.

### Deciding what is waste

13. **Use a sample-size rule, not a fixed click count.** With account conversion rate `CVR` on comparable traffic, the probability that a term with true rate CVR gets zero conversions in `n` clicks is `(1 − CVR)^n`. Zero conversions become meaningful (about 5% chance by luck) when `n ≥ ln(0.05) ÷ ln(1 − CVR)`, roughly `3 ÷ CVR`. At CVR 2% that is about 150 clicks; at 5%, about 60. Below that, a zero-conversion term goes to "watch", however annoying it looks.
14. **Cost rule next to the click rule.** Also flag zero-conversion items whose cost exceeds 2 × target CPA (or 2 × AOV ÷ tROAS when judging by ROAS). This catches expensive terms before the click threshold is reached. Both rules are house decision rules, not Google benchmarks; the store can change the multipliers.
15. **Low-ROAS items with conversions.** Flag when the item has enough clicks by rule 13 and its ROAS is below the break-even ROAS, not merely below target. Below target but above break-even is profitable traffic; that is a bidding question, not a negative.
16. **Intent mismatch overrides volume.** A query for something the store does not sell (a size range, spare parts, rental, second-hand, a service, a competitor's exclusive model) can be negated with few clicks if the catalog confirms it is not sold. Cite the catalog evidence.
17. **Winners are protected, not just praised.** A term with ROAS ≥ 1.5 × target and at least 3 conversions (house rule) should be checked against every proposed negative and against existing negatives. Proposals: make sure the product title contains the query language (`product-feed-optimizer`), give the products a high-priority Standard Shopping campaign or dedicated PMax asset group with a search theme, or cover the exact query in a Search campaign.

## Process

1. **Clean.** Lower-case, trim, collapse spaces. Keep the original string for exact negatives. Build a second, accent-stripped key only for grouping.
2. **Coverage.** Hidden cost share per campaign = (campaign cost − reported cost) ÷ campaign cost.
3. **N-grams.** Split each term into 1-, 2- and 3-grams (on the accent-stripped key). Per n-gram sum impressions, clicks, cost, conversions, value and count distinct terms. Drop n-grams that are stop words or part of protected terms.
4. **Score** terms and n-grams: CPA = cost ÷ conversions; ROAS = value ÷ cost; required clicks = ln(0.05) ÷ ln(1 − CVR) using the campaign's CVR.
5. **Classify** each term and n-gram:
   - `negative-candidate`: zero conversions and (clicks ≥ required clicks or cost ≥ 2 × target CPA), or ROAS below break-even with clicks ≥ required clicks, or confirmed intent mismatch.
   - `watch`: fails the above only because the sample is too small.
   - `winner`: ROAS ≥ 1.5 × target and conversions ≥ 3.
   - `ok`: everything else.
6. **Draft negatives.** Prefer the n-gram over individual terms when the n-gram explains most of the waste across many terms. Assign match type (rules 8) and level (rules 9–11). Add plural, singular and accent variants explicitly.
7. **Collision test.** Apply every candidate negative, with its exact semantics, to every term with conversions > 0 and every protected term. If it blocks any: narrow it (phrase to exact, add a word), move it to a lower level, or drop it. Record what was tested.
8. **Existing negatives.** Check whether any current negative blocks a winner or a protected term; report those separately as "negatives to review", distinguishing routing negatives (rule 12).
9. **Write the report.** Totals first: cost covered by proposed negatives, hidden cost share, number of watch terms.

## Pitfalls and edge cases

- **Model numbers and sizes.** "42" or "xl" as a 1-gram mixes many intents. Negate sizes only as phrase with context (`size 50`) and only after checking the catalog.
- **Multi-language markets.** Czech stores get Slovak and English queries. Negate per language form; accents differ across languages too.
- **Competitor brand queries.** Waste only if they fail the rules; many convert when the store sells comparable products. Legal or policy preference is the owner's call.
- **"Cheap", "sale", "levné".** Often low CVR but high volume; judge by rule 15 against break-even, not by instinct.
- **Seasonal terms.** Zero conversions out of season prove nothing; compare with the same season.
- **PMax with brand exclusions.** Terms blocked by brand exclusions do not appear as waste; do not add duplicate brand negatives.
- **Totals by n-gram overlap.** A term contributes to several n-grams; never sum n-gram costs into a total.

## Rules

- Read-only. The agent produces lists; a human adds negatives in Google Ads. It never edits campaigns, lists or priorities.
- Never negate the store's own brand, brands it sells, or hero products. Brand traffic is managed with brand exclusions, by decision of the owner.
- Every negative in the output has passed the collision test; show the test result.
- No invented conversion rates or benchmarks. CVR comes from the account; multipliers are labelled house rules.
- If conversion value, target or catalog is missing, say which judgments were made without it.

## Output format

```
Shopping search terms — <account> — <date range> (last <n> days excluded from zero-conv. judgments)
Basis: <ROAS on value ex|incl VAT | CPA only (value missing)> · target <x> · break-even ROAS <y|not provided>
Coverage: reported cost <a> of <b> (<c>% hidden by low-volume filtering)

Proposed negatives (collision-tested)
| negative | match | level | terms hit | cost | clicks | conv. | reason | collision check |
|---|---|---|---|---|---|---|---|---|

Negatives to review (existing)
| negative | level | blocks | note |
|---|---|---|---|

Winners to protect
| term | campaign | cost | conv. | ROAS | proposal |
|---|---|---|---|---|---|

Watch (sample too small): <n> terms, <cost> total — top 10 listed
Not assessed: <...>
```

## Worked example

Illustrative numbers only. Store sells adult running and trail shoes; CVR on Shopping 2.5%, so required clicks = ln(0.05) ÷ ln(0.975) ≈ 118; tROAS 400%, break-even ROAS 2.7; AOV 2 000 Kč, so 2 × AOV ÷ tROAS = 1 000 Kč.

```
Shopping search terms — runshop.cz — 2026-06-01..2026-08-31 (last 7 days excluded)
Basis: ROAS on value ex VAT · target 4.00 · break-even 2.70
Coverage: reported cost 61 800 Kč of 74 400 Kč (17% hidden)

Proposed negatives (collision-tested)
| negative | match | level | terms hit | cost | clicks | conv. | reason | collision check |
|---|---|---|---|---|---|---|---|---|
| "oprava" / "opravy" | phrase | account list | 41 | 2 310 | 164 | 0 | repair intent; 164 ≥ 118 clicks | 0 converting terms blocked |
| "bazar" / "bazár" | phrase | account list | 27 | 1 140 | 95 | 0 | second-hand; cost ≥ 1 000 | 0 blocked |
| "detske" / "dětské" | phrase | Shopping list | 33 | 1 620 | 131 | 0 | no kids' range in catalog | "dětské ponožky" converts in Socks → list not attached to Socks campaign |
| [salomon speedcross 3] | exact | campaign Trail | 1 | 1 080 | 52 | 0 | discontinued model, not in feed | 0 blocked |

Winners to protect
| term | campaign | cost | conv. | ROAS | proposal |
|---|---|---|---|---|---|
| trailové boty dámské nepromokavé | Trail | 1 900 | 9 | 7.40 | add phrase to titles; exact in Search |

Watch (sample too small): 212 terms, 5 430 Kč — top 10 listed
```

## Quality checklist

- Hidden cost share stated per campaign.
- Required clicks computed from the account's own CVR and shown.
- Every negative has match type, level, cost evidence and a passed collision test.
- Singular, plural and accent variants added where the language needs them.
- No negative touches a protected term or an existing winner.
- Routing negatives in priority structures identified and left alone.
- N-gram costs never summed into totals.

## Sources

- About the search terms report: https://support.google.com/google-ads/answer/2472708
- About the search terms report in Performance Max: https://support.google.com/google-ads/answer/16327396
- About negative keywords (match types, close variants, symbols, 16-word limit): https://support.google.com/google-ads/answer/2453972
- About account-level negative keywords: https://support.google.com/google-ads/answer/11396330
- Negative keywords in Performance Max campaigns: https://support.google.com/google-ads/answer/15726455
- About brand exclusions: https://support.google.com/google-ads/answer/16669487
- Use campaign priority for Standard Shopping campaigns: https://support.google.com/google-ads/answer/6275296

## License
MIT
