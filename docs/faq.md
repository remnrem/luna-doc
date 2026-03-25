# FAQ and Troubleshooting

Miscellaneous questions and practical troubleshooting notes for Luna.

For general background on what Luna is, who develops it, current versioning,
and project context, see [About](about.md).

## Where should I start?

Start with [Path](path.md), which gives the recommended newcomer route through
the documentation for command-line Luna, [lunapi](lunapi/index.md), and
[LunaScope](https://zzz.nyspi.org/lunascope/). In practice, the recommended
sequence is: tutorial first, then concepts, then the walk-through, then the
reference pages.

## Which interface should I use?

- Use [lunaC](luna/args.md) if you want explicit scripts, batch processing,
  sample lists, and direct access to the full command language.
- Use [lunapi](lunapi/index.md) if you want Python, notebooks, and interactive
  downstream analysis.
- Use [LunaScope](https://zzz.nyspi.org/lunascope/) if you want a GUI for
  visual review and point-and-click exploration.

All three sit on top of the same core library and largely the same concepts.

## What is a sample list?

A sample list is the main project-level input format for command-line Luna. It
maps IDs to EDFs and, optionally, associated annotation files and metadata. See
[sample lists](luna/args.md#sample-lists) on the concepts page.

In practical terms, it is a plain-text ASCII file. You can create one manually
with a text editor, edit an existing one directly, or build one automatically
using Luna helper commands such as [`--build`](ref/helpers.md#-build).

## How do I inspect a file before running a larger analysis?

The usual first step is to look at headers and signal descriptions. See:

- [HEADERS](ref/summaries.md#headers)
- [DESC](luna/args.md#signal-selection)
- [Quick start](tut/tut1.md)

## How do I get results out of Luna?

The main options are:

- write to a database with `-o`, then extract with [destrat](luna/destrat.md)
- write text tables directly with `-t`
- work directly through [lunapi](lunapi/index.md) in Python

See [output and destratification](luna/destrat.md) for the core model.

## Why did Luna change my channel or annotation labels?

By default, Luna sanitizes labels to make command-line and downstream processing
easier. Spaces are usually converted to underscores, and some other special
characters are also normalized. See [spaces in channel and annotation names](luna/args.md#spaces-in-channel-names).

If you want more control, see:

- [`sanitize`](luna/args.md#spaces-in-channel-names)
- [`keep-spaces`](luna/args.md#spaces-in-channel-names)
- [`alias`](luna/args.md#aliases)
- [`remap`](luna/args.md#remapping-annotations)

## Why does `-s` sometimes behave strangely?

Usually because the shell is interpreting characters such as `&`, `|`, `*`, or
`$` before Luna sees them. In most cases, wrap the full Luna expression in
single quotes when using `-s`.

See [Variables and special characters when using `-s`](#variables-and-special-characters-when-using--s).

## Can I use Luna without the command line?

Yes. The main alternatives are [lunapi](lunapi/index.md) for Python and
[LunaScope](https://zzz.nyspi.org/lunascope/) for GUI-based interactive work.
The same core documentation is still relevant, especially the tutorial,
concepts, and reference pages.

## Troubleshooting

### Windows line endings

Windows uses carriage return (CR) and line feed (LF) characters to
denote the end of a line, whereas Unix-like systems (including macOS)
use LF alone.   The `file` command on UNIX-like systems will indicate if this is the case.

```
file *.txt
```
```
foo.txt:     ASCII text, with CRLF line terminators
bar.txt:     ASCII text
```


Use a utility such as
[`unix2dos`](https://en.wikipedia.org/wiki/Unix2dos) to convert these
files.  Otherwise, use the tool `tr` available on most systems:

```
tr -d '\r' < infile.txt > outfile.txt
```



### Spaces and special characters in labels

Luna will automatically convert spaces channel and annotation labels
to a different character (underscore, `_`), to facilitate working in a
command line (or R) environment with these labels.  See
[here](luna/args.md#spaces-in-channel-and-annotation-names).

By default, spaces are converted to underscores (unless `keep-spaces=T`),
as are special characters (unless `sanitize=F`).   Special characters in
this context are:
```
 (space) - + / \ * < > = & ^ ! @ # $ % ( )
```
The following are _not_ converted, as they are already treated as special delimiters by Luna:
```
 " ' | ,
```

That is, by default the label `EEG C3-M2` will become `EEG_C3_M2`.
This facilitates working with channels as variable names in subsequent
applications: e.g. [`TRANS`](ref/evals.md#trans) commands, or for
processing output in R, for example, if variable/columns are labelled
by the `CH` name.  Despite the convention of using labels such as `EEG
C3-M2` in the EDF specification, this is not convenient for automated
processing of data, thus Luna's approach to a) allow those names as inputs, but
also b) by default, change them on-the-fly.

As Luna parses command files by whitespace, it is necessary to handle spaces in labels 
explicitly.  If you've turned off the above options (`keep-spaces=T` and `sanitize=F`)
then you have to explicitly place quotes around labels with spaces: e.g.
```
STATS sig="THOR RES",SpO2
```
Otherwise, the command would be parsed as `sig=THOR` (which would not match any channel) a
and `RES,SpO2` (which would be ignored).

If you are using the `-s` option to specify a commands directly as
arguments to Luna, you can quote the term with a space in it;  this assumes the
entire expression will be in single-quotes (which it should be, to avoid the shell
interpreting characters such as `&`, `$`, etc):

```
luna s.lst -o out.db -s ' EPOCH & MASK if="REM sleep|5" '
``` 

Similarly, if masking on an annotation with a space, you need to put
quotes around it. For example, the NSRR annotation for REM sleep has
spaces and special characters, `REM sleep|5`.  Therefore, in a command
file use:

```
MASK if="REM sleep|5" 
```

Alternatively, you can alias or remap labels as they are initially
read by Luna, to control more explicitly any renaming meaning that
Luna does not have to do this automatically. For example, to change a signal `REF X1` to simply `REF`, one can 
set an signal [`alias`](luna/args.md#alias) in a [_parameter
file_](luna/args.md#parameter-files):

```
 alias     REF|"REF X1"
```

Note how we put `REF X1` in quotes, to assist the parsing of this
term.  All subequent commands can now reference `REF` instead of `REF
X1`.

Paralleling the use of `alias` for channels, you can
use the [`remap`](luna/args.md#remapping-annotations) option for annotations:
```
remap     REM|"REM sleep|5"
```
Again, note the use of quotes around `REM sleep|5` as both spaces and `|` are special characters (here, `|` is used to delimit different annotations that would be mapped 
to the same term, e.g.:
```
remap     REM|"REM sleep|5"|R|Stage_REM|"Stage REM"|5
```
i.e. the above maps five different labels to `REM`; to make it clearer, below we add spaces to show the different terms:
```
       REM   <-  REM sleep|5     or     R     or     Stage_REM     or     Stage REM      or     5
```

In general, we suggest you use [_aliases_](luna/args.md#aliases) and [_remapping_](luna/args.md#remapping-annotations) if
Luna's defaults don't work, but try to use [sensible](#advice-on-channel-names) channel labels and
annotation names whenever possible.


### Advice on channel names

Try to keep channel names to simple alphanumeric characters combined
with the underscore character to delimit terms.  Although Luna will
accept spaces and characters such as `+ - * % ( ) . `, etc, in channel
names, we advise against them if you wish to use `destrat` and other
tools such as `R` to process results downstream.

!!! info
    As as v0.26, Luna will automatically _sanitize_ (replace the above
    type of special characters with underscores) channel and annotation
    labels (unless you set `sanitize=F`).

For any output that is stratified by channel (`CH`), you may
wish to create a dataset where each channel corresponds to a
column/variable in the output.  Without sanitization of labels, if a variable name is, for example,
`SIGMA`, then using a command like 

``` 
destrat out1.db -c CH > my-file.txt 
``` 

may create variables with names such as `SIGMA.CH.C3-M2` or
`SIGMA.CH.EEG(2)`.  When loaded into R, this may lead to variable
names that are harder to work with (i.e. these characters are swapped
to `.` or you need to quote variable/list names, etc).  For example,
if you output with channels are row stratifiers: 

``` 
destrat out1.db -r CH > my-file.txt 
``` 

but subsequently use an R command such as `dcast` (from the `reshape2`
or `data.table` packages) to generate a data frame where channels
correspond to columns, you'll end up with variable names such as
`d$C3-M2` which can make life difficult (i.e. R would complain that
`M2` doesn't exist, as the `-` is interpreted as a minus, so you'd need
to write d$"C3-M2", or find other work-arounds, etc).

To avoid this, use [_aliases_](luna/args.md#aliases).

PS. for other reasons, always good advice to avoid special characters
in IDs too... just stick to alpha-numeric characters and underscores.
In particular, the `^` character which is the reserved symbol
(meaning, within a script, "swap in the ID").


### Variables and special characters when using  `-s`

When writing Luna script on the command line, i.e. directly after `-s`
(rather than having Luna read in a script from a file or pipe), it may
be necessary to handle special characters that the shell (assuming a
`bash` shell here) might try to interpret different.  For example, `&`
would mean to run the prior command in the background; `*` would be
expanded to match all files in the current directory, etc.

The easiest way to handle most scenarios is to use single-quotes around all Luna commands,
in this example, to avoid `&` or `|` being interpreted as special characters
by the shell, e.g.:

```
 luna s.lst -s 'EPOCH & STATS sig=EEG1|EEG & ANNOTS' 
```

By using single-quotes, this tells the shell not to interpret the
characters there in any way.  As such, the input to Luna will be what
you'd expect, i.e. the text written _as is_ above.

One possbile exception is if you want to include shell variables in a
Luna script.   It is important to understand this distinction between _shell_
variables and _Luna_ variables, as they have similar syntax (`${var}`).  However,
these are distinct entities, even if they share the same label.

To set a shell variable, e.g. on the shell command line:
```
eeg=XYZ
```

If there was a channel named `XYZ`, you use the following Luna commands:
```
luna s.lst -s STATS sig=${eeg}
```
or 
```
luna s.lst -s 'STATS sig=${eeg}'
```
In both cases, `${eeg}` will be replaced with `XYZ` _before_ Luna even sees any input/commands.

In contrast, the following would produce different behavior:
```
luna s.lst -s 'STATS sig=${eeg}'
```
as now the shell does not try to expand the _shell variable_ `${eeg}` (because it is enclosed within _single_ quotes).   Rather, now, Luna will read `${eeg}` and 
interpret it as a Luna variable.   (In this case, `${eeg}` is a special Luna variable that is expanded to all channels that have a label matching typical EEG channel names.)


If you did want to pass a shell variable into a script using `-s`, the best way is to define it explicitly prior to the `-s` script.   Say we have a shell variable `${v}`:
```
luna s.lst v=${v} -s 'STATS sig=${v}'
```
The initial assignment sets a Luna variable (that happens to be called `v`) equal to the shell variable `v`; then within the script, it is the Luna variable that is used.   To make this 
clearer, we could give a different label to the Luna variable, but the behavior would be identical:
```
luna s.lst w=${v} -s 'STATS sig=${w}'
```
Note that even if the shell variable `${s}` exists, the following would _not_ work:
```
luna s.lst -s 'STATS sig=${s}'
```
as there is no Luna variable named `${s}`.  (The example above with `${eeg}` was a special case of a _pre-populated_ Luna variable.) 


!!! Note "String literals"
    One or two Luna commands expect single quotes to define
    string literals: e.g.  if using _eval_ expressions, such as
    `c('a','b','c')` to define a vector of characters `a`, `b` and `c`.
    If already using single quotes after the `-s` command, it will not
    work to use additional single quotes in expressions such as this.  For
    this special scenario (that likely will not often arise), either 1)
    place the commands in a separate file rather than use the `-s`
    function, or 2) utilize the fact that `{` an `}` can stand in for
    single quotes in this context: e.g. `c( {a}, {b}, {c} )` is
    interpreted identically to the above expression.


### EDF+ support for long integers and floats

As noted [here](https://www.edfplus.info/specs/edffloat.html), the
EDF+ spec allows for a logarithmic transformation which can be helpful
to represent floating-point data with a large dynamic range.  This is
__not__ currently implemented in Luna.
