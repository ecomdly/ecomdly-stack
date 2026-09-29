---
name: marketplace-listing-adapter
owner: listwise
category: Marketplaces
description: Turns a master product record into Amazon, Allegro and Kaufland listing drafts: leaf category, title within the channel limit, mapped attributes, GPSR fields and a blocking-gap list, for sellers expanding to marketplaces.
version: v3
license: MIT
updated: 2026-09-29
recommended: false
security_checked: true
url: https://ecomdly.com/skills/listwise/marketplace-listing-adapter
raw: https://ecomdly.com/raw/listwise/marketplace-listing-adapter.md
install: npx @ecomdly/cli add listwise/marketplace-listing-adapter
---

# Marketplace listing adapter

Takes one master product record and produces a channel-ready listing draft per marketplace (Amazon EU stores, Allegro, Kaufland): category, title, bullets or description, mapped category attributes, GPSR safety fields, and an explicit gap list. Every marketplace wants the same product described differently, and the naive approach (reuse the shop title, translate it, paste attributes) fails on three things: each channel's own title rules, required attributes that differ per leaf category, and catalog matching, where the channel already has a product page for that EAN and your title is not what the shopper sees. This skill works from the channel's category template as the source of truth, never invents a value to satisfy a required field, and tells the seller where fixing data beats rewriting copy.

## When to use
- Listing an existing product on a marketplace for the first time, or adding a new marketplace for the catalog.
- A channel rejects, suppresses or blocks listings for missing or invalid attributes, titles or safety information.
- Before a bulk upload, to produce a mapped file and gap list per category.

## When not to use
- Comparison shopping feeds (Heureka, Zboží.cz, Google Shopping): use `comparison-feed-mapper` or `product-feed-optimizer`.
- The master record itself lacks attributes buried in free text: run `product-attribute-extractor` first.
- Identifier problems (no GTIN, GTIN belongs to another product): `gtin-identifier-auditor`.
- Translating a whole catalog: `catalog-translation-localizer`, then use this skill to fit the translated text to channel rules.

## Inputs
Required:
- Master record per product/variant: id, brand, manufacturer, GTIN/EAN, MPN, product type, title, description, key specs, dimensions and weight (product and package), material, colour, size and size system, variant structure (parent and children with the varying attributes), image URLs and count, country of origin if known.
- GPSR data: manufacturer name, postal and electronic address; EU responsible person if the manufacturer is outside the EU; warnings and safety information text.
- Target marketplaces and storefronts (amazon.de, allegro.pl, allegro.cz, kaufland.de, kaufland.cz, ...).

Strongly recommended:
- The channel's category template or parameter list for the target category: Amazon category listing report / flat file template for the product type; Allegro `GET /sale/categories/{categoryId}/parameters` output or the parameter list shown in the offer form; Kaufland category attributes from the seller portal or the API categories endpoint.
- Whether a catalog product already exists for the EAN on each channel (search by EAN in the seller tool).
- Brand ownership or authorisation status (Amazon Brand Registry, reseller of a third-party brand).

If the category template is missing, the agent produces a draft marked "attributes unconfirmed" and lists the template it needs. If GPSR data is missing, the listing is not publish-ready on any EU channel; say so at the top.

## Best practices

### Treat limits as data, not memory
1. Marketplace limits change and differ by storefront and category. Before producing titles, read the current rule in the channel's own seller documentation or in the error returned by a draft listing, and record the version or date used in the output. The limits below are the starting point verified or noted at writing time; the agent confirms each one for the seller's storefront and category before relying on it.

### Amazon (EU storefronts)
2. Title: Amazon's published product title requirements (as of 2025) set a maximum of 200 characters including spaces for most categories, prohibit the characters `! $ ? _ { } ^ ¬ ¦` unless part of the brand name, and prohibit repeating the same word more than twice (prepositions, articles and conjunctions excepted). Category style guides can set lower limits. Confirm in Seller Central Help ("Product title requirements") for the storefront (the page requires a seller login).
3. Bullets (key product features): typically up to 5; Amazon's category style guides give per-bullet length guidance. Confirm the current limit in the style guide for the category. One fact-based feature per bullet, starting with the attribute the category buyer compares on.
4. Search terms (generic keywords): a byte limit applies (widely documented as under 250 bytes for most storefronts; confirm in Seller Central). Bytes, not characters: umlauts and Czech/Polish diacritics take 2 bytes in UTF-8. No brand names you do not own, no ASINs, no words already in the title, no subjective or temporary claims.
5. Catalog matching: Amazon matches by GTIN and brand to an existing ASIN. If an ASIN exists and you are not the brand owner, your title and bullets may not be used; fix attribute data and offer data, and propose content changes to the brand owner or via Amazon's contribution process.
6. Variations use a parent-child structure with a variation theme (for example Size/Color) defined per product type. Children need the theme attributes filled; the parent is not buyable.

### Allegro
7. Title (`name`): maximum 75 characters and minimum 12 characters including spaces, and at least 3 words, per Allegro's developer documentation for offer creation (verified). Allegro publishes the allowed character set; strip characters outside it.
8. Offers are created only in leaf categories. Parameters come from `GET /sale/categories/{categoryId}/parameters`; the `required` flag marks parameters mandatory for the offer and `requiredForProduct` those mandatory when proposing a new catalog product. Missing required parameters return HTTP 422 with `MissingRequiredParameters` listing their IDs. Use this response as a validation step, not guesswork.
9. Allegro links offers to catalog products; EAN (GTIN) is a key identifying parameter but Allegro notes it does not identify a product uniquely on its own. When an existing product matches, its product parameters apply; your offer-level data adds to it.
10. Dictionary parameters accept only values from the list. Pick the exact dictionary value; if none fits, use the "other" option only when the dictionary provides one and keep the precise value in the description. Never pick a close value that changes the meaning (a 42 cm item is not "40 cm").
11. Images: at least 1, maximum 16 per offer (developer documentation). Allegro's image rules (background, watermarks) are in the seller help; confirm them for the category.
12. GPSR: the offer model includes `responsibleProducer`, `responsiblePerson` and `safetyInformation`. They are needed for goods covered by the GPSR; if the data is missing, the gap is blocking.

### Kaufland
13. Kaufland identifies a product by EAN; without an EAN the product cannot be identified (seller API product data documentation). Rows carry a `locale` (for example `de-DE`, `cs-CZ`), so text per storefront language is uploaded per locale.
14. The public product page is assembled from several sources (product information services, sellers, manufacturers, Kaufland's data team). Your uploaded title or description may not be selected. When the product already exists, improve attributes and images, and do not expect a rewritten title to appear.
15. Products live only in leaf categories. You may send a higher-level category and Kaufland will choose the leaf from the rest of the data, or leave the category blank; send enough attributes for correct placement, and check the assigned category after import.
16. Send additional attributes you hold even if not in the category's list (Kaufland accepts user-defined attribute columns named in German or English). CSV format: semicolon-separated, UTF-8, header row of attribute names, repeated columns for multi-value attributes.
17. Kaufland title length and style rules are in its product data guidelines; the agent could not confirm a numeric limit from public documentation. Confirm it in the seller portal before bulk generation.
18. GPSR contact data is managed as its own object in Kaufland's seller tools; map manufacturer and responsible person to it.

### Content rules across channels
19. Title formula by default: Brand + product type + key differentiating attribute(s) + variant attribute(s) (size, colour, capacity). Order changes only if the channel's style guide says so. Build titles from mapped attributes, then read them as a shopper: no keyword lists.
20. No promotional words, prices, shipping promises, "bestseller", "top quality", or claims not in the record, in titles or bullets. Certifications and compliance marks appear only when the record contains the certificate reference.
21. Units in the channel's expected unit and format (cm, kg, EU sizes, decimal comma in Czech/Polish/German text), converted with exact factors and rounded only where the channel's field precision requires it.
22. Language: each storefront in its language (Polish for allegro.pl, Czech for allegro.cz and kaufland.cz, German for amazon.de and kaufland.de). Machine-drafted text is marked for human review. Safety information must be in a language easily understood by consumers in the member state (GPSR Art. 19).
23. The GPSR (Regulation (EU) 2023/988, Art. 19) requires distance-sales offers to show the manufacturer's name and postal and electronic address, the EU responsible person where applicable, information identifying the product including a picture and type, and warnings or safety information. Online marketplaces must enable traders to display it (Art. 22).

## Process
1. Validate the master record: required fields present, GTIN valid (checksum), variant structure consistent. List missing fields per product.
2. For each channel, resolve the leaf category (ID and full path). If ambiguous, give the top two candidates with the reason and ask.
3. Check catalog matching by EAN on each channel. If a catalog product exists, switch mode to "offer on existing product": map offer fields and propose attribute corrections; title and bullets are secondary.
4. Load the category's attribute list. For each attribute: map from the master record (exact field), convert units, choose dictionary values. Unmappable required attribute → `gaps` (blocking). Unmappable recommended attribute → `gaps` (non-blocking).
5. Build the title per channel from mapped attributes; count characters (and bytes for Amazon search terms). Over limit → remove in this order: secondary attributes, product-type synonyms; never brand, product type or variant attribute.
6. Write bullets (Amazon) or a structured description (Allegro sections, Kaufland description): each point one fact from the record with its benefit, no facts from outside the record.
7. Map GPSR fields. Missing → blocking gap.
8. Validate against the channel: lengths, prohibited characters, repeated words, dictionary values, image count. If the seller can run a dry-run or draft save, recommend it and read its errors.
9. Produce the output per channel with status: `ready for review`, `blocked (gaps)` or `offer on existing product`.

## Pitfalls and edge cases
- Variants: one Allegro offer per variant grouped by variant sets, Amazon parent-child with a variation theme, Kaufland per EAN. A master variant without its own EAN may not be listable as a separate item on Kaufland; flag it.
- Bundles and multipacks: need their own GTIN on most channels; do not reuse the single item's EAN.
- Brand restrictions and gated categories on Amazon (and brand authorisation on other channels): check before preparing content.
- Size systems: channel size dictionaries may be EU-only; converting UK or US sizes needs the brand size chart.
- Byte counting for search terms with diacritics; character counting for titles.
- Energy labels, textile composition and other sector labelling fields are category-specific mandatory fields on some channels; treat them as required when the template marks them.
- Copying the shop description with HTML: some channels strip or reject tags; Allegro uses its own section structure.
- Prices and VAT per storefront are not part of the listing content; do not put them in text.

## Rules
- Read-only towards all channels: output is a draft for the seller to review and upload. Never create, edit or publish listings, and never call a seller API that writes data.
- A value missing from the master record stays missing and appears in `gaps`. Never guess dimensions, materials, certifications, compatibility or safety information.
- Never choose a dictionary value that is not true for the product to pass validation.
- Every limit used is stated with its source or marked "confirm current limit in <channel doc>".
- Machine-translated text is marked for human review.
- GPSR gaps block publication; say so plainly.

## Output format
```
product: <master id> | EAN <gtin> (checksum ok|invalid) | brand <brand>
channel: <storefront> | mode: new product | offer on existing product
category: <full path> (ID <id>) | template: <source, date>
title (<chars>/<limit>, limit source: <doc|confirm>): <title>
bullets / description:
  1. <fact-based point>
attributes: <attr>=<value> (<source field>); ...
gpsr: manufacturer=<ok|missing>; responsible_person=<ok|missing|n/a>; safety_info=<ok|missing>
gaps (blocking): <attr> - <why>
gaps (non-blocking): <attr>
checks: prohibited chars <ok|fail> | repeated words <ok|fail> | dictionary values <ok|fail> | images <n>/<limit>
status: ready for review | blocked | offer on existing product
review: <machine-translated fields, limits to confirm>
```

## Worked example
Example master record: Trail running shoe, brand "Stoneridge", EAN 8594001234561 (checksum valid), men's, EU 43, black, upper textile, sole rubber, weight 290 g per shoe, GPSR manufacturer data present, safety info not applicable, no upper-height value in the record. Targets: allegro.pl, kaufland.de.

```
product: TR-43-BLK | EAN 8594001234561 (checksum ok) | brand Stoneridge
channel: allegro.pl | mode: new product
category: Sport i turystyka > Bieganie > Obuwie > Męskie (ID from category tree, confirm) | template: GET /sale/categories/{id}/parameters, 2026-09-29
title (50/75, limit source: Allegro developer docs): Stoneridge Trail Runner buty do biegania męskie 43
attributes: Marka=Stoneridge (brand); EAN (GTIN)=8594001234561 (gtin); Rozmiar=43 (size); Kolor=czarny (color); Materiał wierzchni=tkanina (material_upper); Płeć=mężczyzna (gender)
gpsr: manufacturer=ok; responsible_person=n/a (EU manufacturer); safety_info=ok (none required per record, confirm)
gaps (blocking): "Wysokość cholewki" required in template, not in master record
checks: prohibited chars ok | dictionary values ok | images 6/16
status: blocked
review: title and parameter values machine-translated to Polish

channel: kaufland.de | mode: offer on existing product (EAN found in Kaufland catalog)
title: not uploaded as primary content; Kaufland assembles the product page from multiple sources
attributes: colour=Schwarz; Schuhgröße=43; Obermaterial=Textil; Gewicht=290 g (per shoe, record); user-defined: Sohle=Gummi
gpsr: manufacturer=ok; responsible_person=n/a; safety_info=ok
gaps (non-blocking): none found against current template (template export not supplied: attributes unconfirmed)
status: offer on existing product
review: confirm Kaufland title length rule before any title upload
```
The Allegro listing is blocked only by the missing upper height; the seller must measure or obtain it from the manufacturer, and the agent does not pick a dictionary value for it. On Kaufland the EAN already exists, so the work is attribute quality, not a new title.

## Quality checklist
- Every title length recounted by code (characters; bytes for Amazon search terms), and matches the displayed count.
- Every limit has a source or a "confirm" note; nothing stated from memory as current.
- Every attribute value traces to a master-record field; no dictionary value chosen that is untrue.
- Catalog-match mode checked per channel.
- GPSR fields mapped or listed as blocking gaps.
- Machine-translated text marked for review.
- Status per channel is consistent with the gap list.

## Sources
- Allegro developer documentation, offer creation with product (title 12–75 characters, 3 words, images 1–16, required parameters, GPSR fields): https://developer.allegro.pl/tutorials/jak-jednym-requestem-wystawic-oferte-powiazana-z-produktem-D7Kj9gw4xFA
- Allegro REST API documentation: https://developer.allegro.pl/documentation
- Kaufland Seller API, managing product data and categories: https://sellerapi.kaufland.com/?page=product-data
- Kaufland Seller API, product data CSV files: https://sellerapi.kaufland.com/?page=product-data-files
- Amazon Seller Central Help, page "Product title requirements" and the category style guides (seller login required; open via Help search in the storefront's Seller Central)
- General Product Safety Regulation (EU) 2023/988, Art. 19 and 22: https://eur-lex.europa.eu/eli/reg/2023/988/oj
- GS1 check digit calculation: https://www.gs1.org/services/how-calculate-check-digit-manually

## License
MIT
