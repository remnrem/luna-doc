
# Updates, additions and fixes

Current stable version: __v0.28__ (main [downloads](download/index.md) page)

<!--
IN FLIGHT / TODO
  - `RIPPLES`
  - `GED`
  - `NMF` 
  - `ORDER` command (or that command-line 'sig' sets order)
  - loops in scripts [todo]
  - channel range selector [C3][C4] .. use same syntax as [C3][1..4] .. will have to wait and expand on a per-EDF basis ... 

  - add EDF+ to EDF converter  (zero-pad and add MASK for discontinuities)
     add an explicit COUPL command that uses caches
 
  - __Global coherence statistics__ via the [`SYNC` command](ref/cc.md#sync)  
  - prototype [`TCLST`] command for time-series clustering

_Predictive modelling_ 
 - adding [LightGBM]() library support and compilation flag
 - new [`ASSOC`] command
 - added new `MASSOC` set of functions

_LunaR_
 - `lunaR` is now in CRAN
 - documention (e.g. `?leval`) added to the lunaR package 
 - new `lload()`, `lhead()` and `lcols()` convenience functions 
 - additional `lheatmap()` options added

-->


## v0.29 (not-yet-released )

 - fixed `stratify-by-phase` output to avoid double-couting
 
 - added `bins` options to `S2A` and `bins-label` - given `bins=min,max,n` to make `n` bins of equal span, to make annots `B1`, `B2`, etc... `Bn` (or `bins-label` instead of `B`)
 
 - added `REC_DUR_SEC` and `REC_DUR_HMS` in `HEADERS` which used to be `TOT_*`.  Now, `TOT_DUR_SEC` and `TOT_DUR_HMS` reflect the full duration, including any gaps (i.e. if EDF+D)

 - `RESAMPLE` has a new `downsample` argument; only channels with rates above the `sr` value
 
 - can now correctly take `.edfz` and `.edf.gz` file on the command line (versus a sample list) 

 - `SPINDLES` and phsae-coupling
 
 - `PEAKS` now output annotatons (`annot`) as well as cache; can include `w` to add window (sec) around each point

 - upgraded Eigen library to 3.4.0

 - upgraded to sqlite library v3.41.2
 
 - added `EDGER` tool: (slides/ for opts)
 
 - added `ngaus` option to `FILTER` to implement narrow-band filter
   via a frequency-domain Gaussian (parameters: frequency domain
   central mean and FWHM of the Gaussian, `ngaus=11,2`)


## v0.28 (10-Apr-2023)

_Moonlight_

 - new interactive [_Moonlight_](moonlight.md) viewer (public demostration host: [http://remnrem.net](http://remnrem.net))

 - extended [tutorial](tut/tut5.md) that uses _Moonlight_ to recapitulate the prior Luna/lunaR tutorials 

_Vignettes_

 - new vignette on [merging EDFs](vignettes/merge.md) and working with EDF+D files

 - new vignette on how Luna expects [time/date information in annotation files](vignettes/times.md)

_Major new functionality_

 - new [`FREEZE`](ref/freezes.md#freeze) and
   [`THAW`](ref/freezes.md#thaw) commands to make and revert to prior
   snapshots of the in-memory dataset

 - new [`TABULATE`](ref/summaries.md#tabulate) option to summarize
   channels with discrete sets of values, e.g. body position

 - new [`MOVING-AVERAGE`](ref/manipulations.md#moving-average) command 

 - major improvements to [`--merge`](ref/helpers.md#-merge) which can now handle gaps between consecutive files and will generate EDF+Ds

 - new `add` option for [`MTM`](ref/power-spectra.md) to generate a
   new signal (with the same sample rate as the original); also, `MTM`
   now outputs estimates of the spectral slope as well as `PSD` and
   `IRASA`.

 - new [`SET-TIMESTAMPS`](ref/manipulations.md#set-timestamps) utility command

 - new `annot` option for the [`SEGMENTS`](ref/outputs.md#segments)
   command; as demo'ed in [this vignette](vignettes/merge.md), it is
   useful when making standard EDF from EDF+D files
 
 - for [`WRITE-ANNOTS`](ref/annotations.md#write-annots) we now make
   as default `no-specials=T`; i.e. add options `specials` instead of
   `no-specials`; and likewise `headers` from `no-headers`.  We've also added `dhms` and `minimal` options.

 - added `impulse` option to [`SIGGEN`](   (T,A,D)  and `add` not clear; makes new channle, or updates/replaces an existing one. 

 - new [`REMAP`](ref/annotations.md#remap) command to perform annotation remapping on _already-loaded_x data;
 
 - new [`EXE`](ref/clustering.md#representative-epochs) option `representative` to extract _K_ examplar epochs via a clustering heuristic based on permutation distribution clustering
 
_Minor changes/additions_

 - added [`starttime`](luna/args.md#set-edf-start-time) and
   [`startdate`](luna/args.md#set-edf-start-date) command-line special
   variables to force EDF header changes (prior to attaching
   annotations)

 - new [`anon=T`](luna/args.md#anonymize-edf-headers) special variable to set EDF headers to null values
   upon loading; this is similar to runnong [`ANON`](ref/manipulations.md#anon), although `ANON`
   allows a few extra options); as with `starttime` and `startdate`,
   this setting is enacted _before_ attaching annotations (i.e. and so
   can influence how annotations are aligned)

 - added support for [dates in annotation files](ref/annotations.md), e.g. _dd-mm-yyy-hh:mm:ss_, as
 illustrated in [this vignette](vignettes/times.md)

 - added the `pct` option to [`STATS`](ref/summaries.md#stats) to avoid calculation of percentiles `pct=F`. Also `min=T` will only give the mean.

 - [`SOAP`](ref/soap.md) now respects _lights on_ (`L`) epochs, i.e. unlike for missing (`?`) epochs, it completely excludes them

 - new `offset` option for [`WRITE-ANNOT`](ref/annotations.md#write-annots)

 - Luna now gives a warning message to the console if looking at an
   etnire signal (e.g. from `FILTER`) but some epochs are masked -
   i.e. as this means that those epochs will be included in that step
   (if not running `RE` beforehand to remove them)

 - added `annot` feature to [`DUMP-MASK`](ref/masks.md#dump-mask) to create annotation based on the mask (as well as `output=F` and `annot-unmasked=T`)

 - added `file` option to [`RENAME`](ref/manipulations.md#rename) - takes a file of tab-delimited old/new channel name pairs instead of `sig`/`new` from the command line
 
 - added the `xbg` option to specify intervals to excise certain intervals from the background in [`OVERLAP`](ref/intervals.md#overlap)

 - `offset` option of [`EPOCH`](ref/epochs.md#epoch) can now take _hh:mm_ and _hh:mm:ss_
   arguments, as well as elasped seconds

 - now gives an error if specified filename for a new EDF file
   (e.g. from [`WRITE`](ref/outputs.md#write)) is the same as the input EDF. (This was allowed
   previously, but could cause problems and lead to corrupt outputs...)

 - `anchor` argument for [`SPINDLES`](ref/spindles-so.md#spindles) to
   specify between -1 and +1 for spindle start stop (0 = midpoint) for
   SO-coupling analysis.  If not specified, default is the point of
   maximal CWT (i.e. typically, but not necessarily near the middle);
   `offset=0.1` (-N ro +N seconds, default = 0) adds offset to above
   _anchor_ for SO-coupling, i.e. can be combined with `couple`,
   i.e. look at SO phase and overlap 0.2 seconds *before* the start of
   a spindle; Gives new `ANCHOR` strata in output for coupling
   analyses.  Can specify multiple anchor/offsets.

 - EEG microstates `--label-maps` command now outputs
```
ID	KT	FLIP	K1	MAPPED	SPC
.	D	0	1	1	0.994548052142096
.	C	0	2	1	0.972204262924326
.	A	0	3	1	0.988131018834142
.	B	0	4	1	0.996294406621555
```
 and (will) add a message if match is not highest correl SPC `OPTIMAL == 0 `

 - also, it now allows to matching on the minimum sum `(1-r)^p` where `p = 2` by default;
  this is now also the default stat for `compare-maps` (perm.) tests; added `verbose` option for info to the log

 - for EEG microstate analysis, added a `grouped` option to allow A/a --> 'A' two-group mappings (i.e. case-invariant analysis, if K = 4 + 4 = 8 
 

_Misc. fixes_

 - [`HYPNO`](ref/hypnograms.md#hypno) now outputs proper epoch time, i.e. if used EPOCH align so
 that epochs are not starting from 0; (as is, it assumes epoch 1
 starts at 0 sec); also note that `MINS` is the elapsed time since epoch #1,
 not necessarily EDF start under these conditions

 - annotation class/instance delimiter has been changed to have a default value of `:` (rather than `/` as before)

 - NSRR automatic-remapping is now turned off by default: enable with `nsrr-remap=T`; added `annot-remap` to separately control remapping of stage annotations

 - automatic _sanitization_ of annotation labels is now on by default

 - changed behavior of `keep-spaces` and `sanitize` to be similar for both channel and annotation labels; also, labels are now always trimmed
 on both whitespace and sanitized insert character (underscore by default)
  
 - stopped the `HEADERS` command from writing out EDF Annotation channels

 - changed lunaR to attach EDF+ annotations on attaching an EDF+
 

## v0.27 (27-Sep-2022)

In addition to multiple smaller fixes and modifications, this release
contains a number of major new additions including: 1) revised
macro-architecture summary statistics, 2) the SOAP and POPS tools for
automated prediction/evaluation/manipulation of sleep stages, 3) new
tools for assessing temporal coupling of annotation events, and 4) new
signal processing tools including IRASA.  Major changes are noted below:

_Macro-architecture_

 - revised and expanded hypnogram summary metrics, and improved
 documentation for the [`HYPNO`](ref/hypnograms.md#hypno) command
 including: bout number/duration metrics, better incorporation of
 _lights out_ markers, a heuristic to handle excessive
 leading/trailing WASO, revised calculation of total persistent sleep
 time (now `TST_PER`), renamed sleep efficiency/latency (`SE`, `SME`
 and `SOL`), added final wake time (`FWT`).

 - added a new `annot-cycles` option for
  [`HYPNO`](ref/hypnograms.md#hypno), to add an annotation indicating
  which NREM cycle an epoch belongs to

 - fixed an issue when [`HYPNO`](ref/hypnograms.md#hypno) is called twice on the same EDF
 
 - added `first` (mins) and `first-anchor` (`T0`, `T1` or `T2`)
   options, to report hypnogram stats for only the first _N_ minutes
   (setting the rest to `L`)

 - fixed a bug impacting some of the elapsed time metrics from
  [`HYPNO`](ref/hypnograms.md#hypno) (`E_*` etc) if the recording started
  at or after midnight

 - added and documented the new [`SOAP`](ref/soap.md#soap),
   [`REBASE`](ref/soap.md#rebase) and [`PLACE`](ref/soap.md#place)
   stage evaluation/manipulation commands
   
 - added a new automated staging command:
  [`POPS`](ref/pops.md#pops) - _POPulation-based Staging_ that
  implements a flexible syntax for model specification.  Internally,
  POPS uses Microsoft's popular open-source C/C++
  [LightGBM](https://lightgbm.readthedocs.io/en/latest/) library for
  machine learning

 - _Internal_: have depreciated the older (but still experimental)
  SUDS stager; previously, we had added some new features, including a
  canonical correlation based routine.
 

_Data manipulation/handling_

 - major new implementation and syntax for the
   [`CANONICAL`](ref/canonical.md) command 

 - new [`SET-HEADERS`](ref/manipulations.md#set-headers) command to directly edit EDF header values

 - new [`ENFORCE-SR`](ref/manipulations.md#enforce-sr) command 

 - new [`fix-edf`](luna/args.md#fix-truncated-edfs) option, to allow
   (i.e. ignore) truncated final records of a trivially corrupted EDF 

 - new [`--merge`](ref/helpers.md#-merge) command to aggregate multiple EDFs
 
 - the [`--repath`](ref/helpers.md#-repath) utility now accepts `.` as first character,
   meaning always append the second argument if (and only if) the
   sample list has a relative path

 - new [`SET-VAR`](ref/manipulations.md#set-var) command to set individual-level variables from within a script

 - new [`RENAME`](ref/manipulations.md#rename) command to rename channels (unlike signal [aliases](luna/args.md#aliases), this command can accept [variables](luna/args.md#variables))

 - [`HEADERS`](ref/summaries.md#headers) now outputs `STOP_TIME`
 
 - added `pairwise` option to [`REFERENCE`](ref/manipulations.md#reference)

 - new [`RECTIFY`](ref/manipulations.md#rectify) command

 - new [`REVERSE`](ref/manipulations.md#reverse) command


_Micro-architecture/signal processing_

 - new [`IRASA`](ref/power-spectra.md#irasa) command, to implement
   [Irregular-Resampling Auto-Spectral
   Analysis](https://pubmed.ncbi.nlm.nih.gov/26318848/)
 
 - new prototype/alpha implementation of detrended fluctuation
   analysis via the [`DFA`](ref/exp.md#dfa) command

 - added Petrosian fractal dimension and permutation entropy (`pe`,
   with `pe-m` and `pe-t` options) to [`SIGSTATS`](ref/summaries.md#sigstats)

 - new prototype [`ASYMM`](ref/exp.md#asymm) command to evaluate
 (stage-specific) (regional) asymmetries (e.g. inter-hemispheric
 differences) in derived signal metrics
  
 - new [`Z-PEAKS`](ref/intervals.md#z-peaks) command to make annotations based on peak finding in signals

 - added `segment-median` and `segment-sd` to [`PSD`](ref/power-spectra.md#psd)

 - added `precomputed` function to `SPINDLES`

 - added `ch-median` `ch-epoch` `ch-spatial-threshold` `ch-spatial-weight` to `CORREL`,
   which now also prints out disjoint sets of "high corr" channels (`CHS`)

 - added [`OTSU`](ref/helpers.md#otsu) and `--otsu` commands; changed Otsu implementation
  
 - added the [`--ftt`](ref/power-spectra.md#fft) command

 - added the `same-channel` option for
   [`TLOCK`](ref/intervals.md#tlock) to constrain output for when `CH`
   == `sCH`

_Annotations_

 - new [`OVERLAP`](ref/intervals.md#overlap) command to evaluate enrichment of overlap/locality of annotations,
   in both single- and multi-sample contexts; it can also produce new annotations based on overlap with other annotations

 - added the `collapse` argument for
   [`WRITE-ANNOTS`](ref/annotations.md#write-annots) which can be used
   with an EDF+D if an EDF will be output (i.e. changes the times as
   if collapsing discontinuities)

 - currently largely for internal development use, a new [`A2C`](ref/exp.md#a2c) command
   to convert sample points (ints) in annotation meta-data to a cache
   store

 - `EPOCH align` now works for EDF+D - will align the first epoch at
   the start of each segment, i.e. still assuming uniform epochs after
   that (within each segment)
   
 - `skip-edf-annots` now still reads time-track for EDF+D from `EDF
   Annotations` channels

 - added the `numeric-inst` argument to [`A2S`](), to set a signal to
   a numeric-valued _instance ID_ (i.e. not just 0/1)

- when creating a set of epoch-level sleep stages, Luna will automatically detect if
  the stage annotation are 0-duration change-points (as is often the case in EDF+ stage annotations)
  and will extend those new signals up until the next stage annotaiton encountered (or the end of the recording)

 - alternatively, unless `assume-stage-duration=F`, if sleep stages have 0 duration
   (e.g. markers for change) assume they have the default epoch
   duration length (currently only for EDF+ embedded annotations)

 - added the `add-ellipsis` option (for zero-duration annotations) --
   this impacts [`WRITE-ANNOTS`](ref/annotations.md#write-annots) `.annot` format files only

 - for [EDF+ annotations](ref/annotations.md#edf-annotations),
   added `edf-annot-class` special variable, to make these annots
   classes (instead of `edf_annot_t` instanes)
   e.g. `edf-annot-class=N1,N2,N3,R,W`


_Microstate analysis_

 - the EEG microstates command [`MS`](ref/ms.md#ms) has a new `w` argument for `--kmer`
   analysis, which performs local shuffling of microstate sequences (i.e. only within _N_ states)

 - the microstate [`--kmer`](ref/ms.md#-kmer) command will now automatically splice out `?` states, and
   reduce adjacent states to one e.g. `AAA??AABBBCCC` -> `ABC`

 - a new `--label-maps` command assign labels for a microstate map
   `sol` file, given a labelled canonical/template `sol` file (template,
   file, new)

 - new [`--cmp-maps`](ref/ms.md#-cmp-maps) command to test for
   differences between maps, either at the group or individual (one
   versus all others) level, using a permutation procedure

 - new [`--correl-maps`](ref/ms.md#-correl-maps) to print spatial
   correlations for a map (from a text file)
 


_Misc_

 - added `--options` command to allow command-line functions to take args either from stdin OR from command line args (following --options)

 - changed parameter parsing to include added signals when no `sig`
 specified (i.e. match all) in the case when `sig` was specified as an
 initial, top-level special variable

 - param files have `+group` and `-group` flags.
                      // +group  include only if matches group, otherwise skip
                      // -group  exclude if matches group, otherwise parse

-  `IF` and `IFNOT` (or `ENDIF` / `FI` )

 - fixed bug in within-record interval offset calculation for EDF+D
   when gaps present that are fractions of a record duration (i.e. and so
   record start time-points are no longer multiples of the EDF record size)
  




## v0.26 (29-Nov-2021)

_New functionality_

 - [`TRANS`](ref/evals.md#trans) supports arbitrary transformations of signal data

 - [`SIMUL`](ref/simul.md#simul) simulates time-series data given a power spectrum

 - [`PSD`](ref/power-spectra.md#psd) now has
   [`peaks`](ref/power-spectra.md#peaksspikes) and
   [`slope`](ref/power-spectra.md#spectral-slopes) options
 
- [`FFT`](ref/power-spectra.md#fft) performs basic discrete Fourier transform (DFT) via the FFT

 - [`HEAD`](ref/outputs.md#head) shows one epoch of data (requires same SR for selected channels)

 - [`ZC`](ref/manipulations.md#zc) mean-centers signals, and [`ROBUST-NORM`](ref/manipulations.md#robust-norm)
   performs robust normalization (by median & IQR)

 - upated the [`EMD`](ref/power-spectra.md#emd) (empirical mode decomposition) command 
 
 - [`--repath`](ref/helpers.md#-repath) convenience sample-list function
 
 - prototype [`ALTER`](ref/artifacts.md#alter) command to perform reference-channel, regression-based artifact removal

 - `peaks` and `slope` options for `PSD` and `MTM`

 - [`REBASE`](ref/soap.md#rebase), which adopts the
   [`SOAP`](ref/soap.md#soap) framework to (probabilistically)
   re-estimate sleep stages using a different epoch duration (e.g. to
   translate from 20-second manually scored datasets to 30-second
   epochs) given a) manual staging in the original epoch duration, and
   b) one or more signals (i.e. EEG) that are expected to encode sleep
   stage information well (i.e. have a high kappa from the original
   `SOAP` command).


_Annotation format modifications/extensions_

 - `.annot` format now allows key=value meta-data to be specified; also, you
 can have fewer meta-data terms than expected (but not more), assuming order is as
 header; all fields still are required to be in the header; now
 WRITE-ANNOTS always writes meta-data as _key=value_ pairs

- new `+` and `-` options to turn on/off @includes (e.g. `alias`,
    `remap`); note, the variable must be specified __before__ the
    relevant @include on the command line, i.e. as command-line
    arguments are processed in left-to-right order

  - added a check for pipe (`|`) characters in annotation primary names (in a `remap`)

  - annotation times can now include AM/PM modifiers (otherwise assumes 24-hour clock)

  - new `annot-whitelist` option, such that annotations are only accepted if they appear in the `remap` list (either as aliases or primaries);

  - new `annot-unmapped` option to skip if the annotation _is_ on the whitelist (i.e. complement of behavior with `annot-whitelist` alone)

  - if remapping an annotation to `ABD/DEF|XYZ` form (i.e. with a `/`
    delimiter) then for the class `XYZ`, it is set to `ABC` and
    instance ID is set to `DEF`.  If there was an existing instance
    value (non-null), a text meta-tag of `_inst=` is added.  The
    delimiter character can be changed from `/` with
    `class-inst-delimiter=X` (although this only works for annot files
    currently, and is not generally recommended, i.e. as
    _sanitization_ of labels respects `/` for annotations, etc)
   
 - added `combine-annots` option to merge _class_ and _instance_
   identifiers. It accepts a character argument but is `_` by default
   ); this sets the annotation `class` to `class_inst` (and sets
   `inst` to '.' )

 - added the `skip-sl-annots` option to skip all SL-attached
   annotations; i.e. if wanting to only load annotations from an
   alternate, explicitly referenced annot file

 - added the `interval` option to the `EVAL` command, to generate new
   interval-level annotations based on eval expressions

_EEG microstates_

 - `MS` has new `add-spc-sig` option to add spatial correlations as new EDF channels (instead of 0/1 binary variable, as per `add-sig`)

 - `MS` has new `canonical` option to specify a file definining canonical microstates

 - `MS` solutions now always have a header row; you cannot extract based on sol=file,A,B,C,D; also, 'unassigned' states are labeled 1,2,3, etc not A,B,C,...

 - new `--cmp-maps` command to compare (spatial correlation) EEG microstates

 - `--kmer` command takes options `req-len` (only analyse first _N_ sequences) and `indiv-enrichment`


_Other fixes, minor modifications and new features_

 - added ability to specify an empty EDF (`--nr`, `--rs` and filename equals '.' ) 

 - added 1st, 2nd, 5th and 10th percentiles to `STATS` 

 - added _transition frequency_ to slow oscillation output (`SPINDLES`, `SO`)

 - fixed bug in `flanked` mask option (e.g. `flanked=W,1`)

 - made `CANONICAL` definition file format more flexible: 1) it now allows
   whitespace, not just tab-delimitation; 2) only the first three
   fields are required now; if not given, the latter fields/columns
   will be set to `.`; 3) `CANONICAL` now respects the order of
   canonical signals (i.e. rather than processing things
   alphabetical); this allows _multi-stage_ definitions, e.g. to first
   map `S1`, then `S2`, and then apply a rule such as `S3 = S1 - S2`.
 
 - _eval_ syntax takes `{` and `}` instead of `'` to delimit strings;
  it allows nesting, but can also be handy on the command line
  (i.e. if already using `-s ''` form)

 - added `drop` and `keep` options to the `PSC` command

 - added `import=file.txt` command to `CACHE` to read from destrat
   output; can take `factors` and `v` param (as well as required
   `cache=`)

 - added `MASK epoch=all` to set a MASK but have it all empty ;
   i.e. to trim records not in an epoch
 
 - (for .annot only) added `align` option : given list of
   annots (or *) for all, align-annots-on=N1,N2, etc...  if not
   specified, find first instance of this annot, then align with 1
   second boundary (or `align-annots-res=X` if given) align /all/
   annots with this offset (bound at 0); i.e all records beforehand will
   be skipped if subsequent `MASK`/`WRITE` commands are applied 

 - added `pick` option to `SIGNALS` to pick first of pick=a,b,c that
   is present, and drop the rest; can map with 'rename' to rename the
   pick
 
 - allow sample-lists to have a comma-delim list of annotations, or
   '.' to denote no data; in this way, we can have a fixed width
   sample list (i.e. three tab-delimited columns), making it easier to
   parse (case in point: _lunaR_ `lsl()` was broken if fewer than 3
   columns were found after first five rows, reflecting how R
   `read.table()` works)
 
  - `CANONICAL` does not now need an explicit GROUP to be specified;
   the file must still have a first col, it is just ignored now; also,
   new 'drop-originals' option to drop all original (non-CANONICAL)
   signals after making the new signals; matches case-insentive
  
 - changed `epoch-check` to accept number of `.eannot` epochs that are
   different from expected; default is 5; only stops is absolute
   greater than this; otherwise writes warning to log; i.e. set to 0
   for an exact match
   
 - `CONTAINS` can now skip to the next EDF (rather than alter the return code), if the option `skip` is given

 - check for whether an ID contains the ID-wildcard character (by default, ^) and reports an error if it does;  added the `wildcard` option to specify an alternate character
 


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

 - __Time-series clustering__ of epochs and/or channels via the [`EXE` command](ref/clustering.md#exe)

 - __Cluster-based non-parametric linear models__ for association analysis via the [`CPT` command](ref/assoc.md#cpt)

_Other new commands/functionality_

 - added `DUPES` command to find flat signals and digital duplicates

 - Added `cache-metrics` options for `PSD`, `MTM`, `COH`, `PSI`, `SPINDLES` and `SO` (i.e. primarily for use with the `PSC` command)

 - `SO` caches negative peaks (with the `cache` option)

 - New [`MEANS` command](ref/intervals.md#means) to give signal means conditional on annotations
 
 - New [special variables](ref/spatial.md#special-variables) describing topographical EEG 64-channel groups, e.g. `${anterior}`, `${midline}`, etc, 
 
 - Find peaks in signals and subsequently produced peak-locked averages across other channels, via the [`PEAKS` command](ref/intervals.md#peaks) 
   and the [`TLOCK` command](ref/intervals.md#tlock) 

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
  stage; defaults to 4 (2 mins). Added 'CONF' and `OTHR` outputs.

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

- added date-time support for annotations (in clocktime_t) and `dhms` flag for `WRITE-ANNOTS`

- added test for non-integer sample rates

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

