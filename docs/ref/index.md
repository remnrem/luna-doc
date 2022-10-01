# Command Reference

Below is 1) a
list of command [domains](#domains), and 2) a full list of [all
commands](#all-commands).

The pages that are linked to describe the Luna command language, which can be used
either via the terminal-based [_lunaC_](../luna/args.md) tool or the R
extension library [_lunaR_](../ext/R/index.md).  Under each section,
we tabulate the relevant _commands_, along with their input
_parameters_ and output _variables_.  We give examples using a mixture
of [_lunaC_](../luna/args.md) and [_lunaR_](../ext/R/index.md); also
see _lunaC_'s [help function](../luna/args.md#help).


## Domains

| Domain/Section | Description |
| -----  | ----- | 
|[Summaries](summaries.md)         | Basic summary commands | 
|[Annotations](annotations.md)     | Adding and displaying annotations |
|[Expressions](evals.md)           | Evaluating more advanced annotation-based expressions |
|[Epochs](epochs.md)               | Epoching signals and epoch-level annotations |
|[Masks](masks.md)                 | Masking epochs based on annotations and other criteria |
|[Canonical signals](canonical.md) | Harmonizing EDFs through _canonical signal_ specification |
|[Manipulations](manipulations.md) | Manipulating signal data |
|[Outputs](outputs.md)             | Commands to output signals in different formats |
|[FIR filters](fir-filters.md)     | FIR filter design and application |
|[Artifacts](artifacts.md)         | Artifacts detection/correction routines |
|[Hypnograms](hypnograms.md)       | Characterizations of hypnograms |
|[POPS](pops.md)                   | Sleep staging & evaluation |
|[Time/frequency analyses](power-spectra.md) | Power spectral density estimation and other T/F decompositions |
|[Spindles and SO](spindles-so.md) | Spindles and slow oscillations |
|[Coupling/connectivity](cc.md)    | Phase/amplitude coupling, coherence and other multi-signal analyses |
|[Interval-based](intervals.md)    | Time/event-locked signal averaging, peak detection, enrichment |
|[Spatial/topographical](spatial.md) | Channel locations, spatial filtering and interpolation |
|[ICA](ica.md)                     | Independent components analysis |
|[MS](ms.md)                       | EEG microstate analysis |
|[Clustering](clustering.md)       | Time-series clustering |
|[Association](assoc.md)           | Association analysis (linear models) |
|[Simulation](simul.md)            | Simulation of time-series data |
|[Helpers](helpers.md)             | Auxiliary helper commands |
|[Experimental](exp.md)            | Experimental features, under heavy development / for internal use only |

## All commands 

__Summaries:__
[`DESC`](summaries.md#desc): _simple EDF description,_
[`SUMMARY`](summaries.md#summary): _EDF description,_
[`HEADERS`](summaries.md#headers): _EDF header information;_
[`CONTAINS`](summaries.md#contains): _Indicate whether signals present,_
[`ALIASES`](summaries.md#aliases): _show channel/annotation aliases,_
[`TYPES`](summaries.md#types): _show channel types,_
[`VARS`](summaries.md#vars): _show individual-level variables,_
[`TAG`](summaries.md#tag): _add output tags,_
[`STATS`](summaries.md#stats): _basic signal statistics,_
[`SIGSTATS`](summaries.md#sigstats): _signal Hjorth parameters._
__Annotations:__
[Formats](annotations.md#luna-annotations): _overview of annotations,_
[`--xml` & `--xml2`](annotations.md#-xml): _view NSRR XMLs,_
[`ANNOTS`](annotations.md#annots): _tabulate annotations,_
[`WRITE-ANNOTS`](annotations.md#write-annots): _write annotation files,_
[`SPANNING`](annotations.md#spanning): _annotation coverage stats,_
[`A2S`](annotations.md#a2s): _make signal from annotation,_
[`S2A`](annotations.md#s2a): _make annotation from signal._ 
__Expressions:__
[Expressions](epochs.md#eval-expressions): _overview of expressions,_
[`EVAL`](epochs.md#eval): _annotation-based expressions,_
[`TRANS`](epochs.md#trans): _channel-based expressions._
__Epochs:__
[`EPOCH`](epochs.md#epoch): _specify epochs,_
[`EPOCH-ANNOT`](epochs.md#epoch-annot): _attach epoch annotations._
__Masks:__
[`MASK`](masks.md#mask): _mask epochs,_
[`DUMP-MASK`](masks.md#dump-mask): _output epoch masks,_
[`RESTRUCTURE`](masks.md#restructure): _remove masked epochs,_
[`CHEP`](masks.md#chep): _channel/epoch masks._
__Manipulations:__
[`SIGNALS`](manipulations.md#signals): _drop signals,_
[`COPY`](manipulations.md#copy): _copy signals,_
[`RESAMPLE`](manipulations.md#resample): _resample signals,_
[`ENFORCE-SR`](manipulations.md#enforce-sr): _check for sufficient sample rate,_ 
[`REFERENCE`](manipulations.md#reference): _re-reference signals,_
[`MINMAX`](manipulations.md#minmax): _set channel min/max,_
[`EDF`](manipulations.md#edf): _force basic EDF,_
[`TIME-TRACK`](manipulations.md#time-track): _add time-track,_
[`RECORD-SIZE`](manipulations.md#record-size): _change record size,_
[`ALIGN`](manipulations.md#align): _realign records,_
[`ANON`](manipulations.md#anon): _anonymize EDF,_
[`uV`](manipulations.md#uv): _force microvolts,_
[`mV`](manipulations.md#mv): _force millivolts,_
[`FLIP`](manipulations.md#flip): _flip signal,_
[`ZC`](manipulations.md#zc): _zero-center signal,_
[`ROBUST-NORM`](manipulations.md#robust-norm): _robust normalization,_
[`SET-HEADERS`](manipulations.md#set-headers): _set EDF headers,_
[`SET-VAR`](manipulations.md#set-var): _set Luna variables._
[`RECTIFY`](manipulations.md#rectify): _rectify a signal,_
[`REVERSE`](manipulations.md#reverse): _reverse a signal._
__Canonical signals:__
[`CANONICAL`](manipulations.md#canonical): _make canonical signals._
__Outputs:__ 
[`WRITE`](outputs.md#write): _write EDF,_
[`MATRIX`](outputs.md#matrix): _signals to text,_
[`HEAD`](outputs.md#head): _signals snippet to text,_
[`DUMP-RECORDS`](outputs.md#dump-records): _dump by record,_
[`RECS`](outputs.md#recs): _info on EDF record structure,_
[`SEGMENTS`](outputs.md#segments): _continuous intervals,_
[`SEDF`](outputs.md#sedf): _write summary EDF._
__Filters:__
[`FILTER`](fir-filters.md#filter): _apply FIR,_
[`FILTER-DESIGN`](fir-filters.md#filter-design): _FIR properties._
__Artifacts:__
[`CHEP-MASK`](artifacts.md#chep-mask): _CHannel/EPoch masking,_
[`ARTIFACTS`](artifacts.md#artifacts): _bad EEG epochs,_
[`LINE-DENOISE`](artifacts.md#line-denoise): _line denoising,_
[`SUPPRESS-ECG`](artifacts.md#suppress-ecg): _correct ECG artifact,_
[`ALTER`](artifacts.md#alter): _correct artifacts._
__Hypnograms:__
[`STAGE`](hypnograms.md#stage): _dump stages,_
[`HYPNO`](hypnograms.md#hypno): _stage summaries._
__SOAP:__
[`SOAP`](soap.md#soap):	_single observation model,_
[`RESOAP`](soap.md#resoap): _iterative SOAP,_
[`REBASE`](soap.md#rebase): _change epoch length,_
[`PLACE`](soap.md#place): _align stages,_
__POPS:__
[`POPS`](pops.md#pops): _predict sleep stages,_
[`--pops`](pops.md#-pops): _train model,_
[`--priors`](pops.md#-priors): _derive stage priors,_
[`EVAL-STAGES`](pops.md#eval-stages): _evaluate external stages,_
[`--eval-stages`](pops.md#eval-stages): _evaluate external stages._
__Time/frequency analysis:__
[`PSD`](power-spectra.md#psd): _Welch PSD,_
[`MTM`](power-spectra.md#mtm): _Multi-taper PSD,_
[`FFT`](power-spectra.md#fft): _Fourier transform,_
[`HILBERT`](power-spectra.md#hilbert): _Hilbert transform,_
[`CWT`](power-spectra.md#cwt): _wavelet transform,_
[`CWT-DESIGN`](power-spectra.md#cwt-design): _CWT properties,_
[`EMD`](power-spectra.md#emd): _Empirical mode decompositon,_
[`MSE`](power-spectra.md#mse): _Multi-scale entropy,_
[`LZW`](power-spectra.md#lzw): _LZW compression,_
[`1FNORM`](power-spectra.md#1fnorm): _remove the 1/f trend,_
[`TV`](power-spectra.md#tv): _total variation denoiser,_
[`ACF`](power-spectra.md#acf): _autocorrelation function_
__Spindles and SO:__
[`SPINDLES`](spindles-so.md#spindles): _spindles,_
[`SO`](spindles-so.md#so): _slow oscillations._
__Coupling/connectivity:__
[`COH`](cc.md#coh): _coherence,_
[`CORREL`](cc.md#correl): _correlation,_
[`CC`](cc.md#cc): _phase-amplitude coupling & phase lag,_
[`PSI`](cc.md#): _phase slope index,_
[`MI`](cc.md#mi): _mutual information,_
[`TSYNC`](cc.md): _cross-correlation & phase delay,_
[`GP`](cc.md#gp): _Granger prediction._
__Interval-based:__
[`OVERLAP`](intervals.md#overlap): _single-sample overlap analysis,_
[`--overlap`](intervals.md#-overlap): _multi-sample overlap analysis,_ 
[`MEANS`](intervals.md#means): _signal mean by annotation,_
[`PEAKS`](intervals.md#peaks): _detect/cache peaks,_
[`TLOCK`](intervals.md#tlock): _time-locked averaging._
__Spatial/topographical:__
[`CLOCS`](spatial.md#clocs): _set channel locations,_
[`SL`](spatial.md#sl): _surface Laplacian,_
[`INTERPOLATE`](spatial.md#interpolate): _epoch/channel interpolation._
__PSC:__
[`--psc`](psc.md#-psc): _estimate components,_
[`PSC`](psc.md#psc): _project new samples._
__ICA:__
[](ica.md)
[`ICA`](ica.md#ica): _fit ICA,_
[`ADJUST`](ica.md#adjust): _adjust given ICs._
__Microstates:__
[](ms.md)
[`MS`](ms.md#ms): _EEG microstates,_
[`--kmer`](ms.md#-kmer): _sequence motifs,_
[`--cmp-maps`](ms.md#-cmp-maps): _group/indiv map spatial analysis,_
[`--label-maps`](ms.md#-label-maps): _label maps given a template,_
[`--correl-maps`](ms.md#-correl-maps): _spatial correlations._
__Clustering:__
[`EXE`](clustering.md): _time-series clustering._
__Association:__
[`CPT`](assoc.md#cpt): _association models._ 
__Simulation:__
[`SIMUL`](simul.md#simul): _simulate signals._
__Helpers:__
[`--build`](helpers.md#-build): _build sample-lists,_
[`--repath`](helpers.md#-repath): _alter sample-lists,_
[`--merge`](helpers.md#-merge): _merge EDFs,_
[`--otsu`](helpers.md#-otsu): _Otsu thresholding (file),_
[`OTSU`](helpers.md#otsu): _Otsu thresholding (EDF)._

__Experimental:__
[Various](exp.md): _misc. experimental commands._
