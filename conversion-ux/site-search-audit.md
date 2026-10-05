---
name: site-search-audit
owner: checkoutlab
category: Conversion & UX
description: Audits on-site product search from GA4 view_search_results data or vendor logs plus a hands-on test set (zero results, no-click queries, diacritics, SKU/EAN, filters, mobile) and ranks fixes by lost revenue.
version: v1
license: MIT
updated: 2026-10-05
recommended: false
security_checked: true
url: https://ecomdly.com/skills/checkoutlab/site-search-audit
raw: https://ecomdly.com/raw/checkoutlab/site-search-audit.md
install: npx @ecomdly/cli add checkoutlab/site-search-audit
---

# Site search audit

Produces a ranked fix list for a store's on-site product search: every failing query cluster (zero results, results nobody clicks, search exits, missing synonyms, misspellings, unaccented Czech or Slovak spellings, SKU, EAN and brand lookups), the root cause behind it, and the revenue it plausibly loses, plus a hands-on test of sorting, filters and the mobile search UI. The naive audit compares the conversion rate of searchers with non-searchers, declares "search users convert 3× better" and concludes that more search usage means more revenue; that number is mostly selection, because people who search already know what they want. A list of top zero-result terms is not enough either: GA4 does not know how many results a page showed, and a query that returns 400 irrelevant results is a worse failure than one that honestly returns none. **The key insight:** classify every search session by its outcome (clicked a result, saw results but clicked nothing, saw zero results), measure each failure class against the store's own successful-search and non-search conversion rates, and report lost revenue as a range whose bounds are explicit about the selection bias, so that fixes are ranked by money rather than by query count.

## When to use

- The owner or a CRO specialist asks "is our search any good?", or before switching or re-tuning a search vendor.
- Zero-result or search-exit complaints from customer care, or GA4 shows many `view_search_results` sessions without purchases.
- After a catalog import, a platform migration or a theme change that touched the search box, results page or product data.
- Before adding a new market whose customers type in another language or without diacritics.

## When not to use

- Tracking itself is broken or `view_search_results` is missing: fix collection first with `ga4-ecommerce-event-auditor`.
- The drop is in cart or checkout steps, not in finding products: `ga4-funnel-analyst` or `checkout-friction-audit`.
- Missing colour, size, brand or material values that filters and search depend on: produce them with `product-attribute-extractor`, then re-run this audit.
- Invalid or missing EAN/GTIN values behind failed code lookups: `gtin-identifier-auditor`.
- Reading out a test of a search change you already shipped: `ab-test-readout`.
- Queries people use in Google rather than on the site: `search-console-auditor`.

## Inputs

| input | definition | typical source |
|---|---|---|
| Search events | per session: search term, timestamp, device; `view_search_results` with `search_term` (GA4 enhanced measurement) or the vendor's query log | GA4 export / BigQuery / Explorations, Algolia, Luigi's Box, Doofinder, Shoptet or platform logs |
| Result count per query | number of results the engine returned for that query | vendor log, a custom GA4 parameter, or re-running the query (step 3) |
| Clicks from results | `select_item` (with `item_list_name` identifying the search list) or vendor click log | GA4, vendor |
| Purchases per session | sessions with a purchase and their revenue, on the backend revenue basis | GA4 joined to backend orders |
| Sessions and orders, all | total sessions and orders in the same period and scope | GA4, backend |
| Catalog export | SKU, EAN, brand, title, category, key attributes, stock status | platform export, product feed |
| Market and language | country, language(s) of the storefront, whether customers type without diacritics | owner |

Optional: the vendor's synonym and redirect lists; the search configuration (fuzzy matching, field weights, stemming for Czech, Slovak or Polish); customer care tickets mentioning search; device and channel split.

If an input is missing, stop at the step that needs it and ask; never substitute an industry conversion rate or a benchmark zero-result share.

## Best practices

### Measure what GA4 can and cannot see

1. **Know how the event fires.** Enhanced measurement sends `view_search_results` when a results page loads with a search query parameter in the URL (by default `q`, `s`, `search`, `query`, `keyword`; others can be added in the advanced settings), and the term goes into `search_term`. An instant-search overlay that never changes the URL sends nothing, so check the store's parameter and search UI before trusting a low search share.
2. **GA4 has no result count.** Zero-result detection needs the vendor log, a custom event parameter carrying the result count (register it as a custom dimension), or re-running the top queries yourself. Without one of these, report "zero results: not measurable" rather than guessing from exits.
3. **Use the vendor log when it exists.** It records instant-search queries, result counts and clicks that GA4 never sees; use GA4 only to join outcomes (purchase, revenue) by session.

### Classify sessions by outcome

4. **One class per search session, by best outcome:** SUCCESS (clicked at least one result), NO_CLICK (results shown, nothing clicked), ZERO (only zero-result searches). Mutually exclusive classes make the shares add to 100% and stop one session being counted three times.
5. **Search exit = the session ends on a results page.** It is a symptom, not a class; report it beside each cluster because it separates "refined the query" from "gave up".
6. **Cluster queries before ranking them.** Normalise case, whitespace and diacritics, group misspellings and singular/plural, and map each cluster to its cause: missing synonym, misspelling, diacritics, SKU/EAN/brand lookup, out-of-assortment demand, out of stock, relevance or sort order. A cause is what the fix addresses; individual strings are too thin to rank.

### Lost revenue, with the bias stated

7. **Searchers vs non-searchers is not a treatment effect.** Searchers self-select (higher intent, returning customers typing a code), so CR_search ÷ CR_nonsearch overstates what search causes. Always print that sentence next to the comparison.
8. **Report a range, rank by the floor.** Per failing cluster with n sessions, CR_fail and backend AOV: upper bound = n × (CR_success − CR_fail) × AOV (as if the session had become a successful search); conservative = n × max(0, CR_nonsearch − CR_fail) × AOV (as if it converted like an average visitor who never searched). Rank by conservative, break ties by upper. Neither bound is a forecast; a fix proven by an A/B test replaces both.
9. **Clusters under 100 sessions are noise for CR.** Their conversion rate swings on one order; list them with raw counts under "watch" and do not price them.

### Hands-on test set

10. **Test what the logs cannot reveal.** For the top clusters and a fixed probe set, run each query on mobile and desktop and record result count, whether the first row is relevant, and the sort used. The probe set always includes: a top brand; three real SKUs and three real EANs from the catalog; a product name typed without diacritics (`zidle` for `židle`, `kosile` for `košile`); a common misspelling; a plural or declined form (Czech and Polish inflect: `boty` / `bot`); a synonym customers use but the catalog does not; an out-of-stock product; and a category word.
11. **Diacritics are a market input, not an assumption.** Measure the share of the store's own queries typed without accents and test both spellings; if unaccented queries return fewer results, the fix is accent-insensitive matching in the engine, not synonyms per word.
12. **Sorting and filters.** Check the default sort (relevance, not "newest" or margin), that filters on the results page reflect the result set, that applying a filter never yields zero without warning, and that out-of-stock items do not top the list.
13. **Mobile UX.** The search field is visible without opening a menu, the keyboard opens with the right type, suggestions are tappable, the result count and active filters are visible, and the back button returns to the same results.

## Process

1. **Confirm collection.** Record whether search is measured by GA4 enhanced measurement, a custom event or vendor logs; the query parameter; whether instant search fires anything. Stop if no source covers the search UI the store actually uses.
2. **Pull 28–90 days** of search sessions with term, device, outcome class and purchase revenue, plus all sessions and orders for the same scope. Exclude internal traffic and test orders.
3. **Get result counts** from the vendor log or a custom parameter; otherwise re-run the top 200 query clusters and record counts, labelled "re-run on <date>, may differ from what users saw".
4. **Classify sessions** (rule 4) and compute CR_success, CR_no-click, CR_zero, CR_search and CR_nonsearch, all on backend revenue.
5. **Cluster and code causes** (rule 6), with three example queries per cluster.
6. **Price each cluster** with ≥ 100 sessions (rule 8); put smaller ones on the watch list.
7. **Run the probe set** (rules 10–13) on mobile and desktop; record each failure with a screenshot reference.
8. **Write fixes** per cluster: synonym or redirect entry, accent-insensitive matching, SKU/EAN field indexed, attribute fill via `product-attribute-extractor`, sort rule, UI change; name the owner (merchandiser, developer, search vendor).
9. **Rank** by conservative lost revenue, then run the quality checklist.

## Pitfalls and edge cases

- **Instant search without URL change.** GA4 shows almost no searches while the store has many; the low share is a tracking gap, not low usage.
- **Pagination and filter clicks re-fire the event.** One query seen on page 2 or after filtering is not a second search; deduplicate by session, term and short time window.
- **Refinement chains.** "zidle" → zero → "židle" → click: the session is SUCCESS, but the first query is still a diacritics failure worth fixing; count failing queries per cluster as well as sessions.
- **Out-of-assortment demand.** Zero results for products the store does not sell are a merchandising signal, not a search bug; report them separately and do not price them as lost search revenue.
- **Redirect rules hide searches.** A query redirected straight to a category may not produce `view_search_results`; get the redirect list from the vendor.
- **Consent mode and modelled data.** Unconsented sessions may be missing from session-level exports; state the consented share and do not mix modelled totals with session-level counts.
- **Personal data in queries.** People type e-mails, phone numbers and order numbers into search; mask them in the report.
- **Multi-market stores.** Compute per storefront language; a Polish query in a Czech store is not a misspelling.

## Rules

- Read-only: the agent proposes synonyms, settings and UI changes; it does not edit the search configuration, catalog, theme or tracking on its own.
- Every conversion rate and AOV comes from the store's own data for the stated period; no benchmarks.
- Each lost-revenue figure is shown as a range with the selection-bias note; never present the searcher vs non-searcher ratio as the effect of search.
- Clusters under 100 sessions are not priced.
- Queries in the report are anonymised.

## Output format

```
SITE SEARCH AUDIT · <store> · <market/language> · <from>–<to> · prepared <date>
Collection: <GA4 enhanced measurement (param <q>) | custom event | vendor log> · instant search tracked: <yes/no>
Sessions <n> · with search <n> (<x>%) · orders <n> · AOV (backend, <VAT basis>) <amount>
Outcome classes: SUCCESS <n> (<CR>%) · NO_CLICK <n> (<CR>%) · ZERO <n> (<CR>%) · non-search <n> (<CR>%)
Bias note: searchers self-select; CR_search ÷ CR_nonsearch = <x> is not the effect of search.

Ranked fixes (lost revenue per period, conservative – upper):
1. <cluster> · cause <code> · <n> sessions, CR <x>% · <amount> – <amount>
   examples: "<q>", "<q>", "<q>" · fix: <action> · owner: <team>
Probe set: <query> → <result count>, first result <relevant/irrelevant>, sort <x> · mobile <issue>
Watch list (< 100 sessions): <cluster> <sessions>/<orders>
Not priced: out-of-assortment demand <clusters>
Data gaps: <list>
```

## Worked example

Illustrative numbers only. Czech furniture store, 28 days, AOV EUR 62.00 ex VAT from backend.

```
Sessions 120 000 · with search 14 400 (12.0%) · orders 2 966
Outcome classes: SUCCESS 10 000 (760 orders, 7.6%) · NO_CLICK 2 800 (70, 2.5%) · ZERO 1 600 (24, 1.5%) · non-search 105 600 (2 112, 2.0%)
Bias note: CR_search 854 ÷ 14 400 = 5.93% vs 2.0% non-search (3.0×) — selection, not effect.

1. "postel 180x200" and similar size queries · cause SORT (newest first, accessories on top) · NO_CLICK, 900 sessions, 9 orders, 1.0%
   conservative 900 × (2.0% − 1.0%) × 62 = 558.00 · upper 900 × (7.6% − 1.0%) × 62 = 3 682.80
2. Unaccented queries ("zidle", "kreslo", "konferencni stolek") · cause DIACRITICS · ZERO, 600 sessions, 6 orders, 1.0%
   conservative 600 × 1.0% × 62 = 372.00 · upper 600 × 6.6% × 62 = 2 455.20
3. EAN and supplier code lookups · cause SKU_EAN_NOT_INDEXED · ZERO, 250 sessions, 2 orders, 0.8%
   conservative 250 × 1.2% × 62 = 186.00 · upper 250 × 6.8% × 62 = 1 054.00
```

Check: classes 10 000 + 2 800 + 1 600 = 14 400 search sessions; orders 760 + 70 + 24 = 854, plus 2 112 non-search = 2 966. Clusters 2 and 3 (850 sessions, 8 orders) fit inside ZERO (1 600, 24); cluster 1 (900, 9) fits inside NO_CLICK (2 800, 70). Rank 1 has the highest conservative figure, so sort order is fixed before synonyms.

## Quality checklist

- Collection method stated, including whether instant search and redirects are measured.
- Zero results come from a real result count, or are marked not measurable.
- Session classes are mutually exclusive and add up to all search sessions.
- Every price is a range on backend revenue with the selection-bias sentence printed.
- No cluster under 100 sessions is priced; out-of-assortment demand is separate.
- Probe set includes SKU, EAN, brand, unaccented, misspelled, inflected and synonym queries on mobile and desktop.
- Each fix has a cause code and an owner; queries are anonymised.

## Sources

- Google Analytics Help, Enhanced measurement events (site search, default query parameters q, s, search, query, keyword): https://support.google.com/analytics/answer/9216061
- Google Analytics Help, Automatically collected events (view_search_results, search_term, unique_search_term): https://support.google.com/analytics/answer/9234069
- Google Analytics, Data API schema (searchTerm dimension populated by search_term): https://developers.google.com/analytics/devguides/reporting/data/v1/api-schema
- Google Analytics, Recommended events reference (search, select_item, item_list_name): https://developers.google.com/analytics/devguides/collection/ga4/reference/events

## License
MIT
