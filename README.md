# ecomdly stack

Every published skill on [ecomdly](https://ecomdly.com) — Markdown playbooks for AI agents, sorted by category. Each file carries a front-matter header (owner, version, license, URLs). Install any of them with `npx @ecomdly/cli add owner/skill`, fetch the raw file, or connect the catalog over MCP (`claude mcp add --transport http ecomdly https://ecomdly.com/mcp`).

This repository is generated from the catalog; edits happen on ecomdly (submit a skill, it goes through review). 48 skills · updated 2026-10-07.

## Categories

- [Agent tools](#agent-tools) (5)
- [Analytics & tracking](#analytics-tracking) (5)
- [Catalog content](#catalog-content) (5)
- [Conversion & UX](#conversion-ux) (5)
- [Customer care](#customer-care) (4)
- [E-shop SEO](#e-shop-seo) (4)
- [Email & retention](#email-retention) (4)
- [Google Shopping](#google-shopping) (4)
- [Marketplaces](#marketplaces) (4)
- [Pricing & merchandising](#pricing-merchandising) (4)
- [Product feeds](#product-feeds) (4)

## Agent tools

| skill | description | version | |
|---|---|---|---|
| [shopmetric/adyen-payments-operator](agent-tools/adyen-payments-operator.md) | Lets an agent work an Adyen merchant account safely: find payments, explain refusals, prepare owner-approved refunds, captures and cancels, triage disputes by deadline, and reconcile settlement reports to orders. | v1 | 🛡 |
| [marginmath/idoklad-api-operator](agent-tools/idoklad-api-operator.md) | Lets an agent read and write iDoklad through API v3 for an online store: unpaid and overdue invoices, payments backed by bank records, Default-template drafts the owner approves, credit notes and accountant exports. | v1 | 🛡 |
| [cartlift/jira-ticket-collaborator](agent-tools/jira-ticket-collaborator.md) | Lets an agent work e-shop Jira tickets like a good colleague: readiness check, one numbered clarifying comment with defaults and an @mention, ADF comments via REST v3, real transitions only, internal-first replies in JSM. | v1 | 🛡 |
| [cartlift/shopify-admin-api-operator](agent-tools/shopify-admin-api-operator.md) | Lets an agent read and change a Shopify store through the GraphQL Admin API safely: pinned version, least-privilege scopes, cost-aware paging and bulk exports, userErrors checks, owner-approved diffs and a rollback log. | v1 | 🛡 |
| [marginmath/stripe-payments-operator](agent-tools/stripe-payments-operator.md) | Lets an agent work a store's Stripe account safely via the Stripe plugin, MCP or REST: find payments, explain declines, triage disputes by due date, reconcile payouts and fees, and prepare owner-approved refunds. | v1 | 🛡 |

## Analytics & tracking

| skill | description | version | |
|---|---|---|---|
| [shopmetric/ga4-ecommerce-event-auditor](analytics-tracking/ga4-ecommerce-event-auditor.md) | Audits GA4 ecommerce events against Google's schema (currency with value, transaction_id, items, value = price x quantity) and matches purchases to backend orders, returning a prioritised fix list for developers. | v3 | 🛡 |
| [shopmetric/ga4-funnel-analyst](analytics-tracking/ga4-funnel-analyst.md) | Builds a session-level GA4 checkout funnel, tests which step worsened against the store's own baseline, localises it by device and channel, and proposes one fix with a way to confirm it. | v4 | ★ 🛡 |
| [shopmetric/revenue-discrepancy-reconciler](analytics-tracking/revenue-discrepancy-reconciler.md) | Reconciles backend, GA4 and Google Ads revenue with an order-level join and a line-by-line bridge (VAT and shipping, time zones, consent, gateway loss, test orders, click vs conversion date) for store owners and analysts. | v3 | 🛡 |
| [launifycorp/weekly-kpi-slides-deck](analytics-tracking/weekly-kpi-slides-deck.md) | You own the weekly rhythm artifact: a 9slide spine (hard cap 14 including appendix) that tells the store's owner, head of growth and performance buyer what moved last week, why, and what gets decided... | v1 | 🛡 |
| [shopmetric/weekly-store-kpi-report](analytics-tracking/weekly-store-kpi-report.md) | Writes a one-page Monday KPI report for store owners: backend revenue, orders, AOV, conversion, MER and refunds versus last week and last year, with noise tests and at most two decisions. | v3 | 🛡 |

## Catalog content

| skill | description | version | |
|---|---|---|---|
| [tonecheck/catalog-translation-localizer](catalog-content/catalog-translation-localizer.md) | Translates a catalog export in resumable batches with a mandatory glossary, protected brand and placeholder tokens, locale number formats and per-row QA flags, so content managers know exactly which rows need a human. | v4 | ★ 🛡 |
| [rankcraft/category-page-copy-writer](catalog-content/category-page-copy-writer.md) | Writes the title, meta, short intro, buying guide and evidence-based FAQ for a store category page from the category's dated product export and Search Console queries, with every number traceable and unsupported claims flagged. | v3 |  |
| [cartlift/product-attribute-extractor](catalog-content/product-attribute-extractor.md) | Extracts colour, size, material, gender and other attributes from titles, variants and descriptions into Merchant Center-ready values, with source, evidence and confidence per value and a review queue for conflicts. | v3 | 🛡 |
| [tonecheck/product-description-writer](catalog-content/product-description-writer.md) | Writes product descriptions and spec lists from supplier data in the store's brand voice, ordering facts by what decides the purchase for each product type, listing missing specs and flagging unsupported or EU-restricted claims. | v3 |  |
| [launifycorp/product-glossary-builder](catalog-content/product-glossary-builder.md) | You own one deliverable: a machinereadable translation glossary extracted from the live catalogue, splitting every recurring string into terms that must be translated consistently, brand and model nam... | v1 | 🛡 |

## Conversion & UX

| skill | description | version | |
|---|---|---|---|
| [checkoutlab/ab-test-readout](conversion-ux/ab-test-readout.md) | Reads out an e-commerce A/B test in the right order: SRM chi-square, pre-registered primary metric, sample size and peeking, effect with confidence interval, guardrails and novelty, then ship, don't ship or inconclusive. | v4 | 🛡 |
| [checkoutlab/checkout-friction-audit](conversion-ux/checkoutlab--checkout-friction-audit.md) | Audits the checkout from cart to withdrawal flow using GA4 step data and a mobile guest walkthrough, ranks friction by severity and reach, and lists EU must-fixes such as the order button and pre-ticked extras. | v3 | 🛡 |
| [launifycorp/checkout-friction-audit](conversion-ux/launifycorp--checkout-friction-audit.md) | You own the fieldlevel teardown of a live checkout: every input the shopper must touch between "proceed to checkout" and "order placed", scored for cost, ranked for removal, merge or autofill, and han... | v1 | 🛡 |
| [checkoutlab/product-page-cro-review](conversion-ux/product-page-cro-review.md) | Reviews a product page template against the shopper's real purchase questions and EU price, review and safety-info rules, and returns ranked test hypotheses with metrics for store owners and CRO specialists. | v3 |  |
| [checkoutlab/site-search-audit](conversion-ux/site-search-audit.md) | Audits on-site product search from GA4 view_search_results data or vendor logs plus a hands-on test set (zero results, no-click queries, diacritics, SKU/EAN, filters, mobile) and ranks fixes by lost revenue. | v1 | 🛡 |

## Customer care

| skill | description | version | |
|---|---|---|---|
| [helpdeskly/order-status-reply-drafter](customer-care/order-status-reply-drafter.md) | Drafts where-is-my-order replies from order and carrier data, with per-parcel status, only carrier-backed dates, and accurate EU delivery, withdrawal and warranty statements for a human agent to send. | v3 |  |
| [helpdeskly/returns-reason-analyzer](customer-care/returns-reason-analyzer.md) | Codes every return to a root cause with evidence, separates withdrawals from warranty and transit damage, flags SKUs with statistically real return problems, and routes each fix to its owner. | v3 | 🛡 |
| [helpdeskly/review-response-drafter](customer-care/review-response-drafter.md) | Triages new reviews from Google Business Profile, Heureka, Trustpilot and the store's site by severity and drafts one-standard public replies plus internal follow-ups, policy-checked against platform rules and EU review law. | v1 | 🛡 |
| [helpdeskly/warranty-claim-handler](customer-care/warranty-claim-handler.md) | Triages consumer claims for faulty goods under the EU legal guarantee (2019/771) with country-correct deadlines and burden of proof, then drafts the acknowledgement, evidence request and decision letter for the owner to approve. | v1 | 🛡 |

## E-shop SEO

| skill | description | version | |
|---|---|---|---|
| [rankcraft/faceted-navigation-seo-audit](e-shop-seo/faceted-navigation-seo-audit.md) | Decides facet by facet which filter URLs on an online store should be indexed, consolidated, noindexed or blocked from crawling, using crawl, log and Search Console data, and lists crawl traps with a safe rollout order. | v3 | 🛡 |
| [rankcraft/product-schema-validator](e-shop-seo/product-schema-validator.md) | Audits a product page's JSON-LD against Google's merchant listing and product snippet rules and against the visible price, stock, ratings and written shipping/return policy, then lists errors, gaps and confirmed fixes. | v3 | ★ 🛡 |
| [rankcraft/search-console-auditor](e-shop-seo/search-console-auditor.md) | Turns a Search Console performance export into three ranked lists for a store: pages losing clicks with the cause split into demand, ranking and CTR, under-clicked queries against the site's own CTR curve, and URLs competing for one query. | v3 | 🛡 |
| [rankcraft/site-migration-redirect-mapper](e-shop-seo/site-migration-redirect-mapper.md) | Builds and checks the 301 map for a store moving platform, URL structure or domain: multi-source URL inventory, ID/SKU/EAN matching before fuzzy, discontinued and filter URLs, chains, canonicals, hreflang and post-launch checks. | v1 | 🛡 |

## Email & retention

| skill | description | version | |
|---|---|---|---|
| [inboxcart/abandoned-cart-sequence](email-retention/abandoned-cart-sequence.md) | Designs a three-mail abandoned cart flow for an ESP with an EU consent check, exit and frequency rules, a margin-tested discount only in the last mail, and a holdout to measure real recovery. | v3 |  |
| [inboxcart/post-purchase-review-request](email-retention/post-purchase-review-request.md) | Plans a post-delivery review request flow that asks every buyer the same way, with timing by product type, no gating, and incentive rules checked against Google, Trustpilot and EU review law. | v3 | ★ 🛡 |
| [inboxcart/welcome-series-builder](email-retention/welcome-series-builder.md) | Designs a welcome flow from signup to first purchase with provable EU consent, double opt-in, exit rules, a margin-tested discount decision, Gmail/Yahoo sender checks and a holdout. Drafts only, never sends. | v1 | 🛡 |
| [inboxcart/winback-segment-planner](email-retention/winback-segment-planner.md) | Segments lapsed customers by their own purchase cycle and net-of-returns RFM, plans a sequence and honest offer per segment with a holdout and break-even check, and sunsets unresponsive contacts. | v3 | 🛡 |

## Google Shopping

| skill | description | version | |
|---|---|---|---|
| [marginmath/break-even-roas-calculator](google-shopping/break-even-roas-calculator.md) | Calculates break-even ROAS, PNO/COS and target ROAS per category and campaign mix from contribution margin (COGS, fees, fulfilment, returns), on the same VAT and shipping basis as your Google Ads conversion value, with all arithmetic shown. | v4 | ★ 🛡 |
| [adsledger/pmax-asset-group-reviewer](google-shopping/pmax-asset-group-reviewer.md) | Reviews Performance Max asset groups for Merchant Center retailers: product ownership and overlaps, unserved products, asset gaps against Google's specs, message match, signals, brand exclusions and Final URL expansion, with proposals for a human to approve. | v3 | 🛡 |
| [adsledger/shopping-budget-allocator](google-shopping/shopping-budget-allocator.md) | Reallocates a fixed monthly budget across Shopping and Performance Max campaigns by margin-based break-even, lost IS to budget vs rank and marginal return on matured data, as stepwise moves for the owner to approve. | v1 | 🛡 |
| [adsledger/shopping-search-terms-miner](google-shopping/shopping-search-terms-miner.md) | Mines Shopping and Performance Max search terms into collision-tested negatives (match type and level), watch terms and winners to protect, using n-grams and a sample-size rule instead of guesswork; for PPC managers. | v4 | 🛡 |

## Marketplaces

| skill | description | version | |
|---|---|---|---|
| [listwise/comparison-feed-mapper](marketplaces/comparison-feed-mapper.md) | Maps a store catalog to Heureka.cz/.sk and Zboží.cz XML and idealo offer data element by element, including DELIVERY_DATE, carrier IDs and prior-price rules, and reports every row that would be rejected or badly paired. For Czech and EU e-shops. | v4 | 🛡 |
| [listwise/marketplace-account-health-triage](marketplaces/marketplace-account-health-triage.md) | Turns Amazon Account Health, Allegro sales-quality and Kaufland performance exports into a risk-ranked fix plan: metric, window, slack in orders, roll-off date, root cause, fix and owner per issue. | v1 | 🛡 |
| [listwise/marketplace-listing-adapter](marketplaces/marketplace-listing-adapter.md) | Turns a master product record into Amazon, Allegro and Kaufland listing drafts: leaf category, title within the channel limit, mapped attributes, GPSR fields and a blocking-gap list, for sellers expanding to marketplaces. | v3 | 🛡 |
| [listwise/marketplace-profitability-check](marketplaces/marketplace-profitability-check.md) | Calculates per-SKU, per-marketplace contribution on Amazon EU, Allegro and Kaufland after commission, fulfilment, storage, ads, returns and FX, on the right VAT basis, and decides keep, reprice, change fulfilment or delist. | v1 | 🛡 |

## Pricing & merchandising

| skill | description | version | |
|---|---|---|---|
| [marginmath/clearance-markdown-planner](pricing-merchandising/clearance-markdown-planner.md) | Plans SKU-level clearance markdowns from stock, sell rate and exit date, stops at a VAT-correct floor, and shows the EU Omnibus 30-day prior price and correctly based discount percentage for every step, including progressive-markdown rules. | v3 | ★ 🛡 |
| [marginmath/competitor-price-brief](pricing-merchandising/competitor-price-brief.md) | Compares your offers with competitor prices you supply, matched on GTIN and on total price with shipping, flags where you are out of the market or leaving margin, and suggests the smallest price move above your VAT-correct floor. | v3 | 🛡 |
| [marginmath/free-shipping-threshold-planner](pricing-merchandising/free-shipping-threshold-planner.md) | Sets or reviews a free-shipping threshold and shipping prices from your own basket distribution, carrier cost per parcel, contribution margin and bunching below the line, with profit per order and a test readout, no invented lift. | v1 | 🛡 |
| [marginmath/promotion-profit-check](pricing-merchandising/promotion-profit-check.md) | Decides before launch whether a sale, coupon or bundle pays: break-even uplift from contribution margin, pull-forward, cannibalisation, coupon leakage, ad-target impact, EU 30-day prior price check and a baseline readout. | v1 | 🛡 |

## Product feeds

| skill | description | version | |
|---|---|---|---|
| [adsledger/feed-custom-label-planner](product-feeds/feed-custom-label-planner.md) | Plans custom_label_0-4 from your own margins, VAT-correct break-even ROAS and sample-safe performance tiers, and outputs a mapping file for a supplemental feed plus the Shopping or PMax structure each label drives. For PPC specialists. | v3 | 🛡 |
| [cartlift/gtin-identifier-auditor](product-feeds/gtin-identifier-auditor.md) | Audits every GTIN, MPN, brand and identifier_exists value in a product feed by GS1 rule (length, check digit, restricted prefixes, variant uniqueness) and lists the fix per row, without ever inventing a number. For feed and catalog managers. | v3 | 🛡 |
| [cartlift/merchant-center-disapproval-fixer](product-feeds/merchant-center-disapproval-fixer.md) | Turns a Google Merchant Center issue export into a fix plan sorted by lost revenue: root cause per issue (feed, site, account setting or policy), owner, and whether a review is needed. For store owners and PPC specialists. | v4 | ★  |
| [cartlift/product-feed-optimizer](product-feeds/product-feed-optimizer.md) | Rewrites Google Shopping feed titles and descriptions from verified attributes, policy-clean and marked as AI-generated via structured_title, and proposes missing attribute fills in a reviewable CSV. For store owners and feed managers. | v3 | 🛡 |

★ Ecomdly recommended · 🛡 Security checked (automated checks + clean AI safety scan + human review)
