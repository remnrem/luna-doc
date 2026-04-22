# Experimental / Unsupported / Legacy functions

!!! Danger "Warning"
    This section should be considered as __nothing more than our
    own internal notes and reminders on some features that are being
    prototyped or are under very heavy development__, etc.  That is,
    these are a) highly likely to change, b) will not necessarily use
    Luna's standard [output mechanisms](outputs.md), c) may be
    removed in future versions, and d) may be buggy, incomplete or
    both.  Stay well clear! You've been warned!  Whereas some commands listed here will
    (hopefully!) be elevated to be part of the main release, and more fully documented,
    others may remain here or be removed in future releases.

|Command |Description |	       
|---|---|
| [`ASYMM`](#asymm) | Lateralization indices |
| [`A2C`](#a2c) | Annotation to cache entry |
| [`L1OUT`](#l1out)    | Leave-one-out interpolation-based signal check |
| [`FIP`](#fip)    | Frequency-interval plots | 
| [`ALIGN-EPOCHS`](#align-epochs) | Align epochs between files |
| [`ALIGN-ANNOTS`](#align-annots) | Realign annotations given an `ALIGN-EPOCHS` solution |


## ASYMM

_Evaluates (regional) signal asymmetries in specified metrics_

`ASYMM` computes hemispheric lateralization indices from epoch-level power spectral
density data, comparing left and right channel pairs across sleep stages and
NREM–REM cycles. It reads cached PSD data (produced by a prior [`PSD`](power-spectra.md#psd) command run
with `cache`) rather than operating directly on raw signals.

For each left/right channel pair and each frequency bin or band, `ASYMM` computes
a log₂(L/R) asymmetry index per epoch. It then groups epochs by sleep stage and
by NREM–REM cycle, tests whether REM-stage asymmetry differs from flanking NREM
asymmetry (via t-test and optionally a permutation test), and optionally reports
asymmetry time-courses around NREM–REM transitions.

Current implementation notes:

- Requires a hypnogram to be present.
- Outlier epochs (log₂(L/R) beyond ±2.0, or with values below 1e-8) are excluded.
- Up to six NREM–REM cycles are identified; cycles without sufficient flanking NREM
  epochs are skipped.
- Transition analysis requires clean, stable epochs on both sides of the boundary.

<h3>Methods</h3>

`ASYMM` reads epoch-level power spectral density values from a named cache populated by a prior `PSD cache` run, then constructs a hypnogram and identifies up to six NREM–REM cycles using Luna's standard cycle-detection algorithm. For each specified left/right channel pair and each frequency bin or band, the log₂(L/R) lateralization index is computed per epoch; epochs in which either side falls below 1×10⁻⁸ or in which the ratio falls outside ±2.0 are flagged as outliers and excluded. Within each cycle, flanking NREM epochs are drawn from up to 50 epochs before and after the REM period, and cycles without at least 30 combined flanking NREM epochs are discarded. Per-cycle outliers in the log-scaled power values are further removed at ±6 SD. REM epochs are then expressed as z-scores relative to the combined flanking NREM distribution and compared with Welch's unequal-variance t-test; cycles whose leading and trailing NREM periods themselves differ significantly are excluded from the cross-cycle summary. Summary statistics (mean z-score, absolute z-score, signed and absolute −log₁₀(p)) are averaged across included cycles. Optionally, time-courses of |log₂(L/R)| around NREM→REM and REM→NREM boundaries are constructed over a user-specified epoch window, with empirical null distributions obtained by randomly shifting the window within the sleep period for the requested number of permutation replicates.

<h3>Parameters</h3>

| Parameter | Example | Description |
|---|---|---|
| `cache` | `cache=pc` | Name of the cache holding epoch-level PSD values (required) |
| `cache-var` | `cache-var=PSD` | Variable name within the cache to extract (default [`PSD`](power-spectra.md#psd); use `PER` or `APER` for IRASA output) |
| `left` | `left=C3,F3` | Left-hemisphere channel(s); use `+` to sum (e.g. `C3+F3+P3`) |
| `right` | `right=C4,F4` | Right-hemisphere channel(s); must match `left` in count |
| `epoch` | | Also emit epoch-level asymmetry values |
| `trans` | `trans=10` | Window half-width in epochs around NREM–REM transitions (default 10) |
| `nreps` | `nreps=500` | Permutation replicates for empirical transition statistics (default 500 when `trans` is set) |

<h3>Output</h3>

Individual-level summary (strata: `CHS` × `B` or `F`):

| Variable | Description |
|---|---|
| `LR_SLEEP` | Mean log₂(L/R) across all sleep epochs |
| `LR_REM` | Mean log₂(L/R) in REM |
| `LR_NREM` | Mean log₂(L/R) in NREM |
| `LR_WAKE` | Mean log₂(L/R) in wake |
| `NC` | Number of cycles analysed |
| `Z_REM` | Cycle-averaged z-score for REM asymmetry vs flanking NREM |
| `ABS_Z_REM` | Cycle-averaged absolute z-score |
| `LOGP` | Cycle-averaged signed −log₁₀(p) |
| `ABS_LOGP` | Cycle-averaged absolute −log₁₀(p) |
| `TR_R2NR_N` | Number of REM→NREM transitions identified |
| `TR_NR2R_N` | Number of NREM→REM transitions identified |

Per-cycle output (strata: `CHS` × `B`/`F` × `CYCLE`):

| Variable | Description |
|---|---|
| `LR_REM` | Mean log₂(L/R) in REM for this cycle |
| `LR_LEADING_NREM` | Mean log₂(L/R) in NREM preceding REM |
| `LR_TRAILING_NREM` | Mean log₂(L/R) in NREM following REM |
| `N_REM` | REM epoch count |
| `N_NREM` | Flanking NREM epoch count |
| `Z_REM` | Z-score: REM vs flanking NREM |
| `P` | p-value for REM vs flanking NREM difference |
| `LOGP` | Signed −log₁₀(p) |
| `INC` | Inclusion flag |

Per-transition output (strata: `CHS` × `B`/`F` × `TR`):

| Variable | Description |
|---|---|
| `R2NR` | Mean asymmetry at REM→NREM transition epochs |
| `NR2R` | Mean asymmetry at NREM→REM transition epochs |
| `R2NR_Z` | Z-score for R→NR transition (if `nreps` > 0) |
| `NR2R_Z` | Z-score for NR→R transition (if `nreps` > 0) |

Per-epoch output (option `epoch`, strata: `CHS` × `B`/`F` × `E`):

| Variable | Description |
|---|---|
| `L` | Left channel power |
| `R` | Right channel power |
| `LR` | log₂(L/R) asymmetry index |
| `SS` | Sleep stage |
| `C` | Cycle assignment |
| `OUT` | Outlier flag (1 = excluded) |
| `INC` | Included in cycle analysis |


## A2C

_Writes an annotation to a set of cache entries_

`A2C` (annotation-to-cache) converts annotation events into cached integer
sample-point indices. It is primarily used to prepare time-locked event lists for
downstream commands such as [`TLOCK`](intervals.md#tlock). For each annotation event, `A2C` reads a
metadata field containing a time-point value, locates the nearest sample in a
reference signal, and stores the resulting sample index in a named cache.

Current implementation notes:

- Either `sig` or `sr` must be given to establish the sample rate and time-point vector.
- Annotation events whose metadata time-point falls outside the `diff` tolerance are silently skipped.
- By default, each annotation class is stored in its own cache; use `cache` to
  combine all annotations into a single cache.
- Channel stratification is preserved unless `ignore-channel` is set.

<h3>Methods</h3>

For each specified annotation class, `A2C` iterates over all annotation instances and reads a designated metadata field whose value is expected to encode a sample-point time in the form `tp:XXXXXXX` (an integer in Luna's internal time-point units). The time-point vector of a reference signal is obtained and a linear scan finds the pair of adjacent sample points bracketing each annotation time-point; the closer of the two is selected, and the event is accepted only if its distance from the nearest sample does not exceed the `diff` tolerance (defaulting to one sample period). Accepted events are stored as integer sample-point indices in an internal cache keyed by channel and annotation class, with optional merging of all annotation classes into a single named cache. The resulting cache entries can be consumed directly by downstream commands such as [`TLOCK`](intervals.md#tlock).

<h3>Parameters</h3>

| Parameter | Example | Description |
|---|---|---|
| `annot` | `annot=spindles` | Annotation class name(s) to convert (required) |
| `meta` | `meta=tp_sec` | Metadata field within the annotation containing the time value (required) |
| `sig` | `sig=C3` | Reference signal; provides sample rate and time-point vector |
| `sr` | `sr=256` | Alternative to `sig`: specify sample rate directly |
| `cache` | `cache=sp` | If set, store all annotations in a single cache of this name; otherwise each annotation class gets its own cache |
| `ignore-channel` | | Ignore the `CH` stratifier from annotations; map all events to channel `.` |
| `diff` | `diff=0.004` | Maximum tolerated time difference (seconds) to accept a match (default: 1 / sample rate) |

<h3>Output</h3>

`A2C` writes to internal caches rather than to the output database. No output
variables are emitted directly; the resulting caches are consumed by subsequent
commands.


## L1OUT

_Leave-one-out interpolation-based signal check_

`L1OUT` assesses the spatial consistency of EEG channels by leave-one-out
cross-validation using surface spline interpolation. For each channel in turn, it
withholds that channel and reconstructs it from the remaining channels using the
same spherical spline interpolation used by [`INTERPOLATE`](spatial.md). It then
computes the Pearson correlation between the original and reconstructed signals for
each epoch, yielding a per-channel, per-epoch measure of how well each electrode
can be predicted from its neighbours.

Current implementation notes:

- Requires channel location information (CLOCS); default locations are used if none are attached.
- All channels in `sig` must share the same sample rate.
- Data must be epoched before running `L1OUT`.

<h3>Methods</h3>

`L1OUT` performs a leave-one-out cross-validation of spatial EEG consistency using spherical spline interpolation. Before iterating over epochs, the command pre-computes for each channel in turn the pair of interpolation matrices (the inverse of the between-good-channel spline matrix and the good-to-bad cross-channel spline matrix) derived from the full inter-electrode distance matrix — the same matrices used by the [`INTERPOLATE`](spatial.md#interpolate) command. For each epoch and each held-out channel, the pre-computed matrices are applied to the epoch's data from the remaining channels to produce a reconstructed time-series; the Pearson correlation between the reconstructed and the original observed time-series is then computed and reported. Channels with consistently low or negative leave-one-out correlations across epochs indicate electrodes that are spatially inconsistent with their neighbours.

<h3>Parameters</h3>

| Parameter | Example | Description |
|---|---|---|
| `sig` | `sig=C3,C4,F3,F4` | Channels to validate (required) |

<h3>Output</h3>

Per-channel, per-epoch output (strata: `CH` × `E`):

| Variable | Description |
|---|---|
| `R` | Pearson correlation between original and leave-one-out interpolated signal for this channel and epoch |

## FIP

_Frequency-interval plots_

`FIP` produces a two-dimensional frequency-by-time-interval representation of
signal activity. It applies a continuous wavelet transform (CWT) across a range of
frequencies, identifies contiguous above-threshold intervals in the resulting
amplitude time-series, and bins those intervals by duration. The output summarises,
for each frequency and duration bin, how much activity falls in intervals of that
length — useful for characterising the temporal structure of oscillatory bursts
(e.g. spindles, slow oscillations).

As an alternative to CWT, `FIP` can operate on the signal envelope directly.

Current implementation notes:

- Intervals shorter than 2 × `t-upr` samples are ignored.
- When `th=0` (default), all samples are treated as above threshold.
- `ZIP` normalises across frequencies within each time bin; `FIP` normalises within each frequency by total duration.

<h3>Methods</h3>

`FIP` characterises the temporal structure of oscillatory activity by decomposing a signal's amplitude envelope into a two-dimensional frequency-by-duration representation. For each centre frequency in the specified grid (linearly or logarithmically spaced), a Morlet continuous wavelet transform (CWT) is applied to the entire signal; the resulting instantaneous amplitude time-series may optionally be log-transformed or min–max normalised per frequency. Samples falling below a threshold (if `th` > 0, defined as `th` times the mean amplitude) are excluded. The remaining amplitude series is decomposed into a set of nested intervals: starting from each above-threshold sample, the algorithm scans forward until a lower value is encountered, recording the interval as a (start, stop, height) tuple; intervals are processed in descending order of duration so that each sample's amplitude contribution is attributed only once — to the longest interval spanning it — with shorter nested intervals credited for the incremental height they add above the already-attributed baseline. Each interval is assigned to the time-bin corresponding to its duration in seconds (or oscillation cycles if `by-cycles` is set), and the summed amplitude within each frequency-by-bin cell is divided by total signal duration to yield the `FIP` statistic; normalisation across frequencies within each time-bin yields `ZIP`.

<h3>Parameters</h3>

| Parameter | Example | Description |
|---|---|---|
| `sig` | `sig=C3` | Signal channel(s) to analyse (required) |
| `f-lwr` | `f-lwr=1` | Lowest CWT frequency in Hz (default 1) |
| `f-upr` | `f-upr=20` | Highest CWT frequency in Hz (default 20) |
| `f-inc` | `f-inc=1` | Frequency step in Hz for linear spacing (default 1) |
| `f-log` | `f-log=20` | If set, use log-spaced frequencies with this many steps |
| `cycles` | `cycles=7` | Number of Morlet wavelet cycles for CWT (default 7) |
| `t-lwr` | `t-lwr=0.1` | Lower time-bin bound in seconds (default 0.1) |
| `t-upr` | `t-upr=4` | Upper time-bin bound in seconds (default 4) |
| `t-inc` | `t-inc=0.1` | Time-bin step in seconds (default 0.1) |
| `by-cycles` | | Scale time axis by oscillation cycles rather than seconds |
| `th` | `th=1.5` | Threshold multiplier: only include samples above `th` × mean (default 0, i.e. all samples) |
| `norm` | | Normalise amplitude values to 0–1 range per frequency before binning |
| `log` | | Log-transform amplitude values before binning |
| `envelope` | | Use signal envelope (min, max, raw) instead of CWT |

<h3>Output</h3>

Per-channel, per-frequency, per-time-bin output (strata: `CH` × `F` × `TBIN`):

| Variable | Description |
|---|---|
| `FIP` | Summed amplitude within this frequency and duration bin, divided by total duration in seconds |
| `ZIP` | As `FIP` but normalised so values sum to 1.0 across all frequencies for this time bin |

In `envelope` mode, `F` takes special values: `0` = original signal, `-1` = minimum envelope, `+1` = maximum envelope.


## ALIGN-EPOCHS

_Align epochs in a secondary EDF to the currently attached EDF_

`ALIGN-EPOCHS` is implemented and callable, but it is still experimental and is
not exposed through the standard `cmddefs` help system. The current
implementation loads a second EDF, extracts the same named channels from both
files, band-pass filters them to 1 to 20 Hz, robustly rescales each epoch, and
then searches for the best matching epoch in the primary EDF for each epoch in
the secondary EDF.

This is intended for cases where one EDF is a reduced, shifted, or partially
overlapping version of another and you need an epoch-to-epoch mapping before
copying stages or annotations. The command is conservative: a match is accepted
only when the best match is sufficiently better than the overall distribution of
candidate distances and sufficiently better than the second-best match.

Current implementation notes:

- It requires the same channel labels to exist in both EDFs.
- It requires effectively identical sample rates for those channels across files.
- It writes a mapping solution as standard Luna output, but does not itself alter
  the current EDF.
- There is code for ambiguity resolution, but it is currently disabled in the
  implementation.

<h3>Parameters</h3>

| Parameter | Example | Description |
|---|---|---|
| `edf` | `edf=subset.edf` | Secondary EDF to align to the currently attached EDF |
| `sig` | `sig=C3,C4,O1,O2` | Channels to use for alignment; the same labels must exist in both EDFs |
| `th` | `th=2` | Require the best match to be at least this many SD units better than the mean candidate distance (default `2.0`) |
| `th2` | `th2=0.2` | Require the best match to be at least this many SD units better than the second-best match (default `0.2`) |
| `ordered` | `ordered=T` | If `T`, halt if the inferred mapping violates monotonic epoch order |
| `verbose` | `verbose=101` | Dump the standardized matrices for one secondary epoch (1-based display epoch index) |
| `verbose2` | `verbose2=250` | With `verbose`, force comparison against a specific primary epoch for debugging |

<h3>Output</h3>

Primary output strata: `E2`

Individual-level variables:

| Variable | Description |
|---|---|
| `ORDERED` | `1` if aligned epochs are monotonic in the primary EDF, else `0` |
| `N_ALIGNED` | Number of secondary epochs that were matched confidently |
| `N_FAILED` | Number of secondary epochs that were not matched confidently |
| `N_FAILED_TH1` | Number of failed epochs attributable to the first threshold (`th`) |
| `N_FAILED_TH2` | Number of failed epochs attributable to the second threshold (`th2`) |

Per-`E2` variables:

| Variable | Description |
|---|---|
| `E1` | Matched epoch in the primary EDF, if a confident match was found |
| `ORDERED` | Per-epoch ordering flag |
| `NEXT_E1` | Second-best candidate epoch in the primary EDF |
| `D` | Standardized score of the best match |
| `NEXT` | Margin between the best and second-best standardized scores |

<h3>Example</h3>

```
ALIGN-EPOCHS edf=reduced.edf sig=C3,C4,O1,O2 th=2 th2=0.2
```

The resulting `E2 -> E1` mapping can then be passed to [`ALIGN-ANNOTS`](#align-annots).


## ALIGN-ANNOTS

_Rebuild an annotation file in a new epoch space using an `ALIGN-EPOCHS` solution_

`ALIGN-ANNOTS` is also implemented and callable, but remains experimental. It
reads an `ALIGN-EPOCHS` solution file, interprets that as a mapping from epochs
in the currently attached EDF to epochs in a reduced or realigned EDF, and then
rewrites all annotation intervals accordingly.

The intended workflow is:

1. Attach the original EDF that contains the annotations.
2. Run `ALIGN-EPOCHS` against the reduced or shifted EDF and save the output.
3. Run `ALIGN-ANNOTS` on the original EDF using that solution file.

Current implementation notes:

- It expects the `ALIGN-EPOCHS` solution file to have the specific seven-column
  header `ID E2 D E1 NEXT NEXT_E1 ORDERED`.
- It assumes simple sequential epoch numbering and the default Luna epoch length.
- Annotation instances are skipped if either endpoint maps to a missing epoch or
  if the mapped span is inconsistent.
- It copies annotation labels, instance IDs, and channel strings, but currently
  drops annotation meta-data.

<h3>Parameters</h3>

| Parameter | Example | Description |
|---|---|---|
| `sol` | `sol=align.txt` | Tabular `ALIGN-EPOCHS` solution file |
| `out` | `out=realigned.annot` | Output annotation file to write |

<h3>Output</h3>

Primary output strata: `ANNOT`

Individual-level variables:

| Variable | Description |
|---|---|
| `ALIGNED` | Number of annotation instances successfully remapped |
| `SKIPPED` | Number of annotation instances skipped during remapping |

Per-`ANNOT` variables:

| Variable | Description |
|---|---|
| `ALIGNED` | Number of aligned instances in this annotation class |
| `SKIPPED` | Number of skipped instances in this annotation class |

<h3>Example</h3>

```
ALIGN-ANNOTS sol=align.txt out=realigned.annot
```



## INSERT

_This command has moved to the Manipulations reference page._

See [`INSERT`](manipulations.md#insert).
