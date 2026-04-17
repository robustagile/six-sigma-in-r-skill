# Charts and visualizations

Six Sigma uses a standard set of charts for exploratory and presentation work. Base R handles all of them; ggplot2 variants are included for when the user asks.

## Pareto chart

"Which few causes account for most of the problem?" — the 80/20 chart. Bars sorted descending, cumulative percent line overlaid, 80% reference line.

```r
d <- read.csv("defects.csv", header = TRUE)          # columns: Type, Count
d <- d[order(-d$Count), ]
d$Percent     <- d$Count / sum(d$Count) * 100
d$CumPercent  <- cumsum(d$Percent)

op <- par(mar = c(10, 4, 2, 4) + 0.2)                # wider bottom + right
bp <- barplot(d$Percent, names.arg = d$Type, las = 2,
              ylim = c(0, 100), ylab = "Percent", xlab = "")
lines(bp, d$CumPercent, type = "b", pch = 19)        # cumulative line
abline(h = 80, lty = 2)                              # 80 % cutoff
par(op)
```

`bp` (returned by `barplot`) holds the bar midpoints — use it as the x-coordinate for `lines()` so the cumulative dots sit over each bar.

## Run chart

Time-ordered line plot with a reference median. Used in the "Measure" phase to visualize process behavior over time before formal control limits.

```r
d <- read.csv("returns.csv", header = TRUE)          # Month, NumberOfReturns, NumberOfSales
d$ReturnsRate <- d$NumberOfReturns / d$NumberOfSales * 100

op <- par(mar = c(8, 4, 3, 2))
plot(d$ReturnsRate, type = "b", xaxt = "n",
     ylim = c(0, max(d$ReturnsRate) * 1.1),
     xlab = "", ylab = "Returns per 100 sales",
     main = "Run Chart")
axis(1, at = seq_along(d$ReturnsRate), labels = d$Month, las = 2)
abline(h = median(d$ReturnsRate), lty = 2)
par(op)
```

## Histogram

```r
hist(x, breaks = 15, col = "gray80",
     main = "Histogram", xlab = "Value")
```

`breaks` accepts an integer (suggested number of bins), a vector of explicit break points, or a rule name like `"Sturges"` (default), `"Scott"`, `"FD"`. For Six Sigma capability work, overlay the spec limits and a normal curve:

```r
hist(x, breaks = 15, freq = FALSE, col = "gray85",
     main = "Histogram with normal overlay", xlab = "Value")
curve(dnorm(x, mean = mean(x), sd = sd(x)),
      col = "black", lwd = 2, add = TRUE)
abline(v = c(LSL, USL), lty = 2)   # spec limits, if any
```

## Box plot

Compares distribution across groups. Great for before/after or group comparisons in the "Analyze" phase.

```r
d <- read.csv("run_times.csv", header = TRUE)        # Children, Adults
boxplot(d$Children, d$Adults,
        names = c("Children", "Adults"),
        ylab = "1-mile run time (min)",
        main = "Run time by group")
grid(nx = NA, ny = NULL)
```

For one long data column + a grouping factor:

```r
boxplot(value ~ group, data = d, xlab = "Group", ylab = "Value")
```

## Scatter plot with fitted line

Shows relationship between two continuous variables — the setup for a correlation / regression discussion.

```r
d   <- read.csv("parts.csv", header = TRUE)          # PartsPerHour, Defects
fit <- lm(Defects ~ PartsPerHour, data = d)

plot(d$PartsPerHour, d$Defects, pch = 19,
     xlab = "Parts per hour", ylab = "Defects",
     main = "Scatter with linear fit")
abline(fit)
```

## Bar chart (vertical / horizontal)

```r
d <- read.csv("calls.csv", header = TRUE)            # Hour, Calls

barplot(d$Calls, names.arg = d$Hour, las = 2,
        ylab = "Calls", main = "Calls by hour")

barplot(d$Calls, names.arg = d$Hour, horiz = TRUE, las = 1,
        xlab = "Calls", main = "Calls by hour (horizontal)")
```

## Stacked bar chart

`barplot` takes a matrix with rows = stacks, columns = bars. Each bar sums its column.

```r
d <- read.csv("team_calls.csv", header = TRUE)       # Hour, TeamA, TeamB
m <- t(as.matrix(d[, c("TeamA", "TeamB")]))
colnames(m) <- d$Hour

barplot(m, col = c("gray30", "gray70"), las = 2,
        legend.text = c("Team A", "Team B"),
        args.legend = list(x = "topright", bty = "n"),
        ylab = "Calls", main = "Calls by hour and team")
```

Use `beside = TRUE` for grouped (side-by-side) bars instead of stacked.

## Pie chart

Base R supports it, but pie charts are poor at showing differences — prefer a bar chart unless the user specifically asks.

```r
d <- read.csv("steps.csv", header = TRUE)            # Step, Minutes
pie(d$Minutes, labels = d$Step, main = "Time per step")
```

## ggplot2 variants (when requested)

```r
library(ggplot2)

# Pareto (bars + cumulative line)
d <- d[order(-d$Count), ]
d$Type       <- factor(d$Type, levels = d$Type)      # preserve sort
d$Percent    <- d$Count / sum(d$Count) * 100
d$CumPercent <- cumsum(d$Percent)
scale <- max(d$CumPercent) / max(d$Percent)
ggplot(d, aes(x = Type)) +
  geom_col(aes(y = Percent)) +
  geom_line(aes(y = CumPercent / scale, group = 1)) +
  geom_point(aes(y = CumPercent / scale)) +
  geom_hline(yintercept = 80 / scale, linetype = "dashed") +
  scale_y_continuous("Percent",
                     sec.axis = sec_axis(~ . * scale, name = "Cumulative %")) +
  theme(axis.text.x = element_text(angle = 90, hjust = 1))

# Scatter with fit
ggplot(d, aes(PartsPerHour, Defects)) +
  geom_point(size = 3) +
  geom_smooth(method = "lm", se = FALSE, color = "black")

# Histogram
ggplot(data.frame(x = x), aes(x)) +
  geom_histogram(bins = 15, fill = "gray70", color = "black")

# Box plot (long format)
ggplot(d_long, aes(group, value)) + geom_boxplot()
```

## File vs inline output

Wrap with `png()` / `dev.off()` only if the user asked for a file:

```r
png("pareto.png", width = 800, height = 600)
# ...plot code...
dev.off()
```

For inline (RStudio / notebook), leave the plot code bare.
