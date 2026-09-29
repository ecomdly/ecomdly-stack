---
name: ga4-funnel-analyst
owner: shopmetric
category: Analytics & tracking
description: Builds a session-level GA4 checkout funnel, tests which step worsened against the store's own baseline, localises it by device and channel, and proposes one fix with a way to confirm it.
version: v4
license: MIT
updated: 2026-09-29
recommended: true
security_checked: true
url: https://ecomdly.com/skills/shopmetric/ga4-funnel-analyst
raw: https://ecomdly.com/raw/shopmetric/ga4-funnel-analyst.md
install: npx @ecomdly/cli add shopmetric/ga4-funnel-analyst
---

# GA4 funnel analyst

Turns GA4 ecommerce events into a step-by-step purchase funnel. It finds the step that got worse against the site's own baseline, identifies the segment responsible, and proposes one change with a way to confirm it. The naive funnel counts events, compares against remembered industry benchmarks and lists ten fixes. It fails because event counts double-count reloads, benchmarks from other stores say nothing about this one, and a 3-point drop on 150 sessions is usually noise. This skill counts sessions, tests whether a change is larger than chance, and localises it by device, source and step before recommending anything.

## When to use
- Weekly, on the last full week against the previous week or a stable 4-week baseline.
- After a checkout, theme, payment or shipping-option release, comparing equal-length windows before and after.
- When the weekly KPI report (`weekly-store-kpi-report`) shows a conversion-rate movement that needs locating.

## When not to use
- The events are unaudited or recently changed. Run `ga4-ecommerce-event-auditor` first. A funnel on broken tagging produces confident nonsense.
- Deciding whether a variant won a test. Use `ab-test-readout`. Before/after windows are not randomised.
- A qualitative UX review of the checkout. Use `checkout-friction-audit` or `product-page-cro-review` once the leaking step is known.

## Inputs
Required:
1. **Session counts per funnel step** for the analysis period and a comparison period of equal length and the same weekdays. Steps: `view_item`, `add_to_cart`, `begin_checkout`, `add_shipping_info`, `add_payment_info`, `purchase`. Broken down by `device category` and `session default channel group` (or `session source / medium`).
   Best source: BigQuery. Count distinct `user_pseudo_id` + `ga_session_id` that fired each event, and for a closed funnel require the previous step earlier in the same session.
   Alternative: GA4 Explore > Funnel exploration, with the same steps, closed, and the breakdown dimension set. Note that Funnel exploration counts users by default, and that Explorations can be sampled or thresholded.
2. **Release and promotion log** for both periods (date, what changed).
3. **Tracking status**: date of the last event audit, consent mode type, and any tagging change inside either window.

Optional: `shipping_tier` and `payment_type` values at their steps, form-error events, page timing (Core Web Vitals or custom timing events), session recordings tool availability.

If the breakdown is missing, analyse the total only and say that segment localisation was not possible. If the comparison period contains a tagging change, choose another baseline or stop.

## Best practices

### Build the funnel correctly
1. **Count sessions, not events.** Reloads, double-firing tags and multiple `add_to_cart` per session inflate event counts. The unit is the session that reached the step. Use users only when the decision is about people who buy across sessions, and say which unit is used.
2. **Closed vs open funnel.** In GA4 Funnel exploration, an open funnel lets users enter at any step, while a closed funnel requires entry at step 1. For step-to-step conversion, use closed, so each denominator contains only sessions that could have taken the next step. Steps can be "indirectly followed by" (other events in between allowed) or "directly followed by". Use indirectly, because checkouts fire other events between steps.
3. **Step conversion** = sessions reaching step k+1 after step k ÷ sessions reaching step k. **Overall** = purchase sessions ÷ step-1 sessions. The overall rate hides where the change happened, so always report the step rates.
4. **Same time frame.** GA4 funnel steps can be constrained with "within" a time. Use the session as the boundary for checkout steps. Purchases that happen in a later session (bank transfer, returning next day) are real but belong to a user-level analysis.

### Judge change against the site, with statistics
5. **Baseline = the site's own comparison period**, never industry benchmarks. Numbers from other stores differ in traffic mix, product price and tagging, so they cannot tell which step broke here.
6. **Test each step change.** Two-proportion z-test: z = (p₁ − p₀) ÷ √(p̄(1 − p̄)(1/n₁ + 1/n₀)), where p = step conversion, n = sessions entering the step and p̄ = pooled rate. Treat |z| ≥ 2 as movement. Also report the 95% interval for each rate, p ± 1.96 × √(p(1 − p)/n). There are about 5 steps × several segments, so some cells will cross |z| = 2 by chance. Only act on a segment finding if the total also moved, or if the segment's z is well beyond 2.
7. **Rank leaks by lost purchases, not by percentage.** Lost purchases at step k ≈ (p₀ − p₁) × n₁ × (downstream conversion from step k+1 to purchase). A 10-point drop on a small segment can matter less than a 2-point drop on the main one.
8. **A segment is "too small to judge" when its interval is wider than the change being claimed**, rather than below a fixed session count. Report its counts anyway.

### Localise before explaining
9. **Order of breakdown**: device first (templates differ by device), then channel (intent differs by channel), then `shipping_tier` or `payment_type` at the step where they apply. Stop at the first dimension that concentrates the change, meaning one segment carries most of the lost purchases while the others are flat.
10. **Mix shift vs rate change.** If the total rate fell but every segment's rate is flat, the traffic mix changed (for example more low-intent paid social). The fix is then in acquisition, not in the checkout. Check the segment shares of the step's entrants in both periods.
11. **Tie changes to the release log.** A step drop that starts on a release date on the device the release touched is the strongest non-experimental evidence available, but it is still a hypothesis.

### Recommend one change, with a confirmation test
12. **One fix**: the smallest change that addresses the largest lost-purchase leak in the concentrated segment. List the runner-up as next week's candidate.
13. **State what would confirm the diagnosis before shipping.** Examples: form-error events per field, session recordings filtered to the segment and step, payment-gateway error logs, or page timing on that step. If the confirmation is cheap, ask for it first. Where risk allows, ship the fix as a test (hand-off: `ab-test-readout`).

### Know the blind spots
14. **Consent**: GA4 observes only sessions whose users allowed analytics, unless modeled data is in use, and Explorations use observed data. Step ratios are less affected than absolute counts, provided the consent rate is similar at each step. Say that the funnel describes consenting sessions.
15. **Checkout steps outside the site** (hosted payment pages) end the observable session. `add_payment_info` → `purchase` then also contains payment failures and non-returns. Split by `payment_type` before blaming the page.

## Process
1. **Gate.** Confirm that the last event audit post-dates the last tagging change, and that no tagging change falls inside either window. If a later step has more sessions than an earlier one, stop and report "tagging inconsistent at <step>". Hand off to `ga4-ecommerce-event-auditor`.
2. **Build the total funnel** for both periods, with sessions per step, step conversion and interval.
3. **Test each step change** (z, best practice 6), and compute lost purchases per step (best practice 7). The worst leak is the step with the most lost purchases among those with |z| ≥ 2. If no step moved, report "no step changed beyond noise" and stop. That is a valid result.
4. **Localise the worst leak** by device, then channel, then `shipping_tier` or `payment_type`. For each segment, give counts, rates, z and the entrants' share (mix check, best practice 10).
5. **Explain.** Match the timing and segment to the release log. Write the hypothesis and the confirmation step.
6. **Recommend** one change plus a runner-up, and state the expected effect only as the recovery of the measured drop (for example, "back to 74% would restore about N purchases a week"). Do not promise more than the baseline.
7. Write the output and run the quality checklist.

## Pitfalls and edge cases
- **`begin_checkout` fired on cart page load** on some themes, which makes cart → checkout look perfect and moves the leak to the next step. Check where each event fires.
- **Guest vs logged-in flows** skip steps (saved address → no `add_shipping_info`). Closed funnels then drop logged-in buyers. Check purchase sessions missing intermediate steps.
- **Promotions** change intent. A week with a free-shipping threshold raises shipping-step conversion, so do not compare a promo week with a normal week without saying so.
- **Bots and internal traffic** inflate `view_item` and `add_to_cart`. Check for sudden spikes from one source, and filter out internal IPs.
- **Payment-method change at `add_payment_info`**: a new gateway with 3-D Secure failures looks like a checkout UX problem. Split by `payment_type`.
- **Tablet** is often tiny. Merge it with mobile or report it separately as too small, and say which.
- **Holiday weekdays** in one window and not the other distort session mix, even with equal window lengths.
- **Thresholding** in Explorations hides small rows. Use BigQuery or a longer range if segments disappear.

## Rules
- Read-only. No changes to the site, GTM or GA4. The recommendation is a proposal for a human.
- Every percentage is shown with its counts ("58.1% (3,100 → 1,801)").
- No external benchmarks. The baseline is the site's own comparison period.
- Correlation is labelled as a hypothesis, with a named confirmation step.
- One recommended change. A second may be listed as the next candidate, not as a parallel to-do.
- If data fails the gate, report that and stop. Do not produce a funnel from inconsistent events.

## Output format
```
## Funnel — <period> vs <comparison> · unit: sessions · closed funnel · source: <BigQuery|Exploration>
| step | sessions (now) | conv. now | conv. before | Δ pt | z | lost purchases/wk |
|---|---|---|---|---|---|---|
| add_to_cart → begin_checkout | a → b | x% | y% | ±d | z | n |
...
Worst leak: <step> (<n> lost purchases, z = <z>)

## Where it leaks
| segment | entrants now (share) | conv. now | conv. before | Δ pt | z |
...
Mix check: <shares stable | shifted>
Too small to judge: <segments + counts>

## Hypothesis
<what, where, since when; link to release log>
Confirm by: <data to pull / recording filter / error log>, decision rule: <if … then …>

## Try first
<one change> — expected: restore about <n> purchases/week if the step returns to baseline.
Next candidate: <step/segment>.
Caveats: consent-observed sessions only; <other>.
```

## Worked example
Illustrative numbers from a BigQuery session-level closed funnel: 7–13 September 2026 against 31 August–6 September 2026, the same weekdays. The release log shows "new address form on mobile checkout" deployed on 7 September.

```
| step | sessions (now) | conv. now | conv. before | Δ pt | z |
| add_to_cart → begin_checkout | 12 400 → 4 960 | 40.0% | 43.1% (12 100 → 5 215) | −3.1 | −4.9 |
| begin_checkout → add_shipping_info | 4 960 → 3 209 | 64.7% | 74.5% (5 215 → 3 885) | −9.8 | −10.8 |
| add_shipping_info → add_payment_info | 3 209 → 2 920 | 91.0% | 91.1% (3 885 → 3 540) | −0.1 | −0.2 |
| add_payment_info → purchase | 2 920 → 2 480 | 84.9% | 84.9% (3 540 → 3 005) | 0.0 | 0.1 |
```
The worst leak is begin_checkout → add_shipping_info. Downstream conversion from add_shipping_info to purchase is 2,480 ÷ 3,209 = 77.3%. Lost purchases ≈ (0.745 − 0.647) × 4,960 × 0.773 ≈ 376 per week. The cart → checkout drop (−3.1 pt on 12,400 entrants, downstream 50.0%) is worth ≈ 192 per week and is the next candidate.

```
| segment | entrants now (share) | conv. now | conv. before | Δ pt | z |
| mobile (incl. tablet) | 3 100 (62.5%) | 58.1% (→ 1 801) | 74.0% (3 250 → 2 405) | −15.9 | −13.4 |
| desktop | 1 860 (37.5%) | 75.7% (→ 1 408) | 75.3% (1 965 → 1 480) | +0.4 | 0.27 |
```
Mix check: mobile share of entrants is 62.5% now against 62.3% before, which is stable. The whole drop sits on mobile, and desktop is flat. The mobile rate's 95% interval is ±1.7 pt.

Hypothesis: the new mobile address form, live since 7 September, blocks or discourages completion. Confirm by pulling form-error events per field on mobile for 7–13 September, and by watching 20 mobile recordings that stop at the shipping step. If errors concentrate on one field (for example postcode validation rejecting "123 45" with a space), fix the validation. If there are no errors but there is abandonment, test a rollback.

Try first: fix the field that errors, or roll back the mobile address form. Restoring the mobile step to 74.0% would recover about 3,100 × 0.159 × 0.773 ≈ 381 purchases per week, consistent with the total estimate. Next candidate: cart → checkout.

## Quality checklist
- The tagging gate passed, and every later step is ≤ the step before it.
- The unit (sessions/users), funnel type (closed/open) and data source are stated.
- Both periods have equal length and the same weekdays. Promotions and releases in both are listed.
- Every rate has counts. Changes have z. Lost purchases are computed with the formula shown.
- The mix check was done before blaming a step.
- There is exactly one recommendation, with a confirmation step and a decision rule. The runner-up is named.
- No external benchmark is used.

## Sources
- [GA4] Funnel exploration (open/closed, directly/indirectly followed by): https://support.google.com/analytics/answer/9327974
- [GA4] Checkout journey report: https://support.google.com/analytics/answer/14000977
- GA4 recommended events reference: https://developers.google.com/analytics/devguides/collection/ga4/reference/events
- [GA4] About Analytics sessions: https://support.google.com/analytics/answer/9191807
- [GA4] About data thresholds: https://support.google.com/analytics/answer/9383630
- [GA4] Behavioral modeling for consent mode: https://support.google.com/analytics/answer/11161109
- GA4 BigQuery export schema: https://support.google.com/analytics/answer/7029846

## License
MIT
