# Experimental / Unsupported / Legacy functions

!!! Danger "Warning"
    This section should be considered as __nothing more than our
    own internal notes and reminders on some features that are being
    prototyped or are under very heavy development__, etc.  That is,
    these are a) highly likely to change, b) will not necessarily use
    Luna's standard [output mechanisms](../luna/outputs.md), c) may be
    removed in future versions, and d) may be buggy, incomplete or
    both.  Stay well clear! You've been warned!  Whereas some commands listed here will
    (hopefully!) be elevated to be part of the main release, and more fully documented,
    others may remain here or be removed in future releases.

|Command |Description |	       
|---|---|
| [`DFA`](#dfa) | Detrended flucation analysis |
| [`ASYMM`](#dfa) | Lateralization indices |
| [`A2C`](#a2c) | Annotation to cache entry |
| [`L1OUT`](#l1out)    | Leave-one-out interpolation-based signal check |
| [`ED`](#ed)      | Diagnostic for electrical bridging |
| [`POL`](#pol)    | Polarity check heuristic for sleep EEG |
| [`FIP`](#fip)    | Frequency-interval plots | 
| [`HR`](#hr)                 | Estimate per-epoch heart rate from ECG |
| [`SPIKE`](#spike)           | Create a synthetic signal by combining part of one signal with another |
| [`ZR`](#zr)   |  Calculate per-epoch Z-ratio |

## DFA

_Implements the detrended fluctuation analysis (DFA)_

Implements DFA as described by [Nolte et al
(2019)](https://pubmed.ncbi.nlm.nih.gov/31004085/), which adopts an
efficient, Fourier-based approach to DFA, in order to study
long-range temporal correlations in time series.

Briefly, a signal is split into detrended segments of length L, and
the standard deviation (or _fluctuations_) of segments is calculated,
for different length L.  Under a power law, one expects a linear
relationship between the log-scaled flucuations and the log of L; the
slope of that line estimates the so-called Hurst exponent, which is a
rescaled version of the spectral slope as estimated using other
frequency-domain approaches (e.g. [`IRASA`](power-spectra.md#irasa)).   See the above paper
for a clear description of the full method.   That paper also describes
a frequency-domain approach to DFA, which is directly implemented here
(currently w/ the boxcar rather than Gaussian window, however).


<h3>Parameters</h3>

| Parameter | Example |Description |
| ---- | -----| ----- |
| `sig`   | `C3,C4` | Signal(s) to analyse |
| `n`     | `50`  | Number of different window sizes to create (default 100) |
| `min`   | `0.1` | Minimum window length (time in seconds, default 0.1) |
| `m`     | `2`   | Integer power to specify maximum window size (default 2 implies `min` scaled by factor `10^0` to `10^2`, i.e. 0.1s to 10s) | 
| `f-lwr` | `10`  | Lower transition for band-pass filter |
| `f-upr` | `15`  | Upper transition for band-pass filter |
| `ripple` | `0.05` | FIR ripple (default 0.02) |
| `tw` | `2` | FIR transition width (default 0.5 Hz) |
| `epoch` | | Report 

<h3>Outputs</h3>

Epoch-level DFA statistics (strata: `CH` x `SEC`)

| Variable | Description |
| ----- | ----- |
|`FLUCT` | Fluctuations |
|`SLOPE` | Slopes |

Epoch-level DFA statistics (option: `epoch`, strata: `E` x `CH` x `SEC`)

| Variable | Description |
| ----- | ----- |
|`FLUCT` | Epoch fluctuations |
|`SLOPE` | Epoch slopes |


<h3>Example</h3>

_to be added_


## ASYMM

_Evaluates (regional) signal asymmetries in specifie metrics_

To be completed.


## A2C

_Writes an annotation to a set of cache entries_

Primarily an internal command for testing/development.

Documentation to be completed.


## L1OUT

_Leave-one-out interpolation-based signal check_



## ED

_Heuristic method to spot electrical bridging_

To be completed.


## POL

_Polarity checks for EEG sleep signals_

Heuristics to see if signals have been fliped, based on the aysmmetrical shape of slow oscillations.

To be completed.


## FIP

_Frequency-interval plots_

Experimental method, to be completed.


## HR

_A modified Pan-Tompkins algorithm for detect R peaks in the ECG_

_to be completed_


## SPIKE

_Merges two signals_

| Parameter | Example | Description |
| ---- | ---- | ---- |
|`from` | | Original _from_ signal |
|`to`   | | Original _to_ signal |
|`new`  | | Label for new signal|
|`wgt`  | | Weight | 

_to be completed_

## ZR

_Calculates per-epoch Z-ratio_

Implementation of a simple heuristic designed to index sleep depth, as
described
[here](https://www.ncbi.nlm.nih.gov/pubmed/8746389). Requires epoched
30-second EEG data.  It is based on frequency ranges, which are
different from Luna's default band definitions:

- delta: 0.5 to 2hz
- theta: 2.5 to 7.5 Hz
- alpha: 8 to 12.5 Hz
- beta: 13 to 30 Hz
