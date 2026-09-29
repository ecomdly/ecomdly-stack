---
name: clearance-markdown-planner
owner: marginmath
category: Pricing & merchandising
description: Plans SKU-level clearance markdowns from stock, sell rate and exit date, stops at a VAT-correct floor, and shows the EU Omnibus 30-day prior price and correctly based discount percentage for every step, including progressive-markdown rules.
version: v3
license: MIT
updated: 2026-09-29
recommended: true
security_checked: true
url: https://ecomdly.com/skills/marginmath/clearance-markdown-planner
raw: https://ecomdly.com/raw/marginmath/clearance-markdown-planner.md
install: npx @ecomdly/cli add marginmath/clearance-markdown-planner
---

# Clearance markdown planner

Produces a SKU-by-SKU markdown schedule from stock and sales data: whether to mark down now, by how much, when to check again, where to stop, and the legally required "prior price" to show with each reduction in the EU. Markdowns that come too late cost margin twice, once on the discount and once on storage and ageing; markdowns that come too early give away margin on stock that would have sold anyway. The naive plan applies one percentage to everything at season end and shows the last regular price as the crossed-out price. Both are wrong: the depth should follow each SKU's gap between required and actual sell rate, and under Article 6a of the EU Price Indication Directive the reference price for any announced reduction is the lowest price of the previous 30 days, which after a first markdown is the markdown price itself.

## When to use

- End of season, or when a SKU's stock outlasts its selling window.
- Monthly on slow movers before stock ages into write-off.
- Before a sale event (Black Friday, end-of-season sale) to check which SKUs can legally show which "was" price.

## When not to use

- Pricing against competitors on current-season lines: `competitor-price-brief`.
- Perishable food and drinks with short expiry: Member States may apply different rules to goods liable to deteriorate or expire rapidly (Czechia exempts them from the prior-price rule); this skill's legal notes assume non-perishable goods.
- Services and digital content: Article 6a covers goods only; price claims for services fall under general unfair commercial practices rules.
- Loyalty-programme or genuinely personalised discounts (e.g. a voucher after a purchase): outside Article 6a per the Commission guidance, though a code available to all visitors is not personalised.

## Inputs

Required per SKU:

- Units on hand (and in transit, if it will arrive before the exit date).
- Units sold per week, last 8 weeks, and units sold since season start.
- Current selling price incl. VAT, and **daily price history for at least the last 30 days** per sales channel (e-shop, marketplace, physical store), including any promotional prices and codes available to all customers.
- Landed cost ex VAT and VAT rate.
- Exit date: end of season or the last day the store wants to hold the stock.
- Store margin floor (e.g. margin ≥ 10% of the ex-VAT price) or a fixed minimum price.

Optional: salvage value per unit (outlet, liquidator, return-to-vendor terms); holding cost per unit per week; history of past markdowns with before/after weekly rates (to estimate the store's own response to discounts); first date the SKU was offered (for new arrivals); planned promotions in the next 30 days; the store's price-ending convention and allowed discount steps.

If the price history is missing, the plan can still compute depth and timing, but every step is marked `prior price unknown — do not announce as a discount until verified`. Never assume the current price is the 30-day low.

## Best practices

### Timing and depth

1. **Start from the rate you need, not a percentage.** Required weekly rate = units on hand ÷ weeks left. Gap = required rate ÷ current rate. A SKU with gap ≤ 1 needs no markdown; one with gap 3 needs a much deeper change than one with gap 1.2. This turns "slow mover" into a number the owner can check two weeks later.
2. **Measure the current rate honestly.** Use the last 4 weeks; use 8 when sales are lumpy or stock-outs of sizes distorted recent weeks. A SKU that sold out of its main sizes will not recover its rate with a discount on the leftover sizes; plan the remaining size curve, not the SKU total.
3. **Weeks of cover (WOC)** = units on hand ÷ current rate. **Sell-through** since season start = units sold ÷ (units sold + units on hand). Both go in the output; WOC vs weeks left drives the decision.
4. **Default decision grid (house rule, replaceable by the store's own):**
   - WOC ≤ weeks left → no markdown; recheck in 2 weeks.
   - WOC between 1× and 2× weeks left → first step 20–30%.
   - WOC above 2× weeks left, or sell-through under 40% at mid-season → first step 30–40%.
   - Review every 2 weeks; if the rate has not risen by at least half, take the next step (typically +10–20 points), until the floor.
   These are starting heuristics, not benchmarks. If the store has past markdown history, replace them with its observed response: for each past markdown, uplift = rate after ÷ rate before, per discount depth and category, and pick the smallest step whose observed uplift closes the gap.
5. **Earlier and shallower beats later and deeper** when the season still has weeks of demand left, because a discount works on the remaining traffic, and traffic for seasonal goods falls towards the exit date. This is why the plan front-loads the first step and uses short review cycles.
6. **Floor = where the markdown stops.** Floor ex VAT = landed cost ÷ (1 − floor margin); floor incl VAT = floor ex VAT × (1 + VAT). Below the floor, report the SKU for bundle, outlet channel, return-to-vendor or liquidation. If salvage value is known and exceeds what the floor protects, say so: the owner may choose a different floor.
7. **Advertising must follow the markdown.** A marked-down SKU's margin falls, so its break-even ROAS rises; leaving it in a campaign with a target set for full-price margin loses money on each sale. Propose a custom label for clearance items (`feed-custom-label-planner`) and recalculate the target (`break-even-roas-calculator`).

### The EU prior-price rule (Article 6a, Directive 98/6/EC, inserted by Directive (EU) 2019/2161, applicable from 28 May 2022)

8. **Any announcement of a price reduction must show the prior price**, defined as the lowest price the trader applied during a period of at least 30 days before the reduction. "Announcement" includes a percentage ("−30%"), a crossed-out price, "sale", "special offer", "Black Friday" wording and similar claims that create the impression of a reduction, in any channel. A price simply lowered without any reduction claim is not covered by Article 6a.
9. **The prior price includes earlier promotional prices.** The Commission guidance is explicit: failing to take into account prices applied during previous promotions within the 30 days breaches Article 6a. After step 1 at 1 790 Kč, the lowest price of the last 30 days is 1 790 Kč, not the original 2 490 Kč.
10. **The percentage must be calculated from the prior price.** The guidance gives the example: "50% off", lowest 30-day price 100, last price 160 → the 50% is calculated from 100. The CJEU confirmed this in case C-330/23 (Aldi Süd, 2024): showing the prior price only for information while calculating the discount from a higher price is not allowed.
11. **Progressive (step) markdowns: check the national rule.** Article 6a(5) lets Member States provide that, when a reduction is progressively increased, the prior price stays the price before the first reduction. The guidance reads this narrowly: only for an uninterrupted campaign, without any price increase in between. Czechia uses this option (Act No. 634/1992 Coll., section 12a(1)(c): the lowest price in the 30 days before the first discount when the discount is increased gradually). Where a Member State has not adopted the option, each step's prior price is the lowest price in the preceding 30 days, which includes the previous step. The plan must show both the prior price and the percentage on the correct basis for the market concerned; if the market is not known or its rule not verified, use the strict general rule.
12. **Interruptions reset the protection.** If the price goes back up between steps (or the sale ends and restarts, e.g. "−20% every weekend"), the progressive exception does not apply and the prior price is the lowest price of the last 30 days, including the earlier sale prices.
13. **New arrivals and returning seasonal goods.** Member States may allow a shorter period for goods on the market for less than 30 days (Czechia: the lowest price since the product was first offered). Goods returning after a stock-out or from last season are not new arrivals; the guidance allows using a longer reference period in which the product was offered for at least 30 days in total, with the lowest price in that whole period. Seasonal clothing is not "perishable" for the derogation in Article 6a(3).
14. **Per channel.** If the store sells the same SKU at different prices on its e-shop and elsewhere, the prior price is determined per channel.
15. **Other reference prices are allowed only alongside, clearly explained**, e.g. "our regular price outside promotions was …". They must not obscure the Article 6a prior price. RRP comparisons are not price reductions but must not be presented so that shoppers read them as one.
16. **Codes available to everyone count.** "−20% with code XYZ" open to all visitors is an announced reduction and needs prior prices; a voucher sent to one customer after a purchase is personalised and outside Article 6a.
17. **National law governs details and enforcement.** The Directive is transposed nationally; confirm the national provision for each market the store sells to before the plan is published. The Czech text is quoted above; other markets must be verified by the owner or counsel.

## Process

1. **Validate inputs.** Flag SKUs without 30-day price history, landed cost or exit date.
2. **Per SKU compute:** current rate (4 or 8 weeks), WOC, sell-through, weeks left, required rate = on hand ÷ weeks left, gap = required ÷ current.
3. **Decide the step** with the store's grid or the default grid (rule 4), or with observed uplift if history exists.
4. **Price the step:** new price = current regular price × (1 − step), rounded down to the store's price ending; recompute the actual percentage from the rounded price.
5. **Floor check.** If the new price < floor, use the floor if it still closes part of the gap; otherwise mark `exit channel`.
6. **Prior price per channel:** lowest daily price in the 30 days before the step's start date. For a later step: if the market applies the progressive exception and the campaign was uninterrupted with no increase, keep the first step's prior price; otherwise recompute the 30-day low (which includes earlier steps).
7. **Displayed percentage** = 1 − new price ÷ prior price. If that percentage is small or negative, recommend announcing no discount (just lower the price) or waiting.
8. **Schedule the review** 2 weeks later: required rate for the remaining weeks, and the condition for the next step.
9. **Flag** SKUs in planned promotions within 30 days (their promo prices will become the prior price) and SKUs in ad campaigns needing a clearance label.
10. **Output** the plan; nothing is applied.

## Pitfalls and edge cases

- **"Was" price from the ERP's list price.** The list price is not the prior price if a promotion ran in the last 30 days. Always compute from daily history.
- **Black Friday followed by Christmas sale.** Successive campaigns within 30 days: the second campaign's prior price is the Black Friday price.
- **Price raised before a sale.** A raise within 30 days before a reduction does not become the reference; the lowest price does.
- **Size-broken stock.** A discount on remaining odd sizes rarely restores the rate; consider bundles or outlet earlier.
- **Feed and site out of sync.** Changing the site price without updating the feed (or the sale price attributes) causes Merchant Center price-mismatch problems; hand the list to whoever updates the feed.

## Rules

- Never propose a price under the floor or a contractual minimum.
- Every reduction shows the prior price per channel and the percentage calculated from it. If the prior price cannot be computed, the step is marked "do not announce as a discount until verified".
- No invented demand uplift. Expected outcome is stated as the weekly rate required; uplift estimates only from the store's own markdown history, labelled as such.
- Legal notes are general information from the Directive, the Commission guidance and verified national text; they are not legal advice. Markets without a verified national rule use the strict general rule.
- Read-only: the agent does not change prices in the store, feed or marketplaces. A human approves and applies.

## Output format

```
Clearance markdown plan — <store> — prepared <date> — market rule: <CZ §12a (progressive allowed)|general Art. 6a (strict)|unverified → strict>
Missing inputs: <list or none>

SKU <id> <name> — on hand <n>, rate <n>/wk (<4|8> wks), WOC <n>, weeks left <n>, sell-through <x>%, required <n>/wk (gap <x>)
  step <k> (start <date>, channel <e-shop>): <old> → <new> (−<x>% vs prior) | prior price (30-day low) <p> | floor <f>
  basis: <progressive, uninterrupted since <date>|recomputed 30-day low>
  review <date>: next step if rate < <n>/wk
  ads: <move to clearance label|none>
SKU <id> ... → <exit channel: outlet|bundle|return-to-vendor> (reason)

Flags: <promotions within 30 days, missing price history, channels with different prices>
```

## Worked example

Illustrative numbers only. Czech store, VAT 21%, floor margin 10% of ex-VAT price, price ending …90, exit date end of week 44, today week 36.

```
Clearance markdown plan — outdoor-shop.cz — prepared 2026-09-01 — market rule: CZ §12a (progressive allowed)
Missing inputs: none

SKU 4471 Trail jacket M — on hand 180, rate 9/wk (4 wks), WOC 20, weeks left 8, sell-through 31%, required 22.5/wk (gap 2.5)
  step 1 (start wk 36, e-shop): 2 490 → 1 790 (−28.1% vs prior) | prior price 2 490 (no promotion in last 30 days) | floor 1 240
  basis: first reduction
  review wk 38: next step if rate < 22.5/wk
  ads: move to custom_label_2 = clearance; recalculate target

  (wk 38 review: rate 12/wk, +33% < +50% → step 2)
  step 2 (start wk 38, e-shop): 1 790 → 1 490 | prior price 2 490 → displayed −40.2%
  basis: progressive, uninterrupted since wk 36 (CZ §12a(1)(c))
  If sold in a market without the progressive option: prior price = 1 790 → displayed −16.8%

SKU 4490 Fleece S — on hand 164, rate 4/wk, WOC 41, weeks left 8 → step 2 would reach floor 890 and still need 20.5/wk → exit channel: outlet or bundle with Trail jacket

Flags: SKU 4502 ran "−15% weekend" 2026-08-15..16 → its 30-day low is the promo price until 2026-09-15
```

Arithmetic: floor 4471 = landed cost 922 ÷ 0.90 × 1.21 = 1 239.6 → 1 240. Step 1: 2 490 × 0.72 = 1 792.8 → 1 790; 1 − 1 790 ÷ 2 490 = 28.1%. Step 2: 1 − 1 490 ÷ 2 490 = 40.2%; 1 − 1 490 ÷ 1 790 = 16.8%. Required rate: 180 ÷ 8 = 22.5. Fleece: 164 ÷ 8 = 20.5.

## Quality checklist

- Required rate and gap computed for every SKU; decision follows the stated grid or observed history.
- Every step shows the prior price per channel from daily history, and the percentage calculated from it.
- Progressive exception used only where national law provides it and the campaign was uninterrupted with no increase.
- Earlier promotions within 30 days reflected in the prior price.
- No price below floor; exit channel proposed instead.
- Uplift not invented; legal notes carry the market rule used.
- Ad-campaign consequences flagged.

## Sources

- Directive (EU) 2019/2161 (Omnibus), Article 2 inserting Article 6a into Directive 98/6/EC: https://eur-lex.europa.eu/eli/dir/2019/2161/oj
- Directive 98/6/EC (Price Indication Directive), consolidated: https://eur-lex.europa.eu/eli/dir/1998/6/oj
- Commission Notice, Guidance on the interpretation and application of Article 6a (2021/C 526/02): https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX:52021XC1229(06)
- CJEU, Case C-330/23 Aldi Süd, judgment: https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX:62023CJ0330
- Czech Act No. 634/1992 Coll. on consumer protection, section 12a: https://www.zakonyprolidi.cz/cs/1992-634

## License
MIT
