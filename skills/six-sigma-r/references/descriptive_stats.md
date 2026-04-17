# Descriptive statistics

Use when the user asks for summary statistics of a numeric sample — "describe this data", "what's the standard deviation", "give me the five-number summary".

## Standard summary

```r
x <- c(67, 68, 73, 74, 81, 85, 88, 88, 90, 90, 90, 93, 94, 98, 99)

n          <- length(x)
mean_x     <- mean(x)
median_x   <- median(x)
sd_sample  <- sd(x)            # sample sd, divides by (n - 1)
var_sample <- var(x)
sd_pop     <- sqrt(mean((x - mean(x))^2))   # population sd, divides by n
var_pop    <- mean((x - mean(x))^2)
q          <- quantile(x, c(0.25, 0.5, 0.75))
iqr_x      <- IQR(x)
rng        <- range(x)

summary(x)    # min, 1st Q, median, mean, 3rd Q, max in one call
```

## Mode (not built in)

R has no base `mode()` for statistical mode (the `mode()` function returns the storage type). Define one:

```r
stat_mode <- function(v) {
  u <- unique(v)
  u[which.max(tabulate(match(v, u)))]
}
```

## Skewness and kurtosis

Not in base R. Use the `e1071` package:

```r
library(e1071)
skewness(x)   # 0 = symmetric, >0 right-tailed, <0 left-tailed
kurtosis(x)   # excess kurtosis: 0 = normal, >0 heavy-tailed
```

If the user can't install packages, compute directly:

```r
skew <- function(v) {
  n <- length(v); m <- mean(v); s <- sd(v)
  (sum((v - m)^3) / n) / s^3
}
kurt <- function(v) {          # excess kurtosis
  n <- length(v); m <- mean(v); s <- sd(v)
  (sum((v - m)^4) / n) / s^4 - 3
}
```

## Standard error of the mean

```r
se_mean <- sd(x) / sqrt(length(x))
```

Note: `sd(x) / length(x)` (no square root) is a common mistake — don't write it.

## Sample vs population distinction

Six Sigma studies usually work with *samples* drawn from a larger process, so prefer `sd()` / `var()` (which use `n - 1`). Only switch to the population formulas when the user explicitly has the full population or asks for "population standard deviation".
