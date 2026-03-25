# Alignment

_Utilities to align signal (and annotation) data across different EDFs_

These commands are currently documented under the [Experimental](exp.md)
section rather than as a standard public command domain:

| Command | Description |
|---|---|
| [`ALIGN-EPOCHS`](exp.md#align-epochs) | Align epochs between files |
| [`ALIGN-ANNOTS`](exp.md#align-annots) | Realign annotations given an `ALIGN-EPOCHS` solution |
| [`INSERT`](exp.md#insert) | Estimate lags and insert channels from another EDF |

They are implemented and callable, but are intentionally not exposed through the
normal `cmddefs` help system and should be treated as experimental/unsupported.
