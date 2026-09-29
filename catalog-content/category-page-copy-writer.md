---
name: category-page-copy-writer
owner: rankcraft
category: Catalog content
description: Writes category page intros and buying-guide blocks from real product data and search queries — short above the grid, useful below it, no keyword stuffing.
version: v2
license: MIT
updated: 2026-09-05
recommended: false
security_checked: true
url: https://ecomdly.com/skills/rankcraft/category-page-copy-writer
raw: https://ecomdly.com/raw/rankcraft/category-page-copy-writer.md
install: npx @ecomdly/cli add rankcraft/category-page-copy-writer
---

# Category page copy writer

Category pages rank for the money keywords, yet most carry either nothing or 800 words of filler pushed above the products. This skill writes copy that helps the shopper choose and gives Google a reason to rank the page.

## When to use
- Category pages with impressions but low CTR or position 8–20 in Search Console.
- New categories with no copy at all, or duplicated intros across filter pages.

## Input
- Category name, URL, and the product list in it (title, brand, price, key attributes, stock count).
- Search queries for the page (Search Console export: query, clicks, impressions, position).
- Existing copy, the store's voice guide, and real customer questions (support tickets, on-site search) if available.

## Structure
1. **Intro above the grid (40–70 words):** what the category holds, the main range (brands, price span, key sizes), one line on how to choose. Products must stay visible on mobile without scrolling past text.
2. **Buying guide below the grid (250–500 words):** H2s built from the real decision criteria in the data, e.g. "Waterproof or breathable", "Which size for...". Each section links to a matching filter or subcategory.
3. **FAQ (3–5 questions):** only questions that appear in the input queries or support data. No questions made up to fit a schema.
4. **Title and meta:** title ≤ 60 characters with the primary query first; meta ≤ 155 characters naming the range and one concrete differentiator from the data.

## Rules
- Numbers (product count, price range, brands) come from the product list on the day of writing; state that date so it can be refreshed.
- Primary query once in the H1, once in the intro, then natural variants. No lists of keywords, no city spam.
- No claims about delivery, returns or prices that are not in the input.
- Never write product reviews, ratings or "customers love" lines.
- Filter pages (`?color=black`) get no unique copy unless they are indexable by the store's rules; flag them instead.

## Output format
```
URL: /trail-running-shoes
Title: Trail Running Shoes for Men & Women | 64 Models
Meta: Trail shoes from Salomon, Hoka and Inov-8, EUR 89-229, filtered by grip, drop and waterproofing.
Intro: ...
## Waterproof or breathable?
...
FAQ: "Do trail shoes run small?" (source: 38 site searches)
Data as of: 2026-09-05 · Flags: none
```

## License
MIT
