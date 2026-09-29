---
name: ab-test-readout
owner: checkoutlab
category: Conversion & UX
description: Reads out an A/B test honestly: SRM check, sample size against the plan, significance and intervals, guardrail metrics and novelty effect before any "winner" call.
version: v3
license: MIT
updated: 2026-09-25
recommended: false
security_checked: true
url: https://ecomdly.com/skills/checkoutlab/ab-test-readout
raw: https://ecomdly.com/raw/checkoutlab/ab-test-readout.md
install: npx @ecomdly/cli add checkoutlab/ab-test-readout
---

# A/B test readout

Most "winning" tests are noise stopped early. This skill checks whether the test is valid before it looks at the result, then reports the effect with its uncertainty and a clear decision.

## When to use
- A test has reached its planned end date or sample size.
- Someone wants to stop a test early because it "looks significant" — run this to see if that holds.

## Input
Per variant: users (or sessions, as randomised), conversions, revenue, and the guardrail metrics. The test plan: hypothesis, primary metric, planned split, minimum detectable effect (MDE), planned sample size, start and end dates. Daily or weekly breakdown if available.

## Steps
1. **SRM check.** Chi-square test of observed vs planned split. p < 0.01 means sample ratio mismatch: stop, report likely causes (redirect loss, bot filtering, tracking firing in one variant only) and make no result call.
2. **Sample and duration.** Compare to the planned sample size. Under plan = inconclusive. Require at least one full week, ideally two, so weekday patterns are covered.
3. **Primary metric.** Relative lift, absolute difference, 95% confidence interval, p-value (two-sided z-test for proportions; for revenue per user use a t-test or bootstrap, as revenue is skewed). State the observed power against the MDE.
4. **Guardrails.** Check average order value, refund or return rate, page load time, and unsubscribes where relevant. A guardrail that worsens beyond its threshold blocks a ship call.
5. **Novelty and time.** Lift by week. A lift that shrinks each week is likely novelty; report the last week's lift next to the total.
6. **Segments.** Device and new vs returning only as explanations, not as new winners. With many segments, say that some will look significant by chance.

## Decision rules
- **Ship:** valid SRM, planned sample reached, CI excludes zero, guardrails fine.
- **Don't ship:** CI excludes zero in the wrong direction, or a guardrail fails.
- **Inconclusive:** anything else. Say so plainly; an inconclusive test is a result, not a failure.
- Never round up a p of 0.07 into "trending significant", and never report a result from a test that failed SRM.

## Output format
```
Test: PDP delivery date near button · 2026-09-01 to 2026-09-21
SRM: 50.3 / 49.7 vs 50/50 · p = 0.41 · OK
Sample: 41 200 / 40 000 planned · 3 weeks
Primary (purchase conversion): A 2.84% · B 3.02% · +6.3% rel (95% CI +0.4% to +12.2%) · p = 0.036
Guardrails: AOV -0.8% (ok, limit -3%) · return rate +0.1 pp (ok)
Weekly lift: +9.1% · +5.8% · +4.2%  -> possible novelty, re-check in 4 weeks
Decision: Ship, with a holdback check
```

## License
MIT
