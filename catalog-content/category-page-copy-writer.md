---
name: category-page-copy-writer
owner: rankcraft
category: Catalog content
description: Writes the title, meta, short intro, buying guide and evidence-based FAQ for a store category page from the category's dated product export and Search Console queries, with every number traceable and unsupported claims flagged.
version: v3
license: MIT
updated: 2026-09-29
recommended: false
security_checked: false
url: https://ecomdly.com/skills/rankcraft/category-page-copy-writer
raw: https://ecomdly.com/raw/rankcraft/category-page-copy-writer.md
install: npx @ecomdly/cli add rankcraft/category-page-copy-writer
---

# Category page copy writer

Produces the full text package for one category page: title, meta description, H1, a short intro above the product grid, a buying guide below it, and optional FAQ, all built from the category's real product data and real search queries. Category pages rank for the money queries, yet most carry either nothing or 800 words of generic filler pushed above the products. Both fail: empty pages give Google and the shopper no way to tell the page apart from twenty competitors with the same grid, and filler pushes products off the first screen while saying nothing a shopper can act on. The copy that works is short where the shopper is scanning, specific where they are choosing, and every number in it comes from the catalog on the day of writing.

## When to use
- A category page has impressions but low CTR, or sits at positions 8–20 for its main queries in Search Console.
- A new category has no copy, or several categories and filter pages share the same boilerplate intro.
- `search-console-auditor` flagged the page's title as a free-click opportunity, or `faceted-navigation-seo-audit` marked a facet page as indexable and it needs unique copy.
- The assortment changed materially (new brands, price range moved), so existing copy is now wrong.

## When not to use
- Product pages: use `product-description-writer`.
- Filter or sort URLs that are not indexable by the store's rules: they get no unique copy; flag them instead.
- Translating existing category copy: use `catalog-translation-localizer`.
- Pages with no product data available: write nothing and ask for the export; copy without numbers is the filler this skill exists to prevent.

## Inputs

Required:
- **Category**: name, URL, parent and child categories, and indexable facet pages that link from it.
- **Product export for the category** (platform admin export or feed): title, brand, price (gross, with currency), key attributes used for filtering (e.g. waterproofing, drop, size range, material, capacity), stock status. Date of the export.
- **Search queries for the page**: Search Console Performance, filter Page = the category URL, dimension Queries, last 3 months: query, clicks, impressions, CTR, position.

Optional but valuable:
- **Voice guide**: person (we/you, formal/informal address in the target language), spelling variant, banned words, brand terms.
- **Customer questions**: on-site search terms that led to this category, support tickets, chat logs, returns reasons (a common returns reason is a buying-guide topic).
- **Existing copy** and current title/meta, with any rankings it holds.
- **Competitor SERP snapshot** for the main query (top 5 titles and what their category pages cover), taken in the store's market and language.
- **Store policies** shown sitewide (delivery threshold, return window) if the owner wants them mentioned.

If the query export is missing, write from product data only and mark the title and meta `UNVALIDATED - no query data`. If attribute data is too thin to support a buying guide (only titles and prices), write the intro and title/meta only and list the attributes needed.

## Best practices

### Page structure
1. **Intro above the grid: 40–70 words, 2–3 sentences.** What the category holds (types, brands), the range in numbers (product count, price span, main sizes), and one line on the main choice to make. On mobile, the first product row must remain visible on first load; if the template renders more text above the grid, use a "read more" collapse only for text beyond the first 2 sentences. The word range is a working default for keeping products on the first screen, not a Google rule.
2. **Buying guide below the grid: 250–500 words**, as H2 sections that each answer one real decision the shopper faces. Derive the sections from (a) attributes with real variance in the product export (if every product is waterproof, waterproofing is not a decision), (b) modifiers in the query data ("for wide feet", "for beginners", "gore-tex vs"), (c) customer questions and returns reasons. Three to five sections.
3. **Each guide section ends with a link** to the filter, subcategory or indexable facet page that applies the choice ("See all 14 waterproof models"). The guide is navigation, not an essay; a section with no linkable destination is a candidate to cut.
4. **FAQ is optional, 3–5 questions, only from evidence** (queries phrased as questions, on-site search, support tickets), cite the source per question. Google stopped showing the FAQ rich result in Search from 7 May 2026, so FAQ markup brings no rich result; the FAQ earns its place only if it answers questions shoppers actually ask.
5. **Keep product-level facts at product level.** The category copy summarises the range; it does not describe single products beyond naming a representative example.

### Title, meta, H1
6. **Title: primary query first, then the differentiator from the data, then brand if space allows.** Google sets no length limit but truncates the title link to fit the device width, so the first ~50–60 characters must carry the meaning; state the character count. Google also rewrites titles that are keyword-stuffed, boilerplate or vague, so each category title must be distinct from sibling categories by more than one word.
7. **Choose the primary query from data**: the non-brand query with the most impressions whose intent matches the whole category (not a single brand or model). Use the plural/category form users type in that market. Secondary queries become H2s or natural variants, not title additions.
8. **Meta description: one or two sentences naming the range and one concrete differentiator** from the data (brand list, count, price span, a selection feature). Google sets no length limit and truncates to device width, so front-load the specific part; it may also generate its own snippet from page text, which is another reason the intro must be specific. Google encourages programmatic, page-specific descriptions for large sites; if the store needs this at scale, propose a data-driven template instead of one-off text.
9. **H1 = the category name in the form shoppers search**, can differ from the title. One H1.
10. **Primary query once in the H1, once in the intro, then natural variants.** Repeating it or listing keyword variants is keyword stuffing under Google's spam policies and reads badly.

### Numbers and claims
11. **Every number comes from the product export on the export date**: count of products (in stock or total; say which), price span (min–max gross), brands (list up to 4, then "and N more"). Print `Data as of: <date>` in the output so the copy is refreshed when the assortment changes. Prefer ranges that age slowly ("from 1 290 Kč") over exact counts in body text where the platform cannot update them automatically.
12. **No delivery, return, price or discount claims that are not in the input.** If the store wants "free delivery over 1 500 Kč", it must be in the provided policy. Do not write "best prices" or "cheapest" (unprovable superlatives). Any mention of a price reduction in the EU must follow the prior-price rule (the lowest price in the previous 30 days, Directive 98/6/EC as amended); keep discount language out of evergreen category copy.
13. **Environmental and social claims**: from 27 September 2026 the EU rules added by Directive (EU) 2024/825 prohibit generic environmental claims ("eco-friendly", "green", "sustainable") unless the trader can demonstrate recognised excellent environmental performance, and sustainability labels not based on a certification scheme or public authority. In a category like "sustainable outdoor clothing", write the specific, verifiable attribute instead ("made with recycled polyester, share per product listed on each page") only if the product data supports it; otherwise flag the category name itself for owner review.
14. **No reviews, ratings or "customers love" lines**; no competitor names.

### Uniqueness and scale
15. **Each category and indexable facet page must say something only it can say**: its own range numbers, its own decision criteria. Swapping the category name into a template produces boilerplate that Google's spam policies describe as scaled content abuse when done at scale to manipulate rankings, and it does not help the shopper. If two categories genuinely need the same guide, write it once and link to it.
16. **No city or region pages, no lists of locations** ("trail shoes Prague, Brno, Ostrava"): doorway pattern.
17. **Write in the shopper's language with local units and conventions** (CZK with "Kč", EU sizes, metric). Keep product and brand casing as the brand writes it.

## Process
1. **Profile the range** from the export: product count (in stock / total), price min–max and median, brands with counts, and each filter attribute's distinct values and distribution. Note attributes with real variance.
2. **Profile demand**: from the query export, drop brand-of-store queries, group queries by head term and modifiers. Pick the primary query (rule 7) and list the modifiers with impressions.
3. **Map decisions**: intersect attributes with variance, query modifiers and customer questions. Each intersection with evidence is a candidate H2. Rank by impressions + question frequency; keep 3–5.
4. **Check link targets** for each H2: an existing filter, subcategory or facet page that is indexable. If none exists, keep the section only if it answers a frequent question, and note the missing destination.
5. **Draft title and meta** (rules 6–8), count characters, and compare against sibling category titles for distinctness.
6. **Draft the intro** (rule 1), then the guide sections: each section 60–120 words, states the choice, gives the rule of thumb supported by the data, names how many products meet it, and links.
7. **Draft FAQ** only from evidence, with source per question.
8. **Claim check**: highlight every number and claim and trace it to the export, the query data, or the provided policy. Anything untraceable is cut or moved to `flags`.
9. **Word-count check**: intro 40–70, guide 250–500. Over length: cut the weakest section, not sentences from every section.

## Pitfalls and edge cases
- **Small categories (< 10 products)**: a 400-word guide over eight products is out of proportion. Write the intro and 1–2 short sections, or propose merging the category.
- **Mixed-intent categories** ("Accessories"): the guide should route to subcategories, not explain each product type.
- **Seasonal ranges**: a count written in winter will be wrong in summer; prefer ranges or let the platform insert live counts.
- **Existing copy that ranks**: if the page already ranks top 3 for its main query, keep its main terms and structure; change facts and add sections rather than rewriting from zero, and say so.
- **Size-fit advice** ("runs small") needs evidence (returns reasons, brand size charts); otherwise leave it out.
- **Price with VAT**: B2C copy uses gross prices; a B2B store may show net; use what the storefront shows.
- **Duplicate intros on paginated pages**: the intro and guide belong on page 1; confirm the template does not repeat them on page 2+ (each paginated page keeps its own URL and canonical).

## Rules
- Never invent a number, attribute, policy, review or customer question. Unsupported = cut or flag.
- Numbers only from the dated product export; always print the date.
- No copy for non-indexable filter pages; flag them.
- Output is a proposal; a human approves and publishes. Never edit the store directly.
- Follow the voice guide; if none is provided, use neutral second person and say so.
- Environmental, health and legal claims are flagged for owner review, never softened into vaguer wording.

## Output format
```text
URL: <url>
Primary query: "<query>" (<impr> impr., pos <x>, 3m)   Secondary: "<q>", "<q>"
Title (<n> chars): <title>
Meta (<n> chars): <meta>
H1: <h1>

Intro (<n> words, above grid):
<text>

Buying guide (<n> words, below grid):
## <H2>
<text> -> link: <target url> (<n> products)
## <H2>
...

FAQ (optional):
Q: <question> (source: <queries/site search/tickets, count>)
A: <answer>

Data as of: <export date> | products <n> (<in stock/total>) | price <min>-<max> <cur> | brands <list>
Flags:
- <unsupported claim removed / missing attribute / non-indexable page / owner decision>
```

## Worked example
Example data, not a real store. Export 2026-09-25: 64 trail running shoes (58 in stock), 1 290–4 590 Kč, brands Salomon 22, Hoka 15, Inov-8 12, La Sportiva 9, 6 from two other brands; 26 models with a waterproof membrane; drop values 0–10 mm. Queries (3 months): "trail running shoes" 18 400 impr., pos 9.4; "waterproof trail running shoes" 4 100; "trail shoes wide feet" 900; site search "gore-tex" 310 times, "wide" 120 times.

```text
URL: /trail-running-shoes
Primary query: "trail running shoes" (18 400 impr., pos 9.4, 3m)   Secondary: "waterproof trail running shoes", "trail shoes wide feet"
Title (55 chars): Trail Running Shoes - Salomon, Hoka, Inov-8 | 64 Models
Meta (122 chars): 64 trail running shoes from Salomon, Hoka, Inov-8 and La Sportiva, 1 290-4 590 Kč. Filter by waterproof membrane and drop.
H1: Trail running shoes

Intro (47 words, above grid):
Trail running shoes from Salomon, Hoka, Inov-8, La Sportiva and 2 more brands, from 1 290 Kč. Start with the conditions you run in: 26 models have a waterproof membrane for wet and muddy trails, 38 have none. Then choose the drop, from 0 to 10 mm.

Buying guide (310 words, below grid):
## Waterproof membrane or fast-draining mesh?
... -> link: /trail-running-shoes/waterproof/ (26 products)
## How much drop do you need?
... -> link: /trail-running-shoes?drop=0-4 (not indexable - filter link only)
## Trail shoes for wide feet
... -> link: none - wide-fit attribute missing in export

FAQ: none - no question-form queries above 100 impr.

Data as of: 2026-09-25 | products 64 (58 in stock) | price 1 290-4 590 Kč | brands Salomon, Hoka, Inov-8, La Sportiva +2
Flags:
- Wide-fit section needs a width attribute per product; 120 site searches for "wide" support the section.
- Title count "64" must be updated when the range changes, or replaced by a live count.
```

## Quality checklist
- Every number in the copy matches the dated export; the date is printed.
- Primary query chosen from query data (or marked UNVALIDATED); it appears in H1 and intro once each.
- Title and meta are distinct from sibling categories; character counts shown.
- Intro 40–70 words; guide 250–500 words; each guide section has a link target or a stated reason.
- No delivery, return, price, discount, review, superlative or environmental claim without a source in the input.
- FAQ questions each cite their evidence.
- Non-indexable pages flagged, not written.
- Voice guide applied, or its absence stated.

## Sources
- https://developers.google.com/search/docs/appearance/title-link
- https://developers.google.com/search/docs/appearance/snippet
- https://developers.google.com/search/docs/essentials/spam-policies
- https://developers.google.com/search/docs/appearance/structured-data/search-gallery
- https://developers.google.com/search/updates
- https://developers.google.com/search/docs/specialty/ecommerce/pagination-and-incremental-page-loading
- https://eur-lex.europa.eu/eli/dir/2024/825/oj
- https://eur-lex.europa.eu/eli/dir/1998/6/oj
- https://commission.europa.eu/document/download/3c257883-bb2a-4dd9-a6dc-501d587bb34f_en?filename=faq-empowerting-consumers-gtd.pdf

## License
MIT
