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
| [`ICA`](#ica)    | Independent component analysis |  
| [`CLOCS`](#clocs)    | Specify channel topographic locations |
| [`INTERPOLATE`](#interpolate)    | Spherical spline interpolation |
| [`L1OUT`](#l1out)    | Leave-one-out interpolation-based signal check |
| [`SL`](#sl) | Surface Laplacian spatial filter |
| [`EMD`](#emd)    | Empirical mode decomposition |
| [`ED`](#ed)      | Diagnostic for electrical bridging |
| [`POL`](#pol)    | Polarity check heuristic for sleep EEG |
| [`FIP`](#fip)    | Frequency-interval plots | 
| [`EXE`](#exe)    | Epoch-wise distance/similarity matrix | 
| [`TSLIB`](#tslib) | Build library for SSS |
| [`SSS`](#sss) | Simple sleep stager | 


## `ICA`

_Independent components analysis_

To be completed: a wrapper around the
[fastICA](https://research.ics.aalto.fi/ica/fastica/) C/C++ package,
to provide
[ICA](https://en.wikipedia.org/wiki/Independent_component_analysis)
functionality.


## `CLOCS`

_Specify channel topographic locations_


## `INTERPOLATE`

_Spherical spline interpolation_


## `L1OUT`

_Leave-one-out interpolation-based signal check_


## `SL`

_Surface Laplacian spatial filter_


## `EMD`

_Empirical mode decomposition_


Empirical mode decomposition, or the Hilbert-Huang transform
(described
[here](https://en.wikipedia.org/wiki/Hilbert%E2%80%93Huang_transform))

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


## `EXE`

_Calculates an epoch-by-epoch permutation distribution entropy similarity matrix_

To be completed.

## `TSLIB`

_Generates a time-series libary to be used with the [`SSS`](#sss)_

To be completed.

## `SSS`

_Simple Sleep Stager_

A basic k-NN (k-nearest neighbor algorithm) for supervised sleep stage classificaiton.

