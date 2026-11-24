# Take two in Python

This second page revisits [Quick start in Python](python1.md) with
more detail on sample lists, analysis syntax, result tables, and
reusable parameters. It prepares you for the signal analyses in
[Deeper dive in Python](python3.md).

Follow the examples as notebook cells, as described in
[Quick start](python1.md). In a fresh kernel, load the same tutorial data.
Change the path to your extracted `tutorial` folder:

```python
from pathlib import Path
import os

import lunapi as lp
import pandas as pd

data_dir = Path("~/tutorial").expanduser().resolve()
os.chdir(data_dir)
proj = lp.proj()
proj.sample_list("s.lst")
```

## Sample-lists

We loaded `s.lst` in Quick start. Now we will look at how to locate its
files and select recordings for analysis. To view the current list:

```python
samples = proj.sample_list()
samples[["ID", "EDF"]]
```

```text
       ID                    EDF
1  nsrr01  edfs/learn-nsrr01.edf
2  nsrr02  edfs/learn-nsrr02.edf
3  nsrr03  edfs/learn-nsrr03.edf
```

With no argument, `proj.sample_list()` shows the current list. Giving
it a filename loads that list instead. Luna looks for relative EDF
paths from Python's working directory, which is why we first use
`os.chdir(data_dir)`. For example, if your sample list is `/data/s.lst`
and its EDF paths are relative to `/data`, you can load it from any
working directory with:

```python
proj.sample_list("/data/s.lst", path="/data/")
```

---

We can also create that sample list by asking `build()`
to find and link the files in a folder:

```python
proj.build(["edfs", "-ext=-profusion.xml"])
proj.sample_list()
```

```text
       ID                    EDF                        Annotations
1  nsrr01  edfs/learn-nsrr01.edf  {edfs/learn-nsrr01-profusion.xml}
2  nsrr02  edfs/learn-nsrr02.edf  {edfs/learn-nsrr02-profusion.xml}
3  nsrr03  edfs/learn-nsrr03.edf  {edfs/learn-nsrr03-profusion.xml}
```

Luna uses the EDF filenames as IDs and links each recording to its
matching annotation file, identified here by the `-profusion.xml` ending.  Without
an explicit `-ext` option, `build()` matches `.edf` files to `.annot`, `.xml`, `.eannot`
and `.tsv`.

---


For one individual, attach that instance, assigned to the variable `p`:

```python
p = proj.inst("nsrr02")
```

Note that `proj.inst("nsrr02")` and `proj.inst(2)` both open the second
tutorial recording. Luna's positional selection is 1-based; ordinary
pandas `.iloc` indexing is still 0-based.

We can now run Luna commands via `proc()`:
```
result = p.proc("STATS sig=ECG,EMG")
```

Note that `proc()` is similar to `eval()` except it also returns all results to a `ProcResult` object,
here called `result`, that can be queried. So now instead of `p.strata()` we can use `result.strata()`
to view available results:

```python
result.strata()
```

```text
  Command Strata
    STATS     CH
```

To extract this table and show only the `RMS` variable:
```
single = result.table("STATS", "CH")
single[["ID", "CH", "RMS"]]
```

```text
       ID   CH        RMS
   nsrr02  ECG   0.274497
   nsrr02  EMG  19.706464
```

---

To process selected individuals, we can set Luna's `id` option to their
comma-separated IDs. The sample list itself stays unchanged:

```python
proj.var("id", "nsrr01,nsrr03")
```

Now we apply `proj.proc()` rather than `p.proc()` - i.e. this would be to all individuals in the sample list, _except_
we just passed a list of IDs to be included, i.e. to exclude those in the sample list not in this list.

```python
result = proj.proc("STATS sig=EEG")
```
And extracting and displaying a subset of the returned table:
```
subset_stats = result.table("STATS", "CH")
subset_stats[["ID", "CH", "RMS"]]
```
```text
       ID   CH        RMS
   nsrr02  EEG  41.044234
   nsrr03  EEG    54.2708
```

To check the value of a current variable:
```python
proj.var( 'id' )
```

To clear the selection before continuing, so subsequent analyses include
all three recordings:

```python
proj.var("id", "")
```

Or run `proj.reinit()` to reinitialize the entire project, wiping all variables and cached results.

## Luna syntax

An analysis has several parts, from selecting the inputs to retrieving
the results. Here is how they fit together:

| Setting | Example | Purpose |
|---|---|---|
| Project input | `proj.sample_list("s.lst")` | Select recordings and their annotations |
| Project variable | `proj.vars("sig", "EEG,ECG,EMG")` | Select signals when recordings are loaded |
| Analysis operation | `"STATS sig=ECG epoch"` | Calculate statistics, with options for this operation |
| Result selection | `result.table("STATS", "CH_E")` | Retrieve a particular output table |

Setting `sig` on the project chooses which signals Luna **loads**.
A command's `sig=...` then chooses which loaded signals to analyze.
Set project variables and aliases before opening a recording or calling
`proj.proc()`, so they take effect when the EDF is read:

```python
proj.vars("sig", "EEG,ECG,EMG")
p = proj.inst("nsrr02")
p.chs()
```

```text
  Channels
       ECG
       EMG
       EEG
```

Variables also let us reuse an analysis with different settings. For
example, Luna replaces `${epoch_len}` below with the value we supply:

```python
proj.vars("epoch_len", 20)
result = proj.proc("EPOCH len=${epoch_len}")
result.table("EPOCH")[[ "ID", "DUR", "NE" ]]
```

```text
       ID   DUR    NE
   nsrr01  20.0  2046
   nsrr02  20.0  1792
   nsrr03  20.0  2046
```

Alternatively, you can use Python variables instead of Luna variables,
to construct the string with Python, for example `f"EPOCH
len={epoch_len}` when `epoch_len` is a Python variable.  The advantage
with Luna variables is that the same script will work via lunaC or
lunaR.

To list all currently defined variables, use `proj.vars()`.  To clear
a specific variable:

```python
proj.clear_vars("epoch_len")
```

Note that `proj.clear_vars("sig")` clears the project-level signal
selection. It does not add signals back to an already opened instance;
open a new instance after changing load-time settings.



## Luna commands

By convention, operation/command names are usually uppercase, such as
`EPOCH`, `STATS`, `MASK`, and `PSD`. Their options are usually
lowercase. An option can be a flag (`epoch`) or a name/value pair
(`len=30`, `sig=ECG`). Some operations have a special spelling, such
as `mV` for unit conversion.

An ampersand or a newline separates steps in a single analysis string:

```python
result = proj.proc("EPOCH len=30 & STATS epoch sig=EEG,ECG,EMG")
```

For longer analyses, use a Python list and join it. This makes each
step easy to edit or construct programmatically:

```python
steps = [
    "EPOCH len=30",
    "STATS epoch sig=EEG,ECG,EMG",
]
result = proj.proc(" & ".join(steps))
```

Alternatively, simply use a multiline string

```python
result = proj.proc("""
    EPOCH len=30
    % obtain epoch-level statistics 
    STATS epoch sig=EEG,ECG,EMG
""")
```

Note that comments can be included, but start with `%` not the usual Python `#`. 

---

If you already have an analysis in a text file, `lp.cmdfile()` reads
it. The tutorial includes `cmd/first.txt`, which selects a stage and
calculates EEG statistics. The file uses `${stage}`, which must be set
before running it:

```python
proj.vars("stage", "N2")
result = proj.proc( lp.cmdfile("cmd/first.txt") )
n2 = result.table("STATS", "CH_STAGE")
n2[["ID", "CH", "STAGE", "RMS"]]
```

```text
      ID   CH STAGE        RMS
  nsrr01  EEG    N2  10.646079
  nsrr02  EEG    N2   14.74242
  nsrr03  EEG    N2  14.496728
```

```python
proj.clear_vars("stage")
```


## Output format

A processing call returns a result object. Start by asking which tables
were produced:

```python
result = proj.proc("EPOCH & STATS epoch sig=EEG,EMG & PSD spectrum dB sig=EEG")
result.strata()
```

```text
  Command Strata
    EPOCH     BL
      PSD   B_CH
      PSD     CH
      PSD   CH_F
    STATS     CH
    STATS   CH_E
```

Luna groups results according to what they describe, such as a channel
or an epoch. These groupings are called **stratifying factors** and
appear in the table names:

| Factor | Meaning | Example |
|---|---|---|
| `BL` | Individual-level output, with no additional factor | Epoch count for each recording |
| `CH` | Channel | Whole-night metrics for each signal |
| `CH_E` | Channel and epoch | Per-epoch signal statistics |
| `CH_F` | Channel and frequency | Power spectrum |
| `CH_B` | Channel and frequency band | Band power |

`BL` is the label used to select the baseline table; it is not a column
that must be present in that table. Other factors appear as columns,
along with `ID` and the output measurements.

This run produced both whole-signal and per-epoch statistics. Retrieve
them into separate data frames before running another analysis:

```python
whole = result.table("STATS", "CH")
epoch_stats = result.table("STATS", "CH_E")
epoch_stats[["ID", "CH", "E", "RMS"]].head(3)
```
```text
       ID   CH  E        RMS
   nsrr01  EEG  1   9.922389
   nsrr02  EEG  1  11.977782
   nsrr03  EEG  1  12.525565
```

`STATS epoch` produces per-epoch measurements and whole-signal summaries,
including summaries of the epoch measurements. These are different
tables, even when they share a variable name such as `RMS`.

The `result` object gives access to Luna's current results. Once we
retrieve a table into `whole` or `epoch_stats`, that data frame remains
available even after another command replaces the current results.
Retrieve what you need first, including before helper calls such as
`p.stages()`, which also run Luna commands.

---

If user-defined [`TAG`](../../ref/summaries/#tag)
have been applied, they will feature here too.  For example,
if we pass in a variable `${stages}`, the comma-separated
values of which are looped-over with the `#{stage}` loop variable, 
we can track the value in the output using a `TAG`:

```python
proj.var( "stages" , "N2,R" ) 

result = proj.proc("""
 LOOP index=stage vals=${stages}
  TAG STG/#{stage}
  MASK ifnot=#{stage} 
  EPOCH table
  STATS epoch sig=EEG 
  PSD spectrum dB sig=EEG
 END-LOOP
 TAG STG/.
 """ )

result.strata()
```

```text
  Command     Strata
    EPOCH      E_STG
    EPOCH        STG
     MASK  EMASK_STG
      PSD   B_CH_STG
      PSD   CH_F_STG
      PSD     CH_STG
    STATS   CH_E_STG
    STATS     CH_STG
```

To then extract, e.g. epoch counts, these are now stratified by `STG`:
```python
result.table( 'EPOCH', 'STG' ) [[ 'ID', 'STG', 'NE' ]]
```
```
       ID STG   NE
   nsrr01  N2  523
   nsrr02  N2  399
   nsrr03  N2  375
   nsrr01   R  238
   nsrr02   R  120
   nsrr03   R    0
```

To get sigma power per person/stage:
```python
tbl = result.table( 'PSD', 'B_CH_STG' )
tbl[ tbl.B == 'SIGMA' ]
```
```text
      ID      B   CH STG       PSD    RELPSD
  nsrr01  SIGMA  EEG  N2  3.577489  0.022371
  nsrr02  SIGMA  EEG  N2  9.026692  0.041952
  nsrr03  SIGMA  EEG  N2  4.731103  0.017993
  nsrr01  SIGMA  EEG   R  1.093931  0.027083
  nsrr02  SIGMA  EEG   R  4.606675  0.020051
```

Note that we can also pass a Python list with the factors in any
order: this achieves the same as using the string `B_CH_STG`:

```python
tbl = result.table( 'PSD',  [ 'CH' , 'B' , 'STG' ] ) 
```

## Manipulating outputs

Python analyses can retrieve tables directly and use pandas to do the
selection and reshaping.  For example, to select only the fields you
want and arrange channels in columns:

```python
rms = whole.pivot(index="ID", columns="CH", values="RMS").astype(float)
rms.round(2)
```

```text
CH       ECG    EEG    EMG
ID
nsrr01  0.27  37.80  13.97
nsrr02  0.27  41.04  19.71
nsrr03  0.30  54.27  18.90
```

The per-epoch table has a separate row for each channel and epoch,
which is useful for plotting and combining tables. To compare channels
side by side, we can instead put them in columns for one individual:

```python
second = epoch_stats.loc[epoch_stats["ID"] == "nsrr02"].copy()
second["E"] = pd.to_numeric(second["E"])
second["RMS"] = pd.to_numeric(second["RMS"])
epoch_rms = second.pivot(index="E", columns="CH", values="RMS").sort_index()
epoch_rms.head(3).round(2)
```

```text
CH   ECG    EEG    EMG
E
    0.27  11.98   4.91
    0.28  15.88   9.91
    0.27  12.47  22.73
```

Some Luna tables arrive with pandas `object` columns even when the
values are numeric. Use `pd.to_numeric()` or `.astype(float)` on the
measurement columns before calculations or numeric comparisons. Keep
IDs and channel names as text.

Rounding above changes the displayed table; the original data frames
retain their precision. Save an unrounded table as CSV, then load it
with pandas in a later session:

```python
epoch_stats.to_csv("python-epoch-stats.csv", index=False)
loaded = pd.read_csv("python-epoch-stats.csv")
loaded.shape
```

```text
(11769, 27)
```

If you process individuals or channels separately, concatenate the
resulting data frames. For example, this runs EEG statistics for each
individual and combines the results:

```python
parts = []
for individual_id in proj.sample_list()["ID"]:
    p = proj.inst(individual_id)
    result = p.proc("STATS sig=EEG")
    parts.append(result.table("STATS", "CH"))

combined = pd.concat(parts, ignore_index=True)
combined[["ID", "CH", "MEAN", "RMS"]]
```

```text
      ID   CH      MEAN        RMS
  nsrr01  EEG -0.301199  37.801351
  nsrr02  EEG -0.370447  41.044234
  nsrr03  EEG  -0.11223    54.2708
```

When combining tables, keep the columns that identify each result,
such as `ID`, channel, and epoch. This lets you distinguish measurements
that share a name but describe different things, such as whole-night
and per-epoch RMS.

## Parameter files

A parameter file collects project-specific aliases and variables.
The supplied `cmd/vars.txt` contains these tab-separated name/value
pairs:

```text
alias    EEG1|EEG
alias    EEG2|EEG(sec)
alias    OXSTAT|"OX STAT"
eeg      EEG1,EEG2
myepoch  10
nrem     N1,N2,N3
```

The actual file uses two whitespace-separated fields per line, or key `=` value pairs.
In the above, `alias` means that the first name is preferred: source channel `EEG` becomes
`EEG1`, and `EEG(sec)` becomes `EEG2`. The remaining rows define Luna
variables to substitute into analysis strings.

Clear previous settings, reload the sample list, and include the
parameters before opening a recording:

```python
proj.reinit()
proj.sample_list("s.lst", path="")
proj.include("cmd/vars.txt")
p = proj.inst("nsrr01")
proj.vars(["eeg", "myepoch", "nrem"])
```

```text
{'eeg': 'EEG1,EEG2', 'myepoch': '10', 'nrem': 'N1,N2,N3'}
```

All project handles in a Python session share one Luna engine.
`proj.reinit()` clears its settings so we can start this example fresh.
After a reset, open new recording instances, as we did with `p` above.

This analysis uses 10-second epochs and retains NREM sleep:

```python
result = p.proc("""
    EPOCH len=${myepoch}
    MASK all
    MASK unmask-if=${nrem} & RE
    STATS sig=${eeg}
""" )
nrem_stats = result.table("STATS", "CH")
nrem_stats[["ID", "CH", "RMS"]]
```

```text
       ID    CH        RMS
   nsrr01  EEG1  10.239962
   nsrr01  EEG2   9.767967
```

`MASK all` initially excludes all epochs. `MASK unmask-if=${nrem}`
includes epochs matching any of the three stages specified by
`${nrem}` which equals `N1,N2,N3`. In this context, we could also
achieve this simply by `MASK ifnot=${nrem}`

`RE` removes the remaining excluded epochs, then `STATS` measures the
two aliased EEG channels. The same operations are in `cmd/second.txt`,
which you can run with `p.proc(lp.cmdfile("cmd/second.txt"))`.

This separation lets you keep an analysis fixed while adapting aliases
and variables to a different dataset. For individual settings you can
also use a Python dictionary, for example
`proj.vars({"myepoch": "30", "nrem": "N2,N3"})`.

Continue to [Deeper dive in Python](python3.md) for epoch-level plots,
sleep architecture, masks, EDF transformations, artifact detection,
power spectra, and spindle detection.
