# Quick Start

After [downloading](../download/index.md) and installing Luna, this
tutorial should provide a good starting point to familiarize yourself
with working with Luna.  There are four sections that should be
followed in order. This first section jumps straight in, demonstrating
a handful of commands quickly. The [second section](tut2.md) loops
back over the same material, but aiming to give more context and
detail.  The [third section](tut3.md) extends the range of commands
towards some more genuinely useful analyses of the sleep EEG.  The
[fourth section](tut4.md) performs the same steps but using
[_lunaR_](../ext/R/index.md) instead of [_lunaC_](../luna/args.md).
The final section then retraces many of these steps using the interactive
[_Moonlight_](../moonlight.md) tool. 

!!! note "Data used in this tutorial"
    This tutorial, based on
    [tutorials](https://sleepdata.org/tools/ruby-script-tutorial-01)
    at the National Sleep Research Resource
    [http://sleepdata.org](http://sleepdata.org), involves looking at
    three polysomnograms.  Each individual has an EDF (containing
    signal data, including EEG, ECG and EMG) and also an 
    annotation file, which includes information on manual sleep staging, 
    recorded apneas, hypopnea, movements and other events.  


Across these three tutorials, we will:

 - summarize the contents of the EDFs and annotation files
 - calculate signal statistics
 - enumerate types of annotated events
 - manipulate signals by masking and exporting data
 - apply automated artifact detection for sleep EEG 
 - apply spectral and spindle analyses for sleep EEG

 
## Obtaining the data

As mentioned, we'll follow the
[tutorials](https://sleepdata.org/tools/ruby-script-tutorial-01) at
the [National Sleep Research Resource](http://sleepdata.org), which
are based on three EDFs. 

If you are using the [Docker](../download/docker.md) version of Luna,
you'll already have the tutorial EDFs pre-installed.  If not, you'll
need to grab them from the web, from the link below:

| ZIP archive (67Mb) containing 3 EDFs, XML annotation files and 'sample list' |
|----|
| <http://zzz.bwh.harvard.edu/dist/luna/tutorial.zip> |

After downloading, move the ZIP file to your working directory, and
using the terminal, unzip the contents:

```
unzip tutorial.zip 
```

This should create a folder named `tutorial` which contains: a folder named
`edfs` with six files (`learn-nsrr01.edf`, `learn-nsrr01-profusion.xml`, etc), a second
folder named `cmd` (which contains some text files we'll use later in
this tutorial, given here purely to save you copying and pasting or
typing things in) and a _sample-list_ file named `s.lst` which defines
a _project_ for these three individuals:

```
cat s.lst
```

```
nsrr01 edfs/learn-nsrr01.edf	edfs/learn-nsrr01-profusion.xml
nsrr02 edfs/learn-nsrr02.edf	edfs/learn-nsrr02-profusion.xml 
nsrr03 edfs/learn-nsrr03.edf	edfs/learn-nsrr03-profusion.xml
```

!!! hint "Note to Windows users"
    If you are using a Windows command prompt, you may substitute
    `type` for `cat` and `more` for `less`. Some other command-line functions 
    may not be available however. For example, instead of `unzip` you may need to use the GUI, etc.
    We do stress that, as a command-line program, _lunaC_ is fundamentally 
    better suited to macOS and Linux platforms.   The experience with _lunaR_ should be
    similar across platforms however. 


!!! hint "Using `--build` to generate new sample lists automatically"

    If the file `s.lst` didn't exist, you could use the [`--build`](../luna/args.md#-build-option) command to generate it.  Assuming you are in the folder that contains `edfs/`:
    ```
    luna --build edfs > s2.lst
    ```
    This will generate a sample list, with IDs based on file names (the actual EDFs do not contain IDs):
    ```
    learn-nsrr01	edfs/learn-nsrr01.edf
    learn-nsrr02	edfs/learn-nsrr02.edf
    learn-nsrr03	edfs/learn-nsrr03.edf
    ```
    To automatically match each EDF to the correspinding annotation file (that ends with `-profusion.xml` instead of `.edf` appended to the filename) we can run:
    ```
    luna --build edfs -ext=-profusion.xml > s2.lst
    ```
    Luna lets us know what it has found: 
    ```
    wrote 3 EDFs to the sample list
      3 of which had 1 linked annotation files
    ```
    The new sample list
    ```
    learn-nsrr01   edfs/learn-nsrr01.edf	edfs/learn-nsrr01-profusion.xml
    learn-nsrr02   edfs/learn-nsrr02.edf	edfs/learn-nsrr02-profusion.xml
    learn-nsrr03   edfs/learn-nsrr03.edf	edfs/learn-nsrr03-profusion.xml
    ```
    (note: the supplied sample list, `s.lst` wasn't build with `--build` and has the shorter IDs, e.g. `nsrr01`)

## Displaying EDF files

To test that Luna is properly installed, and that the EDFs
downloaded correctly, run the following to apply the `DESC` _command_
to each EDF specified in the `s.lst` _project_:

```
luna s.lst -s DESC > res.txt
```
This should write output to a file called `res.txt`, as well as some
information (the _log_) to the console/terminal, as follows:

```

===================================================================
+++ luna | v0.28.0, 14-Mar-2022 | starting 25-Mar-2023 08:20:29 +++
===================================================================
input(s): s.lst
output  : .
commands: c1	DESC	

___________________________________________________________________
Processing: nsrr01 [ #1 ]
 duration: 11.22.00 | 40920 secs | clocktime 21.58.17 - 09.20.17

 signals: 14 (of 14) selected in a standard EDF file
  SaO2 | PR | EEG_sec | ECG | EMG | EOG_L | EOG_R | EEG
  AIRFLOW | THOR_RES | ABDO_RES | POSITION | LIGHT | OX_STAT

 annotations:
  Arousal (x194) | Hypopnea (x361) | N1 (x109) | N2 (x523)
  N3 (x17) | Obstructive_Apnea (x37) | R (x238) | SpO2_artifact (x59)
  SpO2_desaturation (x254) | W (x477)

 variables:
  airflow=AIRFLOW | ecg=ECG | eeg=EEG_sec,EEG | effort=THOR_RES,A...
  emg=EMG | eog=EOG_L,EOG_R | hr=PR | id=nsrr01 | light=LIGHT
  oxygen=SaO2,OX_STAT | position=POSITION
 ..................................................................
 CMD #1: DESC
   options: sig=*

... (cont'd) ...

___________________________________________________________________
...processed 3 EDFs, done.
...processed 1 command set(s),  all of which passed
-------------------------------------------------------------------
+++ luna | finishing 25-Mar-2023 08:20:29                       +++
===================================================================
```

As well as summarizing headers, Luna validates the structure of each
EDF, for example, checking whether it is the expected size.  Viewing
the text file `res.txt` (i.e. with a text editor, or command such as
`cat` or `less`), we see the description generated by the `DESC`
command:

```
cat res.txt
```
```
EDF filename      : edfs/learn-nsrr01.edf
ID                : nsrr01
Clock time        : 21.58.17 - 09.20.17
Duration          : 11:22:00  40920 sec
# signals         : 14
Signals           : SaO2[1] PR[1] EEG_sec[125] ECG[250] EMG[125] EOG_L[50]
                    EOG_R[50] EEG[125] AIRFLOW[10] THOR_RES[10] ABDO_RES[10] 
                    POSITION[1] LIGHT[1] OX_STAT[1]

EDF filename      : edfs/learn-nsrr02.edf
ID                : nsrr02
Clock time        : 21.18.06 - 07.15.36
Duration          : 09:57:30  35850 sec
# signals         : 14
Signals           : SaO2[1] PR[1] EEG_sec[125] ECG[250] EMG[125] EOG_L[50]
                    EOG_R[50] EEG[125] AIRFLOW[10] THOR_RES[10] ABDO_RES[10]
                    POSITION[1] LIGHT[1] OX_STAT[1]

EDF filename      : edfs/learn-nsrr03.edf
ID                : nsrr03
Clock time        : 20.15.00 - 07.37.00
Duration          : 11:22:00  40920 sec
# signals         : 14
Signals           : SaO2[1] PR[1] EEG_sec[125] ECG[250] EMG[125] EOG_L[50]
                    EOG_R[50] EEG[125] AIRFLOW[10] THOR_RES[10] ABDO_RES[10]
                    POSITION[1] LIGHT[1] OX_STAT[1]

```

In other words, each EDF contains 14 signals, spanning approximately
10 or 11 hours of sleep.  Much of the output of `DESC` mirrors what is
logged to the console when running _any_ Luna command, just in a
slightly different format.

!!! note "Sample list IDs versus EDF header Patient IDs" 
    Note that the
    IDs (`nsrr01`, `nsrr02` and `nsrr03`) are those specified in the
    first column of the `s.lst` file.  Although EDF headers contain a
    field corresponding to _Patient ID_, this is **not** used
    internally by Luna.  The EDF header _Patient ID_ (which can be
    viewed with the [`SUMMARY`](../ref/summaries.md#summary) command)
    is generally ignored by Luna, and can be missing or different from
    the ID specified in the first column of the sample list.  (When
    running without a sample list, the `ID` is simply the filename.)


## Label remapping and sanitization

The labels for channels and annotations that Luna reports above are
actually not the original ones in the EDF/XML files.  Rather, they (by
default) go through a process of remapping and sanitization to make
labels that are more consistent (for NSRR data, at least) and easier
to work with.

!!! note "Data harmonization in the NSRR"
    To get a sense of why harmonization is important for NSRR studies, see the first couple of sections of 
    [this post](https://gitlab-scm.partners.org/zzz-public/nsrr/-/blob/master/common/harm-principles.md).

Specifically, Luna (by default) will:

 - try to identify common _sleep stage annotations_ and change them to a consistent format,
 e.g. `Sleep Stage 1` becomes `N1`.  This behavior can be turned off with the
  `annot-remap=F` flag.  (Note, prior to v0.28, Luna used to convert a greater number of
  terms automatically, e.g. for arousals; for transparency of processing, these
  steps must now be done explicitly, by adding `nsrr-remap=T`, see below).
 
 - replace all spaces in channel and annotation names with an
   underscore (e.g. `THOR RES` to `THOR_RES`); this behavior can be
   turned off with the `keep-spaces=T` flag

 - in addition to the prior point, Luna will also _sanitize_ special
   characters in labels that are likely to cause problems downstream,
   such as `-` or `*`, replacing these with underscores also; this
   behavior can be turned off with the `sanitize=F` flag

Running the same command but with these flags set (and also just running for the first individual in the sample list):

```
luna s.lst 1 sanitize=F keep-spaces=T -s DESC 
```
The console now prints the "original" labels:
```
 signals: 14 (of 14) selected in a standard EDF file
  SaO2 | PR | EEG(sec) | ECG | EMG | EOG(L) | EOG(R) | EEG
  AIRFLOW | THOR RES | ABDO RES | POSITION | LIGHT | OX STAT

 annotations:
  Arousal () (x194) | Hypopnea (x361) | N1 (x109) | N2 (x523)
  N3 (x17) | Obstructive Apnea (x37) | R (x238) | SpO2 artifact (x59)
  SpO2 desaturation (x254) | W (x477)
```
Running with the `nsrr-remap=T` option changes the annotations to these values:
```
 annotations:
  N1 (x109) | N2 (x523) | N3 (x17) | R (x238)
  W (x477) | apnea:obstructive (x37) | arousal (x194) | artifact:SpO2 (x59)
  desat (x254) | hypopnea (x361)
```

For example:

 - `Obstructive Apnea` becomes `apnea:obstructive` (from `nsrr-remap=T`)
 - as before, `NREM2` becomes `N2` (by default, unless `annot-remap=F`)
 - `THOR RES` becomes `THOR_RES` (unless `keep-spaces=T`)
 - `EOG(L)` becomes `EOG_L_` (by `sanitize=T`)

Making these changes can be useful as similar to R or Matlab, Luna
is fundamentally a command-line tool and so uses text input to specify
operations. As such, having spaces or special characters creates many
unnecessary problems (e.g. for commands such as
[`TRANS`](../ref/evals.md#trans) that can perform general arithmetic
operations on signals, which would make the expression `C3-M2`
ambiguous). See this
[FAQ](../faq.md#spaces-and-special-characters-in-labels) for the
rationale for making labels more _machine-readable_.


## Signal summary statistics

Turning to the signals contained in each EDF, here we use the `STATS`
command to generate basic statistics (mean, median, min, max and standard
deviation) per channel, following
[this](https://sleepdata.org/tools/ruby-script-tutorial-04) NSRR
tutorial.

```
luna s.lst -s STATS
```
```
nsrr01     STATS     CH/SaO2     .     MEAN     76.9242
nsrr01     STATS     CH/SaO2     .     MAX      99.1196
nsrr01     STATS     CH/SaO2     .     MIN      0.10071
nsrr01     STATS     CH/SaO2     .     SKEW    -1.54459
nsrr01     STATS     CH/SaO2     .     KURT     0.424721
nsrr01     STATS     CH/SaO2     .     RMS      85.5665
nsrr01     STATS     CH/SaO2     .     SD       37.4744
... (cont'd)
```

For example, for the first individual `nsrr01`, the `SaO2` channel has
a mean of 76.9242 and a standard deviation of 37.4744.  This output is
formatted in a standardized manner, described
[below](tut2.md#output-format), but it is quite verbose and not easily
readable.  In practice, Luna is designed to work with a companion
tool, [`destrat`](../luna/args.md#output), which records and presents
Luna output in a more structured way.  For example, here we re-run the
command, except now saving results to a database `res.db` via the `-o`
flag:

```
luna s.lst -o res.db -s STATS
```

and then use [`destrat`](../luna/args.md#output) to extract, for example, only the
signal means for the ECG, EMG and SaO2 channels (don't worry about the details of this command for now)::

```
destrat res.db +STATS -c CH/ECG,EMG,SaO2 -p 3 -v MEAN | behead
```
```
                       ID   nsrr01              
              MEAN.CH.ECG   0.009               
              MEAN.CH.EMG   -6.856              
             MEAN.CH.SaO2   76.924              

                       ID   nsrr02              
              MEAN.CH.ECG   0.006               
              MEAN.CH.EMG   -0.610              
             MEAN.CH.SaO2   77.873              

                       ID   nsrr03              
              MEAN.CH.ECG   0.004               
              MEAN.CH.EMG   3.014               
             MEAN.CH.SaO2   65.083              
```

More complicated than it needs to be?  For this simple example,
certainly.  However, the value of _destrat_ and its _stratified
output_ (called [_lunout_](../luna/destrat.md)) databases will become more apparent when
working with larger and more complex result sets.  That is, in real
analyses results may be stratified by multiple factors (such as
channel, sleep stage, frequency or power band, epoch, event or class
of annotation) and reside across multiple output databases.
Later in this tutorial, we'll use _destrat_  to handle these situations.

In any case, comparing these results to the NSRR
[tutorial](https://sleepdata.org/tools/ruby-script-tutorial-04),
encouragingly we see similar estimates for these quantities.

## Working with annotations 

Parallel to
[this](https://sleepdata.org/tools/ruby-script-tutorial-05) NSRR
tutorial, here we use Luna to summarize the contents of an
[XML](https://en.wikipedia.org/wiki/XML) annotation file, which are
structured as Compumedics Profusion files (described 
[here](https://github.com/nsrr/edf-editor-translator/wiki/Compumedics-Annotation-Format)).

To take a quick look at the events in an annotation file, the special
`--xml` command displays the time and duration (in seconds) of each
event/annotation, along with the _type_ of annotation (if present) and
its _name_:

```
luna --xml edfs/learn-nsrr01-profusion.xml
```
```
.               .               EpochLength     30
0 - 30          (30 secs)       SleepStage      wake
30 - 60         (30 secs)       SleepStage      wake
60 - 90         (30 secs)       SleepStage      wake
90 - 120        (30 secs)       SleepStage      wake
120 - 150       (30 secs)       SleepStage      wake
150 - 180       (30 secs)       SleepStage      wake
180 - 210       (30 secs)       SleepStage      wake
210 - 240       (30 secs)       SleepStage      wake
240 - 270       (30 secs)       SleepStage      wake

... (cont'd)

2700 - 2730     (30 secs)       SleepStage      NREM2
2711.8 - 2718   (6.2 secs)      .               Arousal ()      
2720.1 - 2739.3 (19.2 secs)     .               Hypopnea        
2730 - 2760     (30 secs)       SleepStage      NREM2
2747.2 - 2751.5 (4.3 secs)      .               Arousal ()      
2750.1 - 2769.3 (19.2 secs)     .               SpO2 desaturation       
2752.6 - 2777.4 (24.8 secs)     .               Hypopnea        
2760 - 2790     (30 secs)       SleepStage      NREM2
2779 - 2784     (5 secs)        .               Arousal ()      
2782.6 - 2807.4 (24.8 secs)     .               SpO2 desaturation       
2790 - 2820     (30 secs)       SleepStage      NREM2

... (cont'd)
```

As shown below, the information in these [XML
files](../ref/annotations.md#nsrr-xml-files) can be used directly to
extract or exclude epochs, via the [`MASK`](../ref/masks.md#mask)
command.  Although the NSRR uses this XML format extensively for
annotations, Luna accepts other, simpler formats too, as described
[here](../ref/annotations.md#annot-files).

The [`ANNOTS`](../ref/annotations.md#annots) command summarizes the
number and duration (in seconds) of the annotations in one or more
annotation files associated with an EDF.  As an example, here we use
it to count the number of obstructive apneas for each individual.
Whereas this can be easily done from the output of the `--xml`
command, one advantage of `ANNOTS` is that other _masks_ can
be applied: for example, to count only apneas occurring during REM
sleep.  Running `ANNOTS` and sending output to `annot.db`:

```
luna s.lst -o annot.db -s ANNOTS
```
we can then view a summary of the output generated: 

```
destrat annot.db
```
```
--------------------------------------------------------------------------------
annot.db: 1 command(s), 3 individual(s), 5 variable(s), 28345 values
--------------------------------------------------------------------------------
  command #1:	c1	Thu Aug 13 13:23:28 2020	ANNOTS	sig=*
--------------------------------------------------------------------------------
distinct strata group(s):
  commands      : factors           : levels        : variables 
----------------:-------------------:---------------:---------------------------
  [ANNOTS]      : ANNOT             : 11 level(s)   : COUNT DUR
                :                   :               : 
  [ANNOTS]      : ANNOT INST        : 11 level(s)   : COUNT DUR
                :                   :               : 
  [ANNOTS]      : ANNOT INST T      : (...)         : CH START START_ELAPSED_HMS START_HMS
                :                   :               : STOP STOP_ELAPSED_HMS STOP_HMS
                :                   :               : VAL
                :                   :               : 
----------------:-------------------:---------------:---------------------------
```

See [`ANNOTS`](../ref/annotations.md#annots) for a description of the
output _strata_ from this command.  As an example, to get the count
(`COUNT`) of obstructive apnea events (`apnea/obstructive` level of
the `ANNOT` factor):

```
destrat annot.db +ANNOTS -r ANNOT/Obstructive_Apnea -v COUNT
```
```
ID         ANNOT                 COUNT
nsrr01     Obstructive_Apnea     37
nsrr02     Obstructive_Apnea     5
nsrr03     Obstructive_Apnea     163
```

To count only apneas that occur during REM sleep, we can _epoch_ the
dataset (using the [`EPOCH`](../ref/epochs.md#epoch) command) and then
add a _mask_ (the [`MASK`](../ref/masks.md#mask) command) based on the
staging information in the XML:  

```
luna s.lst -o annot.db -s 'EPOCH & MASK ifnot=R & ANNOTS'
```

!!! note "Specifying multiple commands after `-s`" 
    Note how in the
    example above, we string together multiple commands, each
    separated by the `&` character.  We've put the entire _script_
    following `-s` now in quotes (this stops special characters such
    as `&` being interpreted incorrectly by the operating system's
    terminal/shell.

Using _destrat_ to extract the `COUNT` variable once more:

```
destrat annot.db +ANNOTS -r ANNOT/Obstructive_Apnea -v COUNT
```

we obtain the number of events during REM: 

```
ID      ANNOT                   COUNT
nsrr01  Obstructive_Apnea       27
nsrr02  Obstructive_Apnea       3
```

There is no output for the last individual `nsrr03`, as he or she did
not have any REM epochs.

By default, the `ANNOTS` command will include all annotations with
_any_ overlap with a REM epoch, in this example.  To include only
events that _start_ during a REM epoch, add the `start` option:

```
luna s.lst -o annot.db -s 'EPOCH & MASK ifnot=R & ANNOTS start'
```

```
destrat annot.db +ANNOTS -r ANNOT/Obstructive_Apnea -v COUNT
```
```
ID      ANNOT                   COUNT
nsrr01  Obstructive_Apnea       26
nsrr02  Obstructive_Apnea       2
```

## Putting it together

We'll end this first section by combining the `STATS` command with the
annotations from the XML, to generate stage-specific summary
statistics for the `EEG` channel.  Here, instead of using `-s`, we'll
give Luna a multi-part set of commands as a separate plain-text file (this 
is in the `cmd` folder of the tutorial, called `first.txt`).
This command file below introduces a number of new commands (text after
the `%` character is a comment and ignored by Luna):

``` 
% Set epoch duration
% (anything following '%' is a comment)

EPOCH len=30

% Assign a label ('tag') in the output,
% which will be set to the value of the 'stage' variable

TAG tag=STAGE/${stage}

% Restrict to epochs that match the ${stage} variable
% i.e. MASK them out if they do *not* match

MASK ifnot=${stage}

% Having set a mask above, now actually remove the masked epochs

RESTRUCTURE

% Produce basic statistics on this reduced dataset

STATS sig=EEG
```

First, `EPOCH` specifies that non-overlapping, 30-second epochs
should be applied to each EDF.  We then set a `TAG`, which helps
keep track of the output, as we'll see below.  The value of the tag
here takes a special form: _level_ / _factor_ where _factor_
indicates sleep stage.  Rather than being hard-coded in the file, the
level is specified as a _variable_, in the form `${variable}`.  A
specific value for `${stage}` is given on the command line when Luna
is invoked, as below.  The next command _masks_ (that is, excludes)
any epoch which _does not_ have an annotation that matches
`${stage}`.  That is, if `${stage}` is `N2`, then only N2
epochs are included in the analysis.  After setting mask values for
one or more epochs, the `RESTRUCTURE` command removes any masked
epochs. Finally, the `STATS` command is invoked, but this time it
will consider only epochs that match the specified sleep stage.

The `first.txt` _script_ is given as the input to Luna (i.e. using
the _standard input_ redirection operator, `<` ).  We specify that
the output should go to a database called `out.db`, by virtue of the
`-o` option.  We set define the value of the _stage_ variable
(here as `N1`), which will be expected when processing the
`TAG` and `MASK` commands. Finally, we set a special variable,
`sig`, which instructs Luna to only consider the first EEG
channel (labeled `EEG`, as indicated by the `DESC` command):

```
luna s.lst -o out.db stage=N1 sig=EEG < cmd/first.txt
```

We now run Luna three more times, for the remaining sleep
stages. However, instead of using the `-o` flag (which always creates
a new database), we'll use `-a` which _appends_ output to an existing
database.  In this way, the `rms.db` database will accumulate all output.
```
luna s.lst -a out.db stage=N2 sig=EEG < cmd/first.txt
```
```
luna s.lst -a out.db stage=N3 sig=EEG < cmd/first.txt
```
```
luna s.lst -a out.db stage=R sig=EEG  < cmd/first.txt
```

To view the contents of `out.db`, we run `destrat`:

```
destrat out.db
```

```
--------------------------------------------------------------------------------
out.db: 20 command(s), 3 individual(s), 19 variable(s), 222 values
--------------------------------------------------------------------------------
  command #1:	c1	Mon Mar 18 15:54:27 2019	EPOCH	
  command #2:	c2	Mon Mar 18 15:54:27 2019	TAG	
  command #3:	c3	Mon Mar 18 15:54:27 2019	MASK	
  command #4:	c4	Mon Mar 18 15:54:27 2019	RESTRUCTURE	

 ... cont'd ...

--------------------------------------------------------------------------------
distinct strata group(s):
  commands      : factors           : levels        : variables 
----------------:-------------------:---------------:---------------------------
  [EPOCH]       : .                 : 1 level(s)    : DUR INC NE
                :                   :               : 
  [MASK]        : STAGE EMASK       : 4 level(s)    : N_MASK_SET N_MASK_UNSET 
                :                   :               : N_MATCHES N_RETAINED 
                :                   :               : N_TOTAL  N_UNCHANGED
                :                   :               : 
  [RESTRUCTURE] : STAGE             : 4 level(s)    : DUR1 DUR2 NA NR1 NR2 NS
                :                   :               : 
  [STATS]       : CH STAGE          : 4 level(s)    : MAX MEAN MIN P01 P02 P05 
                :                   :               : P10 P20 P30 P40 P50 P60 
                :                   :               : P70 P80 P90 P95 P98 P99 
                :                   :               : RMS SD SKEW
                :                   :               : 
----------------:-------------------:---------------:---------------------------
```

which lists the number of commands, individuals and variables in the
database, a list of the commands and their time-stamps, and the
_strata_ groups in this database.

The `TAG` command specified that a user-defined stratum, called
`STAGE`, be added to the output.  This was set to either `N1`,
`N2`, etc, corresponding to the sleep stage values in the XML
file.  There are therefore 4 _levels_ for the _factor_
`STAGE`. We can view the RMS statistics, which are grouped by
channel (`CH`) and sleep stage (`STAGE`), as follows:

```
destrat out.db +STATS -r CH -c STAGE -v RMS -p 3
```
```
ID     CH  RMS.STAGE_N1    RMS.STAGE_N2    RMS.STAGE_N3    RMS.STAGE_R
nsrr01 EEG 7.355           10.646          13.250          7.564
nsrr02 EEG 10.362          14.742          20.055          14.146
nsrr03 EEG 12.302          14.497          18.980          NA
```

The options for `destrat` are described [below](tut2.md#using_destrat)
more fully.  Briefly, here we select the variable RMS (`-v`) from the
`[STATS]` command, optionally formats numeric output to 3 decimal
places (`-p`), sets row strata to correspond to channels (`-r`) and
column strata to correspond to stages (`-c`).  


To view small output files, the [`behead`](../luna/args.md#behead)
utility is often useful: pipe the output of `destrat` into `behead` as
follows:

```
destrat out.db +STATS -r CH -c STAGE -v RMS -p 3 | behead
```
```
                       ID   nsrr01              
                       CH   EEG                 
             RMS.STAGE_N1   7.355               
             RMS.STAGE_N2   10.646              
             RMS.STAGE_N3   13.015              
              RMS.STAGE_R   7.564               

                       ID   nsrr02              
                       CH   EEG                 
             RMS.STAGE_N1   10.362              
             RMS.STAGE_N2   14.742              
             RMS.STAGE_N3   20.055              
              RMS.STAGE_R   14.146              

                       ID   nsrr03              
                       CH   EEG                 
             RMS.STAGE_N1   12.302              
             RMS.STAGE_N2   14.497              
             RMS.STAGE_N3   18.980              
              RMS.STAGE_R   NA                  
```

As another example of extracting output, we can get the number
of records in the EDF for each stage/individual, as this value is
output after any `RESTRUCTURE` command: `DUR2` is the duration in
seconds _after_ a given restructuring:

```
destrat out.db +RESTRUCTURE -c STAGE -v DUR2 | behead
```
```
                       ID   nsrr01              
            DUR2.STAGE_N1   3270                
            DUR2.STAGE_N2   15690               
            DUR2.STAGE_N3   510                 
             DUR2.STAGE_R   7140                

                       ID   nsrr02              
            DUR2.STAGE_N1   330                 
            DUR2.STAGE_N2   11970               
            DUR2.STAGE_N3   5550                
             DUR2.STAGE_R   3600                

                       ID   nsrr03              
            DUR2.STAGE_N1   1560                
            DUR2.STAGE_N2   11250               
            DUR2.STAGE_N3   630                 
             DUR2.STAGE_R   0                   
```

This explains the `NA` (not available) output for the `RMS` for the
third individual's REM sleep: they did not have any sleep scored as
REM that night.  If you examine the contents of the log output
(i.e. sent straight to the console when running the command, you'll
see this is noted here also:)

```
 CMD #3: MASK
 set masking mode to 'force'
 based on R 0 epochs match;  newly masked 1364 epochs, unmasked 0 and left 0 unchanged
 total of 0 of 1364 retained for analysis
```
 
Looking at the results for the stage-specific RMS analysis, even
without filtering and artifact detection of this signal, the results
seem plausible: we generally see higher RMS, corresponding to more
activity due to large amplitude slow waves during N3 sleep.  We
return to this theme when considering spectral analyses of the EEG,
below.

## Using data freezes

The above procedure can be further streamlined with the use of [data
freezes](../ref/freezes.md).  Rather than run a separate script for
each stage (N1, N2, N3 and R), we can make a _freeze_ (i.e. a
snapshot) of the original dataset, and use the same
[`MASK`](../ref/masks.md)/[`RE`](../ref/restructure.md) process to
focus on a given stage, but then use [`THAW`](../ref/freezes.md#thaw)
to return to the original (full) EDF.  Note that if running that same
[`STATS`](../ref/summaries.md#stats) (or indeed, any command) multiple
times in the same script, we need to use the [`TAG`](../ref/summaries.md#tag)
command to distinguish the four sets of outputs: 

```
% note: comments from the original cmd/first.txt removed here
EPOCH len=30

FREEZE F1
TAG tag=STAGE/N1
MASK ifnot=N1
RE
STATS sig=EEG

THAW F1
TAG tag=STAGE/N2
MASK ifnot=N2
RE
STATS sig=EEG

THAW F1
TAG tag=STAGE/N3
MASK ifnot=N3
RE
STATS sig=EEG

THAW F1
TAG tag=STAGE/R
MASK ifnot=R
RE
STATS sig=EEG
```

!!! hint "Using data freezes"
    Using the freeze/thaw mechanism is
    particularly useful when one needs to perform various
    pre-processing at the start of the script that is applied to the
    whole night's recording, e.g. filtering, resampling, etc, which
    can be done before the first freeze is made. In this way, we avoid
    repeating the same steps. (An alternative is to save intermediate
    files also.)

As an exercise, make a new version of `cmd/first.txt` with the above
changes, and call it `cmd/first-freezes.txt`.  This is then run with a
single command:

```
luna s.lst -o out2.db < cmd/first-freezes.txt
```

It is worth inspecting the console output to see the various freeze/thaw steps in action. For the first individual, we
first make a freeze of the whole dataset:
```
 CMD #2: FREEZE
   options: F1 sig=*
  freezing state, with tag F1
  copied 40920 records
  currently 1 freeze(s): F1
```
After restricting the dataset to N1, we return to the original freeze:
```
 CMD #7: THAW
   options: F1 sig=*
  thawing previous freeze F1
  old dataset   : 3270 records, 15 signals, 10 annotations
  thawed freeze : 40920 records, 14 signals, 10 annotations
  copied 40920 records
```
This indicates we went from 3270 records back to the full 40920 records, which matches the earlier output from `RE` for N1:
```
 CMD #5: RE
   options: sig=*
  restructuring as an EDF+:  retaining 3270 of 40920 records
  of 682 minutes, dropping 627.5, retaining 54.5
  resetting mask
  clearing any cached values and recording options
  retaining 109 epochs
```
That is, 3270 records (with each record having 1 second duration) equals 54.5 minutes or 109 30-second N1 epochs.
What about the above `THAW` output noting 15 versus 14 signals, given
the original EDF had only 14 signals and we did not explicitly add any
new channels?  This reflects that fact that after the `RE` command,
the internal representation of the data is now implicitly as an EDF+D,
i.e. with gaps, and so Luna automatically adds an `EDF Annotations`
channel to hold the time-track for each record.  This is dropped when
returning to the original, which is a standard, continuous EDF and so
does not contain a time-track.

The steps for the other stages and individuals follow a similar pattern.  Most importantly, 
this new output database should contain the same information as from the original set of four runs: i.e.
pulling the stage-specific RMS for each individual, as obtain the same table as above:
```
destrat out2.db +STATS -r CH -c STAGE -v RMS -p 3
```
```
ID      CH RMS.STAGE_N1  RMS.STAGE_N2  RMS.STAGE_N3  RMS.STAGE_R
nsrr01 EEG        7.355        10.646        13.015        7.564
nsrr02 EEG       10.362        14.742        20.055       14.146
nsrr03 EEG       12.302        14.497        18.980           NA
```

We can now move on to the [next part](tut2.md) of the tutorial.