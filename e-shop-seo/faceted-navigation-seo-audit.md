---
name: faceted-navigation-seo-audit
owner: rankcraft
category: E-shop SEO
description: Decides facet by facet which filter URLs on an online store should be indexed, consolidated, noindexed or blocked from crawling, using crawl, log and Search Console data, and lists crawl traps with a safe rollout order.
version: v3
license: MIT
updated: 2026-09-29
recommended: false
security_checked: true
url: https://ecomdly.com/skills/rankcraft/faceted-navigation-seo-audit
raw: https://ecomdly.com/raw/rankcraft/faceted-navigation-seo-audit.md
install: npx @ecomdly/cli add rankcraft/faceted-navigation-seo-audit
---

# Faceted navigation SEO audit

Produces a facet-by-facet decision table (index, keep crawlable but not indexed, consolidate, or stop crawling) plus the crawl traps to fix, backed by crawl, log and demand data. Filters help shoppers and flood crawlers: a category with eight filters can expose millions of URL combinations. The naive fix is "noindex all filters" or "canonical everything to the category", which throws away the handful of filter pages that match real searches ("salomon trail shoes", "black waterproof jacket") and still leaves Googlebot crawling millions of useless URLs, because noindex and canonical do not stop crawling. The right answer is a per-facet decision, applied in the right order, since the controls interact: a URL blocked in robots.txt can no longer show Google its noindex or canonical.

## When to use
- A crawl or the Page indexing report shows far more URLs than the store has products and categories.
- Search Console shows large or growing counts of "Crawled - currently not indexed", "Discovered - currently not indexed", "Duplicate without user-selected canonical" or "Duplicate, Google chose different canonical than user" on filter URLs.
- Server logs show Googlebot spending most requests on parameter URLs while new products take long to get crawled.
- Before a platform migration or filter redesign, to set the URL rules up front.
- When the store wants landing pages for brand, colour or type within a category and asks which should exist.

## When not to use
- Product variant URLs (`?size=42` on a product page): that is variant canonicalisation, covered in `product-schema-validator` and the store's variant setup.
- Small catalogs where the crawl shows no parameter explosion and indexing reports are clean: nothing to fix.
- Writing copy for the facet pages that should be indexed: hand off to `category-page-copy-writer` after this audit.
- Diagnosing ranking drops in general: start with `search-console-auditor`.

## Inputs

Required:
- **Facet inventory**: every filter and sort control per category template, its URL pattern (query parameter name, path segment, or fragment), whether combinations are possible, and whether the links are plain `<a href>` links or JavaScript-only.
- **Crawl export** (Screaming Frog, Sitebulb or similar, crawled with robots.txt respected and a second pass ignoring it if possible): URL, status code, indexability, meta robots / X-Robots-Tag, canonical target, inlinks, number of products rendered.
- **robots.txt** current version.
- **Search Console Performance export**, Pages dimension, last 3 months at least, with clicks and impressions per URL; and queries for the category pages.
- **Search Console Page indexing report** examples for the "not indexed" reasons above (the report shows sample URLs; note that it is a sample).

Optional but decisive when available:
- **Server logs** filtered to verified Googlebot (reverse DNS), 30+ days: hits per URL pattern and status code. Search Console's Crawl stats report gives totals and sample URLs if logs are unavailable.
- **Keyword demand** for category + facet value combinations from a keyword tool, with the market and date.
- **Product counts** per category and facet value from the platform.

If demand data is missing, classify the facet `demand unknown` and do not make it indexable or deindex it; recommend the data pull. If logs are missing, say that crawl waste is estimated from the crawl and indexing report, not measured.

## Best practices

### What the controls actually do
1. **robots.txt stops crawling, not indexing.** Google may still index a disallowed URL without its content if it is linked, and the page cannot show Google a noindex or canonical because Google never fetches it. Use it for facet URLs that should never be crawled and are not already indexed or earning clicks.
2. **noindex removes from the index but not from crawling.** Google must crawl the page to see the tag, and keeps crawling it. It is the right tool to get already-indexed facet URLs out, and the wrong tool to save crawl. Google advises against using noindex to choose a canonical within a site.
3. **rel=canonical is a strong signal, not a directive.** Google may pick another canonical. For facet URLs Google says canonical to the unfiltered version "may, over time, decrease the crawl volume" but is less effective long term than robots.txt or fragments. The canonical target must be indexable (200, no noindex, not redirected, not blocked).
4. **Fragments (`#color=black`) are ignored for crawling and indexing.** Filters implemented as fragments or client-side state with no crawlable URL have no crawl cost. They also cannot rank; do not use them for facets that deserve a landing page.
5. **rel=nofollow on filter links** only works if every link to that URL carries it, including sitemaps and external links. Treat it as a supplement, never the primary control.
6. **Do not combine conflicting signals.** A URL that is disallowed in robots.txt and carries noindex will stay indexed if it was indexed; a noindexed URL with a canonical elsewhere sends mixed signals. Pick one primary control per URL class.
7. **The URL Parameters tool in Search Console no longer exists.** Anything that relied on it must be replaced by the controls above.

### Deciding per facet
8. **Index a facet value only when all hold**: (a) there is measurable search demand for category + value in the store's market; (b) the filtered page has enough products to be a useful result (set the threshold with the owner; a page with one or two products competes poorly with the product page itself); (c) the value is stable (brands, types, materials, gender, main colours), not volatile (price range, in stock, discount); (d) the store will give it a unique title, H1 and intro. Brand and product type within a category most often pass; size rarely does except for apparel and footwear searched with size.
9. **Indexable facets get a clean, stable URL**: preferably a path (`/trail-shoes/salomon/`) or a single fixed parameter, a self-referencing canonical, inclusion in the XML sitemap, and a plain `<a href>` link from the parent category with descriptive anchor text. Google recommends the same URL in internal links, sitemaps and canonical.
10. **Single-facet pages only, as a default.** Two-facet combinations get indexed only when demand is proven for the exact pair (e.g. brand + gender). Every added dimension multiplies URLs.
11. **Sort order, view mode, items per page, session and tracking parameters are never indexable** and should not be crawled. Canonicalising them to the parent is acceptable where they already exist and are linked; stopping links to them (or moving them to fragments or POST) is better.
12. **Everything else** (price sliders, ratings, availability, multi-select combinations): stop crawling. If these URLs are already indexed or earning clicks, first let noindex or canonical take effect, confirm they have dropped out, then add the robots.txt rule.

### URL hygiene Google asks for
13. Use `&` as the parameter separator and `key=value` pairs; commas, semicolons and brackets are hard for crawlers to parse as separators.
14. Keep a fixed filter order and no duplicate filters, in paths and parameters. `?color=black&size=43` and `?size=43&color=black` must resolve to one URL (redirect or never generate the second).
15. **Return 404 for filter combinations with no results**, duplicate or nonsensical filters, and non-existent pagination pages, at the URL requested, not by redirecting to a generic error page. A 200 "0 products" page is a soft-404 and a crawl sink.
16. Do not link internally to temporary parameters (session IDs, tracking codes, `location=nearby`).
17. **Pagination is not a facet.** Each paginated page keeps its own URL and self-canonical; Google says not to canonicalise page 2+ to page 1. Facet links rendered on every paginated page multiply combinations; check whether they need to be crawlable there.

### Measuring before deciding
18. **Never deindex a URL that earns clicks without quoting its clicks.** Pull the last 3 months of clicks for every facet URL before proposing noindex, canonical or robots rules. Search Console attributes clicks to the canonical Google chose, so check URL Inspection for the Google-selected canonical on high-value examples.
19. **Crawl share, from logs**: `crawl share of pattern = Googlebot hits to pattern ÷ all Googlebot hits`. Compare it with `click share = clicks to pattern ÷ all organic clicks`. A pattern with high crawl share and near-zero click share is the priority to block.
20. **Index bloat ratio**: `indexable URLs found in crawl ÷ (products + categories + intended facet pages)`. A ratio well above 1 locates the problem; the pattern breakdown shows where. Do not present the ratio as a benchmark; it is a diagnostic.

## Process
1. **Map patterns.** Group every crawled URL into a pattern: category, product, pagination, each facet parameter or path segment, sort, view, tracking, combinations (count parameters per URL). Output counts per pattern.
2. **Attach data per pattern.** Crawl count, indexable count, Googlebot hits (logs), clicks and impressions (Search Console), sample of Page indexing reasons.
3. **Detect traps.** Parameter order duplicates, duplicate parameters, empty results returning 200, infinite combinations (count distinct value sets), facet links on paginated pages, canonicals pointing to non-indexable targets, sort/view parameters linked from every listing, calendar or price-slider values generating unbounded URLs.
4. **Classify each facet** using rules 8–12:
   - demand proven + products sufficient + stable → `INDEX` (single facet, clean URL)
   - demand unknown → `HOLD - demand unknown` (no change until data exists)
   - no demand, URLs indexed or with clicks → `NOINDEX then BLOCK` (two phases)
   - no demand, not indexed, no clicks → `BLOCK` (robots.txt) or `FRAGMENT` (if the platform supports it)
   - sort/view/per-page → `CONSOLIDATE` (canonical to parent) and stop linking; `BLOCK` once dropped
5. **Check clicks at risk.** For every URL moving out of the index, list clicks in the last 3 months. If a single URL has meaningful clicks, move it to `INDEX` review instead of deindexing, and say why.
6. **Write the robots.txt proposal** with Google-supported wildcards (`*`, `$`), scoped to the parameter, e.g. `Disallow: /*?*sort=`. Test each rule against sample allowed URLs (category, indexable facets, products) so nothing needed is blocked.
7. **Sequence the rollout**: (1) fix traps and 404s; (2) build indexable facet pages; (3) apply noindex/canonical; (4) wait until the Page indexing report and a `site:` spot check show the URLs dropping out (weeks, not days; state that timing varies); (5) add robots.txt blocks; (6) monitor Crawl stats and the indexing report.
8. **Define success metrics**: Googlebot hits to blocked patterns, share of crawl on products and categories, indexed facet pages with impressions, clicks to the facet pages kept.

## Pitfalls and edge cases
- **Blocking before deindexing** leaves URLs stuck in the index as "Indexed, though blocked by robots.txt".
- **Canonical to a noindexed or redirected parent**: the signals cancel; Google may choose its own canonical.
- **Filter pages with product-level clicks**: a "brand" filter URL may rank for a model name because the product page is weak. Fix the product page before removing the filter URL.
- **Platform defaults**: some platforms add noindex or canonical to filter pages already, or generate path-style facet URLs that look like categories. Verify on the rendered page, not the admin setting.
- **JavaScript-only filters with pushState URLs**: users see a URL, crawlers may not find it; indexable facets need real `<a href>` links.
- **Multi-language or multi-currency stores**: facet URL rules and robots.txt must cover each locale path.
- **Out-of-stock collapse**: a facet page with sufficient products today can drop to zero; decide whether it then returns 404 or stays with a noindex, and document it.
- **Crawl budget is mostly a large-site concern.** For a catalog of a few thousand URLs, the problem is duplicate and thin pages in the index rather than crawl capacity; weight the recommendations accordingly.

## Rules
- Read-only. Propose robots.txt, meta robots, canonical and linking changes; never edit them. A robots.txt change can remove a site from crawling, so every rule needs owner and developer sign-off and a test against sample URLs.
- Demand comes only from data provided; no demand estimates from memory.
- Never recommend deindexing or blocking a URL with clicks without naming the clicks.
- State whether crawl waste is measured (logs) or estimated (crawl, indexing report samples).
- Name the date range of every Search Console and log figure.

## Output format
```text
Faceted navigation audit - <store> - data: crawl <date>, logs <range or none>, GSC <range>
URL inventory: <n> crawled | products <n> | categories <n> | index bloat ratio <x>

Pattern table
pattern | example | crawled | indexable | Googlebot hits (share) | clicks 3m (share) | decision
<pattern> | <url> | <n> | <n> | <n> (<%>) | <n> (<%>) | INDEX / HOLD / NOINDEX then BLOCK / BLOCK / CONSOLIDATE / FRAGMENT

Facet decisions
- <facet>: <decision> - demand <evidence or "unknown">, products <n range>, reason <one line>

Traps
- <trap>: <n> URLs, e.g. <url> - fix: <what>

Clicks at risk (URLs leaving the index with clicks)
- <url>: <clicks> clicks / <impr> impr. (3m) - proposed: <decision>

Proposed robots.txt additions (apply only after phase 4)
<rules>

Rollout sequence
1. ...

Monitoring
- <metric>: baseline <value>, check <when>

Open questions / data needed
- ...
```

## Worked example
Example data, not a real store. Outdoor shoe shop, 1 850 products, 64 categories. Crawl 2026-09-20; Googlebot logs 2026-08-20 to 2026-09-19 (42 000 hits); GSC 2026-06-21 to 2026-09-19 (18 400 organic clicks).

```text
Faceted navigation audit - example-outdoor.cz - data: crawl 2026-09-20, logs 2026-08-20..09-19, GSC 2026-06-21..09-19
URL inventory: 96 300 crawled | products 1 850 | categories 64 | index bloat ratio 4.1 (7 900 indexable / 1 914 intended)

Pattern table
pattern | example | crawled | indexable | Googlebot hits (share) | clicks 3m (share) | decision
brand path | /trail-shoes/salomon/ | 210 | 210 | 1 300 (3%) | 1 150 (6%) | INDEX (top values)
?color= | /trail-shoes?color=black | 540 | 540 | 2 100 (5%) | 240 (1%) | HOLD - demand unknown
?color=&size= | /trail-shoes?color=black&size=43 | 61 000 | 5 900 | 21 800 (52%) | 12 (0.1%) | NOINDEX then BLOCK
?sort= | /trail-shoes?sort=price_asc | 8 300 | 0 (canonical) | 6 700 (16%) | 0 | CONSOLIDATE, then BLOCK

Facet decisions
- brand: INDEX for 38 brand values with >= 8 products and keyword demand; the remaining 172 values stay linked but noindex.
- color: HOLD - no keyword data provided; request demand for "<category> <color>" pairs.

Traps
- parameter order: 6 100 URLs, e.g. ?size=43&color=black vs ?color=black&size=43 - fix: normalise order server-side, 301 the variant order.
- empty results returning 200: 2 400 URLs - fix: return 404.

Clicks at risk
- /trail-shoes?color=black&size=43: 9 clicks / 610 impr. (3m) - proposed: NOINDEX; clicks expected to move to /trail-shoes/ (owner to confirm acceptable)

Proposed robots.txt additions (apply only after phase 4)
User-agent: *
Disallow: /*?*sort=
Disallow: /*?*color=*&size=
Disallow: /*?*size=*&color=
```

52% of Googlebot hits go to color+size combinations that earned 0.1% of clicks: that pattern is the priority.

## Quality checklist
- Every URL pattern has counts from the crawl and a decision.
- Every deindex or block decision lists clicks at risk with the date range.
- No robots.txt rule is proposed for URLs that are currently indexed without a noindex phase first.
- Each proposed robots.txt rule was tested against sample category, facet and product URLs that must stay crawlable.
- Canonical targets checked: 200, indexable, not redirected.
- Demand evidence cited for every INDEX decision; `demand unknown` used otherwise.
- Measured vs estimated crawl waste stated.

## Sources
- https://developers.google.com/search/docs/crawling-indexing/crawling-managing-faceted-navigation
- https://developers.google.com/search/docs/crawling-indexing/consolidate-duplicate-urls
- https://developers.google.com/search/docs/crawling-indexing/robots/intro
- https://developers.google.com/search/docs/specialty/ecommerce/designing-a-url-structure-for-ecommerce-sites
- https://developers.google.com/search/docs/specialty/ecommerce/pagination-and-incremental-page-loading
- https://support.google.com/webmasters/answer/7440203

## License
MIT
