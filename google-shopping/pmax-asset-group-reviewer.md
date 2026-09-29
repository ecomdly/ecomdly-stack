---
name: pmax-asset-group-reviewer
owner: adsledger
category: Google Shopping
description: Reviews Performance Max asset groups for Merchant Center retailers: product ownership and overlaps, unserved products, asset gaps against Google's specs, message match, signals, brand exclusions and Final URL expansion, with proposals for a human to approve.
version: v3
license: MIT
updated: 2026-09-29
recommended: false
security_checked: true
url: https://ecomdly.com/skills/adsledger/pmax-asset-group-reviewer
raw: https://ecomdly.com/raw/adsledger/pmax-asset-group-reviewer.md
install: npx @ecomdly/cli add adsledger/pmax-asset-group-reviewer
---

# PMax asset group reviewer

Produces a per-campaign review of Performance Max asset groups for a retailer with a Merchant Center feed: which products each asset group owns, whether creative and signals match that product set, which eligible products never get served, and where the controls (brand exclusions, negatives, Final URL expansion) contradict the store's intent. The naive review ranks asset groups by ROAS and pauses the worst. Google itself says lower-ROAS asset groups still contribute to campaign goals and that asset-level ratios are directional only; the levers you actually control in PMax are structure (who owns which products) and inputs (feed, assets, signals). This skill reviews those and leaves targets to `break-even-roas-calculator`.

## When to use

- A PMax campaign has at least 3–4 weeks of data after launch or after its last structural change, plus one full conversion lag cycle.
- PMax spend grows but blended ROAS or product coverage falls.
- Before splitting or merging asset groups or campaigns, or after a feed restructure (new `product_type`, new custom labels).
- Standard Shopping and PMax carry the same products and the owner wants to know which one serves.

## When not to use

- Lead-gen or app PMax without a product feed: the listing group checks do not apply.
- Setting tROAS values: `break-even-roas-calculator`.
- Mining queries into negatives: `shopping-search-terms-miner`. This skill only checks that brand and negative controls exist and are coherent.
- Disapproved products: `merchant-center-disapproval-fixer`. Disapproved items cannot serve whatever the listing groups say.

## Inputs

Required. If any is missing, list it at the top of the report and skip the sections that depend on it; never estimate.

1. **Campaign settings** (Campaign > Settings): budget, bid strategy and target, conversion goals, brand exclusions and whether Shopping ads may still serve on excluded brands, campaign negatives and attached lists, Final URL expansion on/off, URL exclusions, page feeds.
2. **Asset groups** (Asset groups > Table view, Listing groups): name, status, listing group tree (attribute, value, included/excluded), search themes, audience signals, final URL, Ad strength.
3. **Asset report** (Asset group > View asset details): type, text or dimensions, "Added by" (advertiser or Google AI), status, impressions, clicks, cost, conversions, conversion value.
4. **Product performance by campaign and asset group**, 30 days minimum: item ID, `item_group_id`, product type, custom labels, impressions, clicks, cost, conversions, conversion value (Products view with asset group column, or a report on `segments.product_item_id`).
5. **Merchant Center status**: item ID, approval status, availability. Separates "not served" from "not eligible".

Optional: the PMax search terms report ("Search terms and landing pages for Performance Max", Ad format segment; data from March 2023 on); the list of Standard Shopping and Search campaigns with their product scope and exact keywords; the store's written brand-traffic and landing-page policy; a margin tier label.

## Best practices

### Structure: who owns which products

1. **One product, one owner.** When several campaigns in an account could serve the same impression, Google serves the one with the highest Ad Rank; the only exception is a Search keyword that exactly matches (or is spell-corrected to) the query, which beats PMax. PMax is no longer automatically preferred over Standard Shopping for the same products. Overlap therefore splits data between structures and hides which one works. Flag every item eligible in two campaigns, and every item in a catch-all "All products" group that sits next to specific groups in the same campaign.
2. **Split by one business dimension, ideally a custom label.** Listing groups can subdivide by Google product category, brand, item ID, condition, product type (up to 5 levels), channel, and custom labels 0–4. Google caps an asset group at 1,000 listing groups, says a large number is not a best practice, and recommends grouping with custom labels. Labels also survive title and category rewrites. Label design: `feed-custom-label-planner`.
3. **Category subdivision is country-limited.** Google lists the target countries where category subdivision works (US, UK, AU, DE, FR, JP, IT, NL, BR, NO, SE, TR when checked). A Czech or Slovak campaign must use `product_type` or custom labels.
4. **Campaign split versus asset group split.** Bidding targets and budgets are campaign-level. Separate campaigns only when products need a different target or budget (for example a low-margin tier). Separate asset groups when they need different creative, landing page or signals. Every extra campaign thins conversion data: state the monthly conversion count of each proposed campaign so the owner sees the cost of the split.
5. **Zero-impression products are a structure finding only after eligibility is checked.** Remove disapproved, out-of-stock and excluded items first. Then report, per asset group, eligible items with zero impressions ÷ eligible items. Use the store's threshold; without one, use 40% as a house heuristic (labelled as such, not a Google benchmark) to propose a separate campaign with its own budget for those items. Measure by `item_group_id` for variant-heavy catalogs, since one variant usually represents the group.

### Assets: enough to render every format

6. **Compare with Google's current specs** (re-check the specs page on the day; limits change):
   - Headlines 3–15, max 30 characters, at least one of 15 or fewer; Google recommends 11+.
   - Long headlines 1–5, max 90; recommended 2+. Descriptions 2–5, max 90; recommended 4+.
   - Business name (max 25), final URL, call to action.
   - Images: landscape 1.91:1 and square 1:1 required, 4+ of each recommended; portrait 4:5 optional; up to 20 images.
   - Logos: square required, landscape 4:1 optional; up to 5.
   - Video: optional; without one Google may auto-generate videos from the other assets. Google recommends uploading at least one (10 seconds or longer).
   Report the gap per type, not just the Ad strength label.
7. **Feed-only asset groups can be deliberate.** PMax can launch without assets, and some retailers run feed-only groups to stay close to Shopping inventory. If the store says so, report it as a choice with its consequence (auto-generated or no creative elsewhere), not as an error.
8. **Asset-level metrics are directional.** PMax reports impressions, clicks, cost, conversions and value per asset, but one ad impression counts for every asset in it, so asset rows do not sum to the group, and per-asset CTR, CPA and ROAS depend on the assets served alongside. Google's order: fill every slot towards "Excellent" Ad strength first, then replace. Propose replacing an asset only when it has substantial impressions and trails assets of the same type in the same group, and always include the replacement draft.
9. **Review what Google wrote.** "Added by" shows assets created or enhanced by Google AI (text customization, Final URL expansion headlines). Check them for claims the store cannot support (free shipping, prices, "best") and brand voice; they serve under the store's name.

### Message match

10. **Creative names the product set.** A "Trail running shoes" group with generic brand headlines wastes the only place creative can be specific. Several headlines and at least one long headline should name the category, key attribute or offer, and the final URL should be the matching category page, not the home page.
11. **Offers must be true for every product in the group.** "Free shipping" on items below the free-shipping threshold, or "−30%" when only some items are reduced, is a policy and consumer-law risk. EU price-reduction claims need the 30-day prior price (see `clearance-markdown-planner`).

### Signals and controls

12. **Signals and themes must add information.** Audience signals are hints, not targeting; identical signals in every group add nothing, so prefer matching customer lists (past buyers of that category). Search themes: Google recommends themes the site does not already convey, no duplicates or close variants, up to 50. Themes rank like phrase or broad keywords, so a theme identical to a Search exact keyword will lose that traffic to Search anyway.
13. **Brand traffic: decide, then check the control.** PMax brand exclusions apply to Search and Shopping ads, cover misspellings and variants, and can optionally let Shopping ads keep serving on excluded brands. Google recommends them over negatives for brand. If a separate brand Search campaign exists and PMax has no exclusion, PMax ROAS includes brand demand and looks better than it is. With no stated policy, report both options; do not choose.
14. **Negatives reach Search and Shopping inventory only.** Campaign-level PMax negatives and the account-level list (1,000 terms per account, applies to Search and Shopping inventory of all campaign types) do not stop Display or YouTube placements; those need excluded content keywords. Check that no negative blocks the store's own category terms.
15. **Final URL expansion is campaign-level.** On: ads may land on any page of the domain with generated headlines, restricted by URL exclusions (available only when on), URL-contains rules and page feeds. Off: the final URL plus product URLs from the listing groups. If the landing page report shows blog, careers or legal pages, propose URL exclusions.

## Process

1. **Inventory inputs.** List present and missing. Without product data by asset group, report only assets and settings.
2. **Ownership map.** For each item ID, list every campaign and asset group whose tree includes it. Mark `overlap-campaign`, `overlap-catchall`, and `also-in-standard-shopping`.
3. **Eligibility.** Join Merchant Center status. Eligible = approved and in stock. Zero-impression rate = eligible items with 0 impressions ÷ eligible items.
4. **Concentration.** Per group, share of conversion value from the top 10% of items by value. High concentration plus many unserved items argues for a split, not for more assets.
5. **Assets.** Count per type against the spec list; flag missing required types, counts below recommendation, no headline of 15 characters or fewer, no uploaded video, AI-added assets with unsupported claims.
6. **Message match.** Compare each group's dominant product type or label with its headlines, long headlines and final URL.
7. **Signals and themes.** Flag identical sets across groups and themes that duplicate Search exact keywords.
8. **Controls.** Brand exclusions against the policy; negatives against converting terms (if the search terms report is provided); Final URL expansion against landing pages.
9. **Decide** each finding:
   - Overlap between campaigns → one owner (the campaign whose target and budget fit), exclude the item elsewhere.
   - Catch-all next to specific groups → exclude the specific nodes from the catch-all.
   - Zero-impression rate above threshold on eligible items → separate campaign or group with its own budget.
   - Missing required assets, not deliberately feed-only → drafts of the missing assets.
   - Weak ROAS with clean structure and full assets → no group change; refer the target to `break-even-roas-calculator`.
10. **Report** in the output format, every proposal marked for approval.

## Pitfalls and edge cases

- **Listing group metrics differ from campaign metrics.** When one ad shows several products, each product records an impression; the campaign records one. Do not reconcile them to the unit.
- **Conversion lag.** Recent days under-report value. Exclude the last lag window or say it is included.
- **Structure changes reset comparisons.** Compare only periods with the same listing groups and targets.
- **Seasonal items.** Off-season zero impressions are not a structure problem; check seasonality labels and availability first.
- **Private-label brands.** Brand exclusions can remove queries for the store's own product brand too. Check which brands are excluded and whether Shopping is allowed on them before concluding brand traffic is gone.

## Rules

- Read-only. The agent does not create, edit, pause or remove campaigns, asset groups, listing groups, assets, negatives or budgets. Every change is a proposal for a human to approve and apply.
- Never invent metrics, counts or asset data. A missing export produces "not assessed: <export> missing".
- No benchmarks presented as facts. House heuristics are labelled; the store's thresholds override them.
- Draft asset text must be true for every product in the group and must not state prices, discounts or shipping terms the store has not confirmed.
- Brand and landing-page policy is the owner's decision; report the trade-off.

## Output format

```
PMax asset group review — <account> — <date range> (last <n> days: excluded|included)
Missing inputs: <list or "none">

Campaign "<name>" · <bid strategy> <target> · cost <x> · conv. value <y> · ROAS <y/x>
Brand exclusions: <on (list)|off> · Shopping on excluded brands: <yes|no|n/a> · Brand policy: <stated|not stated>
Final URL expansion: <on|off> · URL exclusions: <n> · Non-commercial landing pages: <list|none>
Negatives: campaign <n> · lists <names> · account-level <n>

| asset group | owns (rule) | eligible items | zero-impr. | top-10% value share | H/LH/D | img L/S/P | logo | video | Ad strength | finding |
|---|---|---|---|---|---|---|---|---|---|---|

Overlaps
- <n> items in "<A>" and "<B>" → proposed owner <A> · reason <...>

Proposals (for approval)
1. [structure|assets|message|signals|controls] <finding> → <proposal> · evidence: <numbers>

Not assessed: <section — reason>
```

## Worked example

Illustrative numbers only.

```
PMax asset group review — outdoor-shop.cz — 2026-08-01..2026-08-31 (last 5 days: excluded)
Missing inputs: none

Campaign "PMax – Footwear" · Max conv. value, tROAS 400% · cost 48 200 Kč · conv. value 149 400 Kč · ROAS 3.10
Brand exclusions: off · Shopping on excluded brands: n/a · Brand policy: not stated
Final URL expansion: on · URL exclusions: 0 · Non-commercial landing pages: /blog/ (2.1% of clicks)
Negatives: campaign 12 · lists none · account-level 40

| asset group | owns (rule) | eligible items | zero-impr. | top-10% value share | H/LH/D | img L/S/P | logo | video | Ad strength | finding |
|---|---|---|---|---|---|---|---|---|---|---|
| Trail – high margin | custom_label_1 = high | 184 | 22% | 58% | 11/3/4 | 5/5/2 | 1 | uploaded | Excellent | ok |
| All products | everything | 1 856 | 61% | 91% | 5/1/2 | 1/1/0 | 1 | auto | Poor | overlaps Trail; thin assets |

Overlaps
- 184 items in "Trail – high margin" and "All products" → exclude custom_label_1 = high from "All products".
- 96 items also in Standard Shopping "Shoes – brand X" → proposed owner PMax · reason: the Standard campaign had 0 conversions on them in 30 days.

Proposals (for approval)
1. [structure] 1 132 of 1 856 eligible items in "All products" had zero impressions (61%; store has no threshold, 40% house heuristic) → separate campaign with its own budget for them.
2. [assets] "All products": 5 headlines (11+ recommended), none ≤ 15 characters; 1 landscape and 1 square image (4+ each recommended); auto-generated video only → drafts attached.
3. [controls] Brand exclusions off, no brand policy → owner decides: keep brand in PMax, or exclude it and cover brand in Search.
4. [controls] 2.1% of clicks landed on /blog/ → URL exclusion "URL contains /blog".
```

## Quality checklist

- Missing inputs listed at the top; no section silently skipped.
- Asset counts checked against the specs page on the review date.
- Zero-impression rate computed on eligible items only.
- Each overlap names the item count, both owners and one proposed owner.
- No asset group proposed for pausing on ROAS alone.
- Every proposal carries its evidence and is marked for approval.
- Heuristics labelled; no invented benchmark; draft text has no unconfirmed price or shipping claim.

## Sources

- How asset groups work (asset specs): https://support.google.com/google-ads/answer/10724748
- Performance Max specs: https://support.google.com/google-ads/answer/17091269
- Manage a Performance Max campaign with listing groups: https://support.google.com/google-ads/answer/11596074
- About asset group reporting for Performance Max: https://support.google.com/google-ads/answer/13872527
- About asset-level metrics: https://support.google.com/google-ads/answer/16259414
- How Performance Max interacts with other campaigns: https://support.google.com/google-ads/answer/13810170
- New Performance Max features (PMax and Standard Shopping by Ad Rank): https://support.google.com/google-ads/answer/15535462
- About ad group and asset group prioritization: https://support.google.com/google-ads/answer/2756257
- About search themes: https://support.google.com/google-ads/answer/16669486
- About brand exclusions: https://support.google.com/google-ads/answer/16669487
- Negative keywords in Performance Max campaigns: https://support.google.com/google-ads/answer/15726455
- About account-level negative keywords: https://support.google.com/google-ads/answer/11396330
- About Final URL expansion: https://support.google.com/google-ads/answer/16672777
- About the search terms report in Performance Max: https://support.google.com/google-ads/answer/16327396

## License
MIT
