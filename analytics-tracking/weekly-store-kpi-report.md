---
name: weekly-store-kpi-report
owner: shopmetric
category: Analytics & tracking
description: Builds a one-page weekly store report — revenue, orders, AOV, conversion rate, ad spend, MER and returns — against last week and last year, stating any missing source.
version: v2
license: MIT
updated: 2026-09-22
recommended: false
security_checked: true
url: https://ecomdly.com/skills/shopmetric/weekly-store-kpi-report
raw: https://ecomdly.com/raw/shopmetric/weekly-store-kpi-report.md
install: npx @ecomdly/cli add shopmetric/weekly-store-kpi-report
---

# Weekly store KPI report

Owners do not need thirty charts on Monday. They need ten numbers, what moved, and whether anything needs a decision. This skill writes that page.

## When to use
- Every Monday for the previous Monday–Sunday week.
- After a promotion, with the promo week marked.

## Input
Backend orders for the week, the prior week and the same week last year (net revenue, orders, refunds, new vs returning customers). GA4 sessions and purchase events. Ad spend per channel (Google Ads, Meta, others). Optional: stock-outs of top sellers, promotions calendar.

## Metrics
- **Net revenue** (backend, ex VAT, after refunds) and **orders**.
- **AOV** = net revenue ÷ orders.
- **Conversion rate** = orders ÷ sessions (GA4 sessions; say so).
- **New customer share** = first orders ÷ orders.
- **Ad spend** total and per channel; **MER** = net revenue ÷ total ad spend. Platform ROAS is shown alongside but never summed across channels, because platforms double-count.
- **Refund rate** = refunded value ÷ gross revenue.
- **Top 5 products** by revenue, with any that were out of stock.

## Rules
- Each number states its source (backend, GA4, platform). Revenue is always backend.
- A week-over-week change under ±5% on fewer than 100 orders is called noise, not a trend.
- Compare to the same week last year only when both weeks have the same promotions and holidays; otherwise say why the comparison is skipped.
- Missing input means a blank cell with the reason, never an estimate.
- Close with at most two items that need a decision; if none, say "No action needed".

## Output format
```
## Week 38 (15–21 Sep 2026)
| metric | this week | vs last week | vs last year |
| Net revenue | 48 920 | +6.2% | +14.1% |
| Orders | 612 | +3.9% | +9.3% |
| AOV | 79.93 | +2.2% | +4.4% |
| Conversion rate (GA4) | 2.31% | +0.12 pt | n/a (GA4 migrated) |
| Ad spend | 9 840 | +11.0% | +18.5% |
| MER | 4.97 | −0.22 | −0.19 |
| Refund rate | 6.8% | +0.4 pt | — |
Needs a decision: Meta spend +31% with flat new-customer share (38%); review budget.
```

## License
MIT
