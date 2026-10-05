---
name: free-shipping-threshold-planner
owner: marginmath
category: Pricing & merchandising
description: Sets or reviews a free-shipping threshold and shipping prices from your own basket distribution, carrier cost per parcel, contribution margin and bunching below the line, with profit per order and a test readout, no invented lift.
version: v1
license: MIT
updated: 2026-10-05
recommended: false
security_checked: true
url: https://ecomdly.com/skills/marginmath/free-shipping-threshold-planner
raw: https://ecomdly.com/raw/marginmath/free-shipping-threshold-planner.md
install: npx @ecomdly/cli add marginmath/free-shipping-threshold-planner
---

# Free-shipping threshold planner

Produces a recommendation for the free-shipping threshold and the shipping price below it: profit per order below and above the line, the profit effect of raising or lowering the threshold under scenarios the owner enters or the store has measured, the break-even behaviour that would make a change pay, and a plan for testing it. Copying a competitor's threshold or setting it "a bit above AOV" ignores what the store pays the carrier per parcel, how much contribution a basket actually carries, and how many orders sit just under the line. The key insight: the store already has its own evidence of how customers react to the threshold, in the shape of its basket-value histogram. An excess of orders just above the threshold and a gap just below it is customers topping up; its size, not a generic "conversion lift", is the starting point for any estimate, and everything beyond it is a scenario to be tested.

## When to use

- Setting a first free-shipping threshold, or reviewing one after a carrier price change.
- When AOV, the product mix or the margin has moved and the threshold has not.
- Before announcing a threshold change in ads, feeds or the site banner.
- Reading a threshold test or a before/after change.

## When not to use

- Deciding a one-off sale or coupon: `promotion-profit-check` (it also reruns the shipping line when a discount moves baskets across the threshold).
- Setting ROAS targets: `break-even-roas-calculator`; this skill feeds it the shipping line per order.
- Comparing total price with shipping against competitors: `competitor-price-brief`.
- Checkout usability of the shipping step: `checkout-friction-audit`.
- Statistical readout of a randomised test: `ab-test-readout`.

## Inputs

| input | definition | typical source |
|---|---|---|
| Order export | order ID, date, basket value on the basis the threshold uses (incl. VAT, after or before discounts), shipping charged, carrier, destination zone, parcel weight, returned yes/no | e-shop order export |
| Threshold rule | current threshold, its basis (incl. VAT, after discounts?), which carriers and countries it covers | checkout settings |
| Carrier cost per parcel | price per parcel by carrier, weight band and zone, plus surcharges (fuel, COD, oversize) | carrier price list, invoices |
| Shipping price charged | per carrier and zone, and the VAT rate applied to it | checkout settings, accountant |
| Product contribution margin | contribution of goods ex VAT after COGS, payment fees, pick/pack and returns, before shipping, per category | `break-even-roas-calculator`, store data |
| Order volume and period | at least several full months without a threshold change or a sitewide sale | order export |

Optional: past threshold changes with dates; cart-level data on threshold banners shown; returns per order value band; country split for cross-border shipping.

Behaviour shares for a change (top-up, pay shipping, abandon; new orders after a lowering) are either measured in a past change or test, or entered by the owner as named scenarios. If an input is missing, stop at that line and ask; never use an industry conversion lift or an average carrier rate.

## Best practices

### Measure the current state

1. **Use the threshold's own basis.** If the checkout compares the basket incl. VAT after discounts, build the histogram on exactly that value; otherwise the band "just under" is misplaced.
2. **Carrier cost per parcel, not per order average.** Weight and zone drive cost; a heavy-product category above the threshold can cost twice the average. Split by carrier and zone when costs differ.
3. **Profit per order below = product CM + shipping charged ex VAT − carrier cost; above = product CM − carrier cost.** Shipping charged rarely covers carrier cost plus handling fully; show the net per carrier.
4. **Read the bunching.** Compare order counts in equal-width bins just below and just above the threshold (e.g. 100 CZK bins). Excess orders above and a dip below indicate top-up behaviour; the excess count is the store's own upper bound for orders the threshold currently pulls up.
5. **Look at what top-up items are.** If top-ups are low-margin fillers, the extra basket value adds little contribution; use their real margin, not the store average.

### Model a change

6. **Raising the threshold** affects only orders between the old and the new value. Each either tops up (adds basket value at its margin), pays shipping, or is lost. Profit change = Σ over the three behaviours − current profit of those orders.
7. **Lowering the threshold** gives up shipping revenue on orders between the new and the old value, and may remove the reason current top-uppers buy more. It pays only if enough new or larger orders follow; compute that break-even number of orders.
8. **Solve for the break-even behaviour.** Instead of asserting a lift, compute the abandon share (for a raise) or the extra orders (for a lowering) at which profit is unchanged, and let the owner judge plausibility against the bunching data.
9. **Feed ad targets.** Free-shipped orders carry carrier cost without shipping revenue, so their contribution and break-even ROAS differ; pass the per-order shipping line to `break-even-roas-calculator`, split above and below the threshold where AOV sits near it.

### Test and legal

10. **Test with a pre-registered metric: contribution per visitor**, not conversion rate or AOV alone; a higher threshold usually raises AOV while it can lower conversion. Randomise by user so one person sees one rule; or, without a test tool, use a before/after with a matched prior period and say it is weaker.
11. **Show delivery charges before the order.** Directive 2011/83/EU Art. 6(1)(e) requires the total price and all delivery charges to be given before a distance contract binds the consumer. Threshold and charges must be clear on the product page and cart.
12. **Withdrawals refund standard delivery.** Under Art. 13(1)–(2) the trader refunds the outbound delivery cost, except supplementary costs of a delivery type more expensive than the cheapest standard one; under Art. 14(1) the consumer bears the direct cost of returning goods unless the trader agreed to bear it or failed to inform. Count refunded outbound shipping in returned orders.
13. **Personalised prices need disclosure.** Art. 6(1)(ea), inserted by Directive (EU) 2019/2161, requires informing consumers when a price is personalised on the basis of automated decision-making. Whether a randomised shipping-price test falls under it is not settled here; ask counsel in the target country before testing.

## Process

1. **Extract** matured orders (returns completed) for the period; drop test, B2B and marketplace orders unless the threshold applies to them.
2. **Histogram** on the threshold basis in equal bins; mark the threshold; count orders per bin and the bunching excess.
3. **Per band** (well below, just below, just above, well above): orders, average basket, product CM, shipping charged ex VAT, carrier cost, profit per order.
4. **Candidate thresholds**: the owner's options plus the current one.
5. **For each candidate**, apply the owner's behaviour scenarios to the affected band; compute profit change per month and the break-even behaviour share.
6. **Shipping price below the threshold**: compare charged price with carrier cost per carrier and zone; flag carriers sold at a loss.
7. **Recommend** keep, raise or lower, with the scenario range where it pays, and a test or before/after plan.
8. **After the change**: compare contribution per visitor, conversion, AOV and the new histogram with the baseline period; report against the break-even share computed in step 5.

## Pitfalls and edge cases

- **Discount codes and the basis.** A code applied after the threshold check gives free shipping to baskets that end below it; verify the order of checks in the checkout.
- **Partial withdrawals.** A customer can return items and keep a basket now under the threshold; whether shipping can be recharged depends on the store's terms and national law; ask counsel, do not assume.
- **Cross-border.** Carrier cost and VAT differ by country; run one histogram per country or currency.
- **Pickup points vs home delivery.** Different carrier costs; some stores offer free pickup-point delivery only. Model each option.
- **Seasonality.** Christmas baskets shift the histogram; do not compare a December test with a November baseline.
- **Feeds and ads.** A threshold change must be mirrored in Merchant Center shipping settings (which support free shipping over an order value and weight- or price-based rates) and comparison feeds the same day; the owner makes the change.
- **Small stores.** With few orders per bin the bunching is noise; widen bins and state the uncertainty.

## Rules

- Every number is store data, a carrier price list, or an owner-entered scenario labelled as such. No generic lift or conversion figures.
- Show the arithmetic and the break-even behaviour share for every candidate.
- Legal points are limited to the verified rules above; anything country-specific goes to counsel.
- The agent does not change checkout settings, shipping prices, Merchant Center or feeds, and does not start tests. It recommends; the owner acts.

## Output format

```
FREE-SHIPPING THRESHOLD · <store> · orders <from>–<to> · basis <incl/ex VAT, after/before discounts> · prepared <date>
Current threshold <x> · shipping charged <x> incl. VAT (<x> ex VAT) · carrier cost <x> per parcel (<carrier, zone, weight>)
Bunching: <bin below> <n> orders · <bin above> <n> orders · excess <n>
| band | orders | avg basket | product CM | shipping ex VAT | carrier | profit/order |
|---|---|---|---|---|---|---|
Candidate <x>: affected orders <n> · scenario <name, source>: top-up <x>% / pay <x>% / lost <x>%
  profit now <x> → after <x> · change <x>/month · break-even <lost share or extra orders>
Shipping price check: <carrier, zone> charged <x> vs cost <x>
Recommendation: KEEP / RAISE / LOWER to <x> · test plan <metric, split, duration> · ROAS line to break-even-roas-calculator
Missing inputs: <list or none>
```

## Worked example

Illustrative numbers only. Czech store, CZK, VAT 21% (on goods and, per the store's accountant, on shipping). Threshold 1 500 incl. VAT after discounts; shipping charged 99 incl. VAT = 81.82 ex VAT; carrier cost 85 ex VAT per parcel; product CM 30% of basket ex VAT; 4 000 orders per month.

```
Bunching: 1 400–1 499 280 orders · 1 500–1 599 520 orders → clear top-up behaviour
| band | orders | avg basket | product CM | shipping ex VAT | carrier | profit/order |
|---|---|---|---|---|---|---|
| < 1 000 | 1 200 | 700 | 173.55 | 81.82 | 85.00 | 170.37 |
| 1 000–1 499 | 1 000 | 1 250 | 309.92 | 81.82 | 85.00 | 306.74 |
| 1 500–1 999 | 1 100 | 1 650 | 409.09 | 0.00 | 85.00 | 324.09 |
| ≥ 2 000 | 700 | 2 800 | 694.21 | 0.00 | 85.00 | 609.21 |
Candidate 2 000 (raise): affected 1 100 orders
  scenario "owner estimate", not measured: top-up 30% to avg 2 050 / pay 55% / lost 15%
  top-up 330 × (2 050 ÷ 1.21 × 0.30 − 85) = 330 × 423.26 = 139 677
  pay    605 × (409.09 + 81.82 − 85) = 605 × 405.91 = 245 575
  lost   165 × 0 = 0
  profit now 1 100 × 324.09 = 356 500 → after 385 252 · change +28 752/month
  break-even: with top-up at 30%, lost share 21.4% makes the change zero
Candidate 1 000 (lower): 1 000 orders stop paying 81.82 → −81 818/month
  break-even: 81 818 ÷ (309.92 − 85) = 364 extra orders of 1 250 per month, before any loss of current top-ups
Recommendation: test 2 000 against 1 500, metric contribution per visitor, randomised by user, at least two full weeks; counsel to confirm the test is not a personalised price
```

Check: 1 200 + 1 000 + 1 100 + 700 = 4 000 orders. 1 650 ÷ 1.21 × 0.30 = 409.09. Break-even lost share a: 139 677 + (770 − 1 100a) × 405.91 = 356 500 gives a = 0.214. The 280 and 520 bins are inside the 1 000–1 499 and 1 500–1 999 bands.

## Quality checklist

- Histogram on the threshold's real basis; bunching counted in equal bins.
- Carrier cost per parcel by carrier, zone and weight; shipping charged ex VAT.
- Profit per order shown below and above the threshold.
- Every behaviour share labelled measured or scenario; break-even share computed.
- Refunded outbound shipping on withdrawals counted.
- ROAS implications handed to `break-even-roas-calculator`.
- Test metric is contribution per visitor, pre-registered, randomised by user.
- No setting changed by the agent.

## Sources

- EUR-Lex, Directive 2011/83/EU (Consumer Rights), Art. 6(1)(e), 13(1)–(2), 14(1): https://eur-lex.europa.eu/eli/dir/2011/83/oj
- EUR-Lex, Directive (EU) 2019/2161, Art. 4 inserting Art. 6(1)(ea) into Directive 2011/83/EU: https://eur-lex.europa.eu/eli/dir/2019/2161/oj
- Google Merchant Center Help, Set up shipping settings for an entire account (free shipping over an order value, rates by weight or price): https://support.google.com/merchants/answer/6069284

## License
MIT
