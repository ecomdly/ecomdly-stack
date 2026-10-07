---
name: weekly-kpi-slides-deck
owner: launifycorp
category: Analytics & tracking
description: You own the weekly rhythm artifact: a 9slide spine (hard cap 14 including appendix) that tells the store's owner, head of growth and performance buyer what moved last week, why, and what gets decided...
version: v1
license: MIT
updated: 2026-10-07
recommended: false
security_checked: true
url: https://ecomdly.com/skills/launifycorp/weekly-kpi-slides-deck
raw: https://ecomdly.com/raw/launifycorp/weekly-kpi-slides-deck.md
install: npx @ecomdly/cli add launifycorp/weekly-kpi-slides-deck
---

# Weekly KPI Review Slides

You own the weekly rhythm artifact: a 9-slide spine (hard cap 14 including appendix) that tells the store's owner, head of growth and performance buyer what moved last week, why, and what gets decided this week. You pull numbers from the shop backend and analytics sources available to you, normalise them to one money convention (net of VAT, net of returns, shipping revenue excluded), build the deck in Google Slides when that connector is live, and hand over a link plus a written summary of at most three decisions the deck forces.

The judgement call that separates good from mediocre: a mediocre deck reports every metric that exists; a good deck fixes 8 headline KPIs for the quarter, states a variance threshold for each, and spends slides 7 onward only on the ones that breached. Everything else goes to the appendix table nobody presents. If you cannot explain a movement, write "unexplained" on the slide with the list of cuts you checked — never backfill a plausible-sounding cause. The deck's credibility dies the first time a stated reason turns out to be invented in the meeting, and it does not come back.

## When to use

- Monday or Tuesday morning request: "prepare the weekly KPI review for last week" ahead of a standing trading meeting. Deck due at least 2 hours before the meeting starts; begin no later than T−3h.
- A fresh weekly export lands — GA4 report, Shopify/WooCommerce sales summary, Meta/Google Ads spend — and someone asks for "the deck".
- A seasonal campaign or sale week just closed (Black Friday, back-to-school, Vánoce) and leadership wants WoW, vs-4wk and vs-plan comparison in presentation form.
- The previous week's deck exists and you are asked to produce the next instalment with the same metric set and carried-forward actions.
- A new channel, market or product line launched in the last 4 weeks and the weekly review must be extended to cover it from its first full ISO week.

**Do not use this when:**

- The ask is a one-off deep dive into a single anomaly ("why did AOV drop 18% on Thursday?") — that is root-cause analysis over cohorts, segments and session recordings; write an investigation note, not a 9-slide spine.
- The ask is a monthly or quarterly business review with P&L, contribution margin by SKU, inventory cover and forecast revision — different audience, different artifact; build an MBR document.
- The ask is to fix tracking (consent mode v2, server-side GTM, Meta CAPI dedup) — that is a measurement implementation job. The deck only flags that data is unreliable and quantifies the gap.
- Fewer than 5 of the 7 days in the period have complete order data — reschedule the review rather than publish a part-week.

## Inputs

| Input | Required | If missing |
|---|---|---|
| Reporting period definition (week start/end, timezone, calendar vs ISO week) | Yes | Default to ISO week (Mon 00:00 – Sun 23:59), store's local timezone, print the convention on slide 1 and tag it `[assumption]`. |
| Order/revenue data: orders, gross order value, discounts, returns/refunds, shipping revenue, VAT rate(s) | Yes | Stop and ask. Without order-level or at least aggregated net revenue there is no deck; do not substitute GA4 purchase revenue silently. |
| Traffic and conversion data (sessions, CVR, channel split, funnel steps) | Yes | Build slides 1, 2, 3, 6, 8, 9 only; stamp the funnel and channel slides "data unavailable — GA4 export missing" and list the exact report needed. |
| Paid media spend by channel, ex-VAT, with the invoice VAT treatment stated | Preferred | Report revenue-only KPIs; MER/ROAS/CAC rows get "not computable", never an estimate. |
| Targets/plan for the week (revenue, CAC, CVR) | Preferred | Compare against trailing 4-week average and same ISO week last year; label the column "vs 4wk avg", never "vs plan". |
| Previous week's deck and its action list | Preferred | Open a fresh action register, state "carry-forward status unknown — no prior deck supplied" on slide 9. |
| Promo calendar and price history (for Omnibus 30-day low) | Preferred if a discount ran | Omnibus check = "not verified"; name the SKU count that could not be checked. |
| Google Slides connector availability | No | Deliver the deck as paste-ready Markdown slide blocks and state the connector was unreachable. |

Rank inputs: net revenue and orders are load-bearing; traffic and spend are explanatory; targets and prior actions are framing. With load-bearing data present and explanatory data missing, produce the 7-slide "commercial only" version and open slide 2 with a data-coverage box listing each missing source and each slide it suppressed. Never interpolate a missing day or week. Never collapse two channels into "other" to hide a gap. If consent-mode gaps mean GA4 undercounts, report the modelled-vs-observed split if the export carries it; otherwise mark every traffic figure "consent-affected, directional only" and suppress the YoY traffic column entirely.

## Method

1. **Lock the period, currency and money convention before touching a number.**
   - Fix the week as ISO Mon–Sun in store timezone unless the brief overrides it; print `Period: YYYY-MM-DD to YYYY-MM-DD, {tz}, ISO week` on slide 1.
   - Net revenue = gross order value − discounts − returns/refunds booked in the period − shipping revenue − VAT. At a 21% rate, net = gross / 1.21; subtracting 21% from gross understates net by 2.6% and is the single most common error in these decks.
   - Returns convention: book refunds in the week the refund was issued, not the week of the original order, and say so in the footer. Switching conventions mid-quarter invalidates the trend series.
   - Multiple VAT rates (21% standard, 12% reduced): compute net per line item and sum. A blended rate is acceptable only if the rate-mix shifted under 2.0pp WoW — state the mix shift in the slide-3 footnote.
   - One currency for the whole deck. FX conversion uses the ECB daily rate of the last day of the period, stated once; never a per-order rate.

2. **Assemble the metric set and freeze it for the quarter.**
   - The eight headline KPIs: net revenue, orders, AOV (net), site conversion rate, sessions, MER, cost per acquisition, return rate. Same set every week; adding or removing one requires a one-line reason on slide 3 and must hold for ≥4 consecutive weeks.
   - MER = net revenue / total paid spend ex-VAT. Input VAT is deductible for a VAT-registered store, so VAT-inclusive spend overstates nothing but understates ROAS by ~17.4%; label the row "spend ex-VAT" either way.
   - CAC = paid spend / new customers acquired in the period. If you have only order counts and no new-vs-returning split, label the row "cost per order" and never call it CAC in any sentence of the deck.
   - Blended vs in-platform: use blended (total spend / total net revenue) on slide 3 and platform-attributed ROAS only on slide 5, with the attribution window named (e.g. Meta 7d-click/1d-view).
   - Return rate = returns booked / orders shipped in the same period, units basis. State units or value basis once; do not mix.

3. **Compute variances against three baselines, in this precedence.**
   - For every KPI compute WoW, vs trailing 4-week average (excluding the reported week), and vs the same ISO week last year where ≥52 weeks of clean history exist. If history is 26–51 weeks, show YoY as "n/a — insufficient history"; under 26 weeks, drop the column.
   - The flag is driven by the trailing 4-week comparison. WoW is shown for colour only.
   - Thresholds for ratio/currency metrics: amber at ±10.0% to ±19.9% vs trailing 4wk, red at ±20.0% or beyond.
   - Thresholds for rate metrics, absolute and governing: site conversion rate amber at ±0.50pp, red at ±1.00pp; return rate amber at ±1.50pp, red at ±3.00pp. For these two, the relative test does not apply — a 4.4% → 4.9% return rate is +11% relative but only +0.5pp, so it flags green.
   - Low-volume guard: do not flag any metric computed on fewer than 100 orders or 30 conversions in the period. Mark the cell "n too low" and leave it uncoloured. The same guard applies to every segment on slides 5 and 6.
   - Excluded weeks: if a prior week in the trailing window was itself red for a one-off reason already documented (outage, Black Friday, migration), exclude it from the average, use the nearest clean prior week instead, and footnote which week was swapped.

4. **Explain only what breached, and only with evidence.**
   - Drill one level on every red and the two largest ambers by absolute currency impact. Maximum three drill-downs; a fourth means the thresholds are miscalibrated.
   - Cut order, stop at the first that explains it: channel → device → market → product category → new vs returning → landing page.
   - Contribution rule: the named segment must account for ≥50% of the total delta in currency. Between 30% and 50%, write "largest contributor, {X}% of delta — not sole driver". Under 30% across all segments, write "diffuse decline across ≥3 channels, no single driver".
   - Every cause line names an evidence source with a timestamp or report name (Merchant Center disapproval log, status page incident ID, deploy log, promo calendar row). No source, no cause.
   - If nothing explains it: "unexplained — checked: channel, device, market, promo calendar, deploy log, tracking." That is a complete and acceptable slide line.

5. **Audit promo and pricing claims for Omnibus compliance.**
   - If any discount ran in the period, pull the 30-day price history for every discounted SKU. The lawful reference price is the lowest price actually applied in the 30 days before the discount started, not RRP and not the pre-promo list price.
   - Recompute advertised depth against that reference. If the advertised depth and the lawful depth differ by ≥1.0pp, status is **fail**, the deck restates depth to the lawful figure, and the discrepancy goes on slide 8 with the SKU count.
   - If price history is unavailable for ≥1 discounted SKU, status is "not verified" with the count; never "pass".
   - Split returns into withdrawal returns (14-day EU right) and defect returns when the data carries a reason code. A withdrawal share above 70% after a discount week is a margin story — discounted goods bought on impulse and sent back — not a quality story, and should be read against the promo's net contribution, not the return-rate flag alone.
   - Free-shipping thresholds and bundle pricing count as price claims; check them on the same rule.

6. **Build the narrative spine, then the slides.**
   - Fixed order: verdict → data coverage → scorecard → funnel → channels → products → breach drill-down → risks & compliance → decisions → appendix.
   - Slide titles are assertions with a number: "Net revenue −13.4% vs 4wk, Google Shopping feed error accounts for 95%". Never labels like "Revenue" or "Channel performance".
   - Every chart or table slide carries one takeaway line of ≤20 words at the bottom. If you cannot write it, cut the slide.
   - Tables over images: maximum 7 rows and 7 columns per slide; anything larger goes to the appendix. Minimum 14pt body text at 16:9.
   - Colour only encodes flag status (red/amber/green). No other colour on any slide.

7. **Create it in Google Slides when the connector is available.**
   - Title: `KPI Weekly — {store} — W{ISO week} {year}`. One slide per spine item, 16:9, text and tables only.
   - Share with the requester's domain at the access level they asked for; do not widen sharing, do not set "anyone with the link" unless explicitly requested.
   - On connector error, retry once. After the second failure, fall back to Markdown slide blocks and state: "Google Slides connector unavailable after 2 attempts — deck delivered as paste-ready Markdown."
   - Never embed customer-level rows, screenshots containing order IDs, or exports with email addresses.

8. **Close with three decisions and an owner slot each.**
   - Maximum three asks. Each has: the action, a numeric trigger or success threshold, a date, and `owner: ___` left blank for the human.
   - Carry forward every unresolved action from the prior deck with its age in days and status (not started / in progress / blocked). Any action older than 21 days is escalated on the slide as "aged — close or kill".
   - Propose spend changes only within a ±20% band of current weekly spend per channel. Anything larger is written as "requires owner approval" with the rationale; budget reallocation is the human's call, not yours.
   - Never schedule a decision that depends on data the deck has marked unavailable or unverified.

## Judgement calls

**When WoW vs trailing-4-week baseline** — flag off the trailing 4-week average, show WoW alongside for colour. Tip to WoW when the prior week was itself flagged and documented as a one-off (Black Friday, 6-hour checkout outage, platform migration) — the average is then contaminated and you should instead exclude that week from the window and say which week was swapped out. Always print which baseline drove the flag in the scorecard cell.

**When one more explanatory slide vs shipping on time** — the deck is worthless after the meeting starts. At T−15 minutes, ship the 9-slide spine with one honest "unexplained" line rather than 13 slides containing a guess. Tip to the extra slide only when the unexplained metric is net revenue or site CVR, the breach is red, and the drill-down is a single query away (≤5 minutes).

**When GA4 revenue and shop backend revenue disagree** — the shop backend is truth for money, GA4 is truth for behaviour, with no exceptions. Gap ≥5.0% goes on slide 8 with both absolute figures and the percentage; gap under 5.0% is a footnote on slide 3. Never average the two, never pick the flattering one, never silently switch sources between weeks.

**When granularity vs comparability** — resist new breakdowns mid-quarter. Add a segment only if it will be reported for at least the next 4 consecutive weeks and has ≥100 orders per week; one-off cuts belong in the appendix. A trend series that fractures in week 3 is unreadable by week 12, and the first question in the meeting becomes "is this comparable?" instead of "what do we do?".

## Rules

- Never invent a cause, a target, or a prior-period figure. Missing baseline = "no comparison available".
- Never mix VAT conventions within a deck; every money slide footer reads "net of VAT" or "incl. VAT".
- Never present GA4 modelled conversions as observed; if consent-mode modelling is on, state it once on slide 4 with the modelled share if available.
- Do not state a discount depth or "was" price without the Omnibus 30-day-low check; if unverifiable, write "not verified" with the SKU count.
- Do not flag metrics below the volume guard (100 orders / 30 conversions); mark them "n too low".
- Do not change ad budgets, pause campaigns, edit feeds or alter store settings — the deck recommends, the human decides.
- Mark every assumption inline with `[assumption]` and repeat all of them on the appendix slide; no silent defaults.
- Main deck spine is 9 slides; hard cap 14 including any extra drill-down. Appendix is unlimited and never presented.
- No personal data on slides: no customer names, emails, phone numbers or order IDs. Aggregate only, minimum cell size 10 orders.
- If two sources conflict and neither is clearly authoritative, show both figures and name the conflict; do not resolve it yourself.
- Maximum 3 drill-downs, 3 decisions, 1 takeaway line per slide. These caps are not negotiable under time pressure — cut content, not caps.

## Output format

```
# KPI Weekly — {store} — W{ISO week} {year}
Period: {YYYY-MM-DD} to {YYYY-MM-DD}, {timezone}, ISO week. Money: net of VAT, net of returns, shipping excluded.
Delivery: {Google Slides link} | {Markdown fallback}

## Slide 1 — Verdict
{One sentence: direction, magnitude, single biggest driver.}
Status: {on track / at risk / off track} vs {plan | trailing 4wk}

## Slide 2 — Data coverage
Sources used: {list with export date}. Missing: {list}. Slides suppressed: {list}.
Assumptions: {`[assumption]` lines}

## Slide 3 — Scorecard
| KPI | This week | WoW | vs 4wk avg | YoY | Flag | Baseline used |
|---|---|---|---|---|---|---|
| Net revenue | | | | | | |
| Orders | | | | | | |
| AOV (net) | | | | | | |
| Site conversion rate | | | | | | |
| Sessions | | | | | | |
| MER (spend ex-VAT) | | | | | | |
| CAC / cost per order | | | | | | |
| Return rate | | | | | | |

## Slide 4 — Funnel
Sessions → product views → add to cart → checkout started → purchase, with step CVR and delta vs 4wk.
Takeaway: {≤20 words}

## Slide 5 — Channels
| Channel | Sessions | CVR | Net revenue | Spend ex-VAT | ROAS | Δ revenue vs 4wk |
Takeaway: {≤20 words}

## Slide 6 — Products / categories
Top 5 by net revenue delta up, top 5 down, with currency amounts. Takeaway: {≤20 words}

## Slide 7 — Breach drill-down
{Metric}: {segment} explains {X}% of the delta ({currency amount}). Evidence: {source + timestamp}.
{Or: unexplained — checked: channel, device, market, promo calendar, deploy log, tracking.}

## Slide 8 — Risks & compliance
Omnibus reference price check: {pass / fail / not verified} — {SKU count}
Withdrawal returns vs defect returns: {split}
Tracking/consent note: {text}
Data conflicts: {source A figure} vs {source B figure}, {gap %}

## Slide 9 — Decisions this week
1. {Ask} — trigger: {number} — owner: ___ — by: {YYYY-MM-DD}
2. ...
3. ...
Carry-forward: {item, age in days, status}

## Appendix — Full metric table, definitions, assumptions, excluded weeks
```

- Spine is 9 slides; hard cap 14. Appendix unlimited, never presented.
- Order is fixed: verdict slide 1, decisions second-to-last. Do not bury the verdict after the data.
- When over length, cut in this order: products slide → YoY column → funnel step detail → channel rows below rank 6. Never cut slide 2 (data coverage) or slide 9 (decisions).

## Worked example

**Input:** "Weekly KPI review for Papírnictví Luna, week 2024-W41 (7–13 Oct). Shopify export attached, GA4 export attached, Meta + Google Ads spend attached. No formal plan for Q4. Slides connector is connected."

**Output (abbreviated):**

> **KPI Weekly — Papírnictví Luna — W41 2024**
> Period 2024-10-07 to 2024-10-13, Europe/Prague, ISO week. Money: net of VAT (21% standard, 12% on printed diaries and calendars — rate-mix shift 0.8pp WoW, blended rate acceptable), net of returns booked in period, shipping revenue excluded. Slides link created, shared with @papirnictviluna.cz.
>
> **Slide 1 — Verdict:** Net revenue 486 200 CZK, −13.4% vs trailing 4-week average of 561 500 CZK — **red, off track**. A Google Merchant Center feed error on 11 Oct removed 340 SKUs for the final 3 days of the week and accounts for 71 400 CZK (95%) of the 75 300 CZK shortfall.
>
> **Slide 2 — Data coverage:** Shopify orders export (pulled 14 Oct 07:40), GA4 weekly report (14 Oct 07:55), Google Ads and Meta Ads spend (14 Oct 08:05), Merchant Center diagnostics. Missing: none. Slides suppressed: none. `[assumption]` No written Q4 plan supplied — all "vs" columns are trailing 4-week average, not plan. `[assumption]` Returns booked by refund date.
>
> **Slide 3 — Scorecard:**
>
> | KPI | W41 | WoW | vs 4wk avg | YoY | Flag | Baseline |
> |---|---|---|---|---|---|---|
> | Net revenue | 486 200 CZK | −11.8% | −13.4% (561 500) | −4.1% | **red** | 4wk |
> | Orders | 1 842 | −8.4% | −9.1% (2 026) | −2.6% | amber | 4wk |
> | AOV (net) | 263.9 CZK | −3.7% | −4.7% (277.1) | −1.6% | green | 4wk |
> | Site conversion rate | 1.94% | −0.14pp | −0.17pp (2.11%) | −0.09pp | green | 4wk, pp test |
> | Sessions | 94 900 | −0.9% | −1.2% (96 050) | +3.4% | green | 4wk |
> | MER (spend ex-VAT) | 4.10 | −12.2% | −14.6% (4.80) | −7.0% | amber | 4wk |
> | Cost per order | 64.4 CZK | +10.9% | +11.5% (57.8) | +6.2% | amber | 4wk |
> | Return rate | 4.9% | +0.4pp | +0.5pp (4.4%) | +0.2pp | green | 4wk, pp test |
>
> Footnote: cost per order, not CAC — the Shopify export carries no new-vs-returning customer flag. Spend ex-VAT 118 600 CZK vs 4wk average 117 000 CZK. Return rate +11% relative but +0.5pp absolute, below the 1.50pp amber line, so green.
>
> **Slide 4 — Funnel:** Sessions 94 900 → product views 41 700 (43.9%, −0.6pp) → add to cart 9 480 (22.7% of PV, −0.3pp) → checkout started 3 410 (36.0% of ATC, −1.1pp) → purchase 1 842 (54.0% of checkouts, −0.8pp). No single step breached ±1.0pp; the loss is upstream, in paid traffic quality, not on-site. Consent-mode modelling is on; 11.4% of W41 conversions are modelled. Takeaway: Funnel intact site-wide; the breach is channel-level, not a checkout problem.
>
> **Slide 5 — Channels:**
>
> | Channel | Sessions | CVR | Net revenue | Spend ex-VAT | ROAS | Δ rev vs 4wk |
> |---|---|---|---|---|---|---|
> | Google Shopping | 26 300 | 1.72% | 119 300 | 48 200 | 2.47 | −71 400 |
> | Organic search | 21 800 | 2.44% | 97 500 | — | — | +1 900 |
> | Google Search (brand+generic) | 11 400 | 2.86% | 86 400 | 21 300 | 4.06 | −2 100 |
> | Meta paid | 17 900 | 1.58% | 74 600 | 49 100 | 1.52 | −4 800 |
> | Direct | 9 700 | 2.42% | 62 300 | — | — | −1 300 |
> | Email | 5 100 | 2.51% | 34 100 | — | — | +2 300 |
> | Referral/other | 2 700 | 1.66% | 12 000 | — | — | +100 |
>
> Takeaway: Google Shopping sessions −11.5% and CVR −0.69pp; every other channel within ±5%.
>
> **Slide 6 — Products:** Down: Diář A5 Luna Classic 2025 −18 900; Sada pastelek Koh-i-noor 36 ks −12 400; Zápisník Leuchtturm1917 A5 −9 700; Školní batoh Baagl Ergo −7 300; Pilot G2 blister 4 ks −5 100. Up: Vánoční balicí papír set +9 400; Stolní kalendář 2025 +6 200; Washi pásky mix +3 100; Plnicí pero Lamy Safari +2 800; Fixy Stabilo Boss 6 ks +1 900. All five decliners were inside the 340 disapproved SKUs. Takeaway: Decline is SKU-availability, not demand — Vánoce lines already growing.
>
> **Slide 7 — Breach drill-down:** Net revenue, red. Google Shopping accounts for 71 400 CZK of the 75 300 CZK delta = 95%, above the 50% single-driver threshold. Mechanism: 340 SKUs disapproved for "missing GTIN" after a feed template change; Shopping orders fell 716 → 452 while AOV held at 264.0 CZK. Evidence: Merchant Center disapproval log, 2024-10-11 03:12 CEST; GA4 session-default-channel report; Shopify order-source field. Days affected: 11–13 Oct (3 of 7).
>
> **Slide 8 — Risks & compliance:** Omnibus reference price check — **fail, 14 SKUs**. "Podzimní sleva" advertised −30% from 379 CZK on Diář A5 Luna Classic 2025 (sale price 265.30 CZK), but the lowest price applied in the 30 days before 7 Oct was 349 CZK, so the lawful reference is 349 CZK and true depth is −24.0%. All deck references to promo depth restated to −24.0%; creatives and PLP badges require correction before the next campaign. Returns: 90 total — 61 withdrawal (67.8%), 29 defect (32.2%), within the normal 60–70% withdrawal band. Data conflict: GA4 purchase revenue 471 900 CZK vs Shopify net 486 200 CZK, gap 2.9%, under the 5% line — footnoted on slide 3, Shopify governs. Tracking: consent-mode modelling on, 11.4% of conversions modelled.
>
> **Slide 9 — Decisions this week:**
> 1. Confirm the feed fix is live across all 340 SKUs and request expedited re-review — trigger: Google Shopping CVR back above 2.20% and sessions above 29 000 by 2024-10-18 — owner: ___ — by 2024-10-18.
> 2. Correct the Omnibus reference price from 379 to 349 CZK on 14 discounted SKUs across creatives, PLP badges and the feed `sale_price_effective_date` — trigger: zero SKUs showing a 379 CZK "was" price — owner: ___ — by 2024-10-16.
> 3. Hold Meta weekly budget at 49 100 CZK ex-VAT this week; a cut to the ROAS-1.52 placements would exceed the ±20% band — escalated, requires owner approval — by 2024-10-21.
> Carry-forward: "Test free-shipping threshold at 999 CZK" — opened W38, age 21 days, not started, **aged — close or kill**.
>
> **Appendix:** full 8-KPI table with 13 weekly data points per metric, channel table to rank 14, metric definitions (net revenue, MER, cost per order, return rate basis), both `[assumption]` lines from slide 2, and a note that W39 (checkout outage 2 Oct, 5h40m) is excluded from the trailing average, replaced by W36.

## Quality bar

- [ ] Every money figure on every slide is net of VAT and net of returns, and the footer on each money slide says so.
- [ ] Each flagged KPI states which baseline triggered the flag and clears the 100-order / 30-conversion volume guard, or reads "n too low".
- [ ] Rate metrics (site CVR, return rate) are flagged on the pp test only, not the relative test.
- [ ] No causal claim appears without a named segment, its share of the delta in both % and currency, and an evidence source with a timestamp or report name — or the explicit "unexplained — checked:" line.
- [ ] The Omnibus check reads pass, fail or not verified with a SKU count whenever any discount ran in the period.
- [ ] Slide 2 lists every missing source and every slide it suppressed; every `[assumption]` tag on any slide also appears in the appendix.
- [ ] Main deck is ≤14 slides, verdict is slide 1, drill-downs are ≤3.
- [ ] Decisions slide carries ≤3 asks, each with a number, a date and `owner: ___`; every carry-forward item shows age in days and status.
- [ ] Any proposed spend change is within ±20% of current weekly spend, or is labelled "requires owner approval".
- [ ] No customer name, email or order ID appears on any slide; no aggregate cell is built on fewer than 10 orders.
- [ ] Google Slides link opens and is shared at the requested access level, or the Markdown fallback states the connector failed twice.

## Failure modes

**Deck says revenue is up, finance says it is flat** — gross/net mixing: GA4 purchase revenue includes VAT and shipping while the backend figure does not, a ~25% gap on a 21% VAT store. Reconcile both sources on one appendix line every week and require the gap to be under 5% before publishing; above 5%, it goes on slide 8 before the deck ships.

**Every KPI is amber** — flagging off WoW on a week following a promo spike, or flagging segments under the volume guard. Apply the trailing 4-week baseline, exclude contaminated weeks, apply the 100-order guard, then recount: more than three ambers in an eight-KPI scorecard means the thresholds are wrong, not the business.

**A stated cause turns out to be false in the meeting** — a plausible narrative written before the drill-down ran. Enforce the ≥50% contribution rule and require a named, timestamped evidence source on every cause line. No evidence, no cause — "unexplained" costs you nothing, a wrong cause costs you the deck.

**Promo slide overstates discount depth** — depth computed from RRP or the pre-promo list price instead of the lawful 30-day low. Pull price history for every discounted SKU, recompute, and restate depth before the number reaches a slide; a 30% claim that is lawfully 24% is both a credibility and a regulatory problem.

**Nobody acts on the deck** — decisions written as observations, without triggers, owners or dates ("we should look at Meta performance"). Verify each decision line contains a number, a date and an owner slot; if it reads like a sentence lifted from slide 7, rewrite it as an instruction with a measurable outcome.

**Week 12 of the quarter is unreadable** — metrics added and dropped ad hoc, segments renamed, conventions switched mid-stream. Freeze the eight KPIs, the baseline rules and the money convention for the whole quarter; log any change with its effective week in the appendix so the series can be re-based rather than abandoned.

## License

MIT
