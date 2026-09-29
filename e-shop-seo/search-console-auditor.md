---
name: search-console-auditor
owner: rankcraft
category: E-shop SEO
description: Turns a Search Console performance export into three ranked lists for a store: pages losing clicks with the cause split into demand, ranking and CTR, under-clicked queries against the site's own CTR curve, and URLs competing for one query.
version: v3
license: MIT
updated: 2026-09-29
recommended: false
security_checked: true
url: https://ecomdly.com/skills/rankcraft/search-console-auditor
raw: https://ecomdly.com/raw/rankcraft/search-console-auditor.md
install: npx @ecomdly/cli add rankcraft/search-console-auditor
---

# Search Console auditor

Produces three short, ranked lists from Google Search Console performance data: pages losing clicks (with the cause decomposed into demand, visibility and CTR), queries where the page ranks but under-earns clicks, and queries where two of the store's URLs compete. Search Console has the answers but does not sort them, and the naive readings mislead: summing table rows to "total clicks" ignores anonymized queries, a 0.3 change in average position is noise, and comparing September to August confuses seasonality with a problem. This skill applies the data's actual definitions and limits so that every item on the lists is worth an hour of someone's time.

## When to use
- Monthly review: last 28 days vs the previous 28 days, plus the same 28 days a year earlier when available.
- After a migration, template change, redesign, or platform switch (compare equal windows before and after the release date).
- When the store owner reports "organic traffic dropped" and needs to know where and why.

## When not to use
- Indexing and crawl problems (pages missing from Google, facet URL bloat): use the Page indexing report and `faceted-navigation-seo-audit`.
- Structured-data or rich-result errors: `product-schema-validator`.
- Revenue or conversion questions: Search Console has no revenue; use `ga4-funnel-analyst` or `revenue-discrepancy-reconciler`.
- Writing new content or new category copy: this skill finds problems on existing pages; hand CTR fixes on category pages to `category-page-copy-writer`.

## Inputs

Required (Search Console > Performance > Search results, search type Web):
- **Page × date** for both periods: clicks, impressions, CTR, position.
- **Query × page** for both periods: clicks, impressions, CTR, position. The UI exports at most 1,000 rows per table, which is not enough for a store; pull through the Search Analytics API (up to 25,000 rows per request, paginate with `startRow`; Google documents an upper limit of 50,000 rows per day per site per search type) or the Looker Studio connector / bulk export if the store has it.
- **Query × page × date** (or week) for the cannibalization check.
- **Brand term list**: brand name, misspellings, domain name, own product-line names.

Optional:
- Country and device splits if the store sells in several markets (CZ + SK is common).
- Release log: dates of template changes, migrations, price campaigns, stock-outs.
- Titles and meta descriptions per URL (crawl export) to quote in list 2.
- GA4 landing-page revenue to weight the lists by value.

If the query × page data is capped (row counts exactly at an export limit), say so: the lists cover the top rows only. If there is no brand list, derive a draft from queries containing the domain name and ask the owner to confirm it before excluding anything.

## Best practices

### Read the data by its definitions
1. **CTR = clicks ÷ impressions**; recompute it from summed clicks and impressions, never average CTRs across rows.
2. **Average position is the topmost position of the site's result, averaged over impressions.** A link that did not receive an impression has no position. When impressions grow from new, lower-ranking queries, average position worsens without any ranking loss. Therefore interpret position only at query × page level, and only together with impressions.
3. **Aggregation differs by dimension.** Queries, countries, devices and dates are aggregated by property (two results in one SERP = one impression); pages and search appearance are aggregated by page. Page totals and query totals do not reconcile; do not force them to.
4. **Anonymized queries are omitted from tables but included in chart totals** (unless a query filter is applied). Rows never sum to the chart total. The gap is normal; report the itemized share (`sum of query rows ÷ chart total`) so the reader knows how much of traffic the query lists cover.
5. **Data is attributed to the canonical URL Google selected.** If a page's clicks moved to another URL, check whether Google changed the canonical (URL Inspection) before calling it a loss.
6. **The newest days are preliminary.** Exclude the last 2–3 days from any comparison, or pull with finalized data only (`dataState` final in the API).
7. **Position is recorded only for Google Search** and search type matters: web, image, video and news are reported separately and never combined. Audit Web unless the store gets meaningful image traffic, then report Image separately.

### Compare like with like
8. **Use whole weeks.** 28 days = 4 full weeks, so weekday mix is equal. Align both windows on the same weekday.
9. **Separate season from change.** For each losing page also compute the year-over-year change for the same window if 13+ months of data exist. If the page is down vs the previous period but flat or up YoY, it is seasonal, not a problem. Search Console keeps a limited history; if YoY is needed later, the store must export and archive data now.
10. **Decompose every click change** with `clicks = impressions × CTR`:
    - impression effect = (I₂ − I₁) × CTR₁
    - CTR effect = I₂ × (CTR₂ − CTR₁)
    - these two sum exactly to clicks₂ − clicks₁.
    Then split the CTR effect by looking at position at query level: CTR fell with position stable → the snippet or the SERP layout changed; CTR fell with position worse → ranking loss.
11. **Diagnose impression drops** in order: (a) the query's demand fell (same queries, fewer impressions, position stable; check YoY); (b) the page lost queries (fewer ranking queries, e.g. after a content or template change); (c) the page lost indexing or canonical status (clicks shifted to another URL or vanished).

### Thresholds that avoid noise
12. **Materiality before percentages.** A page going from 8 to 3 clicks is −62% and irrelevant. Defaults: losing pages need ≥ 50 clicks lost in absolute terms and ≥ 20% relative; free-click queries need ≥ 1,000 impressions in the period; cannibalization needs both URLs with ≥ 10% of the query's impressions. Scale these to the store: for a store with 2,000 organic clicks a month, halve them; always print the thresholds used.
13. **Position noise**: do not call a position change below 1.0 a drop at page level, or below 0.5 at query × page level with fewer than a few hundred impressions. Report the change with impressions, and let the decomposition decide.
14. **Build the CTR reference from the store's own data, not a published curve.** For non-brand queries on Web, group query × page rows by rounded position band (1, 2, 3, 4–5, 6–8, 9–10) and compute `band CTR = Σclicks ÷ Σimpressions`. A query is under-performing when its CTR < 0.5 × band CTR. Published industry CTR curves vary by study, vertical and SERP layout; do not use them.
15. **Exclude brand queries from lists 2 and 3.** Brand queries have high CTR and pull the band CTR up, making every non-brand query look bad. Report brand clicks as one line (period vs period) so a brand demand drop is visible. If the property offers a built-in branded-query filter, use it and state that; otherwise use a regex on the brand list (Search Console custom filters use RE2 syntax).
16. **SERP features change CTR without any fault of the page.** Shopping units, AI Overviews and other features can push organic results down or satisfy the query. When CTR fell at stable position, check the live SERP (in the store's country and language) and record what changed before blaming the title.

### Cannibalization, correctly
17. **Two URLs showing for one query is not automatically a problem.** Category + product for a head term can both be legitimate (the SERP shows both). It is a problem when the URLs alternate (the ranking URL swaps week to week) and the combined clicks are lower than one stable URL would earn. Test: count weeks in which the top URL for the query changed; ≥ 2 swaps in 4 weeks with neither URL holding ≥ 70% of impressions qualifies.
18. **Pick the winner by intent first, CTR second.** Broad, plural or category-type queries ("merino socks") → the category page; model-specific queries ("hiker merino sock 2") → the product. If intent is ambiguous, the URL with higher CTR at comparable position wins. The fix is internal linking, title/H1 differentiation and, only if the pages are true duplicates, consolidation; never deletion as a first step.

## Process
1. **Pull and validate.** Confirm date windows, search type, finalized data only, row counts vs export limits, and the itemized share of clicks (rule 4).
2. **Tag brand queries** with the confirmed brand list; report brand clicks per period as one line.
3. **List 1 - Losing pages.** For each page: clicks₁, clicks₂, Δ, Δ%. Keep pages meeting rule 12. Decompose (rule 10), check YoY (rule 9), check canonical shift (rule 5), and attach the top 3 queries by lost clicks with their position and CTR change. Assign one cause: `demand`, `ranking`, `snippet/SERP`, `indexing/canonical`, `seasonal`, or `unclear - needs X`.
4. **List 2 - Free clicks.** Non-brand query × page rows with ≥ threshold impressions, position ≤ 8 (current period), CTR < 0.5 × band CTR. Quote the current title (and meta description if available), and state the mismatch with the query in one line. Do not rewrite it here unless asked; propose the direction.
5. **List 3 - Cannibalization.** Non-brand queries where ≥ 2 URLs each hold ≥ 10% of impressions; add weekly top-URL swaps (rule 17). Name the winner by rule 18 and the concrete linking/title fix.
6. **Rank each list** by clicks at stake: list 1 by clicks lost; list 2 by `impressions × (band CTR − current CTR)` (the clicks the page would get at band CTR); list 3 by combined impressions × band CTR.
7. **Cap each list at 10 items.** Mention how many more qualified.
8. **Hand-offs.** Category titles → `category-page-copy-writer`; product pages → `product-description-writer` or `product-page-cro-review`; suspected indexing/canonical problems → `faceted-navigation-seo-audit` or a URL Inspection check by the owner.

## Pitfalls and edge cases
- **Migration windows**: after a URL change, old URLs lose and new URLs gain; join old→new via the redirect map before listing losers, or every migrated page will appear on list 1.
- **Stock-outs and delisted products** lose clicks for business reasons; check stock status before assigning `ranking`.
- **Country mix**: a store selling in CZ and SK can lose SK traffic while CZ grows; filter by country when the property serves several markets.
- **Parameter URLs** in the page list (facets, tracking) signal a canonical or indexing issue; do not treat them as normal pages.
- **Image search** clicks are attributed to the host page; a product page can gain or lose image traffic independently of Web.
- **Small numbers**: CTR at 40 impressions is not reliable; the impression threshold exists for this reason.
- **Algorithm updates**: if many unrelated pages drop on the same date, check Google's Search Status Dashboard for a ranking update before diagnosing pages one by one.

## Rules
- Read-only. Never change titles, content, redirects or settings; propose them.
- Never compute totals by summing query rows; use chart totals and say which is which.
- Report the thresholds, date windows, search type and data state used.
- Never invent causes. If the data does not show why, write `unclear` and name the check that would tell.
- No new-content ideas; problems on existing pages only.
- Brand exclusion list must be confirmed by the owner before it filters anything.

## Output format
```text
Search Console audit - <property> - Web - <window A> vs <window B> (YoY: <window or n/a>) - finalized data
Thresholds: losing >= <n> clicks and >= <p>%; free clicks >= <n> impr., pos <= 8, CTR < 0.5 x band; cannibalization >= 10% impr. each, >= 2 swaps/4 wk
Totals (chart): clicks <A> -> <B> (<Δ%>), impressions <A> -> <B>; itemized query share <p>%
Brand clicks: <A> -> <B> (<Δ%>)

## Losing pages (<n> shown of <m>)
- <url> - clicks <A> -> <B> (<Δ>, <Δ%>). Impression effect <n>, CTR effect <n>. YoY <Δ% or n/a>. Cause: <cause>. Evidence: <queries with pos/CTR change>. Next: <check or fix>.

## Free clicks (<n> shown of <m>)
- "<query>" -> <url> - <impr> impr., pos <x>, CTR <c>% (band <b>%), clicks at stake ~<n>. Title: "<current title>" - <mismatch>.

## Cannibalization (<n> shown of <m>)
- "<query>" - <url A> (<type>, <share>% impr.) vs <url B> (<type>, <share>%); top URL swapped <k>x in 4 weeks. Winner: <url> because <intent/CTR>. Fix: <linking/title change>.

Data caveats: <row caps, anonymized share, excluded days>
```

## Worked example
Example data, not a real store. Outdoor store, Web, 2026-08-03..08-30 (A) vs 2026-08-31..09-27 (B), finalized data. Band CTR (non-brand, store's own data): position 4–5 = 4.8%, 6–8 = 2.9%.

```text
Search Console audit - example-outdoor.cz - Web - 2026-08-03..08-30 vs 2026-08-31..09-27 (YoY: 2025-09-01..09-28) - finalized data
Thresholds: losing >= 50 clicks and >= 20%; free clicks >= 1 000 impr., pos <= 8, CTR < 0.5 x band; cannibalization >= 10% impr. each, >= 2 swaps/4 wk
Totals (chart): clicks 21 400 -> 19 900 (-7%), impressions 1.02M -> 0.99M; itemized query share 71%
Brand clicks: 6 100 -> 6 050 (-1%)

## Losing pages (1 shown of 4)
- /shoes/trail-runners - clicks 1 240 -> 810 (-430, -35%). Impressions 31 000 -> 30 400: impression effect -24, CTR effect -406. YoY -31%, so not seasonal. Cause: ranking - "trail running shoes" pos 4.1 -> 7.8 on stable impressions. Next: compare top-5 results on the live SERP; page unchanged since March.

## Free clicks (1 shown of 6)
- "waterproof trail running shoes" -> /shoes/trail-runners - 14 200 impr., pos 5.2, CTR 1.1% (band 4.8%), clicks at stake ~525. Title: "Trail Runners | Shop" - omits "waterproof", the query's deciding attribute.

## Cannibalization (1 shown of 2)
- "merino socks" - /socks/merino (category, 58% impr.) vs /socks/merino-hiker-2 (product, 42%); top URL swapped 3x in 4 weeks. Winner: category, broad plural query. Fix: link the product from the category's top grid, link back to the category from the product, keep "Merino Hiker 2" as the product title lead.

Data caveats: query x page pulled via API, 38 200 rows, below limits; last 3 days excluded.
```

Check of the decomposition: CTR A = 1 240 ÷ 31 000 = 4.0%; impression effect = (30 400 − 31 000) × 4.0% = −24; CTR B = 810 ÷ 30 400 = 2.66%; CTR effect = 30 400 × (2.66% − 4.0%) = −406; total −430. Clicks at stake for the free-click query: 14 200 × (4.8% − 1.1%) ≈ 525.

## Quality checklist
- Windows are whole weeks, aligned on weekday, preliminary days excluded.
- Thresholds, search type and data state are printed.
- Every losing page has a decomposition that sums to the click change, and a single named cause or `unclear`.
- Band CTR comes from the store's own non-brand data; no external CTR curves.
- Brand queries excluded from lists 2 and 3 using an owner-confirmed list.
- Cannibalization items show swap counts, not only co-occurrence.
- Row caps and the itemized share are disclosed.
- No list exceeds 10 items; the count of further qualifying items is given.

## Sources
- https://support.google.com/webmasters/answer/7576553
- https://support.google.com/webmasters/answer/7042828
- https://developers.google.com/search/blog/2022/10/performance-data-deep-dive
- https://developers.google.com/webmaster-tools/v1/searchanalytics/query
- https://support.google.com/webmasters/answer/7440203
- https://status.search.google.com/

## License
MIT
