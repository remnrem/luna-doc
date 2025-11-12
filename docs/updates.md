
# Updates, additions and fixes

Current stable version: __v1.3.2__ (main [downloads](download/index.md) page)

## v1.3.2 (12-Nov-2025)

Note: documentation for new features are currently not fully in place.

_New commands_

 - added `CLIP` to clip signals using either relative or absolute thresholds

 - added `RAI` to calculate an index of REM loss of atonia from the EMG

 - added the `REQUIRES` command with the `version` option

 - added the `DROP-ANNOTS` command

 - added the `ROLLING-NORM` command, which uses an iterative sliding window to calculate a signal mean and SD

 - initial (beta) implementation of heart rate variability (`HRV`) metrics


_Masks & epochs_
 
 - added new `splice-gaps` mode for `EPOCH` generation

 - for multi-day recording support, added `dhms` option to mask: e.g. `dhms=d2/06:00:00-d3/06:00:00`

 - added `stable-unique` and `stable-any` to `MASK`; for example
   `stable-unique=X,A1,A2,...` to select one and only one of the
   annotations `A1, A2, etc; or `stable-any=X,A1,A2,...` which means
   to select any. Here X is the minimum number of flanking epochs required

 - added `MASK random-from=N,a1,a2,...` to set up to N from the set of epochs w/ an annot from [a1,a2,...]
 
 - added `MASK trim=W` ( or `MASK trim=W,10` ) ; takes a single annot

 - added `MASK leading`, `trailing`, `leading-trailing` (w/ mask- and unmask- explicit variants);
    can take multiple annots


_Annotations_

 - now allows datetime to advance (e.g. for `WRITE-ANNOTS dhms`) even if `1.1.85` (null value) is in header
 
 - reimplemented _annotation search_ with more efficient interval-tree implementation
 
 - can now attach annotations only, and it will get start date/time from annots if present
    (and duration)

 - reject `hh:mm` or `mm:ss` format time-strings from annots

 - added `ignore-annot` and `ignore-raw-annot` to complement `annot=X` (i.e. to exclude a particular annotation)

 - added `annot-dir` to `WRITE-ANNOTS`, to be used instead of `file`;
   this creates the folder if it does not exist, same as `WRITE` for `edf-dir`

 - `AXA` allows [x]/[-x] and [][] but not root-matching...
    TODO - make a generic annot=<> parser to handle both
     xsigs and root_match

 - can now drop annotations via `DROP-ANNOTS`


_New functionality in core commands_

 - changed `RECORD-SIZE` to allow for non-integer sample rate
   (i.e. will resample signals as well as changing record size, to
   maintain integer number of samples per record, with a Hz sampling
   rate as close to the original as possible

 - added `start` and `stop` to `OVERLAP`;  [ TODO: describe `rp` options ] 
		    

_Minor additions & changes_

 - only flag as a _CONFLICT_ if overlapping stages are actually in conflict (i.e. now allow overlap stage annotations
   that do not induce substantive conflicts)

 - changed `EDFZ` - no longer use indexed-BGZF access - i.e. no .idx files needed; always _preload_; 
   this then means one can keep EDF annotations as is

 - added `SOF`, `DAR`, `TAR`, `TBR` & `GSI` to `PSD` (both total and per-epoch) via the 'slowing' option _[todo: document]_

 - standardized the calculation of Hjorth complexity: to obtain the
   old form, set the special variable `legacy-hjorth=T`

 - added `REPORT` command with `hide`/`show` options
   -- old REPORT needs 'ensure' ... but trying to update cmddefs()
   -- added compress to REPORT
   
 - added `M1` to `MEANS` - gives range-normed values (0..1) for a given `CH`/`ANNOT` pair,
   i.e. if we have > 2 instance IDs

 - added `S2A` w/ waveforms

 - added `slice=m/n` to `--validate`
  
 - `EPOCH require` now works for generic (annotation-based) epochs

 - added `mirror` options to show commands
 
 - added `show-assignments` special variable : print ${a=2} etc


_Script syntax_

 - new variable types: to complement standard variables (`${x}`), we
   now have loop-indices (`#{i}`) and boolean/optional variables
   (`?{i}`)
 
 - added loop functionality in Luna scripts via `LOOP` / `END-LOOP`
   and `#{idx}` loop-index variables

 - added boolean variables (`?{x}`) that can be used in `IF` statements: if not set, it evaluates to `F`
   (cannot += assign ?{x+=1})
  -- can be used to reference a normal `${var}`, just handles missingness
     differently
  
 - scripts are now dynamically evaluated for variable substitution
   (i.e. line-by-line), allowing dynamic assignment, etc, rather than
   all variables being evaluated prior to executing the script
   
 - `[[` conditional blocks are no longer supported: use `IF` and `FI`
   statements instead

 - we now allow `[x]` and `[-x]` in most major `annot=` specifiers (as well as `LOOP`)

 - new `...` or `+` line continuation syntax in scripts: this means we
   can now indent code (i.e. as it useful for loops) to make the
   script more readable.

 - allow `[][1:3]` form of [sequence substitution](luna/args.md#sequence-substitution); 
   (note - `[][#{x}]` are evaluated later... so can have `[][${a}]` and then `[][#{b}]`)

 - allow parameter file to be tab, space or = delimited (`param-spaces=T/F`, `param-equals=T/F`, both default to `T`)
     e.g.   `var = value`;  `var = "escaped value with = char"`

 - added ability to specify expression at variable assignment, with `:=` operator: e.g.  `${b:=${a}+1}`



_Association model_

 - added `strata=Z1,Z2,Z3` option to `GPA stats`, to give stratified group statistics (mean/SD/N)
 
 - added GPA `comp` and `comp-verbose`, to perform _template/sign
   tests_ for a set of tests, with respect to a prior (independent)
   set of tests [todo: to add documentation]

<!--
Generate the “template” : (this obviously assuming you’ve correctly
compiled the GPA matrix etc:; note, you don’t need empirical p-values
here , this just takes the betas from the standard case/control tests:
naturally, you can add covariates (Z) etc; the Yg specifics the
‘groups’ of sleep metrics - here about 1372 tests
 luna --gpa -o out.db --options dat=gpa/grins-cc.dat X=scz Yg=peaks,olap-ch
  Extract a file of coefficients to text file (cmp.1) [ you could restrict to P < threshold here , probably a good idea  — but you don’t need to be too extreme ] 
   destrat out.db +GPA -r X Y -v B | awk ' NR != 1 { print $2  , $3 , $4 } ' > cmp.1
   This file is just one row = X-var,  Y-var, beta   [ note, the X-var is actually ignored subsequently, except it is needed when reading the file in ] 
   Perform the case-only analysis (or, naturally, this could be an analysis of a different sample, etc — just has to have similar Y vars and to be statistically independent of the template-generating contrast (obvs);  the “comp” option is all you specify 
    luna --gpa -o out.db --options dat=gpa/grins-co.dat X=@{o.1} nreps=200 Yg=peaks,olap-ch comp=cmp.1
--->



_Bug fixes_

 - `sig` failed to find channels with single-character labels (e.g. `F`)

 - fixed issue w/ `dynam` when no cycles are defined

 - `THAW` does not fail if the freeze does not exist, unless `strict`
     is set; i.e. this allows cases where a freeze wasn't made due to
     0-records left



## v1.2.2 (07-Mar-2025)

Various minor additions and a few fixes.

_Signals_

 - for [`GPA`](ref/association.md#gpa), if variable name is `sex` or
   `gender`, transform `M`/`F` to `1`/`0` values; also, added the
   ability to specify fixed factors on `inputs`/`make-specs` commands (
   `file|group|F1|F2/LVL` , e.g. adds `F2` w/ fixed values `LVL` )

 - added `sig=root*` wildcard matching

 - added `head=N` to [`MATRIX`](ref/outputs.md#matrix) to show only
   the first _N_ epochs

 - added `modal` to MTM (only for MTM, WREL and MTM currently), which
   gives `CH` level output, for whole signals only (`WREL_PK_FREQ`,
   `WREL_PK_AMPL`, etc)

_Annotations_

 - for [`MAKE-ANNOTS`](ref/annotations.md#make-annots), added several
   new options: `w`/`w-left`/`w-right` added, to add flanking edges;
   `midpoint`/`start`/`stop` to set zero-duration time-points; and
   `pool`, that takes multiple annotations and combines but does not
   flatten, c.f. union/intersection modes

 - added `pos`/`neg` and `pos-pct`/`neg-pct` options to
   [`S2A`](ref/annotations.md#s2a)

 - added more label options (e.g. to include channel labelss) to
   [`S2A`](ref/annotations.md#s2a)

 - added `remap` to [`WRITE-ANNOTS`](ref/annotations.md#write-annots)
   to specify on-the-fly mappings that do not change the internal
   annotations (e.g. `pN1` --> `N1`)

 - fixed occasional issue with AM/PM encoding of times in annotation files

 - fixed a bug in `S2A` when spanning gaps in an EDF+D

_Generic_

 - added the `REPORT` command (with `cmd`, `fac` and `vars` options),
   to explicitly request outputs in text-table mode that might not
   otherwise be emitted

 - added `n/m` numeric slicing of sample lists, i.e. to facilitate
   batch submission of jobs to a cluster

 - added `force-prefix` to [`POPS`](ref/pops.md#pops), i.e. to always
   output `pN1`, `pN2`, etc, whether we have orig staging or not


## v1.2.0/1 (03-Jan-2025)

_New Luna "walk-through" didactic material_

 - a comprehensive step-by-step guide to using Luna as applied to
   real-life datasets; the web pages are available
   [https://zzz.bwh.harvard.edu/luna-walkthrough/](https://zzz.bwh.harvard.edu/luna-walkthrough/)
   and we'll post the accompanying walk-through data on
   [NSRR](sleepdata.org) within the coming month.

_General phenotype association (GPA) command_

 - major new [`GPA`](ref/assoc.md#gpa) command for efficient
   (permutation-based) linear models via `--gpa-prep` and `--gpa`
   commands

 - new binary format created by `--gpa-prep`, read by `--gpa`
 
 - allow kNN missing data imputation with `knn=K` option

 - added `desc`, `stats` and `dump` to give a description, output
   statistics (mean, SD, N) or dump raw values from a binary file;
   added `make-specs` to generate JSON specification files

 - added asymptotic significance values (nominal and adjusted,
   e.g. FDR)

_Other new commands_

 - [`META`](ref/annotations.md#meta) command to calculate/append
   annotation meta-data

 - [`DERIVE`](ref/evals.md#derive) command to generate
   summary statistics from annotation meta-data values

 - [`SCALE`](ref/manipulations.md#scale) to enforce normalized
   scale (min/max based, with `min-max`, `clip-min` and `clip-max`)

 - [`ESPAN`](ref/annotations.md#espan) command to give per-epoch
   statistics about annotation coverage

 - statistics to quantify [_ultradian dynamics_](ref/dynamics.md)
   from epoch-level metrics; typically invoked via `dynam` added to
   `PSD`, `COH`, `SPINDLES`, `SO`, `PSI`, `CORREL`,

 - [`COMBINE`](ref/manipulations.md#combine) command, to make a
   new channel that is the mean or median of others

 - [`--bind`](ref/helpers.md#-bind) command, to combine different
   EDFs into a single EDF (where the files have the same interval but
   have different signals, i.e.  the complement of
   [`--merge`](ref/helpers.md#-merge)

 - [`RUN-POPS`](ref/pops.md#run-pops) command (wrapper around
   `POPS`)
 
_Lunapi_

 - new [_Moonbeam_](lunapi.md#moonbeam) functionality in lunapi, allowing
   NSRR users to directly pull individual recordings from select cohorts
 
 - new [`empty_inst()`](lunapi/ref.md#projempty_inst) function to
   create an empty EDF within Lunapi (i.e. one can then attach other
   signals with `insert_signal()` to use Luna functionality on data
   not from EDF

 - documentation added for the [`scope()` viewer](lunapi/scope.md)

_Signal inputs_

 - new [`preload`](luna/args.md#preloading) special variable to load
   entire file, avoiding random access calls (which can be desirable
   when working on systems with slow `fseek()` I/O calls, e.g. network
   drives, etc.

 - extended the
   [`CANONICAL`](ref/canonical.md#canonical-signal-definitions)
   format, to allow _templates_ and `+=` variable appends

 - the special variable `id` can now take a comma-delimited list:
   `id=id1,id2`; also, `skip=id1,id2` will skip these IDs; (note:
   `include`/`exclude` provide similar functionality but take files of
   IDs, not the IDs themselves)

 - added `retain-case` to keep the case of a channel is it
   (case-insensitive) matches a primary; e.g. for alias `Fp1|EEG-Fp1`,
   if we have `FP1` as a query, Luna will now (as of v1.1) change this
   to `Fp1` (as it matches the primary).  If you wish to retain the
   case of the original, set `retain-case=T`.

 - added `order-signals=T` to make all inputs alphabetically ordered;
   this can help to align outputs, e.g. from `HEADERS signals` or for
   `CH1` x `CH2` pairwise outputs, i.e. so that one doesn't get a
   mixture of `A`x`B` in some outputs but `B`x`A` in others (as
   typically the order of channels is determined by position in the
   EDF rather than alphabetically)


_Annotations_

 - extended [`.annot` format](ref/annotations.md#annot-files) to allow
 meta-data in additional columns (7 onwards); this type of _tabular
 meta-data_ can be enforced with
 [`WRITE-ANNOTS`](ref/annotations.md#write-annots) by adding
 `tab-meta` (or `tab-meta=T`); also added `meta` option to control
 column 6 output

 - support different annotation date formats via `date-format` (for
   annotation files) and `edf-date-format` (for EDF headers, even
   though EDF headers are meant to be European DMY format).  e.g. can
   set `date-format=YMD` or `MDY` or `DMY` (default). `YMD` indicates `yyyy-mm-dd` format.

 - allow `d1`, `d2` format for annotation files, where this indicates
   the first, second, etc day based on the EDF start date:
   `d1-hh:mm:ss` (or `d1 hh:mm:ss`), `d2-hh:mm:ss`, etc.

  - added `annot-meta-default-num` (which accepts `T`/`F` values) to
     set the default type of annotation meta-data to be numeric rather
     than string (this is useful for the `DERIVE` command that will
     typically assume numeric meta-data)
   
 - added `num-atype`, `txt-atype`, `bool-atype` and `int-atype`
   special variables to set particular meta-data fields to a particular
   type

 - allow `;` as well as `|` delimiters for annotation meta-data in
  `.annot` files; these can be changed with `annot-meta-delim1` and
  `annot-meta-delim2` special variables (and can both be set to the
  same value if needed)

 - added `SEED_ANNOT`, `SEED_CH`, `OTHER_ANNOT` and `OTHER_CH` to
   `OVERLAP` output (`OTHER` x `SEED` strata)

 - `annot=` can now accept [_expanded values_](luna/args.md#sequence-expansion) 


_Hypnograms_

 - fixed error where `HYPNO`'s baseline `POST` variable (post-sleep recording time) was
 one epoch smaller than it should be

 - added `pre_sleep` and `post_sleep` annotations from `HYPNO annot`

 - known issue: for long (multi-day) recordings, the timing of some `HYPNO`
   variables is incorrect (e.g. `TX_` statistics); arguably, this is not
   a big issue as values such as sleep-midpoint etc, have no intrinsic
   meaning if the recording contains multiple sleep periods;  _in future,
   we'll aim to add a flag / not calculate various hypnogram metrics when
   there appears to be multiple sleep periods (either based on recording time
   or other heuristics) - although this could get messy)

 
_lunapi (Python interface)_

 - finalized `scope()` interactive viewer
 
 - added `moonbeam()` to `lunapi`


_Misc additions, changes and fixes_

 - added `default-starttime`, `no-default-starttime` and date equivalents
 
 - new `log=file.txt` special option to mirror log output to a file;
   also `--log` to output the command to std::cerr (which can be
   useful when tracking outputs)

 - added `key+=value key+=value2` --> `key=value1,value2` to allow repeated options, building up a comma-delimited list
    ```
    luna s.lst -s PSD sig+=C3  sig+=C4  sig+=F3,F4
    luna s.lst -s PSD sig=C3,C4,F3,F4
    ```

 - added `force-digital-minmax` (and `force-digital-min`/`force-digital-max`) special options
 
 - `SOAP` now iterates over channels and stratifies all outputs by `CH`

 - revised `SO` variable naming scheme (e.g. `SO_P2P` now `SO_AMP_P2P`, etc)

 - added `uppercase-keys` to [`CACHE record`](ref/)

 - added `scale` added to [`PSC`](ref/psc.md) (for scaled-PCA implementation)

 - `PSC` now takes `cmd-var=SPINDLES,DENS` etc to select only specific variables

 - added `q` option to [`S2A`](ref/annotations.md#s2a) e.g. `q=10`
 
 - fixed issue in decimal-place resolution of time tracks in EDF+D
   (i.e. now using same number of decimal places as `.annot` output,
   1e-4s by default

 - fixed bug in qall-by-allq `CORREL` where off-diagonal elements had
   flipped signs (i.e. treated as directed measures of
   connectivity)

 - added decimal-place outputs to HMS times in `SEGMENTS` 

 - known issue: `dynam` does not work with `-t` output (i.e. cannot
   register the col names for arbitrary commands yet...)

 - added `so-fast-trans` and `so-slow-trans` (w/ arg in Hz as trans
   freq) to return only FS or SS (SO fast/slow swtichers)

 - added `winsor` option to `SPINDLES`; also changed the implementation of Q-score filtering; new
   `q-frq={freq},{cycles},{freq2},{cycles2}` option; also `q-verbose` and `q-verbose-all` and `q-max`

 - added outputs from `show-coef` for `SPINDLES`
 

## v1.00 (10-Jul-2024)

This major release contains a number of new analysis features as
well as support for a Python interface to Luna,
[_lunapi_](lunapi/index.md).  It also contains a number fixes and
smaller additions.

_New interfaces_

 - a new [_lunapi_](lunapi/index.md) Python package provides bindings
 to the Luna library; it also contains a number of convenience
 functions and an interactive viewer for signal and annotation data

_New commands_
 
 - new [EDF-MINUS](ref/manipulations.md#edf-minus) command to
   collapse gapped EDF+D whilst aligning records, stage annotations
   and epochs

 - new [PCOUPL](ref/xxx.md) command for generic phase-event coupling

 - new [SVD](ref/ica.md#svd) commad for time-series PCA via SVD
 

_Annotations_

 - the `ANNOTS` command now takes `annot` classes to output only a
   subset (w/ wildcard character `*` allowed to subset, in form
   `root*` only)

 - date formats: two new options `read-mdy-annot-dates` and
   `read-mdy-edf-dates` allow for non-European (_month-day-year_)
   format dates, instead of the default _day-month-year_ (adopted as
   _European Data Format_ files specify this convention).  The first
   form relates to annotation files (`.annot); the second form relates
   to values in the EDF header (i.e. if they have been incorrectly
   specified).

 - `OVERLAP` now has option to output a shuffled set of annotations
   (given the full shuffling scheme - shuffle, event-based,
   constrained, background-filtered, etc): e.g. `add-shuffled-annots=A1,A2`
   adds `s_A1`, `s_A2` as new, shuffled versions of `A1` and `A2`; the default tag `s_`
   can also be altered with `add-shuffled-annots-tag=s_`


_Epoch definitions_

 - EPOCH now takes `fixed` and `trunc` ; output gives `FIXED_DUR` from
   `EPOCH`

 - generic epochs can now be shifted left/right (backwards/forwards)
   using `shift` (-ve/+ve values)

 - new `EPOCH` outputs: `TOT_DUR`, `TOT_PCT`, `TOT_REC`, `TOT_SPANNED`
   and `TOT_UNSPANNED`

 - added `SEGMENTS` largest annotation (`largest1`), (`largest2`) etc;
   the 1,2,3,... index is in the _instance_ ID of the annotation (class
   = `largest`); also, adding e.g.  `requires-min=120` enforces that
   largest segments must be at least 2 hours -- 1,2,3 is in INST ID


_Spectral analysis_

 - fixed `MTM` options; added `speckurt` for fbins (`MTM`)

 - `PSD` and `MTM` now have `band` (=`T`/`F`) to drop band output altogether

 - allow `skip-bands` (e.g. `skip-bands=SIGMA,GAMMA,TOTAL`) for `PSD` to omit certain bands from the output

 - to support ISO analyses: PSD now has an `add` option to emit
   (currently 1Hz) values (requries `EPOCH dur=4 inc=1` and `PSD
   segment-size=4`).  This creates a new 1 Hz time-series in the EDF.

 - `COH` now allows generic epochs - _as long as they are of fixed
    size_ (which requires adding `fixed` along with `annot` when applying
    the `EPOCH` command)


_Spindle/SO analyses_

 - now `so-annots` (instead of `annots`) is needed to emit slow
  oscillation annotations (as is called by SPINDLES)

 - `SPINDLES`/`SO` coupling issue fixed if a record is above epoch span


_Data manipulation_

 - `COPY` allows `new` to specify a full name (when used with a single
   channel); `pretag` adds the tag to the front (better for `[xx][1:8]`);
   also, `tag` no longer adds `"_"` in between, this must be explicitly
   specified, e.g. `tag=_TAG`


_Scripting: filters and sequence expansions_


 - all `sig` options allow `[inc]` and `[-exc]` matching:
   ```
   sig=C3,C4,F3,F4[F]
   ```
   implies
   ```
   sig=C3,C4
   ```
   i.e. must match a `C` character. This can be 
   useful when working with larger sets and you want to select a subset
   of `sig=${s}` where `${s}` may be a very large but structured list 

 - we now allow for more generic `[][]` expansion syntax,
   i.e. `[seq1][seq2]` where the terms or sequences are either in the
   form `n:m` (where `n` and `m` are integers, with `n` <= `m`) or as
   a commad-delimited list.
 
 - no nesting of terms if allowed, but they can be applied sequentially:
   ```
      ${s=[x][a,b,c]} & PSD [${s}][1:5]
   ```
   implies
   ```
   xa1,xa2,xa3,xa4,xa5,xb1,xb2,xb3,xb4,xb5,xc1,xc2,xc3,xc4,xc5
   ```
   Note: conditions are evaluated first, based on prior set variables;
   then line-by-line we evaluate a) swap in variables,
   wildcards,replace variables, b) expand sequences

 - fixed a bug that occurred when using `[n:n]` 



_Misc. fixes and changes_

 - added  `output-both` option to `CORREL`
 
 - fixed `TOT_DUR_HMS` when >24 hrs in `HEADERS`

 - `WRITE` now resets EDF start date if needed (i.e. when changing start time if collapsing to standard EDF)

 - fixed the `ids=<list>` option (to swap in alternate IDs on reading a sample list), as it was previosly not working

 - `POPS` `ignore-obs-staging` option is now fixed

 - `SIMUL` can zero-out frequencies above or below a certain value
   (i.e. after interpolation) with `zero=lwr,upr`

    

<!---

IN FLIGHT / TODO

_Predictive modelling_ 

 - adding [LightGBM]() library support and compilation flag
 - new [`ASSOC`] command
 - added new `MASSOC` set of functions
   - new command PREP-MASSOC 
 
- TODO - `PERI` command ; 
  - `RIPPLES`
  - `GED`
  - `NMF` 
  - `ORDER` command (or that command-line `sig` sets order)
  - loops in scripts [todo]
  - channel range selector [C3][C4] .. use same syntax as [C3][1..4] .. will have to wait and expand on a per-EDF basis ... 
   - __Global coherence statistics__ via the [`SYNC` command](ref/cc.md#sync)  
   - prototype [`TCLST`] command for time-series clustering
_LunaR_
 - `lunaR` is now in CRAN
 - documention (e.g. `?leval`) added to the lunaR package 
 - new `lload()`, `lhead()` and `lcols()` convenience functions 
 - additional `lheatmap()` options added

-->



## v0.99 (5-Dec-2023)

This release (leaping from v0.28 to v0.99 in a single bound) contains
a large number of incremental fixes and additions, as well as a few
more major features including: 1) automatically generated
hypnogram-based annotations, 2) reworked [`MASK`](ref/masks.md#mask)
interface to simplify masks based on multiple annotations, 3) allowing
generic (variably-sized) epochs based on annotations, 4) a new
framework to implement model-based prediction given PSG-derived
features, in the [`PREDICT`](../ref/predict.md) command, also suppored
in Moonlight, 5) epoch-wise analysis using the mutlitaper command
[`MTM`](ref/power-spectra.md#mtm), 6) ability to generate new
annotations conveniently on-the-fly based on pairwise contrasts of
existing ones, 7) the [Moonbeam](moonbeam.md) utility, that allows
NSRR data to be pulled directly into Moonlight (for NSRR users) and 8)
a prototype of a new utility for viewing hypnograms
([Hypnoscope](hypnoscope.md)).


_Hypnograms_

 - __major:__ the [`HYPNO`](ref/hypnograms.md#hypno) now has the `annot` option, which
   adds a series of hypnogram-based annotations, e.g. that can be used
   in subsequent mask operations, or output, etc.

 - [`HYPNO`](ref/hypnograms.md#hypno) epoch-level output now has
   `PRE`, `SPT`, and `POST` flags (0/1) to reflect epochs that are
   pre-sleep, during the sleep period time, or post-slep; also, the
   base-level output has two new variables, `PRE` and `POST` to give
   the recording time pre- and post-sleep (anchored on EDF/recording
   start/top times, rather than lights off, i.e. the
   difference between `SOL` and `PRE` is that `SOL` is defined
   relative to lights off.

 - if a recording is entirely _LightsOut_ (`L` stage
   annotation), then _Lights Off_ and _Lights On_ times will be now be
   correctly shown as `NA`
   
 - changed [`HYPNO`](ref/hypnograms.md#hypno) `FLANKING_MIN` to
   `FLANKING`, which is now an epoch count

 - fixed a minor issue when running with a gapped (EDF+D) recording:
   added new variables `TGT` (total gap time).  Gaps count as _missing_
   and so do not enter the denominator for metrics (i.e. not in
   `TRT` or `TIB`).  For cycle-level output, `NREMC_MINS` includes gaps,
   whereas `NREMC_N` does not include gaps.

 - epoch-level elapsed time outputs are now encoded from the _start_
   of that epoch, i.e. 0 to _N_-1 rather than 1 to _N_

_Prediction models_

 - __major:__  initial implementation of a new [`PREDICT`](ref/predict.md) command 

_Moonlight_

 - __major:__ added a new [_Moonbeam_](moonbeam.md) feature to directly pull [NSRR](sleepdata.org) data into Moonlight for
   NSRR users

 - __major:__ added a new _Models_ tab to support model-based prediction of
    metrics, initially an estimate of adult "brain-age" based on the
    NREM sleep EEG, implementing a model described
    [here](https://pubmed.ncbi.nlm.nih.gov/30448611/)

_Hypnoscope_

 - an initial implementation of the [Hypnoscope](hypnoscope.md)
   utility for viewing multiple hypnograms and generating
   hypnogram-based metrics

_Signal processing_

 - __major:__ the [`MTM`](ref/power-spectra.md#mtm) command now allows
   epochwise analyses (as well as segments within those epochs);
   `epoch-spectra`/`epoch-output` and
   `segment-spectra`/`segment-output` control levels of epochwise and
   segmentwise outputs; renamed `epoch-slopes` to `segment-slopes`

 - [`MTM`](ref/power-spectra.md#mtm) now precomputes tapers, which greatly speeds up epochwise
   analysis

 - added bandpower outputs for [`MTM`](ref/power-spectra.md#mtm)
   (`B`-stratified)

 - added `kurt`/`kurt3` options to `PSD`, to give epoch-level kurtosis
   (of power averaged over channels)

 - added `speckurt`/`speckurt3` options to `MTM`, to give
   _segment-level_ kurtsosis (of power averaged over channels,
   _within_ epochs if any epochs are specified).  Also, added the
   `alternate-speckurt` to support an alternate definition of
   these (only used for Sun et al model)

 - added `ratio`/`ratio1` options to both
   [`PSD`](ref/power-spectra.md#psd) and
   [`MTM`](ref/power-spectra.md#mtm) to give ratios of bandpowers for
   prespecifed pairs of bands; `ratio1` implies `a/(1+b)`.

 - both [`PSD`](ref/power-spectra.md#psd) and
   [`MTM`](ref/power-spectra.md#mtm) now allow setting user-defined
   power bands on the fly, e.g. `PSD sigma=12-16`.  (Previously, this
   could only be done as a special variable setting, which would be
   constant across all analyses.)
 
 - added a new `PSD_CV` output (w/ both `dB` and `sd` specified) for
   the [`PSD`](ref/power-spectra.md) command; this uses the
   calculation for coefficients of variation assuming log-normal data,
   i.e. `CV=sqrt(exp(s^2)-1)` where `s^2` is the natural log-scaled
   variance.

 - new more efficient cross-correlation [`XCORR`](ref/cc.md#XCORR) implementation and
   new outputs


_Artifact and alignment utilities_

 - new [`EDGER`](ref/artifacts.md#edger) command, designed to identify
   leading/trailing portions of recordings likely to be artifactual
 
 - new [`ALIGN-EPOCHS`](ref/alignment.md#align-epochs) command

 - new [`ALIGN-ANNOTS`](ref/alignment.md#align-annots) command

 - new [`INSERT`](ref/alignment.md#insert) command, with
   new FFT-based cross-correlation analysis to align signals from
   different EDFs and insert them, optionally adjusting timings


_Filters:_

 - [`FILTER`](ref/fir-filters.md#filter) now accepts separate transition
   band widths (`tw`) and ripples (`ripple`) 
   for the lower and upper edges of a Kaiser-window
   FIR filter, e.g. by `tw=0.5,5` and `ripple=0.01,0.01`,
   corresponding to lower and upper edges respectively; the new filter
   is generated by convolution of two separate highpass and lowpass
   filters with those parameters.

 - added `butterworth` and `chebyshev` IIR filters to the [`FILTER`](ref/fir-filters.md#filter)
   command
   
 - added `ngaus` option to [`FILTER`](ref/fir-filters.md#filter) to implement narrow-band filter
   via a frequency-domain Gaussian (parameters: frequency domain
   central mean and FWHM of the Gaussian, `ngaus=11,2`)
 
 - `fft` is now the default for [`FILTER`](ref/fir-filters.md#filter)


_Masks, epochs, freezes and caches_

 - __major:__ variable-sized ("generic") epochs are now supported by [`EPOCH`](ref/epochs.md#epoch), 
   defined based on existing annotations: e.g. `EPOCH annot=X`. In
   this context adding `else=Y` will generate a new annotation (`Y`)
   which flags _not_ being in an defined epoch (based on `X`) but
   respects gaps. Multiple annotations are allowed: `annot=A,B,C`.
   Labels are given to epochs in the output now.

 - for either conceptual or practical reasons, not all command
   currently support variable-sized annotations (e.g. `COH` or
   `HYPNO`) but this is flagged if attempting to run such a command in
   the presence of variable-sized (generic, annotation-based) epochs.
 
 - __major:__ reworked the primary [`MASK`](ref/masks.md#mask) command
   to allow multiple annotations to be specified with implied _OR_ /
   _AND_ logic: e.g. `MASK ifnot=N2,N3`.  The type of logic can be
   controlled by explicitly specifying `ifnot-any=N2,N3` or
   `ifnot-all=N2,arousal` to support _OR_ versus _AND_ logic
   respectively (the former is the same as the default `ifnot`).
   These are applied to all masks, e.g. `mask-if`, etc, as described
   [here](ref/masks.md#mask).

 - generic (annotation-based) masks can now use wildcards to complete
   annotation names: `MASK ifnot=artifact_*`

 - new `MASK` syntax `ifnot=+annot` means to match the epoch _only if
   it is completely spanned by that annotation_.  (One cannot use both
   `+` and `*` symbols together, however.)
 
 - added the option [`EPOCH table`](ref/epochs.md#epoch) which dumps
   ouutput (same as `verbose`) _but does not change the epoch
   structure_, i.e. it only gives output.  The extra argument `masked`
   will make the `table` output for both masked and unmasked epochs; the
   `verbose` option also gives more output than previously
 
 - the [`CACHE`](ref/freezes.md#cache) command has a new `record`
   option, which can be used to pull any arbitrary outputs into the
   cache (e.g. for use with a subsequent [`PREDICT`](ref/predict.md#predict) command)

 - the [`THAW`](ref/freezes.md#thaw) command has a new
   `preserve-cache` option so that the cache is not over-written;
   i.e. it can be allowed to build up between successive `FREEZE` /
   `THAW` operations

 - the [`RE`](res/epochs.md#restructure) command has a new
   `preserve-cache` option, which does not wipe the cache when
   restructuring a dataset

 - added a new [`C2A`](ref/freezes.md#c2a) cache to annotation command

_Annotations_

 - __major__: added a new [`MAKE-ANNOTS`](ref/annotations.md#make-annots)
   command, to generate new annotations based on pairwise comparisons
   of existing ones (union, intersection, if overlapped, if not
   overlapped, using the syntax `A|B`, `A*B`, `A+B` and `A-B`
   respectively); this also includes special behaviors if given the
   options `epoch`, `epoch-num`, `flatten` or `split`

 - the [`WRITE-ANNOTS`](ref/annotations.md#write-annots) now accepts
   `prefix` to match w/ all annots starting with that; useful for
   `prefix=h_`

 - added `raw-annot` (and `raw-annots`) to specify subset of annots to
   load - but unlike `annot` this does not apply _sanitization_ first
   (if that is the global default)

 - added `bins` and `bins-label` options to the
   [`S2A`](ref/annotations.md#s2a) command: given `bins=min,max,n` to
   make `n` bins of equal span, to make annots `B1`, `B2`, etc... `Bn`
   (or `bins-label` instead of `B`)


_Interval-based analyses:_

 - the [`OVERLAP`](ref/intervals.md#overlap) command now accepts `*`
   wildcards in seed/other/fixed specifications to match annots
   `seed=sp*,so*` i.e. matches `sp_15`, `sp_11` etc

 - the [`OVERLAP`](ref/intervals.md#overlap) command now has
   `contrasts`, `event-perm`, `offsets` options.  Now one should add
   `seed-seed=T` to get seed-seed pairs; adding seed-seed pileup is
   now turned off by default, unless `pileup=T` is added

- the [`OVERLAP`](ref/intervals.md#overlap) command now accepts
   `rp=<annot>|<tag>,...` to get `rp_tag=xxx` from meta data for that
   annotation, in which case it will set a 0-duration time-point at
   that position


_Spindle/slow oscillation detection:_

 - added `COUPL_ALL` statistics to `SPINDLES`, which performs
   spindle/SO analysis under both default (only using spindles that
   overlap a detected SO) and `all-spindles` (calculating
   `COUPL_ANGLE` and `COUPL_MAG` metrics based on all spindles); if
   the `all-spindles` option is explicitly set, then these two sets
   are identical, and only a single set is reported).

 - Slow oscillation output (`SO`/`SPINDLES` commands) is changed: now
   `SO_SLOPE` is output by default instead of `SLOPE_NEG2`.  The other
   slopes are only generated w/ the `verbose` option. Additionally, it
   now doesn't output `_neg` and `_pos` halfwaves as annotations, but
   instead adds `rp_mid` `rp_pos` and `rp_neg` as _relative positions_
   within the interval (scored 0..1); the
   [`OVERLAP`](ref/intervals.md#overlap) command is now able to use
   these `rp` (relative position) terms.  Finally, `SO` annotations do
   not output `dur` now, only `frq`.
 
 - [`SPINDLES`](ref/spindles-so.md#SPINDLES) annotations (from
   the `annot` argument) now add `rp_mid` (as a 0-1 proportion) to indicate
   the mid/peak of the spindle, i.e. rather than in the old time-point based form
   `mid=tp:123456789`

 - the `SPINDLES` command has a new `cache-peaks-sec` option
 
 - `SPINDLES` now gives a message if `fc` is out-of-range given `q`.
   In this case, added the option `noq` (i.e. equivalent to setting
   `q=-999`)
     
 - fixed the `SPINDLES` option `stratify-by-phase` to avoid
   double-counting



_Minor additions and fixes_

 - the [`SEGMENTS`](ref/outputs.md#segments) command now outputs the
   largest segment size (for an EDF+D w/ gaps); also, option
   `largest=L1` adds an annotation spannong the largest segment
   called `L1`; in the case of ties, only the last will be added

- fixed bug in `EDF` - when forcing a standard EDF structure, this
   now updates the internal timetrack correctly, so that any
   subsequent call to `timeline.wholetrace()` uses the correct
   time-points
 
 - added `min` and `max` options to `MINMAX`, to clip EDF physical
   min/max values; can specify both or either; other wise `MINMAX`
   sets all `sig` channels to the be same PMIN/PMAX and DMIN/DMAX.  If
   `force` is specified with `min` or `max` then that value is always
   set; otherwise, the physical min/max values are only changed if
   they are smaller, i.e. clipping, not expanding the min/max range
 
 - channel locations are now populated by default if not otherwise
   specified (for a standard 64-channel 10/10 EEG only)

 - automatically sets any EDF header reserved field characters 5-44 to
   null (space) (also the first 5 chars if not `EDF+C` or `EDF+D`)

 - new `srand=XXXX` special argument to set the RNG seed
  
 - added `REC_DUR_SEC` and `REC_DUR_HMS` to `HEADERS` which used to be
  `TOT_*`.  Now, `TOT_DUR_SEC` and `TOT_DUR_HMS` reflect the full
  duration, including any gaps (i.e. if an EDF+D)

 - the [`RESAMPLE`](ref/manipulations.md#resample) command has a new
   `downsample` argument: only channels with rates above the `sr`
   value will be altered
 
 - Luna can now correctly take `.edfz` and `.edf.gz` files on the
   command line (versus from sample list)

 - fixed issue with the [`CPT`](ref/association.md#cpt) command when
    multiple classes of DV are specified; also added a `all-dvars` (or
   `dv=*`) option to select all named DVs from the DV files; also, can
   cluster based on time (`T`) as well as `F`, `CH`, `CH1` and `CH2`;
   because of this, `STAT` output variable is now the t-statistic, and
   `T` means any time-level stratifier

 - the [`STATS`](ref/summaries.md#stats) command has renamed
   `MEDIAN.X` to `X_MD`; also added `X_MN`

 - [`STATS`](ref/summaries.md#stats) has a `kurt3` to specify
 unadjusted kurtosis values (i.e.  veresus _excess_ values), such that
 _X~N(0,1)_ has an expected value of 3.0, not 0;

 - [`PEAKS`](ref/intervals.md#peaks) now output annotatons (`annot`) as well as cache; can
   include `w` to add window (sec) around each point


_Interal changes/library upgrades_

 - upgraded to [Eigen](https://eigen.tuxfamily.org/) library to 3.4.0

 - upgraded to [sqlite](https://www.sqlite.org/) library v3.41.2

 - reorganized some of the codebase, in particular, moving most
   TF-analyses into single `spectral/` folder


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

 - fixed issue with `ECG-SUPPRESS` when high sample rate used (insufficient smoothing -> no R-peaks detected)
 
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
 
 - added `first` (mins) and `anchor` (`T0`, `T1` or `T2`)
   options, to report hypnogram stats for only the first _N_ minutes
   (setting the rest to `L`);   also, added `last` (which takes `T4` (default)
   or `T5` or `T6` as _anchor_ values);  also, `clock` which, along with `first`
   specifies an arbitrary anchor, based on the clock time;  this may come before
   the EDF start;   the variable `SHORT` will be flagged if the requested window
   does not fully overlap the available staging

 - added elapsed stage duration times, i.e. seconds after anchor that
   we see N minutes of N2 sleep, etc.   If indiv does not have N mins
   of N2 sleep, a flag will be set;   SS x DUR..  T and SHORT.   Note that
   E_* epoch level output gives the elapsed duration at the START at that epoch;
   in contrast, this is indexed at the end... 
   

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
   `inst` to `.` )

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

 - added ability to specify an empty EDF (`--nr`, `--rs` and filename equals `.` ) 

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

 - added `import=file.txt` command to [`CACHE`]( to read from destrat
   output; can take `factors` and `v` param (as well as required
   `cache=`)

 - added `MASK epoch=all` to set a MASK but have it all empty ;
   i.e. to trim records not in an epoch
 
 - (for `.annot` only) added `align` option : given list of
   annots (or *) for all, align-annots-on=N1,N2, etc...  if not
   specified, find first instance of this annot, then align with 1
   second boundary (or `align-annots-res=X` if given) align /all/
   annots with this offset (bound at 0); i.e all records beforehand will
   be skipped if subsequent `MASK`/`WRITE` commands are applied 

 - added `pick` option to `SIGNALS` to pick first of pick=a,b,c that
   is present, and drop the rest; can map with `rename` to rename the
   pick
 
 - allow sample-lists to have a comma-delim list of annotations, or
   '.' to denote no data; in this way, we can have a fixed width
   sample list (i.e. three tab-delimited columns), making it easier to
   parse (case in point: _lunaR_ `lsl()` was broken if fewer than 3
   columns were found after first five rows, reflecting how R
   `read.table()` works)
 
  - `CANONICAL` does not now need an explicit GROUP to be specified;
   the file must still have a first col, it is just ignored now; also,
   new `drop-originals` option to drop all original (non-CANONICAL)
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
  stage; defaults to 4 (2 mins). Added `CONF` and `OTHR` outputs.

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

