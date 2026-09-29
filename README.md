# ecomdly stack

Every published skill on [ecomdly](https://ecomdly.com) — Markdown playbooks for AI agents, sorted by category. Each file carries a front-matter header (owner, version, license, URLs). Install any of them with `npx @ecomdly/cli add owner/skill`, fetch the raw file, or connect the catalog over MCP (`claude mcp add --transport http ecomdly https://ecomdly.com/mcp`).

This repository is generated from the catalog; edits happen on ecomdly (submit a skill, it goes through review). 30 skills · updated 2026-09-25.

## Categories

- [Analytics & tracking](#analytics-tracking) (4)
- [Catalog content](#catalog-content) (4)
- [Conversion & UX](#conversion-ux) (3)
- [Customer care](#customer-care) (2)
- [E-shop SEO](#e-shop-seo) (3)
- [Email & retention](#email-retention) (3)
- [Google Shopping](#google-shopping) (3)
- [Marketplaces](#marketplaces) (2)
- [Pricing & merchandising](#pricing-merchandising) (2)
- [Product feeds](#product-feeds) (4)

## Analytics & tracking

| skill | description | version | |
|---|---|---|---|
| [shopmetric/ga4-ecommerce-event-auditor](analytics-tracking/ga4-ecommerce-event-auditor.md) | Audits GA4 ecommerce events against the recommended schema — required params, items[] fields, currency and transaction_id dedupe — and reports gaps without editing tags. | v2 | 🛡 |
| [shopmetric/ga4-funnel-analyst](analytics-tracking/ga4-funnel-analyst.md) | Reads a GA4 export and finds where the checkout funnel leaks — drop-off by step, device and source — with the one fix to try first. | v3 | ★ 🛡 |
| [shopmetric/revenue-discrepancy-reconciler](analytics-tracking/revenue-discrepancy-reconciler.md) | Explains why backend, GA4 and Google Ads revenue differ with a bridge of causes — tax, shipping, refunds, consent, attribution, time zones — from supplied numbers only. | v2 | 🛡 |
| [shopmetric/weekly-store-kpi-report](analytics-tracking/weekly-store-kpi-report.md) | Builds a one-page weekly store report — revenue, orders, AOV, conversion rate, ad spend, MER and returns — against last week and last year, stating any missing source. | v2 | 🛡 |

## Catalog content

| skill | description | version | |
|---|---|---|---|
| [tonecheck/catalog-translation-localizer](catalog-content/catalog-translation-localizer.md) | Translates thousands of SKUs in resumable batches with a glossary and do-not-translate list, localises units, sizes and currency, then runs a QA pass before import. | v3 | ★ 🛡 |
| [rankcraft/category-page-copy-writer](catalog-content/category-page-copy-writer.md) | Writes category page intros and buying-guide blocks from real product data and search queries — short above the grid, useful below it, no keyword stuffing. | v2 | 🛡 |
| [cartlift/product-attribute-extractor](catalog-content/product-attribute-extractor.md) | Pulls structured attributes (color, size, material, GTIN, dimensions) out of messy titles and descriptions into feed-ready columns, with a source and confidence per value. | v2 | 🛡 |
| [tonecheck/product-description-writer](catalog-content/product-description-writer.md) | Writes product page descriptions from the spec sheet in the store's voice — benefit first, facts only from the data, missing specs flagged instead of guessed. | v2 | 🛡 |

## Conversion & UX

| skill | description | version | |
|---|---|---|---|
| [checkoutlab/ab-test-readout](conversion-ux/ab-test-readout.md) | Reads out an A/B test honestly: SRM check, sample size against the plan, significance and intervals, guardrail metrics and novelty effect before any "winner" call. | v3 | 🛡 |
| [checkoutlab/checkout-friction-audit](conversion-ux/checkout-friction-audit.md) | Walks the checkout step by step against known friction points — forced accounts, late shipping costs, surplus form fields — and ranks fixes by impact and effort. | v2 | 🛡 |
| [checkoutlab/product-page-cro-review](conversion-ux/product-page-cro-review.md) | Reviews a product detail page element by element — images, price, variants, delivery, returns, reviews — and writes testable fixes, never invented product claims. | v2 | 🛡 |

## Customer care

| skill | description | version | |
|---|---|---|---|
| [helpdeskly/order-status-reply-drafter](customer-care/order-status-reply-drafter.md) | Drafts "where is my order" replies from real order and carrier tracking data, states delays plainly, and never promises a delivery date the data does not support. | v2 | 🛡 |
| [helpdeskly/returns-reason-analyzer](customer-care/returns-reason-analyzer.md) | Codes return reasons from free text into root causes (size/fit, damaged, not as described) and links each cluster to a concrete product page or packing fix. | v2 | 🛡 |

## E-shop SEO

| skill | description | version | |
|---|---|---|---|
| [rankcraft/faceted-navigation-seo-audit](e-shop-seo/faceted-navigation-seo-audit.md) | Audits filter URLs on category pages: which facets deserve an indexable page, which get noindex or a canonical, and which parameter combinations are crawl traps. | v2 | 🛡 |
| [rankcraft/product-schema-validator](e-shop-seo/product-schema-validator.md) | Validates Product, Offer, AggregateRating, shipping and return-policy JSON-LD against Google rich result requirements and the visible page, and reports every mismatch. | v2 | ★ 🛡 |
| [rankcraft/search-console-auditor](e-shop-seo/search-console-auditor.md) | Audits Google Search Console data — pages losing clicks, queries with impressions but no CTR, and product pages cannibalizing each other. | v2 | 🛡 |

## Email & retention

| skill | description | version | |
|---|---|---|---|
| [inboxcart/abandoned-cart-sequence](email-retention/abandoned-cart-sequence.md) | Drafts a three-mail abandoned-cart sequence from the cart contents and the store's voice — useful, specific, and honest about discounts. | v2 | 🛡 |
| [inboxcart/post-purchase-review-request](email-retention/post-purchase-review-request.md) | Drafts review-request mails timed after delivery, not purchase, asking every customer the same way — no incentives for positive reviews, no review gating. | v2 | ★ 🛡 |
| [inboxcart/winback-segment-planner](email-retention/winback-segment-planner.md) | Segments lapsed customers by RFM and their own purchase cycle, then plans a win-back sequence per segment with honest offers taken from the store's real policy. | v2 | 🛡 |

## Google Shopping

| skill | description | version | |
|---|---|---|---|
| [marginmath/break-even-roas-calculator](google-shopping/break-even-roas-calculator.md) | Computes break-even and target ROAS from contribution margin after COGS, fees, fulfilment and returns, using only the store's numbers and showing every step. | v3 | ★ 🛡 |
| [adsledger/pmax-asset-group-reviewer](google-shopping/pmax-asset-group-reviewer.md) | Reviews Performance Max asset groups and listing groups for overlap, thin assets and all-products catch-alls, and recommends restructures without editing the account. | v2 | 🛡 |
| [adsledger/shopping-search-terms-miner](google-shopping/shopping-search-terms-miner.md) | Mines the Shopping search terms report for wasted spend and new winners, proposing negatives by match type for a human to apply, never pushing them itself. | v3 | 🛡 |

## Marketplaces

| skill | description | version | |
|---|---|---|---|
| [listwise/comparison-feed-mapper](marketplaces/comparison-feed-mapper.md) | Maps a store catalog to Heureka, Zboží.cz and Idealo XML/CSV feeds with correct elements, categories and delivery days, and reports rows it cannot fill from real data. | v3 | 🛡 |
| [listwise/marketplace-listing-adapter](marketplaces/marketplace-listing-adapter.md) | Adapts one master product record into Amazon, Allegro and Kaufland listings within each title/bullet limit and category attribute set, flagging every missing value. | v2 | 🛡 |

## Pricing & merchandising

| skill | description | version | |
|---|---|---|---|
| [marginmath/clearance-markdown-planner](pricing-merchandising/clearance-markdown-planner.md) | Plans staged markdowns from weeks of cover and sell-through, never below the margin floor you set, with EU 30-day lowest-price references on every reduction. | v2 | ★ 🛡 |
| [marginmath/competitor-price-brief](pricing-merchandising/competitor-price-brief.md) | Compares your prices with competitor data you supply, matched by EAN, and recommends moves that respect your margin floor. Uses only provided or public data. | v2 | 🛡 |

## Product feeds

| skill | description | version | |
|---|---|---|---|
| [adsledger/feed-custom-label-planner](product-feeds/feed-custom-label-planner.md) | Designs custom_label_0–4 so Shopping and PMax can bid by margin, performance and season, with rules drawn only from data the store actually has. | v2 | 🛡 |
| [cartlift/gtin-identifier-auditor](product-feeds/gtin-identifier-auditor.md) | Validates GTIN check digits, brand and MPN coverage and identifier_exists use across a feed, and flags rows it cannot verify instead of filling them in. | v2 | 🛡 |
| [cartlift/merchant-center-disapproval-fixer](product-feeds/merchant-center-disapproval-fixer.md) | Triages Merchant Center disapprovals by reason and impact, maps each to the attribute or page fix, and proposes feed changes only for a human to approve. | v3 | ★ 🛡 |
| [cartlift/product-feed-optimizer](product-feeds/product-feed-optimizer.md) | Rewrites product titles and descriptions for Google Shopping feeds — attributes first, brand rules kept, no keyword stuffing. | v2 | 🛡 |

★ Ecomdly recommended · 🛡 Security checked (automated checks + clean AI safety scan + human review)
