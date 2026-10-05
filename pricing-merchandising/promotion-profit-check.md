---
name: promotion-profit-check
owner: marginmath
category: Pricing & merchandising
description: Decides before launch whether a sale, coupon or bundle pays: break-even uplift from contribution margin, pull-forward, cannibalisation, coupon leakage, ad-target impact, EU 30-day prior price check and a baseline readout.
version: v1
license: MIT
updated: 2026-10-05
recommended: false
security_checked: true
url: https://ecomdly.com/skills/marginmath/promotion-profit-check
raw: https://ecomdly.com/raw/marginmath/promotion-profit-check.md
install: npx @ecomdly/cli add marginmath/promotion-profit-check
---

# Promotion profit check

Produces a go / change / no-go decision for a planned discount, sale, coupon or bundle, with the volume the promotion must add just to break even, the profit change under the owner's own uplift scenario, the effect on ROAS targets, a prior-price check for the announcement, and a readout plan for after the promotion. The naive test ("revenue went up 40%, the sale worked") ignores that every baseline unit now earns less, that some promo units were only pulled forward from next month or taken from a sister product, and that a public coupon code reaches customers who would have paid full price. The key insight: a discount comes entirely out of contribution, not out of revenue. A 20% discount on a product with 36% contribution margin roughly halves the profit per unit, so volume has to almost double before the store is even level, and the uplift needed rises much faster than the discount does.

## When to use

- Before approving a sitewide sale, category discount, newsletter code, influencer code or bundle price.
- When choosing between discount depths (10% vs 20%) or between a sitewide sale and a targeted code.
- Before Black Friday or seasonal campaigns, including the prior-price wording on product and ad creatives.
- After a promotion ends, to read out what it actually earned against a baseline.

## When not to use

- Clearing aged or end-of-life stock where the goal is cash and exit, not profit: `clearance-markdown-planner`.
- Setting the free-shipping threshold or shipping prices: `free-shipping-threshold-planner`.
- Calculating ROAS targets from scratch: `break-even-roas-calculator` (this skill reuses its contribution lines).
- Discounts inside an abandoned-cart or win-back flow: `abandoned-cart-sequence`, `winback-segment-planner`.
- Reading a randomised coupon test statistically: `ab-test-readout`.

## Inputs

| input | definition | typical source |
|---|---|---|
| Promotion design | mechanic (% off, fixed off, code, bundle, gift), depth, products, channels, dates, who can see it | owner, campaign brief |
| Country of the shoppers addressed | each country whose national price-indication law applies | owner, shop domains |
| Price history per SKU | every selling price with dates for at least the 30 days before the start (longer if national law requires) | e-shop price log, ERP, feed snapshots |
| Contribution per unit before and after | revenue ex VAT minus COGS, payment fees, fulfilment, returns, as in `break-even-roas-calculator` | store data, that skill's output |
| VAT rate | rate per destination country | accounting |
| Baseline units per SKU | expected units in the promo window without the promo | own sales history: pre-period trend, same period last year |
| Uplift scenario | expected units with the promo | a measured past promotion of the same type, or an owner-entered scenario, labelled as such |
| Ads value basis | whether the conversion tag sends pre- or post-discount value, incl. or ex VAT | conversion action setup, GA4 `purchase` event |

Optional: pull-forward estimate (units the promo takes from the weeks after); substitution from sister SKUs with their contribution; coupon redemption export with customer IDs and the target list; return rate of past promo orders; bundle component costs.

If an input is missing, stop at that line, show the formula with a named blank and ask. Never fill uplift, pull-forward or leakage with an industry average.

## Best practices

### Profit arithmetic

1. **Work in contribution per unit, not gross margin.** Payment fees and returns scale with price, fulfilment does not; a gross-margin shortcut overstates what is left after the discount.
2. **Required volume ratio = CM_before ÷ CM_after.** Required uplift = that ratio − 1. If CM_after ≤ 0, no volume saves the promotion; say so before discussing anything else.
3. **Every baseline unit pays the discount.** The cost of the promo is baseline units × (CM_before − CM_after), and only incremental units pay it back. This is why deep discounts on products that sell well anyway are the most expensive.
4. **Subtract pull-forward and cannibalisation.** Units bought now instead of next month, and units moved from a sister SKU, are not incremental. Required promo units U* = (B × CM_before + F × CM_before + S × CM_sister) ÷ CM_after, where B baseline, F pull-forward, S substituted units.
5. **Uplift comes from the store's own past promotions or an explicit scenario.** Elasticity varies by product, season and audience; a number from a blog is not the store's demand curve. If no comparable promotion exists, show the break-even uplift and let the owner judge whether it is reachable.
6. **Model returns on promo orders separately when data exists.** Discount buyers can return at a different rate; use the store's own past promo cohort, matured.

### Coupons and bundles

7. **Coupon leakage is a cost line.** A code meant for subscribers that spreads to code sites gives the discount to non-target buyers. Leakage share = redemptions by customers outside the target list ÷ all redemptions; leakage cost = leaked redemptions × average discount. Single-use or customer-bound codes reduce it.
8. **Codes are visible in data.** GA4 records `coupon` at event and item level, independently; check that the store actually sends it before relying on it for the readout.
9. **Bundles: compute on the bundle, then check the components.** Bundle CM = bundle price ex VAT − sum of component costs − one fulfilment. If buyers would have bought the main item alone at full price, the bundle discount on it is a baseline cost like any other.

### Ad targets

10. **A promotion changes break-even ROAS.** Lower CM per order and a lower order value both raise it. Recompute m = CM ÷ R on the basis the tag sends, as in `break-even-roas-calculator`. If the tag sends pre-discount value, reported ROAS overstates the promo; if post-discount, ROAS falls although nothing broke. Tell the PPC owner the promo-period floor before launch.

### Prior-price rule (EU)

11. **Any announcement of a price reduction must show the prior price.** Directive 98/6/EC Art. 6a, inserted by Directive (EU) 2019/2161 and applied from 28 May 2022: the prior price is the lowest price the trader applied in a period not shorter than 30 days before the reduction. Member States may set different rules for goods that deteriorate or expire rapidly (para 3), a shorter period for products on the market under 30 days (para 4), and, for progressively increased reductions, the price before the first reduction (para 5).
12. **The percentage must be computed from the prior price.** The Commission's 2021 guidance says so, and the CJEU confirmed it in C-330/23 Aldi Süd (26 September 2024). A brief flash sale in the last 30 days therefore lowers the reference for the next sale.
13. **National law decides the details; the country is an input.** For Czechia, § 12a of Act No. 634/1992 Coll. requires the lowest price over the 30 days before the discount, uses the shorter-period and progressive-discount options, and exempts perishable goods and goods with a short shelf life. Other Member States chose differently; look up each target country's transposition and do not apply Czech rules elsewhere.
14. **Scope per the Commission guidance:** "sale" or "Black Friday" claims without a figure are in scope; codes available to potentially all visitors are in scope; long-term loyalty programmes, real personalised reductions and conditional offers such as "buy one, get two" are outside Art. 6a but remain under the Unfair Commercial Practices Directive. Art. 6a covers goods only, not services or digital content.

## Process

1. **Record the design**: mechanic, depth, SKUs, channels, dates, audience, countries.
2. **Prior-price check** per SKU and country: from the price log, find the lowest price in the legally required window; compute the displayable reduction from it; flag any creative whose percentage or crossed-out price uses another reference.
3. **Contribution per unit** before and during the promo, line by line, with fees on the incl-VAT price.
4. **Break-even**: ratio, required uplift, then U* including F and S.
5. **Scenario**: profit change = U × CM_after − (B + F) × CM_before − S × CM_sister, for each uplift the owner provides; add coupon leakage for code mechanics.
6. **Alternatives**: repeat for a smaller depth, a targeted code or a bundle, so the owner sees the cheapest way to the same goal.
7. **Ad targets**: promo-period break-even ROAS on the tag's basis.
8. **Readout plan**: fix the baseline method now (pre-period trend, last year adjusted, or a holdout of SKUs, regions or customers), the post-promo window for pull-forward (at least as long as the promo), and the metric (contribution, not revenue).
9. **After the promo**: actual promo units and contribution vs baseline, post-window dip vs baseline, substitution in sister SKUs, matured returns, leakage. Report realised profit change and whether it matched the scenario.

## Pitfalls and edge cases

- **Baseline from a promo period.** If last month already had a discount, its sales are not a baseline; use an undiscounted window.
- **Stockouts.** Units capped by stock understate demand in the readout and overstate it when used as baseline after restock.
- **Discount stacking.** Sitewide sale plus a newsletter code plus a loyalty discount can push CM negative on some orders; check the checkout's stacking rules.
- **Marketplaces and comparison sites.** Sale prices sent to Heureka, Zboží.cz or a marketplace must carry the same prior price as the shop; map them with `comparison-feed-mapper` and check each channel's current documentation rather than assuming.
- **Free-shipping interaction.** A discount can drop baskets below the free-shipping threshold or push others above it; rerun with the shipping line.
- **Successive campaigns.** Back-to-back promotions within 30 days make the earlier promo price the prior price for the later one, unless the progressive-reduction rule of the country applies.
- **Readout attribution.** Ads-attributed revenue during a promo is not incrementality; use the baseline comparison.

## Rules

- Every input is the store's own data or an owner-entered scenario, labelled. No invented elasticities, uplifts or benchmarks.
- Show arithmetic; round only at the end.
- Legal statements are limited to the verified rules above; national specifics beyond them are a check for the store's counsel.
- The agent does not change prices, create codes, publish creatives, or change bids or budgets. It recommends; the owner acts.

## Output format

```
PROMOTION PROFIT CHECK · <store> · <promo name> · <dates> · prepared <date>
Design: <mechanic, depth, SKUs, channels, audience> · countries <list>
Prior price (<country>, rule <ref>): lowest in window <price> on <date> → displayable reduction <x>% · creatives OK/fix: <list>
Per unit            before     promo
  Revenue kept       <x>        <x>
  COGS net           <x>        <x>
  Payment fees       <x>        <x>
  Fulfilment         <x>        <x>
  Return handling    <x>        <x>
  CM                 <x>        <x>
Required uplift (no F/S): <x>% · U* with F <n>, S <n>: <n> units (+<x>% vs baseline <n>)
Scenario <source>: U <n> → profit change <x> · coupon leakage <x>
Alternatives: <depth/mechanic> → required uplift <x>%, profit change <x>
Ads: break-even ROAS before <x> / during <x> on <basis>
Decision: GO / CHANGE (<what>) / NO-GO · Readout: baseline <method>, post window <dates>
Missing inputs: <list or none>
```

## Worked example

Illustrative numbers only. Czech store, EUR, VAT 21%, one unit per order. Product price 50.00 ex VAT (60.50 incl.); COGS 22.00; payment fee 1.5% + 0.25 on the incl-VAT price; fulfilment 5.00; returns r = 10%, resellable k = 90%, cost per return 6.00. Planned: 20% off for 14 days.

```
Per unit            before     promo (40.00 ex VAT, 48.40 incl.)
  Revenue kept       45.00      36.00     (P × 0.90)
  COGS net           20.02      20.02     (22.00 × (1 − 0.10 × 0.90))
  Payment fees        1.16       0.98
  Fulfilment          5.00       5.00
  Return handling     0.60       0.60     (0.10 × 6.00)
  CM                 18.22       9.40
Required uplift (no F/S): 18.22 ÷ 9.40 − 1 = 93.8%
Baseline B 400 (same 14 days last year, trend-adjusted); F 60 (scenario); S 30 sister units at CM 15.00
  U* = (400 × 18.22 + 60 × 18.22 + 30 × 15.00) ÷ 9.40 = 8 831.20 ÷ 9.40 = 939.5 → 940 units (+135%)
Scenario (last spring's 20% promo, measured): U 700 → 700 × 9.40 − 8 831.20 = −2 251.20
Alternative 10% off (45.00 ex VAT): CM 13.81 → required uplift 18.22 ÷ 13.81 − 1 = 31.9% (no measured uplift; owner to judge)
Ads (tag sends products ex VAT): break-even ROAS 50.00 ÷ 18.22 = 2.74 before;
  during promo 40.00 ÷ 9.40 = 4.26 if tag sends post-discount value, 50.00 ÷ 9.40 = 5.32 if pre-discount
Prior price: 60.50 for 30 days except a weekend flash at 54.45 twelve days before start
  → prior price 54.45; 48.40 is 1 − 48.40 ÷ 54.45 = 11.1% off, so "−20%" against 60.50 must not be shown
Decision: NO-GO at 20%. Test 10% only with a readout plan, or wait until the flash sale is out of the 30-day window.
```

Coupon variant, same product: 300 redemptions, 75 by customers not on the send list → leakage 25%, cost 75 × 10.00 (20% of 50.00 ex VAT) = 750.00 on top of the baseline cost.

## Quality checklist

- CM before and during computed line by line, fees on the incl-VAT price.
- Baseline from an undiscounted, stock-available window; method named.
- Pull-forward and substitution included or listed as missing.
- Uplift source labelled measured or scenario; no external benchmark.
- Prior price found from the price log for each country; reduction computed from it.
- Country-specific rule cited, or flagged for counsel.
- Promo-period break-even ROAS given on the tag's value basis.
- Readout plan fixed before launch; readout uses contribution, not revenue.

## Sources

- EUR-Lex, Directive (EU) 2019/2161 (Omnibus), Art. 2 inserting Art. 6a into Directive 98/6/EC; Art. 7 (apply from 28 May 2022): https://eur-lex.europa.eu/eli/dir/2019/2161/oj
- EUR-Lex, Commission Notice 2021/C 526/02, Guidance on Article 6a of Directive 98/6/EC: https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX:52021XC1229(06)
- EUR-Lex, Judgment of the Court, C-330/23 Aldi Süd, 26 September 2024: https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX:62023CJ0330
- e-Sbírka, Zákon č. 634/1992 Sb., o ochraně spotřebitele, § 12a: https://e-sbirka.gov.cz/sb/1992/634
- Google Analytics, GA4 recommended events reference (`purchase`, `coupon`): https://developers.google.com/analytics/devguides/collection/ga4/reference/events

## License
MIT
