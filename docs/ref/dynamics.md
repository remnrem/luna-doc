# Dynamics

_Commands to characterize ultradian and temporal dynamics of sleep signals and events_

These commands characterize temporal dynamics in sleep signals and events: [`EPDYN`](dynamics.md#epdyn) summarizes epoch-level metrics across NREM cycles, [`EVTDYN`](dynamics.md#evtdyn) analyzes annotation-event timing and clustering, [`SIGDYN`](dynamics.md#sigdyn) summarizes any existing signal using stage/cycle statistics, whole-recording trends, or peri-event averages, and [`DPP`](dynamics.md#dpp) computes multiscale local features over trailing causal windows and (via the cohort-level `--dpp-fit`) trains a model projecting a person-level phenotype back onto the recording as a time-varying signal — which `SIGDYN` can then summarize like any other.

| Command | Description |
|---|---|
| [`EPDYN`](#epdyn) | Summarize epoch-level outputs by NREM cycles |
| [`EVTDYN`](#evtdyn) | Summarize temporal dynamics of annotation events |
| [`SIGDYN`](#sigdyn) | Summarize the temporal dynamics of a signal |
| [`DPP`](#dpp) | Multiscale local features, and (`--dpp-fit`) dynamic phenotype projection |

## EPDYN

_Summarize epoch-level outputs in terms of hypnogram-derived NREM cycle dynamics_

This command will typically be invoked by adding `dynam` as an option to one of the following commands that currently supports it: [`PSD`](power-spectra.md#psd), [`COH`](cc.md#coh), [`SPINDLES`](spindles-so.md#spindles), [`SO`](spindles-so.md#so), [`PSI`](cc.md#psi), [`CORREL`](cc.md#correl). Alternatively, epoch-level inputs can be specified from a file, in which case this functionality is directly invoked via the `EPDYN` command. The same functions are executed in either case.

This command requires that [`HYPNO`](hypnograms.md#hypno) has previously been run, as it relies on knowing the NREM cycle structure of a recording.

<h3>Methods</h3>

Epoch-level metrics are gathered for each variable across the night and, separately, within each NREM sleep cycle as identified by the Feinberg–Floyd heuristic applied during a prior [`HYPNO`](hypnograms.md#hypno) run. Within each analysis window (total night, between cycles, average within-cycle, and per-cycle), the epoch-level series is smoothed via a sequential median then mean filter to reduce epoch-to-epoch noise, then optionally normalized (by default, mean-centering to a positive scale). Linear trend is summarized by the Pearson correlation of the smoothed, normalized series with epoch time (`U`), capturing monotonic increases or decreases over the night. A quadratic (non-linear) term (`U2`) is estimated from a linear model regressing the normalized series on the squared deviation of epoch time from its mean (while including the linear epoch-time term as a covariate), capturing U-shaped or inverted-U dynamics. Peak-to-peak amplitude and timing statistics (`A_P2P`, `T_P2P`) characterize the range and position of the dominant oscillation in the smoothed series. Quantile traces bin the smoothed series into _N_ equal-time segments to form a compact representation of the temporal profile, output at the `QD` × `Q` strata level.

<h3>Parameters</h3>

When invoked as a sub-option of another command, parameters are specified alongside that command (e.g. `PSD sig=EEG dynam dynam-nq=5`). When invoked directly as a standalone `EPDYN` command, the same parameter names apply.

| Parameter | Example | Description |
| ---- | ---- | ---- |
| `dynam-verbose` | | Enable verbose output (additional statistics) |
| `dynam-epoch` | | Output smoothed/normalized series per epoch |
| `dynam-use-ranks` | `dynam-use-ranks=T` | Use epoch rank (not clock-time) as the time axis (default `F`) |
| `dynam-nq` | `dynam-nq=5` | Number of quantile bins for `Q` trace output (default `10`, range 2–100) |
| `dynam-min-ne` | `dynam-min-ne=5` | Minimum epochs required to include a cycle in within-cycle analysis (default `10`) |
| `dynam-trim-epochs` | `dynam-trim-epochs=2` | Trim _N_ epochs from each end of within-cycle windows (or `X,Y` to trim differently at start vs. end) |
| `dynam-winsor` | `dynam-winsor=0.05` | Winsorization proportion before smoothing (default `0.05`; set to `0` to disable) |
| `dynam-median-window` | `dynam-median-window=19` | Median filter window (epochs; default `19`, ~10 mins) |
| `dynam-mean-window` | `dynam-mean-window=9` | Mean filter window applied after median filter (epochs; default `9`) |
| `dynam-norm-mean` | `dynam-norm-mean=T` | Normalize smoothed series by mean (default `T`) |
| `dynam-norm-max` | `dynam-norm-max=F` | Normalize smoothed series so maximum equals 1 (default `F`; mutually exclusive with `dynam-norm-mean`) |
| `dynam-norm-cycles` | `dynam-norm-cycles=T` | Apply normalization independently within each cycle (default `T`) |
| `dynam-max-cycle` | `dynam-max-cycle=4` | Only include up to cycle _N_ (maximum 8) |
| `dynam-cycles` | `dynam-cycles=1,2,3` | Only include specified cycle numbers |
| `dynam-weight-cycles` | `dynam-weight-cycles=T` | Weight each cycle by epoch count when averaging within-cycle results (default `T`) |

<h3>Outputs</h3>

Outputs are organized by the `QD` stratum, which indicates the temporal scope:

| `QD` level | Description |
| ---- | ---- |
| `TOT` | Whole-night series (all included epochs) |
| `BETWEEN` | Between-cycle series (one value per cycle; only if `dynam-norm-cycles=F`) |
| `WITHIN` | Weighted average of within-cycle statistics across cycles |
| `W_C1` … `W_C8` | Per-cycle within-cycle statistics |

Summary statistics (strata: `VAR` × `QD`)

| Variable | Description |
| ---- | ---- |
| `N` | Number of epochs (or cycles, for `WITHIN`) |
| `OMEAN` | Mean of the original (unscaled, but Winsorized) series |
| `MEAN` | Mean of the smoothed and normalized series |
| `SD` | Standard deviation of the smoothed and normalized series |
| `U` | Pearson correlation of the smoothed series with time (linear trend; positive = increasing) |
| `U2` | Coefficient of quadratic time term (positive = U-shaped; negative = inverted-U) |
| `T_P2P` | Epoch distance from the local minimum to the local maximum of the smoothed series |
| `A_P2P` | Amplitude range: smoothed series maximum minus minimum |

Additional verbose outputs (option: `dynam-verbose`; strata: `VAR` × `QD`)

| Variable | Description |
| ---- | ---- |
| `UT` | Weighted clock-time statistic (−100 to +100; positive means signal mass is in later epochs) |
| `CV` | Coefficient of variation of the smoothed series |
| `AT_P2P` | Rate statistic: `A_P2P` / `T_P2P` |
| `T_MX` | Epoch offset from start to the series maximum |
| `A_MX` | Amplitude at the maximum relative to the first epoch |
| `AT_MX` | Rate to maximum: `A_MX` / `T_MX` |
| `T_MN` | Epoch offset from start to the series minimum |
| `A_MN` | Amplitude at the minimum relative to the first epoch |
| `AT_MN` | Rate to minimum: `A_MN` / `T_MN` |

Quantile trace (strata: `VAR` × `QD` × `Q`)

| Variable | Description |
| ---- | ---- |
| `SS` | Mean of the smoothed and normalized series within quantile bin `Q` |
| `OS` | Mean of the original smoothed (unnormalized) series within quantile bin `Q` |

Epoch-level output (option: `dynam-epoch`; strata: `VAR` × `QD` × `E`)

| Variable | Description |
| ---- | ---- |
| `SS` | Smoothed and normalized value for this epoch |

---

## EVTDYN

_Characterize temporal and stage-related dynamics of annotation-defined events_

`EVTDYN` operates on discrete annotation events (e.g. detected sleep spindles or slow oscillations) and quantifies how they are distributed across the night. It computes event density within a background interval (by default, all NREM sleep), temporal position statistics, inter-event intervals, clustering metrics, and cycle-level density slopes. For events that carry meta-data values (e.g. spindle amplitude or duration), it also summarizes those values and their temporal trends. It can be invoked directly via the `EVTDYN` command, or as a sub-module within commands such as [`SPINDLES`](spindles-so.md#spindles) and [`SO`](spindles-so.md#so).

This command requires that [`HYPNO`](hypnograms.md#hypno) has previously been run if hypnogram-based background selection or cycle-level analysis is requested.

<h3>Methods</h3>

Events are filtered to those whose anchor point (by default, the midpoint) falls within the defined background intervals. Inter-event intervals (ISI) are computed between consecutive events within the same contiguous background segment. The temporal position of each event is expressed as the fraction of elapsed background time (0–1), and summary statistics (median position `TB`, time-in-sleep-period `TA`, and time-in-sleep-intervals `TS`) quantify whether events are biased toward the early or late night. Clustering is assessed by whether any pair of events falls within a user-defined window (`cluster`); trains are sequences of ≥ `train-min` events each separated by no more than `train-gap` seconds. Autocorrelogram peak statistics (`AC_PEAK_T`, `AC_PEAK_H`) identify the dominant inter-event periodicity. Stage-specific density ratios contrast event rates in N2 vs. N3 and in ascending vs. descending N2 (as defined by [`HYPNO`](hypnograms.md#hypno)'s `annot` option). If NREM cycles are available, `CYCLE_SLOPE` quantifies the linear change in event density across successive cycles. For events carrying numeric meta-data, the same temporal and stage contrasts are applied to variable values (mean in early vs. late night, N2 vs. N3, and across cycles). Inter-variable Pearson correlations across shared events can be requested via the `corr`, `corr1`, and `corr2` parameters.

<h3>Parameters</h3>

| Parameter | Example | Description |
| ---- | ---- | ---- |
| `annot` | `annot=spindles` | Annotation class(es) to analyze (required for standalone `EVTDYN`) |
| `bg` | `bg=N2,N3` | Background annotation(s) to use as reference interval (default: NREM sleep stages) |
| `bg-none` | | Use all epoched intervals as background (ignores stage information) |
| `hypno` | `hypno=F` | Use NREM sleep stages as background if `bg` not set (default `T`) |
| `vars` | `vars=AMP,DUR` | Annotation meta-data variable(s) to analyze (default: all) |
| `corr` | `corr=AMP,DUR` | Compute all pairwise Pearson correlations among these variables across shared events |
| `corr1` | `corr1=AMP` | First variable set for directed cross-correlations with `corr2` |
| `corr2` | `corr2=DUR,ISA` | Second variable set for directed cross-correlations with `corr1` |
| `anchor` | `anchor=MID` | Point within each event used as its time position: `START`, `MID` (default), or `STOP` |
| `cluster` | `cluster=10` | Window (seconds) to define clustered event pairs (default `10`) |
| `train-gap` | `train-gap=10` | Maximum gap (seconds) between consecutive events in a train (default `10`) |
| `train-min` | `train-min=3` | Minimum events required to form a train (default `3`) |
| `short-lag` | `short-lag=3,6` | Lower and upper bounds (seconds) for the short ISI window (default `3,6`) |
| `refractory` | `refractory=0,2` | Lower and upper bounds (seconds) for the refractory period (default `0,2`) |
| `excitatory` | `excitatory=3,8` | Lower and upper bounds (seconds) for the excitatory period (default `3,8`) |
| `ac-max` | `ac-max=60` | Maximum lag (seconds) for autocorrelogram (default `60`) |
| `ac-bin` | `ac-bin=1` | Bin size (seconds) for autocorrelogram (default `1`) |
| `winsor` | `winsor=0.05` | Winsorize event-level meta-data values at this proportion (default `0`) |
| `z` | | Z-score event-level meta-data values |
| `log` | | Log-transform event-level meta-data values |
| `rank` | | Rank-transform event-level meta-data values |
| `verbose` | | Enable verbose output |

<h3>Outputs</h3>

Base outputs (strata: `DYN` × `ANNOT` [× `CH`])

| Variable | Description |
| ---- | ---- |
| `N` | Number of events within the background |
| `MINS` | Total background duration (minutes) |
| `DENS` | Event density (events per minute of background) |
| `TB` | Median time-in-background: fraction of background elapsed at each event (0–1) |
| `TA` | Median time within the sleep period (onset to final wake; 0–1) |
| `TS` | Median time within sleep intervals only (excluding WASO; 0–1) |
| `EARLY_LATE_R` | Log₂ ratio of events in the first vs. second half of the background |
| `ISI_MD` | Median inter-event interval (seconds) between consecutive within-segment events |
| `P_LAG_SHORT` | Proportion of consecutive event pairs with ISI within the `short-lag` window |
| `REFR_OBS_EXP` | Log₂ ratio of observed to expected event pairs in the refractory window |
| `EXCIT_OBS_EXP` | Log₂ ratio of observed to expected event pairs in the excitatory window |
| `AC_PEAK_T` | Lag (seconds) at the peak of the inter-event autocorrelogram |
| `AC_PEAK_H` | Log₂ ratio of autocorrelogram peak count to Poisson expectation |
| `CLST_FRAC` | Fraction of events classified as clustered (within `cluster` seconds of another) |
| `SOL_FRAC` | Fraction of events classified as solitaire (not within `cluster` seconds of another) |
| `TRAIN_FRAC` | Fraction of events belonging to a train |
| `TRAIN_LEN_MN` | Mean train length (events per train) |
| `TRAIN_DENS` | Train density (trains per minute of background) |
| `N2_N3_R` | Log₂ ratio of event density in N2 vs. N3 (requires hypnogram) |
| `N2_ASC_DSC_DIFF` | Difference in event density: ascending N2 minus descending N2 |
| `N2_ASC_DSC_R` | Log₂ ratio of event density in ascending vs. descending N2 |
| `CYCLE_SLOPE` | Linear slope of event density across NREM cycles (events/min per cycle, capped at cycle 4) |

Per-variable outputs (strata: `DYN` × `ANNOT` [× `CH`] × `VAR`)

| Variable | Description |
| ---- | ---- |
| `MEAN` | Mean of the (optionally transformed) meta-data variable across events |
| `MEDIAN` | Median of the meta-data variable |
| `SD` | Standard deviation of the meta-data variable |
| `TIME_R` | Pearson correlation of the meta-data variable with time-in-background |
| `TIME_BETA` | Linear slope of the meta-data variable with time-in-background |
| `EARLY_LATE_R` | Log₂ ratio of mean variable value: early half vs. late half of the background |
| `N2_N3_R` | Log₂ ratio of mean variable value: N2 events vs. N3 events (requires hypnogram) |
| `CYCLE_SLOPE` | Linear slope of mean variable value across NREM cycles (capped at cycle 4) |
| `CLST_SOL_DIFF` | Mean difference of the meta-data variable: clustered minus solitaire events |
| `N2_ASC_DSC_DIFF` | Mean difference of the meta-data variable: ascending minus descending N2 events |
| `N2_ASC_DSC_R` | Log₂ ratio of the meta-data variable mean: ascending vs. descending N2 events |

Cross-variable correlation outputs (strata: `DYN` × `ANNOT` [× `CH`] × `VAR1` × `VAR2`; requires `corr`, `corr1`, or `corr2`)

| Variable | Description |
| ---- | ---- |
| `CORR` | Pearson correlation between the two variables across events that have both values |
| `N` | Number of events with both variables present |

---

## SIGDYN

_Summarize the temporal dynamics of a signal_

`SIGDYN` summarizes the temporal dynamics of any existing epoched or continuous signal already present in the EDF (e.g. SpO2, heart rate, an EEG power channel, a POPS posterior-probability trace) in a way that is comparable across individuals. It combines three independent, optional views: simple stage/cycle-stratified descriptive statistics, a whole-recording trend/decile/NREM-cycle summary (via the same engine as [`EPDYN`](#epdyn)), and annotation- or hypnogram-anchored peri-event averaging (a superset of [`MEANS`](intervals.md#means)'s `M`/`S`/`L`/`R` summary). If the recording is not already epoched, `SIGDYN` epochs it first using Luna's default epoch length (equivalent to running [`EPOCH`](epochs.md#epoch)), which the first two views require.

<h3>Methods</h3>

The descriptive-statistics view reduces each epoch to its signal mean and tabulates `N`/`MEAN`/`SD`/`MIN`/`MAX`/`RANGE`, overall and stratified by sleep stage, by stage × local stage stability (`REGION`: `STABLE`/`TRANS`, following the same convention as [`HDSTATS`](pops.md#hdstats)), and by NREM cycle (if [`HYPNO`](hypnograms.md#hypno) or [`STAGE`](hypnograms.md#stage) has previously compiled per-epoch stage/cycle annotations). An epoch is `STABLE` if the `stable-flank` epochs on both sides share its stage; a missing or differently-staged neighbor makes it `TRANS`. Disable this view with `epoch-stats=F`.

The trend/decile/cycle view feeds the per-epoch signal mean into the same engine used by [`EPDYN`](#epdyn) (e.g. as embedded in [`PSD`](power-spectra.md#psd)'s `dynam` option) — all `dynam-*` parameters documented there apply here too. It runs automatically whenever the recording is epoched, and cycle-level output additionally requires a prior [`HYPNO`](hypnograms.md#hypno) run.

The peri-event view performs windowed averaging around one or more anchor annotation classes. Anchors are taken from any class(es) named in `annot=`, and/or auto-discovered from [`HYPNO`](hypnograms.md#hypno)'s richer landmark/cycle/transition annotation set (`t0_start` .. `t6_stop`, `cycle_n<N>`, `tr_<stage1>_<stage2>`, etc., written by a prior `HYPNO annot=` run) when `hypno-annot=T` (the default). For each anchor instance, `anchor=` picks where within that instance's own interval the `t=0` time point sits — `start`, `middle`, or `end` (irrelevant for a 0-duration/point instance, where all three coincide). A `+/-w` second window around each anchor is divided into `SEC` offset bins: with `bin=` unset, each bin is a single sample (native resolution); with `bin=`/`inc=` set, bins are `bin` seconds wide and stepped by `inc` seconds (`inc < bin` gives overlapping/sliding bins, `inc > bin` leaves gaps between bins, and the default `inc = bin` gives non-overlapping, tiled bins). `SEC` offsets are always exactly `k * inc` (i.e. `.., -2*inc, -inc, 0, inc, 2*inc, ..`), symmetric about the anchor; `bin-align=` picks which part of each bin — its `start`, `middle`, or `end` — sits at that labeled offset.

An anchor instance contributes to a given `SEC` bin only if it has complete, gap-free coverage of that bin's entire sample range: partial or thinned bins are never formed. An instance near the start or end of the recording, or close to a genuine discontinuity (e.g. after [`MASK`](masks.md#mask) + [`RE`](masks.md#restructure)), simply forms fewer bins rather than being dropped outright, and the `CH,ANNOT` summary row is still built from whichever bins and instances are complete. `require-full=T` instead reverts to requiring an instance's entire nominal window to be available and gap-free, rejecting the whole instance otherwise — a stricter, all-or-nothing-per-instance policy. `min-n=` drops any `SEC` bin, or `CH,ANNOT` summary row, with fewer than this many contributing instances. Before averaging, `tolog` log-transforms the signal, and peri-event outlier control can be applied per bin: `th=` drops values beyond that many SDs of the bin mean, or, if `th` is not set, `win=` instead Winsorizes them at that many SDs.

<h3>Parameters</h3>

| Parameter | Example | Description |
| ---- | ---- | ---- |
| `sig` | `sig=SpO2` | Signal(s) to summarize (required) |
| `epoch-stats` | `epoch-stats=F` | Disable the stage/cycle/stability descriptive-statistics view (default `T`) |
| `stable-flank` | `stable-flank=2` | Number of flanking epochs on each side that must share the same stage for `REGION=STABLE` (default `1`) |
| `annot` | `annot=arousal,resp_event` | Annotation class(es) to use as peri-event anchors |
| `hypno-annot` | `hypno-annot=F` | Disable auto-discovery of `HYPNO`-derived landmark/cycle/transition anchors (default `T`) |
| `w` | `w=30` | Half-window size in seconds around each anchor (default `60`) |
| `anchor` | `anchor=end` | Where in an annotation instance's interval the `t=0` anchor sits: `start`, `middle`, or `end` (default `start`) |
| `bin` | `bin=10` | Bin width in seconds for `SEC` offsets (default: native per-sample resolution) |
| `inc` | `inc=5` | Step between bins in seconds; requires `bin=`; `< bin` overlaps, `> bin` leaves gaps (default: `= bin`) |
| `bin-align` | `bin-align=end` | Which part of each bin sits at its labeled offset: `start`, `middle`, or `end` (default `middle`) |
| `require-full` | `require-full=T` | Require an instance's entire window to be available and gap-free, rather than letting it form fewer bins (default `F`) |
| `min-n` | `min-n=5` | Minimum contributing instances required to emit a `SEC` bin or `CH,ANNOT` summary row (default `1`) |
| `tolog` | | Log-transform the signal before peri-event averaging |
| `th` | `th=4` | Drop peri-event values beyond this many SDs of the bin mean (default: disabled) |
| `win` | `win=4` | Winsorize peri-event values beyond this many SDs of the bin mean (default: disabled; ignored if `th` is set) |

<h3>Outputs</h3>

Descriptive statistics (strata: `CH`, and, where staging and/or NREM cycles are available, also `CH` × `SS`, `CH` × `SS` × `REGION`, `CH` × `C`, and `CH` × `SS` × `C`)

| Variable | Description |
| ---- | ---- |
| `N` | Number of epochs |
| `MEAN` | Mean of per-epoch signal means |
| `SD` | SD of per-epoch signal means |
| `MIN` | Minimum of per-epoch signal means |
| `MAX` | Maximum of per-epoch signal means |
| `RANGE` | `MAX` minus `MIN` |

Whole-recording trend/decile/cycle summary (strata: `CH` × `VAR` × `QD` [× `Q`]) — see [`EPDYN`](#epdyn) for the full set of output variables (`N`, `OMEAN`, `MEAN`, `SD`, `U`, `U2`, `T_P2P`, `A_P2P`, and, at the `Q` level, `SS`/`OS`)

Peri-event summary (strata: `CH` × `ANNOT`)

| Variable | Description |
| ---- | ---- |
| `N` | Number of distinct contributing instances |
| `M` | Mean of per-offset/bin means across the whole window |
| `L` | Mean of per-offset/bin means over negative (pre-anchor) offsets |
| `R` | Mean of per-offset/bin means over positive (post-anchor) offsets |
| `S` | Total span of the anchor annotation's instances, in seconds |

Peri-event offset/bin detail (strata: `CH` × `ANNOT` × `SEC`)

| Variable | Description |
| ---- | ---- |
| `N` | Number of distinct contributing instances at this offset/bin |
| `M` | Mean aligned signal value |
| `SD` | SD of aligned signal values (only if `N` > 1) |
| `MD` | Median aligned signal value |

---

## DPP

_Multiscale local features over trailing causal windows, and (`--dpp-fit`) Dynamic Phenotype Projection_

!!! warning "Under development"
    `DPP` and `--dpp-fit` are under development: documentation is being written,
    but the command is not yet ready for general use and is highly likely to
    change (parameters, feature set, and output format) in upcoming releases.
