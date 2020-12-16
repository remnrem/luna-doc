# Spectral and other time/frequency analyses

_Methods for spectral and other time/frequency analyses, including
power spectral density estimation_

|Command |Description | 
|---|---|
| [`PSD`](#psd)         | Welch's method for power spectral density estimation |
| [`MTM`](#mtm)         | Multi-taper method for power spectral density estimation |
| [`MSE`](#mse)         | Multi-scale entropy statistics | 
| [`LZW`](#lzw)         | LZW compression (information content) index |
| [`HILBERT`](#hilbert) | Hilbert transform |
| [`CWT`](#cwt)         | Continuous wavelet transform |
| [`CWT-DESIGN`](#cwt-design) | Complex Morlet wavelet properties |
| [`1FNORM`](#1fnorm)         | Remove the _1/f_ trend from a signal | 
| [`TV`](#tv)                 | Total variation denoiser |
| [`ACF`](#acf) | Autocorrelation function |

## `PSD`

_Estimates a signal's power spectral density (PSD)_

This command uses [Welch's
method](https://en.wikipedia.org/wiki/Welch%27s_method) to estimate
power spectra and band power for one or more signals.  As well as
estimates for the entire signal (possibly following
[masking](masks.md), etc), this command optionally provides
epoch-level estimates.

Internally, this command operates on an epoch-by-epoch basis:
e.g. taking 30 seconds of signal, and using Welch's method of
overlapping windows (by default, 4-second windows with 2-second
overlap) to estimate the power spectra via FFT.  By default, intervals
are windowed using a 50% tapered Tukey window, although [Hann and
Hamming](https://en.wikipedia.org/wiki/Window_function#Hann_and_Hamming_windows)
windows can also be specified.  If epoch-level output is requested,
e.g. with the `epoch` option, then these spectra are also written to
the output database.  The _overall_ estimate of the PSD is the average
of the epoch-level estimates.


<h5>Parameters</h5>

| Parameter | Example | Description |
| ---- | ---- | ---- |
| `spectrum` | `spectrum` | Estimate power spectra as well as band power |
| `max`     | `max=30` | Upper frequency range to report for spectra (default is 20 Hz) |
| `epoch` | `epoch` | Output epoch-level band power estimates |
| `epoch-spectrum` | `epoch-spectrum` | Output epoch-level power spectra |
| `dB` | `dB` | Give power in dB units |
| `bin` | `bin=1` | Set bin size for power spectra (default is 0.5 Hz, 0 means no binning) |

In addition to the primary parameters above, there are a number of
other, more detailed parameters (that can probably be ignored by most
users), as described in this table:

| Parameter | Example | Description |
| ---- | ---- | ---- |
| `segment-sec`     | `segment-sec=8` | Set window size for Welch's method (default is 4 seconds) |
| `segment-overlap` | `segment-overlap=4` | Set window overlap for Welch's method (default is 2 seconds) |
| `center`     | `center` | First mean-center each epoch (or `centre`) |
| `no-average` | `no-average` | Do not average adjacent points in the power spectra |
| `tukey50`    | `tukey50` | Apply Tukey 50% window (default) |
| `hann`       | `hann` | Apply a Hann window function |
| `hamming`    | `hamming` | Apply a Hamming window function |
| `no-window`  | `no-window` | Do not apply any window function |


!!! warn 
    If the `EPOCH` size is set to a small value (i.e. under 4
    seconds) you will need to adjust the parameters of Welch's method
    accordingly.

<h5>Band definitions</h5>

Luna uses the following band definitions: 

- `SLOW` (0.5 to 1 Hz)
- `DELTA` (1-4 Hz)
- `THETA` (4-8 Hz)
- `ALPHA` (8-12 Hz)
- `SIGMA` (12-15 Hz)
- `BETA` (15-30 Hz) 
- `GAMMA` (30+ Hz).  

!!! hint
    These can be modified by setting [special variables](../luna/args.md#spectral-power-bands) either via the
    command-line or in a [parameter file](../luna/args.md#parameter-files).


In addition, `SLOW_SIGMA` and `FAST_SIGMA` are defined as 12-13.5 Hz
and 13.5-15 Hz respectively.  

<h5>Outputs</h5>


Channel-level information (strata: `CH`)

| Variable | Description |
| ---- |----- | 
| `NE` | Number of epochs included | 


Spectral band power (strata: `B` x `CH`)

| Variable | Description |
| ---- |----- | 
| `PSD` | Absolute spectral power |
| `RELPSD` | Relative spectral power |

Spectral power by frequency bin (option: `spectrum`, strata: `F` x `CH`)

| Variable | Description |
| ---- |----- | 
| `PSD` | Absolute spectral power |


Epoch-level spectral band power (option: `epoch`, strata: `E` x `B` x `CH`)

| Variable | Description |
| ---- |----- | 
| `PSD` | Absolute spectral power |
| `RELPSD` | Relative spectral power |

Epoch-level spectral power by frequency bin (option: `epoch-spectrum`, strata: `E` x `F` x `CH`)

| Variable | Description |
| ---- |----- | 
| `PSD` | Absolute spectral power |


<h5>Example</h5>

Here we calculate band power and the PSD for
[tutorial](../tut/tut1.md) individual `nsrr01`, for all N2 and all N3
sleep separately.  Note, here we run Luna twice but put all output in
the same `out.db` database, by using `-a` to append for the second
command, rather than `-o`.  We also add a `TAG` command to
disambiguate the output:

```
luna s.lst 2 sig=EEG -o out.db -s "EPOCH & MASK ifnot=NREM2 & RE & TAG SS/N2 & PSD spectrum"
luna s.lst 2 sig=EEG -a out.db -s "EPOCH & MASK ifnot=NREM3 & RE & TAG SS/N3 & PSD spectrum"
```
Here we see that all output for `PSD` has an additional `SS` (sleep stage) stratifier:
```
destrat out.db
```
```
--------------------------------------------------------------------------------
distinct strata group(s):
  commands      : factors           : levels        : variables 
----------------:-------------------:---------------:---------------------------
  [EPOCH]       : .                 : 1 level(s)    : DUR INC NE
                :                   :               : 
  [RE]          : .                 : 1 level(s)    : DUR1 DUR2 NR1 NR2
                :                   :               : 
  [MASK]        : EPOCH_MASK        : 2 level(s)    : N_MASK_SET N_MASK_UNSET N_MATCHES
                :                   :               : N_RETAINED N_TOTAL N_UNCHANGED
                :                   :               :
                :                   :               : 
  [PSD]         : CH SS             : 2 level(s)    : NE
                :                   :               : 
  [PSD]         : F CH SS           : 82 level(s)   : PSD
                :                   :               : 
  [PSD]         : B CH SS           : 20 level(s)   : PSD RELPSD
                :                   :               : 
----------------:-------------------:---------------:---------------------------
```

The number of epochs of N2 and N3 sleep respectively:
```
destrat out.db +PSD -r CH SS
```
```
ID        CH     SS    NE
nsrr02	  EEG	 N2    399
nsrr02	  EEG	 N3    185
```

Here we tabulate relative power for N2 and N3 sleep:
```
destrat out.db +PSD -r CH B -c SS -v RELPSD -p 2 
```
```
ID       B           CH     RELPSD.SS.N2  RELPSD.SS.N3
nsrr02   SLOW        EEG    0.18          0.21
nsrr02   DELTA       EEG    0.50          0.61
nsrr02   THETA       EEG    0.15          0.10
nsrr02   ALPHA       EEG    0.07          0.03
nsrr02   SIGMA       EEG    0.04          0.02
nsrr02   SLOW_SIGMA  EEG    0.02          0.01
nsrr02   FAST_SIGMA  EEG    0.01          0.01
nsrr02   BETA        EEG    0.02          0.01
nsrr02   GAMMA       EEG    0.00          0.00
nsrr02   TOTAL       EEG    1.00          1.00
```

As expected, the relative power of delta sleep is higher in N3 (61%)
compared to N2 (50%) for this individual.

To look at per-epoch estimates of band power for all N2 and N3 sleep:

```
luna s.lst 2 sig=EEG -o out2.db -s "MASK if=wake & RE & PSD epoch"
```

For a change, here we'll use [_lunaR_](../ext/R.md) to directly load
`out2.db` into the [R statistical package](https://www.r-project.org).
If you have R and [lunaR](../ext/R.md) installed, then at the R prompt:

```
library(luna)
```
```
k <- ldb("out2.db")
```
To summarize the contents:
```
lx(k)
```
```
MASK : EPOCH_MASK 
PSD : CH B_CH CH_F B_CH_E CH_E_F 
RE : BL 
```
We can directly extract a data frame of epoch by band by channel information:
```
d <- k$PSD$B_CH_E
```
From this, we can further select on delta power:
```
delta <- d[ d$B == "DELTA" , ] 
```
Plotting these data, we see a moderate decrease in relative delta power across the night:
```
plot( delta$E ,delta$RELPSD , pch=20 , col="blue", ylab="Relative delta power" , xlab="Epoch" )
```
![img](../img/delta.png)

The correlation coefficient between epoch number and relative delta
power is _r_ = -0.36 and highly significant:

```
cor.test( delta$E ,delta$RELPSD  ) 
```
```
    Pearson's product-moment correlation

data:  delta$E and delta$RELPSD
t = -10.304, df = 713, p-value < 2.2e-16
alternative hypothesis: true correlation is not equal to 0
95 percent confidence interval:
 -0.4221768 -0.2944507
sample estimates:
       cor 
-0.3599995 
```



## `MTM`

_Applies the multitaper method for spectral density estimation_

This provides an alternative to [`PSD`](#psd) for spectral density estimation,
that can be more efficient in some scenarios (albeit slower): the
multitaper method as described
[here](https://en.wikipedia.org/wiki/Multitaper).  

The time half bandwidth product parameter (`nw`) provides a way to
balance the variance and resolution of the PSD: higher values reduce
both the variance and the frequency resolution, meaning smoother but
potentially blunted and biased power spectra.  The optimal choice of
`nw` will depend on the properties of the data and the research
question at hand. [This manuscript](https://www.ncbi.nlm.nih.gov/pubmed/27927806) 
provides a nice review of the use of multitaper spectral analysis in the sleep
domain, along with considerations for specifying the time half
bandwidth product (`nw`) and the number of tapers (`t`). (By default,
`MTM` will always use `2nw-1` tapers.)

<h5>Parameters</h5>

| Parameter | Example | Description |
| ----- | ------ | ------ |
| `sig`   | `sig=C3,C4` | Analysis of only these signals |
| `epoch` | `epoch` | Perform epoch-level as well as whole-signal analyses |
| `epoch-only`| `epoch-only` | Only perform epoch-level analyses |
| `nw`    | `nw=4` | Time half bandwidth product (default 3, typically: 2, 5/2, 3, 7/2, or 4) | 
| `t`     | `t=7` | Number of tapers (default 2*`nw`-1, i.e. 5) |
| `max`   | `max=25` | Maximum frequency for power spectra (default is 20Hz) |
| `bin`   | `bin=1` | Bin size for power spectra |
| `full-spectrum` | `full-spectrum` | Report the full spectra rather than binning (same as `bin=0`) |
| `dB`    | `dB` | Report power in dB units |

<h5>Output</h5>

Whole-signal power spectra (strata: `CH` x `F`)

| Variable | Description |
| ----- | ----- | 
| `MTM` | Absolute spectral power via the multitaper method |

Epoch-level power spectra (potion: `epoch` or `epoch-only`, strata: `E` x `CH` x `F`)

| Variable | Description |
| ----- | ----- | 
| `MTM` | Absolute spectral power via the multitaper method |

<h5>Example</h5>


To compare results for the N2 power spectra, from `PSD` and `MTM`
commands executed on the second individual from the tutorial
(`nsrr02`).  Here we'll use _lunaR_ (instead of the command-line version, _lunaC_)
for this example (it could be run in either).  In R:

```
sl <- lsl( "s.lst" ) 

lattach( sl , "nsrr02" ) 

k <- leval( "MASK ifnot=NREM2 & RE & PSD sig=EEG bin=0.5 spectrum & MTM sig=EEG bin=0.5" )

luna s.lst -o out.db -s "MASK ifnot=NREM2 & RE & PSD sig=EEG bin=0.5 spectrum dB & MTM sig=EEG bin=0.5 dB" 

```

```
k <- ldb( "out.db" )
```
```
read data on 3 individuals
```

```
lx(k)
```
```
MASK : EPOCH_MASK 
MTM : CH_F 
PSD : CH B_CH CH_F 
RE : BL 
```

```
mtm <- lx( k , "MTM" , "CH" , "F" )
psd <- lx( k , "PSD" , "CH" , "F" )
```

```
par(mfcol=c(1,3))
yr <- range( c( mtm$MTM , psd$PSD ) )
for (i in unique( mtm$ID ) ) { 
plot( mtm$F[mtm$ID==i] ,  mtm$MTM[mtm$ID==i] , type="l" , col="purple" , lwd=2 , xlab="Frequency (Hz)" , ylab="Power (dB)" , ylim=yr ) 
lines( psd$F[psd$ID==i] , psd$PSD[psd$ID==i] , type="l" , col="orange" , lwd=2 ) 
legend(12,20,c("MTM","PSD"),fill=c("purple","orange"))
}
```


![img](../img/mtm1.png)

As expected, in this particular scenario and with long signals, both
methods produce similar results.  Note the difference in the first
plot (`nsrr01`) at the lower frequency range is due to the slightly
different way in which Luna estimates whole-signal (or, in this case,
all-N2) power: for `PSD`, these estimates are the mean of per-epoch
estimates, whereas they are calculated directly for `MTM`.  The former
approach implicitly removes any gross trends across the night.
Naturally, it is easy to take the mean of per-epoch MTM estimates, or
to apply other methods such as detrending or a high-pass filter to
remove these very slow components, if so desired.  In the future,
we'll add extended tutorial-like work-cases to this website, to
further explore the properties of these approaches on real data.

----

As a second example: here is an application of MTM on a smaller
segment of data (a single epoch), which shows sleep spindles in the
MTM spectrogram (plotting the results in the range of 8 to 20 Hz), generated by the commands:

```
EPOCH dur=2.5 inc=0.02 & MTM epoch nw=5 max=30 bin=0 dB
```

Note the use of a small (2.5 seconds) epoch size, which is shifted
only 0.02 seconds at a time, and so gives a considerable smoothing of
estimates in the time domain (which may or may not be desirable,
depending on the goal of the analysis.)

![img](../img/mtm2.png)


## `MSE`

_Calculates per-epoch multi-scale entropy statistics_


This function estimates multi-scale entropy (MSE) as described in the
approach of [Costa et al](https://physionet.org/physiotools/mse/tutorial/), which is based
on the concept of [sample entropy](https://en.wikipedia.org/wiki/Sample_entropy).

In short, there are two steps: first, the time series is
coarse-grained, dependent on scale parameter `s` (typically varied
between 1 and 20); second, sample entropy is calculated for each
coarse-grained time series, dependent on parameters `m` and `r`.
Parameters `m` and `r` define the pattern length and the similarity
criterion respectively, with default values of 2 and 0.15
respectively. Smaller values of (multi-scale) entropy indicate more
self-similarity and less noise in a signal.

<h5>Parameters</h5>

| Parameter | Example | Description |
| ---- | ----- | ----- | 
| `m` | `m=3` | Embedding dimension (default 2) |
| `r` | `r=0.2` | Matching tolerance in standard deviation units (default 0.15) |
| `s` | `s=1,15,2` | Consider scales 1 to 15, in steps of 2 (default 1 to 10 in steps of 1) | 
| `verbose` | `verbose` | Emit epoch-level MSE statistics |


<h5>Outputs</h5>

MSE per channel and scale (strata: `CH` x `SCALE`)

| Variable | Description |
| ----- | ------ |
| `MSE` | Multi-scale entropy |

Epoch-level MSE per channel and scale (option: `verbose`, strata: `E` x `CH` x `SCALE`)

| Variable | Description |
| ----- | ------ |
| `MSE` | Multi-scale entropy |



## `LZW`

_Calculate per-epoch LZW compression index_

Lempel–Ziv–Welch (LZW) is a commonly used data compression algorithm,
which can be applied to coarse-grained sleep signals to provide a
quantitative metric (the ratio of the size of the compressed signal
versus the original signal) of the amount of non-redundant information
in a signal.

<h5>Parameters</h5>

| Parameter | Example | Description |
| ---- | ----- | ----- | 
| `nsmooth` | `nsmooth=2` | Coarse-graining parameter (similar to scale `s` in `MSE`) |
| `nbins` | `nbins=5` | Matching tolerance in standard deviation units (default 10) |
| `epoch` | `epoch` | Emit epoch-level LZW statistics |

<h5>Outputs</h5>

LZW per channel (strata: `CH`)

| Variable | Description |
| ----- | ------ |
| `LZW` | Compression index |

Epoch-level LZW per channel and scale (option: `epoch`, strata: `E` x `CH`)

| Variable | Description |
| ----- | ------ |
| `LZW` | Compression index |


## `HILBERT`

_Applies filter-Hilbert transform to a signal, to estimate envelope and instantaneous phase_

This function can be used to generate the envelope of a (band-pass
filtered) signal.

<h5>Parameters</h5>

| Parameter | Example | Description |
| ---- | ---- | ----- | 
| `sig` | `sig=EEG` | Which signal(s) to apply the filter-Hilbert to |
| `f` | `f=0.5,4` | Lower and upper transition frequencies |
| `ripple` | `ripple=0.02` | Ripple (0-1) |
| `tw` | `tw=0.5` | Transition width (in Hz) |
| `tag` | `tag=v1` | Additional tag to be added to the new signal |
| `phase` | `phase` | Generate a second new signal with instantaneous phase |

<h5>Outputs</h5>

No formal output, other than one or two new signals in the _in-memory_
representation of the EDF, with `_hilbert_mag` and (optionally)
`_hilbert_phase` suffixes.

<h5>Example</h5>

Using [_lunaR_](../ext/R.md), with `nsrr02` attached, we will use the filter-Hilbert method to 
get the envelope of a sigma-filtered EEG signal.  After attaching the sample, we then drop all signals
except the one of interest:

```
leval( "SIGNALS keep=EEG" ) 
```

We then apply the filter-Hilbert method, which will generate two new
channels, `EEG_hilbert_11_15_mag` and `EEG_hilbert_11_15_phase`:

```
leval( "HILBERT sig=EEG f=11,15 ripple=0.02 tw=0.5 phase" )
```

For illustration, we'll also generate a copy of the original signal:

```
leval( "COPY sig=EEG tag=SIGMA" )
```

and then apply a bandpass filter to it, in the same sigma range as above:

```
leval( "FILTER sig=EEG_SIGMA bandpass=11,15 ripple=0.02 tw=0.5" ) 
```

!!! note
    Unlike `HILBERT`, `FILTER` modifies the source channel,
    which is why we `COPY`-ed the original channel first.

We now have four signals in the _in-memory_ representation of the
EDF:

```
lchs()
```
```
[1] "EEG"                     "EEG_hilbert_11_15_mag"  
[3] "EEG_hilbert_11_15_phase" "EEG_SIGMA"              
```

To view some of the results, we can use `ldata()` to extract signals
for a particular epoch.  For better visualization, here we'll select
smaller (15 second) epochs:

```
lepoch(15)
```

We can then pull all four signals for given (set of) epoch(s), say number 480:

```
d <- ldata( 480 , chs=lchs() ) 
```
Using R's plotting functions:  
```
par(mfcol=c(3,1),mar=c(0,4,0,0),xaxt='n',yaxt='n')
plot( d$SEC , d$EEG , ylab = "Raw" , type="l" ,axes=F)
plot( d$SEC , d$EEG_SIGMA , ylab = "Filtered" , type="l" , axes=F)
lines( d$SEC , d$EEG_hilbert_11_15_mag , col="red" , lwd=2 ) 
plot( d$SEC , d$EEG_hilbert_11_15_phase , ylab = "Phase" , type="l" , axes=F)
```

![img](../img/hilbert.png)


## `CWT`

_Applies a continuous wavelet transform by convolution with a complex Morlet wavelet_

The CWT is the basis of the [`SPINDLE`](spindles-so.md#spindles)
command.  This command allows you to generate new signals in the EDF
that correspond to the underlying CWT, e.g. for plotting, or getting
insight into the performance of `SPINDLES` under different
circumstances.


<h5>Parameters</h5>

| Parameter | Example | Description |
| ---- | ---- | ----- | 
| `sig` | `sig=EEG` | Which signal(s) to apply the CWT to |
| `fc`     | `fc=15` | Wavelet center frequency |
| `cycles` | `cycles=12` | Bandwidth of the wavelet, specified in terms of the number of cycles |
| `tag`    | `tag=v1` | Additional tag to be added to the new signal |
| `phase`  | `phase` | Generate a second new signal with wavelet's phase |

<h5>Outputs</h5>

No formal output, other than one (or two) new signals appended to the
_in-memory_ representation of the EDF.



## CWT-DESIGN

_Display the properties of a complex Morlet wavelet transform_

This command does not operate on EDFs _per se_; rather, it produces
analytic output on the properties of a continuous wavelet transform
(CWT) given the design parameters.

Wavelet bandwidth can be specifed in one of two ways: by giving the
number of cycles (`cycles` option) _OR_ by specifying the time-domain
full width at half maximum (FWHM) value (in seconds).  See [this
manuscript](https://www.biorxiv.org/content/10.1101/397182v1.full.pdf)
for a discussion of the advantages of this latter specificiation.

In both cases, the `CWT-DESIGN` will estimate the implied FWHM in the frequency domain,
i.e. the tightness of the wavelet around the specified central frequency (`fc`). 

<h3>Parameters</h3>

| Parameter | Example | Description | 
| ---- | ------ | ---- |  
| `fs` | `fs=200`  | Sample rate |
| `fc` | `fc=15` | Center frequency | 
| `fwhm` | `fwhm=1` | Time-domain FWHM (use instead of `cycles`) |
| `cycles` | `cycles=7` | Number of cycles in wavelet (use instead of `fwhm`) |

<h3>Outputs</h3>

Time/frequency domain FWHM (strata: `PARAM`)

| Variable | Description |
| ----- | ----- |
| `FWHM` | Specified time-domain full width at half max (if `fwhm` option given) (secs) |
| `FWHM_F` | Estimated frequency-domain FWHM (Hz) |
| `FWHM_LWR` | Estimated lower half-max frequency bound (Hz) |
| `FWHM_UPR` | Estimated upper half-max frequency bound (Hz) |

Frequency response for wavelet (strata: `PARAM` x `F`)

| Variable | Description |
| ----- | ----- | 
| `MAG` | Magnitude of response (arbitrary units) |

Wavelet coefficients (strata: `PARAM` x `SEC`)

| Variable | Description |
| ----- | ----- | 
| `REAL` | Real part of wavelet |
| `IMAG` | Imaginary part of wavelet |


<h3>Example</h3>

To display the properties of a wavelet with center frequency of 15 hz
and 12 cycles, applied to a signal with sample rate of 12 Hz.

```
luna s.lst 1 -o out.db -s "CWT-DESIGN fc=15 cycles=12 fs=200" 
```

!!! Note
    The default value of `cycles` for the [`SPINDLES`](spindles-so.md#spindles) command is 7 cycles.


Equivalently, without an EDF/sample list, you can use the `--cwt`
parameter and pipe the parameters (`fc`, `cycles` and `fs`).  Here we
use it for both 11 Hz and 15 Hz wavelets. Also, note  the use of
`-a` instead of `-o` for the second command, so that the output of the
second command _appends_ (rather than _overwrites_) the existing
`out.db`:

```
echo "fc=11 cycles=12 fs=200" | luna --cwt -o out.db 
echo "fc=15 cycles=12 fs=200" | luna --cwt -a out.db 
```

Using [_lunaR_](../ext/R.md) to view the output:

```
k <- ldb("out.db")
```
We see two strata:
```
lx(k)
```
```
CWT_DESIGN : PARAM F_PARAM PARAM_SEC 
```
Extracting the strata defined by `PARAM` (a description of the input parameters) and `F` (frequency):
```
d <- lx( k , "CWT_DESIGN" , "PARAM" , "F" ) 
```
We can then plot the amplitude response (arbitrary units, scaled to 1.0) for the two wavelets:
```
plot( d$F[ d$PARAM == "11_12_200" ]  , d$MAG[ d$PARAM == "11_12_200" ] , 
 xlim=c(0,20) , type="l" , lwd=2 , col="blue" , 
 xlab="Frequency (Hz)" , ylab="Amplitude" , ylim=c(0,1) ) 

lines( d$F[ d$PARAM == "15_12_200" ]  , d$MAG[ d$PARAM == "15_12_200" ] , 
 lwd=2 , col="red" )

legend( 2 , 0.9 , c("11 Hz","15 Hz") , fill = c("blue","red") )
```

![img](../img/cwt-design.png)

Looking at the estimated frequency domain FWHM values, we see these correspond to the _y=0.5_ (i.e. 50%)
values for each wavelet, at lower and upper valeus respectively.

```
lx( k , "CWT_DESIGN" , "PARAM"  )
```
```
ID      PARAM    FWHM_F  FWHM_LWR  FWHM_UPR
.   11_12_200  2.197802   9.89011  12.08791
.   15_12_200  3.003003  13.51351  16.51652
```


## 1FNORM

_Applies a differentiator filter to remove 1/f trends in signals_

Many biological signals such as the EEG have an approximately _1/f_
frequency distribution, meaning that slower frequencies tend to have
exponentially greater power than faster frequencies.  It may sometimes
be useful to normalize signals in such a way that removes this
trend (e.g. in visualization, or detecting peaks against a background
of a roughly flat baseline). The `1FNORM` command is an implementation
of [this method](https://www.ncbi.nlm.nih.gov/pubmed/18070337) to
normalize power spectra, by passing the signal through a
differentiator prior to spectral analysis.

<h3>Parameters</h3>

| Parameter | Example | Description |
| ---- | ---- | ---- | 
| `sig` | `sig=C3,C4` | Optional parameter to specify which channels to normalize (otherwise, all channels are normalized) |

<h3>Outputs</h3>

No output _per se_, other than modifying the _in-memory_ representation of the specified channels.

<h3>Example</h3>

Using the [tutorial](../tut/tut1.md) dataset and [_lunaC_](../luna/args.md) to run the analysis:

```
luna s.lst sig=EEG -o out.db 
  -s "MASK ifnot=NREM2 \
      & RE \
      & TAG NORM/no \
      & PSD spectrum \
      & 1FNORM \
      & TAG NORM/yes \
      & PSD spectrum"
```

Using [_lunaR_](../ext/R.md) to visualize the normalized and raw power spectra (in R):

```
k <- ldb( "out.db" )
```

Looking at the contents of `out.db`, we are interested in the results
of `PSD` stratified by `F` (for power spectra), `CH` and `NORM` (the
[`TAG`](summaries.md#tag) that tracks in the output whether we have
applied the normalization or not):

``` 
lx(k)
```
```
MASK : EPOCH_MASK EPOCH_MASK_NORM 
PSD : CH_NORM B_CH_NORM CH_F_NORM 
RE : BL NORM 
```

Extracting these variables and values:
```
d <- lx( k , "PSD" , "CH" , "F" , "NORM" )

head(d)
```
```
      ID  CH F NORM       PSD
1 nsrr01 EEG 0   no  5.143138
2 nsrr02 EEG 0   no 15.236110
3 nsrr03 EEG 0   no 36.675012
4 nsrr01 EEG 0  yes 17.649644
5 nsrr02 EEG 0  yes 30.562531
6 nsrr03 EEG 0  yes 52.218946
```
Pulling out the pre-normalized spectra:
```
pre <- d[ d$NORM == "no" , ] 
```
and then post normalization:
```
post <- d[ d$NORM == "yes" , ] 
```

Tracking the IDs of the three tutorial individuals, to plot them separately:

```
ids <- unique( d$ID ) 
```

We can then use R basic plotting commands to generate spectra for the
three individuals (columns) corresponding to the raw, unnormalized
spectra showing a 1/_f_ trend (top row), the log-scaled spectra, which
show more of a linear trend (middle row), and the normalized spectra
(bottom row).  Whereas we would not expect these spectra to be
completely flat (e.g. certainly, if bandpass filters have already been
applied to the data), which the range of ~5 to 20 Hz the _baselines_
are relatively flat, and arguably the _"peaks"_ (for `nsrr02` around
13 Hz) are visibly clearer.


```
par( mfrow=c(3,3) , yaxt='n' , mar=c(4,4,1,1) ) 

for (i in ids) {
plot( pre$F[ pre$ID == i ] , pre$PSD[ pre$ID == i ] , 
 type="l" , lwd=2 , col="cornflowerblue" , ylab="Raw" , xlab=i )  }

for (i in ids) {
plot( pre$F[ pre$ID == i ] , 10*log10( pre$PSD[ pre$ID == i ] ), 
 type="l" , lwd=2 , col="goldenrod" , ylab="Log" , xlab=i)  } 

for (i in ids) {
plot( post$F[ post$ID == i ] , post$PSD[ post$ID == i ] , 
 type="l" , lwd=2 , col="olivedrab" , ylab="1/f-norm" , xlab=i)  }
```

![img](../img/1fnorm.png)


## TV

_Applies of fast algorithm for 1D total variation denoising_

The `TV` is a wrapper around the algorithm described
[here](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.372.3867).
In [_lunaC_](../luna/args.md) it operates on EDF channels, modifying
the _in-memory_ representation of the signal.  

!!! note 
    Given that this is not something one typically wants to
    perform on raw physiological signals, a more common use-case may
    be via [_lunaR_](../ext/R.md) however, where the `ldenoise()`
    function provides a simple interface for _any_ time series.  
    It is mentioned here only for completeness.

<h3>Parameters</h3>

| Parameter | Example | Description |
| ---- | ---- | ---- | 
| `sig` | `sig=EEG` | Optional specification of signals (otherwise applied to all signals) |
| `lambda` | `lambda=10` | Smoothing parameter (0 to infinity) |  

See the description of [`ldenoise()`](../ext/R.md#ldenoise) for using
this function with _lunaR_.  Higher values of _lambda_ put more weight
on minimizing variation in the new signal, i.e. producing a more
flattened representation.  The exact choice of _lambda_ will depend on
the numerical scale of the data as well as its variability and the
goal of the analysis.

<h3>Outputs</h3>

No output other than modifying the _in-memory_ representation of the signal.

<h3>Example</h3>

Using [_lunaR_](../ext/R.md) to plot delta power across sleep epochs
and fit a de-noised line using `ldenoise()` (which invokes `TV`), to the 
`nsrr02` individual from the [tutorial](../tut/tut1.md) dataset:

```
library(luna)
sl <- lsl("s.lst")
lattach(sl,2)
```
Get delta power (in dB units) for each sleep epoch:
```
k <- leval( "MASK if=wake & RE & PSD sig=EEG epoch dB" )
d <- k$PSD$B_CH_E
d <- d[ d$B == "DELTA" , ] 
```

Also get sleep stages via the [`lstages()`](../ext/R.md#lstages) function:

```
ss <- lstages()
```

Using `ldenoise()`, we can fit a de-noised line, with _lambda_ of 10 in this particular case:

```
d1 <- ldenoise( d$PSD , lambda = 10 ) 
```

Plotting the original and de-noised versions, also using the
convenience [`lstgcols()`](../ext/R.md#lstgcols) function:

```
plot( d$PSD, col=lstgcols(ss), pch=20, xlab="Sleep Epochs", ylab="Delta power (dB)" ) 
lines( d1 , lwd=5 , col="orange" )
```

![img](../img/tv.png)



## ACF

_Compute the autocorrelation function for a signal_


<h3>Parameters</h3>

| Parameter | Example | Description |
| ---- | ---- | ---- |
| `sig` | `sig=EEG` | Optional specification of signals (otherwise applied to all signals) |
| `lag` | `lag=200` | Maxmimum lag (in sample units) |

<h3>Output</h3>

ACF per channel (strata: `CH` x `LAG`)

| Variable | Description |
| ----- | ------ |
| `SEC` | Lag in seconds |
| `ACF` | Autocorrelation |


<h3>Example</h3>

To estimate the ACF for an example EEG, ECG and EMG channel, for up to 3 seconds lag
(here assuming all channels are sampled at 100 Hz, and so a `lag` of 300):

```
luna s.lst 1 -o out.db -s 'ACF sig=EEG,ECG,EMG lag=300'
```
We can extract the output from the `ACF` function, conditional on `CH` and `LAG` strata
(here putting different channels in different columns (`-c CH`) and different lags in different
rows (`-r LAG`):

```
destrat out.db +ACF -c CH -r LAG > o.txt
```
Plotting these ACF (e.g. from `o.txt`, `SEC.CH_EEG` on the _x_-axis, and `ACF.CH_EEG` on the _y_-axis), we see
strong, regular autocorrelations, with peaks at periodically recuring intervals (top row of plots below).
These would be indicative of artifact in EEG, ECG or EMG channels: indeed, in this particular case (which
is the first [tutorial](../tut/tut1.md) EDF), there is considerable artifact at the end of the recording (i.e.
with spectral peaks at 25 Hz and 12.5 Hz, reflecting harmonics of electrical noise artifacts). 


![img](../img/acf-example.png)

If we repeat the analysis just looking at sleep (N2) epochs (i.e. just
a quick way to chop off the particularly noisy part of the recording),
we see quite different ACF signatures, which are more characteristic
of typical EEG, ECG and EMG respectively.

```
luna s.lst 1 -o out.db -s 'MASK ifnot=NREM2 & RE & ACF sig=EEG,ECG,EMG lag=300'
```
The output from this second run are plotted in the lower panel of the above figure.


<!----
d <- read.table("o.txt",header=T)
d2 <- read.table("o2.txt",header=T)

png(file="~/luna-doc/docs/img/acf-example.png" , res=100 , width=800, height=600) 
par(mfrow=c(2,3))
plot( d$SEC.CH_EEG , d$ACF.CH_EEG , type="l" , ylab="ACF (EEG), all epochs" , xlab="Seconds" , ylim=c(-1,1) ) 
abline( h = 0 , lty=2 ) 
plot( d$SEC.CH_ECG , d$ACF.CH_ECG , type="l" , ylab="ACF (ECG), all epochs" , xlab="Seconds" , ylim=c(-1,1) ) 
abline( h = 0 , lty=2 ) 
plot( d$SEC.CH_EMG , d$ACF.CH_EMG , type="l" , ylab="ACF (EMG), all epochs" , xlab="Seconds" , ylim=c(-1,1) ) 
abline( h = 0 , lty=2 )



plot( d2$SEC.CH_EEG , d2$ACF.CH_EEG , type="l" , ylab="ACF (EEG), N2 epochs only" , xlab="Seconds" , ylim=c(-1,1) ) 
abline( h = 0 , lty=2 ) 
plot( d2$SEC.CH_ECG , d2$ACF.CH_ECG , type="l" , ylab="ACF (ECG), N2 epochs only" , xlab="Seconds" , ylim=c(-1,1) ) 
abline( h = 0 , lty=2 ) 
plot( d2$SEC.CH_EMG , d2$ACF.CH_EMG , type="l" , ylab="ACF (EMG), N2 epochs only" , xlab="Seconds" , ylim=c(-1,1) ) 
abline( h = 0 , lty=2 )
dev.off()

--->