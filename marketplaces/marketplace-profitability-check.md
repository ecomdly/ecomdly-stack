---
name: marketplace-profitability-check
owner: listwise
category: Marketplaces
description: Calculates per-SKU, per-marketplace contribution on Amazon EU, Allegro and Kaufland after commission, fulfilment, storage, ads, returns and FX, on the right VAT basis, and decides keep, reprice, change fulfilment or delist.
version: v1
license: MIT
updated: 2026-10-05
recommended: false
security_checked: true
url: https://ecomdly.com/skills/listwise/marketplace-profitability-check
raw: https://ecomdly.com/raw/listwise/marketplace-profitability-check.md
install: npx @ecomdly/cli add listwise/marketplace-profitability-check
---

# Marketplace profitability check

Produces a contribution statement for every SKU on every marketplace it sells on (Amazon EU stores, Allegro, Kaufland), and one decision per SKU and channel: keep, reprice, change fulfilment or delist. The naive check (price minus COGS minus "the commission %") flatters marketplace sales because the commission is a percentage of the gross price the buyer pays, VAT and buyer-paid shipping included, while the seller keeps only the net price; on top of that come minimum fees, storage that grows with every month of unsold stock, ad spend that was bought to make the SKU sell, and returns that cost a fee and sometimes the goods. The key insight: commission is computed on the gross amount, contribution is computed on the net amount, and every cost must be expressed per unit ordered on that same net basis, with returns and ads included, before two channels or two fulfilment methods can be compared.

## When to use
- Quarterly, or after a marketplace fee change, on the settlement or fee reports of the period.
- Before sending more stock to FBA, or when an aged-inventory or storage charge appears on the invoice.
- When a SKU sells well on a marketplace but the payout does not show it, or ad spend on a marketplace rises.
- Before listing an existing SKU on a further marketplace or EU Amazon store.

## When not to use
- Building the listings themselves: `marketplace-listing-adapter`.
- Late dispatch, cancellations, defect rate or a suspension warning: `marketplace-account-health-triage`.
- Target ROAS for your own store's Google Ads: `break-even-roas-calculator`.
- Choosing the new price against competitor offers: `competitor-price-brief`; for clearing aged stock with markdowns: `clearance-markdown-planner`.
- Analysing why products come back: `returns-reason-analyzer`.

## Inputs
Per SKU, per marketplace (and per Amazon store, because fees and VAT differ by country):

| input | definition | typical source |
|---|---|---|
| Gross price G and buyer-paid shipping Sh | what the buyer pays, incl. VAT, per unit | order or settlement report |
| VAT rate v | rate of the delivery country under the seller's VAT setup (OSS, local registration) | accountant, VAT registrations |
| Commission rate and minimum fee | the category's referral fee / commission % and any per-item minimum or fixed fee | seller's fee report, the platform's current fee page |
| Commission refund on returns | what the platform gives back when an order is refunded, and what it keeps | platform fee page, settlement lines |
| Fulfilment per unit | FBA fulfilment fee, or own pick/pack + packaging + carrier, ex VAT | fee preview report, 3PL and carrier invoices |
| Storage per unit | monthly storage fee per unit × months of cover, plus any aged-inventory surcharge | storage fee and inventory age reports |
| Marketplace ad spend | spend on ads advertising the SKU in the period | Sponsored Products, Allegro Ads, Kaufland ads reports |
| Units ordered | all units of the SKU sold in the period, organic and ad-driven | business or sales report |
| Return rate r and unsellable share u | units refunded ÷ units ordered on a matured cohort; share of returned units that cannot be resold | returns report, warehouse grading |
| Return handling cost | return shipping paid by the seller, inspection, repackaging per return | carrier invoices, 3PL |
| COGS | landed cost per unit, ex VAT | ERP, purchase prices |
| FX | payout currency, conversion rate and conversion fee or spread | payout reports, bank or platform converter |
| Owner's floor | minimum acceptable contribution margin per SKU | owner |

Optional: fixed subscription fees (shown below the SKU lines, not allocated into them), removal or disposal fees, the cost of the alternative fulfilment method for the comparison column.

If an input is missing, stop at that line, show the formula with a named blank, and ask. Never substitute an industry average or a fee rate remembered from elsewhere: rates differ by category and country and change often.

## Best practices

### Get the basis right
1. **Commission base is gross; contribution base is net.** Amazon states that the referral fee is a percentage of the total price, applied to the final price including shipping costs, VAT and gift-wrapping charges, or a minimum amount, whichever is greater. Allegro calculates commission on the final selling price plus the delivery cost when the buyer pays it. Kaufland calculates commission on the gross sales price including delivery charges. Revenue kept, however, is (G + Sh) ÷ (1 + v). Computing the fee on the net price understates it by the VAT rate.
2. **Apply the minimum fee per unit.** Amazon charges the percentage or the minimum per item, whichever is greater; on cheap items the minimum, not the rate, decides the fee. Take the minimum per category from the current fee page.
3. **One country at a time.** VAT, fees, carrier costs and prices differ by Amazon store and Allegro or Kaufland country. Never average across countries before computing contribution.

### Count every cost per unit ordered
4. **Storage is a cost of slow sellers.** Storage per unit = monthly fee per unit × months of cover (stock ÷ monthly sales). Amazon charges additional storage fees for units stored past an age threshold; read the threshold and rate from the fee page and the inventory age report. A SKU with nine months of cover pays nine months of storage on every unit.
5. **Ads belong to the SKU they advertise.** Ad cost per unit = ad spend on the SKU ÷ all units ordered of that SKU. Show contribution before and after ads, because a SKU that is positive only before ads is bought, not earned. If the ad report credits sales of other SKUs to the ad, keep them separate rather than inflating this SKU.
6. **Returns cost four ways:** lost revenue, the part of the commission the platform keeps (Amazon refunds the referral fee minus a refund administration fee of the lesser of EUR 5.00 or 20% of the referral fee), handling, and the COGS of units that cannot be resold. Allegro grants a transaction rebate on returned items when its conditions are met; confirm eligibility in the settlement lines.
7. **FX is a cost.** If the payout is converted (PLN to EUR, CZK to EUR), apply the conversion fee or spread to the payout, and convert all lines at one rate and date basis.
8. **Fixed fees stay out of the SKU line.** Subscriptions are covered by total contribution; allocating them per unit makes small SKUs look worse than they are.

### Decide
9. **Keep** if contribution margin after ads ≥ the owner's floor.
10. **Reprice** if contribution is positive but below the floor, or below zero only because of ads, and the price has room. Recompute the fees at the new price: they move with it.
11. **Change fulfilment** if the gap is driven by fulfilment or storage: compute the alternative (FBA vs own, smaller inbound batches) as a second column. Moving to own fulfilment brings the seller-fulfilled performance metrics into play; check with `marketplace-account-health-triage`.
12. **Delist** if contribution is negative at any price the market accepts with every fulfilment option, and stop replenishment first. Removal or disposal fees go into the decision, not the past.

## Process
1. **Collect** the period's settlement, fee, storage, inventory age, ads, returns and order reports per marketplace and country. Record the period and the fee page dates used.
2. **Match** SKUs across channels (SKU, ASIN, Allegro offer ID, Kaufland EAN); flag unmatched lines.
3. **Per SKU and channel, compute per unit ordered**, rounding each line to cents:
   - Revenue kept = (1 − r) × (G + Sh) ÷ (1 + v)
   - Commission = (1 − r) × max(rate × commission base, minimum) + r × commission retained on a refund
   - Fulfilment = fee or own cost per unit shipped
   - Storage = monthly fee per unit × months of cover + aged surcharge per unit
   - Ads = SKU ad spend ÷ units ordered
   - Return handling = r × cost per return
   - COGS = COGS × (1 − r × (1 − u))
   - FX = conversion fee × payout (gross kept − commission)
   - CM = revenue kept − all cost lines; margin = CM ÷ ((G + Sh) ÷ (1 + v))
4. **Compute CM before ads** and the alternative fulfilment column where relevant.
5. **Apply the decision rules** 9–12 and write the reason in one line per SKU.
6. **Sensitivity:** recompute with r + 5 points and with storage at current cover + 3 months, to show which SKUs flip.
7. **Report** with the template below; fixed fees listed separately.

## Pitfalls and edge cases
- **Fee reports are per order, not per unit.** Multi-unit orders and promotions split differently; divide by units, not lines.
- **Promotions and coupons** lower the price the buyer pays and so the commission base; use the actual G, not the list price.
- **Free delivery programmes** (for example Allegro Smart!) remove buyer-paid delivery from the commission base but leave the carrier cost with the seller or the programme fee.
- **Pan-EU or cross-border fulfilment** changes the VAT country and sometimes the fee; compute per delivery country.
- **Immature returns.** Orders from the last weeks have not finished returning; use a matured cohort or label r as provisional.
- **B2B orders** may be invoiced without VAT; keep them separate from consumer orders.
- **Inventory valuation.** COGS of returned unsellable units is a real loss even if the write-off is booked later.

## Rules
- Every rate and fee comes from the seller's own fee reports or the platform's current fee page, with the date read. No remembered rates.
- Show the arithmetic per line, not only the result.
- State the VAT basis and country in each SKU block.
- Output is advice. The agent does not change prices, delist offers, create removal orders, change ad budgets or switch fulfilment; the owner decides.

## Output format
```
Marketplace profitability — <seller> — period <dates> — fee pages read <date> — prepared <date>
Owner's floor: <x>% contribution margin after ads

SKU <id> · <marketplace/country> · <FBA|own> · VAT <v>% · currency <cur>
  Revenue kept (1−r)(G+Sh)/(1+v)        <x>
  Commission (base <gross>, min <x>)    <x>
  Fulfilment                            <x>
  Storage (<n> months cover + aged)     <x>
  Ads (<spend> ÷ <units>)               <x>
  Return handling                       <x>
  COGS net of resold returns            <x>
  FX                                    <x>
  CM / margin                           <x> / <x>%   (before ads <x>)
  Alternative <fulfilment>              <x> / <x>%
  Decision: <keep|reprice|change fulfilment|delist> — <reason>

| SKU | channel | CM | margin | before ads | r+5 pts | decision |
|---|---|---|---|---|---|---|
Fixed fees (not allocated): <list>
Missing inputs: <list or none>
```

## Worked example
Illustrative numbers only; fee rates are invented for the arithmetic and are not current rates. Czech seller, owner's floor 15%.

```
SKU A · Amazon.de · FBA · VAT 19% · EUR
  G 39.90, no shipping charge; referral 15% (min 0.30); r 8%, u 25%; COGS 11.00
  Revenue kept 0.92 × 39.90 ÷ 1.19 = 0.92 × 33.53      30.85
  Commission 0.92 × 5.99 + 0.08 × 1.20                  5.61
    (15% × 39.90 = 5.99; refund admin fee = min(5.00, 20% × 5.99) = 1.20)
  FBA fulfilment fee                                     4.20
  Storage 0.35 × 2 months                                0.70
  Ads 1 200 ÷ 400 units                                  3.00
  Return handling                                        0.00
  COGS 11.00 × (1 − 0.08 × 0.75)                        10.34
  CM 30.85 − 5.61 − 4.20 − 0.70 − 3.00 − 10.34           7.00 → 20.9% of 33.53
  Decision: keep (20.9% ≥ 15%)

SKU A · Allegro.pl · own fulfilment · VAT 23% · PLN (1 EUR = 4.25 PLN)
  G 169.00 + buyer-paid delivery 12.99 = 181.99; commission 10%; rebate on returns; r 5%, u 20%
  Revenue kept 0.95 × 181.99 ÷ 1.23 = 0.95 × 147.96    140.56
  Commission 0.95 × 18.20                               17.29
  Fulfilment pick/pack 4.00 + carrier 14.00             18.00
  Return handling 0.05 × 10.00                           0.50
  Ads 2 000 ÷ 250 units                                  8.00
  COGS 46.75 × (1 − 0.05 × 0.80)                        44.88
  FX 1% × (0.95 × 181.99 − 17.29) = 1% × 155.60          1.56
  CM 140.56 − 17.29 − 18.00 − 0.50 − 8.00 − 44.88 − 1.56 = 50.33 PLN = 11.84 EUR → 34.0%
  Decision: keep
```

The basis trap on SKU A at Amazon.de: 15% of the net 33.53 would be 5.03, so a fee computed on the net price understates it by 0.96 per kept unit.

SKU B, Amazon.de FBA: G 14.90 (net 12.52), referral 2.24, FBA 3.60, storage 0.30 × 9 months + aged surcharge 0.50 = 3.20, ads 0.80, COGS 4.50, r 3%, u 50%. Revenue kept 12.14; commission 0.97 × 2.24 + 0.03 × 0.45 = 2.19; COGS 4.43. CM = 12.14 − 2.19 − 3.60 − 3.20 − 0.80 − 4.43 = −2.08. With one month of cover (storage 0.30) CM would be 0.82 (6.5%). Decision: change fulfilment plan: stop inbound shipments, compare removal fee with further storage, then reprice or delist if 6.5% cannot be lifted to 15%.

## Quality checklist
- Commission computed on the gross base the platform uses, contribution on the net price.
- Minimum fee checked per unit.
- Storage includes months of cover and any aged surcharge.
- Ads divided by all units ordered; CM shown before and after ads.
- Returns split into revenue, retained fee, handling and unsellable COGS.
- One country and currency per block; FX applied once.
- Every rate traced to a dated fee report or page; missing inputs listed.
- Every SKU has one decision with a one-line reason.

## Sources
- Amazon, Pricing (Amazon.de: referral fee on total price incl. shipping and gift wrap, minimum fee, storage and aged-inventory fees, refund administration fee): https://sell.amazon.de/en/preisgestaltung
- Amazon, How much does it cost to sell on Amazon? (Amazon.es: percentage applied to the final price including shipping costs, VAT and gift-wrapping charges): https://sell.amazon.es/en/precios
- Allegro, How is the sales commission calculated: https://help.allegro.com/en/sell/a/how-is-the-sales-commission-calculated-lL5oW6EW7c2
- Kaufland Global Marketplace, Conditions and fees: https://www.kauflandglobalmarketplace.com/en/conditions/

## License
MIT
