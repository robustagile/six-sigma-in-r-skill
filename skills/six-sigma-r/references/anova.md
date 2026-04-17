# ANOVA

Two different things get called "ANOVA" — they answer different questions and use different R calls. Don't conflate them.

## One-way ANOVA: compare more than two group means

Use when the user has a continuous response measured under three or more factor levels (machines, operators, suppliers, shifts, …) and wants to know whether *any* of the group means differ. T-tests pairwise would inflate the Type I error rate; one-way ANOVA gives a single test of "all means equal".

```r
d <- read.csv("yields.csv", header = TRUE)   # columns: Yield, Machine
d$Machine <- factor(d$Machine)               # factor, not character

fit <- aov(Yield ~ Machine, data = d)
summary(fit)                                 # F, df, p-value
```

Interpretation:
- p < α → at least one group mean differs. Follow up with a post-hoc test to see *which*.
- p ≥ α → no evidence of a difference in means.

Post-hoc (which specific pairs differ):

```r
TukeyHSD(fit)                                # all pairwise, adjusted
plot(TukeyHSD(fit))                          # CI plot of differences
```

Check the assumptions (normal residuals, equal variances across groups):

```r
par(mfrow = c(1, 2))
plot(fit, which = 1)                         # residuals vs fitted — look for equal spread
plot(fit, which = 2)                         # Q-Q — look for straight line
par(mfrow = c(1, 1))

# Formal equal-variance test
bartlett.test(Yield ~ Machine, data = d)     # normal-based
# Or the more robust Levene test (in 'car' package)
```

If variances are clearly unequal, switch to Welch's one-way:

```r
oneway.test(Yield ~ Machine, data = d, var.equal = FALSE)
```

If residuals are strongly non-normal, use the non-parametric Kruskal-Wallis:

```r
kruskal.test(Yield ~ Machine, data = d)
```

## Two-way ANOVA

When there are two factors (say machine *and* operator), and you also want to know whether they interact:

```r
fit <- aov(Yield ~ Machine * Operator, data = d)   # '*' = main effects + interaction
summary(fit)
```

`+` instead of `*` drops the interaction term. If the interaction is significant, interpret main effects carefully — the interaction plot often makes the story clearer than the F-table:

```r
interaction.plot(d$Machine, d$Operator, d$Yield)
```

## Regression ANOVA (F-test for a linear model)

Different use: you already fit a regression with `lm()` and want the overall F-test (and term-by-term sums of squares).

```r
fit <- lm(y ~ x, data = d)
anova(fit)                                   # sums of squares, F, p for each term
summary(fit)                                 # coefficient table + overall F at bottom
```

The F-test reported here asks "does the model explain significantly more variation than the intercept-only model?". `anova(fit1, fit2)` compares two nested models and tests whether the extra terms in the larger model add explanatory power.

## Quick guide to the right call

| User's question                                           | Call                         |
| --------------------------------------------------------- | ---------------------------- |
| "Are these 3+ group means different?"                     | `aov(y ~ group)`             |
| "Which specific groups differ?"                           | `TukeyHSD(aov(...))`         |
| "Two factors, do they interact?"                          | `aov(y ~ a * b)`             |
| "Non-normal response, compare groups"                     | `kruskal.test(y ~ group)`    |
| "Is the slope in my regression significant?"              | `anova(lm(...))` or `summary(lm(...))` |
| "Does adding predictor X2 improve the model?"             | `anova(lm(y ~ x1), lm(y ~ x1 + x2))` |
