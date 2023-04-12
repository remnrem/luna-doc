# Working with output databases

## Overview

The primary output of most [_lunaC_](args.md) commands is a
specially-formatted _database_ file, which can contain the results of
one or more analyses for one for more individuals/EDFs.  Although they
can have any filename (typically in this documentation we call them
`out.db`) we'll refer to these output databases generically as _lunout_
(<b>L</b>una-<b>out</b>put) files.  This section describes how to use
the [destrat](#destrat) command-line tool as well as the
[_lunaR_](../ext/R/index.md) package to extract information from _lunout_
files.

!!! info "Why doesn't Luna just write to plain text files?"
    Although you don't _have_ to use these databases (i.e. you can use [text table](#text-tables) mode, or have Luna
    write
	everything to [standard out](https://en.wikipedia.org/wiki/Standard_streams) which can be redirected to a text file),
    in practice it is often easier to work with _lunout_ files.  In large
    part, this is because Luna's output is often from multiple
    commands, and each command may have output _stratified_ by a
    number of factors: channel, sleep stage, frequency, epoch or sleep
    cycle, pairs of channels, etc. Rather than generate dozens of text
    files for each of these differently-formatted commands, Luna
    stores everything in a single database, along with a set of tools
    for extracting the required information. Although the syntax and
    logic may appear a little opaque at first, there is a consistency
    across commands, meaning that once learned it will help
    with all aspects of Luna.
    
    
 
## destrat

As described [here](args.md#destrat), the `-o` (or `-a`) argument instructs Luna to
write its output to a _lunout_ database file:

```
luna s.lst nsrr01 sig=ECG,EMG -o out.db -s HEADERS 
```

This example generates a _lunout_ file called `out.db`.  This file (which
is actually an [SQLite](http://sqlite.org) database) cannot be directly
displayed in the terminal via a text-editor or spreadsheet.  Rather, a
_lunout_ file is an intermediate form, from which various text-files (or
R objects) can be extracted in a variety of formats, using the
`destrat` program that comes with Luna (or, as described below,
[_lunaR's_](../ext/R/index.md) `ldb()` function).

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

Running `destrat` with just a command label, which should either be in
square brackets or preceded by a `+` character will show the data
from the baseline stratum for that command, if one exists:

```
destrat out.db [HEADERS]
```
which prints the **tab-delimited** output:
```
ID      NR      NS   REC.DUR   TOT.DUR.HMS   TOT.DUR.SEC
nsrr01  40920   2    1         11:22:00      40920
```
The output of the `HEADERS` command is described
[here](../ref/summaries.md#headers).

!!! hint
    Some shells interpret square brackets `[` and `]` as special characters.
    If this appears to be the case, either place quotes around the command, e.g.:
    ```
    destrat out.db "[HEADERS]" 		
    ```
    or (even easier) use a single `+` character (at the start of the command name)
    to indicate it is a command:
    ```
    destrat out.db +HEADERS
    ```

Naturally, one can save any output from `destrat` to a file using
standard redirection operators (i.e. to create files that can be
loaded into other analysis programs such as R). For example, (here
using the `+command` format, which we'll adopt as the default in this
documentation):


```
destrat out.db +HEADERS > my-file.txt
```

!!! Hint 
    All destrat command, variable, factor and level names are _case-sensitive_.

To extract information from the second strata group (which is defined
by the factor `CH`), we need to explicitly list the factor(s) that
define it, use either the `-r` or `-c` options. The choice of `-r`
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
luna s.lst nsrr01 sig="EEG,EEG(sec)" -o out.db -s "EPOCH & PSD epoch"
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

We now see four distinct strata groups.  The `EPOCH` command produces
some basic output in the _baseline_ stratum (such as the number of
epochs, `NE`).  For the `PSD` command, we see three strata groups 
(none of which are the default baseline group) that are collectively 
defined by three _factors_:

| Factor | Description |
| ---- | ----- | 
| `E` | _Epoch_ (due to the `epoch` option on the `PSD` command) |
| `B` | Spectral _band_ | 
| `CH` | _Channel_, because `PSD` always operates on a particular channel |

Based on these three _factors_, there are three distinct strata groups 
from `PSD`, each of which contains its own set of variables/data, are:

| Strata group | Content |
| ----- | ----- | 
| `CH`             | Number of epochs (although this will be similar for each channel) |
| `B` x `CH`       | Spectral band power for each channel for the entire signal | 
| `E` x `B` x `CH` | As above, but output _per-epoch_ (due to the `epoch` option of the `PSD` command) | 

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
nsrr01   THETA       EEG       54.713385057902     0.0378016941869899
nsrr01   ALPHA       EEG       63.1553608045886    0.0436342886275004
nsrr01   SIGMA       EEG       678.134027746239    0.468525482521808
nsrr01   SLOW_SIGMA  EEG       563.260922552984    0.389159199696685
nsrr01   FAST_SIGMA  EEG       114.873105193254    0.0793662828251232
nsrr01   BETA        EEG       225.6867899095      0.155927895983292
nsrr01   GAMMA       EEG       45.8934730333316    0.0317079820769435
nsrr01   TOTAL       EEG       1447.37917796109    1
nsrr01   SLOW        EEG(sec)  173.30016852818     0.11036981788844
nsrr01   DELTA       EEG(sec)  368.806565177362    0.234882134162885
nsrr01   THETA       EEG(sec)  99.1852731160328    0.0631682047628917
nsrr01   ALPHA       EEG(sec)  113.195854898732    0.0720911352655018
nsrr01   SIGMA       EEG(sec)  427.048500439725    0.271974722375411
nsrr01   SLOW_SIGMA  EEG(sec)  346.660966460099    0.220778248874063
nsrr01   FAST_SIGMA  EEG(sec)  80.3875339796257    0.0511964735013477
nsrr01   BETA        EEG(sec)  239.891616774602    0.152779967158952
nsrr01   GAMMA       EEG(sec)  74.5169610826192    0.0474576770128835
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

### Aggregating output


If there are multiple individuals in a Luna project, these will be
compiled and output together.  The `-i` option, followed by a list of
one or more individual IDs can be used to restrict the output to only
those individuals/EDFs.

`destrat` can also compile and integrate information across multiple
databases by listing multiple files as follows, e.g. something like:
 
```
destrat out1.db out2.db +HEADERS -r CH > all-out.txt
```

or 

```
destrat *.db +HEADERS -r CH > all-out.txt
```

The different databases may contain similar or different individuals;
further, they may contain similar or different commands.  One issue to
remember is that if the same data-point is included in more than one
file, only one value will be used (i.e. there is no mechanism for resolving
potential discrepancies, etc).  If an individual did not have data for
that command/variable/level, `destrat` will output `NA` (the missing
code used in R).

!!! note "Restriction on `-c` when combining multiple databases"
    One caveat is that the `-c` option cannot be used when multiple
    databases are specified on the command line.  That is, you have to
    use `-r` instead. (It is always possible to restructure back to
    column-format using other tools, e.g. `dcast()` in R


### Restricting output

The `-v` option can be used to select only certain variables (with spaces between variables, and noting that 
all names are _case-sensitive_):

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

To obtain a list of the levels for a given stratum, run destrat with
the `-x` option (which means _no output_) as follows (here, it doesn't
matter whether `-r` or `-c` is used):

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

### Command summary


| Option | Example | Description | 
| ---- | ----- | ----- | 
| `+command`  | `+ANNOTS` | Select output from this command |
| `[command]`  | `[ANNOTS]` | Equivalent to `+ANNOTS` | 
| `-r` _factor(s)_   | `-r CH` | Select strata group defined by `CH` and organize by rows | 
| `-c` _factor(s)_   | `-c CH` | Select strata group defined by `CH` and organize by columns | 
| `-x`               | `-x`  | Display information about the database, rather than extracting data | 
| `-p` _integer_     | `-p 2` | Restrict numeric output to two decimal places | 
| `-i` _ID(s)_       | `-i nsrr01` | Restrict output to this individual(s) | 
| `-v` _variable(s)_ | `-v DENS` | Restrict output to only this variable(s) | 


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

_LunaR's_ [`ldb()`](../ext/R/ref.md#ldb) function can read _lunout_ files
generated by [_lunaC_](args.md) directly into R.  Although destrat is
more flexible, if you are performing downstream analyses in R anyway,
then using `ldb()` obviates the need to use destrat to create
intermediate text files, if they are then only read into R.
See the documentation on [`ldb()`](../ext/R/ref.md#ldb) for more
information.

## Scaling

There is effectively no formal limit on the size of a _lunout_ database
(i.e. SQLite can _in principle_ handle a database file up to 140TB).
Naturally, very large databases take longer to process, however.  When
destrat first encounters a _lunout_ file, it generates an index that
speeds up subsequent queries: the time it takes to generate the index
is obviously related to how large the database is.  In general, size
and performance issues will only arise if you are placing output for
hundreds of individuals in the same output file, or if you have
commands then generate a _lot_ of output (e.g.  full cross spectra for
all pairs of channels in an hdEEG study, separately for every epoch).
Therefore, follow the usual, common-sense principles of prototyping
analyses on one or two individuals first and see how things scale.  It
may often be easier (or necessary) to have different _lunout_ databases
for different individuals and/or commands, e.g. using the `^` wildcard to generate
a different database for each ID/EDF in the sample list:
```
luna s.lst -o out/run1-^.db < cmd.txt
```  
Alternatively, in certain circumstances, you may want to forego using a _lunout_ database entirely
and use [text-tables](#text-tables) instead.


## Text tables

As described [here](args.md#text-tables), by using `-t folder` instead
of `-o database.db`, Luna will write all tables as text, with one
subfolder per individual/EDF under `folder/`.  As noted above, this
can be advantageous under some scenarios, e.g. for output with large
numbers of strata/levels, including epoch-by-channel-by-frequency
spectrograms as from `PSD epoch-spectrum`.  See also the _lunaR_
function [`ltxttab()`](../ext/R/ref.md#ltxttab) which can facilitate
working with text-table output (i.e. concatenating tables across
individuals/subfolders).


By default, certain files that Luna expects to be large will be
compressed (`.txt.gz`).  To turn off compression, add `compressed=0`.
To force _all_ output to be compressed, add `compressed=1`.

### Naming conventions

For use with the [`merge`](../merge/merge.md) utility, the special variables
`tt-prefix` and `tt-suffix` (equivalently, `tt-prepend` and
`tt-append`) can be set to alter the naming of files generated by the
`-t` option.  If a file would have been generated with
the name:

```
SPINDLES-F_CH_THR_PHASE.txt
```
then adding `tt-prefix=XXX` and `tt-suffix=YYY` will result in:
```
XXX-SPINDLES-F_CH_THR_PHASE_YYY.txt
```
Specifically, the suffix can be used to specify factor/level values for that table,
with factor and level delimited by a hyphen/minus (`-`), e.g.:
```
tt-prepend=SS-N2_TH-4.5
```
implies a factor `SS` with level value `N2`, and a second factor `TH` with level value `4.5`.  The prefix `XXX` is
understood by [`merge`](../merge/merge.md) to indicate the domain-group pair.
See [merge](../merge/merge.md) for more details. 


!!! warning "Known issues"
    You may encounter one of these issues when using to `-t` flag

    - Certain commands may given a message saying that `-t` has not yet been enabled: in this case, use `-o` or plain-text output mode
    
    - Certain commands may add additional variables to the output, even if the option normally required
    (under `-o` output mode) wasn't given.  For example, the
    `SPINDLES` command will generate variables for slow oscillations
    and their coupling with spindles, even if the `sw` parameter was
    not specified. These extra columns will be populated by `NA` (not
    available) values when that option wasn't specified. Whereas the
    database output only lists what it observes, text-table output
    mode has to force the columns of the output file _before_ it knows
    what analyses will be performed; this will occassionally mean that
    extra variables are included.

    - In certain instances (e.g. especially for the `SPINDLES` command
      when using additional options) you may find that information is
      split across multiple rows.  For example
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
      i.e. as the strata (here `ID`, `F` and `CH` are identical, all this information should be on one row.       
      You can use the `-o` option and _destrat_, or just work around the issue with the output files. 

      __We expect all these issues to be fixed in future releases.__
