# Six Sigma in R — AI Skill

**Version 1.0.0** — released 2026-04-17 (see [VERSION](./VERSION))

A reusable skill that teaches Claude Code (and compatible Codex-style agents) how
to produce correct, runnable **R** code for typical **Six Sigma / statistical
process control** tasks.

When this skill is installed, asking your coding agent something like "plot a
P chart from this CSV" or "is this process capable?" yields an R script (or a
reusable function) that uses the right formulas and constants, not
approximations.

## Release notes

### 1.0.0 (2026-04-17)

Initial release. Skill covers the full code-based Six Sigma body of knowledge
listed below, plus a **method-selection layer** (by DMAIC phase, by problem
type, by data shape) that recommends which technique to reach for when the
user describes a practical problem instead of naming a specific test or chart.

**Evaluation summary (Claude Opus 4.7):**
- On mainstream Six Sigma tasks, the skill and a bare model produce equally
  correct R (30/30 = 30/30 across 5 prompts).
- On harder tasks (correct SPC constants for n=5, full Nelson rules, non-
  normal capability, within-vs-overall sigma for subgrouped Cp/Cpk), same
  result (33/33 = 33/33 across 5 prompts).
- On method-selection + implementation prompts the skill wins by +2 on 36
  assertions (36/36 vs 34/36). The differentiators: correct analysis-order
  sequencing (stability before capability) and explicit scope transparency
  on out-of-scope topics (MSA / Gage R&R).

Full benchmarks, outputs, and side-by-side viewers live under
`eval-workspace/iteration-{1,2,3}/`.

## What the skill covers

| Area                          | Techniques                                                                 |
| ----------------------------- | -------------------------------------------------------------------------- |
| Descriptive statistics        | mean, median, sample vs population var/sd, percentiles, skewness, kurtosis |
| Charts                        | Pareto, run chart, histogram, box plot, scatter, bar / stacked bar, pie    |
| Control charts (SPC)          | X-bar R, X-bar S, I-MR, P, NP, C, U — with standard A2/D3/D4/d2 constants  |
| Process capability            | Cp, Cpk, Pp, Ppk, Z-bench, normality check                                 |
| Sigma level / DPMO            | DPMO, long- and short-term sigma (with and without the 1.5σ shift)         |
| Probability distributions     | Binomial, Poisson, Normal — density, CDF, quantile                         |
| Correlation & regression      | Pearson/Spearman, simple and multiple linear regression, residual diagnostics |
| Hypothesis tests              | 1- / 2-sample t, paired t, 1- / 2-sample proportion, Wilcoxon, chi-square  |
| ANOVA                         | One-way `aov`, two-way, post-hoc (Tukey HSD), regression-ANOVA             |
| Sample size / power analysis  | `power.t.test`, `power.prop.test`, ANOVA power                             |
| Method selection              | Recommends which technique fits by DMAIC phase, problem type, or data shape |

## Behaviour conventions baked in

- **Base R by default** for plotting; ggplot2 only on request.
- **Script vs function** and **file vs inline plots** — the skill asks the user
  once when it's ambiguous, then sticks to the choice.
- Proper control-chart constants are used (A2, D3, D4, d2, A3, B3, B4, c4);
  LCLs are clamped to 0 where the statistic is a count or rate.
- Cp/Cpk (within-subgroup σ, short-term) and Pp/Ppk (overall σ, long-term) are
  treated as distinct — not conflated.
- Hypothesis-test scripts compare `p.value` to `alpha` and print a verdict,
  not just the raw test object.
- Every reference sheet explains *why* a choice is made, so edge cases stay
  judgeable.

## Repository layout

```
.
├── README.md                 # this file
├── INSTALL.md                # install for Claude Code or Codex, global or per-project
└── skills/
    └── six-sigma-r/
        ├── SKILL.md          # the skill entry point
        └── references/
            ├── method_by_dmaic.md         # "we're in the Analyze phase, what tools?"
            ├── method_by_problem.md       # "customers are complaining about X"
            ├── method_by_data_shape.md    # "I have this kind of data, what fits?"
            ├── descriptive_stats.md
            ├── charts_visualization.md
            ├── control_charts.md
            ├── capability_sigma_level.md
            ├── distributions.md
            ├── correlation_regression.md
            ├── hypothesis_tests.md
            ├── anova.md
            └── sample_size_power.md
```

The three `method_by_*.md` files are **selection aids** — they take a
practical framing (a DMAIC phase, a business problem, a data shape) and
recommend which technique to use, forwarding to one of the code-level
references. The skill uses them when the user hasn't named a specific
technique.

## Prerequisites on the user's side

The skill produces R source code — running it needs a local R installation:

1. Install R: <https://cran.r-project.org/>
2. Add R's `bin/` folder to `PATH` so `Rscript` is available from the shell.
3. For ggplot2-based requests, install the package once:
   `install.packages("ggplot2")`.
4. For skewness / kurtosis without hand-coded formulas: `install.packages("e1071")`.
5. For ANOVA power (`pwr.anova.test`): `install.packages("pwr")`.

The skill does not assume any package beyond base R; extra packages are only
brought in when the user's request requires them.

## Installation

See [INSTALL.md](./INSTALL.md) for step-by-step instructions to install the
skill **globally** (available in every project) or **locally** (scoped to a
single project), for both **Claude Code** and **Codex**.

## License

The code and skill content in this repository are distributed under the
Apache 2.0.
