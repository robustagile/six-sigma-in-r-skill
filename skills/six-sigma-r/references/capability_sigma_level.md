# Process capability and sigma level

Answers "can this process meet its specification limits?". Capability indices compare the *voice of the process* (natural variation) to the *voice of the customer* (spec limits).

## Cp, Cpk (short-term, within-subgroup sigma)

Use when you have rational subgroups and want to estimate *process potential* — what the process is capable of when only within-subgroup (common-cause) variation is present.

```r
# Input: vector of measurements x plus spec limits.
# Optional sigma_within from an X-bar R chart (sigma = R-bar / d2). If absent,
# fall back to sd(x), but make the approximation explicit to the user.
cp_cpk <- function(x, lsl, usl, sigma_within = NULL) {
  mu <- mean(x)
  s  <- if (is.null(sigma_within)) sd(x) else sigma_within
  cp  <- (usl - lsl) / (6 * s)
  cpu <- (usl - mu)  / (3 * s)
  cpl <- (mu  - lsl) / (3 * s)
  cpk <- min(cpu, cpl)
  list(mu = mu, sigma = s, Cp = cp, CPU = cpu, CPL = cpl, Cpk = cpk)
}
```

Only `usl` or only `lsl`? Drop the missing side: `Cpk = CPU` when there's no `lsl`, and vice versa. Set the undefined index to `NA`, don't make one up.

## Pp, Ppk (long-term, overall sigma)

Use the sample standard deviation of *all* observations (no subgrouping). Captures how the process *actually* performs over time, including between-subgroup drift.

```r
pp_ppk <- function(x, lsl, usl) {
  mu <- mean(x); s <- sd(x)
  pp  <- (usl - lsl) / (6 * s)
  ppu <- (usl - mu)  / (3 * s)
  ppl <- (mu  - lsl) / (3 * s)
  ppk <- min(ppu, ppl)
  list(mu = mu, sigma = s, Pp = pp, PPU = ppu, PPL = ppl, Ppk = ppk)
}
```

Rule of thumb for what to say in the output:
- Cpk or Ppk < 1.00 → process is not capable.
- 1.00 ≤ Cpk < 1.33 → marginal.
- Cpk ≥ 1.33 → typically acceptable (4σ).
- Cpk ≥ 1.67 → 5σ.
- Cpk ≥ 2.00 → 6σ (the "Six Sigma" target).

A large gap between Cpk and Ppk means the process is shifting or drifting — address stability (control charts) before re-scoring capability.

## DPMO and sigma level

DPMO = Defects Per Million Opportunities. Sigma level converts defect rate into a "how many standard deviations fit" number, using the 1.5σ shift by convention for long-term → short-term.

```r
dpmo <- function(defects, units, opportunities_per_unit) {
  defects / (units * opportunities_per_unit) * 1e6
}

# Long-term (process) sigma level from observed defect rate.
# p = fraction defective (0..1).
sigma_level_lt <- function(p) qnorm(1 - p)

# Short-term sigma level includes the 1.5-sigma shift convention.
sigma_level_st <- function(p) qnorm(1 - p) + 1.5

# Inverse: from DPMO to short-term sigma level.
dpmo_to_sigma_st <- function(dpmo_val) qnorm(1 - dpmo_val / 1e6) + 1.5
```

Reference points (short-term sigma, the conventional "Six Sigma" scale):
- 3.4 DPMO ≈ 6σ
- 233 DPMO ≈ 5σ
- 6 210 DPMO ≈ 4σ
- 66 810 DPMO ≈ 3σ

Always print both the raw defect rate and the sigma level so the user can sanity-check the conversion, and state which convention (with/without the 1.5 shift) was used.

## Z-bench / Z-score from spec limits

For a normally distributed process, the probability of a defect is the area beyond each spec limit:

```r
z_bench <- function(x, lsl, usl) {
  mu <- mean(x); s <- sd(x)
  p_below <- if (is.finite(lsl)) pnorm(lsl, mu, s) else 0
  p_above <- if (is.finite(usl)) 1 - pnorm(usl, mu, s) else 0
  p_def   <- p_below + p_above
  list(p_below = p_below, p_above = p_above,
       p_defective = p_def,
       dpmo = p_def * 1e6,
       Z_bench = qnorm(1 - p_def))        # long-term Z
}
```

## Normality check before capability

Cp/Cpk/Pp/Ppk all assume normality. For non-trivial requests, run a Shapiro-Wilk test and plot a histogram with a normal overlay so the user can judge:

```r
shapiro.test(x)                           # p > 0.05 suggests normal is plausible

hist(x, breaks = 15, freq = FALSE, col = "gray85",
     main = "Capability histogram", xlab = "Measurement")
curve(dnorm(x, mean = mean(x), sd = sd(x)),
      col = "black", lwd = 2, add = TRUE)
abline(v = c(lsl, usl), lty = 2)
```

If the data is clearly non-normal, Cpk is misleading — tell the user and suggest a Box-Cox transform or non-parametric capability (percentile method: `Ppk = min((usl - median) / (p99.865 - median),  (median - lsl) / (median - p0.135))`).
