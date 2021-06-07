
# Updates, additions and fixes

Current stable version: __v0.25.5__ (main [downloads](download/index.md) page)

<!--
 - Development version: __v0.26__ ([source only](http://github.com/remnrem/luna-base))
 - Known issues: [this page](known-issues.md)

## v0.26

_New commands_

 - new `REBASE` command, which adopts the `SOAP`](ref/suds.md#soap) framework to (probabilistically) re-estimate sleep stages using a different epoch duration (e.g. to translate from 20-second manually scored datasets to 30-second epochs) given a) manual staging in the original epoch duration, and b) one or more signals (i.e. EEG) that are expected to encode sleep stage information well (i.e. have a high kappa from the original `SOAP` command).

_Modifications and fixes_

 - `MS` has new `add-spc-sig` option to add spatial correlations as new EDF channels (instead of 0/1 binary variable, as per `add-sig`)

- `MS` has new `canonical` option to specify a file

- `MS` solutions now always have a header row; you cannot extract based on sol=file,A,B,C,D ; also, 'unassigned' states are labeled 1,2,3, etc not A,B,C,...


- [ midflight ] adding `ORDER` command (or that command-line 'sig' sets order)

- loops in scripts [todo]

- channel range selector [C3][C4] .. use same syntax as [C3][1..4] .. will have to wait and expand on a per-EDF basis ... 

- add EDF+ to EDF converter  (zero-pad and add MASK for discontinuities)

add an explicit COUPL command that uses caches

- __Global coherence statistics__ via the [`SYNC` command](ref/cc.md#sync)  

-->


## v0.25.5 (24-May-2021)

_Major new functionality_

  - Major changes to the epoch-wise __artifact detection__ via `SIGSTATS`, which is now implemented
    via the [`CHEP-MASK`](ref/artifacts.md#chep-mask) and [`CHEP`](ref/masks.md#chep) commands, as
    illustrated in [this vignette](vignettes/chep.md)

 - Significant __speed improvements__ for spectral analyses (`PSD` and
    especially `MTM`) and other (e.g. `PSC`, `SUDS`, `ICA`, `CPT`)
    commands through code refactoring, tweaking our use of FFTW and
    the incorporation of the Eigen matrix library

- A beta-version of our new Luna-powered __NAP__ (NSRR Automated Pipeline) __cloud-based portal__, described [here](nap.md),
  and available for testing from this URL [http://remnrem.net/nap/](http://remnrem.net/nap/) populated with public
  [Sleep-EDF dataset](https://physionet.org/content/sleep-edfx/1.0.0/). 

- __Principal spectral components analysis__ via the [`PSC` command](ref/psc.md) 

 - __Phase slope index connectivity__ via the [`PSI` command](ref/cc.md#psi)

 - __Epoch-wise spherical spline interpolation__ via the [`INTERPOLATE` command](ref/spatial.md#interpolate)

 - __Independent components analysis__ via the [`ICA` and `ADJUST` commands](ref/ica.md)

 - __EEG microstate analysis__ via the [`MS` command](ref/ms.md)

 - __Sleep stage prediction and evaluation__ via the [`SUDS`](ref/suds.md#suds) and [`SOAP`](ref/suds.md#soap)
   commands (with supporting `RESOAP` and `--copy-suds` commands) 

- __Time-series clustering__ of epochs and/or channels via the [`EXE` command](ref/clustering.md#exe)

- __Cluster-based non-parametric linear models__ for association analysis via the [`CPT` command](ref/assoc.md#cpt)

_Other new commands/functionality_

 - Added `cache-metrics` options for `PSD`, `MTM`, `COH`, `PSI`, `SPINDLES` and `SO` (i.e. primarily for use with the `PSC` command)

 - `SO` caches negative peaks (with the `cache` option)

 - New [`MEANS` command](ref/intervals.md#means) to give signal means conditional on annotations
 
 - New [special variables](ref/spatial.md#special-variables) describing topographical EEG 64-channel groups, e.g. `${anterior}`, `${midline}`, etc, 
 
 - Find peaks in signals and subsequently produced peak-locked averages across other channels, via the [`PEAKS` command](ref/intervals.md#peaks) 
   and the [`TLOCK` command](ref/intervals.md#tlock) 

 - Added a `peaks` option to the `PSD` command

 - Cross-correlation and phase synchrony metrics, via the new [`TSYNC` command](ref/cc.md#tsync)

 - New [`A2S`](ref/annotations.md#a2s) and [`S2A` commands](ref/annotations.md#s2a)  to convert anntotations to binary EDF signals channels, and vice versa

 - Restructure EDFs to align records, annotations and epochs via the 
 [`ALIGN` command](ref/manipulations.md#align) 

 - Added the [`EDF` command](ref/manipulations.md#edf) to convert
   EDF+C and EDF+D to standard EDF

 - Added the [`LINE-DENOISE`](ref/artifacts.md#line-denoise) command, using spectrum interpolation to reduce line noise
 
 - Added the [`SEDF` command](ref/outputs.md#sedf) to produce _thumbnail-like_ summary EDFs (to be used in future NAP iterations)

 - Added a simple [`SIGGEN` command](ref/manipulations.md#siggen) to generate test signals (currently only sine waves)

 - Added the [`CONTAINS` command](ref/summaries.md) that indicates
   (via Luna's exit code) whether one or more signals are present in
   the EDF (i.e. helps guide programmatic driving of Luna pipelines,
   to be used in future NAP iterations)

- new output/options for the [`HYPNO`
  command](ref/hypnograms.md#hypno): 1) stage transition counts and
  probabilities, with `PRE` and `POST` factors; also with conditional
  probabilities `P_PRE_COND_POST` and `P_POST_COND_PRE`; 2) it now
  outputs epoch-level stage transition information in full and with
  different naming scheme: `TR_NR2REM`, etc and `TOT_NR2REM`.  3)
  `FLANKING_MIN` and `FLANKING_ALL` instead of `FLANKING_SIM`; 4)
  option `flanking-collapse-nrem` (default T), i.e. flank based on any
  NREM stage (1, 2 or 3) 4) option `req-pre-post`, to only consider
  stage transitions that have `FLANKING_ALL` >= `x` for the `POST`
  stage; defaults to 4 (2 mins). Added 'CONFLICT' and `TOTHR` outputs.

_Minor modifications/fixes_

 - the `epoch` argument of `MASK` accepts `end` as the final epoch number
 - added the `force-edf` and `EDF+D` options to
   [`WRITE`](ref/outputs.md#write) to force writing as EDF (or EDF+D);
   this command also now properly supports EDF+C, EDF+D files properly
   (e.g. in how the EDF start time is changed, etc)

 -  one can now add generic canonical signals via [`CANONICAL`](ref/manipulations.md#canonical); it will not change the
   `SR` if set to missing (`.`); the canonical signal definition now
   includes units

 - [channel _types_](luna/args.md#channel-types) now include specific _reference_ and _independent
   component_ types; further, channel type variables are now
   automatically updated after adding new channels (e.g. added `IC` to
   types; when channels added, now they are typed (i.e. the variable
   is updated by any commands that add channels, ICA --> ${ic} will be
   available afterwards to match on IC_1, IC_2, etc).

 - we now allow `.eannot` to be attached with EDF+C but not EDF+D files

- `SPINDLES` now has an `annot` function to generate an internal
  annotation track (which should be used by `WRITE-ANNOTS` but can also
  be used by, e.g. `TLOCK`, etc.  The old `ftr` format is now retired.

- all commands that use epoch-level sleep stages (`SOAP`, `HYPNO`,
  `STAGE`, etc) will note if an epoch has multiple spanning stage
  annotations (i.e. which might happen if stages and epochs are not
  temporally well-aligned); it now reports tje `CONF` variable
  describing the overlap

- fixed an issue in selecting ranges when annotations do not align
  with sample points (fixed for continuous EDFs; _need to check
  whether EDF+D requires additional tweaks_)

- added the `epoch` option to `HYPNO`, which now give less verbose
  output by default

- added `offset` and `align` to `EPOCH`

- added `annot` output for the `SOAP`command

- added a `regional` MASK which masks epochs surrounded by masked
  epochs

- added `annot`, `hms` and `no-specials` options to `WRITE-ANNOTS`

- now the `HEADERS` command respects the `sig` option; also, some
  changes to variable output names

- fixed the `mkdir` system call during `WRITE` for the Windows
  platform

- the `ANON` command now conforms with EDF+ specifications for the
  null ID

- added the `root` option to the `ANON` command, to specify "dummy"
  IDs, in the form `root_N` where `N` is 1, 2, 3, etc. As the `N`
  count is always from 1 for a given run of luna, this can be
  inconvenient if splitting a sample-list and running in parallel:
  thus the `ids` option as above is also given.

- added the `ids` command-line option to supply an ID mapper [ old ->
  new , tab-delim file ]; this can be used with the `ANON` to set the
  EDF header IDs

- `--build` has an option to add quotes around file paths if they contain spaces

 - the [`CLOCS`](ref/spatial.md#clocs) command now has an option
   `verbose`, to dump pairwise channel distances

 - multiple changes to the [`.annot`](ref/annotations.md#annot) file
   format: allow durations (+seconds); allow hh:mm:ss elapsed time
   (0+hh:mm:ss offsets); allow fractional seconds in hh:mm:ss
   specfication; added channel labels and six-column format (but
   allowing reduced 3-col and 4-col formats for backwards
   compatibility); headers now optional; allow space/tab or tab-only
   delimiters; fixed an off-by-one-time-point glitch with clocktime
   specification; now fully allow 0-duration annotations [a,a); allow
   `...` in the stop field to read until the start of the next; in all
   imports, times are scalled to 1/1000th second resolution to avoid
   floating-point nastiness to cause too much trouble; added
   `sep-dp=N` option to control the decimal place in `.annot` outputs;

 - added `fac` instead of `bin` for PSD (but, in general, removing
   support/documentation around this).
  
- for `SIGNALS` command, added the `req` option to _require_ that
  specified signals are present/kept; in contrast, the original `keep`
  option now will not complain if a requested signal is not present
  (i.e. `keep` implies keep if present)

- added the `id` command-line option, so that a numeric ID can be
  specified (i.e. and not be interpreted as a sample-list line number)

- added a `min` option to `EPOCH`, which gives minimal output: just
  the epoch count to standard output

- added a `dump` option to [`STAGE`](ref/hypnograms.md#stage) to write stages to standard out
  (i.e. parallels `eannot` option, but does not write to a file)

- added `guess` (and `eeg`) options to the `CANONICAL` command, to
  guess `csEEG` without a file/group specified


_Internal_

- added `cache_t` for `PSC` command `SPINDLES`/`SO`, and [in flight]
  adding a `COUPL` command to separate out spindle/SO detection from
  coupling analyses

- trims trailing whitespace from the _physical dimension_ and
  _transducer type_ fields in the EDF header; this was already done
  for channel labels

- swapped in functions and classes from the Eigen matrix library for
  many numerical intensive procedures

- added a runs test to the stats helper function library


## v0.24 (22-August-2020)


_New commands_

 - [`ALIASES`](ref/summaries.md#aliases) command to list any channel/annotation aliases done for that individual

 - [`MINMAX`](ref/manipulations.md#minmax) command to set EDF header digital/physical min/max
   to same values across multiple channels

 - [`TYPES`](ref/summaries.md#types) command and accompanying options `ch-match`, `ch-exact` and
   `ch-clear`, which groups channels by [_types_](luna/args.md#channel-types) (e.g. EEG, EMG,
   respiratory effort, etc) and defines correspondong automatic
   variables (e.g. `${eeg}` and `${emg}`); the `HEADERS` command
   now also outputs a `TYPE` field

 - [`CANONICAL`](ref/manipulations.md#canonical) command to generate _canonical signals_,
   `cs_EEG`, `cs_LOC`, `cs_ROC`, `cs_EMG` and `cs_ECG` 

 - [`VARS`](ref/summaries.md#vars) command to output all variables defined for a given
   individual (designed to be paired with the new individual-level
   variables feature, via the [`vars`](luna/args.md#individual-variables)
   option)

 - [`CC`](ref/cc.md#cc) command for connectivity and coupling statistics 

 - [`ACF`](ref/power-spectra.md#acf) command to calculate signal autocorrelation
 

_New options/behaviors for existing commands_

 - added `SENS` and `POS` variables to [`HEADERS`](ref/summaries.md#headers) channel-level output
 
 - added `flat` and `max` options to [`SIGSTATS`](ref/summaries.md#sigstats)

- [`CWT-DESIGN`](ref/power-spectra.md#cwt-design) now outputs FWHM in
   time and frequency domains, and accepts newer FWHM-basewd
   specification (instead of cycles); see [this
   manuscript](https://www.biorxiv.org/content/10.1101/397182v1.full.pdf)
     
 - added the `wrapped` and `fwhm` options to [`CWT`](ref/power-spectra.md#cwt)
 
 - added the `fft` option to [`FILTER`](ref/fir-filters.md#filter) for FFT convolution

 - [`FILTER`](ref/fir-filters.md#filter) and [`FILTER-DESIGN`](ref/fir-filters.md#filter-design)
   can now read a FIR from a `file`; also, 
   instead of Kaiser windows to design a FIR, one can now fix the
   filter order and specify the type of window (e.g. Hamming, Blackman, etc)

- added a `new` option for the [`REFERENCE`](ref/manipulations.md#reference) command, and the ability
  to specify `.` as a reference (i.e. to do nothing, to facilitate
  automatic processing)

 - added summaries to [`CORREL`](ref/cc.md#correl), and improved the speed of this command

- allow now a comma-delimited list when defining stage labels for
  [`STAGE`](ref/hypnograms.md#stage) and [`HYPNO`](ref/hypnograms.md#hypno), e.g. `N1=n1,NREM2`

 - added `epoch` and `per-spindle` options to the [`SPINDLES`](ref/spindles-so.md#spindles),
   making this level of output optinal, and omitted by default (i.e. for `E`x`F`x`CH` and `N`x`F`x`CH`) 

 - new added `cstats` and `astats` options for [`SIGSTATS`](ref/summaries.md#sigstats)

 - added the `channels` option to the [`DESC`](ref/summaries.md#desc) command
   (to write a simple list of all channel labels to standard output)

 - the `ANNOTS` command now outputs `START_HMS`/`STOP_HMS` and `START_ELAPSED_HMS`/`STOP_ELAPSED_HMS`
   under `ANNOT` x `INST` x `T` output strata, which are the _hh:mm:ss_ version of clock-time and elapsed
   time since EDF start, respectively

_Other changes_

 - default log output is now less verbose (unless `verbose=1` set)
 
- small fixes in processing EDF fields: the reserved field is now set
   to space (rather than null) characters; also, all EDF header fields
   (including the reserved field) are now checked for being within the
   7-bit US-ASCII range 32-126 (any character outside this range is
   changed to a `?` character)

 - changed behavior to replace [spaces in channel names](luna/args.md#spaces-in-channel-names)
   with underscore (`_`) characters for ease of processing; setting the `spaces` variable
   can specify alternate replacement characters; `keep-spaces` option, if true,
   means that spaces are retained 
 
 - added support for individual-level variables to be loaded from a file
   (via the [`vars`](luna/args.md#individual-variables) option)

 - added automatic [special variable `${id}`](luna/args.md#individual-variables),
   which can act in the same manner as the `^` special
   character in scripts, i.e. to represent the current EDF ID

 - added the `add=` option to turn on/off conditional blocks

 - added a check that any variable defined in a script does not clash
   with a special variable (i.e. command line _option_).

 - added `include={file list}` option, to mirror `exclude={file list}`
 (nb. you cannot have both exclude and include)
 
 - Luna can now read gzipped (compressed) ASCIIs directly

 - added a check that named `TAG`s do not clash with existing,
   internal tags, e.g. `F` or `CH`

 - fixed an issue with 0-duration annotations, e.g. as may occur with
   _marks_ from `EDF Annotations` channel

 - fixed an issue with the `ANNOTS` command not reporting all output
   if the `EPOCH` command hadn't been explicitly called first

 - Luna now explicitly gives an error message if trying to use an
   `.eannot` file (or `e:1` notation in an `.annot` file) with EDF+
   (i.e. which may be discontinuous).  That is, if the input file is
   an EDF+, Luna requires annotation formats with exact times
   (i.e. either seconds elapsed since the start of the EDF, or
   _hh:mm:ss_ format).
 
 - added [EDF+ Annotation parsing](ref/annotations.md#luna-annotations);
  Luna now reads all `EDF Annotations`
  events as an annotation track, which is automatically combined with
  any other annotations (e.g. from XML or `.annot` files).  Currently,
  all annotations have the class `edf_annot`, which values are
  instance IDs/label. 

 - added new options to [force/skip annotations](luna/args.md#other-annotation-options):
 `force-edf`, `skip-annots`, `skip-edf-annots` and `skip-all-annots`.

 - added support for using braces `{` and `}` instead of `'` to denote string literals in _eval_ expressions
 (i.e. which can be convenient if writing expressions on the command line with `-s` where `'` is already
 used)
 
 - changed formatting of `-t` [text table](luna/args.md#text-tables) file names;
  added `--tt-prepend` and `--tt-append` options (equivalently, `--tt-prefix` and `--tt-suffix`)

 - `SPANNING` now works on EDF+D files

 - `HEADERS` now reports `EDF_TYPE`


## v0.23 (15-Jan-2020)

 - new __help function__ ([`-h` command line
   argument](luna/args.md#help)), to list commands, parameters and
   their output tables/variables

 - new __spindle/slow oscillation coupling [permutation
   option](ref/spindles-so.md#)__ and output variables

 - provisional support for [__text-table output__](luna/args.md#text-tables) mode with `-t` option,
   optionally writing gzipped text (with corresponding _lunaR_
   [`ltxttab()`](ext/R/ref.md#ltxttab) function

 - new option to __read data from a__ [__plain text
   file__](luna/args.md#plain-text-input) rather than an EDF

 - __automatically generate sample lists__
   with a new [__--build__](luna/args.md#-build-option) option, i.e. to
   recursively find all EDFs and match annotation files in a set of
   folders

 - added support to __read and write compressed EDF files__, via the
   [EDFZ](luna/args.md#edfzs) format, as highlighed in [this
   vignette](vignettes/edfz.md)

 - __new scripting features:__ command files now allow [_conditional
clauses_](luna/args.md#command-files), via the `[[var`  syntax, [_within-script variable
definitions_](luna/args.md#variables), via the `${var=xyz}`syntax, and [expanding numeric
sequences](luna/args.md#variables), e.g. `[ICA][1:10]` expands to
`ICA1,ICA2,ICA3,...`

 - new [`SEGMENTS`](ref/outputs.md#segments) command to show contiguous time intervals within a discontinuous EDF+

 - can now [write EDF+ files](ref/outputs.md#write)

 - new _experimental_ commands for __multi-channel EEG__:
[`CLOCS`](ref/cc.md#clocs) (to read channel locations), 
[`CHEP`](ref/cc.md#chep) (channel/epoch masks), 
[`INTERPOLATE`](ref/cc.md#interpolate) (to interpolate missing channels/epoch based on spherical splines), 
[`SL`](ref/cc.md#sl) (to compute the surface Laplacian), and 
[`ICA`](ref/cc.md#ica) (implementation of the fastICA algorithm).

 - added individual-ID wildcards in the specification of output
   database names (`-o` or `-a`), i.e. to write to a separate database
   for each EDF/individual, e.g. `-o path/to/out-^.db`

 - [`COH`](ref/cc.md#coh) performance has
   increased, and it now reports imaginary and lagged coherence; also,
   can accept `sig1` and `sig2` parameters to more flexibly specify
   which pairs of channels are considered

 - the `CHS` output factor is now split into two separate factors, `CH1` and `CH2`, for `COH` `CORREL` and `MI`

 - added [`SPANNING`](ref/annotations.md#spanning) command to report on
   "coverage" of an EDF by one or more annotations

 - added special variables `silent` to turn off all console logging
 
 - added `--xml2` command line option for verbose view of XML tree

 - removed the `skip-annots` special variable; replaced with two new
   variables: `skip-edf-annots` and `skip-all-annots`

 - the [`MASK`](ref/masks.md#mask) `epoch` and `mask-epoch` parameters
   can now take comma-delimited lists of ranges,
   e.g. `epoch=1,6,8-10,22`

 - changed the behavior of
   [`assume-pm-start`](luna/args.md#force-evening-start-time); it is now off by
   default, and accepts a time parameter to define whether ambiguous
   times are assigned AM or PM values

 - added `START_TIME`, `START_DATE` and (optionally) `SIGNALS` output
   variables to the [`HEADERS`](ref/summaries.md#headers) command
 
 - now make new, unique labels for any duplicate channel labels found
   in an EDF (e.g. EEG, EEG.1, EEG.2, etc) and writes a message to the
   log

 - now check whether the same channel _alias_ points to more than one
   channel, and if so, will give an error message

 - can now use _hh:mm:ss_ clock-time format in `.annot` files (assumes
   24 hour; requires start & stop specified)

 - changed clock-time format from `hh:mm:ss` to `hh.mm.ss` for
   compliance with the EDF spec; Luna can still read clock-times with
   the colon-delimiter however

 - new mode (`-a` instead of `-o`) to append to, rather than overwrite, output databases

 - fixed an issue with the `MTM` command's `epoch` option, when used with
   multiple signals

 - fixed issue with command parameters not being recorded in the output
   database

 - added the `nsrr-remap` option to turn off [auto-remapping of NSRR
   annotations](nsrr.md#annotation-aliases)

 - new [`RECS`](ref/outputs.md#recs) and [`SEGMENTS`](ref/outputs.md#segments) commands to dump basic EDF record
   and interval info to stdout

 - added a `fail-list` option [note: _need to check implementation_]

 - fixed issue with `ANNOTS` where an annotation was flagged as
   overlapping a region/epoch is it ended exactly 1 time-unit
   beforehand (i.e. ignored convention that intervals are internally
   represented as [start] to [one past the end] of the interval).

## v0.22 (31-March-2019)

Initial public release.

