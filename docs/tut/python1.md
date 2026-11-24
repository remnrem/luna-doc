# Quick start in Python

This is the first of three Python tutorials: **Quick start**, [Take two](python2.md),
and [Deeper dive](python3.md). They cover the same analyses as the main
Luna tutorial, with all setup, processing, tables, and plots handled from
Python. You do not need to read the command-line tutorial first.

If you are new to Python, we recommend following this tutorial in a
**Jupyter notebook**. Run each code block as a cell, in order; results
appear below the cells. The tables here show selected outputs, with
Luna's processing logs omitted.

!!! note "Outside a notebook"
    In a script or another Python session, use `print(...)` when needed
    to display results—for example, `print(valid)` instead of `valid`.

We will work with three overnight polysomnograms from the National Sleep
Research Resource. Each has an EDF containing signals, including EEG,
ECG, and EMG, and a separate XML file containing manual sleep stages and
annotations such as apneas and arousals. In this first page we inspect
these files, calculate signal statistics, count events, and compare EEG
amplitude between sleep stages.

## Installing lunapi

Follow the [Python installation instructions](../lunapi/index.md#installation),
then open a notebook using the environment where you installed `lunapi`.

## Obtaining the data

Download the [tutorial ZIP archive](https://github.com/remnrem/luna-resources/releases/latest/download/luna-resources-tutorial.zip)
and extract it. The resulting `tutorial` folder contains:

- `edfs/`: three EDFs and their three XML annotation files;
- `cmd/`: text files containing example analysis steps and parameters;
- `s.lst`: a sample list linking each recording to its annotations.

Later examples will save tables and new EDFs in this folder, so use a
copy you can write to. Change `~/tutorial` below to the folder you
extracted. `os.path.expanduser()` expands `~` to your home directory.

```python
import os
os.chdir(os.path.expanduser("~/tutorial"))
```

Now import Luna and create an initial project:

```python
import lunapi as lp
proj = lp.proj()
```

You can check both package and engine versions with `lp.version()`. As
of writing, the current version is 1.8.1. Note that Luna, Lunapi and Lunascope
may have different version numbers.

The `proj` object is a **project**, which holds the list of recordings
and their analysis settings, but is currently empty.  We'll load a
sample list that has already been prepared: the tutorial includes
`s.lst`, a tab-separated text file without a header, that links the three tutorial
EDFs with their IDs and annotation files:

```python
proj.sample_list("s.lst")    # load a file
proj.sample_list()           # display the loaded list
```

```text
       ID                    EDF                        Annotations
1  nsrr01  edfs/learn-nsrr01.edf  {edfs/learn-nsrr01-profusion.xml}
2  nsrr02  edfs/learn-nsrr02.edf  {edfs/learn-nsrr02-profusion.xml}
3  nsrr03  edfs/learn-nsrr03.edf  {edfs/learn-nsrr03-profusion.xml}
```

Each row links an ID to an EDF and its annotations. Loading or building
a list replaces the project's current list. IDs can differ from EDF
filenames: `s.lst` uses `nsrr01`, `nsrr02`, and `nsrr03`. 

It is generally good practice to check the inputs before processing them:

```python
valid = proj.validate()
valid
```

```text
       ID                         Filename  Valid
1  nsrr01            edfs/learn-nsrr01.edf   True
2  nsrr01  edfs/learn-nsrr01-profusion.xml   True
3  nsrr02            edfs/learn-nsrr02.edf   True
4  nsrr02  edfs/learn-nsrr02-profusion.xml   True
5  nsrr03            edfs/learn-nsrr03.edf   True
6  nsrr03  edfs/learn-nsrr03-profusion.xml   True
```

Luna checks both the EDFs and their annotation files, giving one row
for each file. Every value in `Valid` should be `True`. If one is
`False`, check that the file exists at the path shown.  See some examples
in the [Luna walkthrough](https://zzz-luna.org/luna-walkthrough/p1/valid/)
where the validate command flags corrupt files.


## Displaying EDF files

Open the first recording by its sample-list ID:

```python
p = proj.inst("nsrr01")
p.desc()
```

`p` is an **instance**, or one person: it holds one recording and its
annotations in memory. You can also open it with `proj.inst(1)`, using
its position in the sample list. These positions are **1-based**, even
in Python.

Inspect the channels and their headers:

```python
p.chs()
```

```python
headers = p.headers()
headers[["CH", "SR", "PDIM"]].head()
```

The first few header rows are:

```text
         CH     SR PDIM
0  ABDO_RES   10.0    .
1   AIRFLOW   10.0    .
2       ECG  250.0   mV
3       EEG  125.0   uV
4   EEG_sec  125.0   uV
```

The headers tell us how each signal was recorded: `SR` is its sampling
rate in Hz, and `PDIM` is its physical unit. Here the ECG is in
millivolts and the EEG in microvolts. Their sampling rates also differ,
so check the headers before combining channels in an array.

We can also run, or _evaluate_ any Luna command(s) on this instance, for example, using the
[`DESC` command](../../ref/summaries/#desc) to describe the contents of the attached instance:

```python
p.eval("DESC table")
```

Strings such as `"DESC table"` or `"STATS sig=EEG"` are examples of
short _scripts_ that tell Luna which commands to run. Here, `p.eval()`
runs a command on the recording held in `p`. Running a command will
generally generate a set of results, manipulate the in-memory
recording, or write new EDF/annotation outputs to disk.

After running the above, we can look at these results, for example to see the result tables
returned:
```python
p.strata()
```
```
  Command Strata
0    DESC  ANNOT
1    DESC     BL
2    DESC     CH
```

To see the actual table (for the baseline, or `BL` level):
```python
p.table( 'DESC' )
```
```
       ID EDF_TYPE  NA  NS NSEGS REC_DUR_HMS START_TIME
0  nsrr01      EDF  10  14     1    11:22:00   21:58:17
```
or
```python
p.table( 'DESC' , 'CH' )
```
```
        ID        CH PDIM     SR
0   nsrr01  ABDO_RES    .   10.0
1   nsrr01   AIRFLOW    .   10.0
2   nsrr01       ECG   mV  250.0
3   nsrr01       EEG   uV  125.0
4   nsrr01   EEG_sec   uV  125.0
5   nsrr01       EMG   uV  125.0
6   nsrr01     EOG_L   uV   50.0
7   nsrr01     EOG_R   uV   50.0
8   nsrr01     LIGHT    .    1.0
9   nsrr01   OX_STAT    .    1.0
10  nsrr01  POSITION    .    1.0
11  nsrr01        PR  BPM    1.0
12  nsrr01      SaO2    %    1.0
13  nsrr01  THOR_RES    .   10.0
```

Whereas here we've run only a single recording, below we will use
`proj.proc()` to run commands across all recordings and retrieve those
tables in Python.


## Label remapping and sanitization

Luna makes labels easier to use in analysis strings: spaces normally
become underscores, and punctuation in signal names is sanitized. For
example, the source signals `EEG(sec)` and `THOR RES` appear as
`EEG_sec` and `THOR_RES`. Standard sleep-stage names are also mapped to
`W`, `N1`, `N2`, `N3`, and `R`.

To see which annotation names to use in your analysis, ask Luna to
list them:

```python
p.annots()
```

```text
         Annotations
         -----------
             Arousal
            Hypopnea
                  N1
                  N2
                  N3
   Obstructive_Apnea
                   R
       SpO2_artifact
   SpO2_desaturation
                   W
```

(Note: we could also have looked at `p.table( "DESC","ANNOT")` having
run the previous command to get this list, along with some other
information: this just provides a shortcut.)

Use these names exactly, because labels are **case-sensitive**. Luna
replaces spaces with underscores by default, so the names here include
`Arousal` and `Obstructive_Apnea`.

## Signal summary statistics

We can now calculate basic signal statistics across the project, using the [`STATS` command](../../ref/summaries/#stats).
`proj.proc()` runs the command on all three recordings, and `result.strata()` lists the tables it produced:

```python
result = proj.proc( "STATS sig=ECG,EMG,SaO2" )
result.strata()
```
```
  Command Strata
    STATS     CH
```
The `CH` table contains the results for each channel. Retrieve it as
a pandas data frame called `stats`:

```python
stats = result.table("STATS", "CH")
stats
```
```
       ID    CH       KURT        MAX       MEAN        MIN        P01  
   nsrr01   ECG  13.984303       1.25   0.009396  -1.240196  -1.240196   
   nsrr02   ECG   8.922235       1.25   0.005671  -1.240196  -0.779412   
   nsrr03   ECG  11.165007       1.25   0.003571  -1.240196  -1.240196   
...
... (output truncated)
...
```

To compare the channel means across recordings, use pandas to arrange
the channels in columns:
```python
means = stats.pivot(index="ID", columns="CH", values="MEAN").astype(float)
means.round(3)
```

```text
CH        ECG    EMG    SaO2
ID
nsrr01  0.009 -6.856  76.924
nsrr02  0.006 -0.610  77.873
nsrr03  0.004  3.014  65.083
```

`STATS` calculates statistics such as mean, minimum, maximum, standard
deviation, and root mean square (RMS). The `CH` table has one row per
individual and channel; `ID` identifies the recording and `CH` the
channel.

!!! info "Whole-night statistics" 
    Note that these are whole-recording summaries, including wake and bad
    data. In particular, the low overnight mean oxygen saturations here
    reflect unfiltered signal values, including invalid periods; they are
    not a quality-controlled clinical summary. Later examples restrict
    sleep stages and remove likely artifacts.

Luna keeps the results from the previous analysis internally, meaning that 
later commands can replace them. Assigning a table to a variable such
as `stats` gives you a pandas data frame you can keep using after the
next analysis. That is: retrieve any tables you want to keep before running
another Luna command.

## Working with annotations

As an example of working with interval-level _annotations_, here we
fetch individual obstructive-apnea events for the first recording:

```python
p = proj.inst("nsrr01")
apneas = p.fetch_annots("Obstructive_Apnea")
apneas.head(3)
```

```text
               Class   Start    Stop
   Obstructive_Apnea  2300.7  2310.7
   Obstructive_Apnea  2329.6  2343.1
   Obstructive_Apnea  4933.6  4953.2
```

`Start` and `Stop` are seconds from the recording start. The XML files
are read directly when the recording is opened; you do not need to
convert them first. `p.fetch_fulls_annots("Obstructive_Apnea")` also
returns event instance labels, channels, and metadata.

To count obstructive apneas and their cumulative duration across all three individuals,
we can use the Luna [`ANNOTS` command](../../ref/annotations/#annots):

```python
result = proj.proc("ANNOTS")
counts = result.table("ANNOTS", "ANNOT")
```
From the resulting table `counts`, which has information for all annotations,
we can use basic Python commands to extract only the relevant rows and columns:
```python
apnea_counts = counts[counts["ANNOT"] == "Obstructive_Apnea"]
apnea_counts[["ID", "COUNT", "DUR"]]
```
```text
        ID COUNT     DUR
    nsrr01    37   824.5
    nsrr02     5    67.7
    nsrr03   163  3795.6
```

`COUNT` is the number of events; `DUR` is their total duration in
seconds.

To count only events overlapping REM sleep, here we use the [`MASK` command](../../ref/masks/#mask) to first divide each
recording into 30-second epochs, then exclude epochs without the `R` annotation, before running `ANNOTS`:

```python
result = proj.proc("MASK ifnot=R & ANNOTS")
rem_counts = result.table("ANNOTS", "ANNOT")
rem_apneas = rem_counts[rem_counts["ANNOT"] == "Obstructive_Apnea"]
rem_apneas[["ID", "COUNT", "DUR"]]
```
```text
       ID COUNT    DUR
   nsrr01    27  668.4
   nsrr02     3   43.4
```

This only emits two rows, as the third individual (`nsrr03`) does not
have any scored REM sleep.

By default, an event counts if it has any overlap with a retained
epoch. To count only events whose __start__ lies in REM, use the
`start` option:

```python
result = proj.proc("MASK ifnot=R & ANNOTS start")
rem_starts = result.table("ANNOTS", "ANNOT")
rem_apnea_starts = rem_starts[rem_starts["ANNOT"] == "Obstructive_Apnea"]
rem_apnea_starts[["ID", "COUNT"]]
```
```text
       ID COUNT
   nsrr01    26
   nsrr02     2
```

## Putting it together

So far, we have summarized whole recordings and counted events during
REM. Now we will ask how EEG amplitude differs between N1, N2, N3, and
REM sleep. For each person, we will select one stage at a time and
calculate its EEG RMS, giving us four stage-specific measurements to
compare.

A [Luna script](../../luna/args/) describes a sequence of analysis steps. Alongside
commands such as `MASK` and `STATS`, Luna supports variables for settings,
`LOOP` / `END-LOOP` to repeat steps, and `IF` / `FI` to run steps
conditionally. These features let us write an analysis once and reuse
it with different selections or settings.

Here we will use a loop whose variable, `stage`, takes the values `N1`,
`N2`, `N3`, and `R`. Within the loop, `#{stage}` refers to its current
value. We first set the Luna variable `s` to `EEG`, then use `${s}` in
the script wherever we want that signal. We pass the complete script to
`proj.silent_proc()`, which runs it for each recording without displaying
the processing log:

```python
proj.vars("s", "EEG")

result = proj.silent_proc("""
    EPOCH len=30
    FREEZE full_night
    LOOP index=stage vals=N1,N2,N3,R
        THAW full_night
        TAG tag=STAGE/#{stage}
        % Keep only the current stage before calculating signal statistics
        MASK ifnot=#{stage}
        RE
        STATS sig=${s}
    END-LOOP
""")
stage_stats = result.table("STATS", "CH_STAGE")
```

Within a Luna script, `%` starts a comment, where Python would use `#`.

The commands work together as follows:

1. `EPOCH` divides the recording into 30-second epochs. `FREEZE` saves
   this full recording in memory under the name `full_night`.

2. `LOOP` repeats the enclosed steps for each stage. Each repetition
   starts with `THAW`, restoring the full night before making a new
   selection.

3. `TAG` labels the results with the current stage. For N2, for example,
   `TAG tag=STAGE/#{stage}` becomes `TAG tag=STAGE/N2`.

4. `MASK ifnot=#{stage}` excludes epochs without that stage annotation.
   `RE` then removes those epochs from the in-memory recording, so the
   next calculation uses only the selected stage.

5. `STATS sig=${s}` uses the signal named by `s`, here `EEG`. Note that Luna
   substitutes ordinary variables using `${x}` and loop variables using
   `#{x}`. The loop then moves to the next stage and repeats the process.

Here we need `RE` because the whole-signal `RMS` from `STATS` includes
all samples still in the recording, even those in masked epochs.
The `epoch` option respects masks for per-epoch results, but does not
make the whole-signal `RMS` stage-specific.

The stage label added by `TAG` gives us a `CH_STAGE` table: each row
identifies a person, channel, and stage. We can put the stages in columns
to compare their RMS values:

```python
rms = stage_stats.pivot(index=["ID", "CH"], columns="STAGE", values="RMS")
rms = rms.astype(float)
rms.round(3)
```

```text
STAGE           N1      N2      N3       R
ID     CH
nsrr01 EEG   7.355  10.646  13.015   7.564
nsrr02 EEG  10.362  14.742  20.055  14.146
nsrr03 EEG  12.302  14.497  18.980     NaN
```

The larger RMS in N3 is consistent with higher-amplitude slow waves.
The missing value (`NaN`) for `nsrr03` reflects its lack of REM sleep.
These signals have not yet been filtered or screened for artifacts;
we will do that in the deeper-dive tutorial. The source EDFs are
unchanged by this analysis.

To save the results:

```python
stage_stats.to_csv("python-stage-stats.csv", index=False)
```

Continue to [Take two in Python](python2.md) for more on sample lists,
analysis strings, result tables, and parameter files.
