# Luna command-line tool documentation

## Overview

Luna is fundamentally a console/command-line package, i.e. there is no
_point-and-click_. When talking specifically about the command-line
interface to Luna, we'll often refer it as _lunaC_, to distinguish it from the
[_lunaR_](../ext/R/index.md) package for R.  Familiarity with the
basic Unix/macOS console environment and shell scripting is recommended.

Once [installed](../download/index.md) and in your command path,
_lunaC_ is invoked via the `luna` command, often in the following
form:

``` 
luna sample.lst -o out.db < commands.txt
```

Here, Luna expects a list of IDs, EDFs (and possibly [annotation files](#annotations)) in
a [sample list](args.md#sample-lists) file (`sample.lst`), reads a
series of [commands](../ref/index.md) (`commands.txt`) to be applied to
each EDF, and writes the output to a
[_lout_](destrat.md) database file (`out.db`).  

!!! note "Types of command line arguments" 
    _lunaC_ expects the first
    argument to be either 1) an EDF (or EDF+ or [EDFZ](#edfzs)) file,
    2) a [_sample-list_](#sample-lists), 3) a plain-text ASCII file
    or 4) a special command that does not require signal data,
    e.g. such as [`--build`](#-build) or [`--xml`].  Subsequent arguments can
    come in any order and are interpreted as follows

    - terms containing `=` are interpreted as [_variables_](#variables), with the exception of certain special _reserved_ names that are options, e.g. `annot` or `alias`, as described below
    - terms starting with `-` are interpreted as special options, e.g. primarily `-o` and `-s`
    - once a `-s` option is encountered, _all_ subsequent terms are interpreted as Luna [_commands_](../ref/index.md) rather than command line options; thus, if given, then `-s` must be the final argument
    - terms starting with `@` are interpreted as [_parameter files_](#parameter-files), the contents of which are loaded in to define new variables and options
    - otherwise, terms that are numbers are interpreted as sample-list [_row numbers_](#ranges) (either a single row, or a range, depending if one or two numbers are specified)
    - otherwise, that term is assumed to be the ID of a _single_ individual to be analysed from the sample-list

## Help

Luna has a help function that describes the available
[_commands_](../ref/index.md), their parameters and their outputs.  Commands are organized by _domains_, listed by the following command:

```
luna -h 
```
```
usage: luna [sample-list|EDF] [n1] [n2] [@parameter-file] [sig=s1,s2] [v1=val1] < command-file

List of domains
---------------

annot      Annotations                 
artifact   Artifacts                   
cfc        Cross-frequency coupling    
cmdline    Command-line options        
epoch      Epochs                      

...
```
To view the commands within a domain:
```
luna -h manip
```
```
ANON         Strips EDF ID and Start Date headers
COPY         Duplicate one or more EDF channels
FLIP         Flips the polarity of a signal
RECORD-SIZE  Alters the record size and writes a new EDF
REFERENCE    Re-reference signal(s)
RESAMPLE     Resample signal(s)
SIGNALS      Retain/remove specific EDF channels
mV           Converts a signal to mV units
uV           Converts a signal to uV units
```
To view the parameters and outputs for a given command:
```
luna -h PSD
```
```
PSD : Power spectral density estimation (Welch) (Power spectra)
    : http://zzz.bwh.harvard.edu/luna/ref/power-spectra/#psd

Parameters:
===========

  epoch                           Calculate per-epoch band power
  epoch-spectrum                  Calculate per-epoch power spectra
  max              max=100        Specify maximum frequency for power spectra
  bin              bin=0.5        Specify bin-size for power spectra
  sig              sig=C3,C4      Restrict analysis to these channels
  spectrum                        Calculate power spectra

Outputs:
========

   CH                      Number of epochs
   ------------------------------------------------------------
     NE                    Number of epochs

   B x CH                  Whole-night, per-channel band power
   ------------------------------------------------------------
     PSD                   Power
     RELPSD                Relative power

   CH x F                  Whole-night, per-channel power
   ------------------------------------------------------------
     PSD                   Power

   B x CH x E              Whole-night, per-channel per-epoch band power
   ------------------------------------------------------------
     PSD                   Power
     RELPSD                Relative power

   CH x E x F              Whole-night, per-channel per-epoch power
   ------------------------------------------------------------
   (compressed output)
     PSD                   Power
```

!!! note
    By convention, all Luna commands should be in _CAPITAL LETTERS_. 



## Input 

### EDFs

Luna can read EDF and EDF+ files.  The latter allow for
discontinuities (by having an explicit time-track) and provide some
support for annotations.  After
[restructuring](../ref/index.md#restructure) file (i.e. removing
certain epochs), it will be represented internally in a form that
corresponds to EDF+.  

Luna can read a single EDF (rather than a [sample
list](args.md#sample-lists)) by specifying a filename (with an `.edf`
or `.EDF` extension):

```
luna test1.edf < commands.txt
```

!!! note
    In this _single EDF_ mode, it is not possible to attach
    annotations, or include a parameter file.  Using sample lists is
    the preferred way to work for most projects.

<h4>In-memory representations</h4>

A note on some of the terminology used in this documentation: we often
make a distinction between the _on-disk_ EDF and its _in-memory_
representation, or the _internal_ EDF.

When Luna first "attaches" an EDF, it does not load anything other
than the header.  Subsequently, Luna uses a _lazy-loading_ approach,
whereby it only pulls records from disk when needed.  These
are cached in memory, so that if they are needed again, they do not
need to be read in afresh.

All commands that manipulate EDFs and the signals therein operate on
the internal EDF.  Although we may use language such as "drop a
channel from the EDF" we typically do not imply that the actual
_on-disk_ file has changed in any way.  

Likewise, most commands refer to the _internal, in-memory_ version of
the EDF both for their input and output. As noted, this may differ
from the _on-disk_ version, for example, if channels and/or epochs
have been dropped or added by `SIGNALS`, `MASK`, `RESTRUCTURE` and
other commands. When describing a command such as
[`DESC`](../ref/summaries.md#desc), if we say it reports
"...information on the attached EDF", this implies it is the
_in-memory_ form that is being queried.  That is, if the _on-file_ EDF
contains 64 channels, `DESC` will report 64 all other things being
equal.  However, if _within that run of Luna_ other commands have been
previously been issued that alter the _internal_ EDF, then `DESC` may
report something other than 64.

Finally, there is no _memory_ or persistence of changes between
different runs of _lunaC_: nothing is cached between sessions.
For example, the following may report that 6 channels are present:
```
luna my.edf -s DESC 
```
You might subsequently run a command that drops channels and then calls `DESC`: 
```
luna my.edf -s "SIGNALS drop=ECG,EMG & DESC"
```
in which case, only 4 channels will be reported. However, on 
running the first command again:
```
luna my.edf -s DESC
```
Luna will report 6 channels again, 
i.e. as nothing was changed in the original file `my.edf`. 

### EDFZs

As described in this [vignette](../vignettes/edfz.md), to save disk space
(and sometimes speed up analysis), Luna can read and write compressed
EDF files, using the [BGZF](https://samtools.github.io/hts-specs/SAMv1.pdf) library.

EDFZ files must be created by Luna's
[`WRITE`](../ref/outputs.md#write) command, with the `edfz` parameter
option added.  


For example, taking the first tutorial EDF, we can write it out as an EDFZ:
```
luna s.lst 1 -s WRITE edfz edf-dir=z/ edf-tag=compressed sample-list=z.lst 
```

That is, if the original EDF were called `file.edf`, this would create
two files (along with a sample list `z.lst` that points to them):
`z/file-compressed.edfz` and `z/file-compressed.edfz.idx`.

In the above example, we can see the reduction in disk space:
```
ls -lh z
```
```
    21M  z/learn-nsrr01-compressed.edfz
   528K  z/learn-nsrr01-compressed.edfz.idx
```
In contrast, the original EDF is almost three times the size:
```
    59M  edfs/learn-nsrr01.edf
```

For a single PSG study that is not particularly large to start with,
this saving doesn't really matter.  Across thousands of studies,
naturally, savings can become more significant.

Although EDFZ files must be created by a special Luna command, they
can be read (i.e. decompressed) as any other
[gzip](https://en.wikipedia.org/wiki/Gzip) file.  That is, the
following standard unix/Mac command will convert an EDFZ file to a
standard EDF file:

```
cat file.edfz | gunzip > file.edf
```

See information on the [`WRITE`](../ref/outputs.md#write) command's
`edfz` option for more details.

!!! hint 
    Although compression as an EDFZ is _lossless_ (i.e. all
    information is preserved), there may be very small differences
    between the original EDF and an uncompressed EDFZ simply due to
    floating point accuracy of the EDF format. This is not speicfic to
    EDFZ files -- it also applies to standard EDFs generated by the
    [`WRITE`](../ref/outputs.md#write) command.

### Plain-text input

Luna can read signals from ASCII-formatted, tab-delimited plain-text files as well as EDFs.
Consider if we had a file `signals.txt` with 15,360 rows and three columns (i.e. three signals):

```
head signals.txt
```
```
 4.18192918193	 9.73748473748	6.33394383394
 3.93772893773	 6.74603174603	6.94444444444
 3.44932844933	 4.12087912088	7.09706959707
 2.41147741148	 2.28937728938	6.33394383394
 0.21367521367	 1.12942612943	5.72344322344
-3.02197802198	-0.27472527472	4.96031746032
-6.74603174603	-2.53357753358	3.89194139194
-9.43223443223	-4.91452991453	3.12881562882
-9.92063492063	-6.86813186813	2.67094017094
-8.45543345543	-8.15018315018	1.75518925519
```

If Luna finds `--fs` on the command line, it will
interpret the first argument to be a text file (rather than a sample
list or an EDF).  In this instance, we know that the sample rate is
265 Hz (and so, implies 15,360/256 = 60 seconds of signal).

```
luna signals.txt --fs=256 -s DESC
```
```
Processing: signals.txt [ #1 ]
  reading 3 signals, 60 seconds (15360 samples 256 Hz) from signals.txt
 ..................................................................
```
```
EDF filename      : signals.txt
ID                : signals.txt
Clock time        : 00.00.00 - 00.01.00
Duration          : 00.01.00
# signals         : 3
Signals           : S1[256] S2[256] S3[256]
```

Here, we see the recording is of the expected length (1 minute).  If
combined with the [`WRITE`](../ref/outputs.md#write) command, one can
use Luna to convert a text file into an EDF.

The new signals are, by default, labelled `S1`, `S2`, etc.  If the `--chs` option is specified on the command-line, different channel labels can be assigned:
```
luna signals.txt --fs=256 --chs=Cz,Fz,Pz -s DESC
```
```
Signals           : Cz[256] Fz[256] Pz[256]
```

Alternatively, if the first row of the text file starts with a `#`
symbol, Luna assumes this is a comma-delimited list of channel labels:
```
#EEG1,EEG2,EEG3
```
```
Signals           : EEG1[256] EEG2[256] EEG3[256]
```

!!! warning 
    If both a header row and `--ch` are specified, the values
    in `--chs` will be used.  If the number of channels specified by
    either `--chs` or the header does not match the number of columns
    in the file, Luna may give an error, as it expects the total
    number of sample points read to be a multiple of the number of
    channels and the sampling rate.  In other words, internally, Luna
    will structure the data as an EDF with a 1-second record size, and
    therefore any input must contain an integer number of seconds.

Naturally, if a different/incorrect sampling rate is given, Luna will
assume the recording is of a different length (subject to the
constraint above about integer number of seconds being required): e.g.

```
luna signals.txt --fs=128 -s DESC
```
```
Duration          : 00.02.00
```


### Sample lists

The sample list (`sample.lst` in the example at the top of this page)
file defines a _project_, i.e. a collection of EDFs and their
associated IDs and [annotations](#annotations).

!!! warning 
    The sample list file must be ASCII plain text and
    tab-delimited, i.e. not containing any special characters or
    formatting as may occur if using a word processor to generate this
    file. Additionally, be sure to [remove any Windows-style
    line-ending characters](../faq.md#line-endings).

Sample lists must contain at least two tab-delimited fields: the
subject ID, followed by the EDF file location:

```
id001	test1.edf
id002	test2.edf
```

In this next example we specify an [_annotation file_](#annotations)
(e.g. containing stage information) for each EDF.  Note, in the
first case we use an absolute file path, but a relative one in the
second.

```
id001	test1.edf	/data/annots/test1-staging.xml
id002	test2.edf	staging2.xml	
```

Relative paths are evaluated relative to the directory that Luna is
run from.  On a single system, and if files won't be moved often, it
is better to use absolute paths.  See [below](args.md#search-paths)
for notes on using relative paths, which can be more convenient in
some circumstances.  Also see the [`--repath`](../ref/helpers.md#-repath)
function that can be used to quickly search/replace paths in a sample list.

Instead of an annotation file, it is also possible to specify a
_folder_, which will be searched (but not recursively) for annotation files
(those with extensions `.annot`, `.txt`, `.tsv`, `.eannot` and `.xml`):

```
id001	test1.edf	/data/annots/indiv/id001/
```

If there are multiple annotation files (or folders) they can be listed a separate tab-delimited fields (i.e. columns 3, 4, 5, etc).
Alternatively, they can be given as a comma-delimited list in the third column. Also, if there are no annotation files for that EDF,
you can put a period (`.`) character.  These two rules allow for a clean three tab-delimited column scheme, irrespective of the number of
annotation files, which can make sample lists easier to work with (e.g. to load into R as a tab-delimited file):
```
id001	test1.edf	test1.tsv
id002	test2.edf	.
id003	test3.edf	test3.tsv,staging-id003.eannot
```

### _--build_ option

Luna's `--build` option can generate [_sample-lists_](#sample-lists)
automatically, by recursively scanning one or more folders (and their
subfolders) to find EDF files (ending `.edf` or `.EDF`) and
_associated_ [annotation files](#annotations) (e.g. `.annot`, `.xml`, etc, or a
user-specified alternative).  An annotation file is said to be
associated with an EDF if it has the same file-name (except for the
extension); it is not necessary for it to be in the same folder as the
EDF.
 
For example, given these two folders: 
```
ls edfs/
```
```
f1.edf f2.edf f3.edf
```
and 
```
ls xmls/
```
```
f1.xml f2.xml f3.xml
```
The command
```
luna --build edfs xmls > s.lst
```
will scan these two folders (there can be any number, and each can contain EDFs and/or annotations), matching `f1.edf` with `f1.xml`, etc, to generate a sample-list (written to standard output) with three entries:
```
cat s.lst
```
```
f1   edfs/f1.edf   xmls/f1.xml
f2   edfs/f2.edf   xmls/f2.xml
f3   edfs/f3.edf   xmls/f3.xml
```
By default, Luna will look at the header of each EDF file to determine the ID (i.e. the first column of the sample list).  If the ID is the same as the filename (or if the EDF headers do not contain unique IDs), you can add the option 
```
 -usefilename
```
to use the root filename as the ID (in this example, they are the same, i.e. `f1`, `f2` and `f3`).

If the EDFs and the XMLs were in the same folder (say `mydata`), this
command would still work as intended: e.g.
```
luna --build mydata > s.lst
```

Alternatively, different folders might contain different EDFs but that
have similar file names. For example:

```
ls subj1 subj2
```
```
subj1:
night1.edf	night1.xml	night2.edf	night2.xml

subj2:
night1.edf	night1.xml	night2.edf	night2.xml
```

Here, we do not want to associated `subj1/night1.edf` with
`subj2/night1.edf` or `sub2/night1.xml`.  Here, add the option
`-nospan` to instruct Luna not to span folders when associating files:
```
luna --build subj1 subj2 -nospan > s.lst 
```
```
s1night1	subj1/night1.edf	subj1/night1.xml
s1night2	subj1/night2.edf	subj1/night2.xml
s2night1	subj2/night1.edf	subj2/night1.xml
s2night2	subj2/night2.edf	subj2/night2.xml
```

In this example, the IDs (`s1night1`, etc) come from the EDF headers:
these should be unique, or else Luna will print a warning message
about dulplicate IDs.  That is, every row of a sample-list should
contain a unique ID.

To specfify special extensions for annotation files (i.e. third column
onwards of sample lists), add the `-ext` option: e.g. to match
`.txt` instead of `.xml` files:
```
luna --build mydata -ext=txt > s.lst
```

If no `-ext` command is given, then `--build` will match on `.xml`,
`.annot`, `.eannot`, `.txt` and `.tsv`.  You can specify a
comma-delimited list of multiple extensions.

It can be convenient to specify extensions if annotation files have a
regular naming scheme but do not match the EDF files identically.  For
example, many NSRR files have EDF and annotation files as follows:

```
file1.edf     file1-nsrr.xml
file2.edf     file2-nsrr.xml
```
In this instance, add the full tag and extension: 
```
luna --build mydata -ext=-nsrr.xml > s.lst
```
i.e. this matches __file1__[.edf] with __file1__[-nsrr.xml].

Finally, whether or not the resulting sample-list uses relative or
absolute paths depends on how you specify the search folders on the
command line:

```
luna --build edfs xmls
```
```
f1	edfs/f1.edf	xmls/f1.xml
f2	edfs/f2.edf	xmls/f2.xml
f3	edfs/f3.edf	xmls/f3.xml
```
versus 
```
luna --build /Users/mary/proj1/edfs /Users/mary/proj1/xmls
```
```
f1	/Users/mary/proj1/edfs/f1.edf	/Users/mary/proj1/xmls/f1.xml
f2	/Users/mary/proj1/edfs/f2.edf	/Users/mary/proj1/xmls/f2.xml
f3	/Users/mary/proj1/edfs/f3.edf	/Users/mary/proj1/xmls/f3.xml
```

### Empty EDFs

Rather than read in data, it is possible to specify an empty or "virtual" EDF.
This will have a
fixed durataion and sample rate, but initially no channels/signal
data.  This can be convenient when using certain commands that do not
require signal data, e.g. the [`SIMUL`](../ref/simul.md#simul) or [`OVERLAP`](../ref/intervals.md#overlap) commands.

Here, you specify `.` (period character) as the sample-list/file-name
along with `--nr` and `--rs` on the command line, to give the number of
records (`nr`) and the EDF record size (`rs`) respectively.

Luna will then create an EDF of this duration (i.e. with headers
speciying the length of the recording) but with 0 signals, i.e. a
collection of empty records.


### Command files

Command files or _scripts_ (i.e. `commands.txt` in the example 
at the top of this page), contain one or more [Luna commands](../ref/index.md).  This
can be more flexible and convenient than placing all commands after
the `-s` argument.

Some conventions:
 
 - blank lines are skipped
 - all text after a `%` character on a line is treated as a _comment_ and skipped 
 - new commands should start on a new line
 - lines that begin with one or more spaces are assumed to be continuations of the same command, meaning that 
commands can be spread over several lines
 - scripts can contain [_variables_](#variables)

For example, say we wished to set [`EPOCH`](../ref/epochs.md#epoch) an
EDF and apply power spectral density estimation via the
[`PSD`](../ref/power-spectra.md#psd) command to the channel named
`EEG`, outputting spectra for each epoch.  For a sample-list `s.lst`
and [output file](#lout-databases) `out.db`, we could specify this on
the command line as:

```
luna s.lst -o out.db -s "EPOCH & PSD epoch sig=EEG"
```

Alternatively, we could place the following in a _plain-text_ file
called `commands.txt` (or anything else, there are no limitations on
the file name or extension):

```
EPOCH 
PSD epoch sig=EEG
```
and run it as follows:

```
luna s.lst -o out.db < commands.txt
```

Alternatively, we could achieve the same outcome but with a more
verbosely-expressed script, with ample comments and spreading commands
and options over multiple lines:

```
% This is a new command file
% v1.0, 5-Feb-19
% First epoch the data, default epoch duration is 30 seconds

EPOCH

% Power spectral density estimation for the EEG channel
% Note the we start lines after PSD with spaces (otherwise, they
% would be interpreted as new commands)

PSD
  epoch     % produce per-epoch level output 
  sig=EEG   % only apply this command to channels named EEG
```


<h4>Conditional blocks</h4>

You can define blocks within a command file that are only executed if
a [variable](#variables) is set to a non-null value, e.g. `1`.  If the variable is
null (undefined or `0`) then those blocks are skipped, using the
double-bracket syntax as follows:

```

EPOCH len=${l}

[[var

  SIGSTATS sig=${eeg} mask th=${thresholds}

]]var

PSD sig=${eeg}

```

If the variable `${var}` is null, then _all text_ (i.e. including other
variable definitions and conditional statements as well as commands)
will be skipped, up until the block closing (here `]]var`).  In this
case, the `SIGSTATS` command will only be executed if `${var}` has
been set to a non-null value:

```
luna my.edf var=1 < cmd.txt
```

Note how the variable _var_ is referenced without the usual `${}`
syntax when paired with `[[` or `]]`. Also note that every block
opening (e.g. `[[var`) requires a matching block closing
(e.g. `]]var`).  Luna will give an error if it encounters a block closing 
without having first encountered a closing.  

It is possible to set nested conditional blocks:

```
% commands here always executed

[[a
 
 % commands here only executed if ${a} is non-null

 [[b
   % commands here only executed if both ${a} and ${b} are non-null
 ]]b

 % commands here only executed if ${a} is non-null

]]a

% commands here always executed
```

Note that the indentation of blocks as above is optional; in fact,
conditional statements can occur all on the same line, e.g. as below,
where `${setmask}` determines whether the `SIGSTATS` command also
includes optional parameters to set a mask:

```
EPOCH len=${l}

SIGSTATS sig=${eeg} [[setmask  mask th=${thresholds}  ]]setmask

PSD sig=${eeg}
```


### Variables

In a [command file](#command-files), a variable _`var`_ is denoted by
the following syntax:

```
${var}
```

For example, to take the example from the previous section, we could
modify it to allow different epoch durations and channel names:

```
EPOCH len=${l}
PSD epoch sig=${s}
```

When running Luna on this command file, it would then be necessary to
specify a value for the variables _l_ and _s_, e.g.:

```
luna s.lst l=20 s=EEG < commands.txt 
```

which will then perform the spectral analyses using 20-second epochs.
If _l_ and _s_ were not specified, Luna would give an error.



Using variables can make command files more generic by reducing the
need to make minor edits across different projects.  For example,
consider one project that labels stage N2 sleep `NREM2` and has EEG
channels `C3-M2` and `C4-M1`, whereas another project uses `Stage2`
with EEG channel `EEG1` only.  In this scenario, we could still use a
single command file:

```
EPOCH
MASK ifnot=${nrem2}
PSD sig=${eeg}
```

but then invoke Luna with project-specific values for
these variables (spaces added for clarity):

```
luna proj1.lst nrem=N2      eeg=C3-M2,C4-M1  -o out1.db < commands.txt
```

```
luna proj2.lst nrem=Stage2  eeg=EEG1         -o out2.db < commands.txt
```

!!! Hint 
    Using project-specific [_parameter files_](#parameter-files)
    as described below can further simplify working with multiple
    projects.


<h4>Within-script variable assignment</h4>

It is also possible to define (or redefine) variables within a command
script, using the following syntax:

```
${var=xyz}
```

which sets `${var}` equal to the string `xyz`; this variable can subsequently be used as above, e.g.:
```
PSD sig=${var}
```

It is also possible to use previously defined variables in a variable
definition:

```
${z=${var}_v2}
```
would set `${z}` equal to `xyz_v2`.  Note, it is _not_ possible to use
arithmetic expressions in variable definitions: in command scripts,
all variables are treated as simple text strings.  The following would
not work therefore:

```
${a=2}          
${b=${a}+1}  % NO!!  i.e. sets ${b} to string '2+1' rather than '3'
```

<h4>Using variables on the command-line</h4>

If using variables on the command line, i.e. after `-s` or
piping in from `echo`, etc, rather than using a separate command file,
you will need to make sure the shell does not interpret the `$` as a
shell variable.  For example, if using `bash`, then use single quotes
instead of double quotes.  That is, this will _not_ work:

```
 luna s.lst -s "EPOCH & STATS sig=${eeg}" 
```

as the shell will try to expand the variable `${eeg}` before passing
it to Luna.  In this instance, using single quotes to stop the `bash`
shell from expanding the variable will achieve the desired effect:

```
 luna s.lst -s 'EPOCH & STATS sig=${eeg}'
```


<h4>Expansions of numeric sequences</h4>

It can sometimes be convenient to automatically specify a series of
channels, e.g for the output of
[ICA](../ref/cc.md#ica), where new channels are
created as `ICA1`, `ICA2`, etc.  Use the form `[root][n:m]` to obtain a comma-delimited 
list `root1,root2,root3`, e.g. if _n=1_ and _m=3_.  That is,
```
PSD sig=[ICA][1:5]
```
is identical to typing:
```
PSD sig=ICA1,ICA2,ICA3,ICA4,ICA5
```
This can of course be combined with a variable to specify how many components are analysed:
```
PSD sig=[ICA][1:${k}]
```

```
luna s.lst k=10 < cmd.txt
```

!!! hint 
    Numeric sequences are expanded after other variables have been swapped in.



### Individual variables

As well as _run-level_ variables (i.e. specified on the command line
or in a parameter file) that are common to all individuals/EDFs processed
in a given run, you can also assign values to variables on an
individual-by-individual basis, using values stored in a file.

For example, the following file defines three variables `${var1}`, `${var2}` and `${var3}`
for the [tutorial](../tut/tut1.md) individuals:
```
ID       var1    var2       var3
nsrr01   22      EEG1,EEG2  T
nsrr02   12      EEG1       F
nsrr03   98      .          T
```

This file should:

- be a ASCII, plain-text file (no special characters, etc)
- have a header row that includes the column `ID` in the first field
- be tab-delimited, with the same number of columns on each row

Variables can be given any name, does not need to be `var1` etc (but
no special characters, and they are case-sensitive). You can then
include these variables by setting the `vars` special variable.  If
the above file is named `indiv.dat`, for example, then:

```
luna s.lst vars=indiv.dat < my-commands.txt
```

The log information will give a summary of the attached variables for each individual: e.g. for the first individual:
```
 variables:
  airflow=AIRFLOW | ecg=ECG | eeg=EEG(sec),EEG | effort=THOR_RES,A...
  emg=EMG | eog=EOG(L),EOG... | hr=PR | id=nsrr01 | light=LIGHT
  oxygen=SaO2,OX_STAT | position=POSITION | var1=22 | var2=EEG1,EEG2 | var3=T
```
Note that the other _automatic_ variables losted here (`airflow`, etc) come from Luna's automatic assignment of [_channel types_](#channel-types).
If the script (`my-commands.txt`) contained references to `${var1}`, etc, then these would be substituted
as appropriate, for each individual: e.g. if the files had channels labelled (or aliased) to `EEG1` and `EEG2`,
then this command would run the `PSD` command for both channels, only for `EEG1`, or for neither, in the first, second
and third individual, respectively:
```
PSD sig=${var2}
```

The [`VARS`](../ref/summaries.md#vars) can be used to dump a list of which variables are defined for each individual.


<h4>Individual-ID substitution</h4>

One special variable is the `^` symbol (or, equivalently, `${id}`), which denotes the
individual/EDF ID for the file currently being processed. That is, it
does not need to be specified directly, but will be automatically
substituted for each EDF processed. This can be used to point to
individual-specific files. For example, below we use a combination of a
project-specific variable _p_ and the special individual-ID `^`
variable with the [`EPOCH-ANNOT`](../ref/epochs.md#epoch-annot) command:

``` 
...
EPOCH-ANNOT file=/path/to/${p}/data/^.eannot 
...  
```

One might then issue commands such as:

```
luna proj1.lst p=proj1 -o out1.db < commands.txt
```

```
luna proj2.lst p=proj2 -o out2.db < commands.txt
```

and Luna would look to the correct places to attach the
epoch-annotations, e.g. perhaps something like:

```
/path/to/proj1/data/id0001.eannot
/path/to/proj1/data/id0002.eannot
/path/to/proj1/data/id0003.eannot
...
```
and
```
/path/to/proj2/data/id0001.eannot
/path/to/proj2/data/id0002.eannot
/path/to/proj2/data/id0003.eannot
...
```


### Parameter files

As well as defining _variables_, there are a number of other
command-line options that control aspects of Luna's behavior, as
tabulated below.  For example, to set the search
[`path`](#search-paths) one might use:

```
luna s.lst path=/home/joe/data/edfs/ -o out1.db < commands.txt
```

Rather than having to retype such things every time, it can be
convenient to wrap them up in a _parameter file_, which is included
with a special `@` syntax, as follows:

```
luna s.lst @param.txt -o out.db < commands.txt
```

where the file `param.txt` (which can be called anything) is a
plain-text file that includes (possibly among other options) the line:

```
path	/home/joe/data/edfs
```

That is, the options in the file `param.txt` are expanded out as
though they were typed on the command line.  Whereas the command line
expects key/value pairs to be connected with an equals (`=`)
character, in a parameter file we use a tab instead.  Thus, a
parameter file should contain exactly **two tab-delimited columns** on
every row. As a larger example:

```
alias	EEG|C3-M2|C3|EEG1
sig	EEG
annot-folder	/path/to/my/annots/
path	/path/to/my/data/
sr	256
sample	project-name
nrem1   "Stage 1 sleep|1"
nrem2   "Stage 2 sleep|2"
nrem3   "Stage 3 sleep|3"
nrem4   "Stage 4 sleep|4"
rem     "REM sleep|5"
wake    "Wake|0"
excl    "Arousal resulting from respiratory effort|Arousal (ARO RES)","Arousal|Arousal ()","ASDA arousal|Arousal (ASDA)","Limb movement - left|Limb Movement (Left)","Periodic leg movement - left|PLM (Left)","Respiratory artifact|Respiratory artifact","Respiratory effort related arousal|RERA","Spontaneous arousal|Arousal (ARO SPONT)","Unscored|9","Unsure|Unsure"
```

In this example, we define an [_alias_](#aliases) (`EEG`) which is
specified to be the only channel loaded (via the [`sig`](#signal-lists)
option).  As well as setting the [`annot-folder`](#annotations) and
[`path`](#search-paths) folders, this parameter file also defines a
number of other [_variables_](#variables) that might be used in
command scripts.  For example, `${sr}` might be the sampling rate;
`${nrem1}`, `${nrem2}`, etc, specify the annotation labels used for
sleep stages; the comma-delimited list in `${excl}` might define a
list of exclusionary annotation, e.g. to be used with [`MASK`](../ref/masks.md#mask):

```
MASK if=${excl}
```


Specifying all relevant project-specific variables in a parameter file
allows generic command files to be applied more easily across multiple
projects.  For example (and as noted above), this is handy if
different EDFs have different labels, e.g. `Stage 2 sleep|2` and `N2`


```
nrem2	"Stage 2 sleep|2"
```

whereas a second study may use the annotation `N2`

```
nrem2	N2
```

but a single command file can be used (that references the variable
`${nrem2}`) for both studies, because project-specific parameter files
are used.  (Note that the use of quotes above is necessary because the
annotation `Stage 2 sleep|2` contains spaces.)

### Annotations 

The standard EDF format does not allow for any additional
_annotations_, i.e. labels that typically describe events in the data,
such as sleep stages, artifacts, or other scored features
(e.g. spindles).  The EDF+ format has some limited support for
annotations but is awkward to work with: although Luna can read EDF+ annotations,
these are not the preferred mode.

!!! note "EDF+ annotations"
    EDF+ annotations are limited by record size, and hard to generate
    or edit without altering the original EDF+ file.  For many
    practical applications, where the annotations reflect derived
    features of the sleep time series data, it is simpler to keep 
    the signal data and the annotation information separate.

As well as EDF+ Annotations, Luna accepts various other types of explicit _annotation
files_ that contain information about events in the EDF.  As noted
above in the [_sample-list_](#sample-lists) section, any files and
folders specified after the second tab-delimited column
(i.e. following ID and EDF) are interpreted as annotations files or
folders.  For an annotation folder, Luna will try to read _all_ files
in that folder.  Typically, one might
specify an individual-specific annotation folder, for example:

```
id0001    edf1.edf    /path/to/annotations/id0001/
id0002    edf2.edf    /path/to/annotations/id0002/
... (etc) ...
```

and place all annotations for that individual/EDF in that folder. 

<h5>Annotation formats</h5>

Luna accepts a number of formats of annotation file, which are
described in more detail in the respective reference sections linked
to below:

| Format | Description |
| ---- | ---- | 
| [EDF+](../ref/annotations.md#edf-annotations-channel) | EDF+ Annotations Channel | 
| [NSRR XML](../ref/annotations.md#nsrr-xml-files) | Format used by the [National Sleep Research Resource](http://sleepdata.org) to distribute sleep staging, and information on manually-scored arousals, movements and artifacts |
| [.annot](../ref/annotations.md#annot-files) | Generic Luna annotation files (as well as `.annot`, `.txt` and `.tsv` extensions are valid) | 
| [.eannot](../ref/annotations.md#eannot-files) | Simple epoch-level annotation files (can also be loaded with the [`EPOCH-ANNOT`](../ref/epochs.md#epoch-annot) command) | 

### Ranges

Given a sample list, by default Luna will iterate through all EDFs in
that list.  To restrict analyses to a single EDF, you can specify it
directly on the command line,  either by its ID or by the number
that is its position in the sample list.  For the sample list used in
the [tutorial](../tut/tut1.md), for example, if this is called
`s.lst`:

```
nsrr01	edfs/learn-nsrr01.edf	edfs/learn-nsrr01-profusion.xml
nsrr02	edfs/learn-nsrr02.edf	edfs/learn-nsrr02-profusion.xml
nsrr03	edfs/learn-nsrr03.edf	edfs/learn-nsrr03-profusion.xml
```
you could analyze only `nsrr02` by either
```
luna s.lst nsrr02 < commands.txt
```
or
```
luna s.lst 2 < commands.txt
```

!!! Note
    If a number is given after the sample list, it is always
    interpreted as the position in the sample list, not an ID.  In
    other words, best not to use pure numbers as IDs in the sample
    list if possible.  

To operate on a range of subjects within a sample list, just give two numbers: e.g. 
```
luna large.lst 50 100 < commands.txt
```

The above would analyze from the 50th to the 100th EDFs in the
`large.lst` project sample list.  This can be useful if using Luna on
a cluster, to parallelize processing via batch submission, for example.


### Time points

In a few contexts (various outputs) Luna encodes time in _time-points_
where 1 unit is 10<sup>-9</sup> seconds
(stored internally as `uint64_t` types).  For a given EDF, time-points
start at 0, corresponding to the start of the EDF. 


### Channel location files

Some commands that work with hdEEG data require a channel location map.  These should be in Cartesian X, Y, Z format. 

- Four tab-delimited columns
- First column is channel name
- Second to fourth columns are X, Y and Z coordinates

Internally, all coordinates are converted to spherical coordinates on a unit sphere. 


## Special variables

<----
silent
verbose
id
wildcard=^
sanitize
fix-edf "auto-correct" truncated/over-long EDFs
sec-dp
spaces=_
keep-spaces
keep-annot-spaces
keep-channel-spaces
class-instance-delimiter split class/annot remappings (ABC/DEF|XYZ)
combine-annots
annot-whitelist
annot-unmapped
annot-keyval  (key=val char)
align-annots  e.g. align-annots=W,N1,N2,N3,R


inst-hms         make instance ID time, if blank
force-inst-hms   force above

epoch-check=5    for .eannot length


annot-folder annot-folders
annots-file  (and +3 plurals)
annots annot

skip-sl-annots
skip-edf-annots
skip-annots
skip-all-annots     do not read EDF or ANNOT annotations

force-edf


tt-prepend
tt-append

ss-prefix sleepp stage prefix
vars
ids  ID remapper

ch-match TYPES
ch-exact
ch-clear

fail-list

compressed

nsrr-remap
remap
tab-only  fix delimiter to tab only for .annot

upper set channel names as all UPPERCASE

epoch-len

<----

Currently, the _special variables_ used by Luna are as tabulated below.  These can be assigned values on the
command line, or via an @included [_parameter file_](#parameter-files).

| Special Variable | Description |
| ---- | ---- | 
| [`path`](#search-paths) | Set search path for files in sample lists | 
| [`exclude`](#exclude-lists) | Specify a file of IDs to exclude from analysis |
| [`include`](#include-lists) | Specify a file of IDs to include from analysis |
| |
| [`sig`](#signal-lists)| Include this signal(s) in analysis | 
| [`alias`](#aliases)| Specify a channel alias |
| [`ch-exact`](#ch-exact)| Add an exact match for a channel type |
| [`ch-match`](#ch-match)| Add a partial match for a channel type |
| [`ch-clear`](#ch-clear)| Clear all channel type mappings |
| [`spaces`](#spaces-in-channel-names ) | Alternate character for space substitution in channel/annotation names |
| [`keep-spaces`](#spaces-in-channel-names) | Retain spaces in channel/annotation names if set to true |
| [`keep-channel-spaces`](#spaces-in-channel-names) | Retain spaces in channel names if set to true|
| [`keep-annot-spaces`](#spaces-in-channel-names) | Retain spaces in annotation names if set to true|
| |
| [`remap`](#remapping-annotations)| Specify an annotation remapping (cf. channel aliases) | 
| [`annot-file`](#annot-file) | Specify annotations to attach on the command line |
| [`annots`](#annotations)| Only load this (comma-delimited) list of annotations (rather than all) |
| [`force-edf`](#annotations)       | Skip EDF annotations _and_ time-track from any EDF+, and force as a continuous EDF |
| [`skip-annots`](#annotations)     | Skip XML and other (external) annotations (default: no)  |
| [`skip-edf-annots`](#annotations) |	Skip EDF Annotations tracks from any EDF+ (default: no) |
| [`skip-all-annots`](#annotations) | Same as `skip-annots=1` and `skip-edf-annots=1` combined (default: no) |
| [`inst-hms`](#inst-hms) | Automatically assign missing annotation instance IDs (from XML) based on time |
| [`nsrr-remap`](#nsrr-remap) | Set whether NSRR automatic remapping is on or off (annotation labels) | 
| |
| [`vars`](#vars) | Specify file with individual-level variables/values |
| [`tt-prepend`](#text-tables) | Add value to start of text-table file names (equiv. `tt-prefix`) |
| [`tt-append`](#text-tables) | Add value to end of text-table file names (equiv. `tt-suffix`) |
| [`compressed`](#text-tables) | Y/N to force all `-t` text-table output to compressed (Y) or not (N) |
| |
| [`epoch-len`](#epoch-len) | Specify the default epoch duration |
| [`no-epoch-check`](#no-epoch-check) | Do not enforce epoch check for .eannot files |
| [`assume-pm-start`](#force-evening-start-time)| Force morning times (after _X_ am) to be _X_ pm  |
| |
| [_power bands (various)_](#spectral-power-bands)| Change default power bands (delta, theta, etc.) | 


Any other variables specified on the command line or a [_parameter
file_](#parameter-files) are interpreted as typical variables, that
can be used in scripts: e.g.

```
luna s.lst xyz=123 < cmd.txt 
```

will set `${xyz}` to `123` if used in scripts.  Naturally, special
variables (i.e. those tabulated above) are reserved names, and cannot
be used in scripts.


!!! note "Specifying special variable values"
    They do not function as command-line
    options _per se_, meaning that they always need to be assigned a particular value, e.g.
    ```
    luna s.lst keep-spaces=T < cmd.txt
    ```
    rather than just
    ```
    luna s.lst keep-spaces < cmd.txt
    ```
    For variables that expect true/false values:

    - matches are case-insenstive
    - values of `1` or starting with `T` (true) or `Y` (yes) are all interpreted as _true_
    - all other values are interpreted as _false_; for clarity, `0`, `F` or `N` should be used in practice



### Signal lists

To restrict analyses to a subset of signals/channels in an EDF, use
the `sig` option either on the command line
```
luna s.lst sig=EEG,ECG,EMG -s DESC
```
or in a [_parameter file_](#parameter-files) as separate lines

```
sig	EEG
sig	ECG
sig	EMG
```
or as a comma-delimited list
```
sig	EEG,ECG,EMG
```

!!! Note 
    These options (when used on the command line, or via a parameter file) 
    mean that only these channels are loaded from
    the EDF, i.e. it is as though the other channels do not exist from
    Luna's perspective.  This is different from the using the 
    `sig` option to modify the behavior of an individual Luna command: 
    in this latter case, it is only that particular command that is
    restricted to those signals/channels, and so other channels are
    still part of the in-memory EDF for subsequent processing.  For
    example:
    ```
    ...
    FILTER bandpass=0.3,35 ripple=0.01 tw=0.2 sig=C3 
    STATS sig=C3,EMG,ECG
    ...
    ```

!!! Note
    The terms _signals_ and _channels_ are used interchangeably throughout this documentation.


The `sig` option works with the `alias` options also, i.e. 
```
luna s.lst sig="my-new-label,EMG,ECG" alias="my-new-label|EEG" -s DESC
```
Here, we've relabeled `EEG` as `my-new-label` and used `sig` to select it; the output from `DESC` reads:
```
Signals           : ECG EMG my-new-label
```

  
!!! danger "Restrictions for channel names" 
    Channel names should not have any of the following characters: comma, tab, new-line, pipe
    (|).  Ideally, there are advantages to not using any special
    characters (e.g. space, parentheses, asterisks, etc) or even
    things such as minus, plus signs, etc, as it makes it easier to
    specify channels on the command line and in scripts; it also can
    make processing the output much easier, i.e. if you use [`destrat`](destrat.md)
    to produce a data table where the channel labels are used to make
    variable names, you will want to restrict channel labels to
    alphanumeric characters and the underscore character as a
    separator. See this [FAQ](../faq.md#advice-on-channel-names). You can use Luna [_aliases_](#aliases) to make better channel names. 

### Search paths

As noted above, when parsing [_sample-lists_](#sample-lists) Luna can
use either absolute or relative paths for EDFs and annotation
file-names.  To use the same project on different computers, it is
often convenient to hard-code absolute path names, i.e.

```
id001	/home/joe/edfs/proj1/edf1.edf
id002	/home/joe/edfs/proj1/edf2.edf
id003	/home/joe/edfs/proj1b/edf3.edf
```

as this means that you can run Luna from any folder and it will still
know where to look for the EDFs. Alternatively, the `path` option (either on the
command line, or in a [_parameter file_](#parameter-files)) lets you
write a sample list in relative terms:

```
id001	proj1/edf1.edf
id002	proj1/edf2.edf
id003	proj1b/edf3.edf
```

One would then give the path separately, which will be added as a
prefix to all relative paths in the sample list:

```
luna s.lst path=/home/joe/edfs < commands.txt 
``` 

(i.e. otherwise Luna would not be able to find the EDFs unless it was run 
from the `/home/joe/edfs/` folder directly.)

This can also be convenient if the project is moved, i.e. if
`/home/joe/edfs/` becomes `/home/mary/projects/`, then you only need
change the `path` rather than recreate the sample list:

```
luna s.lst path=/home/mary/projects/ < commands.txt 
``` 



### Aliases

Aliases are different names for channels/signals in an EDF.  Aliases
can be useful if different EDFs in a project have different labels for
the same channel, e.g. `C3`, `C3-M1` and `C3-A1`.  Alternatively,
aliases can be useful if a channel has a long unwieldy name, perhaps
that contains spaces or other special characters.

Aliases can be specified either on the command line:

```
luna s.lst alias="C3|C3-M1|C3-A1" -s STATS sig=C3
```
or in a [_parameter file_](#parameter-files):
```
alias	C3|C3-M1|C3-A1
```

where the format is a series of `|`-delimited labels, with the first
being the _primary_ alias, or _canonical channel label_..  That is,
all subsequent terms are remapped to the primary alias in output
(i.e. `C3`).  When specifying channels, one can either use the
original EDF terms (e.g. `C3-M1`) _or_ the primary alias (`C3`). That
is, even if an EDF in this project only has a channel `C3-M1`, it will
still be included in any analysis that requires `C3`.  Similarly, in
all output (including new EDFs generated via the
[`WRITE`](../ref/outputs.md#write) command), the canonical channel
labels (_primary aliases_) will be swapped in whenever needed. (It is
not necessary that the canonical form exists in _any_ of the project's
EDFs: it could be an entirely new label you wish to apply to this
group of differently-named but otherwise identical channels.)

!!! Note
    Because most shell scripts interpret `|` as a special control character (pipe), we
    had to use quotes when specifying the `alias` on the command line above, i.e. `alias="C3|C3-M1|C3-A1"`. This 
    also implies that channel names should not contain `|` characters in.

### Remapping annotations

The `remap` option operates in the same way as the `alias` option, except for annotation labels instead of channel names.

For example, to change any annotation `REMS` to `REM`, add the following:
```
luna s.lst remap="REM|REMS" < cmd.txt
```
As with [aliases](#aliases), you can specify multiple, `|`-delimited remappings, i.e. in a many-to-one fashion.  Likewise, you
can put these in a [parameter file](#parameter-files) rather than write these out on the command line.  This will also be easier if
you annotations have spaces and special characters.  Note, you'll still need to use quotes if the labels have spaces: e.g.
```
remap      REM|REMS|"REM Sleep"|"Rapid eye movement sleep"
```
This will remap any of the three forms listed to the primary label: `REM`.

!!! warning "Automatric NSRR remappings"
    Note that Luna by defaults
    add in some _default_ annotaton remappings, to help working with
    NSRR data.  See [here](../nsrr.md#annotation-aliases) for more
    details.  This can sometimes mean that any remapping you specify
    conflicts with an internal one.  This is because, by definition,
    the same label cannot be both a primary value (i.e. to-be-mapped-to)
    as well as listed as an alias (to-be-mapped-from) value.  If you
    get an error message, then add `nsrr-remap=F` _before_ the `remap`
    you want to add. This will turn off all automatic remappings of
    annotations.  (These remappings are all listed on
    [this](../nsrr.md#annotation-aliases) page.)  Note that this is one of the few instances in which
    the _order_ of options is important, i.e. the `nsrr-remap=F` (which effectively clears the internal cache of mapping terms) must occur 
    _before_ new remappings are added (whether this is on the command line or via a [parameter file](#parameter-files).

### Spaces in channel and annotation names

As of v0.24, Luna will by default swap all spaces in channel or
annotation names with an underscore (`_`) character.  You can change the character swapped in
by setting the special variable `spaces`
```
luna s.lst spaces=/ < cmd.txt
```
Note: if the desired character would cause problems with the shell (e.g. an `*` character), then you can instead put
these assignments in a [parameter file](#parameter-files), e.g. `param.txt`:
```
spaces     *
```
```
luna s.lst @param.txt < cmd.txt
```
(In general, we do not recommend _swapping in_ special characters into channel and annotation labels though... the whole point of the
`spaces` option is to make these labels easier to interact with in a command-line environment.)

To turn off this functionality, set `keep-spaces=T`.  To turn it off
for _only_ channel labels use `keep-channel-spaces=T`.  To turn it off
for _only_ annotation labels use `keep-annot-spaces=T`.

Any aliases and remappings will be matched to both the pre-translated
(i.e. with spaces present) and post-translated (i.e. with spaces
potentially swapped out).


### Channel types

Based on the channel label, Luna will attempt to classify the _type_ of channel and automatically
assign corresponding variables (e.g. `${eeg}` and `${emg}`).  The current channel types are listed below: 

| Type | Variable | Description |
|---  | --- | --- |
|`IGNORE` | `${ignore}` | Flag to ignore these channels |
|`REF`| `${ref}` | Mastoid EEG reference |
|`IC`| `${ic}` | Independent components |
|`EOG` | `${eog}` | Electrooculogram |
|`ECG` | `${ecg}` | Electrocardiogram |
|`EMG` | `${emg}` | Electromyogram (chin) |
|`LEG` | `${leg}` | Leg EMG |
|`AIRFLOW`| `${airflow}` | Nasal or oral airflow |
|`EFFORT` | `${effort}` | Respiratory effort indicators (e.g. chest belts) | 
|`OXYGEN` | `${oxygen}` | Oxygen saturation (e.g. pulse oximetry) |
|`POSITION`| `${position}` | Position |
|`LIGHT` | `${light}` | Light channel | 
|`SNORE` | `${snore}` | Snore channel |
|`HR`| `${hr}` | Heart rate/pulse |
|`GENERIC`| `${generic}` | Generic channel (i.e. unknown) type |
|`EEG`| `${eeg}` | Any EEG channel |

Types are assigned based on channel names, as defined
[here](../resources/types.txt), and are set up for typical PSG and
sleep EEG studies, but can be altered by the user:

- Setting `ch-clear=Y` will clear the current list of type definitions

- `ch-match` can add a _partial match_ to the list of type definitions.  Partial matches are case insensitive and do not need
to fully match the channel label, e.g. `eeg` will match `EEG1`, `EEG A`, and `eeg_Cz`.

- `ch-exact` can add an _exact match_ to the list of type definitions.  Exact matches are case sensitive and do need to fully match the channel label, e.g. `eeg` will _only_ match `eeg` and __not__ `EEG1`, `eeg1`, etc.

Both `ch-match` and `ch-exact` expect a comma-delimited list of definitions, where each definition is a pipe-delimited list of a) the type, and b) one or more matches. For example, this assigns `A1` and `A2` as partial matches for the type `EEG`:
```
ch-match="EEG|A1|A2"
```
This example does as above, but additionally assigns `EKG1` to the type `ECG`:
```
ch-match="EEG|A1|A2,ECG|EKG1"
```

As with channel [_aliases_](#aliases), these are better placed in an include file, e.g. if the text file
called `param` param contains (two tab-delimited columns):
```
ch-match    EEG|A1|A2
ch-match    ECG|EKG1
```
then they will be included here:
```
luna s.lst @param < cmd.txt 
```

When determining channel type, Luna will attempt to match exact
matches first, and then partial matches.  Furthermore, within each
class of match, Luna will try to match types in the order as listed in
the table above (i.e. `IGNORE` first, then `EOG`, etc, until `EEG`).

Whenever new channels are added within a Luna run (e.g. via `ICA` or
`COPY`), a new type label will be assigned as appropriate, and the
corresponding variables (e.g. `${ic}` or `${eeg}`) will be updated.


### Exclude lists

To skip one or more EDFs in a project, you can create an exclude list:
a plain-text file with one ID per row.  You can then use the `exclude`
option to point to this file, either on the command line:

```
luna s.lst exclude=skip.txt -s DESC
```
or by specifying it in a [_parameter file_](#parameter-files)
```
exclude	skip.txt
```
These individuals/EDFs will be skipped in all analyses. 

### Include lists

Similar to [	exclude-lists](#exclude-lists) except this means that
only individuals in the specified file will be included, everybody
else will be excluded.  You cannot specify an `include-list` and
`exclude-list` together.


### Attaching annotations

To attach an annotation file without having to edit the sample-list,
you can use the `annot-file` (or equivalently `annot-files`,
`annots-file` or `annots-files`) option.
```
luna s.lst 1 annot-file=path/to/file.annot < cmd.txt
```
This will be most useful when working with a single EDF.


### Selecting annotations

To only load certain [annotation classes](../ref/annotations.md#luna-annotations) use the `annot` (or equivalently `annots`)
option.  For example, the following only loads classes labeled `wake`
and `artifact1`:
	
```
luna s.lst annots=wake,artifact1 < commands.txt 
```
To specify this in a [parameter file](#parameter-files):
```
annot          wake,artifact1
```

This can be useful if the sample-list otherwise specifies that many
annotations are loaded by default (e.g. by pointing to an annotation
folder for that individual/EDF).


### Other annotation options


| Option | Default | Description |
| --- | --- | --- |
|`force-edf` | F  | Skip EDF annotations _AND_ time-track from any EDF+ , and force a continuous EDF |
|`skip-annots` | F  | Skip XML and other (external) annotations |
|`skip-edf-annots` | F  | Skip `EDF Annotations` tracks from any EDF+  |
|`skip-all-annots` | F | Same as `skip-annots=1` and `skip-edf-annots=1` combined |


The `skip-all-annots` option (yes/no) indicates whether to skip
loading _all_ annotations (default no):

```
skip-all-annots    Y
```

The `skip-edf-annots` option (yes/no) indicates whether to skip
loading EDF annotations tracks (default no):

```
skip-edf-annots    1
```

The `epoch-len` command can be used to specify a default epoch
duration (in seconds) different from 30, which will be used when
attaching `.eannot` files specified in the sample-list (i.e.  to
calculate the implied number of epochs in the EDF).

```
epoch-len	20
```

### Fix truncated EDFs

Some exporters generate EDFs that do not have a complete final EDF
record; alternatively, file transfer may result in a truncated
file. For example, if the EDF record size is 1 second, but the total
recording length is, say, 20000.5 seconds, and so the last 0.5 seconds
does not constitute a full record, but _something_ is nonetheless written
to disk (i.e. the EDF shoule either be 20,000 or 20,001 seconds, in this case).

For these cases, where there is a small difference (typically < 1
record fewer than expected), you can add the `fix-edf=T` option to the
command line.  For example, here is an error encountered with a real
EDF as found in the wild:

```
luna s.lst -s DESC
```
```
Processing: id_XXXXXX [ #1 ]
 uniqifying Resp/Abd1-Gnd to Resp/Abd1-Gnd.1

 error : corrupt EDF: expecting 288820320 but observed 288817152 bytes
 details:
   header size ( = 256 + # signals * 256 ) = 5120
   num signals = 19
   record size = 7600
   number of records = 38002
   implied EDF size from header = 5120 + 7600 * 38002 = 288820320
 
   assuming header correct, implies the file has -0.416842 records too many
   (where one record is 1 seconds)
```

Luna then goes on to give the following suggestion - but stressing the point that it
is only an _assumption_ that the data are truncated (i.e. and otherwise okay).  (That is,
there could be other reasons why the EDF size does not match that expected based on the EDF headers.)

```

 IF you're confident about the remaining data you can add the option:
 
    luna s.lst fix-edf=T ... 
 
  to attempt to fix this.  This may be appropriate under some circumstances, e.g.
  if just the last one or two records were clipped.  However, if other EDF header
  information is incorrect (e.g. number of signals, sample rates), then you'll be
  dealing with GIGO... so be sure to carefully check all signals for expected properties;
  really you should try to determine why the EDF was invalid in the first instance, though
```
Re-running but with the `fix-edf=T` option added:
```
luna s.lst fix-edf=T -s DESC
```
allows Luna to proceed (with a warning)
```
  assuming header correct, implies the file has -0.416842 records too many
  (where one record is 1 seconds)
 
  attempting to fix this, ...
    changing the header number of records from 38002 to 38001 ... good luck!
 
 duration: 10.33.21 | 38001 secs ( clocktime 20.38.43 - 07.12.03 )
 
 signals: 19 (of 19) selected in a standard EDF file:
  F3-Ref | F4-Ref | C3-Ref | C4-Ref | O1-Ref | O2-Ref | A1-Ref | A2-Ref
  ...

```

In other words, _proceed with care if you find yourself having to use
this option..._


### Force evening start time

By default, Luna assumes a 24-hour clock format, and so a start time
in the EDF of 8:00 means 8am and not 8pm.  If you suspect an EDF has
used a 12-hour time format instead (which does not properly specify
whether it is AM or PM), you can set the variable `assume-pm-start=X`,
meaning that Luna will assume a 12-hour clock but convert all times
between _X_ AM and noon to _X_ PM.  For example:

```
 assume-pm-start=4
```

means that EDF header start times of `04:00`, or `5:30` will be
converted to `16:00` and `17:30` respectively.  However, `1:30` will
remain as `01:30`.  That is, this function works on the assumption of
"typical" bedtimes being in the late evening or very early morning.
If the EDFs use 24-hour notation correctly, then you should have no
reason to use this option.

### Spectral power bands

The following special variables can be changed on the command line or
via a [parameter file](#parameter-files):

| Band name | Definition | 
| ---- | ---- |
| `slow`  | Slow power band (default 0.5-1 Hz) |
| `delta` | Delta power band (default 1-4 Hz) |
| `theta` | Theta power band (default 4-8 Hz) |
| `alpha` | Alpha power band (default 8-12 Hz) |
| `sigma` | Sigma power band (default 12-15 Hz) |
| `beta`  | Beta power band (default 15-30 Hz) |
| `gamma` | Gamma power band (default 30-50 Hz) |
| `total` | Total power band, denominator for relative power (default 0.5-50 Hz) |

For example, to change the definition of sigma power to 11 to 15 Hz (instead of 12 to 15 Hz):

```
luna s.lst sigma=11-15 -s "PSD sig=C3,C4"
```



## Output 

### Logs

Luna writes a log to the standard error (`stderr`) stream, i.e. the
console, that reports the progress of a given command.  This will
display the version of Luna, the date and time the command was
initiated, and the input commands.  For each EDF processed, Luna will
then list the ID along with some basic information about the EDF, its
attached annotations and the commands applied to that EDF.  Any error
messages will be sent to the `stderr` also.

```
===================================================================
+++ luna | v0.2, 12-Dec-2018 | starting process 2019-01-10 14:20:10
===================================================================
input(s): s.lst
output  : .
commands: c1	DESC	

___________________________________________________________________
Processing: nsrr01 [ #1 ]
 total duration 11:22:00, with last time-point at 11:22:00
 40920 records, each of 1 second(s)

 signals: 14 (of 14) selected in a standard EDF file:
  SaO2 | PR | EEG(sec) | ECG | EMG | EOG(L) | EOG(R) | EEG
  AIRFLOW | THOR RES | ABDO RES | POSITION | LIGHT | OX STAT

 annotations:
  [Arousal ()] 194 event(s) (from edfs/learn-nsrr01-profusion.xml)
  [Hypopnea] 361 event(s) (from edfs/learn-nsrr01-profusion.xml)
  [NREM1] 109 event(s) (from edfs/learn-nsrr01-profusion.xml)
  [NREM2] 523 event(s) (from edfs/learn-nsrr01-profusion.xml)
  [NREM3] 16 event(s) (from edfs/learn-nsrr01-profusion.xml)
  [NREM4] 1 event(s) (from edfs/learn-nsrr01-profusion.xml)
  [Obstructive Apnea] 37 event(s) (from edfs/learn-nsrr01-profusion.xml)
  [REM] 238 event(s) (from edfs/learn-nsrr01-profusion.xml)
  [SpO2 artifact] 59 event(s) (from edfs/learn-nsrr01-profusion.xml)
  [SpO2 desaturation] 254 event(s) (from edfs/learn-nsrr01-profusion.xml)
  [Wake] 477 event(s) (from edfs/learn-nsrr01-profusion.xml)
 ..................................................................
 CMD #1: DESC

___________________________________________________________________
...processed 1 EDFs, done.
...processed 1 command(s),  all of which passed
-------------------------------------------------------------------
+++ luna | finishing process 2019-01-10 14:20:10
===================================================================
```


### Default text output

Beyond the log information described above, if no database option is
specified (with `-o` or `-a`) then, by default, all primary _output_
goes to `stdout`, i.e. the console/terminal, and will be interleaved
with the logging information.  That is, here _output_ refers to the
information produced by individual Luna commands, rather than the log
information _per se_.

This is typically not very useful. You can _redirect_ the output to a
separate file, say `out.txt`:

``` 
luna s.lst < commands.txt > out.txt 
``` 

Whilst a few commands such as `DESC` or `SUMMARY` produce their own
format of output (simple text formatted for human reading), most
commands adopt the same output framework, such that the same
information can be channeled to either a text file (described here), a
database (described [next](destrat.md)) or even an object in R if
using the Luna [R extension library](../ext/R/index.md).

Using the `HEADERS` command as an example on the first EDF in the [tutorial](../tut/tut1.md) dataset: 

```
luna s.lst nsrr01 sig=ECG,EMG  -s HEADERS > out.txt
```

The `out.txt` file contains the following: 

```


nsrr01	HEADERS	.	.	NS	2
nsrr01	HEADERS	.	.	NR	40920
nsrr01	HEADERS	.	.	REC.DUR	1
nsrr01	HEADERS	.	.	TOT.DUR.SEC	40920
nsrr01	HEADERS	.	.	TOT.DUR.HMS	11.22.00
nsrr01	HEADERS	.	.	EDF_ID		
nsrr01	HEADERS	.	.	START_TIME	21.58.17
nsrr01	HEADERS	.	.	START_DATE	01.01.85
nsrr01	HEADERS	CH/ECG	.	SR		250
nsrr01	HEADERS	CH/ECG	.	PDIM		mV
nsrr01	HEADERS	CH/ECG	.	PMIN		-1.25
nsrr01	HEADERS	CH/ECG	.	PMAX		1.25
nsrr01	HEADERS	CH/ECG	.	DMIN		-128
nsrr01	HEADERS	CH/ECG	.	DMAX		127
nsrr01	HEADERS	CH/EMG	.	SR		125
nsrr01	HEADERS	CH/EMG	.	PDIM		uV
nsrr01	HEADERS	CH/EMG	.	PMIN		-31.5
nsrr01	HEADERS	CH/EMG	.	PMAX		31.5
nsrr01	HEADERS	CH/EMG	.	DMIN		-128
nsrr01	HEADERS	CH/EMG	.	DMAX		127
```

Each row is one value for one variable from one command.  In this
case, only the `HEADERS` command was performed.  The six tab-delimited
columns are:

- Individual/EDF ID
- Command name
- Any stratifying factors (or `.` if not)
- Any epoch/interval time factors (or `.` if not)
- Variable name
- Value

_Stratifying factors_ mean that the same _variable_ is repeated in
different contexts: for example, for different channels in the same
EDF.  This is represented above by the factor (channel, `CH`) and the
two levels (`ECG` and `EMG`).  Given that `SR` is the sample rate for
a channel, we can see that the ECG channel has a sample rate of 250
Hz, whereas the EMG has a sample rate of 125 Hz, for example. The
first variables, e.g. `NR` and `NS` do not have any stratifying
factors (`.` in columns 3 and 4) as these are general properties of
the entire EDF, and so only occur once.

This format is primarily used for debugging or in some other very
focused cases.  Although it is relatively easy to parse, in general
you'll want to use Luna's [text-tables](#text-tables) or [_lout_ databases](#lout-databases),
described next.

### Text-tables 

Text-table output format creates a folder for every individual/EDF
analysed, containing one plain-text file (tab-delimited, rectangular
file with a header row) for every output strata of every command
performed.

Text-table output mode is engaged by adding `-t` to the command line,
followed by the name of a root folder (this will be created if it does
not exist). For example:

```
luna s.lst -t out1 -s 'MASK ifnot=NREM2 & RE & PSD spectrum sig=EEG'
```

will create a folder `out1/`.  Applied to the [tutorial
data](../tut/tut1.md), this would create the following three subfolders, each named by the ID of the individual/EDF:

```
ls out1/
```
```
 nsrr01  nsrr02  nsrr03
```

Each subfolder will typically contain a set of identical files, reflecting the analyses that were performed:

```
ls out1/nsrr01
```
```
 MASK-EPOCH_MASK.txt  PSD-B,CH.txt  PSD-CH.txt  PSD-F,CH.txt  RE.txt 
```

Certain commands that tend to generate very large output files
(e.g. `PSD epoch-spectrum`) will generate compressed (gzipped) output
files by default (with extension `.txt.gz`).  To force _all_ output
files to be compressed, set the `compressed` special variable to true
(`Y` or `1`):

```
luna s.lst -t out1 compressed=Y < s.lst
```

Alternatively, to set _all_ output files to not be compressed, set
`compressed` to false (`N` or `0`).

!!! warning "Known issues"
    Text-tables are provisionally introduced in Luna v0.23; please see [this link](destrat.md#text-tables) for some known issues with the initial implementation of the `-t` flag.

### _lout_ databases

This is the primary mode of output for most Luna commands.  Here we
run the same command as in the [previous section](#default-text-output)
but instead using a [_lout_ database](destrat.md) to collect the
output, with `-o`:

```
luna s.lst nsrr01 sig=ECG,EMG -o out.db -s HEADERS 
```

This generates a file `out.db` (actually an
[SQLite](http://sqlite.org) database) which is not designed to be
directly displayed in the terminal via a text-editor or spreadsheet.
Rather, it is an intermediate form, from which various text-files can
be extracted in a variety of formats.  See [this page](destrat.md) for
information on how to work with output databases, using either
[destrat](destrat.md) or [_lunaR_](destrat.md#lunar).

By default, Luna will overwrite an existing database; use `-a` instead
of `-o` to _append_ to an existing database.


!!! info "Which should I use: text-tables or data-bases?"  
    Advantages
    of the database output format is that all information (potentially from multiple people) is contained
    in a single place, and can be queried with `destrat`.  Databases
    can also be loaded directly into R, using
    [`ldb()`](../ext/R/ref.md#ldb). _Disadvantages_ are 1) for very large
    projects or output files, performance can suffer, and 2) to export
    into other software (other than R via _lunaR_), you first need to
    extract the information as an intermediate text file.  Text-table
    mode (`-t`) effectively is the same as dumping (via `destrat`) all
    possible tables from a given database; for certain types of large
    output files, this can be advantageous.  The _disadvantage_ of
    text-table mode is that results are not automatically compiled
    across individuals: if your project contains 100 individuals,
    you'll have 100 separate subfolders, each with the same text-table
    files.  The _lunaR_ package provides the function
    [ltxttab()](../ext/R/ref.md#ltxttab) that can help by automatically compiling output across
    different individuals/subfolders.  

    In general, use text-table mode
    to improve performance and avoid duplicating information for
    commands that generate very large output files (e.g. `PSD
    epoch-spectrum` for EDFs with many channels; typically, one is
    less interested in combining this type of output across different
    individuals in any case).  See the [tutorials](../tut/tut1.md) 
    for examples of using both formats.
 

### New EDFs

Luna can output new EDFs (or EDF+ files, or [compressed EDFs](#edfz))
after manipulating signals, masking epochs, etc, via the
[`WRITE`](../ref/outputs.md#write) command.

### Annotations

Some commands, such as [`WRITE-ANNOTS`](../ref/annotations.md#write-annots),
can produce [.annot files](../ref/annotations.md#annot) containing interval-based annotations.

### Misc text-file dumps

Some commands produce flat-file text output distinct from the usual
output mechanism (via the database or text-tables, as described
above), for example `MATRIX` or `ICA`.  These outputs can often be quite large files.



