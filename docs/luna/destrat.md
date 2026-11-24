# Working with output databases

## Overview

Most [_lunaC_](args.md) commands write results for one or more
individuals/EDFs to an output database (called `out.db` throughout this
documentation). This page describes how to extract those results with
the [destrat](#destrat) command-line tool or [_lunaR_](../ext/R/index.md).

!!! info "Why doesn't Luna just write to plain text files?"
    You can instead use [Luna text-tables](#luna-text-tables) or redirect
    [standard output](https://en.wikipedia.org/wiki/Standard_streams) to a
    text file. Databases are usually more convenient when output comes from
    several commands and is stratified by channel, stage, frequency, epoch,
    or similar factors: one database holds all results, and `destrat` extracts
    the required table.
    
    
 
## destrat

As described [here](args.md#output-databases), the `-o` (or `-a`) argument instructs Luna to
write its output to an output database file:

```
luna s.lst nsrr01 sig=ECG,EMG -o out.db -s HEADERS 
```

This generates an [SQLite](http://sqlite.org) output database called
`out.db`. Use `destrat` to inspect it and extract text tables, or use
the [R package `luna`](../ext/R/index.md) to import it with `p$import_db()`.

To view the contents of this file, run destrat without any other
options:

```
destrat out.db 
```

```
----------------------------------------------------------------------------
out.db: 1 command(s), 1 individual(s), 11 variable(s), 17 values
----------------------------------------------------------------------------
  command #1:	c1	Thu Feb  7 10:19:51 2019	HEADERS	
----------------------------------------------------------------------------
distinct strata group(s):
  commands      : factors       : levels        : variables 
----------------:---------------:---------------:---------------------------
  [HEADERS]     : .             : 1 level(s)    : NR NS REC.DUR TOT.DUR.HMS 
                :               :               : TOT.DUR.SEC
                :               :               : 
  [HEADERS]     : CH            : 2 level(s)    : DMAX DMIN PDIM PMAX PMIN 
                :               :               : SR
----------------:---------------:---------------:---------------------------
```


Here we see when the file was generated and some information about the
number of individuals, commands and variables stored within. We
also see two distinct _strata_ groups:
 
 - a _default_ (or sometimes referred to here as a _baseline_) group,
   meaning that there are no stratifying factors

- a second group defined by the factor `CH`, which has two levels
  (i.e. corresponding to the two channels specified, `ECG` and `EMG`)

In each case, the variables defined for each strata group are listed
on each row.

A strata group corresponds to a table, where each row of that table
corresponds to one unique combination of levels for the factor(s) in
that stratum. `destrat` will extract information from only _one_
strata group at a time.  Think of each strata group as a virtual
table, defined by a particular set of factors: it does not make sense
to mix the information about the general EDF with the information
about individual channels, for example.

Running `destrat` with a `+` command label shows the data from the
baseline stratum for that command, if one exists:

```
destrat out.db +HEADERS
```
which prints the **tab-delimited** output:
```
ID      NR      NS   REC.DUR   TOT.DUR.HMS   TOT.DUR.SEC
nsrr01  40920   2    1         11:22:00      40920
```
The output of the [`HEADERS`](../ref/summaries.md#headers) command is described
[here](../ref/summaries.md#headers). Only a subset of columns is shown above for
clarity.

### Main options

| Option | Example | Description |
| ---- | ----- | ----- |
| `+command` | `+ANNOTS` | Select output from this command |
| `-r` _factor(s)_ | `-r CH` | Select strata group defined by `CH` and organize by rows |
| `-c` _factor(s)_ | `-c CH` | Select strata group defined by `CH` and organize by columns |
| `-x` | `-x` | Display information about the database, rather than extracting data |
| `-l` | `-l +PSD` | Emit one long-format row per stored value |
| `-d` | `-d` | Emit the database variable dictionary |
| `-p` _integer_ | `-p 2` | Restrict numeric output to two decimal places |
| `-a` _prefix_ | `-a PSG` | Prefix output variable names |
| `-i` _ID(s)_ | `-i nsrr01` | Restrict output to these individuals |
| `-v` _variable(s)_ | `-v DENS` | Restrict output to these variables |
| `-e` | `-e` | Retain selected row strata that are empty for an individual |
| `-t` _directory_ | `-t extracted` | Extract all selected tables to a directory; see below |
| `-z` | `-t extracted -z` | Gzip files written by `-t` |
| `--mean`, `--median`, `--sd`, `--range` | `--mean` | Summarize numeric values across individuals |
| `--metrics-cols`, `--metrics-rows` | `--mean --metrics-rows` | Put numeric summary measures in columns (default) or rows |
| `--tabulate` | `--tabulate` | Count observed values rather than extract them |
| `--tabulate-all` | `--tabulate --tabulate-all` | Allow tabulations with more than 100 groups |

Naturally, one can save any output from `destrat` to a file using
standard redirection operators (i.e. to create files that can be
loaded into other analysis programs such as R). For example:


```
destrat out.db +HEADERS > my-file.txt
```

!!! hint
    All destrat command, variable, factor and level names are _case-sensitive_.

To extract information from the second strata group (which is defined
by the factor `CH`), we need to explicitly list the factor(s) that
define it, using either the `-r` or `-c` options. The choice of `-r`
versus `-c` influences the layout of the output, in terms of whether
factors are listed as additional _rows_ or _columns_.  This is
probably easiest to show by example.  In the first instance:

```
destrat out.db +HEADERS -r CH
```

will list each _level_ of `CH` (i.e. each channel) as a separate _row_ in the output:
```
ID        CH     DMAX   DMIN    PDIM  PMAX    PMIN     SR
nsrr01    ECG    127    -128    mV    1.25    -1.25    250
nsrr01    EMG    127    -128    uV    31.5    -31.5    125
```

Alternatively, the same information can be listed in a _column-wise_
format, where each level of `CH` is a new column,  with the `-c` option:

```
destrat out.db +HEADERS -c CH
```

```
ID      DMAX.CH.ECG DMAX.CH.EMG DMIN.CH.ECG DMIN.CH.EMG PDIM.CH.ECG PDIM.CH.EMG PMAX.CH.ECG PMAX.CH.EMG PMIN.CH.ECG PMIN.CH.EMG SR.CH.ECG SR.CH.EMG
nsrr01  127         127         -128        -128        mV          uV          1.25        31.5        -1.25       -31.5       250       125
```

Note how each individual variable, e.g. `DMAX`, is split into two
variables in the output, either `DMAX.CH.ECG` or `DMAX.CH.EMG`.
Depending on how you want to analyse the data, and the number of
factors/levels, either `-r` or `-c` formatted output may be the more
appropriate choice.


### Multiple factors

To further illustrate `destrat` with multiple factors, consider this
example of power spectral density estimation for two channels (named
`EEG` and `EEG(sec)` as per the NSRR tutorial data), performed for
both the entire record as well as per-epoch:


```
luna s.lst nsrr01 sig="EEG,EEG(sec)" -o out.db -s 'EPOCH & PSD epoch'
```

(note the use of quotes around the `sig` list, which avoids the
shell from interpreting the parentheses as special characters)

```
destrat out.db
```
```
--------------------------------------------------------------------------------
out.db: 2 command(s), 1 individual(s), 6 variable(s), 51877 values
--------------------------------------------------------------------------------
  command #1:	c1	Thu Feb  7 10:33:45 2019	EPOCH	
  command #2:	c2	Thu Feb  7 10:33:45 2019	PSD	
--------------------------------------------------------------------------------
distinct strata group(s):
  commands      : factors           : levels        : variables 
----------------:-------------------:---------------:---------------------------
  [EPOCH]       : .                 : 1 level(s)    : DUR INC NE
                :                   :               : 
  [PSD]         : CH                : 2 level(s)    : NE
                :                   :               : 
  [PSD]         : B CH              : 20 level(s)   : PSD RELPSD
                :                   :               : 
  [PSD]         : E B CH            : (...)         : PSD RELPSD
                :                   :               : 
```

We now see four distinct strata groups.  The [`EPOCH`](../ref/epochs.md#epoch) command produces
some basic output in the _baseline_ stratum (such as the number of
epochs, `NE`).  For the [`PSD`](../ref/power-spectra.md#psd) command, we see three strata groups 
(none of which are the default baseline group) that are collectively 
defined by three _factors_:

| Factor | Description |
| ---- | ----- | 
| `E` | _Epoch_ (due to the `epoch` option on the [`PSD`](../ref/power-spectra.md#psd) command) |
| `B` | Spectral _band_ | 
| `CH` | _Channel_, because [`PSD`](../ref/power-spectra.md#psd) always operates on a particular channel |

Based on these three _factors_, there are three distinct strata groups 
from [`PSD`](../ref/power-spectra.md#psd), each of which contains its own set of variables/data, are:

| Strata group | Content |
| ----- | ----- | 
| `CH`             | Number of epochs (although this will be similar for each channel) |
| `B` x `CH`       | Spectral band power for each channel for the entire signal | 
| `E` x `B` x `CH` | As above, but output _per-epoch_ (due to the `epoch` option of the [`PSD`](../ref/power-spectra.md#psd) command) | 

In other words, `out.db` contains four _virtual tables_, and
we can output any one of them by specifying the appropriate
factors with the `-r` and/or `-c` options, as well as the command name.
(as `+command`).  When a stratum is defined by more than one
factor (i.e. `B` and `CH` for the third group), it is possible to
specify some factors as _rows_ and some as _columns_.  Here, both factors 
are requested with _row-wise_ formatting:

```
destrat out.db +PSD -r CH B 
```

```
ID       B           CH        PSD                 RELPSD
nsrr01   SLOW        EEG       105.991683363628    0.0732300733474261
nsrr01   DELTA       EEG       198.692418792271    0.137277378186528
...      ...         ...       ...                 ...
nsrr01   TOTAL       EEG(sec)  1570.17717201771    1
```

!!! note 
    If you're looking at these power estimates, they may seem
    strange for sleep data (i.e. sigma higher than delta).  Note that
    this command is looking over _all_ epochs, including many
    artifactual wake/end-of-study epochs that the end of the
    recording.  Examining the _epoch-level_ estimates will make this clear, e.g. 
    extracted with:
    ```
    destrat out.db +PSD -r E B CH > out.txt
    ```
    You'll see in the [tutorial](../tut/tut1.md) how to
    [mask](../ref/masks.md), [filter](../ref/fir-filters.md) and
    [detect artifacts](../ref/artifacts.md) in EEG data using Luna. 


To instead specify that channels are listed as columns:
```
destrat out.db +PSD -r B -c CH 
```

```
ID       B          PSD.CH.EEG        PSD.CH.EEG(sec)   RELPSD.CH.EEG       RELPSD.CH.EEG(sec)
nsrr01   SLOW       105.991683363628  173.30016852818   0.0732300733474261  0.11036981788844
nsrr01   DELTA      198.692418792271  368.806565177362  0.137277378186528   0.234882134162885
nsrr01   THETA      54.713385057902   99.1852731160328  0.0378016941869899  0.0631682047628917
nsrr01   ALPHA      63.1553608045886  113.195854898732  0.0436342886275004  0.0720911352655018
nsrr01   SIGMA      678.134027746239  427.048500439725  0.468525482521808   0.271974722375411
nsrr01   SLOW_SIGMA 563.260922552984  346.660966460099  0.389159199696685   0.220778248874063
nsrr01   FAST_SIGMA 114.873105193254  80.3875339796257  0.0793662828251232  0.0511964735013477
nsrr01   BETA       225.6867899095    239.891616774602  0.155927895983292   0.152779967158952
nsrr01   GAMMA      45.8934730333316  74.5169610826192  0.0317079820769435  0.0474576770128835
nsrr01   TOTAL      1447.37917796109  1570.17717201771  1                   1
```

!!! hint
    The order in which you specify the `-r` and `-c` options does not matter.

### Multiple databases

To combine results from multiple databases, list their files before the
selection options:
 
```
destrat out1.db out2.db +HEADERS -r CH > all-out.txt
```

or 

```
destrat *.db +HEADERS -r CH > all-out.txt
```

Databases may contain different individuals and commands. If a data point
occurs in more than one database, `destrat` uses one value. Missing values
are output as `NA`.

!!! note "Combining multiple databases"
    `destrat` supports both `-r` and `-c` when combining databases. Columns
    are matched by their factor names and levels; values absent from an input
    database are emitted as `NA`.


### Restricting output

When a database contains several individuals, use `-i` followed by one or
more IDs to limit the output to those individuals.

Use `-v` followed by one or more space-separated variables to restrict
output; names are _case-sensitive_:

```
destrat out.db +EPOCH -r E -v START STOP 
```

Also, you can restrict output to only certain _levels_ of particular
_factors_, by specifying `-r` or `-c` in the form _`factor/level`_ or
_`factor/level1,level2`_. For example, using the `out.db` generated
above, we could extract only relative sigma and beta power:

```
destrat out.db +PSD -r B/SIGMA,BETA CH -v RELPSD -p 2
```

```
ID      B      CH        RELPSD
nsrr01  SIGMA  EEG       0.47
nsrr01  BETA   EEG       0.16
nsrr01  SIGMA  EEG(sec)  0.27
nsrr01  BETA   EEG(sec)  0.15
```

That is, this extracts only `RELPSD` (from `-v`) for only sigma and
beta power (from `B/SIGMA,BETA`).  Furthermore, it uses the `-p 2`
option to restrict numeric output to two decimal places.

To list the levels in a stratum, use `-x` (no data are extracted). It makes
no difference whether the factors follow `-r` or `-c`:

```
destrat out.db +PSD -r B CH  -x
```
```
Factors: 2
     [B] 10 levels
     -> ALPHA, BETA, DELTA, FAST_SIGMA, GAMMA, SIGMA, SLOW, SLOW_SIGMA, 
        THETA, TOTAL

     [CH] 2 levels
     -> EEG, EEG(sec)

Individuals: 1
     nsrr01

Commands: 1
     PSD

Variables: 2
     PSD/PSD PSD/RELPSD
```

### Extract all selected tables

`destrat -t` is distinct from [Luna text-tables](#luna-text-tables)
option of the same name. It reads one or more output databases and writes a
separate, row-oriented text file for every selected command/factor-set table:

```
destrat out.db -t extracted
```

The directory is created if needed. File names combine the command and factor
names (for example, `PSD_CH_B.txt`), and each file combines the corresponding
table across all supplied databases. `+command`, `-i`, and `-v` may be
used to limit the exported tables, individuals, or variables. Add `-z` to
write `.txt.gz` files.

Because `-t` determines the row strata itself, it cannot be combined with
`-r`, `-c`, `-l`, `-x`, or `-d`.

### Long, dictionary, and summary output

`-l` writes one tab-delimited row per stored value, with database, ID, command,
strata, variable, and value columns. `-d` writes the variable dictionary.

The numeric summary options (`--mean`, `--median`, `--sd`, and `--range`)
aggregate numeric values across individuals; output also includes `N`.
`--metrics-rows` places each summary measure on a separate row, whereas
`--metrics-cols` (the default) writes them as columns. `--tabulate` instead
counts observed values. It cannot be combined with numeric summary options;
by default it stops if more than 100 groups would be produced, unless
`--tabulate-all` is specified.


## behead

_behead_ is a very simple text utility that is supplied with Luna and
destrat, which can be used to make output more human-friendly.  The
input is a tab-delimited _rectangular_ file (i.e. with the same number
of columns on each row) and a _header row_ (i.e. containing variable
names), as produced by _destrat_.

For example, if this file is `out.txt`
```
ID      CH        DMAX    DMIN     PDIM   PMAX   PMIN   SR
nsrr01  SaO2      32767   -32768   %      100    0      1
nsrr01  PR        32767   -32768   BPM    200    0      1
nsrr01  EEG(sec)  127    -128      uV     125    -125   125
nsrr01  ECG       127    -128      mV     1.25   -1.25  250
nsrr01  EMG       127    -128      uV     31.5   -31.5  125
nsrr01  EOG(L)    127    -128      uV     125    -125   50
nsrr01  EOG(R)    127    -128      uV     125    -125   50
nsrr01  EEG       127    -128      uV     125    -125   125
nsrr01  AIRFLOW   127    -128      NA     -1     1      10
nsrr01  THOR RES  127    -128      NA     -1     1      10
nsrr01  ABDO RES  127    -128      NA     -1     1      10
nsrr01  POSITION  3      0         NA     3      0      1
nsrr01  LIGHT     1      0         NA     1      0      1
nsrr01  OX STAT   3      0         NA     3      0      1
```
then 
```
behead < out.txt 
```
produces a file where each row of the input is represented as several rows in the output:
```
                       ID   nsrr01              
                       CH   SaO2                
                     DMAX   32767               
                     DMIN   -32768              
                     PDIM   %                   
                     PMAX   100                 
                     PMIN   0                   
                       SR   1                   

                       ID   nsrr01              
                       CH   PR                  
                     DMAX   32767               
                     DMIN   -32768              
                     PDIM   BPM                 
                     PMAX   200                 
                     PMIN   0                   
                       SR   1                   

... (etc) ...
```

In practice, you may want to pipe straight from _destrat_ and combine _behead_ with _less_ or a similar pager:

```
destrat out.db +HEADERS -r CH | behead | less 
```
(press `q` to quit)


<h5>Options</h5>

Add `-t` to _behead_ to get tab-delimited output instead of the format
above; add `-n` to get additional row/column numbering in the output;
add `-nt` for both.


## lunaR

The R package `luna` 1.8.0 reads output databases directly:

```r
library(luna)
p <- lp$proj()
p$import_db("out.db")
p$strata()
headers <- p$table("HEADERS", "CH")
```

Choose the command and strata listed by `p$strata()` to retrieve another
table. Importing a new database replaces the current project results,
so save any data frames you need before the next import. See the
[R reference](../ext/R/ref.md#running-commands-across-a-project).

## Scaling

There is effectively no formal limit on the size of an output database
(i.e. SQLite can _in principle_ handle a database file up to 140TB).
Naturally, very large databases take longer to process, however.  When
destrat first encounters a database, it generates an index that
speeds up subsequent queries: the time it takes to generate the index
is obviously related to how large the database is.  In general, size
and performance issues will only arise if you are placing output for
hundreds of individuals in the same output file, or if you have
commands then generate a _lot_ of output (e.g.  full cross spectra for
all pairs of channels in an hdEEG study, separately for every epoch).
Therefore, follow the usual, common-sense principles of prototyping
analyses on one or two individuals first and see how things scale.  It
may often be easier (or necessary) to have different output databases
for different individuals and/or commands, e.g. using the `^` wildcard to generate
a different database for each ID/EDF in the sample list:
```
luna s.lst -o out/run1-^.db < cmd.txt
```  
Alternatively, in certain circumstances, you may want to forego using an output database entirely
and use [Luna text-tables](#luna-text-tables) instead.


## Luna text-tables

This section describes the `-t` option to _lunaC_, not `destrat -t` described
above. As described [here](args.md#text-tables), by using `-t folder` instead
of `-o database.db`, Luna will write all tables as text, with one
subfolder per individual/EDF under `folder/`.  As noted above, this
can be advantageous under some scenarios, e.g. for output with large
numbers of strata/levels, including epoch-by-channel-by-frequency
spectrograms as from `PSD epoch-spectrum`.  See also the _lunaR_
helper [`lp$read_text_table()`](../ext/R/ref.md#text-output) which can facilitate
working with text-table output (i.e. concatenating tables across
individuals/subfolders).


By default, certain files that Luna expects to be large will be
compressed (`.txt.gz`).  To turn off compression, add `compressed=0`.
To force _all_ output to be compressed, add `compressed=1`.

### Advanced filename conventions

For legacy `dmerge` workflows, `tt-prefix` and `tt-suffix` (also
`tt-prepend` and `tt-append`) alter names generated by `-t`. For example,
the default name:

```
SPINDLES-F_CH_THR_PHASE.txt
```
becomes the following with `tt-prefix=XXX tt-suffix=YYY`:
```
XXX-SPINDLES-F_CH_THR_PHASE_YYY.txt
```
`tt-prepend` can also encode factor/level values, with factor and level
delimited by a hyphen (`-`):
```
tt-prepend=SS-N2_TH-4.5
```
This specifies factor `SS` at level `N2` and factor `TH` at level `4.5`.


!!! warning "Known issues"
    You may encounter one of these issues when using the `-t` flag:

    - Some commands do not yet support `-t`; use `-o` or plain-text output
      mode instead.
    
    - Text-table mode sometimes creates columns for optional output that was
      not requested. Those columns contain `NA`. For example,
      [`SPINDLES`](../ref/spindles-so.md#spindles) includes slow-oscillation
      and coupling columns even when `sw` was not specified.

    - In some cases, particularly [`SPINDLES`](../ref/spindles-so.md#spindles)
      with additional options, values for identical strata may be split across
      rows. For example:
      ```
      ID   F   CH   V1    V2    V3
      id1  15  C3   1.2   -0.8  NA
      id1  15  C3   NA    NA    44.5
      ```
      instead of
      ```
      ID   F   CH   V1    V2    V3
      id1  15  C3   1.2   -0.8  44.5
      ```
      Here the identical `ID`, `F`, and `CH` strata would ideally occupy one
      row. Use `-o` with _destrat_ if a rectangular table is required.
