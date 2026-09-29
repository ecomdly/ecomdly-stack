---
name: shopping-search-terms-miner
owner: adsledger
category: Google Shopping
description: Mines the Shopping search terms report for wasted spend and new winners, proposing negatives by match type for a human to apply, never pushing them itself.
version: v3
license: MIT
updated: 2026-09-15
recommended: false
security_checked: true
url: https://ecomdly.com/skills/adsledger/shopping-search-terms-miner
raw: https://ecomdly.com/raw/adsledger/shopping-search-terms-miner.md
install: npx @ecomdly/cli add adsledger/shopping-search-terms-miner
---

# Shopping search terms miner

Shopping has no keywords, so the search terms report is the only place you see what you are paying for. This skill turns it into a short, safe negative list and a list of terms worth their own attention.

## When to use
- Every two weeks on Standard Shopping; monthly on PMax (search term insights are aggregated there).
- After a feed title rewrite, to see which queries changed.

## Input
Search terms report for 30–90 days: search term, campaign, ad group, impressions, clicks, cost, conversions, conversion value. The target ROAS or CPA, and the brand names the store sells.

## Process
1. **Roll up n-grams.** Split terms into 1- and 2-word n-grams and sum cost, clicks, conversions and value per n-gram. A word like "free", "used", "repair", "manual" or a competitor model often wastes more across 300 terms than any single term.
2. **Waste:** n-grams or terms with cost ≥ 2× target CPA and zero conversions, or ROAS < 30% of target with ≥ 50 clicks.
3. **Intent mismatch:** terms asking for something the store does not sell (other size range, rentals, parts, jobs, "diy").
4. **Winners:** terms with ROAS ≥ 1.5× target and ≥ 3 conversions; suggest a priority or campaign split to protect them.

## Negative match type
- Phrase for n-grams (`"repair"`), exact for single bad queries (`[nike air max 90 kids]`), broad only for unambiguous words.
- Check each proposed negative against the converting terms; if it would block any converting term, drop it or narrow it.

## Rules
- Never negate the store's own brand names or the name of a `hero` product.
- Small samples are not waste: under 20 clicks, list as "watch".
- The agent does not add negatives to the account. It produces a list; a human applies it.
- If conversion value is missing, judge by CPA only and say so.

## Output format
```
| negative | match | level | 60d cost | conv. | reason |
| "repair" | phrase | account list | 412.30 | 0 | service intent |
| [running shoes size 50] | exact | campaign | 88.10 | 0 | size not stocked |
Winners: "trail shoes waterproof women" ROAS 7.4 (target 4.0), 9 conv. → high priority campaign.
Watch (under 20 clicks): 37 terms.
```

## License
MIT
