# Physiological Signal Analysis

_Commands for heart-rate, muscle-tone, and arousal analyses_

| Command | Description |
| ---- | ------ |
| [`HRV`](#hrv) | Estimate heart-rate variability metrics from ECG |
| [`RAI`](#rai) | Calculate the REM atonia index from chin EMG |
| [`AROUSALS`](#arousals) | Detect candidate sleep arousals from EEG and optional EMG |

## `HRV`

_Estimate heart-rate variability metrics from ECG_

`HRV` detects ECG R peaks, derives RR intervals, optionally annotates those peaks and intervals, and then reports time-domain and frequency-domain HRV metrics. It can be run on a whole recording, by epoch, and optionally within specified annotation classes.

### Primary parameters

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
| `w` | `5` | Median-filter width for RR cleaning; default `5` |
| `ns` | `512` | Welch segment length for frequency-domain HRV; default `512` |

### Secondary parameters

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

### Outputs

`HRV` can emit up to four tables:

| Table | Description |
| ---- | ---- |
| `CH` | Whole-trace or epoch-summarized HRV metrics |
| `CH,E` | Epoch-level HRV metrics with `epoch` |
| `CH,ANNOT` | Annotation-stratified HRV metrics with `annot` |
| `CH,ANNOT,INST` | Annotation-instance-stratified HRV metrics with `annot` and `by-instance` |

The main output variables are:

| Variable | Description |
| ---- | ---- |
| `IMPUTED` | Number or proportion of RR intervals that were imputed after range filtering |
| `P_INV` | Estimated probability that the ECG is inverted |
| `INV` | 0/1 inversion flag |
| `NP` | Number of retained R peaks / RR intervals |
| `NP_TOT` | Total number of R peaks before exclusions or summarization |
| `RR` | Mean RR interval |
| `HR` | Mean heart rate |
| `SDNN` | Standard deviation of NN intervals |
| `SDNN_R` | Relative or robust SDNN summary |
| `RMSSD` | Root mean square successive RR difference |
| `RMSSD_R` | Relative or robust RMSSD summary |
| `pNN50` | Proportion of successive RR differences exceeding 50 ms |
| `LF` | Low-frequency HRV power |
| `HF` | High-frequency HRV power |
| `LF_N` | Normalized low-frequency power |
| `HF_N` | Normalized high-frequency power |
| `LF_PK` | Peak low-frequency HRV frequency |
| `HF_PK` | Peak high-frequency HRV frequency |
| `LF2HF` | Low-to-high frequency power ratio |

### Notes

- `HRV` uses Luna's internal ECG R-peak detector before deriving RR intervals.
- RR intervals outside the accepted range are excluded when estimating the mean valid RR, then mean-imputed before interpolation and spectral estimation.
- Frequency-domain HRV is based on cubic-spline interpolation of RR intervals to a uniform 4 Hz grid followed by Welch estimation.
- Annotation-stratified output currently reports the reduced time-domain set only.

### Example

```bash
luna s.lst -s 'HRV sig=ECG epoch annot=REM add-annot=RPK add-annot-rr=RRINT'
```

## `RAI`

_Calculate the REM atonia index from chin EMG_

`RAI` computes a simple REM atonia index from a chin-EMG-like signal using existing 1-second epochs. For each epoch, Luna averages the rectified signal, subtracts a moving-minimum baseline, and compares the corrected amplitude to lower and upper thresholds.

### Parameters

| Parameter | Example | Description |
| ---- | ---- | ---- |
| `sig` | `EMG` | Required EMG channel or channels |
| `th` | `1` | Lower atonia threshold; default `1` |
| `th2` | `2` | Upper exclusion threshold; default `2` |
| `verbose` |  | Emit per-epoch baseline-corrected amplitudes |

### Outputs

| Table | Variable | Description |
| ---- | ---- | ---- |
| `CH` | `REM_AI` | REM atonia index |
| `CH` | `NE` | Number of epochs contributing to the index |
| `CH,N` | `X` | Baseline-corrected mean rectified amplitude for that epoch |

### Notes

- `RAI` requires epochs to exist and requires those epochs to be exactly 1 second long.
- The implementation assumes a chin-EMG-like signal and was written with REM-focused analyses in mind.
- Epochs with corrected amplitudes between `th` and `th2` are excluded from the main index.

### Example

```bash
luna s.lst -s 'EPOCH len=1 & RAI sig=EMG th=1 th2=2'
```

## `AROUSALS`

_Detect candidate sleep arousals from EEG and optional EMG_

`AROUSALS` detects candidate arousals from EEG and optional EMG channels using short overlapping windows and a small set of derived features. In the current active implementation, Luna re-epochs the record into overlapping windows, derives EEG and EMG feature summaries, applies a heuristic event classifier, writes summary counts and feature means by class, adds annotation tracks for detected events, and can optionally add derived channels.

### Primary parameters

| Parameter | Example | Description |
| ---- | ---- | ---- |
| `eeg` | `C3,C4` | EEG channels used to derive arousal features |
| `emg` | `EMG` | Optional EMG channels used to augment detection |
| `win` | `1.0` | Window length in seconds for feature extraction; default `1.0` |
| `inc` | `0.5` | Window increment in seconds; default `0.5` |
| `winsor` | `0.005` | Winsorization fraction for outlier handling |
| `no-winsor` |  | Disable winsorization |
| `add` | `a_` | Add derived feature channels using this prefix |

### Secondary parameters

| Parameter | Example | Description |
| ---- | ---- | ---- |
| `annot` | `l` | Annotation prefix registered for detected events |
| `prefix` | `ar_` | Registered prefix for newly derived feature channels |
| `per-channel` | `T` | Retain channel-specific feature metrics when adding channels |

### Outputs

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

### Notes

- The current implementation resets epochs internally using `win` and `inc`, and requires EDF record durations that are a multiple of 1 second.
- EEG sample rates must be sufficiently high and consistent across EEG channels; the current code rejects EEG sample rates below 60 Hz.
- The active implementation currently analyzes NREM windows only and emits heuristic event classes including arousal, micro-arousal, and artifact.
- In the active heuristic code path, annotations are written with fixed names such as `arousal_nrem`, `micro_arousal_nrem`, and `art_nrem`.
- When `add` is used, Luna can write derived feature channels representing total power, beta, EMG, sigma, and complexity-like summaries.

### Example

```bash
luna s.lst -s 'AROUSALS eeg=C3,C4 emg=EMG add=a_'
```
