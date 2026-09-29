---
name: pmax-asset-group-reviewer
owner: adsledger
category: Google Shopping
description: Reviews Performance Max asset groups and listing groups for overlap, thin assets and all-products catch-alls, and recommends restructures without editing the account.
version: v2
license: MIT
updated: 2026-09-21
recommended: false
security_checked: true
url: https://ecomdly.com/skills/adsledger/pmax-asset-group-reviewer
raw: https://ecomdly.com/raw/adsledger/pmax-asset-group-reviewer.md
install: npx @ecomdly/cli add adsledger/pmax-asset-group-reviewer
---

# PMax asset group reviewer

Performance Max hides most of its levers, so structure is what you control. This skill checks that each asset group has a clear product set, a matching message and enough assets to run.

## When to use
- After a new PMax launch has run 3–4 weeks.
- When PMax spend grows but Shopping-like ROAS falls.

## Input
Per campaign: budget, bidding (tROAS / max conversion value), target, and brand exclusions. Per asset group: listing group tree, audience signals, final URL expansion setting, asset counts (headlines, long headlines, descriptions, images, logos, videos) and asset-level performance ratings. Product-level performance by asset group for 30 days.

## Checks
1. **Listing groups.** Each asset group should subdivide by one dimension (custom label, product type, brand) and cover a coherent set. Flag an "All products" group sitting next to specific ones in the same campaign; flag products appearing in two campaigns, since only one will serve.
2. **Assets.** Minimum to run well: 5 headlines (up to 15), 1–5 long headlines, 2–5 descriptions (one ≤ 60 chars), landscape and square images, a logo, and a real video (otherwise Google auto-generates one). Flag "Low" rated assets older than 30 days.
3. **Message match.** Headlines should name the product set in the listing group; a generic brand message on a category group is a finding.
4. **Signals.** Customer-list and search-theme signals should differ per group; identical signals everywhere add nothing.
5. **Final URL expansion and brand.** Note whether expansion is on and whether brand terms are excluded; report, do not judge, if the store has no stated policy.
6. **Zombies.** Share of products with zero impressions per group; above 40% suggests a separate zombie campaign.

## Rules
- Read-only. Recommendations only; no asset, listing group or budget change is made.
- Asset-level conversion data is not available in PMax; do not invent it. Use ratings and product-level data.
- Missing exports are listed at the top of the report.

## Output format
```
Campaign "PMax – Footwear" · tROAS 400% · 30d ROAS 3.1
| asset group | products | zero-impr. | headlines | video | finding |
| Trail – m-high | 184 | 22% | 11 | yes | ok |
| All products | 2 040 | 61% | 5 | auto | catch-all overlaps Trail; split by label_1 |
Next: move 1 240 zombie products to a separate low-budget campaign.
```

## License
MIT
