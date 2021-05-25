# Experimental / Unsupported / Legacy functions

!!! Danger "Warning"
    This section should be considered as __nothing more than our
    own internal notes and reminders on some features that are being
    prototyped or are under very heavy development__, etc.  That is,
    these are a) highly likely to change, b) will not necessarily use
    Luna's standard [output mechanims](#../luna/outputs.md), c) may be
    removed in future versions, and d) may be buggy, incomplete or
    both.  Stay well clear! You've been warned!

|Command |Description |	       
|---|---|
| [`L1OUT`](#l1out)    | Leave-one-out interpolation-based signal check |
| [`EMD`](#emd)    | Empirical mode decomposition |
| [`ED`](#ed)      | Diagnostic for electrical bridging |
| [`POL`](#pol)    | Polarity check heuristic for sleep EEG |
| [`FIP`](#fip)    | Frequency-interval plots | 
| [`HR`](#hr)                 | Estimate per-epoch heart rate from ECG |
| [`SPIKE`](#spike)           | Create a synthetic signal by combining part of one signal with another |
| [`ZR`](#zr)   |  Calculate per-epoch Z-ratio |



<h5>Parametes</h5>

| Option | Description | 
| ---- | ---- | 
| `signal` | Specify which channels/signals to include |

<h5>Outputs</h5>

_to be completed_

<h5>Example</h5>

_to be completed_


## `L1OUT`

_Leave-one-out interpolation-based signal check_


## `EMD`

_Empirical mode decomposition_

Empirical mode decomposition, or the Hilbert-Huang transform
(described [here](https://en.wikipedia.org/wiki/Hilbert%E2%80%93Huang_transform))

## `ED`

_Heuristic method to spot electrical bridging_

To be completed.


## `POL`

_Polarity checks for EEG sleep signals_

Heuristics to see if signals have been fliped, based on the aysmmetrical shape of slow oscillations.

To be completed.


## `FIP`

_Frequency-interval plots_

Experimental method, to be completed.


## `HR`

_A modified Pan-Tompkins algorithm for detect R peaks in the ECG_

_to be completed_


## `SPIKE`

_Merges two signals_

| Parameter | Example | Description |
| ---- | ---- | ---- |
|`from` | | Original _from_ signal |
|`to`   | | Original _to_ signal |
|`new`  | | Label for new signal|
|`wgt`  | | Weight | 

_to be completed_

## `ZR`

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
