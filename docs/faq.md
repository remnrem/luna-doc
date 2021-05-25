# FAQ and trouble-shooting

Although _not necessarily asked with respect to Luna_, 
here are some frequently asked questions:

## What?

Luna is a C/C++ library focused on the analysis of large numbers of
sleep studies encoded as EDFs.  This is a free, open-source project.
Currently, there is a command-line tool ([_lunaC_](luna/args.md)) and an
extension library for R ([_lunaR_](ext/R/index.md)).

## Which? 

The current version is the _beta-release_ __v0.25.5 (31-March-2021)__.
Use `luna -v` to display the specific build date/time.

## Where?

Luna is developed at the [Brigham & Women's
Hospital](https://www.brighamandwomens.org/) and [Harvard Medical
School](https://hms.harvard.edu/), Boston, MA, United States.

## Who?

Luna was primarily developed by Shaun Purcell, with input from
a number of colleagues:

- Shyamal Agarwal for work on automating the build distribution -- and
  onging work on the to-be-released web-based NSRR Automated Pipeline
  (NAP) built around Luna
- Nataliia Kozhemiako for input into multiple EEG analytic components and the revised artifact detection workflows
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



### Spaces or special characters in channel names or annotations

As of v0.24, Luna will automatically convert spaces channel labels and
annotation labels to a different character (underscore, `_`), to
facilitate working in a command line (or R) environment with these
labels.  See [here](luna/args.md#spaces-in-channel-and-annotation-names).  The
text below shows different approaches to using aliases, to effectively
remove spaces (or any other spaceial character) in a channel name.

Set an signal [alias](luna/args.md#alias) in a [_parameter
file_](luna/args.md#parameter-files). If the original label is `REF
X1`, for example, enter the line:

```
 alias     REF|"REF X1"
```

to create a new label alias `REF` which can be used, e.g. on the
command line, instead of `REF X1`.

Similarly, if masking on an annotation with a space, you need to put
quotes around it. For example, the NSRR annotation for REM sleep has
spaces and special characters, `REM sleep|5`.  Therefore, in a command
file use:

```
MASK if="REM sleep|5" 
```

Alternatively, use the [`remap`](luna/args.md#remapping-annotations) option.

If you are using the `-s` option to specify a commands directly as
arguments to Luna, you will likely already be using quotes for the
entire command, thus you need to escape those additional quotes: i.e.

```
luna s.lst -o out.db -s "EPOCH & MASK if=\"REM sleep|5\""  
``` 

In general, this can get a bit messy.  Therefore, 1) use command files
for most things, not `-s`,  2) use
[_aliases_](luna/args.md#aliases), and 3) [sensible](#advice-on-channel-names) channel labels and
annotation names whenever possible.


### Advice on channel names

Try to keep channel names to simple alphanumeric characters combined
with the underscore character to delimit terms.  Although Luna will
accept spaces and characters such as `+ - * % ( ) . `, etc, in channel
names, we advise against them if you wish to use `destrat` and other
tools such as `R` to process results downstream.

That is, for any output that is stratified by channel (`CH`), you may
wish to create a dataset where each channel corresponds to a
column/variable in the output.  If a variable name is, for example,
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

PS. for other reasons, always good advice to avoid special characters in IDs
too... just stick to alpha-numeric characters and underscores.

### Variables and special characters when using  `-s`

It may be necessary to use quotes, or escape special characters such
as `$` if specifying Luna commands on the command line after `-s` (instead
from standard input), to stop the shell from processing those as shell
directives.

Use quotes to avoid `&` or `|` being interpreted as special characters
by the shell, e.g.:

```
 luna s.lst -s "EPOCH & STATS sig=EEG1|EEG & ANNOTS" 
```

Also, if using Luna [variables](luna/args.md#variables) after `-s`, you will need to make 
sure the shell does not interpret the `$` as a shell variable.  For example, if using `bash`, 
then use single quotes instead of double quotes.  That is, this will _not_ work:

```
 luna s.lst -s "EPOCH & STATS sig=${eeg}" 
```
as there is no shell variable called `${eeg}`,  whereas this will:

```
 luna s.lst -s 'EPOCH & STATS sig=${eeg}'
```
(because `${eeg}` is a special Luna variable that is automatically defined based on channel names).

It is important to understand this distinction between _shell_
variables and _Luna_ variables: i.e. these are distinct entities, even
if they share the same label. If you want to pass a shell variable
into a script as a Luna variable (either via `-s` or using a command file), you
need to define this Luna variable prior to the `-s` statements, setting it to the
shell variable value: e.g. say you define a bash (shell) variable `s` as follows:

```
s="C3"
```
To use this shell variable in a Luna script, you could do one of two things: use double quotes, so that
the `${s}` variable that is expanded will be the shell variable:

```
luna s.lst -s "EPOCH & STATS sig=${s}" 
```
Note that this version does not use a Luna variable,
rather it is identical to typing the following on the command line (i.e. this is what Luna 'sees'):
```
luna s.lst -s "EPOCH & STATS sig=C3" 
```

Alternatively, if using single-quotes (e.g. because the script had other Luna variables) you have to
define the Luna variable first:
```
luna s.lst s=${s} -s 'EPOCH & STATS sig=${s}'
```
In the above, the first instance of `${s}` refers to the shell variable, i.e. it is the same as typing `s=C3`.   The second
instance of `${s}` is instead interpreted as a Luna variable, as it is in single quotes and so not expanded by the shell.
In this case, it has been set to `C3`.   The Luna variable need not have the same name: i.e. the following is functionally identical:
```
luna s.lst myvar=${s} -s 'EPOCH & STATS sig=${myvar}'
```
Finally, even if the shell variable `${s}` exists, the following would _not_ work:
```
luna s.lst -s 'EPOCH & STATS sig=${s}'
```
i.e. because there is no `${s}` Luna variable defined as on this run.


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

