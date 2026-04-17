# Probability distributions

Use when the user asks "what's the probability of k defects" / "what's the chance of observing X when the process averages Y".

## Binomial (pass/fail, fixed n trials)

P(exactly k defectives in n items when defect rate is p):

```r
dbinom(k, size = n, prob = p)
```

P(at most k): `pbinom(k, n, p)`.  P(at least k): `pbinom(k - 1, n, p, lower.tail = FALSE)`.

Example — probability of exactly 4 defects in a box of 20 when the defect rate is 5%:

```r
dbinom(4, size = 20, prob = 0.05)
```

Full distribution:

```r
n <- 20; p <- 0.05
k <- 0:n
probs <- dbinom(k, n, p)

barplot(probs, names.arg = k,
        xlab = "Number of defectives", ylab = "Probability",
        main = sprintf("Binomial(n=%d, p=%.2f)", n, p))
```

## Poisson (counts over a fixed interval)

P(exactly k events when the mean rate is λ):

```r
dpois(k, lambda = lambda)
```

Example — a call center averages 10 errors per hour; P(observing exactly 15 in an hour):

```r
dpois(15, lambda = 10)
```

Cumulative and tails:

```r
ppois(k, lambda)                        # P(X <= k)
ppois(k - 1, lambda, lower.tail = FALSE) # P(X >= k)
```

Full distribution plot:

```r
lambda <- 10
k <- 0:(lambda * 3)
barplot(dpois(k, lambda), names.arg = k,
        xlab = "Events", ylab = "Probability",
        main = sprintf("Poisson(lambda=%g)", lambda))
```

## Normal

```r
dnorm(x, mean = mu, sd = sigma)        # density
pnorm(x, mean = mu, sd = sigma)        # CDF
qnorm(p, mean = mu, sd = sigma)        # quantile
rnorm(n, mean = mu, sd = sigma)        # random sample
```

"Probability a measurement falls outside [LSL, USL] for a normal process":

```r
p_defect <- pnorm(LSL, mu, sigma) + (1 - pnorm(USL, mu, sigma))
```

## Choosing the right distribution

- Counting *defectives* (pass/fail) in a fixed sample of size n → **binomial**.
- Counting *defects* (flaws) in a unit or interval, with no natural n → **Poisson**.
- When n is large and p is small, binomial ≈ Poisson with λ = n·p. Either gives the same answer to three decimals.
- Continuous measurement with roughly symmetric distribution → **normal** (and check this — see `capability_sigma_level.md`).

## Manual formulas

Rarely needed, but if the user wants to avoid built-in functions:

```r
# Binomial pmf
choose(n, k) * p^k * (1 - p)^(n - k)

# Poisson pmf
exp(-lambda) * lambda^k / factorial(k)
```

Prefer the built-ins — they handle large-n edge cases via log-space arithmetic.
