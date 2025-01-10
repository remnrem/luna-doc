# Misc

_Various commands and digital signal processing tools features not mentioned elsewhere_

| Command | Description | 
| ---- | ------ | 
| [`HR`](#hr)                 | Estimate per-epoch heart rate from ECG |
| [`SPIKE`](#spike)           | Create a synthetic signal by combining part of one signal with another |
| [`ZR`](#zr)   |  Calculate per-epoch Z-ratio |

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


<h3>Parametes</h3>

| Option | Description | 
| ---- | ---- | 
| `signal` | Specify which channels/signals to include |

<h3>Outputs</h3>

_to be completed_

<h3>Example</h3>

_to be completed_
