---
name: returns-reason-analyzer
owner: helpdeskly
category: Customer care
description: Codes every return to a root cause with evidence, separates withdrawals from warranty and transit damage, flags SKUs with statistically real return problems, and routes each fix to its owner.
version: v3
license: MIT
updated: 2026-09-29
recommended: false
security_checked: true
url: https://ecomdly.com/skills/helpdeskly/returns-reason-analyzer
raw: https://ecomdly.com/raw/helpdeskly/returns-reason-analyzer.md
install: npx @ecomdly/cli add helpdeskly/returns-reason-analyzer
---

# Returns reason analyzer

Produces a monthly returns diagnosis: every return coded to a root cause with a quoted evidence line, return rates per SKU that are statistically meaningful, a ranking by what the returns cost, and a fix routed to the team that can make it (product page, size chart, packing, supplier, warehouse). "Didn't fit" is not a reason, it is a symptom; and a raw return-rate ranking fails because small SKUs top it by chance, returns lag sales, and warranty claims, transit damage and withdrawals get mixed into one number although each has a different owner and a different legal footing.

## When to use
- Monthly or quarterly, on the returns export for the period.
- When a SKU's return rate jumps after a new batch, supplier, or page change.
- Before rewriting product pages or size charts, to know which fact is missing.

## When not to use
- Deciding a single customer's refund or claim. That is a support decision; see `order-status-reply-drafter` for customer replies.
- Pricing out overstock from returned goods: hand the list to `clearance-markdown-planner`.
- Fewer than about 30 returns in the period: report the coded list without rates and say so.

## Inputs
Required:
- **Returns export**: return ID, order ID, order date, return request date, SKU (variant level), units, return type if recorded (withdrawal / warranty claim / transit damage / exchange), reason picked in the form, customer comment, warehouse inspection note and grade (resellable / B-stock / write-off). Shopify: Returns or Refunds report; Shoptet: Reklamace and Vrácení zboží modules; or the returns platform export.
- **Sales**: units shipped per SKU (variant level) by order date for the same and the preceding period.
- **Product page facts per SKU**: size chart present (yes/no), garment or product measurements in cm, number of photos, material composition stated, colour name, supplier and batch if tracked.

Optional:
- Cost per return: reverse shipping, handling time cost, refurbishment, write-down share by grade.
- Exchanges (same product, other size) if they are separated from refunds.
- Carrier per shipment (for damage patterns).

If inspection notes are missing, code from comments only and mark the confidence as low. If sales are only available at product level, not variant level, say that size-specific rates cannot be computed.

## Best practices

### Separate the legal channels first
1. **Withdrawal, warranty, transit damage and wrong item are different things.** Under Directive 2011/83/EU Art. 9 an EU consumer may withdraw from a distance purchase within 14 days of receiving the goods "without giving any reason"; these are returns of conforming goods. Under Directive (EU) 2019/771 the seller is liable for lack of conformity that becomes apparent within two years of delivery (Art. 10), with a presumption that defects appearing within the first year existed at delivery (Art. 11; some countries use two years). Damage in transit is the trader's risk until the customer takes physical possession (2011/83/EU Art. 20). Report each channel separately; a defect rate buried in "returns" hides a supplier problem.
2. **Do not force a reason out of withdrawals.** Because no reason is needed, a mandatory reason field is questionable and produces junk answers ("other"). Treat the picked reason as a hint, not evidence.

### Code with evidence and precedence
3. **Precedence: warehouse inspection note > customer comment > picked reason.** Inspection sees the product; the picked reason is often the first option in the dropdown.
4. **One primary code, optional secondary.** "Too small and the colour was off" is SIZE_SMALL primary, NOT_AS_DESCRIBED secondary. Rates use primary codes only so they add to 100%.
5. **Keep a verbatim quote** (short, personal data removed) for each coded return; it is the evidence for the fix.
6. **UNCLEAR is a valid code.** Never force a code on "doesn't suit me" or an empty comment; a high UNCLEAR share is itself a finding about the return form.

### Compute rates that mean something
7. **Cohort by order date, not return date.** Return rate = units returned from orders placed in the period ÷ units shipped in the period. Mixing return-date returns with order-date sales inflates rates after sales peaks and deflates them during peaks.
8. **Let the cohort mature.** Exclude orders from the last 14 days plus typical delivery and return-transit time (for most EU stores, about 30 days) or report them as provisional; the withdrawal window has not closed.
9. **Minimum volume and uncertainty.** Report SKUs with at least 20 units shipped. Flag a SKU only when its rate is ≥ 1.5 × its category's rate **and** the lower bound of its 95% Wilson interval is above the category rate. Wilson lower bound with p = returned ÷ shipped, n = shipped, z = 1.96: (p + z²/2n − z·√(p(1−p)/n + z²/4n²)) ÷ (1 + z²/n). This stops a SKU with 2 returns out of 6 from leading the report.
10. **Bracketing is its own pattern.** When one order contains the same product in two or more sizes and one is returned, code the return BRACKETING (with the size kept) rather than SIZE_*. It signals low confidence in the size chart, not a wrong cut.
11. **Rank by cost, not only by rate.** Return cost per SKU = returned units × (reverse shipping + handling + write-down by grade). A 9% rate on a EUR 300 jacket can cost more than 30% on a EUR 8 T-shirt.

### Fix routing
12. **Each code has an owner and a standard fix:**
    - SIZE_SMALL / SIZE_LARGE / FIT → product content: garment measurements in cm, a data-backed fit note ("runs small, 19 of 31 returns"), fit photos on a model with stated height and size.
    - BRACKETING → size advice tool or chart clarity; check if chart uses body or garment measurements and says which.
    - NOT_AS_DESCRIBED → product content and photography: colour shot in daylight with a neutral reference, material percentages, dimensions; hand to `product-description-writer` or `product-page-cro-review`.
    - DEFECT (inspection-confirmed) → purchasing/supplier with batch numbers; these are warranty cases, not page problems.
    - DAMAGED_TRANSIT → operations: packing spec, carrier claim pattern by carrier and route.
    - WRONG_ITEM → warehouse: picking errors by location or look-alike SKUs.
    - LATE_DELIVERY → operations and delivery-promise text.
    - CHANGED_MIND / PRICE (found cheaper) → usually no product fix; watch pricing via `competitor-price-brief`.
13. **Make the fix measurable.** Record the change date; compare the next matured cohort with the same length pre-change cohort, same season if possible.

## Process
1. **Load and clean**: de-duplicate return lines, map variants to SKUs, drop test orders, attach order date and shipped units.
2. **Split by channel**: withdrawal, warranty claim, transit damage, wrong item, exchange. If return type is not recorded, infer from inspection note and timing (a claim after 14 days of receipt is almost certainly not a withdrawal) and label as inferred.
3. **Code each return** with precedence rule 3; store primary code, secondary code, confidence (high: inspection-confirmed; medium: clear comment; low: picked reason only), and quote.
4. **Compute per SKU** on the matured cohort: shipped, returned, rate, Wilson lower bound, category rate, code mix, cost.
5. **Flag** SKUs meeting rule 9. Name the dominant code (≥ 40% of the SKU's returns); if none reaches 40%, report the top two.
6. **Check the page facts** for flagged SKUs: is the fact the customers complain about missing from the page? That is the fix.
7. **Rank** flagged SKUs by return cost; write the fix and the owner.
8. **Summarise the store level**: rate by channel and code, UNCLEAR share, defect rate by supplier, damage rate by carrier.
9. **Run the quality checklist.**

## Pitfalls and edge cases
- **Exchanges counted as returns** double-count size problems if the replacement is also returned; follow the chain by order ID.
- **Partial returns of multi-packs**: count units, not lines.
- **New SKUs** with fewer than 20 units: list them under "watch" with raw counts.
- **Category averages dominated by one SKU**: compute the category rate excluding the SKU being tested.
- **Batch changes**: a jump after a new batch points to the supplier, not the page; split rates by batch if the data allows.
- **Marketplace returns** (Allegro, Amazon, Kaufland) follow the marketplace's rules and reasons lists; keep them separate from web store returns.
- **Personal data in comments**: names, addresses, phone numbers. Strip them from quotes.
- **Serial returners**: a few customers can drive a SKU's rate. Report the share of returns from customers with ≥ 3 returns in the period, but do not name individuals in the report.
- **Inspection says "no fault found" on a defect claim**: code as UNCLEAR with note; it may be user error or an intermittent fault, and the claim decision belongs to support.

## Rules
- Read-only: produce the analysis and recommended fixes; never edit product pages, change policies or contact customers.
- Never invent a reason, a quote, a cost or a sales figure; missing data is flagged with its effect.
- Quotes are verbatim and anonymised.
- Legal statements about withdrawal or warranty are limited to the verified rules above; anything country-specific is a check for the store's counsel.
- Supplier escalations and packing changes are recommendations for a human owner.

## Output format
```
RETURNS ANALYSIS · <store> · orders <from>–<to> (matured) · provisional after <date>
Store level: shipped <n> · returned <n> (<x>%)
  by channel: withdrawal <x>% · warranty <x>% · transit damage <x>% · wrong item <x>% · exchange <x>%
  by code: <code> <x>% · ... · UNCLEAR <x>%
Flagged SKUs (rate ≥ 1.5× category and Wilson lower bound > category), ranked by cost:
<SKU name variant> — shipped <n>, returned <n> (<x>%, Wilson LB <x>%, cat. <x>%) · cost <amount>
  dominant: <CODE> <k>/<n> — "<quote>", "<quote>"
  page check: <fact present/missing>
  fix: <action> · owner: <team>
Watch list (< 20 units): <SKU> <returned>/<shipped>
Supplier/carrier notes: <defect rate by supplier/batch> · <damage by carrier>
Data gaps: <list>
```

## Worked example
Illustrative apparel store, orders 1–31 July, matured on 31 August; returns cost EUR 7.80 per unit (reverse shipping 4.20, handling 2.10, average write-down 1.50).

```
RETURNS ANALYSIS · LinenLab · orders 2026-07-01–2026-07-31 (matured) · provisional after 2026-08-01
Store level: shipped 4 820 · returned 612 (12.7%)
  by channel: withdrawal 10.9% · warranty 0.8% · transit damage 0.6% · wrong item 0.4% · exchange n/a
  by code: SIZE_SMALL 3.1% · FIT 2.2% · BRACKETING 1.9% · NOT_AS_DESCRIBED 1.6% · CHANGED_MIND 1.8% · DEFECT 0.8% · DAMAGED_TRANSIT 0.6% · WRONG_ITEM 0.4% · UNCLEAR 0.3%
Flagged SKUs, ranked by cost:
SKU 3302 Linen shirt M — shipped 140, returned 31 (22.1%, Wilson LB 16.1%, cat. 11.0%) · cost EUR 242
  dominant: SIZE_SMALL 19/31 — "tight across shoulders", "order one size up"
  page check: size chart gives body sizes only; no garment chest or shoulder width
  fix: add chest and shoulder width in cm; fit note "runs small: size up" · owner: product content
Watch list (< 20 units): SKU 3417 Linen trousers XS 4/12
Supplier/carrier notes: DEFECT concentrated in batch 2606 of SKU 3350 (7 of 9 claims, seams)
Data gaps: exchanges not separated from refunds; 15 returns without comment coded UNCLEAR (4 of them on SKU 3302)
```

Check: 31 ÷ 140 = 22.1%; Wilson lower bound with n = 140, p = 0.221 is 16.1%, above 11.0%; 22.1 ÷ 11.0 = 2.0× ≥ 1.5×; cost 31 × 7.80 = EUR 241.80. Channel shares sum to 12.7%, code shares sum to 12.7%.

## Quality checklist
- Withdrawals, warranty claims, transit damage and wrong items are reported separately.
- Rates use order-date cohorts and exclude unmatured orders (or mark them provisional).
- Every flag meets both the 1.5× and the Wilson test; SKUs under 20 units are only on the watch list.
- Every coded return has a confidence level; every dominant code has at least two anonymised quotes.
- Code shares add up to the store return rate.
- Each fix names a missing page fact or an operational cause and an owner.
- Ranking is by cost where cost inputs exist, otherwise stated as rate-only.
- No personal data in the output.

## Sources
- Directive 2011/83/EU (Consumer Rights), Art. 9, 13, 14, 20: https://eur-lex.europa.eu/eli/dir/2011/83/oj
- Directive (EU) 2019/771 (Sale of Goods), Art. 10, 11, 13, 14: https://eur-lex.europa.eu/eli/dir/2019/771/oj
- Your Europe, Guarantees and returns: https://europa.eu/youreurope/citizens/consumers/shopping/guarantees-returns/index_en.htm
- Wilson, E. B. (1927), Probable inference, the law of succession, and statistical inference, Journal of the American Statistical Association 22(158): https://doi.org/10.1080/01621459.1927.10502953

## License
MIT
