---
name: six-sigma-r
description: Generates R implementations for Six Sigma / statistical-process-control tasks — control charts (X-bar R, I-MR, P, NP, C, U), process capability (Cp/Cpk/Pp/Ppk), DPMO and sigma level, Pareto and run charts, histograms, box plots, scatter/regression, correlation, hypothesis tests (1- and 2-sample t, proportion, Wilcoxon, chi-square), one-way and regression ANOVA, binomial/Poisson/normal distribution calculations, and sample-size / power analysis. It ALSO recommends which technique to use when the user describes a practical problem, a DMAIC phase, or just a data shape — "customers are complaining about delivery times", "we're in the Analyze phase, where do we start", "I have pass/fail data per day, what chart applies" — and then produces the R code for the recommended technique. Use this skill whenever the user asks for R code related to quality improvement, process control, DMAIC analysis, defect-rate statistics, capability studies, or any of the techniques named above — even when they don't say "Six Sigma" explicitly. Also trigger on phrases like "control chart in R", "Cpk in R", "Pareto of defects", "is this process capable", "sample size for a t-test", "I-MR chart", "rational subgroups", "which Six Sigma tool should I use", "we're in the Measure/Analyze/Improve/Control phase", "how do I decide between X and Y chart", or a Minitab-style analysis request that needs to be done in R instead.
---

# Six Sigma in R

Produce correct, runnable R code for Six Sigma and statistical-process-control problems. The skill is organized by *task*, not by statistical method — a user rarely asks for "a chi-square test", they ask "is my defect rate different from the target?". Map the request to the right technique, then emit code.

## Before writing code: confirm two choices

Different users want different output shapes. Before producing code, ask the user once per session (skip if the request already makes it obvious, or if the user has answered previously):

1. **Script or function?**
   - *Script*: a self-contained, top-to-bottom `.R` file with the user's data (or a clearly marked placeholder) baked in. Run-it-and-see.
   - *Function*: a reusable `my_cpk(x, lsl, usl)`-style definition the user can drop into their project and call on arbitrary data.
2. **Plot output: file or inline?**
   - *File*: wrap plot calls in `png(filename="...")` / `dev.off()` — good for scripts run from the terminal.
   - *Inline*: no device wrapper — plot renders in the user's RStudio / Quarto / notebook session.

If the user phrases the request as "write me a function that…", default to function + inline and don't ask. If they say "a script to analyze this CSV", default to script + file. Ask only when ambiguous.

## Plotting: base R by default

Use base R graphics (`plot`, `barplot`, `boxplot`, `hist`, `pie`, `abline`, `lines`, `points`, `segments`) as the default. Base R covers every chart this skill produces — including horizontal bars (`barplot(..., horiz=TRUE)`), stacked bars (pass a matrix), and Pareto overlays. Base R has no external dependency and keeps scripts short.

Use `ggplot2` only when the user explicitly asks for it, when they mention tidyverse / `ggplot`, or when a visualization genuinely benefits (e.g. grouped scatter with color aesthetics). When you do use ggplot2, load it explicitly with `library(ggplot2)` at the top.

## Map: what the user wants → which reference to read

The references below contain the working code templates. Read the one that matches the task before emitting code — don't guess formulas from memory, especially for control-chart limits and capability indices, where the constants matter.

If the user doesn't name a technique and instead describes a practical problem or a DMAIC phase, start with the method-selection references. They translate practical framings ("customers are complaining about cycle time", "we're in the Control phase", "I have pass/fail data per day") into a technique recommendation, then forward to the code-level reference.

| User's framing                                              | Reference file                               |
| ----------------------------------------------------------- | -------------------------------------------- |
| *"Where do I start? We're in the Analyze phase."*           | `references/method_by_dmaic.md`              |
| *"Customers are complaining about X, what do we do?"*       | `references/method_by_problem.md`            |
| *"I have this kind of data — what fits?"*                   | `references/method_by_data_shape.md`         |
| Summary statistics, percentiles, population vs sample       | `references/descriptive_stats.md`            |
| Pareto chart, run chart, histogram, bar, box, scatter, pie  | `references/charts_visualization.md`         |
| Is my process in control? (SPC charts)                      | `references/control_charts.md`               |
| Is my process capable? Cp, Cpk, Pp, Ppk, DPMO, sigma level  | `references/capability_sigma_level.md`       |
| Probability of k defects / k events                         | `references/distributions.md`                |
| Relationship between two variables (cor / regression)       | `references/correlation_regression.md`       |
| Is A different from B? (hypothesis tests)                   | `references/hypothesis_tests.md`             |
| Compare >2 group means (one-way ANOVA)                      | `references/anova.md`                        |
| "How many samples do I need?" (power analysis)              | `references/sample_size_power.md`            |

If a request spans multiple categories (common — e.g. "capability study on this data, then plot an I-MR chart"), read both references and stitch the outputs.

The three `method_by_*.md` references are *selection* aids — they recommend which technique to use and always forward to one of the lower references for the actual code template. Read the selection reference first when the user hasn't named a technique, then read the technique reference before writing code.

## Six Sigma context to keep in mind

These shape correct code, not just pretty code:

- **Subgroup size matters for control charts.** X-bar R and X-bar S charts use different constants (A2/A3, D3/D4, B3/B4) depending on subgroup size `n`. For `n = 1` (individuals data) use an I-MR chart, not X-bar R. `references/control_charts.md` has the constant tables.
- **Cpk vs Ppk are different things.** Cp/Cpk use *within-subgroup* standard deviation (short-term, process potential). Pp/Ppk use *overall* standard deviation (long-term, actual performance). Don't conflate them. `references/capability_sigma_level.md` covers both.
- **Attribute vs variable data drives the chart choice.** Counts of defectives → P or NP chart. Counts of defects → C or U chart. Continuous measurements → X-bar R / X-bar S / I-MR. The control-charts reference has a decision tree.
- **The "1.5 sigma shift" is convention, not math.** When converting between long-term sigma level and short-term, practitioners typically add 1.5. Make this explicit in the output, don't bake it in silently.
- **Lower control limits can't go negative** when the statistic is a count or rate. Clamp LCL to 0 for P, NP, C, U, and MR charts.
- **Assume `data.frame` with named columns** when reading CSV unless the user shows a different shape. Use `read.csv(..., header = TRUE)` and reference columns by `$name`.

## Output conventions

- One R file per request unless the user asks otherwise. Name it after the task (e.g. `pareto_defects.R`, `pchart_returns.R`), not the data.
- Put constants, thresholds, and tolerances at the top of the script as named variables — never hardcode them mid-expression. Makes the script easy to re-run on new data.
- Print interpretive output, not just numbers. After a hypothesis test, compare `p.value` to `alpha` and print "reject H0" / "fail to reject H0". After a capability study, state whether `Cpk >= 1.33` (typically-acceptable threshold). The user will copy the code into a report.
- Don't add apologetic comments about edge cases that don't apply. Do add a one-line comment above any formula whose choice isn't obvious (e.g. "using unbiased sigma estimator σ̂ = R̄ / d2").
- If the user's data is missing, invent a small, realistic placeholder (5–30 rows) and mark it with a `# ---- replace with your data ----` banner so they know where to edit.
