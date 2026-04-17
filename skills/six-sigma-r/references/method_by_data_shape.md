# Method selection — by data shape

Use when the tool choice hinges on what *kind* of data is in hand. Typical
trigger: the user describes the measurement and asks "what should I do with
this?" — *"I have 50 go/no-go inspections per day for 30 days"*,
*"I have temperature readings every 10 minutes"*, *"I have categorical defect
types"*.

Companion references:
- [by DMAIC phase](method_by_dmaic.md) — when the user is organized around
  the project framework.
- [by problem type](method_by_problem.md) — when the user describes a
  business problem rather than a data type.

## Start here: what are you measuring?

- **Continuous** (length, weight, time, temperature, pressure) → **section A**
- **Counts of defects** (scratches per part, complaints per day) → **section B**
- **Defectives — pass/fail per unit** → **section C**
- **Categorical** (type A / type B / type C labels) → **section D**

The key distinction between B and C: a *defective* is a unit that fails
overall (one per unit); a *defect* is a flaw that can occur multiple times
on the same unit.

## A. Continuous measurements

| Goal                                         | Technique                                    | Reference                        |
| -------------------------------------------- | -------------------------------------------- | -------------------------------- |
| Monitor process over time, n=1 per reading   | I-MR chart                                   | `control_charts.md`              |
| Monitor over time, n=2–10 rational subgroups | X-bar R chart                                | `control_charts.md`              |
| Monitor over time, n>10 subgroups            | X-bar S chart                                | `control_charts.md`              |
| Is mean different from target?               | 1-sample t (Wilcoxon signed-rank if non-normal) | `hypothesis_tests.md`         |
| Are two sample means different?              | 2-sample t (Mann-Whitney if non-normal)      | `hypothesis_tests.md`            |
| Before/after on same units?                  | Paired t (paired Wilcoxon if non-normal)     | `hypothesis_tests.md`            |
| Are 3+ group means different?                | One-way ANOVA (Kruskal-Wallis if non-normal) | `anova.md`                       |
| Two continuous variables related?            | Correlation, then regression                 | `correlation_regression.md`      |
| Process capable vs spec?                     | Cp/Cpk/Pp/Ppk                                | `capability_sigma_level.md`      |
| Variation shape / outliers?                  | Histogram, box plot, normal check            | `charts_visualization.md`, `descriptive_stats.md` |

**Always check normality** (`shapiro.test` or a Q-Q plot) before using a
t-test, ANOVA, or a parametric capability index on a small-to-moderate
sample. On capability indices, non-normality silently overstates or
understates the true fraction outside spec — use the percentile method
(see `capability_sigma_level.md`) instead.

## B. Counting defects (multiple per unit possible)

Defects = flaws or events counted within a unit, a time interval, or an
inspection area. A single item can carry several defects.

| Situation                                                          | Technique                         | Reference               |
| ------------------------------------------------------------------ | --------------------------------- | ----------------------- |
| Monitor rate over time, constant opportunity size                  | C chart                           | `control_charts.md`     |
| Monitor rate over time, varying opportunity size                   | U chart                           | `control_charts.md`     |
| Probability of seeing k events given rate λ                        | Poisson pmf                       | `distributions.md`      |
| Comparing counts between two systems                               | 2-sample Poisson test or chi-sq.  | `hypothesis_tests.md`   |

Common mistake: using a P chart for defect counts. P is for defective
*units*, not defect *counts*. If you want to know "how many flaws per
piece", use C or U.

## C. Counting defectives (yes/no per unit)

Defectives = units classified overall as pass or fail. Each unit contributes
a 0 or a 1.

| Situation                                                    | Technique                         | Reference               |
| ------------------------------------------------------------ | --------------------------------- | ----------------------- |
| Monitor proportion over time, constant sample size           | NP chart                          | `control_charts.md`     |
| Monitor proportion over time, varying sample size            | P chart                           | `control_charts.md`     |
| Probability of k defectives in n trials given rate p         | Binomial pmf                      | `distributions.md`      |
| Is proportion different from target?                         | 1-sample proportion test          | `hypothesis_tests.md`   |
| Are two proportions different?                               | 2-sample proportion test          | `hypothesis_tests.md`   |
| How many samples to detect a change in proportion?           | `power.prop.test`                 | `sample_size_power.md`  |

## D. Categorical data

Named categories without a natural numeric order (defect type, claim
reason, supplier, failure mode).

| Situation                                                     | Technique                         | Reference               |
| ------------------------------------------------------------- | --------------------------------- | ----------------------- |
| Frequency distribution matches expected?                      | Chi-square goodness-of-fit        | `hypothesis_tests.md`   |
| Two categorical variables independent?                        | Chi-square independence           | `hypothesis_tests.md`   |
| Rank categories by frequency for prioritization?              | Pareto chart                      | `charts_visualization.md` |

## Quick cross-cuts

- **Subgroup size determines the control chart.** n=1 → I-MR; 2–10 → X-bar
  R; >10 → X-bar S. Use `control_charts.md`'s constant table for A2, D3,
  D4, d2, etc.
- **Count vs proportion changes the chart.** Defects → C/U. Defectives →
  NP/P. These are not interchangeable.
- **Clamp LCL to 0** for any chart whose statistic is a count or a rate
  (P, NP, C, U, MR) — negative limits don't exist for those.
- **Paired beats two-sample** when the same unit is measured twice; paired
  tests remove between-unit variation and are almost always more powerful.
- **Normality check matters more for capability than for hypothesis tests.**
  t-tests are robust at moderate n; Cpk is not.

## What this skill does NOT cover

See `method_by_dmaic.md` for the full list with pointers. Short version:
MSA / Gage R&R, DOE / factorial experiments, project artifacts (charter,
SIPOC, fishbone), and cost/benefit models are out of scope.
