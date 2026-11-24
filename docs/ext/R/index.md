# Luna in R

The R package **`luna` 1.8.0** brings Luna's sleep-signal analysis commands
into R through [lunapi](../../lunapi/index.md). It follows the Python
interface: create a project, open a recording, run commands, and retrieve
results as R data frames.

The package lives at [`remnrem/lunaR`](https://github.com/remnrem/lunaR)
and is loaded with `library(luna)`.

## Installation

In R 4.1 or later:

```r
install.packages("pak")
pak::pak("remnrem/lunaR")
library(luna)
p <- lp$proj()
luna_config()
```

For the pinned R release, replace the `pak::pak()` line with
`pak::pak("remnrem/lunaR@v1.8.0")`. See the
[version-pinning instructions](../../download/lunar.md#specifying-particular-versions)
to pin the Python engine too.

First use sets up Python and lunapi automatically. If you already have a
Python environment you want to use, select it before loading Luna.
See [installation](../../download/lunar.md) for those steps, help with
setup errors, and a pinned 1.8.0 route.

## A first analysis

Download and extract the [tutorial data](../../download/data.md). Run R
from the directory containing the extracted `tutorial` folder:

```r
library(luna)
p <- lp$proj()
p$sample_list("tutorial/s.lst", path = "tutorial")
i <- p$inst("nsrr02")
i$chs()
i$annots()
i$headers()
```

Here `p` represents the project and its sample list. `i` represents one
recording, selected by ID. You can also use `p$inst(2)` for the second
sample-list entry.

Run a spectral analysis on N2 sleep and retrieve the spectrum:

```r
r <- i$proc("EPOCH & MASK ifnot=N2 & RE & PSD sig=EEG spectrum max=30")
r$strata()
psd <- r$table("PSD", "CH_F")
head(psd)
```

`CH_F` means one row for each channel/frequency combination. The result
is an ordinary R data frame: use it with base R, dplyr, ggplot2, or your
usual analysis tools. For a spectrum plot:

```r
lp$psd(psd, "EEG", dB = TRUE)
```

Follow the [worked example](example.md) for filtering, artifact handling,
signal extraction, and processing all recordings in a sample list.
The full [/R/epeat tutorial](../../tut/tut4.md) follows the command-line
tutorials through annotations, masks, EDF writing, and spindle analysis.

## Finding the right method

| Task | R interface |
| --- | --- |
| Start a project | `p <- lp$proj()` |
| Read a sample list | `p$sample_list("tutorial/s.lst", path = "tutorial")` |
| Open a recording | `i <- p$inst(1)` or `i <- p$inst("nsrr02")` |
| Inspect signals and annotations | `i$chs()`, `i$headers()`, `i$annots()` |
| Run commands on that recording | `i$eval("PSD sig=EEG spectrum")` |
| Keep the results of a run | `r <- i$proc("PSD sig=EEG spectrum")` |
| Retrieve an output table | `r$table("PSD", "CH_F")` |
| Extract signals into R | `i$data("EEG", time = TRUE)` |
| Extract one data frame per epoch | `i$slices(i$e2i(1:3), "EEG")` |
| Run commands across the sample list | `r <- p$proc("HEADERS & PSD sig=EEG spectrum")` |
| Process recordings with worker processes | `r <- p$procn("PSD sig=EEG spectrum", workers = 4)` |
| Write large outputs directly to disk | `r <- p$procn("PSD sig=EEG spectrum", workers = 4, out_db = "results")` |
| Plot a recording | `i$hypno()`, `i$psd("EEG")`, `i$spec("EEG")` |

For cohort processing, `p$procn()` and `p$proc_parallel()` are equivalent.
They collect results in memory by default; use `out_db` or `out_text` to
write directly to disk when outputs are large. See the
[parallel processing guide](ref.md#parallel-project-processing) for
arguments, examples, memory use, and error handling.

The [R reference](ref.md) covers projects, instances, results, and helper
methods. The [plotting page](viz.md) covers the base R graphics functions.
Luna commands and their options are shared across R, Python, and the
command line; look these up in the [command reference](../../ref/index.md).

## Translating lunapi examples

Replace Python's `.` method access with `$`, use `TRUE`/`FALSE`/`NULL`,
and use R vectors for channel names or epoch numbers:

| Python | R |
| --- | --- |
| `import lunapi as lp` | `library(luna)` |
| `p = lp.proj()` | `p <- lp$proj()` |
| `i = p.inst(1)` | `i <- p$inst(1)` |
| `i.eval('PSD sig=EEG spectrum')` | `i$eval("PSD sig=EEG spectrum")` |
| `i.table('PSD', 'CH_F')` | `i$table("PSD", "CH_F")` |
| `i.data(['EEG', 'EMG'], time=True)` | `i$data(c("EEG", "EMG"), time = TRUE)` |
| `r['PSD: CH_F']` | `r[["PSD: CH_F"]]` |

A few differences matter when translating:

- Signal extraction returns data frames directly. There is no separate
  pair of column names and a NumPy matrix to unpack.
- `p$get_n()`, `p$get_id()`, `p$get_edf()`, and `p$get_annots()` use
  **1-based** sample-list positions in R; those Python helpers use 0-based
  positions. `inst(1)` selects the first recording in both interfaces.
- A saved R `ProcResult` keeps its tables after later commands. Project
  and instance `table()` methods query their current results instead.
- Missing output tables and unknown signal/annotation names raise R
  errors. Use `strata()`, `chs()`, and `annots()` to see available names.
- R plotting methods use base graphics. See [plotting](viz.md) for sizing
  and data requirements.

The R interface wraps the main project/instance workflow, rather than
all Python convenience functions. For Python-only functionality, see
[accessing lunapi directly](ref.md#accessing-lunapi-directly).

## Moving from the old R package

For scripts written before 1.8.0, `library(luna)` stays the same but
the analysis calls need updating. See the
[migration table](ref.md#migrating-old-r-scripts) for their replacements.
