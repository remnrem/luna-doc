# R visualization code for the CHEP vignette

These examples use the **`luna` 1.8.0** R package and the output databases
created in the [CHEP vignette](chep.md). Run the setup and raw-data
sections first; they keep the original PSD for later comparisons.

## Setup

```r
library(luna)
p <- lp$proj()

# Plot an epoch-by-channel table with ordinary R graphics.
plot_channels <- function(data, metric, colors = grDevices::hcl.colors(100, "Viridis"),
                          log_scale = FALSE, title = metric) {
  epochs <- seq(min(data$E), max(data$E))
  channels <- sort(unique(as.character(data$CH)))
  values <- data[[metric]]
  if (log_scale) values <- log(values)
  z <- tapply(values, list(factor(data$E, levels = epochs),
                          factor(data$CH, levels = channels)), mean, na.rm = TRUE)
  image(epochs, seq_along(channels), z, col = colors,
        xlab = "Epoch", ylab = "Channel", yaxt = "n", main = title)
  axis(2, at = seq_along(channels), labels = channels, las = 2, cex.axis = 0.6)
}

# Compare saved PSD data frames; importing another database does not alter them.
compare_psd <- function(raw, cleaned, channels = unique(raw$CH)) {
  limits <- range(10 * log10(c(raw$PSD, cleaned$PSD)), finite = TRUE)
  plot(range(raw$F), limits, type = "n",
       xlab = "Frequency (Hz)", ylab = "Power (dB)")
  for (ch in channels) {
    before <- raw[raw$CH == ch, ]
    after <- cleaned[cleaned$CH == ch, ]
    lines(before$F, 10 * log10(before$PSD), col = "maroon")
    lines(after$F, 10 * log10(after$PSD), col = "blue")
  }
  legend("topright", c("Before", "After"), col = c("maroon", "blue"), lty = 1, bty = "n")
}
```

The databases in this vignette contain one individual. Select an `ID`
first if adapting the plotting helpers to cohort output.

## Raw data PSD

```r
p$import_db("ss_psd1.db")
p$strata()
psd_raw <- p$table("PSD", c("CH", "F"))
hjorth_raw <- p$table("SIGSTATS", c("CH", "E"))

channels <- unique(psd_raw$CH)
palette <- rainbow(length(channels))
plot(range(psd_raw$F), range(10 * log10(psd_raw$PSD), finite = TRUE),
     type = "n", xlab = "Frequency (Hz)", ylab = "Power (dB)")
for (n in seq_along(channels)) {
  d <- psd_raw[psd_raw$CH == channels[n], ]
  lines(d$F, 10 * log10(d$PSD), col = palette[n])
}
```

For channel coordinates in your own plots, `lp$default_xy()` returns
`CH`, `X`, and `Y` for the standard montage. To show one value per
channel, use [`lp$topo_heat()`](../ext/R/viz.md#channel-maps).

## Raw data Hjorth parameters

```r
par(mfrow = c(1, 3), mar = c(4, 4, 3, 1))
for (metric in c("H1", "H2", "H3")) {
  plot_channels(hjorth_raw, metric, log_scale = metric == "H1")
}
par(mfrow = c(1, 1))
```

## Bad channel detection

```r
p$import_db("chep_bchs.db")
p$strata()
chep <- p$table("CHEP", c("CH", "E"))
plot_channels(chep, "CHEP", colors = c("lightgray", "brown"), title = "CHEP mask")
bad_channels <- unique(chep$CH[chep$CHEP > 0])
```

## Interpolating bad channels

```r
p$import_db("ss_psd2.db")
chep <- p$table("CHEP", c("CH", "E"))
psd_clean <- p$table("PSD", c("CH", "F"))
bad_channels <- unique(chep$CH[chep$CHEP > 0])
par(mfrow = c(1, 2), mar = c(4, 4, 3, 1))
plot_channels(chep, "CHEP", colors = c("lightgray", "brown"), title = "CHEP mask")
compare_psd(psd_raw, psd_clean, channels = bad_channels)
par(mfrow = c(1, 1))
```

## Epoch-level interpolation

```r
p$import_db("ss_psd3.db")
chep <- p$table("CHEP", c("CH", "E"))
psd_clean <- p$table("PSD", c("CH", "F"))
par(mfrow = c(1, 2), mar = c(4, 4, 3, 1))
plot_channels(chep, "CHEP", colors = c("lightgray", "brown"), title = "CHEP mask")
compare_psd(psd_raw, psd_clean)
par(mfrow = c(1, 1))
```

## Masking remaining outlier epochs

```r
p$import_db("ss_psd4.db")
chep <- p$table("CHEP", c("CH", "E"))
psd_clean <- p$table("PSD", c("CH", "F"))
par(mfrow = c(1, 2), mar = c(4, 4, 3, 1))
plot_channels(chep, "CHEP", colors = c("lightgray", "brown"), title = "CHEP mask")
compare_psd(psd_raw, psd_clean)
par(mfrow = c(1, 1))
```
