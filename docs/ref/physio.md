# Physiological Signal Analysis

_Commands for cardiovascular, muscle-tone, arousal, oximetry, and respiratory analyses_

These commands analyse non-EEG physiological channels commonly recorded in PSG. [`HRV`](physio.md#hrv) detects R-peaks in an ECG channel and computes a standard set of time-domain and frequency-domain heart-rate variability metrics, optionally by epoch or by annotation class. [`RAI`](physio.md#rai) quantifies muscle tone suppression during REM sleep by computing the REM Atonia Index from a chin EMG channel. [`AROUSALS`](physio.md#arousals) detects candidate sleep arousals from EEG spectral power and optional EMG signals, without requiring pre-scored annotations. [`DESAT`](physio.md#desat) detects oxygen desaturation events from SpO2 signals, and [`RESPBREATH`](physio.md#respbreath) segments respiratory signals into breath-level intervals for downstream timing and phase-locking analyses.

| Command | Description |
| ---- | ------ |
| [`HRV`](#hrv) | Estimate heart-rate variability metrics from ECG |
| [`RAI`](#rai) | Calculate the REM atonia index from chin EMG |
| [`AROUSALS`](#arousals) | Detect candidate sleep arousals from EEG and optional EMG |
| [`DESAT`](#desat) | Detect oxygen desaturation events from SpO2 |
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

[`AROUSALS`](physio.md#arousals) detects candidate arousals from EEG and optional EMG channels using short overlapping windows and a small set of derived features. In the current active implementation, Luna re-epochs the record into overlapping windows, derives EEG and EMG feature summaries, applies a heuristic event classifier, writes summary counts and feature means by class, adds annotation tracks for detected events, and can optionally add derived channels.

<h3>Methods</h3>

[`AROUSALS`](physio.md#arousals) segments the recording into short overlapping windows (default 1-second duration, 0.5-second increment) and computes a multi-dimensional feature vector for each window from the provided EEG and optional EMG channels. Per-channel EEG features comprise total broadband log-power, relative beta-band power, relative sigma-band power, and a complexity index (Hjorth parameter H3); the EMG feature is the root-mean-square amplitude thresholded against a whole-night median-absolute-deviation (MAD) estimate. All per-channel features are baseline-corrected by subtracting a 2-minute median filter, then averaged across channels and robustly normalized — using the median and 1.4826 × MAD computed over sleep-only windows — to produce a normalized feature matrix. The normalized features are clipped to ±6 standard deviations. Candidate arousal events are detected from NREM windows: windows are first flagged as artifact if they show implausible broadband power with low beta or extreme complexity. Among non-artifact windows, local beta-power peaks exceeding a primary threshold are identified; each peak is expanded backward and forward using a hysteresis threshold, subject to a sigma-band veto that suppresses spindle activity from being misclassified as arousal. Contiguous expanded regions form candidate arousal events; events below a duration threshold are classified as micro-arousals.

<h3>Primary parameters</h3>

| Parameter | Example | Description |
| ---- | ---- | ---- |
| `eeg` | `C3,C4` | EEG channels used to derive arousal features |
| `emg` | `EMG` | Optional EMG channels used to augment detection |
| `win` | `1.0` | Window length in seconds for feature extraction; default `1.0` |
| `inc` | `0.5` | Window increment in seconds; default `0.5` |
| `winsor` | `0.005` | Winsorization fraction for outlier handling |
| `no-winsor` |  | Disable winsorization |
| `add` | `a_` | Add derived feature channels using this prefix |

<h3>Secondary parameters</h3>

| Parameter | Example | Description |
| ---- | ---- | ---- |
| `annot` | `l` | Annotation prefix registered for detected events |
| `prefix` | `ar_` | Registered prefix for newly derived feature channels |
| `per-channel` | `T` | Retain channel-specific feature metrics when adding channels |

<h3>Outputs</h3>

| Table | Variable | Description |
| ---- | ---- | ---- |
| `BL` | `MINS` | Total analyzed duration in minutes |
| `BL` | `N` | Number of detected arousals |
| `BL` | `AI` | Arousal index per hour |
| `BL` | `DUR` | Mean duration of detected arousals |
| `BL` | `N_MICRO` | Number of detected micro-arousals |
| `BL` | `AI_MICRO` | Micro-arousal index per hour |
| `BL` | `DUR_MICRO` | Mean duration of detected micro-arousals |
| `BL` | `N_ART` | Number of detected artifact events |
| `BL` | `AI_ART` | Artifact-event index per hour |
| `CLS` | `NE` | Number of windows assigned to this class |
| `CLS` | `PWR` | Mean total-power feature for this class |
| `CLS` | `BETA` | Mean relative beta feature for this class |
| `CLS` | `EMG` | Mean EMG feature for this class |
| `CLS` | `SIGMA` | Mean sigma-band feature for this class |
| `CLS` | `CMPLX` | Mean complexity feature for this class |

<h3>Notes</h3>

- The current implementation resets epochs internally using `win` and `inc`, and requires EDF record durations that are a multiple of 1 second.
- EEG sample rates must be sufficiently high and consistent across EEG channels; the current code rejects EEG sample rates below 60 Hz.
- The active implementation currently analyzes NREM windows only and emits heuristic event classes including arousal, micro-arousal, and artifact.
- In the active heuristic code path, annotations are written with fixed names such as `arousal_nrem`, `micro_arousal_nrem`, and `art_nrem`.
- When `add` is used, Luna can write derived feature channels representing total power, beta, EMG, sigma, and complexity-like summaries.

<h3>Example</h3>

```bash
luna s.lst -s 'AROUSALS eeg=C3,C4 emg=EMG add=a_'
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

## [`RESPBREATH`](physio.md#respbreath)

_Segment respiratory channels into breath events_

!!! warning "Under development"
    [`RESPBREATH`](physio.md#respbreath) is under development. Interfaces and defaults may still change
    as the implementation is exercised on more respiratory channel types.

[`RESPBREATH`](physio.md#respbreath) detects individual breaths from one or more respiratory PSG
channels, such as nasal cannula, thermistor, or effort-belt signals, and emits
breath-level annotations plus summary timing statistics. The primary use case is
to obtain robust breath timing for downstream event-locking or phase/coupling
analyses against EEG or other signals.

<h3>Methods</h3>

Each contiguous EDF segment is processed independently. For each respiratory
channel, Luna applies a Butterworth band-pass filter and light smoothing, then
either infers or accepts a user-specified polarity so that inspiration is
upward. The algorithm identifies alternating troughs and peaks, filters them by
prominence and physiologic timing constraints, and uses accepted
trough-peak-trough triplets to define breaths. Regions with unreliable timing
are annotated as respiratory artifact. If multiple respiratory channels are
provided, Luna can select a primary channel and fuse evidence across channels so
that secondary channels increase confidence or rescue timing when the primary
channel is poor.

<h3>Primary parameters</h3>

| Parameter | Example | Description |
| ---- | ---- | ---- |
| `sig` | `NASAL,THOR` | Required respiratory channel or channels |
| `hp` | `0.03` | High-pass cutoff for band-pass filtering |
| `lp` | `1.5` | Low-pass cutoff for band-pass filtering |
| `smooth` | `0.20` | Smoothing half-window in seconds |
| `flip` | `auto` | Polarity handling: auto, yes, or no |

<h3>Timing and quality parameters</h3>

| Parameter | Example | Description |
| ---- | ---- | ---- |
| `min-half-cycle` | `0.40` | Minimum inspiratory or expiratory half-cycle duration |
| `max-half-cycle` | `6.0` | Maximum inspiratory or expiratory half-cycle duration |
| `min-cycle` | `0.80` | Minimum total breath duration |
| `max-cycle` | `12.0` | Maximum total breath duration |
| `prom-z` | `1.0` | Minimum extremum prominence relative to local MAD |
| `amp-rel` | `0.15` | Minimum amplitude relative to recent good breaths |
| `conf` | `0.40` | Threshold below which breaths are flagged low-confidence |

<h3>Artifact, fusion, and annotation parameters</h3>

| Parameter | Example | Description |
| ---- | ---- | ---- |
| `recover` | `2.0` | Artifact recovery window in seconds |
| `merge-art` | `1.0` | Gap threshold used to merge nearby artifact intervals |
| `min-art` | `2.0` | Minimum artifact duration to annotate |
| `min-seg` | `5.0` | Minimum segment duration required to attempt detection |
| `primary` | `auto` | Primary channel: first, auto, or an explicit label |
| `fuse` | `yes` | Enable multi-channel fusion |
| `fuse-window` | `0.75` | Matching window used when fusing channels |
| `annot-breath` | `BREATH` | Breath-interval annotation label |
| `annot-insp` | `INSP` | Inspiratory sub-interval label |
| `annot-exp` | `EXP` | Expiratory sub-interval label |
| `annot-art` | `RESP_ART` | Artifact / unusable interval label |

<h3>Outputs</h3>

Recording-level summary variables include:

| Variable | Description |
| ---- | ---- |
| `N_BREATH` | Total detected breaths |
| `BREATH_RATE` | Breaths per minute over valid time |
| `MEAN_TTOT` | Mean total cycle duration |
| `MEDIAN_TTOT` | Median total cycle duration |
| `SD_TTOT` | SD of total cycle duration |
| `CV_TTOT` | Coefficient of variation of total cycle duration |
| `MEDIAN_TINSP` | Median inspiratory duration |
| `MEDIAN_TEXP` | Median expiratory duration |
| `IE_RATIO` | Median inspiratory:expiratory ratio |
| `MEDIAN_AMP` | Median breath amplitude |
| `LOWCONF_PCT` | Percent of breaths flagged low confidence |
| `ARTIFACT_PCT` | Percent of recording time annotated as respiratory artifact |
| `FUSED_PCT` | Percent of breaths supported by multiple channels |

Per-breath output (strata: `CH,N`) includes `START`, `PEAK`, `END`, `TINSP`,
`TEXP`, `TTOT`, `AMP_INSP`, `AMP_EXP`, `AMP_SYM`, `CONF`, `LOW_CONF`,
`FUSED`, and `N_SUPPORT`.

With multi-channel input, Luna also emits a per-channel summary table (`CH`)
including `PRIMARY_USED_PCT`.

<h3>Example</h3>

```bash
luna s.lst -s 'RESPBREATH sig=NASAL,THOR primary=auto fuse=yes'
```
