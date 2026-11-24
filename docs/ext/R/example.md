# R worked example

This example uses the **1.8.0** project/instance API to inspect tutorial
data, analyze N2 EEG, extract signals, and collect results across a
sample list. See [installation](../../download/lunar.md) if you have not
yet installed the R package `luna`.

## Open the tutorial data

Download and extract the [tutorial data](../../download/data.md), then
start R in the directory containing the `tutorial` folder. The sample
list is `tutorial/s.lst`, with EDFs and annotations under `tutorial/edfs/`.

```r
library(luna)
p <- lp$proj()
p$sample_list("tutorial/s.lst", path = "tutorial")
p$sample_list()
p$nobs()
p$validate()
```

Check that `Valid` is `TRUE` for the inputs before processing them.
The `path` argument tells Luna where to resolve relative paths in the
sample list.

Open the second recording by ID:

```r
i <- p$inst("nsrr02")
i$desc()
i$chs()
i$annots()
headers <- i$headers()
headers[, c("CH", "SR")]
```

Use the labels returned by `chs()` and `annots()` in commands. In this
dataset, the primary EEG channel is `EEG`, N2 sleep is labeled `N2`, and
the arousal annotation is `Arousal`. Annotation names are case-sensitive.

## Epochs, stages, and raw signals

Set 30-second epochs and inspect the staging:

```r
i$epoch("len=30")
epochs <- i$epochs()
stages <- i$stages()
head(epochs)
table(stages$STAGE)
```

Get the first five seconds of EEG as a data frame, with a time column:

```r
interval <- i$s2i(c(0, 5))
raw <- i$slice(interval, "EEG", time = TRUE)
head(raw)
plot(raw$T, raw$EEG, type = "l", xlab = "Seconds", ylab = "EEG")
```

`s2i()` converts start/stop seconds into the time-point intervals needed
by `slice()`. Use `e2i()` for epoch numbers. `slices()` returns a list
with one data frame per interval:

```r
by_epoch <- i$slices(i$e2i(1:3), "EEG", annots = "Arousal", time = TRUE)
head(by_epoch[[1]])
rms <- vapply(by_epoch, function(d) sqrt(mean(d$EEG^2)), numeric(1))
rms
```

The `Arousal` column marks samples inside arousal events with 1 and
other samples with 0. These operations read signals into R; they do not
change the recording.

## Restrict to N2 and filter

Save the current state so that you can return to it later:

```r
i$freeze("original")
```

Keep N2 epochs, remove other signals from the in-memory recording, and
apply a bandpass filter:

```r
i$eval(c(
  "MASK ifnot=N2",
  "RE",
  "SIGNALS keep=EEG",
  "FILTER sig=EEG bandpass=0.3,35 ripple=0.02 tw=0.5"
))
```

`RE` removes masked epochs from subsequent analyses. Commands change
the in-memory recording; they do not overwrite the source EDF.

## Check artifacts

Run artifact checks and keep their output:

```r
qc <- i$proc("ARTIFACTS sig=EEG mask & CHEP-MASK sig=EEG ep-th=3,3,3 & CHEP epochs")
qc$strata()
```

Inspect the tables listed by `qc$strata()` to see what was flagged.
These settings illustrate a workflow; choose artifact criteria for your
signals and study. Remove the flagged epochs before estimating power:

```r
i$eval("RE")
i$epochs()
```

If all epochs have been excluded, review the annotations and artifact
settings before continuing.

## Calculate and save a spectrum

```r
r <- i$proc("PSD sig=EEG spectrum max=30")
r$strata()
psd <- r$table("PSD", "CH_F")
head(psd)
```

The spectrum has columns including `ID`, `CH`, `F`, and `PSD`: individual,
channel, frequency in Hz, and power spectral density. Band-level results
are available separately:

```r
bands <- r$table("PSD", "B_CH")
head(bands)
lp$psd(psd, "EEG", dB = TRUE)
write.csv(psd, "nsrr02-N2-psd.csv", row.names = FALSE)
```

`dB = TRUE` converts this linear spectrum to decibels for plotting.
The [PSD reference](../../ref/power-spectra.md#psd) explains the analysis
options and outputs. The saved result `r` remains available after later
commands.

## Restore the recording

Return to the state saved before masking and filtering:

```r
i$thaw("original")
i$chs()
i$empty_freezer()
```

For a file-backed instance, `i$refresh()` instead reloads the original
EDF and annotations from disk.

## Process the whole sample list

`p$proc()` loads each recording in the sample list and runs the same
commands. It starts from each source EDF, so the edits to `i` above do
not carry over into this run:

```r
cohort <- p$proc("EPOCH & MASK ifnot=N2 & RE & PSD sig=EEG spectrum max=30")
cohort$strata()
cohort_psd <- cohort$table("PSD", "CH_F")
table(cohort_psd$ID)
write.csv(cohort_psd, "cohort-N2-psd.csv", row.names = FALSE)
```

For a larger workflow, put the Luna commands in a text file and pass
`lp$cmdfile("commands.txt")` to `p$proc()`.

See the [R reference](ref.md) for the full method list, saved results,
variables, and output-database import; see [plotting](viz.md) for
hypnograms, spectrograms, and scalp maps.
