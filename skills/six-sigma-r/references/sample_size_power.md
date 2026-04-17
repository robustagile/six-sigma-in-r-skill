# Sample size and statistical power

"How many samples do I need?" or "With n samples, what effect size can I detect?" — every Measure/Analyze phase planning conversation.

The four quantities of a power calculation are linked: fix any three, R gives you the fourth.

- **n**: sample size per group (for 2-sample tests) or total n (for 1-sample).
- **delta**: effect size to detect, in the same units as the measurement.
- **sd**: standard deviation (known or estimated from pilot data).
- **sig.level**: α, the Type-I error rate (commonly 0.05).
- **power**: 1 − β, probability of detecting the effect when it's real (commonly 0.8 or 0.9).

Pass `NULL` for the quantity you want R to solve for — but the functions let you omit it too.

## t-test power

```r
# Sample size to detect delta = 0.5 at sd = 1, alpha = 0.05, power = 0.8 (one-sample, one-sided)
power.t.test(delta = 0.5, sd = 1,
             sig.level = 0.05, power = 0.8,
             type = "one.sample", alternative = "one.sided")

# Power given n (how much can I detect with 18 samples?)
power.t.test(n = 18, delta = 0.5, sd = 2.2,
             sig.level = 0.05,
             type = "one.sample", alternative = "one.sided")

# 2-sample case
power.t.test(delta = 1, sd = 2.2,
             sig.level = 0.05, power = 0.8,
             type = "two.sample", alternative = "one.sided")

# Paired (before/after on the same unit)
power.t.test(delta = 1, sd = 2.2,
             sig.level = 0.05, power = 0.8,
             type = "paired")
```

For the 2-sample case, `n` in the output is per group — if the function returns `n = 32`, you need 32 in A *and* 32 in B.

## Proportion-test power

```r
# Detect a shift from 92 % to 97 % at alpha = 0.05, power = 0.8
power.prop.test(p1 = 0.92, p2 = 0.97,
                sig.level = 0.05, power = 0.8,
                alternative = "one.sided")

# Given n, find achievable power
power.prop.test(n = 200, p1 = 0.92, p2 = 0.97,
                sig.level = 0.05, alternative = "one.sided")
```

## ANOVA power

Uses `pwr` package (not in base R). If the user can install it:

```r
library(pwr)
# k groups, effect size f (Cohen), alpha, power
pwr.anova.test(k = 4, f = 0.25, sig.level = 0.05, power = 0.8)
```

Cohen's f conventions: 0.10 = small, 0.25 = medium, 0.40 = large. When translating from the user's own numbers, `f = sqrt(between_var / within_var)`.

## Reporting the result

Whatever's being solved for, always print in the script:

- The three inputs, clearly labeled.
- The solved-for quantity, rounded sensibly (sample size → `ceiling()`; power → 3 decimals).
- A one-line restatement: "To detect a 0.5-unit shift at α = 0.05 with 80% power, n ≥ 32 per group."

```r
res <- power.t.test(delta = 0.5, sd = 1,
                    sig.level = 0.05, power = 0.8,
                    type = "two.sample", alternative = "two.sided")
cat(sprintf("n per group >= %d (delta=%.2f, sd=%.2f, alpha=%.2f, power=%.2f)\n",
            ceiling(res$n), res$delta, res$sd, res$sig.level, res$power))
```

## Common pitfalls

- Use a *realistic* sd. Guessing too low gives an optimistic (undersized) sample. If you have pilot data, use it; otherwise inflate conservatively.
- For `power.prop.test` the result can be very sensitive to small changes in p1/p2 — report n for a couple of nearby scenarios.
- `alternative = "one.sided"` halves the required n relative to two-sided — only use it when the direction is genuinely known in advance.
