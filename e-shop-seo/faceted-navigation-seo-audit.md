---
name: faceted-navigation-seo-audit
owner: rankcraft
category: E-shop SEO
description: Audits filter URLs on category pages: which facets deserve an indexable page, which get noindex or a canonical, and which parameter combinations are crawl traps.
version: v2
license: MIT
updated: 2026-08-28
recommended: false
security_checked: true
url: https://ecomdly.com/skills/rankcraft/faceted-navigation-seo-audit
raw: https://ecomdly.com/raw/rankcraft/faceted-navigation-seo-audit.md
install: npx @ecomdly/cli add rankcraft/faceted-navigation-seo-audit
---

# Faceted navigation SEO audit

Filters help shoppers and flood crawlers. This skill decides, facet by facet, what gets a real landing page and what stays out of the index, from crawl and demand data.

## When to use
- When the crawl shows far more URLs than the store has products.
- When Search Console reports many "Crawled – currently not indexed" or "Duplicate without user-selected canonical" URLs on filter pages.

## Input
A crawl export (URL, status, indexability, canonical, meta robots, inlinks), the list of facets and their URL pattern (parameter or path), server log hits by Googlebot if available, and Search Console or keyword-tool demand for facet terms (e.g. "black trail shoes"). The robots.txt file.

## Classify each facet
- **Indexable** — real search demand for category + value (brand, main color, gender, type), and at least 3 products behind it. Give it a clean path URL, a self-canonical, its own H1 and title, and link it from the category.
- **Noindex, follow** — useful for shoppers, no search demand (price range, rating, in stock). Keep crawlable only if Googlebot is not wasting time on it.
- **Canonical to parent** — near-duplicates: sort order, view mode, items per page.
- **Block in robots.txt** — combinations that explode: two or more facets together, sort + filter, session or tracking parameters. Blocked URLs cannot pass noindex, so only block what is already out of the index.

## Crawl trap checks
- Parameter order variants (`?color=black&size=43` vs `?size=43&color=black`) served as separate URLs.
- Empty result pages returning 200.
- Facet links on pagination pages multiplying combinations.
- Canonical pointing to a URL that is itself noindex or redirected.

## Rules
- Demand comes from data the user provides; without it, mark the facet `demand unknown` instead of guessing.
- Never recommend deindexing a URL that earns clicks without naming the clicks it earns.

## Output format
```
facet: brand (path /shoes/salomon/) — indexable; 2 900 impr./28d, 46 products
facet: color+size combos (?color=&size=) — 38 400 crawled URLs, 0 clicks → robots.txt block after noindex drops them
trap: sort param order duplicates, 6 100 URLs → normalise parameter order
```

## License
MIT
