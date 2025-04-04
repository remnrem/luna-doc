# Key concepts & syntax

## Overview

This page introduces some key concepts for using Luna.  Although
introduced in terms of the _lunaC_ command-line version of Luna, almost all
core concepts are applicable in the Python and R versions too. This
page contains a lot of detail, covering three main areas:

 - [basic syntax](#luna-syntax) for invoking Luna and [specifying inputs and scripts](#inputs)

 - a list of [_special variables_](#special-variables) -- options that can be set to control the behavior of Luna

 - an overview of the [output mechanisms](#outputs)
 
!!!hint "Prerequisites"
    Luna is fundamentally a console/command-line package,
    i.e. there is no _point-and-click_. Familiarity with the basic
    Unix/macOS console environment and shell scripting is recommended.
    The walk-through tutorial contains a brief [shell orientation](https://zzz.bwh.harvard.edu/luna-walkthrough/prep/#shell-orientation) section
    that outlines some core commands useful when using Luna (or similar command-line tools).  Naturally, there is a huge amount of easily accessible online tutorial information
    to getting started with the command shell too...

## Luna syntax

Once [installed](../download/index.md) and in your command path,
_lunaC_ is invoked via the `luna` command, often in the form:

``` 
luna sample.lst -o out.db < commands.txt
```

Here, Luna expects a list of IDs, EDFs (and possibly [annotation
files](#annotations)) in a [sample list](args.md#sample-lists) file
(`sample.lst`), reads a series of [commands](../ref/index.md)
(`commands.txt`) to be applied to each EDF, and writes the output to a
[_lunout_](destrat.md) database file (`out.db`).

!!!note "Types of command line arguments"
    _lunaC_ expects the first
    argument to be either 1) an EDF (or EDF+ or [EDFZ](#edfzs)) file,
    2) a [_sample-list_](#sample-lists), 3) a plain-text ASCII file or
    4) a special command that does not require signal data, e.g. such
    as [`--build`](#-build-option) or
    [`--xml`](../../ref/helpers/#-xml).  With the exception of command scripts specified by `-s`, 
    all subsequent arguments can come in any order and are interpreted as follows:

    - terms containing `=` are interpreted as
      [_variables_](#variables), with the exception of certain
      [special _reserved_ names](#special-variables) that change the behaviour of Luna, 
      e.g. `annot` or `alias`, as described [below](#special-variables)
    
    - terms starting with `-` are interpreted as [options](#options),
      e.g. primarily `-o` and `-s`

    - once an `-s` option is encountered, _all_ subsequent terms are
      interpreted as Luna [_commands_](../ref/index.md) rather than
      command line options; thus, if given, `-s` (and any subsequent commands) must be the
      final argument

    - terms starting with `@` are interpreted as [_parameter
      files_](#parameter-files), the contents of which are loaded in
      to define/set new (special) variables

    - otherwise, terms that are numbers are interpreted as sample-list
      [_row numbers_](#ranges) (either a single row, or a range,
      depending if one or two numbers are specified)

    - otherwise, that term is assumed to be the ID of a _single_
      individual to be analysed from the sample-list

    - if specifying one or more IDs on the command line, it is best to use the
      special variable form `id` (which can handle purely numeric IDs,
      i.e. interpreting it as an ID rather than than a row number in a
      sample list )


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

## Options

As well as specifying the main input, setting any variables and
supplying Luna commands to be processed, the following flags can also
appear on the Luna command line.

Primary options:

| Option | Description |
|--------|----------|
| `-s`   | Directly specify Luna commands after `-s`, rather than reading them from a command file; if specified, this option must come last |
| `-o`   | Write all output to a [lunout](#lunout-databases) database; if it already exists, it will be overwritten |
| `-a`   | Similar to `-a` except appends to an existing output database rather than overwriting it |
| `-t`   | Write all output as [text tables](#text-tables) rather than to a database |
| `@<file>` | Read (special) variables from a parameter file, e.g. `@param.txt` |
| _N_ | Read only observation _N_ (_nth_ row) from the sample list |
| _N_ _M_ | Read only observations _N_ to _M_ from the sample list |
| `id=<id>` | Read only observation with sample list ID matching `<id>` from the sample list, e.g. `id=night001` |

Special-case, secondary options:

| Option | Description |
|--------|----------|
| `--fs` | Specify a sample rate when reading an ASCII (non-EDF) file |
| `--nr` | Specify the number of records when creating an [empty EDF](#empty-edfs) |
| `--rs` | Specify the record duration (seconds) when creating an [empty EDF](#empty-edfs) |
| `--opt` or `--options` | Only applicable when using a special helper/utility function such as `--psc`; options to be passed to that command come after this |

## Input 

### EDFs

Luna can read EDF and EDF+ files.  The latter allow for
discontinuities (by having an explicit time-track) and provide some
support for annotations.  After
[restructuring](../ref/index.md#restructure) a file (e.g. removing
certain epochs), it will be represented internally in a form that
corresponds to an EDF+ (i.e. with gaps/discontinuities).
Luna can read a single EDF (rather than a [sample
list](args.md#sample-lists)) by specifying a filename (with an `.edf`
or `.EDF` extension - or alternatively, a compressed [`.edf.gz`](#edfzs)):

```
luna test1.edf < commands.txt
```

To attach annotations to a single EDF, use `annot-file` (although [sample-lists](#sample-lists) are generally
more convenient to link EDF and annotation files, as described below):
```
luna test1.edf annot-file=test1.annot < commands.txt
```

<h4>In-memory vs on-disk representations</h4>

In this documentation, we often make a distinction between the
_on-disk_ EDF and the _in-memory_ (_internal_) representation of the
EDF.

 - When Luna first "attaches" an EDF, it does not load anything other
   than the header.  Subsequently, Luna uses a _lazy-loading_
   approach, whereby it only pulls records from disk when needed.
   These are cached in memory: if they are needed again, they
   do not need to be read in afresh.

  - All commands that manipulate EDFs and the signals therein operate
   only on the _internal_ EDF.  Although we may use language such as "drop a
   channel from the EDF" we generally do not imply that the original
   _on-disk_ file has been changed.

 - The _in-memory_ EDF may differ from the _on-disk_ version, for
   example, if channels and/or epochs have been dropped or added by
   `SIGNALS`, `MASK`, `RESTRUCTURE` and other commands.  As noted,
   all Luna commands report on the current, _in-memory_ representation:
   for example, the original EDF may have 64 channels, but 
   [`DESC`](../ref/summaries.md#desc) may report fewer (or more) depending if
   channels have been dropped/added _within that particular Luna run_.

  - __For command-line Luna__, changes made to the _in-memory_ EDF do not persist after that run. 
    For example, the following may report that 6 channels are present:

    ```
    luna my.edf -s DESC 
    ```
    One could subsequently run a command that drops two channels and then calls `DESC`: 
    ```
    luna my.edf -s 'SIGNALS drop=ECG,EMG & DESC'
    ```
    in which case, only 4 channels are reported. However, on 
    running the first command again:
    ```
    luna my.edf -s DESC
    ```
    Luna will report 6 channels again,  i.e. as nothing was changed in the original file `my.edf`.
    (Note that the [Python](../lunapi/index.md) and [R](../ext/R/index.md) packages differ here, meaning
    that the same _in-memory_ EDF persists between commands.)

### Sample lists

The sample list (`sample.lst` in the example at the top of this page)
file defines a _project_, i.e. a collection of EDFs and their
associated IDs and [annotations](#annotations). Sample lists should
contain at least two tab-delimited fields: the subject ID, followed by
the EDF file location:

```
id001	test1.edf
id002	test2.edf
```

An optional additional column(s) specifies any [_annotation files_](#annotations)
(e.g. containing stage information) for that EDF.  As in the example below,
you can use either absolute or relative file paths:
```
id001	test1.edf	/data/annots/test1-staging.xml
id002	test2.edf	staging2.xml	
```

Relative paths are evaluated relative to the directory that Luna is
run from.  On a single system, and if files won't be moved often, it
is better to use absolute paths.  See [below](args.md#search-paths)
for notes on using relative paths, which can be convenient in
some circumstances.  Also see the [`path`](#search-paths) special variable and
the [`--repath`](../ref/helpers.md#-repath) helper function that can be used to
quickly search/replace paths in a sample list.

In place of an annotation file, it is possible to specify a _folder_
to be searched (but not recursively) for annotation files (those with
extensions `.annot`, `.txt`, `.tsv`, `.eannot` and `.xml`):

```
id001	test1.edf	/data/annots/indiv/id001/
```

Multiple annotation files (or folders) can be listed as separate
tab-delimited fields (i.e. columns 3, 4, 5, etc), or as a
_comma-delimited_ list in the third column.  If there are no
annotation files, you can put a period (`.`) character.  These last two
rules allow for a clean three tab-delimited column scheme
irrespective of the number of annotation files. This can make sample
lists easier to work with (e.g. to load into R as a tab-delimited
file with a regular, rectangular structure):

```
id001	test1.edf	test1.tsv
id002	test2.edf	.
id003	test3.edf	test3.tsv,staging-id003.eannot
```

!!! warning 
    Sample lists must be tab-delimited plain-text/ASCII files,
    i.e. not containing any special characters or
    formatting.  Use a text-editor (not or word processor) or generate them programmatically (see [`--build`](#-build-opion)).
    Also be aware of potential issues that [Windows-style
    line-ending characters](../faq.md#line-endings) can cause.

### _--build_ option

Luna's `--build` option can generate [_sample-lists_](#sample-lists)
automatically, by recursively scanning one or more folders (and their
subfolders) to find EDF files (ending `.edf`, `.EDF`, or `.edf.gz` etc) and
_associated_ [annotation files](#annotations) (e.g. `.annot`, `.xml`, or a
user-specified alternative).  By default, an annotation file is said to be
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
will scan these two folders, matching `f1.edf` with `f1.xml`, etc, to generate a sample-list (written to standard output) with three entries:
```
cat s.lst
```
```
f1   edfs/f1.edf   xmls/f1.xml
f2   edfs/f2.edf   xmls/f2.xml
f3   edfs/f3.edf   xmls/f3.xml
```

Note that you can specify any number of folders for `--build` to search, and each folder can contain EDFs, annotation files or both.

By default, Luna looks at the _filename_ of the EDF to determine the
ID (i.e. what will be placed in the first column of the sample list).
Alternatively, you can instruct `--build` to use the identifier in the
EDF header (i.e. which may or may not be the same as the filename of the EDF)
to be the sample list ID, by adding the option:
```
 -edfid
```

In general, we advise equating the EDF filenames with IDs is better,
as this tends to encourage better alignment between the other files
associated with that individual/recording.

<h5>Matching across folders</h5>

If the EDFs and the XML annotations were in the same folder (say `mydata`), the following
would still work as intended:
```
luna --build mydata > s.lst
```

Alternatively, different folders might contain distinct EDFs but that
have similar file names. For example, we may have two subjects in two folders:

```
ls subj1 subj2
```
```
subj1:
night1.edf	night1.xml	night2.edf	night2.xml

subj2:
night1.edf	night1.xml	night2.edf	night2.xml
```

In this scenario, we would not not want to associate `subj1/night1.edf` with
`subj2/night1.edf` or `sub2/night1.xml` (which `--build` would do by default, based
on the root of the filenames). Here, adding the option `-nospan` instructs Luna not to
_span_ folders when associating files:
```
luna --build subj1 subj2 -nospan -edfid > s.lst 
```
```
s1night1	subj1/night1.edf	subj1/night1.xml
s1night2	subj1/night2.edf	subj1/night2.xml
s2night1	subj2/night1.edf	subj2/night1.xml
s2night2	subj2/night2.edf	subj2/night2.xml
```

In this example, the IDs (`s1night1`, etc) come from the EDF headers
as we used the `-edfid` option.  If the IDs weren't unique, Luna will
print a warning message about dulplicates.  That is, every row of a
sample-list should have a unique ID.

<h5>Annotation extensions</h5>

To specify special extensions for annotation files (i.e. other than
`.annot`, `.xml` etc), add the `-ext` option: e.g. to match on `.dat`
files:

```
luna --build mydata -ext=dat > s.lst
```

By default, `--build` will match potential annotation files based on
`.xml`, `.annot`, `.eannot`, `.txt` and `.tsv` extensions.  You can
specify a comma-delimited list of multiple extensions with `-ext`.

It can be convenient to specify extensions if annotation files have a
regular naming scheme but do not match the EDF files identically.  For
example, many NSRR files have EDF and annotation files in the form:

```
file1.edf     file1-nsrr.xml
file2.edf     file2-nsrr.xml
```
To associate these files, add the full tag and extension: 
```
luna --build mydata -ext=-nsrr.xml > s.lst
```
i.e. this matches __file1__[.edf] with __file1__[-nsrr.xml].

<h5>Absolute and relative paths</h5>

Whether or not the resulting sample-list uses relative or
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

### Command files

Command files or _scripts_ (i.e. `commands.txt` in the example 
at the top of this page), contain one or more [Luna commands](../ref/index.md).  This
can be more flexible, convenient (and reproducible) than placing all commands after
the `-s` argument.

Some conventions:
 
 - new commands should start on a new line
 - lines that begin with one or more spaces are assumed to be continuations of the same command, meaning that 
   commands can be spread over several lines
 - blank lines are skipped
 - all text after a `%` character on a line is treated as a _comment_ and skipped 

Say we wished to [`EPOCH`](../ref/epochs.md#epoch) an
EDF and apply power spectral density estimation via the
[`PSD`](../ref/power-spectra.md#psd) command for a channel named
`EEG`, outputting spectra for each epoch.  For a sample-list `s.lst`
and [output file](#lunout-databases) `out.db`, we might write:

```
luna s.lst -o out.db -s ' EPOCH & PSD epoch sig=EEG '
```

Note that we place the command string in single quotes, as otherwise
the `&` character would be interpreted by most shells, to mean that
the job should be run in the background.  In general, putting any
command-line script after `-s` in single quotes is advised.  (Single
versus double quotes has implications for how you want variables to be interpret,
as shell variables versus Luna variables, as described [below](#variables).)

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

Alternatively, we would achieve the same outcome using a more
verbose script, with ample comments and spreading commands
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

!!!info "Multi-line statements using `-s`"
    It is also possible to specify multiline scripts with the `-s` option.  First, _different commands_ can be separated by `&` or new lines:
    ```
    luna s.lst -o out.db id=person1 -s ' HEADERS & SEGMENTS '
    ```
    is the same as
    ```
    luna s.lst -o out.db id=person1 \
      -s ' HEADERS
           SEGMENTS '
    ```
    and is also the same as
    ```
    luna s.lst \
      -o out.db \
      id=person1 \
      -s ' HEADERS
           SEGMENTS '
    ```
    where `\` is the shell line-continuation character.  Note, it is not needed between HEADERS and SEGMENTS as the Luna script is quoted; also
    note that `&` is not needed as the commands are on different lines.

    Second, a _single command command_ can be split onto multiple lines when using `-s`, by using
    the `\` line continuation symbol (which, as with the shell, must be the _last_ character on that line, no trailing spaces):
    ```
      -s ' HEADERS \
            signals
           SEGMENTS '
     ```
     This can be convenient if the command has a very long option list.
     
     Note, you __don't__ want or need to put a `\` between _different_ quoted Luna commands: if you did, then
     ```
       -s ' HEADERS \       	    
            SEGMENTS '
     ```
     would be read as
     ```
       -s ' HEADERS SEGMENTS '
     ```
     i.e. without the implicit `&`. In this case, you could choose to write 
     ```
       -s ' HEADERS \
            & SEGMENTS '
     ```
     although this is unnecessary when the following achieves the same result: 
     ```
       -s ' HEADERS 
            SEGMENTS '
     ```
     that is, internally, this is passed to Luna as
     ```
       -s ' HEADERS & SEGMENTS '
     ```




### Variables

In a [command file](#command-files), a Luna variable _`var`_ is denoted by
the following syntax:

```
${var}
```

Building on the previous section's example script, we could
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
If _l_ or _s_ were not specified, Luna would give an error.

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

You can also use the _append_ syntax with the `+=` operator which generates a _comma-delimited_ string, for example: 
```
${var=a}
${var+=b}
${var+=c,d}
```

would result in `${var}` equalling `a,b,c,d`.  This is used primarily in
the context of the
[CANONICAL](../ref/canonical.md#canonical-signal-definitions) command.

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

That is, in the second scenario, Luna "sees" `${eeg}` as the `sig`
option for `STATS`, and so figures out what replacement is desired
(e.g. that might be based on data from a [vars](#individual-variables)
file, or from a default built-in definition, as is the case for
`${eeg}`).  In contrast, in the first (double-quote) scenario, the
shell (and not Luna) will attempt to figure out what the `${eeg}`
variable means: unless otherwise specified, `${eeg}` will not be
defined by the shell/prior script, and so Luna will simply see nothing
instead of `${eeg}`, i.e. `sig=`.

Inside a command/script, Luna variables have the form `${l}`, whereas
when setting that variable on the command line, one just writes
`l=20`, for example.  Instead of a fixed value (`20`), one can also
set a Luna variable to equal a shell variable, which might even have
the same name. Rather than setting `l` to a fixed value of `20`, it would also be
possible to use a _shell_ variable to set the Luna variable `l` - and
the shell variable might even have the same name (`l`):
```
luna s.lst l=${l} s=EEG -s ' EPOCH len=${l} '
```

In the above, the first `${l}` is the shell variable `l`, whereas the
second `${l}` is the Luna variable `l`.


!!! info "Best practice"
    At least when using bash as a shell, single quotes after `-s` is the most likely best practice.  If you
    want to pass in shell variables to Luna, use the form:
    ```
    luna s.lst x=${x} -s ' COMMAND option=${x} '
    ```
    To make the distinction between shell and Luna variables clear, assuming shell variable `${x}`
    the above is equivalent to:
    ```
    luna s.lst y=${x} -s ' COMMAND option=${y} '
    ```
    which is also equivalent to
    ```
    luna s.lst -s " COMMAND option=${x} "
    ```
    (although this latter form is not advised as it arguably blurs the distinction between shell and Luna variables)
        

### Conditional blocks

There are two ways to specify conditional blocks: the `IF` command,
and `[[` blocks. The preferred mechanism is `IF` because it a) is
clearer to read, and b) it is sensitive to variables that are defined
dynamically (i.e. during execution of the script).

<h5>`IF`/`FI` commands</h5>

Within a command file, you can execute only certain parts of the code
depending on the value of a variable.  Specifically, 

 - _empty_, `0`, or values starting with `N` or `F` (i.e. _no_ or _false_) all evaluate to _false_ / _null_

 - all other values evaluate to true (non-null) 


The commands here will only be evaluated if `${x}` is non-null:
```
IF x

 ... some commands ....
 ... some commands ....

FI 
```

The term `ENDIF` can be used instead of `FI`.    Here, the commands within that block will be executed here:
```
luna s.lst x=T < cmd.txt 
```
but not here:
```
luna s.lst x=F < cmd.txt 
```
or here:
```
luna s.lst < cmd.txt 
```

Importantly, `IF` is responsive to variables defined within the script:

```
${x=T}

... other commands ....

IF x

 ... some commands ....
 ... some commands ....

FI

```

This can be useful if the variable is dynamically set by the script (e.g. from the [`CONTAINS`](ref/summaries.md#contains) command). 


<h5> `[[` blocks </h5>

_This syntax is supported for historical reasons only: using the `IF` command is preferred._

Within a command file, you can define blocks that are only executed if
a [variable](#variables) is set to a non-null value, e.g. `1`.

Importantly, these blocks are evaluated _when first reading the script_: this means they
are not sensitive to variables defined on-the-fly (i.e. within a script, unlike the `IF` command.

If the variable is null (undefined or `0`) those blocks are skipped. This
uses the following double-bracket syntax:

```
EPOCH len=${l}

[[var

  SIGSTATS sig=${eeg} mask th=${thresholds}

]]var

PSD sig=${eeg}
```

If `${var}` is null, then _all text_ (i.e. including any other
variable definitions and conditional statements as well as commands)
will be skipped, up until the end of the block (here `]]var`).  In this
example, the `SIGSTATS` command will only be executed if `${var}` has
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



### Sequence expansion

Luna scripts support a simple convenience feature to specify
regularly-structured sequences of items, for places where Luna is
expecting a comma-delimited list of entries (e.g. for `sig` or `annot`
etc).  This is done using the form `[x][y]` where _x_ and _y_ are of
the form:


 - a single value, e.g. `ch-`

 - a comma-delimited list, e.g. `a,b,c`

 - an `n:m` integer numeric sequence, e.g. `1:3` (which implies `1,2,3`)

The two lists are expanded as follows:

 - `[a][1:3]` = `a1,a2,a3`

 - `[a][x,y,z]` = `ax,ay,az`

 - `[a,b,c][1:3]` = `a1,a2,a3,b1,b2,b3,c1,c2,c3`

These can also accept [variables](#variables). For example, the command:

```
HILBERT sig=${eeg} f=11,15
```

will add a new channel for every channel in `${eeg}` with a
default suffix `_ht_mag`, e.g. `C3_ht_mag`, `C4_ht_mag`, etc, reflecting the instantaneous magnitude (envelope)
from the filter-Hilbert transform.

If one wants to subsequently refer to the whole group of these new
channels, you can write `[${eeg}][_ht_mag]`, i.e. which expands to
`C3_ht_mag,C4_ht_mag` etc.

Similarly, it can sometimes be convenient to automatically specify a
series of channels, e.g for the output of [ICA](../ref/cc.md#ica),
where new channels are created as `ICA1`, `ICA2`, etc.  Use the form
`[root][1:3]` to obtain a comma-delimited list `root1,root2,root3`. For example:

```
PSD sig=[ICA][1:5]
```
is identical to
```
PSD sig=ICA1,ICA2,ICA3,ICA4,ICA5
```

As above, this can be combined with a variables to specify the number
of components:

```
PSD sig=[ICA][1:${k}]
```

```
luna s.lst k=10 < cmd.txt
```


### Individual variables

As well as _run-level_ variables that are common to all
individuals/EDFs processed, you can assign values on an individual-by-individual basis,
using values stored in a text file - here called _vars files_.

For example, the following _vars file_ defines three variables `${var1}`, `${var2}` and `${var3}`
for the [tutorial](../tut/tut1.md) individuals:
```
ID       var1    var2       var3
nsrr01   22      EEG1,EEG2  T
nsrr02   12      EEG1       F
nsrr03   98      .          T
```

A _vars file_ should:

  - be a ASCII, plain-text file (no special characters, etc)

  - have a header row that includes the column `ID` in the first field

  - be tab-delimited, with the same number of columns on each row

Variables can be given any valid name: i.e. they do not need to be `var1`, `var2`, etc,
but they should not include special characters and they are case-sensitive. You then
include these variables by setting the `vars` special variable to point to the file(s). If
the above file is named `indiv.dat`:

```
luna s.lst vars=indiv.dat < my-commands.txt
```
You can have multiple `vars` statements, or pass `vars` a comma-delimited list of multiple files to include.


If the command script contained references to `${var1}`, etc, they
would be substituted as appropriate for each individual. For example,
all the EDFs had channels `EEG1` and `EEG2`, the following command would
run the `PSD` command 1) for both, 2) only for `EEG1`, and 3) neither, for
the first, second and third individual respectively:
```
PSD sig=${var2}
```

The log summaizes any attached variables for each individual: e.g. for the first individual (`var1`, etc are
listed at the end):
```
 variables:
  airflow=AIRFLOW | ecg=ECG | eeg=EEG(sec),EEG | effort=THOR_RES,A...
  emg=EMG | eog=EOG(L),EOG... | hr=PR | id=nsrr01 | light=LIGHT
  oxygen=SaO2,OX_STAT | position=POSITION | var1=22 | var2=EEG1,EEG2 | var3=T
```

The other _automatic_ variables listed above (`airflow`, etc) derive
from Luna's automatic assignment (aka guessing) of [_channel
types_](#channel-types). The [`VARS`](../ref/summaries.md#vars)
command dumps list of which variables are defined for each individual.

You can add the special variable `verbose=T` to make the console print out the
variables in full (i.e. above we see the default reduced format).

<h4>Individual-ID substitution</h4>

One special variable is the `^` symbol (or, equivalently, `${id}`),
which denotes the ID (from the sample-list) of the current
observation, which is automatically set for each EDF processed.  This
can be useful to point to individual-specific files as inputs or outputs.

For example, below we use a combination of a project-specific variable _p_ and the
special individual-ID `^` variable with the
[`EPOCH-ANNOT`](../ref/epochs.md#epoch-annot) command:

``` 
...
EPOCH-ANNOT file=/path/to/${p}/data/^.eannot 
...  
```

With the above script, one might then issue commands such as:

```
luna proj1.lst p=proj1 -o out1.db < commands.txt
```

```
luna proj2.lst p=proj2 -o out2.db < commands.txt
```

and (assuming both projects had IDs `id0001`, `id0002`, etc)
Luna would look to the correct places to attach the
epoch-annotations, e.g.:

```
/path/to/proj1/data/id0001.eannot
/path/to/proj1/data/id0002.eannot
/path/to/proj1/data/id0003.eannot
...
```
or
```
/path/to/proj2/data/id0001.eannot
/path/to/proj2/data/id0002.eannot
/path/to/proj2/data/id0003.eannot
...
```

When using a command such as as [`WRITE`](../ref/outputs.md#write) or
[`WRITE-ANNOTS`](../ref/annotations.md#write-annots), you will almost
always want to use `^`: e.g. 
```
 WRITE-ANNOTS file=annots/^-v2.annot
```
would generate a file `annots/id0001-v2.annot` for an individual with ID `id0001`.

### Parameter files

As well as defining _variables_, other command-line options control
aspects of Luna's behavior, as tabulated below.  These are called
[_special variables_](#special-variables), which might be needed for
any analysis of an entire project. For example, to set the search [`path`](#search-paths) for files from a
sample list, one might use:

```
luna s.lst path=/home/joe/data/edfs/ -o out1.db < commands.txt
```

i.e. if the sample list just had the relative path `p1/id1.edf`,
adding the above `path` would make Luna search for
`/home/joe/data/edfs/p1/id1.edf` instead.

Rather than having to retype such things on every Luna command line, it can be
convenient to wrap them up in a _parameter file_, which is included
with a special `@` syntax, as follows:

```
luna s.lst @param.txt -o out.db < commands.txt
```

where `param.txt` (which can be called any legal filename) is a 
plain-text file that includes (possibly among other things) the line:

```
path	/home/joe/data/edfs
```

Arguments in `param.txt` are inserted as though they were explicitly
typed on the command line.  Note that whereas the command line expects
`key=value` pairs, parameter files use a tab to delimit the key
(variable name) and value.  Thus, parameter files should contain
exactly two tab-delimited columns on every row. As a larger example:

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

Note that the use of quotes is necessary when the annotation
(e.g. `Stage 2 sleep|2`) contains spaces (or commas or the special
`|` character).

In the above example, we define an [_alias_](#aliases) (`EEG`) which is
specified to be the only channel loaded (via the [`sig`](#signal-lists)
option).  As well as setting the [`annot-folder`](#annotations) and
[`path`](#search-paths) folders, this parameter file also defines a
number of other [_variables_](#variables) that might be used in
command scripts.  For example, `${sr}` might be the sampling rate;
`${nrem1}`, `${nrem2}`, etc, specify the annotation labels used for
sleep stages; the comma-delimited list in `${excl}` might define a
list of exclusionary annotations, e.g. to be used with [`MASK`](../ref/masks.md#mask):

```
MASK if=${excl}
```


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

As well as EDF+ Annotations, Luna accepts various other types of
_annotation file_ that typically describe _events_ in the EDF, which
are specified in the [_sample-list_](#sample-lists) and associated
with the EDF. Luna accepts the following annotation formats:

| Format | Description |
| ---- | ---- | 
| [.annot](../ref/annotations.md#annot-files) | Generic Luna annotation format (`.txt` and `.tsv` extensions are also valid) | 
| [.eannot](../ref/annotations.md#eannot-files) | Simple epoch-level annotation files (can also be loaded via the [`EPOCH-ANNOT`](../ref/epochs.md#epoch-annot) command) | 
| [EDF+](../ref/annotations.md#edf-annotations-channel) | EDF+ Annotation channels | 
| [XML](../ref/annotations.md#nsrr-xml-files) | XML format used by the [National Sleep Research Resource](http://sleepdata.org) to distribute sleep staging, and information on manually-scored arousals, movements and artifacts |

### Dates

By default, all dates are assumed to be in _European_ (day-month-year)
format (following from the use of EDF as the primary input format).
This extends to annotation (e.g. `.annot`) files.  The special
variables `read-mdy-annot-dates` and `read-mdy-edf-dates` allow for
non-European (_month-day-year_) format dates to be read from files --
either in annotations or EDF headers -- if set to true (`T`).
Currently, all outputs are in European format, and any other commands
(e.g. to specify the EDF header via `SET-HEADERS`) still assumes
European date format.


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

To operate on a range of subjects within a sample list, just give two numbers: e.g. 
```
luna large.lst 50 100 < commands.txt
```

The above would analyze from the 50th to the 100th EDFs in the
`large.lst` project sample list.  This can be useful if using Luna on
a cluster, to parallelize processing via batch submission, for example.

The special variables `id` and `skip` (also `include` and `exclude` ID
list files) can alternatively be used to control which individuals are
analysed from a sample list.

!!! Note
    If a number is given after the sample list, it is always
    interpreted as the position in the sample list, not an ID.  In
    other words, best not to use pure numbers as IDs in the sample
    list if possible.  If the IDs are numeric, you can always us `id`

    ```
    lune s.lst id=22 -o out.db < cmd.txt
    ```
    i.e. this will look for an EDF with the ID (first column in `s.lst`) that matches the ID `22` (nb. matching
    is for a string, so `022` != `22`) 




### EDFZs

As described in this [vignette](../vignettes/edfz.md), to save disk space
(and sometimes speed up analysis), Luna can read and write compressed
EDF files, using the [BGZF](https://samtools.github.io/hts-specs/SAMv1.pdf) library.  EDFZ files must be created by Luna's
[`WRITE`](../ref/outputs.md#write) command, with the `edfz` parameter
option added.  For example, taking the first [tutorial](../tut/tut1.md) EDF, we can write it out as an EDFZ:
```
luna s.lst 1 -s ' WRITE edfz edf-dir=z/ edf-tag=compressed sample-list=z.lst '
```

If the original EDF was`file.edf`, this creates two files
`z/file-compressed.edfz` and `z/file-compressed.edfz.idx` (along with
a sample list `z.lst` that points to them).  As well as `.edfz`, the extension
`.edf.gz` (paired with a `.edf.gz.idx` file) is supported - this latter form
is preferrable as utilities such as `gunzip` may expect to see a final `.gz` extension.

In the above example, we see a reduction in disk space:
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

Although for a single PSG this saving is negligible, across thousands
of studies, savings can become more significant.

Although EDFZ files must be created by a special Luna command, they
can be read (i.e. decompressed) as any other
[gzip](https://en.wikipedia.org/wiki/Gzip) file.  The
following standard Unix/Mac `gunzip` command decompresses an EDFZ to a standard EDF:

```
cat file.edfz | gunzip > file.edf
```

See information on the [`WRITE`](../ref/outputs.md#write) command's
`edfz` option for more details.

!!! hint 
    Although compression as an EDFZ is _lossless_ (i.e. all
    information is preserved), there may be small differences
    between the original EDF and an uncompressed EDFZ simply due to
    floating point accuracy of the EDF format. This is not specific to
    EDFZ files _per se_ -- it also applies to standard EDFs generated by the
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

If Luna finds the option `--fs` (specifying a sample rate) on the command line, it will
interpret the first argument to be a text file (rather than a sample
list or an EDF).  In this instance, the sample rate is set to
256 Hz (and so, implies 15,360/256 = 60 seconds of signal).

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

The new signals are, by default, labelled `S1`, `S2`, etc.  If the
`--chs` option is specified on the command-line, different channel
labels can be assigned:

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

Additionally, all channels must have the same sample rate when being
input as ASCII.

!!! hint "Raw data and EDF comparisons"
    As a simple demonstration of converting between ASCII and EDF formats (and also to show the relative speed and filesize benefits of EDF),
    here we'll generate random signals for 4 channels for a 10 hour recording at 256 Hz using `awk`:
    ```
    echo | awk '{for (i=0;i<256*60*60*10;i++) print rand(),rand(),rand(),rand()}' OFS="\t" > i.txt
    ```
    Whereas R takes about 25 seconds to read these data (using a vanilla call to `read.delim()`), Luna takes about 7 seconds:
    ```
    luna i.txt --fs=256 -s DESC
    ```
    ```
    reading 4 signals, 36000 seconds (9216000 samples 256 Hz) from i.txt

    ID                : i.txt
    Clock time        : 00.00.00 - 10.00.00
    Duration          : 10:00:00  36000 sec
    # signals         : 4
    Signals           : S1[256] S2[256] S3[256] S4[256]
    ```
    We can convert to an EDF, `i.edf`:
    ```
    luna i.txt --fs=256 -s WRITE edf=i
    ```
    Luna now reads the EDF in approximately 0.1 seconds:
    ```
    luna i.edf -s READ
    ```
    (we've added the `READ` to force Luna to load all the signal data from EDF)

    In terms of file size, whereas `i.txt` is 316M, the corresponding `i.edf` is only 70M, i.e. one fifth the file size.   


### Empty EDFs

Rather than reading data from a file, it is possible to specify an
empty or "virtual" EDF. This will have a fixed duration and sample
rate, but initially no channel/signal data. This can be convenient
when using certain commands that do not require signal data, e.g. the
[`SIMUL`](../ref/simul.md#simul) or
[`OVERLAP`](../ref/intervals.md#overlap) commands.

To create an empty EDF, specify `.` (period character) as the
sample-list/filename along with `--nr` and `--rs` to give the number
of records (`nr`) and the EDF record size (`rs`) respectively. Luna
will create an EDF of this duration (i.e. with headers speciying the
length of the recording) but with no signals, i.e. a collection of
empty records.

### Time points

Internally, Luna encodes time in _time-points_ where 1 unit is 10<sup>-9</sup> seconds (stored internally as
`uint64_t` types). A handful of outputs from some commands use this format (rather than in seconds, clock-time
or _sample-points_). For a given EDF, time-points start at 0,
corresponding to the start of the EDF.  One consequence is
that Luna cannot represent any event that occurs _before_ the EDF
start.  (If this becomes an issue, it is always possible to work with
EDF+D files that have a sufficiently early EDF start, but then the
first signal records starting at some later point, i.e. which would be
after initial annotations.)


### Channel location files

Some commands that work with hdEEG data require a channel location map
- a "_clocs_" file.  Locations should be in Cartesian X, Y, Z format.

- Four tab-delimited columns
- First column is channel name
- Second to fourth columns are X, Y and Z coordinates

Internally, all coordinates are converted to spherical coordinates on
a unit sphere, so any scaling can be used. 

Here is an example file:
[https://zzz.bwh.harvard.edu/dist/luna/clocs/clocs64](https://zzz.bwh.harvard.edu/dist/luna/clocs/clocs64).

If a _clocs_ file is not specified but a particular command requires one,
Luna will use a default map, applicable for typical 64-channel
applications (matching the example file above).

## Special variables

Special variables (options that control Luna's behavior) are tabulated
below.   For command-line Luna, these should be placed after the samlpe-list/EDF (first argument)
but before any `-s` command;  otherwise, in general the order typically doesn't matter.
__All special variables must be assigned (`=`) an explicit value.__  Often this will simply be `T` meaning _true_
to turn on that option. 
For example, this is _incorrect_ as Luna would assume that `force-edf` is an ID in the `s.lst` sample list:
```
luna s.lst force-edf < cmd.txt
```
The correct format is:
```
luna s.lst force-edf=T < cmd.txt
```
i.e. Luna uses the presence of the `key=value` form to let it know this is _not_ an ID.


_Controlling inputs_

| Special Variable | Description |
| ---- | ---- | 
| [`id`](#ranges) | Only analyse these IDs from the sample list, e.g. `id=study-001` or `id=p1,p3` |
| [`skip`](#ranges) | Skip these IDs from the sample list, e.g. `skip=study-001` or `skip=p1,p3` | 
| [`vars`](#individual-variables) | Specify a file with individual-level variables/values |
| [`ids`](#swapping-ids) | Specify a file of remapped IDs |
| [`exclude`](#exclude-lists) | Specify a file of IDs to exclude from analysis |
| [`include`](#include-lists) | Specify a file of IDs to include from analysis |
| [`path`](#search-paths) | Set search path for files in sample lists |
| [`preload`](#preloading) | Preload the entire EDF if `T` | 
| [`order-signals`](#ordering-signals) | Force EDF signals to be in alphabetical order | 
| [`fix-edf`](#fix-truncated-edfs) |  Attempt to correct truncated/over-long EDFs if `T` |
| [`force-edf`](#annotations)       | Skip EDF annotations _and_ time-track from any EDF+, and force as a continuous EDF if `T`|
| [`read-mdy-annot-dates`](#dates) | Assume US-style month-day-year dates when reading all annotation files if `T`|
| [`read-mdy-edf-dates`](#dates) | Assume US-style month-day-year dates when reading the EDF start date if `T` |

_Controlling outputs_

| Special Variable | Description |
| ---- | ---- | 
| [`silent`](#output-verbosity) | Runs Luna silently if `T`  |
| [`verbose`](#output-verbosity) | Runs Luna in verbose mode if `T` |
| [`tt-prepend`](#text-tables) | Add value to start of text-table file names (equiv. `tt-prefix`) |
| [`tt-append`](#text-tables) | Add value to end of text-table file names (equiv. `tt-suffix`) |
| [`compressed`](#text-tables) | Y/N to force all `-t` text-table output to compressed (Y) or not (N) |


_Signals and EDF headers_

| Special Variable | Description |
| ---- | ---- | 
| [`sig`](#signal-lists)| Include this signal(s) in analysis | 
| [`anon`](#anonymize-edf-headers) | Anonymize EDF headers | 
| [`starttime`](#set-edf-start-time) | Set EDF start time |
| [`startdate`](#set-edf-start-date) | Set EDF start date | 
| `wildcard` | Set individual ID wildcard character (default is `^`) |

_Annotations_

| Special Variable | Description |
| ---- | ---- | 
| [`annot-file`](#attaching-annotations) | Specify annotations to attach on the command line |
| [`annots`](#selecting-annotations)| (Or `annot`). Load only this (comma-delimited) list of annotation classes (rather than all) |
| `tab-only` | Only allow tabs (vs tabs and spaces) as delimiters in `.annot` files |
| `annot-keyval` | Set _key=value_ delimiter for annotation meta-date (default: `=`) |
| `align-annots` | (Advanced) Align these annotations (comma-delimited list) to EDF record start, assuming 1 second records | 
| `class-inst-delimiter` | Specify character to delimit annotation classes and instances (default: `:`) |
| `combine-annots` | Character to use when combining classes and instance IDs (default: '_' ) | 
| `annot-whitelist` | Read only these annotations |
| `annot-unmapped` | Read only these annotations |
| `annot-remap` | Read only these annotations |
|
| `sec-dp` | Set number of decimal places for annotation time outputs (default: 3) |
| `add-ellipsis` | For `WRITE-ANNOTS` of `.annot`only, set zero-duration events to have `...` stop fields |
| `annot-segment` | Label for segment annotation from `SEGMENTS annot` (default: `segment`) |
| `annot-gap` | Label for gap annotation from `SEGMENTS annot` (default: `gap`) |
|
| [`skip-annots`](#annotations) | (Or `skip-all-annots`). Same as `skip-sl-annots=T` and `skip-edf-annots=T` combined (default: F)  |
| [`skip-sl-annots`](#annotations)  | Skip annotation files specified in the sample list (default: F) |
| [`skip-edf-annots`](#edf+-annotations) | Skip EDF Annotations tracks from any EDF+ (default: F) |
| [`inst-hms`](#inst-hms) | Assign missing annotation instance IDs based on time |
| [`force-inst-hms`](#inst-hms) | Always assign annotation instance IDs based on time |
| [`annot-remap`](#remapping-annotations) | Set automatic remapping of stages (default: `T`) |
| [`nsrr-remap`](#remapping-annotations) | Set extra NSRR remapping of annotations (default: `F`) |
| [`edf-annot-class`](#edf-annotations) | Read these EDF+ (comma-delimited) labels as _classes_ (default: `N1,N2,N3,R,W,?,arousal,LM,NR`)|
| [`edf-annot-class-all`](#edf-annotations) | Read all EDF+ labels as _classes_ (default: `F`) |


_Annotation meta-data_

| Special Variable | Description |
| ---- | ---- | 
| `annot-meta-default-num` | Set default annotation meta-data type to numeric if present/`T` |
| `num-atype`    | Set these meta-data keys to numeric type, e.g. `num-atype=dur,amp,frq` | 
| `int-atype`    | As above, for integer type |
| `txt-atype`    | As above, for string (text) type |
| `bool-atype`   | As anove, for boolean (T/F) type |
| `annot-meta-delim1` | Set meta-data delimiter 1 (default `;`, e.g. `a=1;b=2` ) |
| `annot-meta-delim2` | Set meta-data delimiter 2 (default `|`, e.g. `a=1|b=2` ) |
| `annot-keyval` | Set _key=value_ delimiter for annotation meta-date (default `=`) |


_Remapping channel/annotation labels_

| Special Variable | Description |
| ---- | ---- | 
| [`alias`](#aliases)| Specify a channel alias |
| [`remap`](#remapping-annotations)| Specify an annotation remapping (cf. channel aliases) |
| `sanitize` | Change special character labels to `_` if this is set to `T` |
| `upper` | Set all channel labels to uppercase if set to `T` | 
| [`spaces`](#spaces-in-channel-names ) | Alternate character for space substitution in channel/annotation names |
| [`keep-spaces`](#spaces-in-channel-names) | Retain spaces in channel/annotation names if set to true |
| [`keep-channel-spaces`](#spaces-in-channel-names) | Retain spaces in channel names if set to true|
| [`keep-annot-spaces`](#spaces-in-channel-names) | Retain spaces in annotation names if set to true|
| `retain-case` | Keep signal label case when it (case-insensitive) matches primary alias | 

_Epochs and sleep staging_

| Special Variable | Description |
| ---- | ---- |
| [`epoch-len`](#epoch-len) | Specify the default epoch duration |
| [`no-epoch-check`](#no-epoch-check) | Do not enforce epoch check for .eannot files |
| `epoch-check` | Set tolerance value for epoch check (default: 5) |
| [`assume-pm-start`](#force-evening-start-time)| Force morning times (after _X_ am) to be _X_ pm  |
| [`assume-stage-duration`](#stage-annotations) | Assume zero-duration stage labels are of epoch-length |
| [`ss-prefix`](#stage-annotations) | Set stage annotation prefix (for reading in stage data) |
| [`ss-pops`](#stage-annotations) | Same as `ss-prefix=p` |
| [`ss-soap`](#stage-annotations) | Same as `ss-prefix=s` |

_Misc_

| Special Variable | Description |
| ---- | ---- | 
| [_power bands (various)_](#spectral-power-bands)| Change default power bands (delta, theta, etc.) |
| `srand` | Set the seed of the random number generator to this fixed value, e.g. `srand=123456` | 
| [`ch-exact`](#ch-exact)| Add an exact match for a channel type |
| [`ch-match`](#ch-match)| Add a partial match for a channel type |
| [`ch-clear`](#ch-clear)| Clear all channel type mappings |


Any other variables specified on the command line or a [_parameter
file_](#parameter-files) are interpreted as typical variables, that
can be used in scripts: e.g.

```
luna s.lst xyz=123 < cmd.txt 
```

will set `${xyz}` to `123` if used in scripts.  Special variables
(i.e. those tabulated above) are __reserved__ names and cannot be used in
scripts.


!!! note "Specifying special variable values"
    Variables (special or otherwise) always need to be explicitly assigned a value: e.g.    
    ```
    luna s.lst keep-spaces=T < cmd.txt
    ```
    rather than just
    ```
    luna s.lst keep-spaces < cmd.txt
    ```
    as in the second case, `keep-spaces` would be interpreted as an ID to be matched in the sample list `s.lst`.  (In this sense,
    the syntax varies from command line options, e.g. one can write `PSD dB` versus having to explicitly write `PSD dB=T`.)
    
    For variables that expect true/false values:

    - matches are case-insenstive
    - values of `1` or starting with `T` (true) or `Y` (yes) are all interpreted as _true_
    - all other values are interpreted as _false_; for clarity, `0`, `F` or `N` should be used in practice

    Special variables can be assigned values on the command line, or via an
    `@`-included [_parameter file_](#parameter-files).



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
    Luna's perspective. (One minor but virtuous side effect is that this can speed up
    reading an EDF, if one knows that only a subset of channels is needed.)

    This is different from the using the 
    `sig` option to modify the behavior of an individual Luna command: 
    in this latter case, it is only that particular command that is
    restricted to those signals/channels, and so other channels are
    still part of the in-memory EDF and available for subsequent processing.  For
    example:
    ```
    ...
    FILTER bandpass=0.3,35 ripple=0.01 tw=0.2 sig=C3 
    STATS sig=C3,EMG,ECG
    ...
    ```

!!! Note
    The terms _signals_ and _channels_ are used interchangeably throughout this documentation.

!!! warning "`sig` and EDF+"
    The `sig` special option cannot be used with EDF+ files.


The `sig` option works with the `alias` options also.  For example, if the original EDF label is `CH_X001`
but an [alias](#aliases) has been defined to convert it to `EEG`, we can use that aliased form
in the `sig` statement:
```
luna s.lst sig="EEG,EMG,ECG" alias="EEG|CH_X001" -s DESC
```
That is, we've relabeled `CH_X001` as `EEG` and used `sig` to select it; the output from `DESC` reads:
```
Signals           : EEG EMG ECG
```

  
!!! danger "Restrictions for channel names" 
    Channel names should not have any of the following characters: comma, tab, new-line, pipe
    (|).  Moreover, there are advantages in _not using any special
    characters_ (e.g. space, parentheses, asterisks, etc) or characters that are common
    logical or arithmetic operators, e.g. minus, plus signs, etc.   Avoiding such characters
    makes it easier to specify channels on the command line and in scripts.

    Avoding special characters in channel labels can
    also make processing output easier, i.e. if you use [`destrat`](destrat.md)
    to produce a table that uses channel labels as 
    variable (column) names.

    In general, it is desirable to restrict channel labels to
    alphanumeric characters and the underscore character as a
    separator. See this [FAQ](../faq.md#advice-on-channel-names).
    You can also use Luna [_aliases_](#aliases) to make better channel names. 

### Ordering signals

Setting `order-signals` to true (`T`, `Y` or `1`) will ensure that EDF
channels are loaded in alpha-numeric order: e.g.

```
luna s.lst 1 -s DESC
```
```
Signals           : C3[128] C4[128] A1[128] A2[128] LOC[128] ROC[128]
                    ECG2[256] ECG1[256] LEFT_LEG1[256] LEFT_LEG2[256] RIGHT_LEG1[256] RIGHT_LEG2[256]
                    EMG1[256] EMG2[256] EMG3[256] AIRFLOW[32] THOR_EFFORT[32] ABDO_EFFORT[32]
                    SNORE[256] SUM[32] POSITION[1] OX_STATUS[1] PULSE[1] SpO2[1]
                    NASAL_PRES[64] PlethWV[128] Light[1024] HRate[1024]
```

In contrast, with ordering:

```
luna s.lst 1 order-signals=T -s DESC
```

```
Signals           : A1[128] A2[128] ABDO_EFFORT[32] AIRFLOW[32] C3[128] C4[128]
                    ECG1[256] ECG2[256] EMG1[256] EMG2[256] EMG3[256] HRate[1024]
                    LEFT_LEG1[256] LEFT_LEG2[256] LOC[128] Light[1024] NASAL_PRES[64] OX_STATUS[1]
                    POSITION[1] PULSE[1] PlethWV[128] RIGHT_LEG1[256] RIGHT_LEG2[256] ROC[128]
                    SNORE[256] SUM[32] SpO2[1] THOR_EFFORT[32]
```

This can be useful when wanting to harmonize EDFs, i.e. to ensure that
the `SIGNALS` output of `HEADERS` is not trivially different due only
to different ordering.  One other context where this can matter is for
commands that output pairwise channel combinations: typically this is
done according to the order of the EDF header, and so whereas one EDF
might output `CH1` as `C3` and `CH2` as `F3`, for example, another may
output these reversed (if the EDF order differs).  Having a fixed
order can make it easier to combine outputs (i.e. without having to
double-enter all outputs).


### Preloading

By default, Luna performs _lazy loading_ of EDFs, meaning it only
pulls a record when it is needed.  On first attaching an EDF, only the
header is loaded from disk, which is always very quick.  One exception
is for EDF+D channels: on first attaching a file, Luna must scan _all_
records in order to be able to build the timeline for the recording.
If the file is large and contains a large number of records, this can
take a non-trivial amount of time.

Further, by default Luna uses random-access reading, i.e. skipping to
a record within the EDF and only reading that record, one record at a
time.  The performance of this type of random access approach can vary
depending on the operating system and file system.  For example,
modern solid state drives (SSDs) typically do not incur any
penalty for randomly accessing a file, even if the entire file is to
be read from disk.  In contrast, hard disk drives (HDDs) often perform
better when streaming sequentially across a large file.  Further,
network-attached storage may incur different costs for random versus
sequential access of data.

If performance is slow, especially for EDF+D files, it can be useful
to set `preload=T` which means that the entire file is read in a
single, sequential pass, which is often quicker (especially as, in
practice, many commands will end up pulling all records from disk
anyway).



The default is _not_ to preload data, as quite a few commands don't
actually need to read any signal data from the EDF (e.g. reporting
headers, working with annotations, or in the context of using the Luna
library as part of an interacrive viewer, as in _lunapi's_ `scope()`,
etc).

 

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
being the _primary_ alias, or _canonical channel label_.  
When specifying channels, one can either use the
original EDF terms (e.g. `C3-M1`) _or_ the primary alias (`C3`). That
is, even if the EDF only has a channel `C3-M1`, it will
still be included in any analysis that requires `C3`.  Similarly, in
all output (including new EDFs generated via the
[`WRITE`](../ref/outputs.md#write) command), the canonical channel
labels (_primary aliases_) will be swapped in whenever needed (i.e. to `C3` here). It is
not necessary that the canonical form exists in _any_ of the project's
EDFs: it could be an new label you wish to apply to this
group of differently-named but otherwise identical channels.

!!! Note
    Because most shell scripts interpret `|` as a special control character (pipe), we
    use quotes when specifying `alias` on the command line, i.e. `alias="C3|C3-M1|C3-A1"`. This 
    also implies that channel names should not contain `|` characters.

### Remapping annotations

The `remap` option operates in the same way as the `alias` option, except for annotation labels instead of channel names.
For example, to change an annotation `REMS` to `REM`, add the following:
```
luna s.lst remap="REM|REMS" < cmd.txt
```
As with [aliases](#aliases), you can specify multiple, `|`-delimited remappings, i.e. in a many-to-one fashion.  Likewise, you
can put these in a [parameter file](#parameter-files) rather than write these out on the command line.  This will also be easier if
you annotations have spaces and special characters; you'll still need to use quotes if the labels have spaces: e.g.
```
remap      REM|REMS|"REM Sleep"|"Rapid eye movement sleep"
```
This will remap any of the three forms listed to the primary label: `REM`.

!!! warning "Automatic annotation remappings"
    Note that Luna by defaults
    add in some _default_ annotaton remappings for sleep stages, e.g.
    turning `Stage NREM1` to `N1`, etc, so that the `HYPNO`, `SOAP` and `POPS` commands
    know which labels to expect.  This can be disabled by setting `annot-remap=F`. It is also possible to turn on some more
    mappings for common NSRR labels (e.g. arousals, apnea, etc) by adding `nsrr-remap=T`.  Note that the order
    of `annot-remap` and `nsrr-remap` will matter (as `annot-remap` turns off _all_ annotation remappings.

### Spaces in channel and annotation names

BY default, Luna swaps all spaces in channel or annotation names with an underscore (`_`) character,
and will also _trim_ any leading or trailing space/underscore characters.  You can change the character swapped in
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
As an aside: we do not recommend _swapping in_ special characters into channel and annotation labels: the point of the
`spaces` option is to make these labels _easier_ to interact with in a typical command-line environment.

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

Using the `include` keyword, this is similar to [exclude-lists](#exclude-lists) except this means that
only individuals in the specified file will be included, everybody
else will be excluded.  You cannot specify an `include` list and
`exclude` list together.

### Swapping IDs

The `ids` special variable takes a file (e.g. `ids=file.txt`), for which each row should have exactly two tab-delimited fields
refleting the observed (original) ID in the first column, and the new ID in the second column: e.g.
```
id1	newid1
id2	newid2
```
Luna swaps the original ID (as supplied from the [sample list](#sample-lists)) to the new value, if it is encountered. 

### Anonymize EDF headers

Adding `anon=T` will wipe the EDF headers (the EDF header _Patient
ID_, _Recording Information_ and _Start date_ fields will be set to
their default _null_ values as per the EDF spec.)  This is similar to the [`ANON`](../ref/manipulations.md#anon) command
except this is performed _before_ any annotations are attached.  This will influence how dates are interpreted in
any annotation files that use dates, therefore.  In contrast, the `ANON` command allows a greater degree of flexibility
in terms of which fields are wiped.

### Set EDF start time

This special variable sets the EDF start time (header) value to the specified string, e.g. `starttime=20.30.05`.   This should
use EDF spec. for the values: i.e. 8 characters, 24-hour times, _hh.mm.ss_ format with period (`.`) delimiters.   Unlike
the [`SET-HEADERS`](../ref/manipulations.md#set-headers) command, this change is made on first attaching the EDF, _before_ any
annotations are attached: annotation times will therefore reflect these changes, depending on whether they use relative or
absolute values, etc.

### Set EDF start date

This special variable sets the EDF start date (header) value to the
specified string, e.g. `startdate=25.12.00`.  This should use EDF
spec. for the values: i.e. 8 characters, _dd.mm.yy_ format with period
(`.`) delimiters. Dates cannot be before `01.01.85` which is the
_null_ date as per EDF spec. Unlike the
[`SET-HEADERS`](../ref/manipulations.md#set-headers) command, this
change is made on first attaching the EDF, _before_ any annotations
are attached: annotation times will therefore reflect these changes,
depending on whether they use relative or absolute values, etc.


### Attaching annotations

To attach an annotation file without having to edit the sample-list,
you can use the `annot-file` (or equivalently `annot-files`,
`annots-file` or `annots-files`) option.
```
luna s.lst 1 annot-file=path/to/file.annot < cmd.txt
```
This will be most useful when working with a single EDF - especially
as the `^` special character for individual ID is not translated on
the shell command line.  To use `annot-file` (or any other special variable) in a programmatic manner,
one can use shell variables, e.g. within a loop, where here `${id}` is a _shell_ variable, e.g.:
```
for id in `cut -f1 s.lst`
do
luna data/${id}.edf annot-file=data/${id}.annot -o out/${id}.db < cmd.txt
done
```
Note that the above scenario would be better approached by simply using a sample list that links EDFs and annotations (although this implies
all output in a single database `all.db`), i.e.:
```
luna s.lst -o all.db < cmd.txt
```

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

### EDF+ annotations

Unless `skip-edf-annot=T` or `skip-annots=T` are set, Luna will read any `EDF
Annotations` channels present in EDF+ files and treat them as standard
annotations (i.e. as if they were read from an `.annot` or XML file).

As EDF+ annotations can often contain notes, Luna does not assign a
_class_ to each unique annotation: rather, all annotations are
assigned to the `edf_annot` _class_, with the _instance_ ID set the
the value in the EDF+.  This behavior can be changed by setting the
special variable `edf-annot-class-all=T`.  Alternatively, a subset of
EDF+ annotations can be set to _classes_, and everything else pooled
under the `edf_annot` class, with `edf-annot-class=X,Y,Z` option.  By
default, the following special values are treated this way (after
annotation remapping, unless `annot-remap=F` to handle variation in
stage labels): `N1,N2,N3,R,W,?,arousal,LM,NR`


### Other annotation options

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

### Epochs and sleep staging

The `epoch-len` command can be used to specify a default epoch
duration (in seconds) different from 30, which will be used when
attaching `.eannot` files specified in the sample-list (i.e.  to
calculate the implied number of epochs in the EDF).

```
epoch-len	20
```

### Stage annotations

Unless `annot-remap=F`, Luna will attempt to map a set of "typical"
stage labels (e.g. `Stage N1`, `Stage NREM1`, `NREM1`, etc) to a
standard set of labels: the five main stages are:

```
  N1  N2  N3  R   W 
```
and `L` is a lights-on epoch and `?` is unknown/missing.

If working with other sets of stage labels (e.g. from manual scoring as well as POPS, etc) it can be useful to distinguish them by a prefix.  The `ss-prefix` special variable can achieve this, e.g. by `ss-prefix=p` will change the above to
```
  pN1  pN2  pN3  pR  pW 
```

Commands such as `HYPNO` that depend on these labels will then use
those values if they exist, instead of the standard ones.

Also, when reading stage labels from an annotation file, the
`assume-stage-duration=T` will change zero-duration labels to be of
epoch length.  (Sometimes an annotation file does not explicitly
specify the duration/end of each stage, and so (if each annotation is
indeed one epoch), this can be convenient.)


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
luna s.lst 1 -o out.db -s HEADERS
```
```
===================================================================
+++ luna | v0.99, 04-Dec-2023 | starting 04-Dec-2023 12:24:49 +++
===================================================================
input(s): s.lst
output  : out.db
commands: c1	HEADERS	

___________________________________________________________________
Processing: nsrr01 [ #1 ]
 duration 11.22.00, 40920s | time 21.58.17 - 09.20.17 | date 01.01.85

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
 CMD #1: HEADERS
   options: sig=*

___________________________________________________________________
...processed 1 EDFs, done.
...processed 1 command set(s),  all of which passed
-------------------------------------------------------------------
+++ luna | finishing 04-Dec-2023 12:24:49                       +++
===================================================================
```

### Output verbosity

The special variables `verbose` and `silent` control the level of console output.

```
luna s.lst 1 verbose=T -o out.db -s HEADERS
```
```
===================================================================
+++ luna | v0.99, 04-Dec-2023 | starting 04-Dec-2023 12:25:41 +++
===================================================================
input(s): s.lst
output  : out.db
commands: c1	HEADERS	

___________________________________________________________________
Processing: nsrr01 [ #1 ]
 duration 11.22.00, 40920s | time 21.58.17 - 09.20.17 | date 01.01.85
  40920 records, each of 1 second(s)

 signals: 14 (of 14) selected in a standard EDF file
  SaO2 | PR | EEG_sec | ECG | EMG | EOG_L | EOG_R | EEG
  AIRFLOW | THOR_RES | ABDO_RES | POSITION | LIGHT | OX_STAT

 annotations:
  [Arousal] 194 instance(s) (from edfs/learn-nsrr01-profusion.xml)
  [Hypopnea] 361 instance(s) (from edfs/learn-nsrr01-profusion.xml)
  [N1] 109 instance(s) (from edfs/learn-nsrr01-profusion.xml)
  [N2] 523 instance(s) (from edfs/learn-nsrr01-profusion.xml)
  [N3] 17 instance(s) (from edfs/learn-nsrr01-profusion.xml)
  [Obstructive_Apnea] 37 instance(s) (from edfs/learn-nsrr01-profusion.xml)
  [R] 238 instance(s) (from edfs/learn-nsrr01-profusion.xml)
  [SpO2_artifact] 59 instance(s) (from edfs/learn-nsrr01-profusion.xml)
  [SpO2_desaturation] 254 instance(s) (from edfs/learn-nsrr01-profusion.xml)
  [W] 477 instance(s) (from edfs/learn-nsrr01-profusion.xml)

 variables:
  airflow=AIRFLOW
  ecg=ECG
  eeg=EEG_sec,EEG
  effort=THOR_RES,ABDO_RES
  emg=EMG
  eog=EOG_L,EOG_R
  hr=PR
  id=nsrr01
  light=LIGHT
  oxygen=SaO2,OX_STAT
  position=POSITION

 ..................................................................
 CMD #1: HEADERS
   options: sig=*

___________________________________________________________________
...processed 1 EDFs, done.
...processed 1 command set(s),  all of which passed
-------------------------------------------------------------------
+++ luna | finishing 04-Dec-2023 12:25:41                       +++
===================================================================
```

As expected, when running with `silent=T`, Luna doesn't generate any output to the (standard error) console:
```
luna s.lst 1 silent=T -o out.db -s HEADERS
```
```

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
you'll want to use Luna's [text-tables](#text-tables) or [_lunout_ databases](#lunout-databases),
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
    Please see [this page](destrat.md#text-tables) for some current known issues with the initial implementation of the `-t` flag.

### _lunout_ databases

This is the primary mode of output for most Luna commands.
Here we run the same command as in the [previous section](#default-text-output)
but instead using a [_lunout_ database](destrat.md) to collect the
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

!!!note "Naming"
    We don't actually use this name throughout most of the documentation as a) it is the implicit default
    in most cases, and b) it is an ugly-sounding name, but we couldn't come up with anything better ;-).
    So, when referring to an _output database_ generically, this means the same as a _lunout database_.

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
 
### Caches

[Caches](../ref/outputs.md#cache) are an advanced, internal feature, to be used to share
information between different Luna commands within the same single
dataset, in particular the [`PREDICT`](../ref/predict.md#predict) command.


### New EDFs

Luna can output new EDFs (or EDF+ files, or [compressed EDFs](#edfz))
after manipulating signals, masking epochs, etc, via the
[`WRITE`](../ref/outputs.md#write) command.

### Annotations

Some commands, such as
[`WRITE-ANNOTS`](../ref/annotations.md#write-annots), can produce
[.annot files](../ref/annotations.md#annot) containing interval-based
annotations.  Other commands
(e.g. [`SPINDLES`](../ref/spindles-so.md#spindles)) may generate
annotations internally, which can be used by subsequent commands; to
output those annotations, use
[`WRITE-ANNOTS`](../ref/annotations.md#write-annots).

### Misc text-file dumps

Some commands produce flat-file text output distinct from the usual
output mechanism (via the database or text-tables, as described
above), for example [`MATRIX`](../ref/outputs.md#matrix) or [`ICA`](../ref/ica.md#ica).
These outputs can often be quite large files.
