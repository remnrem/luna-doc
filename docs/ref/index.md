# Command Reference

Below is 1) a
list of command [domains](#domains), and 2) a full list of [all
commands](#all-commands).

The pages that are linked to describe the Luna command language, which can be used
via the terminal-based [_lunaC_](../luna/args.md) tool, the Python
[_lunapi_](../lunapi/index.md) package, or the R extension library
[_lunaR_](../ext/R/index.md). There is also
[_LunaScope_](https://zzz.nyspi.org/lunascope/), an interactive
point-and-click GUI built on top of _lunapi_. Importantly, the Luna
command language (script syntax) is the same across all these
interfaces. Under each section, we tabulate the relevant _commands_,
along with their input _parameters_ and output _variables_. We give
examples using a mixture of [_lunaC_](../luna/args.md) and
[_lunapi_](../lunapi/index.md); also see _lunaC_'s [help
function](../luna/args.md#help).

## Domains

| Domain/Section | Description |
| -----  | ----- |
|[Summaries](summaries.md)         | Basic summary commands |
|[Annotations](annotations.md)     | Adding and displaying annotations |
|[Expressions](evals.md)           | Evaluating more advanced annotation-based expressions |
|[Epochs](epochs.md)               | Epoching signals and epoch-level annotations |
|[Masks](masks.md)                 | Masking epochs based on annotations and other criteria |
|[Freezes](freezes.md)             | Working with dataset freezes (snapshots) | 
|[Canonical signals](canonical.md) | Harmonizing EDFs through _canonical signal_ specification |
|[Manipulations](manipulations.md) | Manipulating signal data |
|[Outputs](outputs.md)             | Commands to output signals in different formats |
|[FIR filters](fir-filters.md)     | FIR filter design and application |
|[Artifacts](artifacts.md)         | Artifacts detection/correction routines |
|[Physiological signal analysis](physio.md) | Heart-rate, atonia, and arousal analyses |
|[Hypnograms](hypnograms.md)       | Characterizations of hypnograms |
|[SOAP](soap.md)                   | Self-contained modelling and evaluation of sleep staging |
|[POPS](pops.md)                   | Sleep staging & evaluation |
|[Time/frequency analyses](power-spectra.md) | Power spectral density estimation and other T/F decompositions |
|[Spindles and SO](spindles-so.md) | Spindles and slow oscillations |
|[Coupling/connectivity](cc.md)    | Phase/amplitude coupling, coherence and other multi-signal analyses |
|[Interval-based](intervals.md)    | Time/event-locked signal averaging, peak detection, enrichment |
|[Spatial/topographical](spatial.md) | Channel locations, spatial filtering and interpolation |
|[PSC](psc.md)                     | Principal spectral components |
|[ICA/PCA](ica.md)                 | Independent components and principal components analyses |
|[MS](ms.md)                       | EEG microstate analysis |
|[Clustering](clustering.md)       | Time-series clustering |
|[Association](assoc.md)           | Association analysis (linear models) |
|[Prediction](predict.md)       | Prediction models | 
|[Simulation](simul.md)            | Simulation of time-series data |
|[Helpers](helpers.md)             | Auxiliary helper commands |
|[Actigraphy](actigraphy.md)       | Multi-day actigraphy: circadian metrics and wake/sleep scoring |
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
[`SIGSTATS`](summaries.md#sigstats): _signal Hjorth parameters,_
[`TABULATE`](summaries.md#tabulate): _tabulate signal values,_
[`DUPES`](summaries.md#dupes): _check for signal duplicates._
__Annotations:__
[Formats](annotations.md#luna-annotations): _overview of annotations,_
[`--xml` & `--xml2`](annotations.md#-xml): _view NSRR XMLs,_
[`REMAP`](annotations.md#remap): _remap annotations,_
[`ANNOTS`](annotations.md#annots): _tabulate annotations,_
[`AXA`](annotations.md#axa): _pairwise annotation cross-tabulation,_
[`MAKE-ANNOTS`](annotations.md#make-annots): _make new annotations,_
[`WRITE-ANNOTS`](annotations.md#write-annots): _write annotation files,_
[`SPANNING`](annotations.md#spanning): _annotation coverage stats,_
[`ESPAN`](annotations.md#espan): _epoch-based annotation coverage,_
[`META`](annotations.md#meta): _add meta-data to annotations,_
[`A2S`](annotations.md#a2s): _make signal from annotation,_
[`S2A`](annotations.md#s2a): _make annotation from signal,_
__Expressions:__
[Expressions](evals.md): _overview of expressions,_
[`EVAL`](evals.md#eval): _annotation-based expressions,_
[`TRANS`](evals.md#trans): _channel-based expressions,_
[`DERIVE`](evals.md#derive): _derive summaries from annotation meta-data._
__Epochs:__
[`EPOCH`](epochs.md#epoch): _specify epochs,_
[`EPOCH-ANNOT`](epochs.md#epoch-annot): _attach epoch annotations._
__Masks:__
[`MASK`](masks.md#mask): _mask epochs,_
[`DUMP-MASK`](masks.md#dump-mask): _output epoch masks,_
[`RESTRUCTURE`](masks.md#restructure): _remove masked epochs,_
[`CHEP`](masks.md#chep): _channel/epoch masks._
__Freezes & caches:__
[`FREEZE`](freezes.md#freeze): _freeze snapshot,_
[`THAW`](freezes.md#thaw): _revive a prior freeze,_
[`CLEAN-FREEZER`](freezes.md#clean-freezer): _empty freezer,_
[`CACHE`](freezes.md#cache): _cache operations._
__Canonical signals:__
[`CANONICAL`](canonical.md#canonical): _make canonical signals._
__Manipulations:__
[`SIGNALS`](manipulations.md#signals): _drop signals,_
[`RENAME`](manipulations.md#rename): _rename signals,_
[`COPY`](manipulations.md#copy): _copy signals,_
[`RESAMPLE`](manipulations.md#resample): _resample signals,_
[`ENFORCE-SR`](manipulations.md#enforce-sr): _check for sufficient sample rate,_ 
[`REFERENCE`](manipulations.md#reference): _re-reference signals,_
[`DEREFERENCE`](manipulations.md#dereference): _de-reference signals,_
[`MINMAX`](manipulations.md#minmax): _set channel min/max,_
[`uV`](manipulations.md#uv): _force microvolts,_
[`mV`](manipulations.md#mv): _force millivolts,_
[`TIME-TRACK`](manipulations.md#time-track): _add time-track,_
[`FLIP`](manipulations.md#flip): _flip signal,_
[`SCALE`](manipulations.md#scale): _scale a signal,_
[`CLIP`](manipulations.md#clip): _clip a signal,_
[`COMBINE`](manipulations.md#combine): _combine multiple signals,_
[`ZC`](manipulations.md#zc): _zero-center signal,_
[`ROBUST-NORM`](manipulations.md#robust-norm): _robust normalization,_
[`RECORD-SIZE`](manipulations.md#record-size): _change record size,_
[`ALIGN`](manipulations.md#align): _align channels/records,_
[`EDF-MINUS`](manipulations.md#edf-minus): _convert EDF+ to EDF,_
[`ANON`](manipulations.md#anon): _anonymize EDF,_
[`SET-HEADERS`](manipulations.md#set-headers): _set EDF headers,_
[`SET-VAR`](manipulations.md#set-var): _set Luna variables,_
[`SET-TIMESTAMPS`](manipulations.md#set-timestamps): _set EDF record time,_
[`RECTIFY`](manipulations.md#rectify): _rectify a signal,_
[`REVERSE`](manipulations.md#reverse): _reverse a signal,_
[`MOVING-AVERAGE`](manipulations.md#moving-average): _moving average filters._
__Outputs:__ 
[`WRITE`](outputs.md#write): _write EDF,_
[`MATRIX`](outputs.md#matrix): _signals to text,_
[`HEAD`](outputs.md#head): _signals snippet to text,_
[`DUMP-RECORDS`](outputs.md#dump-records): _dump by record,_
[`RECS`](outputs.md#recs): _info on EDF record structure,_
[`SEGMENTS`](outputs.md#segments): _continuous intervals,_
[`SEDF`](outputs.md#sedf): _write summary EDF,_
__Filters:__
[`FILTER`](fir-filters.md#filter): _apply FIR,_
[`FILTER-DESIGN`](fir-filters.md#filter-design): _FIR properties._
__Artifacts:__
[`QC`](artifacts.md#qc): _multi-domain PSG signal quality control,_
[`EDGER`](artifacts.md#edger): _identify leading/trailing noise,_
[`CHEP-MASK`](artifacts.md#chep-mask): _CHannel/EPoch masking,_
[`ARTIFACTS`](artifacts.md#artifacts): _bad EEG epochs,_
[`POL`](artifacts.md#pol): _signal polarity diagnostics,_
[`LINE-DENOISE`](artifacts.md#line-denoise): _line denoising,_
[`SUPPRESS-ECG`](artifacts.md#suppress-ecg): _correct ECG artifact,_
[`ALTER`](artifacts.md#alter): _correct artifacts._
__Physiological signal analysis:__
[`HRV`](physio.md#hrv): _estimate heart-rate variability metrics from ECG,_
[`RAI`](physio.md#rai): _calculate the REM atonia index from chin EMG,_
[`AROUSALS`](physio.md#arousals): _detect candidate sleep arousals from EEG and optional EMG._
__Hypnograms:__
[`HYPNO`](hypnograms.md#hypno): _stage summaries,_
[`STAGE`](hypnograms.md#stage): _dump stages,_
[`DYNAM`](hypnograms.md#dynam): _summarize epoch-level outputs by NREM cycles._
__SOAP:__
[`SOAP`](soap.md#soap):	_single observation & probabilities,_
[`REBASE`](soap.md#rebase): _change epoch length,_
[`PLACE`](soap.md#place): _align stages,_
__POPS:__
[`RUN-POPS`](pops.md#run-pops): _predict sleep stages,_
[`POPS`](pops.md#pops-prediction): _lower-level POPS command,_
[`EVAL-STAGES`](pops.md#eval-stages): _evaluate external stages,_
[`--eval-stages`](pops.md#eval-stages): _evaluate external stages,_
[`POPS train`](pops.md#pops-training): _create level 1 training features,_
[`--pops`](pops.md#pops-training): _train models._
__Time/frequency analysis:__
[`PSD`](power-spectra.md#psd): _Welch PSD,_
[`MTM`](power-spectra.md#mtm): _Multi-taper PSD,_
[`FFT`](power-spectra.md#fft): _Fourier transform,_
[`IRASA`](power-spectra.md#irasa): _IRASA spectral analysis,_
[`HILBERT`](power-spectra.md#hilbert): _Hilbert transform,_
[`CWT`](power-spectra.md#cwt): _wavelet transform,_
[`CWT-DESIGN`](power-spectra.md#cwt-design): _CWT properties,_
[`PCOUPL`](power-spectra.md#pcoupl): _generic phase coupling,_
[`EMD`](power-spectra.md#emd): _Empirical mode decomposition,_
[`MSE`](power-spectra.md#mse): _Multi-scale entropy,_
[`LZW`](power-spectra.md#lzw): _LZW compression,_
[`1FNORM`](power-spectra.md#1fnorm): _remove the 1/f trend,_
[`TV`](power-spectra.md#tv): _total variation denoiser,_
[`ACF`](power-spectra.md#acf): _autocorrelation function,_
[`DFA`](power-spectra.md#dfa): _detrended fluctuation analysis._
__Spindles and SO:__
[`SPINDLES`](spindles-so.md#spindles): _spindles,_
[`SO`](spindles-so.md#so): _slow oscillations._
__Coupling/connectivity:__
[`CORREL`](cc.md#correl): _correlation,_
[`XCORR`](cc.md#xcorr): _cross-correlation,_
[`COH`](cc.md#coh): _coherence,_
[`PSI`](cc.md#psi): _phase slope index,_
[`IPC`](cc.md#ipc): _instantaneous phase coherence,_
[`CC`](cc.md#cc): _phase-amplitude coupling & phase lag,_
[`MI`](cc.md#mi): _mutual information,_
[`GP`](cc.md#gp): _Granger prediction._
__Interval-based:__
[`OVERLAP`](intervals.md#overlap): _single-sample overlap analysis,_
[`--overlap`](intervals.md#-overlap): _multi-sample overlap analysis,_ 
[`MEANS`](intervals.md#means): _signal mean by annotation,_
[`PEAKS`](intervals.md#peaks): _detect/cache peaks,_
[`Z-PEAKS`](intervals.md#z-peaks): _detect/cache peaks (Z method),_
[`TLOCK`](intervals.md#tlock): _time-locked averaging,_
[`S2C`](intervals.md#s2c): _signal-to-cycle annotation and summaries._
__PSC:__
[`--psc`](psc.md#-psc): _estimate components,_
[`PSC`](psc.md#psc): _project new samples._
__Spatial/topographical:__
[`CLOCS`](spatial.md#clocs): _set channel locations,_
[`SL`](spatial.md#sl): _surface Laplacian,_
[`INTERPOLATE`](spatial.md#interpolate): _epoch/channel interpolation._
__ICA:__
[`ICA`](ica.md#ica): _fit ICA,_
[`ADJUST`](ica.md#adjust): _adjust given ICs,_
[`SVD`](ica.md#svd): _fit SVD/PCA._
__Microstates:__
[`MS`](ms.md#ms): _EEG microstates,_
[`--kmer`](ms.md#-kmer): _sequence motifs,_
[`--cmp-maps`](ms.md#-cmp-maps): _group/indiv map spatial analysis,_
[`--label-maps`](ms.md#-label-maps): _label maps given a template,_
[`--correl-maps`](ms.md#-correl-maps): _spatial correlations._
__Clustering:__
[`EXE`](clustering.md#exe): _time-series clustering._
__Association:__
[`--gpa-prep`](assoc.md#gpa): _general permutation-based association model prep,_
[`--gpa`](assoc.md#gpa): _general permutation-based association models,_
[`CPT`](assoc.md#cpt): _association models._ 
__Prediction:__
[`PREDICT`](predict.md#predict): _prediction models._
__Simulation:__
[`SIMUL`](simul.md#simul): _simulate signals,_
[`SIGGEN`](simul.md#siggen): _basic signal simulation._
__Actigraphy:__
[`DAYS`](actigraphy.md#days): _day and clock-time annotations,_
[`ACTIG`](actigraphy.md#actig): _circadian metrics and wake/sleep scoring._
__Helpers:__
[`--build`](helpers.md#-build): _build sample-lists,_
[`--validate`](helpers.md#-validate): _validate files,_
[`--repath`](helpers.md#-repath): _alter sample-lists,_
[`--merge`](helpers.md#-merge): _merge EDFs,_
[`--bind`](helpers.md#-bind): _bind files to IDs,_
[`--xml`](helpers.md#-xml): _view XMLs,_
[`--xml2`](helpers.md#-xml2): _dump EDFs (raw),_
[`--otsu`](helpers.md#-otsu): _Otsu thresholding (file),_
[`OTSU`](helpers.md#otsu): _Otsu thresholding (EDF)._
__Experimental:__
[Various](exp.md): _misc. experimental commands,_
[`ALIGN-EPOCHS`](exp.md#align-epochs): _align epochs between files,_
[`ALIGN-ANNOTS`](exp.md#align-annots): _realign annotations given an ALIGN-EPOCHS solution,_
[`INSERT`](exp.md#insert): _estimate lags and insert channels from another EDF._


<!--

__Notes/to add: __

# to add/remove?
FILE-MASK
ANNOT-MASK?
HB?
SYNC?
COVAR?
ASYMM
PREP-MASSOC
PERI
SEDF
PAC
CFC
GED
ZOH 
L1OUT
FIP

--->
