---
name: marketplace-listing-adapter
owner: listwise
category: Marketplaces
description: Adapts one master product record into Amazon, Allegro and Kaufland listings within each title/bullet limit and category attribute set, flagging every missing value.
version: v2
license: MIT
updated: 2026-09-02
recommended: false
security_checked: true
url: https://ecomdly.com/skills/listwise/marketplace-listing-adapter
raw: https://ecomdly.com/raw/listwise/marketplace-listing-adapter.md
install: npx @ecomdly/cli add listwise/marketplace-listing-adapter
---

# Marketplace listing adapter

Every marketplace wants the same product told differently. This skill takes your master record and fits it into each channel's limits and attribute schema, without inventing a single spec.

## When to use
- Before listing a new product on a marketplace you already sell on.
- When a channel rejects or suppresses listings for missing attributes.

## Input
The master record (title, description, brand, GTIN/EAN, MPN, dimensions, weight, material, color, size, variants, images count), the target marketplaces, and the target category on each (ID or full path). If the user has the channel's category template export, use its required-attribute list as the source of truth.

## Channel limits
- **Amazon (EU stores):** title ≤ 200 characters, no word repeated more than twice, none of `! $ ? _ { } ^ ¬ ¦`; 5 bullets, keep each under 250 characters; search terms under 250 bytes, no brand names you do not own, no repeats of words already in the title.
- **Allegro:** title ≤ 75 characters; required parameters depend on the category and are validated on save; EAN required where the category is catalog-driven.
- **Kaufland / Heureka Marketplace:** matched by EAN first; the title comes from the catalog when a match exists, so fix attributes rather than the title.

## Mapping steps
1. Resolve the category and list its required and recommended attributes.
2. Map each master field to the channel attribute; convert units to what the channel expects (cm, kg, EU sizes).
3. For enumerated attributes (color, material), pick the closest allowed value and keep the original in the free-text field.
4. Build the title per channel from mapped attributes only: Brand · Product type · Key attribute · Variant.
5. Write bullets as facts: one benefit per bullet, anchored to a spec from the record.

## Rules
- A value missing from the master record stays missing and goes in `gaps`. Never guess dimensions, materials or certifications.
- No promotional words, prices or shipping claims in titles or bullets.
- Translate into the marketplace language (Polish for Allegro.pl, German for Kaufland.de) and mark text as machine-drafted for human review.

## Output format
```
channel: allegro.pl | category: Obuwie > Męskie > Sportowe (ID 257938)
title (61/75): Salomon Speedcross 6 GTX buty trailowe męskie czarne 43
params: Marka=Salomon; Rozmiar=43; Kolor=czarny; Materiał wierzchni=tkanina
gaps: "Wysokość cholewki" required, not in master record
review: title machine-translated, check before publishing
```

## License
MIT
