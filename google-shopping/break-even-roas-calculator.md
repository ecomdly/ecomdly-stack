---
name: break-even-roas-calculator
owner: marginmath
category: Google Shopping
description: Calculates break-even ROAS, PNO/COS and target ROAS per category and campaign mix from contribution margin (COGS, fees, fulfilment, returns), on the same VAT and shipping basis as your Google Ads conversion value, with all arithmetic shown.
version: v4
license: MIT
updated: 2026-09-29
recommended: true
security_checked: true
url: https://ecomdly.com/skills/marginmath/break-even-roas-calculator
raw: https://ecomdly.com/raw/marginmath/break-even-roas-calculator.md
install: npx @ecomdly/cli add marginmath/break-even-roas-calculator
---

# Break-even ROAS calculator

Produces break-even ROAS, break-even PNO/COS and a target ROAS per category (and per campaign mix), with every line of arithmetic shown. A 400% ROAS target copied from a blog can lose money on every order, and a correct margin can still give a wrong target when it is measured on a different basis than the conversion value Google Ads receives (VAT included or not, shipping included or not, before or after returns). The key insight: break-even ROAS = 1 ÷ contribution margin, where the margin is contribution per order divided by exactly the value the ad platform reports for that order. Get the basis wrong and the answer can be off by the VAT rate or the return rate before any cost is even considered.

## When to use

- Before setting or changing tROAS on Shopping, PMax or Search, or a PNO/COS limit on a comparison site.
- When a category's margin, fees, shipping policy, free-shipping threshold or return rate changes.
- When an agency reports "ROAS 5" and the owner asks whether that is profitable.

## When not to use

- Deciding which products to advertise at all given stock age: `clearance-markdown-planner`.
- Diagnosing why Ads revenue differs from store revenue: `revenue-discrepancy-reconciler`. Run it first if the gap is large, because a target built on inflated conversion value is too low.
- Pricing against competitors: `competitor-price-brief`.

## Inputs

All inputs come from the store. Required, per category or per margin tier:

| input | definition | typical source |
|---|---|---|
| Ads value basis | what the conversion tag sends: incl. or ex VAT; products only or incl. shipping; before or after discounts | conversion action setup, tag/dataLayer, GTM |
| Average order value on that basis (R) | mean conversion value per order | Google Ads conversion value ÷ conversions, cross-checked with store |
| Product revenue ex VAT (P) and shipping charged ex VAT (S) | per order | store order export |
| VAT rate (v) | rate applied to the orders | store / accounting |
| COGS per order | landed cost of goods sold, ex VAT | ERP, purchase prices |
| Payment fees | % and fixed fee, and whether charged on the incl-VAT amount (normally yes) | payment provider contract |
| Fulfilment per order | pick and pack, packaging, carrier cost paid by the store | 3PL invoice, carrier price list |
| Return rate (r) | share of revenue refunded, by value, over a period long enough for returns to complete | returns log, `returns-reason-analyzer` |
| Resellable share (k) | share of returned COGS that goes back to sellable stock | warehouse |
| Cost per return | return shipping paid by the store, inspection, repackaging | returns log |
| Other variable costs | per-order costs that scale with orders: marketplace commission, COD fee, gift packaging, loyalty points | contracts |

Optional: desired profit as a share of revenue (p) or per order (π); order mix by category within each campaign; repeat-purchase data if the owner wants a new-customer target.

If an input is missing, stop at that line, show the formula with a named blank, and ask. Never substitute an "industry average".

## Best practices

### Basis first

1. **Match the denominator to the conversion value.** ROAS = conversion value ÷ ad cost, so the margin must be computed as contribution ÷ the same value. If the tag sends value incl. VAT, R = P × (1 + v) (plus shipping if included). Break-even ROAS on an incl-VAT basis = (1 + v) × break-even ROAS on the ex-VAT basis. At 21% VAT that is 21% higher; forgetting it makes every campaign look more profitable than it is.
2. **Returns are not in Ads value.** Google Ads records value at purchase. Unless the store uploads conversion adjustments (retractions/restatements), reported value includes orders later returned or cancelled. Google states that adjustments count for bidding only if made within 7 days of the conversion, and in reports within 54 days; most returns arrive later than 7 days, so Smart Bidding keeps seeing pre-return value either way. So the margin must subtract returns while keeping the denominator on the before-returns basis. This is why margin uses R in the denominator, not revenue after returns.
3. **Discounts and coupons.** If the tag sends pre-discount value, the store's actual revenue is lower than R. Compute P after discounts, keep R as sent, and let the margin carry the difference.
4. **Shipping.** If shipping charged is included in R, include it in revenue and include carrier cost in costs. If excluded from R, still include both in contribution (the order's economics include them), but R stays products-only.

### Contribution, not gross margin

5. **Contribution per order (CM)** includes every cost that scales with an order and nothing that does not. Fixed costs (salaries, rent, software, the agency fee) stay out; they are covered through the profit share p. Ad cost is also out, because it is what the ROAS target limits.
6. **Payment fees are charged on what the customer pays**, which includes VAT. Check whether the provider refunds its fee on refunded orders; many do not, so keep the fee on returned orders unless the contract says otherwise.
7. **Returns cost three ways:** lost revenue, goods that cannot be resold, and handling. Model them separately, because they move independently (a better returns process raises k without changing r).
8. **Measure the return rate by value, over a matured window.** Orders from the last few weeks have not finished returning. Use a cohort old enough for the store's return period plus processing.

### From margin to targets

9. **Break-even formulas.**
   - Contribution margin m = CM ÷ R.
   - Break-even ROAS = 1 ÷ m. Break-even PNO (COS, cost of sale) = m, because PNO = ad cost ÷ value = 1 ÷ ROAS.
   - Target ROAS for profit share p of R: 1 ÷ (m − p). Target PNO = m − p.
   - Target ROAS for profit π per order: R ÷ (CM − π).
   - If m ≤ p (or CM ≤ π), no ROAS target reaches the goal at this cost structure; say so and point to the levers (price, COGS, shipping policy, returns).
10. **Campaign target = value-weighted margin of what the campaign sells.** m_campaign = Σ CM_i ÷ Σ R_i across categories, weighted by the revenue each category actually gets through that campaign, not by the catalog. A campaign whose sales drift towards low-margin products needs a higher target even if no category's margin changed.
11. **Break-even is the average, not the edge.** A campaign run at exactly break-even ROAS on average is buying its last orders at a loss, because marginal orders convert at worse ROAS than the average. Google's own guidance on asset groups refers to marginal rather than average ROI. Use break-even as the floor and set the target above it by the profit the store wants.
12. **Different bases on different platforms.** Comparison sites (Heureka, Zboží.cz) and Google Ads may report PNO or ROAS on different VAT bases and attribution models. Confirm the basis each platform reports before comparing their numbers or applying one target to both.
13. **New-customer targets are a separate, labelled decision.** Bidding below break-even to acquire customers is valid only with the store's own repeat data: expected contribution from repeat orders within a stated horizon, times the observed repeat probability. Show it as a second column, never as the default.
14. **Conversion value quality caps precision.** If Ads value differs from store revenue by more than the rounding in this calculation, reconcile first. Modeled conversions, attribution model and cross-device credit all change reported value; state the attribution model and conversion action used.

## Process

1. **Establish the basis.** Record what R contains (VAT, shipping, discounts). If unknown, ask for the tag configuration or a sample order compared with its Ads conversion value.
2. **Per category, compute CM line by line:**
   - Revenue kept = (P + S) × (1 − r)
   - COGS net of resold returns = COGS × (1 − r × k)
   - Payment fees = fee% × (P + S) × (1 + v) + fixed fee
   - Fulfilment = pick/pack + packaging + carrier
   - Return handling = r × cost per return
   - Other variable costs
   - CM = revenue kept − COGS net − payment fees − fulfilment − return handling − other
3. **Margin and break-even:** m = CM ÷ R; break-even ROAS = 1 ÷ m; break-even PNO = m.
4. **Targets:** 1 ÷ (m − p) and/or R ÷ (CM − π). Check m > p.
5. **Campaign mix:** weight by category revenue in each campaign; compute m_campaign and its targets.
6. **Sensitivity:** recompute break-even ROAS with r + 5 points and with COGS + 5%, so the owner sees which input moves the answer most.
7. **Report** with all arithmetic, rounded only at the end (margins to 0.1%, ROAS to 2 decimals).

## Pitfalls and edge cases

- **ROAS as percentage vs ratio.** 3.5 and 350% are the same; Google Ads tROAS is entered as a percentage. State both.
- **Free-shipping threshold.** Orders just above the threshold carry carrier cost with no shipping revenue. If the category's AOV sits near the threshold, split orders above and below it.
- **Cash on delivery and refused parcels.** A refused COD parcel is a return with outbound and return shipping and no revenue; include the refusal rate in r if the store has it.
- **Bundles and multi-category orders.** Allocate CM by line revenue, not by the category of the clicked product, when computing category margins.
- **Currency.** If the Ads account currency differs from the store's, convert value and cost at the same rate and date basis.
- **Very small categories.** A return rate from 20 orders is noise; use the parent category's rate and say so.
- **VAT regime changes (OSS, cross-border sales).** Different destination countries can have different VAT rates; compute per market when Ads value is incl. VAT.

## Rules

- Every number comes from the store or its accounts. Missing inputs stop the calculation at that line with a request; no averages from elsewhere.
- Show the arithmetic, not only the result.
- State the basis (VAT, shipping, discounts, before returns) in the output header.
- If m ≤ p, state that no ROAS target meets the goal and list which inputs would have to change.
- Output is advice. The agent does not change bid strategies, targets or budgets.

## Output format

```
Break-even ROAS — <store> — data period <dates> — prepared <date>
Ads value basis: <ex|incl> VAT (<v>%), <products only|incl. shipping>, <before|after> discounts, before returns
Attribution / conversion action: <name, model>

Category: <name>
  R (Ads value per order)          <x>
  Revenue kept (P+S)×(1−r)         <x>
  COGS net of resold returns       <x>
  Payment fees                     <x>
  Fulfilment                       <x>
  Return handling                  <x>
  Other variable                   <x>
  Contribution per order (CM)      <x>
  Margin m = CM ÷ R                <x>%
  Break-even ROAS / PNO            <x> (<x>%) / <x>%
  Target ROAS (p = <p>%)           <x> (<x>%)

| category | m | break-even ROAS | break-even PNO | tROAS (p) | sensitivity r+5 pts |
|---|---|---|---|---|---|

Campaign mix: <campaign> = <share>% <cat A> + <share>% <cat B> → m <x>% → break-even <x>, tROAS <x>
Missing inputs: <list or none>
```

## Worked example

Illustrative numbers only. Store in Czechia, amounts in EUR, VAT 21%. Ads value = products ex VAT, before returns, no shipping (free shipping).

Footwear, per order: P = 80.00; S = 0; COGS 32.00; payment fee 1.5% + 0.25 on the incl-VAT amount (96.80) = 1.70; fulfilment = pick/pack 1.50 + packaging 0.50 + carrier 4.50 = 6.50; returns r = 15%, k = 80%, cost per return 7.00.

```
Category: Footwear
  R (Ads value per order)          80.00
  Revenue kept 80.00 × 0.85        68.00
  COGS net 32.00 × (1 − 0.15×0.80) 28.16
  Payment fees                      1.70
  Fulfilment                        6.50
  Return handling 0.15 × 7.00       1.05
  Other variable                    0.00
  CM = 68.00 − 28.16 − 1.70 − 6.50 − 1.05 = 30.59
  Margin m = 30.59 ÷ 80.00         38.2%
  Break-even ROAS / PNO            2.62 (262%) / 38.2%
  Target ROAS (p = 10%)            1 ÷ (0.382 − 0.100) = 3.54 (354%)
  Target ROAS (π = 5.00 per order) 80.00 ÷ (30.59 − 5.00) = 3.13 (313%)
  If Ads value were incl. VAT      96.80 ÷ 30.59 = 3.16 break-even

| category | m | break-even ROAS | break-even PNO | tROAS (p 10%) | sensitivity r+5 pts |
|---|---|---|---|---|---|
| Footwear | 38.2% | 2.62 | 38.2% | 3.54 | 2.91 |
| Socks | 52.1% | 1.92 | 52.1% | 2.38 | not computed (inputs not shown) |

Campaign mix: PMax – Footwear = 70% Footwear + 30% Socks (by Ads value)
  m = 0.70 × 38.2% + 0.30 × 52.1% = 42.4% → break-even 2.36, tROAS (p 10%) 3.09
```

Sensitivity for Footwear at r = 20% (the table's last column): revenue kept 64.00; COGS net 32.00 × 0.84 = 26.88; return handling 1.40; CM = 64.00 − 26.88 − 1.70 − 6.50 − 1.40 = 27.52; m = 34.4%; break-even 2.91. Five points of returns move break-even by about 0.3, more than a one-point change in payment fees would.

## Quality checklist

- Basis stated and matches the conversion action's actual value.
- Payment fee computed on the incl-VAT amount unless the contract says otherwise.
- Returns reduce revenue and COGS separately, with handling as its own line.
- No fixed costs and no ad cost inside CM.
- m > p checked; if not, stated plainly.
- Campaign targets weighted by the campaign's actual sales mix.
- Every number traceable to a named input; missing inputs listed.
- Rounding only at the end; ROAS shown as ratio and percentage.

## Sources

- Google Ads, About target ROAS bidding: https://support.google.com/google-ads/answer/6268637
- Google Ads, About conversion values: https://support.google.com/google-ads/answer/3419241
- Google Ads, How to adjust your conversions (retract, restate; 7-day bidding window, 54-day limit): https://support.google.com/google-ads/answer/7686280
- Google Ads, About asset group reporting for Performance Max (marginal vs average ROI note): https://support.google.com/google-ads/answer/13872527

## License
MIT
