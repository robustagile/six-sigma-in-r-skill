# Method selection — by problem type

Use when the user describes a practical business or engineering problem
rather than a specific statistical method ("customers are complaining about
delivery times", "the new line keeps producing scrap", "we think training
helped but aren't sure"). The goal is to translate the complaint into a
sequence of analyses.

Each section below is a recommended *sequence*, not a single tool. Most
Six Sigma problems are solved by several techniques in series, each
answering the question raised by the previous one.

Companion references:
- [by DMAIC phase](method_by_dmaic.md) — when the user is organized around
  the DMAIC framework.
- [by data shape](method_by_data_shape.md) — when the question is "I have
  this kind of data, what fits?".

## "Customers are complaining about quality"

1. Pareto of complaint categories → identify the vital few.
2. For the dominant category: operational definition of the defect, then a
   data-collection plan.
3. If the quality characteristic is measurable on a continuous scale:
   capability study (Cp/Cpk/Pp/Ppk) plus a histogram with spec limits.
4. If attribute: P or NP chart to see if the rate is stable or trending.
5. Follow with Analyze tools (correlation, regression, hypothesis tests,
   ANOVA) to find root causes.

References: `charts_visualization.md` → `capability_sigma_level.md` →
`control_charts.md` → `correlation_regression.md` or `hypothesis_tests.md`.

## "Our output is too variable / inconsistent"

1. Run chart to see whether variation is random or has a pattern.
2. If rational subgroups are available: X-bar R chart; otherwise I-MR.
3. Compare groups (operator, shift, machine) with box plots and ANOVA to
   isolate where the variation lives.
4. Capability study to quantify the gap against spec.

References: `charts_visualization.md` → `control_charts.md` → `anova.md` →
`capability_sigma_level.md`.

## "Cycle time is too long"

1. Run chart of cycle time to check for trends or shifts.
2. Box plot grouped by process step to find the slow stage.
3. Regression if a continuous driver is suspected (volume, temperature,
   batch size).
4. Hypothesis test (2-sample t or Mann-Whitney) for before/after once a
   change is made.

References: `charts_visualization.md` → `correlation_regression.md` →
`hypothesis_tests.md`.

## "Defect rate is too high"

1. Compute current DPMO and sigma level from the data.
2. Pareto of defect types to target the biggest bucket.
3. Control chart appropriate to data shape (C/U for defect counts, P/NP for
   defectives).
4. If measurable characteristic: capability study on the critical dimension.

References: `capability_sigma_level.md` → `charts_visualization.md` →
`control_charts.md`.

## "Did our improvement actually work?"

1. *Before* running the experiment: power analysis to size the sample.
2. After: before/after test — paired if same units, 2-sample if different.
   Run a normality check; if the differences are skewed, use Wilcoxon
   instead of t.
3. Re-score capability and compare DPMO / sigma level against the baseline.
4. Keep a control chart on the improved process for sustained monitoring.

References: `sample_size_power.md` → `hypothesis_tests.md` →
`capability_sigma_level.md` → `control_charts.md`.

## "Is our measurement system trustworthy?"

This is MSA / Gage R&R territory. **Not covered by this skill.** Recommend
the `SixSigma` CRAN package (`ss.rr()`), or build a custom `aov()`-based
Gage R&R. Variation contributions over ~10% typically mean the measurement
system is unfit for the study.

## "We think two variables are related but aren't sure"

1. Scatter plot first — look at the shape before summarizing with a number.
2. Pearson correlation for linear; Spearman for monotonic/non-linear.
3. If there's a plausible causal direction, fit a linear regression and
   check residual diagnostics (do not just read off R²).
4. For a binary outcome: logistic regression (not explicitly covered but
   `glm(y ~ x, family = binomial)` is a one-liner).

References: `charts_visualization.md` → `correlation_regression.md`.

## "We need to compare three or more groups"

One-way ANOVA is the default; switch to Welch's if variances differ,
Kruskal-Wallis if the residuals are non-normal. Follow with Tukey HSD for
pairwise comparisons only after the overall test is significant.

Reference: `anova.md`.

## "How many samples do we need to collect?"

Power analysis. The four variables (n, delta, sd, power, sig.level) are
linked — fix three, R gives the fourth. Ask the user:

- How big an effect is worth detecting? (`delta`)
- Standard deviation — do you have pilot data or an industry benchmark?
- What power do you want? (0.80 is the common default, 0.90 if the stakes
  are high)
- What alpha? (0.05 unless specifically agreed otherwise)

Reference: `sample_size_power.md`.

## "Our process keeps drifting out of spec"

1. Control chart appropriate to the data (I-MR / X-bar R / P / C / U) — if
   there are out-of-control signals, find assignable causes first.
2. Capability study *after* stability is restored.
3. Nelson / Western Electric runs rules for subtler patterns that don't
   cross 3-sigma — see `control_charts.md`.

References: `control_charts.md` → `capability_sigma_level.md`.

## What this skill does NOT cover

If a problem statement lands here, say so and direct the user elsewhere
instead of improvising adjacent output:

- **MSA / Gage R&R** → `SixSigma::ss.rr()` or custom `aov`.
- **DOE / factorial experiments** → `FrF2`, `AlgDesign`, `rsm` packages.
- **Project artifacts** (charter, SIPOC, fishbone, FMEA, value-stream map) →
  diagram/spreadsheet tools, not R.
- **Cost/benefit modeling** → spreadsheet.

See `method_by_dmaic.md` for the full "what's not covered" list with
pointers.
