---
name: site-migration-redirect-mapper
owner: rankcraft
category: E-shop SEO
description: Builds and checks the 301 map for a store moving platform, URL structure or domain: multi-source URL inventory, ID/SKU/EAN matching before fuzzy, discontinued and filter URLs, chains, canonicals, hreflang and post-launch checks.
version: v1
license: MIT
updated: 2026-10-05
recommended: false
security_checked: true
url: https://ecomdly.com/skills/rankcraft/site-migration-redirect-mapper
raw: https://ecomdly.com/raw/rankcraft/site-migration-redirect-mapper.md
install: npx @ecomdly/cli add rankcraft/site-migration-redirect-mapper
---

# Site migration redirect mapper

Produces a complete, tested redirect map for an online store that changes platform (for example to Shopify or Shoptet), URL structure or domain, plus the canonical, hreflang, sitemap and Search Console steps that go with it and a post-launch monitoring plan. The naive approach exports the old sitemap, matches URLs by slug similarity and sends everything unmatched to the home page. That loses the URLs the sitemap never listed (old campaigns, filter pages with backlinks, products renamed years ago), pairs look-alike variants wrongly, and creates what Google says may be treated as soft 404s. The key insight: the old and new platforms share the product's identity, not its URL. Join on product ID, SKU or EAN first, use fuzzy matching only for what is left, and weight every decision by the clicks and links each old URL actually carries.

## When to use

- Before launching a store on a new platform or theme with a different URL scheme.
- Before a domain change (rebrand, ccTLD consolidation such as moving `.sk` into `.cz/sk/`, or the reverse).
- After a launch, when organic clicks dropped and the redirects were never audited.
- When an old migration left redirect chains that the next migration would make longer.

## When not to use

- Deciding which filter URLs should be indexable on the new site: `faceted-navigation-seo-audit`. Run it before mapping filter URLs, because the map can only point to filter pages that will exist and be indexable.
- Diagnosing a click loss that is not linked to a migration: `search-console-auditor`.
- Checking product JSON-LD on the new templates: `product-schema-validator`.
- Merchant Center disapprovals caused by changed product links after launch: `merchant-center-disapproval-fixer`.
- Rewriting category copy on the new site: `category-page-copy-writer`.

## Inputs

| input | definition | typical source |
|---|---|---|
| Old URL crawl | every URL a crawler reaches from the old home page, with status code, canonical, indexability | Screaming Frog, Sitebulb or similar on the old site |
| Old XML sitemaps | all sitemap files, including image and language sitemaps | old site, Search Console Sitemaps report |
| Search Console pages | URLs with clicks and impressions over the full 16 months Search Console keeps; top linked pages | Search Console Performance (API or bulk export beyond 1,000 rows) and Links report |
| GA4 landing pages | landing pages with sessions and revenue, all channels, 12+ months | GA4 landing page report or BigQuery export |
| Backlinked URLs | old URLs with external links, including ones that already 404 | Search Console Links report, backlink tool export |
| Old catalog export | URL per product, category, brand, page, with product ID, SKU, EAN, variant IDs | old platform export |
| New catalog export | the same fields on the new platform, with the new URLs | new platform export or staging |
| Existing redirects | redirects already live on the old site from earlier moves | server config, old platform redirect list |
| Migration type | platform, URL structure, domain, protocol, or a combination; country and language versions | owner |

Optional: server log files (URLs Googlebot requests that no crawl finds); paid and email campaign URLs; old Merchant Center and comparison-site feed URLs (Heureka, Zboží.cz, idealo); the new platform's redirect limits and rule syntax.

If a required input is missing, stop and ask. Never estimate which URLs "probably" matter; an unlisted URL with backlinks is exactly the one a sitemap-only map loses.

## Best practices

### Inventory

1. **Union every source, then deduplicate.** Normalise scheme, host, trailing slash, case and encoded characters, keep the original string alongside. Each source misses something: crawls miss orphan pages, sitemaps miss filters, Search Console misses URLs without impressions. Google's own site-move guide lists sitemaps, analytics or logs, the Links report and the CMS as sources.
2. **Attach value to each URL.** Clicks, impressions, sessions, revenue, external linking domains. The value column decides review effort and lets you report what share of clicks is mapped one-to-one.
3. **Keep parameters separate.** Record query strings as their own field. Tracking parameters (`utm_*`, `gclid`) are not separate pages; filter and sort parameters may be.

### Mapping

4. **Identifier join first: product ID, then SKU, then EAN.** Product IDs survive a platform move only if migrated; SKU and EAN usually survive. Match on the variant level when variants had their own URLs, otherwise a size variant lands on the wrong colour.
5. **Fuzzy matching only for the remainder, and always reviewed.** Title or slug similarity proposes candidates; a human confirms. Mark every row with its match method so reviewers know which rows to check.
6. **One-to-one, to the closest equivalent.** Google recommends a one-to-one mapping and warns that redirecting many old URLs to one irrelevant destination such as the home page can be treated as a soft 404.
7. **Discontinued products: successor, close equivalent or 404/410.** Redirect to a direct successor or a page that serves the same intent. If none exists, let the URL return 404 or 410; Google's guide asks for exactly that for deleted content, and both remove the URL from the index.
8. **Old filter URLs follow the new facet decisions.** Map filter URLs with clicks or external links to the new indexable equivalent. Map the rest by pattern rule only where the target is a relevant parent category and the platform supports the rule.
9. **Server-side permanent redirects (301 or 308).** Google treats them as a strong signal that the target is canonical; 302 and 307 are weak signals. Use JavaScript redirects only as a last resort.

### Chains, canonicals and signals

10. **Flatten chains and break loops.** Rewrite every existing redirect whose target is now itself redirected so it points to the final URL. Googlebot follows up to 10 hops, but Google advises redirecting to the final destination directly.
11. **New URLs carry self-referencing canonicals and updated hreflang.** A canonical or hreflang still pointing to the old domain contradicts the redirect.
12. **Update internal links, sitemaps and feeds to new URLs.** Internal links that rely on redirects add a hop to every click and every crawl.
13. **Change of Address only for domain or subdomain moves.** It requires ownership of both properties under the same Google account and working 301s; it cannot be used for path-level moves or for an http to https move.

## Process

1. **Build the inventory** from all sources, normalise, deduplicate and attach value columns. Report counts per source and per page type.
2. **Classify** each old URL: product, variant, category, filter, brand, content, static, system (cart, account, search), tracking duplicate.
3. **Join identifiers** to the new catalog: ID, then SKU, then EAN; record method and any conflicts (one SKU on two new URLs).
4. **Fuzzy-match the remainder** and queue every proposal for human review, highest value first.
5. **Decide unmatched URLs:** successor, same-intent page, or 404/410, with a reason per row.
6. **Map filter URLs** using the `faceted-navigation-seo-audit` decisions; write pattern rules separately from one-to-one rows.
7. **Merge with existing redirects**, flatten chains to final targets, detect loops (A → B → A) and self-redirects.
8. **Check platform constraints:** limits, rule syntax, query-string support. On Shopify, URL redirects work only from URLs that return 404, standard plans allow up to 100,000 redirects (Plus 20,000,000), and URLs with query strings might not redirect as expected.
9. **Test on staging:** every mapped URL returns a single 301/308 to a 200 page that is indexable and canonical to itself; every 404/410 row returns that code; no row returns 302 or a chain.
10. **Prepare the launch list:** new sitemaps; old sitemap kept submitted until old URLs drop; canonicals; hreflang; robots.txt for the new site; internal links; Merchant Center and comparison-feed links; Change of Address if the domain changes.
11. **Monitor after launch** (process step owned by a human with access): re-crawl the full old list on day 1, then weekly; read Search Console Page indexing, Sitemaps and Performance for old and new URLs; fix 404s that carry clicks or links.

## Pitfalls and edge cases

- **Variant URLs collapsing into one product URL.** Fine when the new platform uses one URL per product; map each variant URL to the product with the variant preselected if the platform supports it.
- **Case and trailing-slash duplicates** on the old site multiply rows; normalise for matching, but redirect every live variant string.
- **Encoded diacritics.** Czech and Slovak slugs may exist as `č` and `%C4%8D`; test both forms.
- **Language versions.** Map each language URL to the same language on the new site; never collapse `/sk/` into `/cz/` unless the owner decided to drop that market.
- **Old redirects from previous migrations** are inventory too; forgetting them breaks links that still point to URLs two migrations old.
- **Blocked old URLs.** A redirect Googlebot cannot request is a redirect it cannot see; check that robots.txt on the old host does not disallow mapped paths.
- **Merchant Center and comparison sites** keep old product links until the feed is updated; old links then cost one redirect per click and may trigger landing-page issues.
- **Fluctuation is expected.** Google says visibility may fluctuate during a move and that showing new URLs can take a few weeks or more for medium sites.

## Rules

- The agent builds and tests the map and the checklist. It does not deploy redirects, change DNS, edit robots.txt, submit sitemaps or run Change of Address; a human with access does.
- Never map many unrelated URLs to the home page to reduce the unmatched count.
- Every fuzzy match is reviewed by a human before it enters the final map.
- Every row records its match method and the value it carries; nothing is dropped silently.
- Redirect type is 301 or 308 unless the owner documents a reason.
- Keep redirects for at least one year (Google's site-move guidance) and, for domain moves, at least 180 days and longer while Search traffic still reaches them (Change of Address guidance). Recommend longer where the owner can.

## Output format

```
REDIRECT MAP · <store> · <old> → <new> · migration type <platform|structure|domain> · prepared <date>
Inventory: <n> unique URLs (crawl <n>, sitemap <n>, Search Console <n>, GA4 <n>, backlinks <n>, logs <n>)
By type: product <n> · category <n> · filter <n> · content <n> · static <n> · system <n>

Mapping                    rows   clicks (period)   share of clicks
  ID / SKU / EAN match      <n>    <n>               <x>%
  fuzzy, reviewed           <n>    <n>               <x>%
  successor / same intent   <n>    <n>               <x>%
  404 / 410                 <n>    <n>               <x>%
  pattern rule              <n>    <n>               <x>%
Chains flattened: <n> · loops fixed: <n> · platform limit used: <n> of <limit>

Map file columns: old_url | new_url | status | method | page_type | clicks | linking_domains | reviewer | note
Staging test: <n> pass · <n> fail (list)
Launch checklist: sitemaps <ok> · canonicals <ok> · hreflang <ok> · internal links <ok> · feeds <ok> · robots.txt <ok> · Change of Address <n/a|ready>
Monitoring dates: <day 1> · <weekly until> · redirects kept until at least <date>
Open questions: <list>
```

## Worked example

Illustrative numbers only. Czech store moving from a custom platform on `old.example` to Shopify on `new.example`, Search Console data for the last 16 months.

```
Inventory: 12,480 unique URLs
By type: product 6,200 · category 410 · filter 4,950 · content 320 · static 600

Products (6,200): ID 5,640 · SKU 310 · EAN 95 · fuzzy reviewed 40 · discontinued 115
  discontinued: successor 38 · same-intent category 29 · 410 48
Filters (4,950): 140 with clicks or links → one-to-one to new indexable collection filters
                 4,810 → pattern rule to parent collection, tested per pattern
Chains: 214 redirects from the 2021 move (A → B) re-pointed to final target C; 3 loops removed
Clicks covered: one-to-one 179,900 · 410 1,100 · pattern rule 1,400 · total 182,400
```

Check: 6,200 + 410 + 4,950 + 320 + 600 = 12,480. 5,640 + 310 + 95 + 40 + 115 = 6,200. 38 + 29 + 48 = 115. 140 + 4,810 = 4,950. 179,900 + 1,100 + 1,400 = 182,400; one-to-one share = 179,900 ÷ 182,400 = 98.6%. Because Shopify redirects from 404 URLs only and query strings may not redirect as expected, the 4,810 parameter URLs are tested on staging before relying on the rule; any that do not redirect are listed for the owner instead of assumed to work.

## Quality checklist

- Inventory uses at least crawl, sitemap, Search Console and GA4, with counts per source.
- Every row has a method, value columns and either a target or a 404/410 reason.
- No unrelated many-to-one redirects to the home page or a generic category.
- All fuzzy rows reviewed; reviewer named.
- No chains, loops or temporary redirects in the staging test.
- New pages are 200, indexable and self-canonical; hreflang points to new URLs.
- Platform limits and query-string behaviour checked.
- Change of Address included only for a domain or subdomain move.
- Monitoring plan with dates and the minimum redirect retention date.

## Sources

- Google Search Central, Site moves with URL changes: https://developers.google.com/search/docs/crawling-indexing/site-move-with-url-changes
- Google Search Central, Redirects and Google Search: https://developers.google.com/search/docs/crawling-indexing/301-redirects
- Google Search Central, How HTTP status codes and network errors affect Google Search: https://developers.google.com/search/docs/crawling-indexing/http-network-errors
- Google Search Console Help, Change of Address tool: https://support.google.com/webmasters/answer/9370220
- Google Search Central Blog, A deep dive into Search Console performance data filtering and limits: https://developers.google.com/search/blog/2022/10/performance-data-deep-dive
- Google Analytics Help, Connect Search Console to Google Analytics (16 months of Search Console data): https://support.google.com/analytics/answer/10737381
- Shopify Help Center, URL redirects: https://help.shopify.com/en/manual/online-store/menus-and-links/url-redirect

## License
MIT
