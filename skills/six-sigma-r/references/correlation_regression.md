# Correlation and regression

Use when the user wants to understand the relationship between two (or more) continuous variables — the typical "Analyze" phase step after a scatter plot suggests a relationship.

## Correlation

```r
d <- read.csv("data.csv", header = TRUE)    # Time, TemperatureA, TemperatureB, ...

cor(d$Time, d$TemperatureA)                 # Pearson (default, linear)
cor(d$Time, d$TemperatureA, method = "spearman")  # rank-based, monotone relationships

# Whole-matrix correlation
cor(d[, sapply(d, is.numeric)])

# With p-values
cor.test(d$Time, d$TemperatureA)
```

Rough interpretation guide (absolute r):
- 0.0–0.3 — weak / negligible
- 0.3–0.7 — moderate
- 0.7–1.0 — strong

Always pair a correlation with a scatter plot — `cor()` cannot tell a linear relationship from a curved one with the same r.

## Simple linear regression

```r
fit <- lm(y ~ x, data = d)

summary(fit)                           # coefficients, R^2, F test
coef(fit)                              # intercept and slope
confint(fit, level = 0.95)             # coefficient confidence intervals
anova(fit)                             # regression F-test (is the slope non-zero?)
fitted(fit); residuals(fit)
```

Reported statistics (from `summary(fit)`):

```r
r   <- cor(d$x, d$y)
rsq <- summary(fit)$r.squared
adj <- summary(fit)$adj.r.squared      # penalizes extra predictors
# residual standard error (a.k.a. S) — average error in y-units
s   <- summary(fit)$sigma
```

## Plot the fit with prediction line

```r
plot(d$x, d$y, pch = 19,
     xlab = "x", ylab = "y", main = "Linear fit")
abline(fit, lwd = 2)

# Optional: confidence and prediction bands
newx  <- data.frame(x = seq(min(d$x), max(d$x), length.out = 100))
ci    <- predict(fit, newx, interval = "confidence")
pi    <- predict(fit, newx, interval = "prediction")
lines(newx$x, ci[, "lwr"], lty = 2)
lines(newx$x, ci[, "upr"], lty = 2)
lines(newx$x, pi[, "lwr"], lty = 3)
lines(newx$x, pi[, "upr"], lty = 3)
```

## Residual diagnostics

Always before trusting a regression, especially for capability / prediction use:

```r
par(mfrow = c(2, 2))
plot(fit)                              # residuals vs fitted, Q-Q, scale-location, leverage
par(mfrow = c(1, 1))
```

Red flags:
- Residuals vs fitted shows a curve → relationship isn't linear, try a transformation or polynomial term.
- Q-Q plot bends off the line at the tails → non-normal errors, inference is unreliable.
- Scale-location plot has a funnel → heteroscedasticity, consider weighted least squares or a log transform.

## Multiple regression

```r
fit <- lm(y ~ x1 + x2 + x3, data = d)
summary(fit)
# With interaction
fit_int <- lm(y ~ x1 * x2, data = d)
# Stepwise variable selection (use judiciously)
step(fit, direction = "both")
```

## When to report what

- "How correlated are X and Y?" → `cor()` plus scatter plot.
- "Can I predict Y from X?" → `lm()` plus summary (R², slope, p-value of slope) and a residual diagnostic.
- "Is the slope meaningfully different from zero?" → check the p-value on the slope row in `summary(fit)$coefficients`, or use `anova(fit)`.
- "How much of the variation does X explain?" → R² (simple regression) or adjusted R² (multiple regression).
