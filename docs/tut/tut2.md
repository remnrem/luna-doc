
# Take Two

This is a slightly more technical section, in which we flesh out a
little more of the details surrounding some of the commands, formats
and functions used above.

## Sample-lists

The sample-list file (`s.lst` in this tutorial) is a _tab-delimited_
[plain-text](https://en.wikipedia.org/wiki/Plain_text) file, that
defines a _project_, that is, a set of EDFs and annotation files.
Sample-lists have one row per individual/EDF, with two or more
tab-delimited columns: ID, EDF file path, and optionally, one or more
annotation files or folders.  In this example, the `s.lst` file
specifies 3 EDFs and their corresponding annotation files (the XML
files).

Use a text editor (e.g. TextEdit on Mac, if not emacs or similar), **not
a word processor**, to generate or edit these files.  Here the sample
list has been automatically generated and you should not need to
change it if running the tutorial from the same directory that
contains this file.  Be aware that spaces will be interpreted as part
of the filename -- so make sure not to introduce any superfluous
whitespace or special characters, as it will mean that Luna may not be
able to locate your data.

A note on file naming conventions. For convenience, here the files are
referenced by their relative paths, but which means that you need to
run Luna from the folder containing this file.  If you want to run
Luna from another folder, you can set the [search path](../luna/args.md#search-paths).

!!! note
    We've used the `/` character to indicate folders here, which is the standard for Linux and Mac OS.

## Luna syntax

Luna expects the following arguments:

- always as the first argument, _EITHER_ a sample list
  (i.e. `s.lst` in this example), _OR_ a single EDF file (that
  ends `.edf` or `.EDF`)

- one or more _commands_, either via standard input or specified
  after the `-s` argument. If the `-s` is specified, it must be
  the final option given. That is, all other arguments are interpreted
  as Luna _commands_ rather than arguments.

- optionally, if the first argument is a sample list, then either one
  (_i_) or two (_i_ and _j_) integers, to specify that only the
  _i'th_ or the _i'th_ to the _j'th_ rows from the sample-list
  should be processed

- optionally, a series of variable assignments, (e.g. `sr=100` or
  `sig=EEG1`)

- optionally, a _parameter file_ prefixed with an _at sign_: for
  example, `@param.txt`.  A parameter file lists variable
  definitions in a plain-text file, with one variable name/value pair
  per line, separated by a tab character.  This is simply a
  convenience feature, and is equivalent to entering the same
  `variable=value` pairs on the command line.

- optionally, a _lout_ output database following the `-o` ( or `-a`)
  argument.  If it doesn't exist it will be created; with `-o` it will
  overwrite any existing data, whereas `-a` appends on any existing
  data.
 
There are a few exceptions: for example, some special commands
(e.g. `--xml` as illustrated above) do not require a sample list to be
specified.

## Luna commands

Luna _commands_ refer to a set of functions (summarized
[here](../ref/index.md)), which are primarily operations performed on
the EDFs.  Here we are using the term _command_ in a specific sense,
as distinct from the _arguments_ added on the command line when
running Luna, which were described in the previous section.

By default, Luna iterates through all individual EDFs in the
sample-list, applying the requested command(s). Commands can be
specified either via the command-line (`-s` option) or from a separate
file, directed into the _standard input_ of the Luna program (with the
`<` shell operation).

All command names should be UPPERCASE.  Multiple commands can be
separated by the ampersand character (`&`).  When commands are given via standard
input, a newline character also separates commands.  

Options that follow commands are typically lowercase, and are either
single keywords or `variable=value` pairs.  When submitting multiple
commands via the command line `-s` option, you will want to use quotes
to stop the `&` being interpreted as a shell directive. For example,
here we run two commands: `EPOCH` and `STATS`.

```
luna s.lst -s "EPOCH len=20 & STATS epoch" > res.txt
```

By default, all commands are assumed to come from _standard input_,
unless the `-s` is given (as above).  If the file `command.txt`
contains only the word `DESC`, then all four invocations of Luna
below are identical (although the first one is preferred):

```
luna s.lst < command.txt > results.txt 
```
```
cat command.txt | luna s.lst > results.txt 
```
```
echo "DESC" | luna s.lst > results.txt 
```
```
luna s.lst -s DESC > results.txt 
```

If you are not familiar with pipes and redirection, the following
[tutorial](http://ryanstutorials.net/linuxtutorial/piping.php) (or any
other web-search) will be helpful.

<a name="output"></a>
## Output format

By default, all output goes to _standard out_, i.e. the
console/terminal by default.  When outputting plain text (instead of
using a _lout_ database, see below), most Luna commands generate
output in a fixed format.  (The initial `DESC` command is in fact an
exception, as one of the few commands that generate a simple,
"human-readable" text file.)  The standard format comprises 6
tab-delimited columns, with one row per value:

```
 nsrr01	    STATS      CH/SaO2 .	  MIN	 0.10071
 nsrr01	    STATS      CH/SaO2 .	  MAX	 99.1196
 nsrr01	    STATS      CH/SaO2 .	  MEAN	 76.9242
```
The 6 columns represent

- individual identifier/ID from the sample-list file
- the Luna _command_ that is generating this output
- any stratifying factors that logically organize the output (e.g. by channel, or by sleep stage and frequency range).
- any temporal stratifying factors generated by Luna (e.g. epoch, time-point, event number)
- the variable name
- the value for that variable (for that individual, for that combination of stratifying factors)

If there are no stratifiers, columns 3 and 4 are set to empty(`.`).
Here the strata are defined by a single _factor_ (`CH` which
indicates the channel).  Strata for a single factor are represented as
`factor/level` where `level` is the name of a particular channel
(i.e. one of the 14 for these EDFs).  The fifth and sixth columns then
give the variable _name_ and _value_ for that person/strata
combination.  For example, the `MEAN` value for the `SaO2` channel
is `76.9242`, as given on the third row of the output.  This format
is verbose but designed to be easily parsed by standard text
processing tools: e.g. to see only the `MEAN` values, using the
ubiquitous [awk](https://www.gnu.org/software/gawk/manual/gawk.html#Getting-Started)
tool:

```
awk ' $5 == "MEAN" { print $1, $3, $6 } ' res.txt
```
```
nsrr01	 CH/SaO2	76.9242
nsrr01	 CH/PR		57.3485
nsrr01	 CH/EEG(sec)	1.18558
nsrr01	 CH/ECG		0.00939557
nsrr01	 CH/EMG		-6.8557
nsrr01	 CH/EOG(L)	0.472551
nsrr01	 CH/EOG(R)	0.321447
nsrr01	 CH/EEG		-0.301199
nsrr01	 CH/AIRFLOW	0.0664418
... (cont'd)
```

## Using `destrat`

If the `-o` argument is given to Luna, instead of the column-based
text output format described above, all output is sent to a _lout_
database, which is designed for handling _stratified output_.  The
tool [`destrat`](../luna/destrat.md) is designed to extract
information from such databases, and _de-stratify_ it, to produce
simple, rectangular tables that can be read into other analysis
programs or spreadsheets.  For example, consider the following
command:

```
luna s.lst sig=EEG,ECG,EMG -o out.db -s "EPOCH & STATS epoch" 
```

This generates a database, named `out.db` as requested.  The
`destrat` program will report on the contents of this file:

```
destrat out.db
```
```
--------------------------------------------------------------------------------
out.db: 2 command(s), 3 individual(s), 15 variable(s), 70731 values
--------------------------------------------------------------------------------
  command #1:	c1	Thu Aug 13 13:30:19 2020	EPOCH	sig=ECG,EEG,EMG
  command #2:	c2	Thu Aug 13 13:30:19 2020	STATS	epoch=T sig=ECG,EEG,EMG
--------------------------------------------------------------------------------
distinct strata group(s):
  commands      : factors           : levels        : variables 
----------------:-------------------:---------------:---------------------------
  [EPOCH]       : .                 : 1 level(s)    : DUR INC NE
                :                   :               : 
  [STATS]       : CH                : 3 level(s)    : MAX MEAN MEDIAN MEDIAN.MEAN MEDIAN.MEDIAN
                :                   :               : MEDIAN.RMS MEDIAN.SKEW MIN NE NE1
                :                   :               : RMS SKEW
                :                   :               : 
  [STATS]       : E CH              : (...)         : MAX MEAN MEDIAN MIN RMS SKEW
                :                   :               : 
----------------:-------------------:---------------:---------------------------
```

That is, we see that 2 commands were performed, generating output for
3 individuals.  Different Luna commands will produce different levels
of stratified output, depending on how they are run.  By default, the
`STATS` command produces one set of values for each channel.  This is
reflected in the _strata group_ labeled `CH`.  The 3 _levels_ of this
_factor_ are the 3 channels specified in the command (i.e. EEG, ECG
and EMG).  The five core variables (`MIN`, `MAX`, `MEAN`, `SD` and
`RMS`) are equivalent to the results generated at the start of this
tutorial using the `STATS` command: that is, whole-signal statistics.

Running `destrat` with the `-x` option will give information about the
factors and levels for that strata, rather than extracting the tabular
output _per se_:

```
destrat out.db +STATS -r CH -x
```
```
Factors: 1
     [CH] 3 levels
     -> ECG, EEG, EMG

Individuals: 3
     nsrr01 nsrr02 nsrr03

Commands: 1
     STATS

Variables: 12
     STATS/MAX STATS/MEAN STATS/MEDIAN STATS/MEDIAN.MEAN STATS/MEDIAN.MEDIAN STATS/MEDIAN.RMS STATS/MEDIAN.SKEW
      STATS/MIN STATS/NE STATS/NE1 STATS/RMS STATS/SKEW
```

Adding the `epoch` option to the `STATS` command generated
some additional output: 

- per-epoch values for these five measures (`MIN`, `MAX`, `MEAN`, `SD`
and `RMS`) -- these values are _stratified_ by both channel (`CH`) and
epoch (`E`)

- a set of additional whole-signal summaries, based on combining the
per-epoch level data and taking the median value -- like the original
output, these values are only stratified by channel `CH`

Specifically, the new variables are 

- the digital and physical min/max values from the EDF header (`DMIN`, `DMAX`, `PMIN` and `PMAX`)
- the observed physical min/max values from the signal data (`OMIN`, `OMAX`)
- the median of the per-epoch mean (`MEDIAN.MEAN`), SD (`MEDIAN.SD`) and RMS (`MEDIAN.RMS`) 
- the unit of measurement from the EDF header (`UNIT`)

!!! hint 
    The goal of the `STATS` command's `epoch` option is primarily
    to generate per-epoch level data, although using the median of
    per-epoch means will be more robust to outliers compared to the
    whole-signal mean, which is why both are given.


To extract information on a subset of variables, use the `-v` option
(where multiple variable names can be comma-delimited).  The `[STATS]`
argument tells `destrat` which command's output we are interested in;
the `-r` option specifies that channels (`CH`) are listed as separate
rows in the output.

```
destrat out.db +STATS -r CH -v RMS
```
```
ID        CH     RMS
nsrr01    ECG    0.266660881869383
nsrr01    EEG    37.801350632511
nsrr01    EMG    13.9700738671353
nsrr02    ECG    0.274496551353303
nsrr02    EEG    41.0442336241345
nsrr02    EMG    19.7064640585027
nsrr03    ECG    0.300029835524692
nsrr03    EEG    54.2707996328254
nsrr03    EMG    18.8981013309648
```

Running the same command but swapping from rows to columns (i.e. `-c`
instead of `-r`), we get the following:

```
destrat out.db +STATS -c CH -v RMS 
```
```
ID        RMS.CH.ECG          RMS.CH.EEG         RMS.CH.EMG
nsrr01    0.266660881869383   37.801350632511    13.9700738671353
nsrr02    0.274496551353303   41.0442336241345   19.7064640585027
nsrr03    0.300029835524692   54.2707996328254   18.8981013309648
```

By default, all decimal places are given for numeric data; this can be
modified with the `-p` option:

```
destrat out.db +STATS -c CH -v RMS -p 2
```
```
ID       RMS.CH.ECG   RMS.CH.EEG   RMS.CH.EMG
nsrr01   0.27         37.80        13.97
nsrr02   0.27         41.04        19.71
nsrr03   0.30         54.27        18.90
```

The `-i` option can be used to subset results to one or more
individuals: in this case, `nsrr02`.  Also, note here that we are
requesting different values -- although we request the same variable
(`RMS`), the strata are defined by channel (`CH`) _and_ epoch (`E`).
That is, these values are the per-epoch RMS values for each of the
three channels.  We can use both `-r` and `-c` to organize the output:
e.g. arrange values for different channels as columns, but epochs as
rows, as below.  New variable names are created in the form
`{VAR}.{FAC}.{LVL}` where `{VAR}` is the original variable name,
`{FAC}` is the column-factor (i.e. `CH`), and `{LVL}` is the level for
that factor (e.g. `ECG`).  Multiple column-stratifiers can be combined
in this way, e.g. `-c FAC1 FAC2`).

```
destrat out.db +STATS -r E -c CH -v RMS  -p 2 -i nsrr02 
```
```
ID       E   RMS.CH.ECG   RMS.CH.EEG   RMS.CH.EMG
nsrr02   1   0.27         11.98        4.91
nsrr02   2   0.28         15.88        9.91
nsrr02   3   0.27         12.47        22.73
nsrr02   4   0.26         11.35        13.61
nsrr02   5   0.26         12.29        19.06
nsrr02   6   0.27         15.23        23.92
nsrr02   7   0.28         14.47        20.08
nsrr02   8   0.30         14.88        23.88
nsrr02   9   0.30         19.56        22.70
... (cont'd) ...
```

Alternatively, here we run the same command but with both epoch and
channel as row-stratifiers, which may be more convenient for some
types of downstream analysis.

```
destrat out.db +STATS -r E CH -v RMS -p 2 -i nsrr02
```
```
ID       CH     E     RMS
nsrr02   ECG    1     0.27
nsrr02   ECG    2     0.28
nsrr02   ECG    3     0.27
nsrr02   ECG    4     0.26
nsrr02   ECG    5     0.26
nsrr02   ECG    6     0.27
nsrr02   ECG    7     0.28
 ... cont'd ...  
nsrr02   ECG    1193  1.08
nsrr02   ECG    1194  1.02
nsrr02   ECG    1195  0.13
nsrr02   EEG    1     11.98
nsrr02   EEG    2     15.88
nsrr02   EEG    3     12.47
  ... cont'd ...  
```

Finally, `destrat` can combine results across multiple databases, if
these are listed on the command line.  For example, if we were to run
different channels and individuals separately, recorded as two
databases:

The first two individuals for the EEG channel:

```
luna s.lst 1 2 sig=EEG -o p1.db -s STATS
```

Then the second and third individuals for ECG and EMG:

```
luna s.lst 2 3 sig=ECG,EMG -o p2.db -s STATS
```

Finally, all individuals for SaO2:

```
luna s.lst sig=SaO2 -o p3.db -s STATS
```

Multiple databases can be specified on the `destrat` command line, and
the output is compiled into a single file:

```
destrat p1.db p2.db p3.db
```
```
attaching databases...
scanning 1 of 3: p1.db
--------------------------------------------------------------------------------
p1.db: 1 command(s), 2 individual(s), 6 variable(s), 12 values
--------------------------------------------------------------------------------
  command #1:	c1	Mon Mar 18 16:04:45 2019	STATS	
--------------------------------------------------------------------------------
distinct strata group(s):
  commands      : factors           : levels        : variables 
----------------:-------------------:---------------:---------------------------
  [STATS]       : CH                : 1 level(s)    : MAX MEAN MEDIAN MIN RMS SD
                :                   :               : 
----------------:-------------------:---------------:---------------------------
scanning 2 of 3: p2.db
--------------------------------------------------------------------------------
p2.db: 1 command(s), 2 individual(s), 6 variable(s), 24 values
--------------------------------------------------------------------------------
  command #1:	c1	Mon Mar 18 16:04:49 2019	STATS	
--------------------------------------------------------------------------------
distinct strata group(s):
  commands      : factors           : levels        : variables 
----------------:-------------------:---------------:---------------------------
  [STATS]       : CH                : 2 level(s)    : MAX MEAN MEDIAN MIN RMS SD
                :                   :               : 
----------------:-------------------:---------------:---------------------------
scanning 3 of 3: p3.db
--------------------------------------------------------------------------------
p3.db: 1 command(s), 3 individual(s), 6 variable(s), 18 values
--------------------------------------------------------------------------------
  command #1:	c1	Mon Mar 18 16:04:52 2019	STATS	
--------------------------------------------------------------------------------
distinct strata group(s):
  commands      : factors           : levels        : variables 
----------------:-------------------:---------------:---------------------------
  [STATS]       : CH                : 1 level(s)    : MAX MEAN MEDIAN MIN RMS SD
                :                   :               : 
----------------:-------------------:---------------:---------------------------
```

```
destrat p1.db p2.db p3.db +STATS -v MEAN RMS -r CH -p 3
```

```
ID       CH     MEAN      RMS
nsrr01   EEG    -0.301    37.801
nsrr01   SaO2   76.924    85.567
nsrr02   EEG    -0.370    41.044
nsrr02   ECG    0.006     0.274
nsrr02   EMG    -0.610    19.706
nsrr02   SaO2   77.873    86.460
nsrr03   ECG    0.004     0.300
nsrr03   EMG    3.014     18.898
nsrr03   SaO2   65.083    77.731
```

!!! Note
    Currently, if drawing from multiple databases, only row-based formatting can be requested (i.e. `-r` and not `-c`).

!!! Note 
    If the same command/variable/individual/strata combination appears
    more than once (either within a single database, or across
    multiple databases), then only the last encountered will be used in the output.

## Parameter files

As we have seen, Luna can accept variables in a command file.  These
can be defined on the command line (as `variable=value` pairs), but
they can also be included in a _parameter_ file, which defines these.
For example, consider this file (which is in `cmd/vars.txt` in the tutorial folder):

```
alias   EEG1|EEG
alias   EEG2|EEG(sec)
alias   OXSTAT|"OX STAT"
eeg     EEG1,EEG2
myepoch 10
nrem    NREM1,NREM2,NREM3
```

All parameter file rows must be __exactly two tab-delimited columns__. 
(That is, be careful if copying and pasting the text from above, as 
the web formatting puts _spaces_ rather than _tabs_.)

The first contains a variable name, the second contains the value.
For example, `${myepoch}` is defined as a variable meaning `10`.  The
`${nrem}` variable is defined to list all NREM sleep annotations.
Parameter files can be useful for keeping command scripts generic (i.e
applicable to different studies, which may have different wording for
a given annotation or signal), by supplying a project-specific
parameter file.

Here `alias` is a reserved keyword that _for signals_ can be used to
specify alternate names for the same signal.  It expects a
pipe-delimited set of values (2 or more) in the second column,
indicating that the first value (e.g. `EEG1`) is the preferred name,
but that `EEG` means that same thing.  If a signal called `EEG` is
encountered, it is replaced to `EEG1` internally, and in the output.

Finally, as a shortcut, the variable `${eeg}` is defined here to
indicate both EEG channels, comma-delimited and labeled here by their
aliases: `EEG1,EEG2`.

So, if this file `cmd/second.txt` (also in the tutorial folder) is as follows:

```
EPOCH len=${myepoch}
MASK all
MASK unmask-if=${nrem}
RESTRUCTURE
STATS sig=${eeg}
```

then the following command will calculate statistics for both EEG
channels, label them as `EEG1` and `EEG2`, only for 10-second epochs
of either stage N1, N2 or N3 sleep, and only for the individual `nsrr01`:

```
luna s.lst nsrr01 @cmd/vars.txt -o res.db < cmd/second.txt
```
```
===================================================================
+++ luna | v0.24, 12-Aug-2020 |  starting 13-Aug-2020 13:32:56  +++
===================================================================
input(s): s.lst
output  : res.db
commands: c1	EPOCH	len=${myepoch}
        : c2	MASK	all=T
        : c3	MASK	unmask-if=${nrem}
        : c4	RESTRUCTURE	
        : c5	STATS	sig=${eeg}

___________________________________________________________________
Processing: nsrr01 [ #1 ]
 duration: 11.22.00 ( clocktime 21.58.17 - 09.20.17 )

 signals: 14 (of 14) selected in a standard EDF file:
  SaO2 | PR | EEG2 | ECG | EMG | EOG(L) | EOG(R) | EEG1
  AIRFLOW | THOR_RES | ABDO_RES | POSITION | LIGHT | OX_STAT

 annotations:
  NREM1 (x109) | NREM2 (x523) | NREM3 (x16) | NREM4 (x1)
  REM (x238) | apnea_obstructive (x37) | arousal_standard (x194) | artifact_SpO2 (x59)
  desat (x254) | hypopnea (x361) | wake (x477)

 variables:
  airflow=AIRFLOW | ecg=ECG | eeg=EEG2,EEG1 | effort=THOR_RES,A...
  emg=EMG | eog=EOG(L),EOG... | hr=PR | light=LIGHT | oxygen=SaO2,OX_STAT
  position=POSITION
 ..................................................................
 CMD #1: EPOCH
   options: len=10 sig=*
 set epochs, length 10 (step 10), 4092 epochs
 ..................................................................
 CMD #2: MASK
   options: all=T sig=*
 reset all 4092 epochs to be masked
 ..................................................................
 CMD #3: MASK
   options: sig=* unmask-if=NREM1,NREM2,NREM3
 set masking mode to 'unmask'
 based on NREM1 327 epochs match; 0 newly masked, 327 unmasked, 3765 unchanged
 total of 327 of 4092 retained
 based on NREM2 1569 epochs match; 0 newly masked, 1569 unmasked, 2523 unchanged
 total of 1896 of 4092 retained
 based on NREM3 48 epochs match; 0 newly masked, 48 unmasked, 4044 unchanged
 total of 1944 of 4092 retained
 ..................................................................
 CMD #4: RESTRUCTURE
   options: sig=*
  restructuring as an EDF+ :   keeping 19440 records of 40920, resetting mask
  retaining 1944 epochs
 ..................................................................
 CMD #5: STATS
   options: sig=EEG2,EEG1
 processing EEG2 ...
 processing EEG1 ...

___________________________________________________________________
...processed 1 EDFs, done.
...processed 1 command set(s),  all of which passed
-------------------------------------------------------------------
+++ luna | finishing 13-Aug-2020 13:32:57                       +++
===================================================================
```
