---
name: ab-test-readout
owner: checkoutlab
category: Conversion & UX
description: Reads out an e-commerce A/B test in the right order: SRM chi-square, pre-registered primary metric, sample size and peeking, effect with confidence interval, guardrails and novelty, then ship, don't ship or inconclusive.
version: v4
license: MIT
updated: 2026-09-29
recommended: false
security_checked: true
url: https://ecomdly.com/skills/checkoutlab/ab-test-readout
raw: https://ecomdly.com/raw/checkoutlab/ab-test-readout.md
install: npx @ecomdly/cli add checkoutlab/ab-test-readout
---

# A/B test readout

Produces the readout for an online-store A/B test in a fixed order. It checks whether the test is trustworthy first: sample ratio, pre-registered plan, sample size and peeking. Only then does it report the primary effect with a confidence interval, check guardrails and novelty, and state one decision: ship, don't ship or inconclusive. The naive readout opens the testing tool, sees "95% chance to beat control" on the metric that moved most, and ships. That fails because an unplanned metric chosen after the fact, a test stopped at the first significant peek, or a broken randomisation all produce "wins" that are noise or artefacts. Validity comes before effect size, always.

## When to use
- A test has reached its pre-registered sample size or end date.
- Someone wants to stop early because it "looks significant". This readout says whether the plan allows it.
- A past "winner" did not show up in the weekly numbers, and the readout needs to be re-examined.

## When not to use
- There was no randomisation (before/after comparison, a holdout picked by hand). Use `ga4-funnel-analyst` for descriptive analysis and call it observational.
- The test has not started and needs planning. The power section below can size it, but the plan must be written and agreed before launch.
- Tracking of the primary metric is in doubt. Run `ga4-ecommerce-event-auditor` on the events the testing tool uses first.

## Inputs
Required:
1. **Test plan** written before launch: hypothesis, randomisation unit (user or cookie, session, device), planned split, **one primary metric** with its exact definition, guardrail metrics with thresholds, significance level α, power, minimum detectable effect (MDE), planned sample per arm, start date, planned end date, and the stopping rule (fixed horizon or a named sequential method).
2. **Per variant**: units assigned (randomised), units exposed (analysis population), conversions for the primary metric, and the guardrail metric values. For revenue metrics, user-level values (one row per user: revenue, orders), or at least the mean and standard deviation of per-user revenue.
3. **Per week of first exposure**, per variant: units and conversions, for the novelty check.
4. **Change log**: deploys, promotions, tracking changes and traffic incidents during the test.

Optional: device and new/returning splits; pre-experiment metric per user (for CUPED); testing-tool export of assignment events.

If there is no written plan, say so at the top, treat the metric the team names now as exploratory, and cap the decision at "inconclusive — rerun with a pre-registered plan" unless the effect is also confirmed on an untouched follow-up. If user-level revenue is missing, report revenue metrics as point estimates only, with no interval and no ship call based on them.

## Best practices

### Validity gates (in this order; any failure stops the readout)
1. **Sample ratio mismatch (SRM).** Test observed units against the planned split with a chi-square goodness-of-fit test:
   χ² = Σ (Oᵢ − Eᵢ)² ÷ Eᵢ, where Eᵢ = total units × planned shareᵢ, df = number of arms − 1.
   For two arms, df = 1, and χ² = 3.84 corresponds to p = 0.05, 6.63 to p = 0.01 and 10.83 to p = 0.001. Use a strict, pre-agreed threshold (this skill defaults to p < 0.001), because SRM is checked on every test and a genuine mismatch usually produces a very small p. Run it on assigned units and, separately, on exposed units. If assigned passes and exposed fails, the variant changes who gets counted: a redirect loses users, a variant page errors, a tag fires in one arm only, or bot filtering acts differently on one arm. With SRM, report no effect. Report likely causes, following Fabijan et al.'s taxonomy of assignment, execution, log processing and analysis causes.
2. **Randomisation unit = analysis unit.** If users are randomised but the metric is per session (conversion per session), sessions within a user are correlated and a naive z-test understates variance. Use the delta method or aggregate to user level. The same applies to AOV (revenue ÷ orders), which is a ratio metric.
3. **Pre-registered primary metric.** Judge the test on the one metric the plan named. Other metrics that moved are hypotheses for the next test. With k metrics or segments tested at α, expect about k × α false positives. Use a Bonferroni-adjusted α ÷ k when a decision rests on more than one.
4. **Planned sample reached and full weeks covered.** Under the planned sample, the test is inconclusive regardless of p. Duration should cover whole weeks, because weekday and weekend buyers differ. Tests that include a payday, a promotion or a holiday that the plan did not anticipate are flagged.
5. **Peeking.** A fixed-horizon test evaluated repeatedly and stopped at the first p < α has a false-positive rate well above α. Early stopping is valid only if the plan named a sequential method in advance, such as group-sequential boundaries with alpha spending (for example O'Brien–Fleming) or always-valid p-values (mSPRT, Johari et al.). Otherwise an early "win" is reported as "not evaluable before the planned horizon".

### Estimating the effect
6. **Conversion rate (binary per user).** pᵢ = conversionsᵢ ÷ unitsᵢ, absolute difference d = p_B − p_A, standard error SE = √(p_A(1 − p_A)/n_A + p_B(1 − p_B)/n_B), z = d ÷ SE, 95% CI = d ± 1.96 × SE. Relative lift = d ÷ p_A. An approximate relative CI is the absolute CI ÷ p_A. Say that it is approximate, and use the delta method or bootstrap if precision matters.
7. **Revenue per user** is heavily right-skewed, with most users at 0 and a few large orders. Use Welch's t-test on user-level values with large n, or a bootstrap of the difference in means (resample users, at least a few thousand resamples). Cap extreme values at a pre-registered percentile (winsorising) and report both capped and uncapped results. One B2B order must not decide a test.
8. **Report absolute and relative effect, CI and p together.** A p-value alone says nothing about size. A CI that excludes zero but includes effects too small to pay for the change is a "don't bother", not a win.
9. **Power and MDE.** Planned sample per arm for a two-sided test of proportions:
   n = (z₁₋α/₂ + z₁₋β)² × (p₁(1 − p₁) + p₂(1 − p₂)) ÷ (p₂ − p₁)², where p₂ = p₁ × (1 + relative MDE).
   z₀.₉₇₅ = 1.96, z₀.₈₀ = 0.84, z₀.₉₀ = 1.28. Do not compute "observed power" from the observed effect, because it is a restatement of the p-value. Instead, state the planned power against the planned MDE, and whether the CI rules out the MDE.
10. **Winner's curse.** A significant result from a test powered for a larger MDE tends to overstate the true effect. Expect the effect after launch to be smaller than measured, and plan revenue forecasts on the CI's lower half, not the point estimate.
11. **Variance reduction** (CUPED: regress the metric on the same user's pre-experiment value) can shorten tests substantially when pre-period data exists for most users. It must be in the plan, not applied after seeing the results.

### Guardrails, time and segments
12. **Guardrails** block shipping even when the primary wins. Typical ones for a store: AOV, return or refund rate (lagging, so check again after the return window), page load time, checkout error rate, and unsubscribe rate for e-mail tests. Each guardrail has a pre-set non-inferiority limit, such as "AOV not worse than −3%". Judge a guardrail by whether its CI crosses the limit, not only by its point estimate.
13. **Novelty and primacy.** Compute lift by week of first exposure. A lift that shrinks every week suggests novelty. A lift that grows suggests learning. Report the last week's lift and CI next to the total. If novelty is suspected and the decision is to ship, recommend a holdback (a small share kept on control) to measure the long-run effect.
14. **Segments explain, they do not decide.** Look at device and new/returning only, to explain the total result. A segment "winner" in a flat test is a hypothesis for a new test with that segment pre-registered.
15. **Interference.** Price and promotion tests can leak between arms, because users compare prices, share links or switch devices, and cookie-based assignment splits one person into two users. Note this as a limitation where it applies.

## Process
1. **Read the plan.** List the primary metric, guardrails, α, power, MDE, planned n, dates and stopping rule. If there is no plan, flag it (see Inputs).
2. **SRM** on assigned units, then on exposed units. If p < threshold, write the SRM section with likely causes and stop, with the decision "invalid — fix and rerun".
3. **Check sample, duration, peeking and change log.** If under plan or stopped early without a sequential design, the decision is "inconclusive"; continue only to describe the data. If an unplanned promotion or deploy hit one arm, the decision is "invalid".
4. **Primary metric**: compute effect, SE, z, p, CI (best practices 6–7). Compare the CI with the MDE.
5. **Guardrails**: effect and CI for each, against its limit.
6. **Novelty**: lift and CI per exposure week.
7. **Segments**: device and new/returning, labelled explanatory.
8. **Decide**:
   - **Ship**: SRM passed, planned sample reached, primary CI excludes zero in the right direction, no guardrail CI crosses its limit.
   - **Don't ship**: primary CI excludes zero in the wrong direction, or a guardrail fails.
   - **Provisional ship**: all ship conditions met except that a guardrail has only a point estimate (data for its CI missing). Name the missing data; the rollout is not final until the CI is checked.
   - **Inconclusive**: everything else, including a significant result that failed a process gate. Say what it rules out ("an effect larger than +X% is unlikely").
   - Add a holdback recommendation if novelty is suspected. Add a re-check date for lagging guardrails (returns).
9. Write the output and run the quality checklist.

## Pitfalls and edge cases
- **Testing-tool dashboards** often show "probability to beat" from Bayesian models with their own priors, or run continuous frequentist tests. Recompute from counts rather than quoting the dashboard.
- **Flicker or redirect tests** lose users in the redirected arm (slow page, blocked scripts), which is a classic SRM cause and biases conversion.
- **Conversions counted by GA4** under consent mode are observed only for consenting users. That is fine if consent is balanced across arms, which should be checked. Backend orders joined on the assignment ID are better.
- **Bots** concentrated in one arm show up as an SRM on exposed units and as a flat conversion in that arm.
- **Returns-driven guardrails** need data after the test ends. A ship decision is provisional until the return window has passed.
- **Multiple variants (A/B/C)**: SRM uses df = arms − 1. Comparisons against control need a correction for the number of comparisons.
- **A p of 0.07 is not "trending significant"**, and a p of 0.049 after five peeks is not significant either.
- **Sample counted in sessions while randomised by user** inflates n and shrinks the CI (best practice 2).

## Rules
- Read-only. Do not stop, start, re-weight or ship tests in any tool. The decision is a recommendation for a human owner.
- Validity gates come before results. A test that fails SRM gets no effect statement.
- Judge on the pre-registered primary metric only. Everything else is labelled exploratory.
- Never invent missing counts, variances or plan values. If the plan's MDE or α is missing, ask. Do not assume 5% and 80% silently. If you must proceed, state the assumption in the output.
- Report every effect with its interval. Never write "trending", "almost significant" or "directionally positive" as a ship reason.
- No user-level data (IDs, e-mails) in the output.

## Output format
```
Test readout: <name> · <start> – <end> · unit: <user|session> · split <x/y>
Plan: primary <metric> · α <0.05> two-sided · power <80%> · MDE <x% rel> · planned n <n>/arm · stopping <fixed|method>

## Validity
SRM (assigned): <nA> / <nB> vs <split> · χ² = <x> · p = <p> · <OK|FAIL>
SRM (exposed):  ... 
Sample: <n total> vs <planned> · duration <d days, full weeks: yes/no> · peeking: <none|per plan|unplanned>
Change log: <none | items>

## Primary metric: <definition>
A <pA> (<cA>/<nA>) · B <pB> (<cB>/<nB>)
Abs diff <d pp> (95% CI <lo> to <hi>) · rel <x%> (approx. CI <lo%> to <hi%>) · z <z> · p <p>
MDE check: CI <includes|excludes> planned MDE

## Guardrails
| metric | A | B | effect (95% CI) | limit | status |

## Time
| exposure week | lift | 95% CI |

## Segments (explanatory only)
...

## Decision: <Ship | Provisional ship | Don't ship | Inconclusive | Invalid>
Why: <one line per gate>
Follow-up: <holdback / re-check date / next test>
```

## Worked example
Illustrative numbers. The test puts a delivery date next to the add-to-cart button on the product page. It ran 1–21 September 2026 (3 full weeks), randomised by user, 50/50. The plan was: primary = purchase conversion per user; α = 0.05 two-sided; 80% power; MDE +10% relative on a 2.84% baseline; fixed horizon. That gives n = (1.96 + 0.84)² × (0.0284 × 0.9716 + 0.03124 × 0.96876) ÷ 0.00284² ≈ 56,300 per arm. Guardrail: AOV not worse than −3%.

SRM: 56,900 vs 56,500 against 56,700 expected each, so χ² = (200² + 200²) ÷ 56,700 = 1.41 and p = 0.23 (OK).

Primary: A 1,616 ÷ 56,900 = 2.840%. B 1,735 ÷ 56,500 = 3.071%. d = 0.231 pp; SE = 0.101 pp; z = 2.29; p = 0.022; 95% CI +0.034 to +0.428 pp, or approx. +1.2% to +15.1% relative (point +8.1%).

Weekly, by first exposure:

| week | A | B | lift | 95% CI (rel., approx.) |
|---|---|---|---|---|
| 1 | 535/19,100 | 592/18,950 | +11.5% | −0.6% to +23.7% |
| 2 | 540/18,950 | 580/18,800 | +8.3% | −3.8% to +20.3% |
| 3 | 541/18,850 | 563/18,750 | +4.6% | −7.3% to +16.5% |

```
Test readout: PDP delivery date near button · 2026-09-01 – 2026-09-21 · unit: user · split 50/50
Plan: primary purchase conversion/user · α 0.05 two-sided · power 80% · MDE +10% rel · planned n 56 300/arm · fixed horizon

## Validity
SRM (assigned): 56 900 / 56 500 vs 50/50 · χ² = 1.41 · p = 0.23 · OK
Sample: 113 400 vs 112 600 planned · 21 days, full weeks: yes · peeking: none
Change log: none

## Primary metric
A 2.840% (1 616/56 900) · B 3.071% (1 735/56 500)
Abs diff +0.231 pp (95% CI +0.034 to +0.428) · rel +8.1% (approx. CI +1.2% to +15.1%) · z 2.29 · p 0.022
MDE check: CI includes +10%; point estimate below MDE, expect a smaller effect after launch

## Guardrails
| AOV | 1 980 | 1 962 | −0.9% (CI not computed: order-level data not provided) | −3% | point estimate within limit; confirm with CI |
Revenue per user: A 56.23 · B 60.25 (+7.1%), point estimate only: user-level revenue not provided

## Time
Week lifts +11.5% · +8.3% · +4.6% (week 3 CI −7.3% to +16.5%) → declining, possible novelty

## Decision: Provisional ship, with a 10% holdback for 4 weeks
Why: SRM OK · planned sample reached · primary CI excludes zero · AOV point estimate within limit (CI pending)
Follow-up: re-read holdback on 2026-10-26; recompute AOV CI from order-level data before rollout is final
```

## Quality checklist
- The plan is quoted, and the primary metric is the pre-registered one. Missing plan elements are flagged.
- SRM was computed with χ², p and threshold shown, on assigned (and exposed, if available) units.
- The randomisation unit matches the analysis unit, or the delta method or aggregation is stated.
- Every effect has absolute and relative values, a CI and a p. Revenue metrics use a skew-robust method or are marked as point estimates.
- There is no "observed power". The MDE is compared with the CI.
- Guardrails are judged against their limits, and lagging guardrails have a re-check date.
- Novelty was checked by exposure week. Segments are labelled explanatory.
- The decision follows the stated rules exactly, and all numbers recompute from the counts shown.

## Sources
- Fabijan et al., Diagnosing Sample Ratio Mismatch in Online Controlled Experiments, KDD 2019: https://doi.org/10.1145/3292500.3330722
- Johari, Pekelis, Walsh, Always Valid Inference: Bringing Sequential Analysis to A/B Testing: https://arxiv.org/abs/1512.04922
- Johari et al., Peeking at A/B Tests, KDD 2017: https://doi.org/10.1145/3097983.3097992
- Deng, Xu, Kohavi, Walker, Improving the Sensitivity of Online Controlled Experiments by Utilizing Pre-Experiment Data (CUPED), WSDM 2013: https://doi.org/10.1145/2433396.2433413
- Kohavi, Tang, Xu, Trustworthy Online Controlled Experiments, Cambridge University Press, 2020: https://experimentguide.com/
- NIST/SEMATECH e-Handbook of Statistical Methods, chi-square goodness-of-fit test: https://www.itl.nist.gov/div898/handbook/eda/section3/eda35f.htm

## License
MIT
