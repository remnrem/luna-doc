# FAQ and trouble-shooting

Although _not necessarily asked with respect to Luna_, 
here are some frequently asked questions:

## What?

Luna is a C/C++ library focused on the analysis of large numbers of
sleep studies encoded as EDFs.  This is a free, open-source project.
Currently, there is a command-line tool ([_lunaC_](luna/args.md)) and an
extension library for R ([_lunaR_](ext/R/index.md)).

## Which? 

The current version is the _beta-release_ __v0.28 (10-Apr-2023)__.
Use `luna -v` to display the specific build date/time.

## Where?

Luna is developed at the [Brigham & Women's
Hospital](https://www.brighamandwomens.org/) and [Harvard Medical
School](https://hms.harvard.edu/), Boston, MA, United States.

## Who?

Luna was primarily developed by Shaun Purcell, with input from
a number of colleagues:

- Senthil Pananivelu for maintaining distributions and work on Moonlight & Moonbeam
- Nataliia Kozhemiako for input into multiple EEG analytic components and the revised artifact detection workflows
- Shyamal Agarwal for work on automating the build distribution and NSRR's Automated Pipeline (NAP) built around Luna
- Michael Rueschman for input on Moonlight & Moonbeam
- Alexander Kent for testing and feedback
- Susan Redline and her team developing the [National Sleep Research Resource](http://sleepdata.org)
- Dennis Dean for sharing his original SpectralTrainFig code-base
- Sara Mariani and Charmaine Demanuele for input on several EEG and ECG analysis components

Interested to contribute (either as a colleague or as a job)? Please
[contact me](http://zzz.bwh.harvard.edu/index.html#contact).

## How?

Luna development is indirectly supported via a number of NIH grants: NHLBI
R01HL146339 (PI Purcell), NHLBI R21HL145492 (PI Purcell), NIMH R03
MH108908 (PI Purcell), as well as NHLBI R35HL135818 (PI
Redline) and NHLBI R24HL114473 (PI Redline).


## Why?

This is a good question and deserves a longer answer...  The primary
aim of Luna was to provide a platform for 1) adopting some of the
elegant methods and models that have emerged from animal and lab-based
cognitive neuroscience studies over the past decade or so, and 2) for
applying them in the context of large (albeit sometimes noisy)
epidemiological studies with polysomnography.

As a relative newcomer to sleep research (my personal background is
primarily in [psychiatric
genetics](http://zzz.bwh.harvard.edu/publications.html)), the
development of Luna has tracked with my (still steep) learning curve,
in how to think about sleep signal data.  Because of this, I adopted
the tools I was most familiar with (namely
[C/C++](https://en.wikipedia.org/wiki/C%2B%2B) and
[R](http://www.r-project.org)), rather than the ubiquitous "in-house
[Matlab](https://www.mathworks.com/products/matlab.html) script". In
developing Luna though, I've been constantly reminded of how powerful Matlab
and its associated toolboxes are for working with electrophysiological
signal data.  I can also appreciate that working with Luna's
particular instantiations of specific methods may be unnecessarily
restrictive for some.

So, _why wouldn't I just use Matlab?_ There was, from my perspective,
still an unmet need for tools to work with sleep data in 
[thousands of individuals](https://www.ncbi.nlm.nih.gov/pubmed/28649997), such as
from the [NSRR](http://sleepdata.org).  In my (limited) experience of
seeing how others approached sleep data, it seemed clear that although
the substantive _core_ of a particular analysis (e.g. power spectral
density estimation) could be efficiently and flexibly implemented in a
single Matlab command (i.e. `pwelch()` or similar), a lot of the
_scaffolding_ around these one or two central functions (i.e. most of
the "work" from a practical perspective) was more often than not a
tangle of brittle, error-prone and undocumented scripting.  Although
not a perfect solution even for our own work, Luna represents a modest
step in the direction of building more robust and scalable analysis
tools.

I had originally conceived of Luna just as my own personal library of
functions that would assist me in my sleep research. However, I 
decided to document and distribute this code for a number of reasons:

- _to make the tool better:_ documenting and distributing code has
  _intrinsic value_, as this process tends to make the underlying tool better,
  even if it will only ever be used by yourself or a very small number of people.

- _accessibility and transparency:_ the sleep field is unfortunately
  replete with black box proprietary software and file formats which
  can be limiting; making things open-source lets others see what you've done, and use it without restriction.

- _community:_ others can build upon your work; in
genetics, for example, I developed a tool PLINK, which has been quite
[widely-used](https://scholar.google.com/citations?user=t5o90hCxVMkC&hl=en&oi=sra).
Since it was first developed (in 2007), however, there have been considerable
advances in the scale of data, and in the types of analytic approaches taken.
Being an open-source tool, others were able to very significantly
augment and even [rewrite](https://www.cog-genomics.org/plink/1.9/)
it, to produce an order-of-magnitude more powerful tool, whilst at the
same time maintaining the pipelines and community experience that had
been built over more than a decade with PLINK.

For both larger and smaller projects, I'd strongly recommend the
document/distribute model whenever practically possible.

## Acknowledgments

Luna uses a number of excellent open-source components, in particular:

- [FFTW](http://www.fftw.org) library

- [SQLite](https://www.sqlite.org/index.html) embedded database

- [R Project for Statistical Computing](http://www.r-project.org)

- [Eigen](https://eigen.tuxfamily.org/index.php?title=Main_Page) C/C++ matrix/linear algebra library

- [LightGBM](https://lightgbm.readthedocs.io/en/v3.3.5/index.html) gradient boosting, tree-based learning algorithm library

- Chapters and example code from [Mike X
  Cohen](http://mikexcohen.com)'s fabulously clear and practical book:
  _Analyzing neural time series data_

- Lees, J. M. and J. Park (1995): Multiple-taper spectral analysis: A
  stand-alone C-subroutine: Computers & Geology: 21, 199

- Laurent Condat (2013) A Direct Algorithm for 1-D Total Variation
  Denoising .  IEEE Signal Processing Letters, 20:11.

- [Multi-scale entropy (MSE)
  algorithm](https://physionet.org/physiotools/mse/) by Madalena Costa
  et al. (Costa M., Goldberger A.L., Peng C.-K. Multiscale entropy
  analysis of biological signals. Phys Rev E 2005;71:021906.)


## Trouble-shooting

### Windows line endings

MS Windows uses carriage return (CR) and line feed (LF) characters to
denote the end of a line, whereas UNIX-like systems (including Mac)
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
luna s.lst -s "STATS sig=${eeg}"
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

