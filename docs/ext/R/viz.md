# Plotting in R

The **`luna` 1.8.0** plotting methods use base R graphics. You can plot a
recording directly or pass existing results as R data frames.
See the [worked example](example.md) for preparing the data.

## Open a recording

After extracting the [tutorial data](../../download/data.md), start R
from the directory containing `tutorial`:

```r
library(luna)
p <- lp$proj()
p$sample_list("tutorial/s.lst", path = "tutorial")
i <- p$inst("nsrr02")
i$epoch("len=30")
```

Instance plotting methods run Luna commands and can replace the
instance's current output tables. Keep important results in a saved
`r <- i$proc(...)` before plotting.

## Hypnograms

For a recording with sleep-stage annotations:

```r
i$hypno()
```

To plot a stage vector or a saved `STAGE` table:

```r
stages <- i$stages()
lp$hypno(stages$STAGE, e = stages$E - 1, title = "nsrr02")
```

Supported labels include `W`, `N1`, `N2`, `N3`, `R`, `?`, and `L`.
The time axis assumes **30-second epochs**. `e` contains zero-based
30-second positions; when omitted, the stages are treated as consecutive.
For other epoch durations, draw your own time axis using the stage table's
time columns.

```r
lp$hypno(ss = c("W", "N1", "N2", "N3", "N2", "R", "W"),
         title = "Example stages")
```

For custom plots, `lp$stgcol(ss)` returns stage colors and `lp$stgn(ss)`
returns vertical positions. `lp$stgcol(NA)` returns a transparent color.

## Hypnodensity

`lp$hypno_density()` plots a data frame of stage probabilities. It needs
columns `PP_N1`, `PP_N2`, `PP_N3`, `PP_R`, and `PP_W`, with one row per
epoch. These can come from automated staging output; see
[POPS](../../ref/pops.md).

Here is a standalone illustration with made-up probabilities:

```r
probs <- data.frame(
  PP_N1 = c(0.10, 0.20, 0.05, 0.05),
  PP_N2 = c(0.10, 0.55, 0.80, 0.10),
  PP_N3 = c(0.00, 0.10, 0.10, 0.00),
  PP_R  = c(0.00, 0.05, 0.00, 0.80),
  PP_W  = c(0.80, 0.10, 0.05, 0.05)
)
lp$hypno_density(probs, title = "Example stage probabilities")
```

The R plot uses row positions as epochs on its x-axis; the compatibility
argument `e` does not change that axis.

## Power spectra

Calculate and plot spectra directly from an instance:

```r
i$psd("EEG", maxf = 30)
```

This runs `PSD spectrum dB` and draws a decibel spectrum. Pass a channel
vector for multiple traces, and use `minf`, `maxf`, `minp`, `maxp`,
`xlines`, and `ylines` to control the displayed range and reference lines.
With `var = "MTM"`, the method runs the multitaper estimator instead.

To plot a saved linear-power spectrum:

```r
r <- i$proc("PSD sig=EEG spectrum max=30")
spectrum <- r$table("PSD", "CH_F")
lp$psd(spectrum, "EEG", dB = TRUE, minf = 0.5, maxf = 30,
       xlines = c(4, 8, 12, 16))
```

`dB = TRUE` applies `10 * log10()` for display. If the analysis already
used `PSD ... dB`, leave `dB = FALSE` to avoid converting twice:

```r
r_db <- i$proc("PSD sig=EEG spectrum max=30 dB")
lp$psd(r_db$table("PSD", "CH_F"), "EEG")
```

The helper expects `CH`, `F`, and the power column named by `var`
(default `PSD`). Select one individual from cohort output before
plotting, for example with `lp$subset(df, ids = "nsrr02")`.

## Spectrograms

Calculate an epoch-by-frequency spectrum and plot it:

```r
i$spec("EEG", minf = 0.5, maxf = 30)
```

Or plot saved per-epoch output:

```r
r_tf <- i$proc("PSD sig=EEG epoch-spectrum dB min=0.5 max=30")
tf <- r_tf$table("PSD", "CH_E_F")
lp$spec(tf, ch = "EEG", minf = 0.5, maxf = 30)
```

The table needs `E`, `F`, `CH`, and the column named by `var` (default
`PSD`). `mine`/`maxe` restrict epoch numbers; `minf`/`maxf` restrict
frequencies. `w = 0.025` clips the lowest and highest 2.5% of displayed
values to limit color-scale outliers; use `w = 0` to disable this.
Select one individual before plotting cohort output.

## Channel maps

`lp$topo_heat()` draws one colored point per channel at its scalp
location. It does not interpolate a continuous scalp surface.
Channel names and values must be in matching order:

```r
chs <- c("C3", "C4", "CZ", "F3", "F4", "O1", "O2")
z <- c(-1.2, 0.8, 0.1, -0.5, 0.6, -0.2, 0.4)
lp$topo_heat(chs, z, lmts = c(-1.5, 1.5),
             colormap = "bwr", title = "Example channel values", lab = "Value")
```

These are illustrative values. For analysis results, take channel names
and values from the same rows of your output table.

| Argument | Use |
| --- | --- |
| `chs`, `z` | Channel labels and numeric values of equal length; matching is case-insensitive |
| `topo = NULL` | Use the standard 64-channel coordinates; otherwise supply a data frame with `CH`, `X`, `Y` |
| `lmts = NULL` | Color limits; if supplied, they must contain all values in `z` |
| `colormap = "bwr"` | Blue/white/red, `"turbo"`, an R `hcl.colors()` palette name, or a vector of colors |
| `ths = NULL`, `th = 0.05` | Optional per-channel values, e.g. p-values; values below `th` receive thicker outlines |
| `sz = 70` | Point-size control |
| `rimcolor = "black"` | Point-outline color |
| `title = ""`, `lab = "dB"` | Plot title and color-scale label |

Get the built-in coordinates for use in your own plots:

```r
xy <- lp$default_xy()
head(xy)
```

Unmatched channels are omitted; no matching channels produces an error.
Supply `topo` for a montage with other labels or locations.

## Saving plots

Plots appear on the current R graphics device. In RStudio, use the
Plots pane, or save a file by opening a device first:

```r
pdf("EEG-spectrum.pdf", width = 8, height = 4)
lp$psd(spectrum, "EEG", dB = TRUE)
dev.off()
```

The `xsize` and `ysize` arguments accepted by `lp$hypno()` and
`lp$hypno_density()` do not size R plots. Set dimensions on `pdf()`,
`png()`, or your other graphics device instead.

## Plot method signatures

```text
i$hypno()
i$psd(ch, var = "PSD", minf = NULL, maxf = 25,
      minp = NULL, maxp = NULL, xlines = NULL, ylines = NULL)
i$spec(ch, mine = NULL, maxe = NULL, minf = NULL, maxf = NULL, w = 0.025)

lp$hypno(ss, e = NULL, xsize = 20, ysize = 2, title = NULL)
lp$hypno_density(probs, e = NULL, xsize = 20, ysize = 2, title = NULL)
lp$psd(df, ch, var = "PSD", minf = NULL, maxf = NULL,
       minp = NULL, maxp = NULL, xlines = NULL, ylines = NULL, dB = FALSE)
lp$spec(df, ch = NULL, var = "PSD", mine = NULL, maxe = NULL,
        minf = NULL, maxf = NULL, w = 0.025)
lp$topo_heat(chs, z, ths = NULL, th = 0.05, topo = NULL, lmts = NULL,
             sz = 70, colormap = "bwr", title = "", rimcolor = "black", lab = "dB")
lp$default_xy()
lp$stgcol(ss)
lp$stgn(ss)
```

## Moving from old R plots

These old plotting functions are not exported by 1.8.0. Use the new
helpers or draw plots directly from R data frames:

| Old helper | Current approach |
| --- | --- |
| <span id="lheatmap"></span>`lheatmap()` | `lp$spec()` for epoch/frequency spectra, or R `image()` for other matrices |
| <span id="ltopoheat"></span>`ltopo.heat()` | `lp$topo_heat()` for channel-value maps |
| <span id="ltoporb"></span>`ltopo.rb()` | `lp$topo_heat(..., colormap = "bwr")` |
| <span id="ldefaultxy"></span>`ldefault.xy()` | `lp$default_xy()` |
| <span id="ltopoxy"></span>`ltopo.xy()` | Make channel-specific plots from the output data frame using base R or ggplot2 |
| <span id="ltopoheat2"></span><span id="ltopoconn"></span><span id="ltopodconn"></span><span id="ltopotopo"></span><span id="ldefaultcohxy"></span>Other `ltopo.*` connectivity/panel plots and `ldefault.coh.xy()` | No direct replacement in the new plotting helpers; use the returned data frames with your preferred R plotting tools |
