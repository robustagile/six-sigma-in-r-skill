# Hypothesis tests

Use when the user frames a question as "is A different from B?", "is this value different from the target?", "is the defect rate actually above 5%?".

## Choosing a test

| Question                                             | Test                      | R function   |
| ---------------------------------------------------- | ------------------------- | ------------ |
| "Is the mean of a continuous sample different from target µ₀?" | 1-sample t-test | `t.test(x, mu = mu0)` |
| "Are two group means different?"                     | 2-sample t-test           | `t.test(a, b)` |
| "Are paired measurements different?" (before/after on the same unit) | paired t-test | `t.test(a, b, paired = TRUE)` |
| "Is the proportion of defectives different from p₀?" | 1-sample proportion test  | `prop.test(x, n, p = p0)` |
| "Are two proportions different?"                     | 2-sample proportion test  | `prop.test(c(x1, x2), c(n1, n2))` |
| Non-normal continuous data, "is the median ≠ target?" | Wilcoxon signed-rank     | `wilcox.test(x, mu = mu0)` |
| Non-normal, two independent groups                   | Wilcoxon rank-sum (Mann-Whitney) | `wilcox.test(a, b)` |
| "Do observed counts match an expected distribution?" | Chi-square goodness-of-fit | `chisq.test(obs, p = expected_props)` |
| "Are two categorical variables independent?"         | Chi-square independence   | `chisq.test(table(x, y))` |
| Compare >2 group means                               | One-way ANOVA             | see `anova.md` |

## The standard output pattern

Every hypothesis-test script should produce the same three things:

1. The test call + printed result (so the user sees CI, p-value, alternative).
2. An explicit comparison of `p.value` to `alpha` with a readable conclusion.
3. A comment about the alternative being tested (two-sided, greater, less) and what H0 was.

```r
alpha <- 0.05
v <- t.test(x, mu = 175, alternative = "greater", conf.level = 1 - alpha)
print(v)
cat(sprintf("p = %.4f, alpha = %.2f -> %s\n",
            v$p.value, alpha,
            if (v$p.value < alpha) "reject H0 (evidence for alternative)"
            else                   "fail to reject H0"))
```

## 1-sample t-test (mean vs target)

```r
x <- c(174.2, 175.1, 176.0, 173.8, 175.5, 174.9, 176.3, 175.0, 174.4, 175.8)
alpha <- 0.05

v <- t.test(x, mu = 175, alternative = "two.sided", conf.level = 1 - alpha)
v
```

## 2-sample t-test

```r
d <- read.csv("two_groups.csv", header = TRUE)   # columns A, B

# Welch's t-test (default) allows unequal variances — use this unless you know variances are equal.
v <- t.test(d$A, d$B, alternative = "two.sided", conf.level = 0.95)

# For paired measurements (same unit before/after)
v <- t.test(d$After, d$Before, paired = TRUE)
```

## 1-sample proportion test

"Is the defect rate different from 20%?"

```r
N      <- 142          # trials
x_succ <- 38           # "successes" (here, defectives)
p0     <- 0.20         # null probability

v <- prop.test(x_succ, N, p = p0, alternative = "greater",
               conf.level = 0.95, correct = TRUE)
v
```

For small n, use `binom.test(x_succ, N, p = p0)` (exact binomial) instead.

## Wilcoxon (non-parametric)

When the data is skewed or ordinal, the t-test's normality assumption fails. Wilcoxon tests the median instead of the mean:

```r
# One-sample against a target median
br <- c(4, 4, 4, 5, 5, 3, 3, 6, 7, 2, 1)
v  <- wilcox.test(br, mu = 3, alternative = "greater",
                  exact = TRUE, conf.int = TRUE, conf.level = 0.95)

# Two independent samples
v <- wilcox.test(group_a, group_b)
```

## Chi-square goodness-of-fit

"Does the observed frequency distribution match what we'd expect under a theoretical model?"

```r
# Observed counts in 6 bins
observed <- c(5, 9, 14, 12, 7, 3)

# Expected proportions (must sum to 1). If testing against a normal distribution,
# compute them from pnorm() over the bin edges first.
expected_p <- c(0.05, 0.20, 0.30, 0.25, 0.15, 0.05)

v <- chisq.test(observed, p = expected_p)
v
```

To test a continuous sample against normality by binning, compute expected bin probabilities from the fitted normal:

```r
x     <- c(...)
bins  <- seq(10, 75, by = 5)
obs   <- hist(x, breaks = bins, plot = FALSE)$counts

mu    <- mean(x); s <- sd(x)
cdf   <- pnorm(bins, mu, s)
p_bin <- diff(cdf)
p_bin <- p_bin / sum(p_bin)                    # renormalize (edges clipped)

v <- chisq.test(obs, p = p_bin)
# Adjust df manually if you estimated mu and s from the same data:
# test_stat <- v$statistic
# df <- length(obs) - 1 - 2
# p  <- pchisq(test_stat, df, lower.tail = FALSE)
```

A dedicated normality test (`shapiro.test(x)` for n ≤ 5000) is usually easier when that's the actual question.

## Test of equal variances

Before a 2-sample t-test you might want to decide between Welch's and pooled:

```r
var.test(a, b)                  # F test for equal variances
```

Welch's is robust to unequal variances and is R's default — reach for pooled (`var.equal = TRUE`) only when variance equality is known or assumed.

## One- vs two-sided

Default to `"two.sided"` unless the user's question is directional ("is it *greater than*…", "did it *increase*…"). One-sided tests double the evidence against H0 in the specified direction but provide none in the other — don't pick one-sided just to get a lower p-value.
