# R interface reference

This reference describes **`luna` 1.8.0** from
[`remnrem/lunaR`](https://github.com/remnrem/lunaR). Start with
[installation](../../download/lunar.md) and the [worked example](example.md).
For the commands inside `eval()` and `proc()`, use the
[Luna command reference](../../ref/index.md).

```r
library(luna)
p <- lp$proj()
```

`lp` provides module-level helpers. `p` is the project and sample list;
`i <- p$inst(...)` opens one recording. Methods use `$`, just as in
`p$sample_list()` or `i$eval()`.

## Setup and version checks

| Call | Use |
| --- | --- |
| `luna_config()` | Show the Python path, R package version, lunapi version, engine version, and setup status; starts Python if needed |
| `lunapi_available(initialize = FALSE)` | Check availability without starting Python; `FALSE` before first use can simply mean Python has not started |
| `lunapi_available(initialize = TRUE)` | Try loading the engine and return `TRUE` or `FALSE` |
| `install_lunapi(envname = NULL, ...)` | Install a compatible lunapi into a user-managed Python environment; additional arguments go to `reticulate::py_install()` |
| `lp$version()` | Return a list with `lunapi` and `luna` versions |

See [installation troubleshooting](../../download/lunar.md#fixing-a-missing-or-outdated-lunapi)
for selecting an existing Python or fixing a missing lunapi.

## Projects

### `lp$proj()`

```r
p <- lp$proj(verbose = TRUE)
```

Return the project object. There is one Luna engine per R session:
calling `lp$proj()` again returns the same project and enables console
output. Use `p$silence(TRUE)` after creating it if you want quiet output.

### Sample lists and instances

| Method | Arguments and result |
| --- | --- |
| `p$sample_list(filename = NULL, path = NULL, df = TRUE)` | Read a sample list and return its count invisibly. With no filename, return the current sample list as a data frame with `ID`, `EDF`, and `Annotations`; `df = FALSE` returns a list instead |
| `p$build(args)` | Scan folders using Luna's `--build` syntax; `args` is a character vector of arguments. Replaces the sample list and returns the count invisibly |
| `p$nobs()` | Number of entries in the sample list |
| `p$validate()` | Check inputs; return a data frame with `ID`, `Filename`, and `Valid` |
| `p$clear()` | Clear the sample list |
| `p$inst(n)` | Open entry `n`, using a 1-based number or a sample-list ID |
| `p$empty_inst(id, nr, rs, startdate = "01.01.00", starttime = "00.00.00")` | Create an in-memory recording of `nr` EDF records, each `rs` seconds long |
| `p$get_n(id)` | Return the 1-based sample-list position of an ID, or `NA` if absent |
| `p$get_id(n)` | Return the ID at 1-based position `n` |
| `p$get_edf(x)` | Return the EDF path for an ID or 1-based position |
| `p$get_annots(x)` | Return annotation paths for an ID or 1-based position |

Read the tutorial project and open a recording:

```r
p$sample_list("tutorial/s.lst", path = "tutorial")
i <- p$inst("nsrr02")
```

`path` is the base directory for relative paths in the sample list.
Without it, those paths are resolved relative to the current working
directory. Reading a new sample list replaces the current one.

If a character ID is absent from the sample list, `p$inst(id)` creates
an empty instance with that ID. You can attach files directly:

```r
j <- p$inst("my-recording")
j$attach_edf("tutorial/edfs/learn-nsrr01.edf")
j$attach_annot("tutorial/edfs/learn-nsrr01-profusion.xml")
```

This opens an instance; it does not add a row to the sample list used
by `p$proc()`.

### Running commands across a project

| Method | Result |
| --- | --- |
| `p$proc(cmdstr)` | Run commands across the sample list; return a saved `ProcResult` |
| `p$silent_proc(cmdstr)` | The same run with console output suppressed |
| `p$strata()` | Available `Command`/`Strata` pairs for current project output, or `NULL` if none |
| `p$table(cmd, strata = "BL")` | Retrieve one current project table as a data frame |
| `p$variables(cmd, strata = "BL")` | Variable names in a current project table |
| `p$commands()` | A data frame of commands with current output |
| `p$empty_result_set()` | `TRUE` if no project output is available |
| `p$desc()` | Print and invisibly return a summary of sample-list recordings |
| `p$import_db(f, s = NULL)` | Import an output database, optionally restricted to IDs `s`; return imported IDs invisibly |

```r
r <- p$proc("HEADERS & PSD sig=EEG spectrum max=30")
r$strata()
head(r$table("PSD", "CH_F"))
```

Run `p$validate()` before a cohort analysis. As in serial lunapi, an
input that cannot be loaded stops the run. The native R `proc()` method
does not provide Python's parallel/partial-failure processing options;
see [accessing lunapi directly](#accessing-lunapi-directly).

Import an existing `luna -o` SQLite output database:

```r
p$import_db("out.db")
p$strata()
db_psd <- p$table("PSD", "CH_F")
```

Use `p$import_db("out.db", s = c("nsrr01", "nsrr02"))` to restrict IDs.
Importing results does not attach their EDFs. For large databases, see
also the Python [`destrat` reader](../../lunapi/ref.md#lpdestrat).

### Variables and parameters

| Method | Use |
| --- | --- |
| `p$vars(key = NULL, value = NULL)` | No arguments: named list of current variables. A key: get its value. Key and value: set it. A named list: set several variables |
| `p$var(key = NULL, value = NULL)` | Alias for `vars()` |
| `p$clear_vars(key = NULL)` | Remove selected variables, or clear project variables when no key is given |
| `p$clear_ivars()` | Clear individual-level variable definitions in the engine |
| `p$include(f)` | Read variable assignments from a Luna parameter file |
| `p$aliases()` | Return current aliases as a data frame |

```r
p$vars("analysis_sig", "EEG")
i$eval("PSD sig=${analysis_sig} spectrum max=30")
p$vars("analysis_sig")
p$clear_vars("analysis_sig")
```

A vector value is converted to a comma-separated string; logical values
become `1` or `0`. To set several variables:

```r
p$vars(list(analysis_sig = "EEG", upper_frequency = 30))
p$vars(c("analysis_sig", "upper_frequency"))
```

Luna's `${name}` substitution stays inside the command string; R does
not expand it. See [variables](../../luna/args.md#variables) for the
underlying command-language rules.

### Console output and session state

| Method | Use |
| --- | --- |
| `p$silence(b = TRUE, verbose = FALSE)` | Suppress console output, or restore it with `b = FALSE` |
| `p$is_silenced()` | Return the current console-output setting |
| `p$flush()` | Flush console output |
| `p$reset()` | Clear the engine's problem flag after an error |
| `p$reinit()` | Reinitialize the engine, clearing its state and results |

Errors from R methods normally clear the problem flag automatically.
`reinit()` resets the shared engine; reopen the project data and create
new instances afterwards.

## Instances

An instance represents a recording and its current in-memory signals,
annotations, masks, and outputs. Commands have cumulative effects on
that instance. File-backed data can be restored with `refresh()`;
use a freeze to save an intermediate analysis state.

### Attach and inspect

| Method | Result |
| --- | --- |
| `i$attach_edf(f)` | Attach an EDF to the instance |
| `i$attach_annot(annot)` | Attach annotations from a file or directory |
| `i$refresh()` | Reload the original file-backed recording and annotations |
| `i$drop()` | Release the instance's recording |
| `i$id()` | Recording ID |
| `i$desc()` | Print and invisibly return a recording summary |
| `i$stat()` | Data frame with a `Value` column and named rows for recording properties |
| `i$channels()` / `i$chs()` | Data frame with a `Channels` column |
| `i$headers()` | Channel-level `HEADERS` output as a data frame, including `CH` and `SR` |
| `i$annots()` | Data frame with an `Annotations` column |
| `i$has_channels(ch)` / `i$has(ch)` | Logical result for each channel in `ch` |
| `i$has_annots(anns)` / `i$has_annot(anns)` | Logical result for each annotation class in `anns` |
| `i$has_staging()` | Whether the recording has sleep-stage annotations |

```r
i$chs()$Channels
i$has_channels(c("EEG", "EMG"))
i$annots()$Annotations
```

### Run commands and retrieve output

| Method | Result |
| --- | --- |
| `i$eval(cmdstr)` | Run commands and invisibly return available `Command`/`Strata` pairs |
| `i$proc(cmdstr)` | Run commands and return a saved `ProcResult` |
| `i$silent_proc(cmdstr)` | Return a saved result with console output suppressed |
| `i$eval_dummy(cmdstr)` | Return parsed command text without running the analysis |
| `i$strata()` | Current output's `Command`/`Strata` pairs, or `NULL` if none |
| `i$table(cmd, strata = "BL")` | One current output table as a data frame |
| `i$variables(cmd, strata = "BL")` | Variable names in that table |
| `i$empty_result_set()` | `TRUE` if no instance output is available |

Use `&` to separate commands, a multiline string, or a character vector:

```r
i$eval(c("EPOCH len=30", "PSD sig=EEG spectrum max=30"))
i$strata()
spectrum <- i$table("PSD", c("CH", "F"))
```

`"CH_F"`, `"F_CH"`, and `c("CH", "F")` select the same factor set.
`"BL"` selects baseline output with no factors; it does not mean all
output tables. Requesting a missing table raises an error listing the
available commands or strata.

Current output can be replaced by later commands, including helpers
such as `headers()`, `epochs()`, `stages()`, and the instance plots.
Use `r <- i$proc(...)` to keep the results of a particular run.

### Epochs, masks, and saved states

| Method | Use |
| --- | --- |
| `i$epoch(f = "")` | Run `EPOCH`, appending options `f`, e.g. `"len=30"` |
| `i$epochs()` | Return an epoch table with available columns `E`, `E1`, `LABEL`, `HMS`, `START`, `STOP`, `DUR` |
| `i$mask(f = NULL)` | Apply one or more `MASK` option strings, then run `RE`. With `NULL`, do nothing |
| `i$segments()` | Return the `SEGMENTS` table, stratified by `SEG` |
| `i$stages()` | Return per-epoch `STAGE` output as a data frame |
| `i$freeze(f)` | Save the current state under tag `f` |
| `i$thaw(f, remove = FALSE)` | Restore a saved tag; optionally remove the saved copy |
| `i$empty_freezer()` | Remove all saved freezes |

```r
i$epoch("len=30")
i$freeze("before_mask")
i$mask("ifnot=N2")
i$epochs()
i$thaw("before_mask")
i$empty_freezer()
```

`mask()` restructures the recording immediately. To inspect or combine
masks before restructuring, use `i$eval("MASK ...")` and run `RE` yourself.
See [masks](../../ref/masks.md) and [freezes](../../ref/freezes.md).

### Annotation events

| Method | Result |
| --- | --- |
| `i$fetch_annots(anns, interp = -1)` | Event data frame: `Class`, `Start`, `Stop` |
| `i$fetch_fulls_annots(anns, add_keys = FALSE)` | Event data frame: `Class`, `Instance`, `Channel`, `Meta`, `Start`, `Stop` |
| `i$fetch_full_annots(anns, add_keys = FALSE)` | Alias for the spelling above |
| `i$insert_annot(label, intervals, durcol2 = FALSE)` | Add events from start/stop seconds; with `durcol2 = TRUE`, the second column is duration |

Fetch times are seconds from recording start, sorted by event timing
and rounded to milliseconds. `add_keys` controls metadata-key inclusion
in the full event output. `interp` is passed to the engine's annotation
fetcher; leave the default for ordinary interval annotations.

```r
events <- i$fetch_annots("Arousal")
i$insert_annot("review", list(c(10, 12), c(40, 45)))
i$fetch_full_annots("review")
```

Insert intervals can be a two-column matrix/data frame, a list of
start/stop pairs, or one pair `c(start, stop)`. To read annotations from
a file, use `attach_annot()` instead.

### Signals and time intervals

| Method | Result |
| --- | --- |
| `i$data(chs, annots = NULL, time = FALSE)` | All requested signals as one data frame |
| `i$e2i(epochs)` | Two-column `start`/`stop` matrix for 1-based epoch numbers |
| `i$s2i(secs)` | Convert start/stop seconds into a two-column time-point matrix |
| `i$slice(intervals, chs, annots = NULL, time = FALSE)` | One data frame containing samples across the intervals |
| `i$slices(intervals, chs, annots = NULL, time = FALSE)` | A list of data frames, one per interval |
| `i$insert_signal(label, data, sr)` | Add a new numeric signal with sample rate `sr` Hz |
| `i$update_signal(label, data)` | Replace an existing channel's samples |

With `time = TRUE`, `T` gives seconds. Requested annotation classes
become 0/1 columns, alongside the signal columns. Unknown names raise
an error. Extract channels with a common sample rate; use
[`RESAMPLE`](../../ref/manipulations.md#resample) first if needed.

```r
whole <- i$data("EEG", annots = "Arousal", time = TRUE)
intervals <- i$s2i(list(c(10, 15), c(30, 35)))
parts <- i$slices(intervals, "EEG", time = TRUE)
i$epoch("len=30")
first_epoch <- i$slice(i$e2i(1), "EEG", time = TRUE)
```

`e2i()` and `s2i()` return Luna time-points in **nanoseconds**, for use
with `slice()`/`slices()`. Pass seconds to `s2i()`, rather than supplying
seconds directly to `slice()`. By contrast, `insert_annot()` takes
**seconds** directly. Use `i$epochs()` to identify available epochs after
masking or restructuring.

Signals inserted or updated from R must cover the current recording
with the expected number of samples. Missing values are rejected.
Use `update_signal()` to replace an existing channel; `insert_signal()`
requires a new label.

Create a short recording entirely in R:

```r
demo <- p$empty_inst("demo", nr = 120, rs = 1)
sr <- 128
t <- seq(0, 120 - 1 / sr, by = 1 / sr)
demo$insert_signal("EEG", sin(2 * pi * 10 * t), sr)
demo$insert_annot("event", c(10, 12))
demo$eval("PSD sig=EEG spectrum max=20")
head(demo$table("PSD", "CH_F"))
```

### Instance variables

| Method | Use |
| --- | --- |
| `i$vars(key = NULL, value = NULL)` | Get all instance variables, get one variable, set one variable, or set a named list |
| `i$var(key = NULL, value = NULL)` | Alias for `vars()` |
| `i$clear_vars(keys = NULL)` | Clear selected instance variables, or all if no keys are given |
| `i$refresh_channel_vars()` | Refresh built-in channel-group variables after changing signals |

These variables belong to `i`. Use project `p$vars()` for project-wide
settings.

## Saved results: `ProcResult`

Both project and instance `proc()` methods return a `ProcResult`.
It holds a snapshot of that run's tables, so later commands do not
change the saved result. Data frames are converted when requested.

```r
r <- i$proc("PSD sig=EEG spectrum max=30")
r$keys()
psd <- r[["PSD: CH_F"]]
# Equivalent table lookup:
psd <- r$table("PSD", "CH_F")
i$eval("HEADERS")
# Still the spectrum from the earlier run:
head(r$table("PSD", "CH_F"))
```

| Member or method | Result |
| --- | --- |
| `r$keys()` / `names(r)` | Character vector of `"COMMAND: STRATA"` keys |
| `length(r)` | Number of output tables |
| `r$table(cmd, strata = "BL")` | One data frame, accepting factor vectors or strings |
| `r[["PSD: CH_F"]]` | A table by key; `r[[c("PSD", "CH_F")]]` also works |
| `r$strata()` | `Command`/`Strata` pairs, or `NULL` for no output |
| `r$commands()` | Commands represented in the result |
| `r$has_table(cmd, strata = "BL")` | Whether a table is available |
| `r$get(key, default = NULL)` | Table by key, or `default` if absent |
| `r$tables` / `r$items()` | Named list of all data frames |
| `r$values()` | Unnamed list of all data frames |
| `r$ok` | `TRUE` for a returned native R result; failures raise R errors |
| `r$errors` | Empty `ID`/`error` data frame for native R results |
| `r$workers` | `1` for native R processing |

`r$tables` is a property, without parentheses. The Python parallel-result
metadata and `.copy()` method do not apply to the native R snapshot.

## Results helpers

### Tables, subsets, and repeated analyses

| Helper | Use |
| --- | --- |
| `lp$strata(ts)` | List command/strata pairs in a `ProcResult` or named list of tables |
| `lp$table(ts, cmd, strata = "BL")` | Retrieve a table |
| `lp$tables(ts)` | Return the named list of tables |
| `lp$show(dfs)` | Print each table |
| `lp$subset(df, ids = NULL, qry = NULL, vars = NULL)` | Filter IDs, an R query expression, and/or columns; `vars` keeps `ID` too |
| `lp$concat(dfs, tlab, vars = NULL, add_index = NULL, ignore_index = TRUE)` | Row-bind table `tlab` across a named list of results; fill absent columns with `NA` |
| `lp$cmdfile(f)` | Read a command file into a single string |

`qry` is an **R expression**, for example `"F >= 1 & F <= 20"`, not a
pandas query. With `concat()`, `add_index` names a column containing the
input list names. `vars` selects output columns; include `ID` explicitly
if you want it. `ignore_index` is accepted for compatibility; R row names
are rebuilt.

```r
selected <- lp$subset(psd, ids = "nsrr02", qry = "F >= 1 & F <= 20",
                      vars = c("CH", "F", "PSD"))
r1 <- i$proc("PSD sig=EEG spectrum max=20")
r2 <- i$proc("PSD sig=EEG spectrum max=30")
combined <- lp$concat(list(to20 = r1, to30 = r2), "PSD: CH_F", add_index = "RUN")
```

### Text output

| Helper | Use |
| --- | --- |
| `lp$list_text_tables(path, id = NULL)` | Inspect one individual's tables under a Luna text-output root |
| `lp$read_text_table(path, cmd_or_file, factors = NULL)` | Read the requested table across individuals into one data frame |

For a folder produced by command-line `luna -t`:

```r
lp$list_text_tables("out/txt")
text_psd <- lp$read_text_table("out/txt", "PSD", factors = c("B", "CH"))
```

Use a command name and factors, or a filename such as `"PSD_B_CH.txt"`.
See [text output](../../luna/destrat.md) for the folder layout.

### Command discovery

| Helper | Result |
| --- | --- |
| `lp$fetch_doms()` | Available command domains |
| `lp$fetch_cmds(dom)` | Commands in a domain |
| `lp$fetch_params(cmd)` | Command parameters |
| `lp$fetch_tbls(cmd)` | Command output tables |
| `lp$fetch_vars(cmd, tbl)` | Output variables in a table |
| `lp$fetch_desc_dom(dom)` | Domain description |
| `lp$fetch_desc_cmd(cmd)` | Command description |
| `lp$fetch_desc_param(cmd, param)` | Parameter description |
| `lp$fetch_desc_tbl(cmd, tbl)` | Table description |
| `lp$fetch_desc_var(cmd, tbl, var)` | Output-variable description |

```r
lp$fetch_cmds("power")
lp$fetch_params("PSD")
lp$fetch_tbls("PSD")
```

Use the table labels returned by `fetch_tbls()` when calling
`fetch_vars()` or the description helpers.

## Plots

Instance methods `i$hypno()`, `i$psd()`, and `i$spec()` run the needed
commands and draw the result. Module helpers `lp$hypno()`,
`lp$hypno_density()`, `lp$psd()`, `lp$spec()`, and `lp$topo_heat()` plot
existing R data. See [plotting](viz.md) for signatures and examples.

## Accessing lunapi directly

The main R workflow uses the methods above. Additional Python facilities,
such as `destrat`, EDF utilities, waveform extraction, and parallel cohort
processing, can be reached through reticulate. Use the
[lunapi reference](../../lunapi/ref.md) for those Python APIs.

| R member | Underlying Python object |
| --- | --- |
| `lp$py` | The imported `lunapi` module |
| `p$eng` | The low-level engine |
| `i$edf` | The low-level recording instance |

For high-level Python instance helpers, wrap the existing low-level
instance using `lp$py$inst(i$edf)`. For example, this retrieves the same
recording's channel labels through the Python interface:

```r
py_i <- lp$py$inst(i$edf)
py_i$chs()
```

To reach project helpers that have no native R method:

```r
py_p <- lp$py$proj()
```

This shares the same engine and sample list; it is not an independent
project. Python results follow reticulate's conversion rules and may
remain Python objects; they are not the native R `ProcResult` described
above. Check Python method arguments and indexing rules before using
these objects. Model-based helpers still require their model files.

For automated sleep staging, you can also run the shared
[`RUN-POPS`](../../ref/pops.md#run-pops) or
[`POPS`](../../ref/pops.md#pops-prediction) command with `i$eval()` or
`p$proc()` once the required resources are available.

## Migrating old R scripts

The old `l*` functions are replaced by methods on `lp`, a project, or an
instance. The table below is a translation guide for older examples;
those functions are not exported by 1.8.0.

| Old call | 1.8.0 replacement |
| --- | --- |
| <span id="lsl"></span>`sl <- lsl("s.lst")` | `p$sample_list("s.lst")` |
| <span id="lattach"></span>`lattach(sl, 2)` | `i <- p$inst(2)` |
| <span id="ledf"></span>`ledf(...)` | `i <- p$inst("id")`, then `i$attach_edf()` and `i$attach_annot()` |
| <span id="lemptyedf"></span>`lempty.edf(...)` | `i <- p$empty_inst(id, nr, rs)` |
| <span id="lstat"></span>`lstat()` | `i$stat()` |
| <span id="ldrop"></span>`ldrop()` | `i$drop()` |
| <span id="lrefresh"></span>`lrefresh()` | `i$refresh()` |
| <span id="lchs"></span>`lchs()` | `i$chs()$Channels` for a character vector |
| <span id="lannots"></span>`lannots()` | `i$annots()$Annotations` for a character vector |
| <span id="leval"></span>`leval(cmd)` | `i$eval(cmd)` for current output, or `r <- i$proc(cmd)` to keep results |
| <span id="levalproject"></span>`leval.project(sl, cmd)` | `r <- p$proc(cmd)` |
| <span id="lcmd"></span>`lcmd(file)` | `lp$cmdfile(file)`, passed to `eval()` or `proc()` |
| <span id="lepoch"></span>`lepoch()` | `i$epoch()` |
| <span id="letable"></span>`letable()` | `i$epochs()` |
| <span id="lstages"></span>`lstages()` | `i$stages()`; use `$STAGE` for the label vector |
| <span id="le2i"></span>`le2i(e)` | `i$e2i(e)` |
| <span id="ldata"></span>`ldata(e, chs, annots)` | `i$slices(i$e2i(e), chs, annots)` |
| <span id="ldataintervals"></span>`ldata.intervals(...)` | Convert seconds with `i$s2i()`, then use `i$slice()` or `i$slices()` |
| <span id="laddannot"></span>`ladd.annot(...)` | `i$insert_annot(label, intervals)` with start/stop seconds |
| <span id="laddsignal"></span>`ladd.signal(...)` | `i$insert_signal(label, data, sr)` |
| <span id="laddannotfile"></span>`ladd.annot.file(file)` | `i$attach_annot(file)` |
| <span id="ldb"></span>`ldb(file)` | `p$import_db(file)`, then `p$table()` |
| <span id="lx"></span>`lx(k, ...)` | `r$strata()`, `r$table(cmd, strata)`, or `r[["CMD: STRATA"]]` |
| <span id="lx2"></span>`lx2(...)` | `lp$concat()` over a named list of results |
| <span id="lid"></span>`lid(...)` | `lp$subset(df, ids = ...)` |
| <span id="ltxttab"></span>`ltxttab(...)` | `lp$read_text_table()` |
| <span id="lset"></span>`lset(key, value)` | `p$vars(key, value)`; `p$include(file)` for a parameter file |
| <span id="lvar"></span>`lvar(key)` | `p$vars(key)` |
| <span id="lreset"></span>`lreset()` | Use `p$clear_vars()` for variables, or `p$reinit()` for a full engine reset; `p$reset()` only clears the error flag |
| <span id="llog"></span>`llog(0)` | `p$silence(TRUE)` |
| <span id="lstgcols"></span>`lstgcols(stages)` | `lp$stgcol(stages)` |
| <span id="literate"></span>`literate(...)` | Apply an R function to `i$slices(...)` with `lapply()` or `vapply()` |
| <span id="lbands"></span>`lbands(...)` | Use shared Luna filter commands and extract the resulting signals |
| <span id="lsanitize"></span>`lsanitize(...)` | Use the labels returned by `chs()`, `annots()`, and `strata()`; configure label handling through Luna variables |
| <span id="ldenoise"></span>`ldenoise(...)` | No native R replacement; apply your chosen R denoising method to extracted signals |

For the plotting replacements, see [plot migration](viz.md#moving-from-old-r-plots).
