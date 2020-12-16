# Artifact detection

_Commands to perform artifact detection/correction_

|Command |Description | 
|---|---|
| [`SIGSTATS`](#sigstats)      | Per-epoch outlier detection (RMS, Hjorth parameters, clipped signals) |
| [`ARTIFACTS`](#artifacts) | Per-epoch Buckelmueller et al. (2006) artifact detection | 
| [`SUPPRESS-ECG`](#suppress-ecg) | Correct cardiac artifact based on ECG | 

## `SIGSTATS`

_Epoch-wise Hjorth parameters and other statistics_

This command calculates and reports per-epoch (and whole-signal)
[Hjorth parameters](https://en.wikipedia.org/wiki/Hjorth_parameters)
and (optionally) other statistics: signal root mean square (RMS), indices of signal
clipping (the proportion of points that equal the minimum or maximum
for that epoch), absolute maximum absolute values and flatness
(proportion of points of a similar value to the preceding value).

Optionally, Luna will also mask epochs that are (statistical) outliers
(i.e. potentially indicative of signal artifacts) for these measures.
For Hjorth parameters, thresholds for outlier detection can be set
iteratively: for example, first, to flag epochs more than 2 standard
deviations (SD) from the mean, and then second, to flag more epochs
that are more than 2 SDs from the mean _based on the epochs that
survived round 1_, etc.  This iterative heuristic can be useful if
some epochs are outliers by many orders of magnitude, as those can
effectively hide other epochs that are less pronounced but outliers
nonetheless.
 
<h5>Parameters</h5>

| Parameter | Example | Description |
| --- | --- | --- |
| `sig`     | `sig=C3,F3` | Restrict analysis to these channels | 
| `epoch` | `epoch` | Report epoch-level as well as channel-level statistics |
| `th` | `th=2,2` | Set standard unit threshold(s) for (iterative) outlier detection on Hjorth parameteres |
| `rms` | `rms` | Also report RMS as well as Hjorth parameters |
| `clipped` | `clipped=0.05` | Flag epochs with this proportion of _clipped_ sample points |
| `flat` | `flat=0.05` | Flag epochs with this proportion of _flat_ sample points (optionally `flat=0.05,1e-4`) |
| `max` | `max=200,0.05` | Flag epochs with this proportion of points with absolute value above 200 |
| `mask`    | `mask` | Set mask for outlier epochs (only needed if no Hjorth/`th` mask set) |

The following options are relevant for high-density EEG studies:

| Parameter | Example | Description |
| --- | --- | --- |
| `chep`    | `chep` | Set a __CH__annel/__EP__och mask for outlier epochs |
| `cstats`  | `cstats` | Per epoch, find outlier channels |
| `cstats-unmasked-only` | `cstats-unmasked-only` | Only calculate _c_-stats on unmasked epochs |
| `astats`  | `astats` | Find outlier epoch/channel pairs |

!!! note "Methods for hdEEG epoch/channel outlier removal"
    These options will be described later.  In brief, the standard Hjorth-based outlier detection methods:
    - by default, condition on a channel, and look for outlier epochs
    - alternatively, _c_-stats conditions on each epoch, and looks for outlier channels
    - finally, _a_-stats considers each channel/epoch pair individually, against the distribution of _all_ other epochs for all other channels

    Whereas the default thresholding flags epochs as outliers (i.e. if one
    _or more_ channels are flagged as an outlier), `chep` mode means that
    individual channel/epoch combinations are flagged (i.e. epoch-level
    masking is channel-specific_.

<h5>Output</h5>

Per-channel whole-signal statistics (strata: `CH`)

| Variable | Description |
| --- | --- |
| `H1`    | First Hjorth parameter (activity) |
| `H2`    | Second Hjorth parameter (mobility) |
| `H3`    | Third Hjorth parameter (complexity) |
| `CLIP`  | Proportion of clipped sample points |
| `MAX`   | Proportion of maxed out sample points |
| `FLAT`  | Proportion of flat sample points |
| `RMS`   | Signal root mean square |

Per-channel whole-signal statistics (option: `mask` or `th`, strata: `CH`)

| Variable | Description |
| --- | --- |
| `FLAGGED_EPOCHS`  | Number of epochs flagged as outliers    | 
| `ALTERED_EPOCHS`  | Number of epochs whose mask was altered |
| `TOTAL_EPOCHS`    | Total number of masked epochs |
| `CNT_ACT`         | Number of epochs flagged based on H1 |
| `CNT_MOB`         | Number of epochs flagged based on H2 |
| `CNT_CMP`         | Number of epochs flagged based on H3 |
| `CNT_CLP`         | Number of epochs flagged based on clipping metric |
| `CNT_FLT`         | Number of epochs flagged based on flatness metric |
| `CNT_MAX`         | Number of epochs flagged based on max metric |
| `CNT_RMS`         | Number of epochs flagged based on RMS |


Per-epoch statistics (option: `epoch`, strata: `CH` x `E`)

| Variable | Description |
| --- | --- |
| `H1`    | First Hjorth parameter (activity) |
| `H2`    | Second Hjorth parameter (mobility) |
| `H3`    | Third Hjorth parameter (complexity) |
| `CLIP`  | Proportion of clipped sample points |
| `MAX`   | Proportion of maxed out sample points |
| `FLAT`  | Proportion of flat sample points |
| `RMS`   | Signal root mean square |
| `MASK`  | If `mask` is specified, whether this epoch's mask is set |


<h5>Example</h5>

Taking the first individual from the [tutorial](../tut/tut1.md), here
we calculate epoch-level metrics for the first EEG channel, and flag
outliers that are greater than 2 standard deviations from the mean
(performed twice, iteratively).

```
luna s.lst 1 sig=EEG -o out.db -s "SIGSTATS epoch mask threshold=2,2"
```

We first extract epoch-level output (i.e. stratified by `E` as well as
`CH`) into a plain-text file `stats.txt`:


```
destrat out.db +SIGSTATS -r CH E > stats.txt
```

We can then use a package such as R to load and plot these data.  For
example, from the R command line:

```
d <- read.table( "stats.txt", header = T )
```
```
head(d)
```
```
      ID  CH E         CLIP        H1        H2       H3 MASK       RMS
1 nsrr01 EEG 1 0.0002668090  98.45154 0.4807507 1.105732    0  9.922275
2 nsrr01 EEG 2 0.0002668090 234.34135 0.3164355 1.152268    1 15.308212
3 nsrr01 EEG 3 0.0005336179 159.33651 0.3511658 1.089591    0 12.622857
4 nsrr01 EEG 4 0.0000000000 181.39586 0.6554725 1.115427    1 13.468328
```

As noted in Luna's log when running `SIGSTATS`, we expect 385 epochs
to be flagged as outliers and potential artifacts:

```
table( d$MASK ) 
```

```
   0   1 
 979 385 
```

This high number in large part reflects that there were many wake
epochs at the end of this recording, that contained high levels of
artifact.

!!! note 
    Here, we've applied artifact detection across the entire
    night.  In practice, it might make more sense to apply stage-specific
    outlier detection, if one expects 
    large differences in these 
    metrics between wake versus sleep, or different sleep stages, etc.

Below, still using R, we plot four epoch-level metrics (`CLIP`, `H1` (log-scaled), 
`H2` and `H3), with the epochs set as outliers in purple:

```
f <- function(x) { ifelse(x,"purple","darkgray") } 
par(mfcol=c(4,1))
plot(d$E,d$CLIP    ,pch=20,cex=0.8,col=f(d$MASK),xlab="Epoch",ylab="CLIP")
plot(d$E,log(d$H1) ,pch=20,cex=0.8,col=f(d$MASK),xlab="Epoch",ylab="H1")
plot(d$E,d$H2      ,pch=20,cex=0.8,col=f(d$MASK),xlab="Epoch",ylab="H2")
plot(d$E,d$H3      ,pch=20,cex=0.8,col=f(d$MASK),xlab="Epoch",ylab="H3")
```

![img](../img/sigstats.png)

As noted above, all epochs past epoch 1100 or so are wake, essentially
containing nothing but noise (e.g. if recording continued after
electrodes were removed from the scalp). As expected, these have been
flagged as likely artifact.

A few notes on using `SIGSTATS` to detect outlying epochs:

- When applied to __multiple channels__, the mask will be set for any
  epoch that has at least one channel for which it is an outlier.
  The primary mask in Luna is not channel-specific: this ensures that
  all downstream analyses are performed on the same intervals of time.
  If you want to treat channels individually, run the entire set of
  commands (i.e. including outlier detection followed by whatever
  other analyses) separately for each channel.

- You can use `chep` instead of the `mask` option to specify
  channel-specific masks; these are respected by certain downstream
  commands (i.e. [`CHEP`](masks.md#chep),
  [`INTERPOLATE`](cross-signal-analyses.md#interpolate)), and can be
  converted to a standard (all-channel) mask.

- `SIGSTATS` respects __previously set [masks](masks.md)__ and does
  not include them in the outlier detection process, i.e. as they have
  already been removed.  That is, epochs masked from a previous
  [`MASK`](masks.md#mask), [`ARTIFACTS`](#artifacts) or `SIGSTATS`
  command will be ignored (even if a
  [`RESTRUCTURE`](masks.md#restructure) has not been executed).

- You can set a __smaller epoch size__ such as `EPOCH len=4` for
  finer-grained artifact detection; in this case, however, you may
  need to be careful that existing epoch-level annotations
  (i.e. staging) are properly encoded and respected in downstream
  analyses. (As such, the simplest approach is to first set masks
  based on sleep stages and restructure the data, if desired, and then
  apply this second level of artifact detection.)

To demonstrate the use of smaller epochs concretely, this first
command file applies outlier detection to N2 epochs only (that is, we
first mask out all other epochs other than N2) using default 30-second epochs:

```
% Applying a mask will implicitly set 30-second epochs
MASK ifnot=NREM2    

% Drop all masked epochs (RE is short for RESTRUCTURE)
RE      

% The default epoching is preserved, now run outlier detection
SIGSTATS epoch mask threshold=2,2
```

When running this, we see the following message in the log, showing
that 92 of 523 30-second N2 epochs were flagged as outliers:

```
 RMS/Hjorth filtering EEG, threshold +/-2 SDs: removed 36 of 523 epochs of iteration 1
 RMS/Hjorth filtering EEG, threshold +/-2 SDs: removed 56 of 487 epochs of iteration 2
 Overall, masked 92 of 523 epochs (RMS:28, CLP:0, ACT:44, MOB:35, CMP:40)
```

Alternatively, in this second script we use 4-second epochs for the outlier detection step:

```
% As before, 30-second epochs will be set 
% (i.e. staging information may be in an .eannot file which assumes 30-second epochs 

MASK ifnot=NREM2    

% As before, drop all masked epochs

RE      

% Now reset the epoch length to four seconds 
% Note, changing the epoch duration will mean that mappings 
% to the original epoch numbering scheme are lost

EPOCH len=4

% Run outlier detection on 4-second epochs 

SIGSTATS epoch mask threshold=2,2

% If you then wish to use 30-second epochs for downstream analyses, you need to 
% set them again 

EPOCH len=30

PSD epoch
```

In this instance, we now see a total of 3922 4-second N2 epochs, of which 838 are masked:

```
 RMS/Hjorth filtering EEG, threshold +/-2 SDs: removed 389 of 3922 epochs of iteration 1
 RMS/Hjorth filtering EEG, threshold +/-2 SDs: removed 449 of 3533 epochs of iteration 2
 Overall, masked 838 of 3922 epochs (RMS:295, CLP:0, ACT:352, MOB:312, CMP:324)
```

Whether or not using smaller (or longer) epochs performs better or
worse will depend on the particulars of the data, as well as whether
it is more important for the subsequent analyses to retain as much data
as possible, versus for all retained data to be "clean".


## `ARTIFACTS`

_Applies an artifact detection algorithm for EEG data_

This command applies the method described in [Buckelmueller et
al](https://www.ncbi.nlm.nih.gov/pubmed/16388912), to detect likely
artifacts in an EEG channel. Briefly, the method takes 10 4-second
windows for each 30-second epoch to estimates delta (0.6 to 4.6 Hz)
and beta (40-60Hz) spectral power, using 
[Welch's method](https://en.wikipedia.org/wiki/Welch%27s_method).  Based
on a sliding window of 15 epochs (i.e. seven either side of each
epoch), an epoch is flagged to be masked if either the delta power is
more than 2.5 times the local average (from the sliding window) or the
beta power is more than 2.0 times the local average.

<h5>Parameters</h5>

| Parameter | Example | Description |
| --- | --- | --- |
| `sig`     | `sig=C3,F3` | Restrict analysis to these channels | 
| `verbose` | `verbose` | Report epoch-level statistics |
| `no-mask` | `no-mask` | Run, but do not set any masks |

<h5>Output</h5>

Channel-level output (strata: `CH`):

| Variable | Description |
| ---- | ---- |
|`FLAGGED_EPOCHS`| Number of epochs failing Buckelmueller |
|`ALTERED_EPOCHS`| Number of epochs actually masked |
|`TOTAL_EPOCHS`  | Number of epochs tested | 


Epoch-level output (option: `verbose`, strata: `CH` x `E`):

| Variable | Description |
|---- | ---- |
|`DELTA` | Delta power |
|`DELTA_AVG` | Local average delta power |
|`DELTA_FAC` | Relative delta power factor |
|`BETA` | Beta power | 
|`BETA_AVG` | Local average beta power |
|`BETA_FAC` | Relative beta power factor |
|`DELTA_MASK` | Masked based on delta power? |
|`BETA_MASK` | Masked based on beta power? |
|`MASK` | Whether the epoch is masked |

<h5>Example</h5>

Taking the same EEG channel from the first tutorial EDF as in the
`SIGSTATS` example above:
```
luna s.lst 1 sig=EEG -o out.db -s "ARTIFACTS verbose" 
```

We see this command flags 60 epochs as potentially containing artifact:

```
masked 60 of 1364 epochs, altering 60
```
We can extract the epoch-level statistics from this analysis:

``` 
destrat out.db +ARTIFACTS -r E CH > d.txt 
``` 

We can then use R to show the results, for delta (left column) and
beta (right column) separately, showing log-scaled power per epoch
(top row), the local average (middle row) and the epoch/local relative
factor (bottom row), where purple points indicate that epoch was
flagged as an outlier (by either delta or beta criterion).


```
d <- read.table("d.txt",header=T)
par(mfcol=c(3,2), mar=c(4,4,2,2) )
plot( log(d$DELTA)     , type="l" , ylab="Delta avg" )
plot( log(d$DELTA_AVG) , type="l" , ylab="Delta local")
plot( d$DELTA_FAC , pch=20 , col = f( d$MASK ) , ylab="Beta fac") 
plot( log(d$BETA)     , type="l" , ylab="Beta avg" )
plot( log(d$BETA_AVG) , type="l" , ylab="Beta local")
plot( d$BETA_FAC , pch=20 , col = f( d$MASK ) , ylab="Beta fac") 
```

![img](../img/buck.png)

Comparing the results to the [`SIGSTATS`](#sigstats) method described
above, which is applied to the same data, we see that this approach
does a good job of flagging individual epochs with unusual artifacts,
but does not flag the very extended stretches of gross artifact
towards the end of the recording (i.e. because the entire local window
is itself aberrant) .  This latter type of artifact is, of course,
easier to spot by eye, and is flagged by `SIGSTATS`. In practice,
running multiple artifact detection/correction routines is usually
warranted.


## `SUPPRESS-ECG`

_Given an ECG channel, detect R peaks and subtract cardiac-related contamination from the EEG_

This command applies a modified
[Pan-Tompkins](https://www.ncbi.nlm.nih.gov/pubmed/3997178) algorithm
to detect QRS in the ECG.  Based on the detected R-peaks, it estimates
the average signature for other channels based on 2-second intervals
time-locked with each R-peak.  This is subtracted from the EEG, as
described [here](https://www.ncbi.nlm.nih.gov/pubmed/28649997). It
also estimates heart rate per-epoch, and flags values likely to
represent artifact. If needed, channels will be resampled to have
similar sampling rates (to be set to the value of the parameter `sr`).

<h5>Parameters</h5>

| Parameter | Example | Description |
| ---- | ----- | ----- |
| `ecg` | `ecg=ECG` | Specify which channel is the ECG |
| `sr` | `sr=125` | Set the sample rate that all channels should to resampled to |
| `no-suppress` | `no-suppress` | Run the command but do not alter any EEG channels |

<h5>Output</h5>

Individual-level summaries (strata: _none_):

| Variable | Description |
| --- | --- |
| `BPM` | Mean heart rate (bpm) |
| `BPM_L95` | Lower 95% confidence interval for mean HR |
| `BPM_U95` | Upper 95% confidence interval for mean HR |
| `BPM_N_REMOVED` | Number of epochs flagged as having invalid HR estimates |
| `BPM_PCT_REMOVED` | Proportion of epochs flagged as having invalid HR estimates |

Epoch-level metrics (strata: `E`):

| Variable | Description |
| --- | --- |
| `BPM` | Heart rate for this epoch (bpm) |
| `BPM_MASK` | Was this epoch invalid? | 

Channel-level summaries (strata: `CH`):

| Variable | Description |
| --- | --- |
| `ART_RMS` | Root mean square of correction signature |

Details of artifact signature (strata: `CH` x `SP`) 
 
| Variable | Description |
| --- | --- |
| `ART` | For each sample point in a 2-second window, the estimated correction factor |


<h5>Example</h5>

Here we look at individual `nsrr02` from the
[tutorial](../tut/tut1.md) data, to identify and remove possible 
cardiac contamination in the EEG:

```
luna s.lst nsrr02 sig=EEG,ECG -o out.db < cmd.txt
```

The command file `cmd.txt` restricts analysis to NREM2 sleep,
estimates EEG/ECG [coherence](cross-signal-analyses.md#coh) (and
[spectral power](power-spectra.md#psd)), then estimates and corrects for
cardiac contamination in the EEG, and finally, repeats the coherence
and spectral analyses:

```
MASK ifnot=NREM2 
RE 
TAG R/pre 
COH 
PSD spectrum 
SUPPRESS-ECG ecg=ECG 
TAG R/post 
PSD spectrum 
COH
```

!!! hint
    Note how we've used the [`TAG`](summaries.md#tag) command to structure the output, by adding a factor `R` to denote
    whether the coherence and spectral analysis was performed `pre` or `post` the `SUPPRESS-ECG` command.

We can extract the per-epoch estimates of heart rate (bpm) and plot as follows:

```
destrat out.db +SUPPRESS-ECG -r R E > hr.txt
```

Plotting the `BPM` field of `hr.txt` against epoch (`E`), e.g. in R: 

```
plot(d$E,d$BPM,pch=20,col="red",ylim=c(50,80),ylab="HR (bpm)",xlab="NREM2 epoch")
```

![img](../img/hr.png)

Second, we can extract the estimates of coherence between EEG and ECG as follows:

```
destrat out.db +COH -r F -c R -r CHS -v COH > d.txt
```

Plotting the `pre` (black) and `post` (blue) coherence values, for
frequencies up to 20 Hz, we see that a non-trivial association between
ECG and EEG channels has effectively been removed based on these
metrics:

```
plot( d$F , d$COH.R.pre , ylim=c(0,1) , type="b" , pch=20, ylab="Coherence" , xlab="Frequency (Hz)" ) 
points( d$F , d$COH.R.post , ylim=c(0,1) , type="b" , pch=20, col="blue" ) 
```

![img](../img/hrcoh.png)

Note, this is only a _rough-and-ready_ approach to working with ECG data,
which is not a primary focus of Luna.  Other artifacts may well remain
(e.g. in higher frequencies) and this approach may not work well with
other datasets.  That is, as usual, your mileage may vary...

!!! hint 
    When comparing different individual or signals, the `ART_RMS`
    metric can be taken as an approximate guide to the relative extent
    of contamination.  The actual _signatures_ (i.e. the R-peak
    sync'ed averaged EEG) can be viewed by looking at the `ART`
    variable, which is a channel by sample-point (`CH` x `SP`) metric.
