---
name: competitor-price-brief
owner: marginmath
category: Pricing & merchandising
description: Compares your offers with competitor prices you supply, matched on GTIN and on total price with shipping, flags where you are out of the market or leaving margin, and suggests the smallest price move above your VAT-correct floor.
version: v3
license: MIT
updated: 2026-09-29
recommended: false
security_checked: true
url: https://ecomdly.com/skills/marginmath/competitor-price-brief
raw: https://ecomdly.com/raw/marginmath/competitor-price-brief.md
install: npx @ecomdly/cli add marginmath/competitor-price-brief
---

# Competitor price brief

Produces a per-SKU brief that puts the store's offers next to competitor prices the user supplies: where the store is out of the market, where it gives margin away by being far cheaper than it needs to be, and what price change (if any) is worth making, never below the store's floor. The naive approach compares item prices on product names and recommends matching the cheapest offer. That fails three ways: names match the wrong variant or pack size, the shopper compares total price including shipping (comparison sites like Heureka and Zboží.cz and Google Shopping all show it), and the cheapest offer is often stale, out of stock, or below cost for the store. Price gaps matter where shoppers compare; this skill measures them on identical products, on total price, against fresh in-stock offers.

## When to use

- Weekly, on a competitor export from a price-monitoring tool, a comparison site's shop listing, or a spreadsheet the user keeps.
- Before a promotion, to see which SKUs need a price move and which are already competitive.
- When Shopping or comparison-site traffic drops on specific products and price is a suspect.
- With Merchant Center's price benchmark data, to prioritise which GTINs to research further.

## When not to use

- Planning markdowns for slow stock: `clearance-markdown-planner` (it applies the EU prior-price rule to announced discounts).
- Computing ad targets from margin: `break-even-roas-calculator`.
- Collecting competitor data: this skill does not scrape, log in, or bypass anything. It works on data the user provides or public pages the user points to.

## Inputs

Required:

1. **Own offers**, per SKU: GTIN/EAN, SKU, title, variant attributes (size, colour, pack size), price incl. VAT, cheapest shipping price to the reference location, free-shipping threshold, stock, landed cost ex VAT, VAT rate, margin floor (as % of ex-VAT price) or a fixed minimum price.
2. **Competitor observations**: shop name, GTIN/EAN (or product URL/name if no GTIN), price incl. VAT, cheapest shipping price, availability as shown ("in stock", "within 3 days", "out of stock"), pack size if different, observation date and time, and the source (tool name, comparison site, shop page).

Optional: Merchant Center Analytics > Pricing export (benchmark price and price gap per GTIN); clicks or sales per SKU for the last 28 days (to rank by importance); the store's price-ending convention (…9, …90); pricing policy per brand (MAP or distributor agreements the store must respect).

If landed cost or floor is missing for a SKU, classify it but give no price suggestion, and list it. If the observation date is missing for a competitor row, drop the row.

## Best practices

### Compare like with like

1. **Match on GTIN.** GTIN identifies the exact product. Name matching confuses variants, colours and pack sizes. Allow name matching only when the user permits it, mark those rows `name-match`, and keep them out of automatic suggestions. Missing or wrong GTINs in the own catalog are a separate problem: `gtin-identifier-auditor`.
2. **Normalise pack size to unit price.** A 3-pack against a 5-pack is a unit-price comparison (price ÷ units), not a price comparison. For goods sold by weight or volume, compare the unit price per kg/l, which the EU Price Indication Directive already requires shops to display.
3. **Compare total price.** Total = item price + cheapest shipping to the reference location (the method most customers would choose, e.g. pickup point). Use free shipping only if the single item reaches the free-shipping threshold; a threshold of 1 500 does not help a 449 item. If the user compares item price only, state it in the header.
4. **Freshness and availability decide which offers count.** Default: drop competitor rows older than 7 days (shorter for volatile categories; ask), and drop rows marked out of stock. Keep "ships in N days" offers but mark them; an in-stock offer and a two-week backorder are not the same market.
5. **Look at the distribution, not just the minimum.** Compute the gap to the cheapest valid offer and to the median. A single outlier (clearance, a marketplace seller with no reviews, a pricing error) should not set the store's price. If the cheapest offer is more than a chosen distance below the second cheapest, report it as a possible outlier.

### Decide with margin in view

6. **Floor price must include VAT correctly.** Margin floors are defined on the ex-VAT price: floor ex VAT = landed cost ÷ (1 − floor margin); floor incl VAT = floor ex VAT × (1 + VAT rate). Forgetting the VAT step sets a floor 1 ÷ (1 + v) too low (about 17% at 21% VAT). If the store sets floors on contribution, use `break-even-roas-calculator` for the per-unit variable costs.
7. **Classify with a band, not a point.** House defaults (the store can change them):
   - `out of market`: total more than 5% above the cheapest valid offer and above the median.
   - `leaving money`: cheapest by more than 5% versus the next valid offer.
   - `in band`: everything else.
   - `no data`: no valid competitor row.
   Report the numbers, not only the label.
8. **Suggest the smallest move that changes the class.** For `out of market`: target total = min(median, cheapest × (1 + band)), then item price = target total − own shipping, rounded down to the store's price ending. For `leaving money`: target total just under the next offer (default: 2% below), rounded down. If the target item price is below the floor, suggest the floor price and state that the SKU cannot reach band at the floor, or suggest no change.
9. **Not every gap is actionable.** Rank suggestions by importance (clicks or sales in the last 28 days). A 12% gap on a product nobody views is not worth a price change; a 3% gap on the top seller may be.
10. **Price increases have a later consequence in the EU.** If the store raises a `leaving money` price and later announces a discount, the prior price it must show is the lowest price of the last 30 days (Article 6a of Directive 98/6/EC). A raise shortly before a planned sale is therefore not usable as the "was" price. Flag SKUs that are in an upcoming promotion.
11. **Comparative claims are regulated.** Statements such as "cheaper than Shop X" are comparative advertising (Directive 2006/114/EC) and fall under unfair commercial practices rules; they are outside the prior-price rule but must be verifiable and not misleading. This skill's output is internal; it does not produce public comparisons.
12. **Merchant Center benchmark data is a prioritiser, not a competitor list.** Google's benchmark is an average price, from all retailers selling the same GTIN on Google, that typically leads to more successful auctions; it requires a valid GTIN. It does not name competitors. Merchant Center terms restrict pricing data to the retailer's internal use: it must not be resold, publicly displayed or aggregated across businesses. Use it to choose which SKUs to check against named competitors.
13. **Contracts override the brief.** If a brand has a minimum advertised price or a distributor pricing agreement, mark the SKU `policy-bound` and give no suggestion below it.

## Process

1. **Validate own data.** Every SKU needs GTIN, price, shipping, stock; landed cost and floor for suggestions. List gaps.
2. **Clean competitor rows.** Drop rows without date, older than the freshness limit, or out of stock. Normalise pack sizes to unit prices. Mark `name-match` and backorder rows.
3. **Compute totals.** Own total and each competitor total = item price + cheapest shipping (free only if the item alone reaches the threshold).
4. **Per SKU:** cheapest valid total, second cheapest, median; gap to cheapest = own total ÷ cheapest − 1; gap to median = own total ÷ median − 1; outlier flag if cheapest is far below the second.
5. **Classify** by rule 7.
6. **Floor:** floor incl VAT = landed cost ÷ (1 − floor margin) × (1 + v).
7. **Suggest** by rule 8; check floor, policy-bound status, and upcoming promotions; compute margin at the suggested price = (price ÷ (1 + v) − landed cost) ÷ (price ÷ (1 + v)).
8. **Rank** by clicks or sales; cut the list to what a human can act on this week.
9. **Report** with observation dates and the shipping used for each comparison.

## Pitfalls and edge cases

- **Marketplace offers.** A comparison-site listing may show a marketplace seller, not the brand's shop. Record the seller name as shown; do not merge sellers.
- **Price incl. vs ex VAT.** B2B shops show ex-VAT prices. Normalise before comparing, and never mix.
- **Cross-border offers.** A competitor shipping from another country may show a lower price with long delivery and different return terms. Mark by country; compare within the same delivery promise.
- **Bundles and gifts.** "Free socks with shoes" changes the offer; note it, do not price it.
- **Currency.** Convert with one stated rate and date.
- **Rounding.** Round suggested prices only at the end and only down to the store's price ending, so the class change still holds.
- **Frequent repricing.** If the store changes prices often, feed and landing page prices must stay in sync for Shopping; stale feed prices cause product disapprovals (`merchant-center-disapproval-fixer`).

## Rules

- Work only with data the user provides or public pages the user names. Never log into competitor accounts, bypass paywalls or bot protection, or pull prices from behind a login.
- Never infer a price for a competitor without a row. No row means `no data`.
- Every competitor price in the output carries its observation date and source.
- Always state the shipping cost used in each comparison.
- Never propose a price below the floor or a contractual minimum.
- The agent does not change prices in the store or feed. Suggestions are for a human to approve.
- Merchant Center pricing data stays internal to the retailer.

## Output format

```
Competitor price brief — <store> — observations <date range> — prepared <date>
Basis: total price incl. VAT, cheapest shipping to <location> | freshness ≤ <n> days | band ±<b>%
Excluded rows: <n> stale, <n> out of stock, <n> without date · Missing own data: <list or none>

<EAN> <title> [<pack>] · clicks 28d <n>
  you <price> + <ship> = <total> | cheapest <shop> <price> + <ship> = <total> (<date>, <source>) | 2nd <total> | median <total> (<n> offers)
  status: <out of market|leaving money|in band|no data|policy-bound> (<gap to cheapest>%, <gap to median>%)
  suggest: <price> → total <total> | floor <floor> | margin now <x>% → <y>% | note: <outlier|name-match|promo in <n> days|none>

Summary: <n> out of market, <n> leaving money, <n> in band, <n> no data
Top actions (ranked by clicks): 1. ... 2. ...
```

## Worked example

Illustrative numbers only. Store in Czechia, VAT 21%, shipping 79 Kč to a pickup point, free shipping from 1 500 Kč, band 5%, price ending …9, floor margin 25%.

```
Competitor price brief — outdoor-shop.cz — observations 2026-09-19..2026-09-24 — prepared 2026-09-25
Basis: total price incl. VAT, cheapest shipping to pickup point | freshness ≤ 7 days | band ±5%
Excluded rows: 3 stale, 2 out of stock, 0 without date · Missing own data: none

8594012345678 Merino socks [3-pack] · clicks 28d 640
  you 449 + 79 = 528 | cheapest ShopA 399 + 69 = 468 (2026-09-21, Heureka listing) | 2nd 510 | median 510 (3 offers)
  status: out of market (+12.8% vs cheapest, +3.5% vs median)
  target total min(510, 468 × 1.05 = 491) = 491 → item 412 → rounded 409
  suggest: 409 → total 488 | floor 323 (200 ÷ 0.75 × 1.21) | margin now 46.1% → 40.8% | note: none

8594012349999 Trail bottle 750 ml · clicks 28d 310
  you 289 + 79 = 368 | next ShopB 349 + 79 = 428 (2026-09-22, shop page) | median 428 (2 offers)
  status: leaving money (next offer +16.3% above you)
  target total 428 × 0.98 = 419 → item 340 → rounded 339
  suggest: 339 → total 418 | floor 226 (140 ÷ 0.75 × 1.21) | margin now 41.4% → 50.0% | note: in "Autumn sale" in 12 days: raising now makes 289 the prior price for that sale

Summary: 1 out of market, 1 leaving money, 0 in band, 0 no data
Top actions: 1. Merino socks → 409 (highest clicks). 2. Trail bottle: raise only if the Autumn sale is dropped or re-planned.
```

Margin checks: socks (landed cost 200) at 409: 409 ÷ 1.21 = 338.02 ex VAT; (338.02 − 200) ÷ 338.02 = 40.8%. Bottle (landed cost 140) at 339: 339 ÷ 1.21 = 280.17; (280.17 − 140) ÷ 280.17 = 50.0%.

## Quality checklist

- Only GTIN-matched rows drive suggestions; name matches are labelled.
- Every competitor price has date and source; stale and out-of-stock rows excluded and counted.
- Totals use the cheapest shipping and respect free-shipping thresholds.
- Floor computed on ex-VAT margin and converted to incl VAT.
- No suggestion below floor or contractual minimum; margin before and after shown.
- Promotions within 30 days flagged for the prior-price consequence.
- Actions ranked by clicks or sales, not by gap size alone.

## Sources

- Directive 98/6/EC (Price Indication Directive), consolidated, incl. Article 6a: https://eur-lex.europa.eu/eli/dir/1998/6/oj
- Commission guidance on Article 6a (2021/C 526/02): https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX:52021XC1229(06)
- Directive 2006/114/EC on misleading and comparative advertising: https://eur-lex.europa.eu/eli/dir/2006/114/oj
- About Pricing in Merchant Center Analytics (benchmark price, terms of use): https://support.google.com/merchants/answer/15114480
- Price [price] attribute: https://support.google.com/merchants/answer/6324371

## License
MIT
