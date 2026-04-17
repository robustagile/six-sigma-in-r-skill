# Control charts (SPC)

Statistical Process Control charts detect whether a process is "in statistical control" — stable and predictable. A point outside ±3σ control limits (or a non-random pattern inside them) signals an assignable-cause variation to investigate.

## Choosing the right chart

| Data type                   | Subgroup size n   | Chart          |
| --------------------------- | ----------------- | -------------- |
| Continuous measurements     | n = 1             | I-MR           |
| Continuous measurements     | 2 ≤ n ≤ 8 (~10)   | X-bar R        |
| Continuous measurements     | n > 10            | X-bar S        |
| Defectives (yes/no), fixed n   | any               | NP             |
| Defectives (yes/no), varying n | any               | P              |
| Defects (count), fixed opportunity   | any         | C              |
| Defects (count), varying opportunity | any         | U              |

"Defectives" = units classified as pass/fail. "Defects" = count of flaws per unit (a unit can have multiple defects).

## Control chart constants

These come from the standard SPC constant tables. Keep this table at the top of any X-bar R / X-bar S / I-MR script:

```r
# Subgroup-size-dependent constants (n = 2..10)
#   A2: X-bar R chart control limit factor for X-bar
#   D3, D4: R chart lower/upper factors
#   d2: unbiasing constant for sigma estimate sigma = R-bar / d2
#   A3, B3, B4, c4: X-bar S chart factors
spc_const <- data.frame(
  n  = 2:10,
  A2 = c(1.880, 1.023, 0.729, 0.577, 0.483, 0.419, 0.373, 0.337, 0.308),
  D3 = c(0,     0,     0,     0,     0,     0.076, 0.136, 0.184, 0.223),
  D4 = c(3.267, 2.574, 2.282, 2.114, 2.004, 1.924, 1.864, 1.816, 1.777),
  d2 = c(1.128, 1.693, 2.059, 2.326, 2.534, 2.704, 2.847, 2.970, 3.078),
  A3 = c(2.659, 1.954, 1.628, 1.427, 1.287, 1.182, 1.099, 1.032, 0.975),
  B3 = c(0,     0,     0,     0,     0.030, 0.118, 0.185, 0.239, 0.284),
  B4 = c(3.267, 2.568, 2.266, 2.089, 1.970, 1.882, 1.815, 1.761, 1.716),
  c4 = c(0.7979, 0.8862, 0.9213, 0.9400, 0.9515, 0.9594, 0.9650, 0.9693, 0.9727)
)
# E2 = 2.66 for the I chart (used with MR-bar/d2 sigma estimate, n=2)
```

For I-MR, the moving-range subgroup is 2, so `d2 = 1.128`, and the individual-chart limits use `E2 = 2.66`.

## X-bar R chart

```r
# Input: matrix or data frame with one row per subgroup, n columns per sample.
# Rows = time-ordered subgroups.
d <- read.csv("subgroups.csv", header = TRUE)
subgroups <- as.matrix(d[, -1])            # drop time/id column if present
n <- ncol(subgroups)
stopifnot(n >= 2 && n <= 10)

k   <- spc_const[spc_const$n == n, ]
xb  <- rowMeans(subgroups)
rng <- apply(subgroups, 1, function(v) max(v) - min(v))

xbb  <- mean(xb)                           # grand mean
rbar <- mean(rng)                          # average range

x_ucl <- xbb + k$A2 * rbar
x_lcl <- xbb - k$A2 * rbar
r_ucl <- k$D4 * rbar
r_lcl <- k$D3 * rbar                       # 0 for n <= 6

sigma_hat <- rbar / k$d2                   # within-subgroup sigma estimate

plot_xbar_r <- function(stat, center, ucl, lcl, ylab, main) {
  plot(stat, type = "b", pch = 19, xlab = "Subgroup", ylab = ylab,
       ylim = range(c(stat, ucl, lcl)) + c(-0.05, 0.05) * diff(range(c(stat, ucl, lcl))),
       main = main)
  abline(h = center, lty = 1)
  abline(h = c(ucl, lcl), lty = 2)
  ooc <- which(stat > ucl | stat < lcl)
  if (length(ooc)) points(ooc, stat[ooc], pch = 19, col = "red")
}

par(mfrow = c(2, 1))
plot_xbar_r(xb,  xbb,  x_ucl, x_lcl, "Subgroup mean",  "X-bar chart")
plot_xbar_r(rng, rbar, r_ucl, r_lcl, "Subgroup range", "R chart")
par(mfrow = c(1, 1))
```

Read the R chart first — if range is out of control, the X-bar limits are unreliable.

## X-bar S chart

Same structure, swap range for subgroup sd and use A3, B3, B4:

```r
s   <- apply(subgroups, 1, sd)
sbar <- mean(s)

x_ucl <- xbb + k$A3 * sbar
x_lcl <- xbb - k$A3 * sbar
s_ucl <- k$B4 * sbar
s_lcl <- k$B3 * sbar

sigma_hat <- sbar / k$c4
```

## I-MR chart (individuals + moving range)

For continuous data with n = 1 per time point.

```r
x <- read.csv("measurements.csv", header = TRUE)$Value

mr <- c(NA, abs(diff(x)))                  # moving range, window = 2
mr_bar <- mean(mr, na.rm = TRUE)
d2_mr  <- 1.128                            # n = 2
sigma_hat <- mr_bar / d2_mr

x_center <- mean(x)
x_ucl    <- x_center + 3 * sigma_hat       # equivalently + 2.66 * mr_bar
x_lcl    <- x_center - 3 * sigma_hat

mr_ucl   <- 3.267 * mr_bar                 # D4 at n=2
mr_lcl   <- 0                              # D3 at n=2

par(mfrow = c(2, 1))
plot_xbar_r(x,  x_center, x_ucl, x_lcl, "Individual value", "I chart")
plot_xbar_r(mr, mr_bar,   mr_ucl, mr_lcl, "Moving range",   "MR chart")
par(mfrow = c(1, 1))
```

Use `sigma_hat = mr_bar / d2_mr`, not `sd(x)`. `sd(x)` mixes within- and between-subgroup variation; the whole point of the MR approach is to estimate *within*-subgroup sigma for a process that only has n = 1 per time slice.

## P chart (proportion defective, varying n)

```r
d <- read.csv("pchart.csv", header = TRUE)     # columns: Samples, Defects
p     <- d$Defects / d$Samples
p_bar <- sum(d$Defects) / sum(d$Samples)

# Per-point limits because sample size varies
se  <- sqrt(p_bar * (1 - p_bar) / d$Samples)
ucl <- pmin(1, p_bar + 3 * se)
lcl <- pmax(0, p_bar - 3 * se)                 # clamp at 0

plot(p, type = "b", pch = 19, ylim = c(0, max(ucl) * 1.05),
     xlab = "Subgroup", ylab = "Proportion defective", main = "P chart")
abline(h = p_bar)
lines(ucl, lty = 2); lines(lcl, lty = 2)
ooc <- which(p > ucl | p < lcl)
if (length(ooc)) points(ooc, p[ooc], pch = 19, col = "red")
```

When all subgroups have the same n, `ucl` / `lcl` collapse to single values and you can use `abline(h = ...)` instead of `lines()`.

## NP chart (count defective, fixed n)

```r
np    <- d$Defects
n     <- d$Samples[1]                          # constant
p_bar <- sum(np) / (length(np) * n)

center <- n * p_bar
se     <- sqrt(n * p_bar * (1 - p_bar))
ucl    <- center + 3 * se
lcl    <- max(0, center - 3 * se)
```

## C chart (count of defects, fixed opportunity)

```r
d      <- read.csv("cchart.csv", header = TRUE) # column: Defects
c_bar  <- mean(d$Defects)
ucl    <- c_bar + 3 * sqrt(c_bar)
lcl    <- max(0, c_bar - 3 * sqrt(c_bar))
```

## U chart (defects per unit, varying opportunity)

```r
d     <- read.csv("uchart.csv", header = TRUE)  # columns: Errors, Calls
u     <- d$Errors / d$Calls
u_bar <- sum(d$Errors) / sum(d$Calls)

se  <- sqrt(u_bar / d$Calls)
ucl <- u_bar + 3 * se
lcl <- pmax(0, u_bar - 3 * se)
```

## Western Electric rules (optional)

A point outside ±3σ is the most common signal, but mature SPC also flags:

1. Any single point beyond 3σ from center.
2. 2 of 3 consecutive points beyond 2σ (same side).
3. 4 of 5 consecutive points beyond 1σ (same side).
4. 8 consecutive points on one side of the center line.

Only add these rules when the user explicitly asks for "Nelson rules" / "Western Electric" / "runs rules" — otherwise the single-point-beyond-limits rule is enough for typical requests.
