# Physiological Signal Analysis

_Commands for cardiovascular, muscle-tone, arousal, oximetry, and respiratory analyses_

These commands analyse non-EEG physiological channels commonly recorded in PSG. [`HRV`](physio.md#hrv) detects R-peaks in an ECG channel and computes a standard set of time-domain and frequency-domain heart-rate variability metrics, optionally by epoch or by annotation class. [`RAI`](physio.md#rai) quantifies muscle tone suppression during REM sleep by computing the REM Atonia Index from a chin EMG channel. [`AROUSALS`](physio.md#arousals) detects candidate sleep arousals from EEG spectral power and optional EMG signals, without requiring pre-scored annotations. [`LM`](physio.md#lm) detects leg movements and periodic leg movements from anterior-tibialis EMG following the WASM 2016 event grammar. [`DESAT`](physio.md#desat) detects oxygen desaturation events from SpO2 signals, [`RESP-LINK`](physio.md#resp-link) links respiratory events to likely desaturation and arousal responses, and [`RESPBREATH`](physio.md#respbreath) segments respiratory signals into breath-level intervals for downstream timing and phase-locking analyses.

| Command | Description |
| ---- | ------ |
| [`HRV`](#hrv) | Estimate heart-rate variability metrics from ECG |
| [`RAI`](#rai) | Calculate the REM atonia index from chin EMG |
| [`AROUSALS`](#arousals) | Detect candidate sleep arousals from EEG and optional EMG |
| [`LM`](#lm) | Detect leg movements and periodic leg movements (WASM 2016) |
| [`COMBINE-EMG`](#combine-emg) | Build a single continuous EMG channel from 2+ candidate channels |
| [`DESAT`](#desat) | Detect oxygen desaturation events from SpO2 |
| [`RESP-LINK`](#resp-link) | Link respiratory events to desaturation and arousal responses |
| [`RESPBREATH`](#respbreath) | Segment respiratory channels into breath events |

## [`HRV`](physio.md#hrv)

_Estimate heart-rate variability metrics from ECG_

!!! warning "Active development"
    [`HRV`](physio.md#hrv) is under active development and expansion. Defaults, derived metrics,
    and output details may still evolve.

[`HRV`](physio.md#hrv) detects ECG R peaks, derives RR intervals, optionally annotates those peaks and intervals, and then reports time-domain and frequency-domain HRV metrics. It can be run on a whole recording, by epoch, and optionally within specified annotation classes.

<h3>Methods</h3>

R peaks are detected over the full retained trace by band-pass filtering the ECG signal between `rp-flwr` and `rp-fupr` Hz using a Kaiser-window FIR filter, differentiating and squaring the bandpass output, applying a moving-window integration and median filter, and then identifying candidate QRS regions using a smoothed-z peak-detection algorithm. Within each region, the sample with the largest absolute bandpass amplitude is taken as the QRS reference point, and the local extremum within a tight evaluation window around that reference is taken as the R-peak location. ECG polarity is estimated automatically from the dominance ratio of positive versus negative peak amplitudes and can be forced with `rp-invert`. RR intervals are derived from consecutive R-peak timestamps and converted to milliseconds. Intervals outside the bounds [`lwr`, `upr`] seconds are excluded when computing the mean valid RR, then replaced with that mean for frequency-domain interpolation; the `IMPUTED` variable reports the proportion of such replacements. Time-domain metrics (SDNN, RMSSD, pNN50) are computed exclusively from non-imputed intervals. Frequency-domain HRV is computed by fitting a cubic spline to the full (including imputed) RR time series to generate a uniformly sampled 4 Hz tachogram, then applying Welch's method with a Tukey window and 50% overlap to estimate the power spectral density; LF power (0.04–0.15 Hz) and HF power (0.15–0.4 Hz) are integrated from the binned spectrum. When epochs are defined, R-peak detection is performed once over the whole trace and peaks are subsequently partitioned into epochs.

<h3>Primary parameters</h3>

| Parameter | Example | Description |
| ---- | ---- | ---- |
| `sig` | `ECG` | Required ECG channel or channels |
| `epoch` |  | If epochs are already defined, emit epoch-level HRV output |
| `annot` | `REM` | Also summarize HRV within these annotation classes |
| `by-instance` |  | With `annot`, stratify separately by annotation instance ID |
| `time-domain` | `T` | Enable time-domain HRV metrics; default `T` |
| `freq-domain` | `T` | Enable frequency-domain HRV metrics; default `T` |
| `lwr` | `0.3` | Lower valid RR interval bound in seconds; default `0.3` |
| `upr` | `2` | Upper valid RR interval bound in seconds; default `2` |
| `ns` | `512` | Welch segment length for frequency-domain HRV; default `512` |

<h3>Secondary parameters</h3>

| Parameter | Example | Description |
| ---- | ---- | ---- |
| `add-annot` | `Rpk` | Add generic R-peak annotations |
| `add-annot-rr` | `RRint` | Add generic RR-interval annotations |
| `add-annot-ch` | `Rpk` | Add channel-specific R-peak annotations |
| `add-annot-rr-ch` | `RRint` | Add channel-specific RR-interval annotations |
| `rp-lag` | `0.2` | R-peak detector lag window in seconds |
| `rp-infl` | `0.01` | R-peak detector influence parameter |
| `rp-th` | `3.5` | Primary threshold for R-peak detection |
| `rp-th2` | `1.5` | Secondary threshold for R-peak detection |
| `rp-max` | `2` | Maximum allowed R-peak width |
| `rp-dur` | `0.02` | Minimum R-peak duration in seconds |
| `rp-dur2` | `0.04` | Secondary minimum R-peak duration in seconds |
| `rp-ripple` | `0.02` | Kaiser ripple parameter for ECG band-pass filtering |
| `rp-tw` | `1` | Transition width for ECG band-pass filtering |
| `rp-flwr` | `5` | Lower ECG band-pass edge for R-peak detection |
| `rp-fupr` | `25` | Upper ECG band-pass edge for R-peak detection |
| `rp-w` | `0.02` | Median-filter window for the R-peak detector |

<h3>Outputs</h3>

[`HRV`](physio.md#hrv) can emit up to four tables:

| Table | Description |
| ---- | ---- |
| `CH` | Whole-trace or epoch-summarized HRV metrics |
| `CH,E` | Epoch-level HRV metrics with `epoch` |
| `CH,ANNOT` | Annotation-stratified HRV metrics with `annot` |
| `CH,ANNOT,INST` | Annotation-instance-stratified HRV metrics with `annot` and `by-instance` |

The main output variables are:

| Variable | Description |
| ---- | ---- |
| `IMPUTED` | Proportion of RR intervals that were out-of-range and replaced by the mean |
| `P_INV` | Estimated probability that the ECG is inverted |
| `INV` | 0/1 inversion flag |
| `NP` | Number of retained R peaks / RR intervals |
| `NP_TOT` | Total number of R peaks before exclusions or summarization |
| `RR` | Mean RR interval (ms), computed from valid non-imputed intervals only |
| `HR` | Mean heart rate (BPM) |
| `SDNN` | Standard deviation of non-imputed NN intervals |
| `SDNN_R` | Robust (outlier-resistant) standard deviation of non-imputed NN intervals |
| `RMSSD` | Root mean square of successive differences between non-imputed NN intervals |
| `RMSSD_R` | Robust RMSSD |
| `pNN50` | Proportion of non-imputed successive NN differences exceeding 50 ms |
| `LF` | Low-frequency HRV power |
| `HF` | High-frequency HRV power |
| `LF_N` | Normalized low-frequency power |
| `HF_N` | Normalized high-frequency power |
| `LF_PK` | Peak low-frequency HRV frequency |
| `HF_PK` | Peak high-frequency HRV frequency |
| `LF2HF` | Low-to-high frequency power ratio |

<h3>Notes</h3>

- [`HRV`](physio.md#hrv) uses Luna's internal ECG R-peak detector over the full retained trace before deriving RR intervals. Peak detection is not repeated per epoch.
- RR intervals outside `[lwr, upr]` are excluded when computing the mean valid RR, then replaced with that mean before spectral interpolation. The `IMPUTED` variable reports the proportion of such intervals.
- Time-domain metrics (`SDNN`, `RMSSD`, `pNN50`) are computed exclusively from non-imputed intervals. Successive-difference metrics (`RMSSD`, `pNN50`) additionally skip any pair where either neighbour was imputed.
- `Rpk` annotations mark each R-peak as a point event. `RRint` annotations span from one R-peak to the next, covering the exact inter-beat interval. Because R-peak detection runs once over the whole retained trace, `Rpk` annotations are continuous across epoch boundaries; `RRint` annotations are written per epoch and therefore have a one-interval gap at each epoch boundary (the interval crossing the boundary is not emitted).
- Frequency-domain HRV is based on cubic-spline interpolation of the full (including imputed) RR series to a uniform 4 Hz grid, followed by Welch spectral estimation.
- When summarizing over epochs, epochs with fewer than 10 valid NN intervals are excluded. The summary is a NP-weighted average across valid epochs, so longer or higher-HR epochs contribute proportionally more. `LF_N`, `HF_N`, and `LF2HF` are derived from the weighted `LF` and `HF` means rather than averaged per-epoch ratios.
- Epoch duration affects frequency-domain reliability. Standard guidelines recommend ≥5 minutes for LF power (0.04–0.15 Hz). For typical 30 s sleep epochs the LF estimate is based on a single periodogram window and should be interpreted cautiously; HF (0.15–0.4 Hz, respiratory band) is more tractable. To run HRV over the full recording as a single unit, use `EPOCH len=<total_duration>` or `EPOCH contig` (which for a non-gapped EDF collapses the recording into one epoch).
- Annotation-stratified output reports time-domain metrics only; frequency-domain metrics require a continuous tachogram and are not meaningful for scattered annotation intervals.

<h3>Example</h3>

```bash
luna s.lst -s 'HRV sig=ECG epoch annot=REM add-annot=RPK add-annot-rr=RRINT'
```

## [`RAI`](physio.md#rai)

_Calculate the REM atonia index from chin EMG_

[`RAI`](physio.md#rai) computes a simple REM atonia index from a chin-EMG-like signal using existing 1-second epochs. For each epoch, Luna averages the rectified signal, subtracts a moving-minimum baseline, and compares the corrected amplitude to lower and upper thresholds.

<h3>Methods</h3>

For each 1-second epoch, [`RAI`](physio.md#rai) computes the mean of the absolute (rectified) signal amplitude. A moving-minimum baseline is estimated over a 61-epoch (approximately 60-second) window centred on each epoch, and the baseline-corrected amplitude for that epoch is defined as the mean rectified value minus this moving minimum. Epochs with baseline-corrected amplitudes below the lower threshold `th` are counted as atonic (consistent with REM atonia), epochs with amplitudes above the upper threshold `th2` are excluded from the index entirely. The REM Atonia Index is the proportion of non-excluded epochs that are atonic, computed as the count of atonic epochs divided by the sum of atonic and high-amplitude epochs.

<h3>Parameters</h3>

| Parameter | Example | Description |
| ---- | ---- | ---- |
| `sig` | `EMG` | Required EMG channel or channels |
| `th` | `1` | Lower atonia threshold; default `1` |
| `th2` | `2` | Upper exclusion threshold; default `2` |
| `verbose` |  | Emit per-epoch baseline-corrected amplitudes |

<h3>Outputs</h3>

| Table | Variable | Description |
| ---- | ---- | ---- |
| `CH` | `REM_AI` | REM atonia index |
| `CH` | `NE` | Number of epochs contributing to the index |
| `CH,N` | `X` | Baseline-corrected mean rectified amplitude for that epoch |

<h3>Notes</h3>

- [`RAI`](physio.md#rai) requires epochs to exist and requires those epochs to be exactly 1 second long.
- The implementation assumes a chin-EMG-like signal and was written with REM-focused analyses in mind.
- Epochs with corrected amplitudes between `th` and `th2` are excluded from the main index.

<h3>Example</h3>

```bash
luna s.lst -s 'EPOCH len=1 & RAI sig=EMG th=1 th2=2'
```

## [`AROUSALS`](physio.md#arousals)

_Detect candidate sleep arousals from EEG and optional EMG_

!!! warning "Under development"
    [`AROUSALS`](physio.md#arousals) is under development. Defaults, heuristics, and output details
    may still evolve.

[`AROUSALS`](physio.md#arousals) detects candidate arousals from EEG and optional EMG channels using short overlapping windows and a small set of derived features. It writes stage-stratified summary counts and feature means by class, adds annotation tracks for detected events, and can optionally add derived channels. With `manual=<annotation>`, it also summarizes an existing manual event annotation and, when automated events are available, reports one-to-one event-level comparison statistics.

<h3>Methods</h3>

[`AROUSALS`](physio.md#arousals) segments the recording into short overlapping windows (default 4-second duration, 0.5-second increment) and computes EEG/EMG rise features for each window. Candidate events are seeded by local EEG fast-frequency increases relative to a preceding baseline, with optional EMG-only NREM candidates and independent EMG confirmation for REM. Large local delta increases are reported as artifacts, and candidates below the standard duration threshold are discarded. Remaining events are reported as standard or long arousals.

<h3>Primary parameters</h3>

| Parameter | Example | Description |
| ---- | ---- | ---- |
| `eeg` | `C3,C4` | EEG channels used to derive arousal features |
| `emg` | `EMG` | Optional EMG channels used to derive EMG-rise features |
| `win` | `4.0` | Window length in seconds for feature extraction; default `4.0` |
| `inc` | `0.5` | Window increment in seconds; default `0.5` |
| `add` | `a_` | Add derived feature channels using this prefix |
| `manual` | `man_arousal` | Existing annotation class containing manual arousal events; can be used without EEG/EMG to summarize manual events |
| `manual-min-overlap` | `0` | Minimum positive overlap in seconds for a comparison match; matching is one-to-one |

<h3>Secondary parameters</h3>

| Parameter | Example | Description |
| ---- | ---- | ---- |
| `broad` | `T` | Use broader EEG seed thresholds |
| `nrem-emg-only` | `T` | Allow EMG-only NREM candidates (use cautiously) |
| `arousal-dur` | `3` | Minimum duration for reporting a standard event |
| `long-dur` | `30` | Maximum duration for long event classification |

<h3>Outputs</h3>

| Table | Variable | Description |
| ---- | ---- | ---- |
| `SS` | `N`, `AI`, `DUR` | Standard automated event count, index, and mean duration by NREM/REM stage |
| `SS` | `N_ALL`, `N_LONG` | Automated standard/long and long event counts |
| `SS` | `N_MAN`, `AI_MAN`, `DUR_MAN` | Manual event count, index, and mean duration when `manual=` is supplied |
| `SS,CLS=manual` | `NE`, `AI`, `DUR` | Manual event summary class by stage |
| `COMP=1,SS` | `N_AUTO`, `N_MANUAL`, `TP`, `FP`, `FN` | Automated/manual event counts and one-to-one matches |
| `COMP=1,SS` | `PRECISION`, `RECALL`, `F1` | Event-level comparison metrics |
| `COMP=1,SS` | `OVERLAP_SEC` | Total overlap duration for matched pairs |
| `COMP=1,SS` | `UNION_ALL_SEC` | Duration of the union of all automated and manual intervals |
| `COMP=1,SS` | `OVERLAP_UNION_ALL` | Matched-pair `OVERLAP_SEC` divided by the union of all automated and manual intervals |
| `COMP=1,SS` | `UNION_SEC`, `OVERLAP_UNION` | Union duration and intersection/union fraction for matched automated/manual pairs |
| `COMP=1,SS` | `P_AUTO`, `P_MANUAL` | Mean fraction of matched automated/manual event duration covered by the other event |
| `COMP=1,SS` | `START_D`, `START_DABS` | Mean signed and absolute automated-minus-manual onset difference in seconds |
| `COMP=1,SS` | `DUR_D` | Mean signed automated-minus-manual duration difference in seconds |

<h3>Notes</h3>

- The implementation resets epochs internally using `win` and `inc`, and requires EDF record durations that are a multiple of 1 second for signal-based detection.
- EEG sample rates must be sufficiently high and consistent across EEG channels; the current code rejects EEG sample rates below 60 Hz.
- Automated events are emitted as `arousal_nrem/rem`, `arousal_long_nrem/rem`, and aggregate classes such as `arousal_all_nrem/rem`.
- Manual events are assigned to NREM or REM by the stage with the greatest temporal overlap. Events with no sleep-stage overlap are not included in stage-stratified manual summaries.
- Comparison uses `arousal_all_nrem/rem` versus the manual annotation, any positive overlap by default, and greedy one-to-one matching. `manual-min-overlap` can require a larger overlap. `START_D` is automated start minus manual start, so positive values mean the automated event starts later; `DUR_D` is automated duration minus manual duration.
- If no EEG or EMG signal is available, `manual=<annotation>` still produces the manual summaries; comparison fields then have zero automated events.

<h3>Example</h3>

```bash
luna s.lst -s 'AROUSALS eeg=C3,C4 emg=EMG manual=man_arousal add=a_'

# Manual-only summary (no EEG/EMG required)
luna s.lst -s 'AROUSALS manual=man_arousal'
```

## [`DESAT`](physio.md#desat)

_Detect oxygen desaturation events from SpO2_

!!! warning "Under development"
    [`DESAT`](physio.md#desat) is under development. Parameter defaults, event definitions, and
    output details may still evolve.

[`DESAT`](physio.md#desat) detects oxygen desaturation events from an SpO2 signal using a forward
scanning algorithm with a rolling median baseline. The implementation is
designed for overnight oximetry traces that can contain integer quantization,
dropouts, transient spikes, and slow drift.

<h3>Methods</h3>

[`DESAT`](physio.md#desat) first flags hard-artifact samples, including values below a user-set
lower bound and large sample-to-sample spikes. It then processes each
contiguous EDF segment independently. At each point in the valid signal, Luna
maintains a rolling median baseline computed from the prior `baseline` seconds
of valid, non-artifact, non-desaturation signal. A desaturation event starts
when SpO2 falls at least `drop` units below this local baseline and ends when
the trace recovers to at least `baseline - drop * recovery`. Events lasting at
least `dur` seconds are retained. By default, Luna writes artifact intervals and
desaturation intervals back as annotations and reports both recording-level and
per-event summaries. An alternative `mode=matlab` path uses a peak-valley style
algorithm intended to match an earlier Matlab-oriented workflow more closely.

<h3>Primary parameters</h3>

| Parameter | Example | Description |
| ---- | ---- | ---- |
| `sig` | `SpO2` | Required SpO2 channel; expected on a 0-100% scale |
| `drop` | `3` | Minimum drop below baseline required to trigger an event |
| `dur` | `10` | Minimum event duration in seconds |
| `baseline` | `120` | Rolling baseline window in seconds |
| `recovery` | `0.5` | Recovery fraction used to define event termination |

<h3>Secondary parameters</h3>

| Parameter | Example | Description |
| ---- | ---- | ---- |
| `low` | `50` | Samples below this SpO2 value are treated as artifact |
| `spike` | `10` | Consecutive-sample spike threshold for artifact detection |
| `min-bsln` | `30` | Minimum valid samples required before baseline-based detection starts |
| `pct-th` | `90,88,85,80` | SpO2 thresholds used for time-below-threshold summaries |
| `annot` | `desat` | Annotation label for desaturation events |
| `art-annot` | `spo2-artifact` | Annotation label for artifact regions |
| `mode` | `matlab` | Use the alternative peak-valley Matlab-style implementation |

<h3>Outputs</h3>

Recording-level output includes:

| Variable | Description |
| ---- | ---- |
| `N` | Number of desaturation events |
| `T_VALID` | Total valid non-artifact signal duration (s) |
| `T_ART` | Total artifact duration (s) |
| `T_DESAT` | Total time spent in desaturation events (s) |
| `MEAN_SPO2` | Mean SpO2 across valid samples |
| `NADIR_MEAN` | Mean event nadir |
| `NADIR_MIN` | Minimum nadir observed |
| `DROP_MEAN` | Mean desaturation magnitude |
| `DUR_MEAN` | Mean event duration |
| `DUR_TOTAL` | Total event duration |
| `PCT_LT90` | Percent valid time below 90% |
| `PCT_LT88` | Percent valid time below 88% |
| `PCT_LT85` | Percent valid time below 85% |
| `PCT_LT80` | Percent valid time below 80% |

Per-event output (strata: [`DESAT`](physio.md#desat)) includes `START`, `STOP`, `DUR`, `NADIR`,
`BASELINE`, `DROP`, and `SEG`.

With `mode=matlab`, Luna also emits Matlab-mode event tables (`DESAT_M`) and
summary variables such as `N2`, `N3`, `N4`, `ODI2`, `ODI3`, `ODI4`, and
`T_SLEEP_VALID`.

<h3>Example</h3>

```bash
luna s.lst -s 'DESAT sig=SpO2 drop=3 dur=10'
```

## [`RESP-LINK`](physio.md#resp-link)

_Link respiratory events to likely desaturation and arousal responses_

!!! warning "Under development"
    [`RESP-LINK`](physio.md#resp-link) is under development: documentation is
    being written, but the command is not yet ready for general use and is
    highly likely to change (template learning, fallback timing, and linkage
    thresholds may still evolve).

## [`RESPBREATH`](physio.md#respbreath)

_Respiratory breath segmentation for timing / phase-locking analyses_

!!! note "Reconstructed section"
    This section was rebuilt from the `RESPBREATH` command definition in
    `cmddefs.cpp` after the original prose was accidentally deleted during an
    editing pass; please review it against the actual command behavior.

[`RESPBREATH`](physio.md#respbreath) detects individual breaths from one or more respiratory PSG
channels (nasal cannula, thermistor, effort belt) and emits breath-level timing
annotations plus summary statistics. The primary goal is robust breath timing
for downstream phase-locking / coupling analyses with EEG or other signals.

<h3>Methods</h3>

Each contiguous EDF segment is processed independently (no bridging across
gaps). Each channel is bandpass-filtered (Butterworth IIR) and lightly
smoothed, and signal polarity is auto-detected or user-specified so that
inspiration is upward. Alternating local extrema (troughs and peaks) are then
detected and filtered by prominence and physiologic timing constraints; each
accepted trough-peak-trough triplet defines one breath. Regions with no
reliable timing are annotated as `RESP_ART`. With multiple channels, results
are fused: the best-quality channel is primary and secondary channels boost
confidence or rescue timing where the primary fails.

<h3>Primary parameters</h3>

| Parameter | Example | Description |
| ---- | ---- | ---- |
| `sig` | `NASAL,THOR` | _(Required)_ Comma-separated list of respiratory channel labels |
| `primary` | `auto` | Primary channel: `first`, `auto` (best quality), or a channel label |
| `fuse` | `yes` | Perform multi-channel fusion when multiple `sig=` channels are given |
| `fuse-window` | `0.75` | Time window (s) for matching breath peaks across channels during fusion |

<h3>Preprocessing parameters</h3>

| Parameter | Example | Description |
| ---- | ---- | ---- |
| `hp` | `0.03` | High-pass cutoff for bandpass filter (Hz) |
| `lp` | `1.5` | Low-pass cutoff for bandpass filter (Hz) |
| `smooth` | `0.20` | Box-car smoothing half-window (s) applied after filtering |
| `flip` | `auto` | Polarity: `auto` (inferred), `yes` (invert), `no` (keep as-is) |

<h3>Timing-constraint parameters</h3>

| Parameter | Example | Description |
| ---- | ---- | ---- |
| `min-half-cycle` | `0.40` | Minimum inspiratory or expiratory half-cycle duration (s) |
| `max-half-cycle` | `6.00` | Maximum inspiratory or expiratory half-cycle duration (s) |
| `min-cycle` | `0.80` | Minimum total breath cycle duration (s) |
| `max-cycle` | `12.0` | Maximum total breath cycle duration (s) |

<h3>Quality and artifact parameters</h3>

| Parameter | Example | Description |
| ---- | ---- | ---- |
| `prom-z` | `1.0` | Minimum extremum prominence (signal / local MAD) |
| `amp-rel` | `0.15` | Minimum breath amplitude relative to recent-good-breath median |
| `conf` | `0.40` | Confidence threshold below which a breath is flagged low-confidence |
| `recover` | `2.0` | Artifact recovery window (s): breaths within this distance of artifact are flagged |
| `merge-art` | `1.0` | Gap (s) between artifact intervals to merge into one |
| `min-art` | `2.0` | Minimum duration (s) for an artifact interval to be annotated |
| `min-seg` | `5.0` | Minimum segment duration (s) required to attempt breath detection |

<h3>Other parameters</h3>

| Parameter | Example | Description |
| ---- | ---- | ---- |
| `use-hilbert` | `no` | Use Hilbert envelope for amplitude/noise estimation (currently a placeholder) |
| `verbose` | `no` | Verbose logging |
| `annot-breath` | `BREATH` | Annotation label for breath intervals |
| `annot-insp` | `INSP` | Annotation label for inspiratory sub-intervals (omit to suppress) |
| `annot-exp` | `EXP` | Annotation label for expiratory sub-intervals (omit to suppress) |
| `annot-art` | `RESP_ART` | Annotation label for artifact/unusable intervals |

<h3>Outputs</h3>

Individual-level summary (all segments combined):

| Variable | Description |
| ---- | ---- |
| `N_BREATH` | Total number of detected breaths |
| `BREATH_RATE` | Breaths per minute (over total valid recording time) |
| `MEAN_TTOT` | Mean total cycle duration (s) |
| `MEDIAN_TTOT` | Median total cycle duration (s) |
| `SD_TTOT` | SD of total cycle duration (s) |
| `CV_TTOT` | CV of total cycle duration (SD/mean) |
| `MEDIAN_TINSP` | Median inspiratory duration (s) |
| `MEDIAN_TEXP` | Median expiratory duration (s) |
| `IE_RATIO` | Median I:E ratio (`TINSP` / `TEXP`) |
| `MEDIAN_AMP` | Median breath amplitude (symmetric: peak - mean adjacent troughs) |
| `LOWCONF_PCT` | Percentage of breaths flagged as low confidence |
| `ARTIFACT_PCT` | Percentage of recording time covered by `RESP_ART` |
| `FUSED_PCT` | Percentage of breaths supported by multiple channels |

Per-breath output (strata: `CH,N`) includes `START`, `PEAK`, `END`, `TINSP`,
`TEXP`, `TTOT`, `AMP_INSP`, `AMP_EXP`, `AMP_SYM`, `CONF`, `LOW_CONF`,
`FUSED`, and `N_SUPPORT`.

With multi-channel input, Luna also emits a per-channel summary table (`CH`)
including `PRIMARY_USED_PCT`.

<h3>Example</h3>

```bash
luna s.lst -s 'RESPBREATH sig=NASAL,THOR primary=auto fuse=yes'
```

## [`LM`](physio.md#lm)

_Detect leg movements and periodic leg movements (WASM 2016)_

!!! warning "Under development"
    [`LM`](physio.md#lm) is under development: documentation is being written,
    but the command is not yet ready for general use and is highly likely to
    change (detection methods, parameters, and output format).

## [`COMBINE-EMG`](physio.md#combine-emg)

_Build a single continuous EMG channel from 2+ candidate channels_

!!! warning "Under development"
    [`COMBINE-EMG`](physio.md#combine-emg) is under development: documentation
    is being written, but the command is not yet ready for general use and is
    highly likely to change.
