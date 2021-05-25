# lunaR : example usage

Here, we will briefly step through some of the same steps of the
[tutorial](../../tut/tut3.md#spectral-and-spindle-analyses) using _lunaR_
instead of `luna`.  See also [this tutorial page](../../tut/tut4.md) for a
fuller application of _lunaR_ to the tutorial data.

We assume that you are running R and the current working directory is
the one where `tutorial.zip` was unzipped.

```
library(luna)
```

Attach the sample-list with [`lsl()`](#lsl):
```
sl <- lsl("s.lst")
```
```
3 observations in s.lst 
```

Attach just the second individual, `nsrr02`, with [`lattach()`](#lattach):
```
lattach( sl , "nsrr02" ) 
```
```
nsrr02 : 14 signals, 10 annotations, 09:57:30 duration
```

We will then use sequential [`leval()`](#leval) commands, to restrict
analysis to N2 epochs only, band-pass filter the EEG signal,
automatically scan for epochs with high levels of artifact, and then
estimate the PSD.  First, we mask out all epochs that are not N2 sleep:

```
leval( "MASK ifnot=NREM2" )
```
```
nsrr02 : 14 signals, 10 annotations, 09:57:30 duration, 399 unmasked 30-sec epochs, and 796 masked
```
Next, we [restructure](../../ref/masks.md#restructure) the dataset, to actually remove the non-N2 epochs:
```
leval( "RE" )
```
```
nsrr02 : 14 signals, 10 annotations, 03:19:30 duration, 399 unmasked 30-sec epochs, and 0 masked
```
Next, we restrict analyses to the EEG channel, by dropping all other channels:
```
leval( "SIGNALS keep=EEG" )
```
```
nsrr02 : 1 signals, 10 annotations, 03:19:30 duration, 399 unmasked 30-sec epochs, and 0 masked
```
Next, we apply a 0.3-35 Hz bandpass filter:
```
leval( "FILTER bandpass=0.3,35 ripple=0.02 tw=0.5" )
```

Next, we scan for artifacts.  Note, for the prior [`leval()`](#leval) commands,
we have not been explicitly saving any returned values, as the prior
commands typically do not return values of interest.  Here we will
save return values (in a list named `k0`) for future use, however:

```
k0 <- leval( "ARTIFACTS mask & SIGSTATS epoch mask threshold=3,3,3" ) 
```
```
nsrr02 : 1 signals, 10 annotations, 03:19:30 duration, 368 unmasked 30-sec epochs, and 31 masked
```

We see that 31 epochs have been masked (out of 399).  You can examine
the returned `k0` list (with [`lx()`](#lx) or just directly), to see
the other output of these commands.  We next restructure the dataset one
more time to remove these masked epochs:
 
```
leval( "RE" )
```
```
nsrr02 : 1 signals, 10 annotations, 03:04:00 duration, 368 unmasked 30-sec epochs, and 0 masked
```
Finally, we run the [`PSD`](../../ref/power-spectra.md#psd) command, with the `spectrum` option:

```
k <- leval( "PSD spectrum max=30" ) 
```
The returned list has three distinct strata, or data-frames:
```
lx(k)
```
```
PSD : CH B_CH CH_F 
```

Of primary interest, is spectral power (`PSD` variable) stratified by
frequency (`F`) and channel (`CH`, although note that in this
particular analysis we only have a single channel, `EEG`):

```
k$PSD$CH_F
```
```
    CH     F        PSD
1  EEG  0.00  1.1965277
2  EEG  0.25 30.2506337
3  EEG  0.75 62.8448068
4  EEG  1.25 44.4322119
5  EEG  1.75 36.9482211
6  EEG  2.25 26.4534428
7  EEG  2.75 19.9197660
8  EEG  3.25 15.1230627
9  EEG  3.75 12.5903582
10 EEG  4.25 10.3720280
11 EEG  4.75  8.4881753
12 EEG  5.25  7.4004198
13 EEG  5.75  6.3812719
14 EEG  6.25  5.8546718
15 EEG  6.75  5.6997780
16 EEG  7.25  5.1773553
17 EEG  7.75  4.9549092
18 EEG  8.25  4.6586603
19 EEG  8.75  4.1608020
20 EEG  9.25  3.7108913
21 EEG  9.75  3.6358155
22 EEG 10.25  3.2967826
23 EEG 10.75  3.0307075
24 EEG 11.25  2.6924206
25 EEG 11.75  2.7074242
26 EEG 12.25  3.8733526
27 EEG 12.75  4.2031480
28 EEG 13.25  3.2350931
29 EEG 13.75  1.7120545
30 EEG 14.25  1.0612353
31 EEG 14.75  0.7444655
32 EEG 15.25  0.5226373
33 EEG 15.75  0.4335665
34 EEG 16.25  0.3993188
35 EEG 16.75  0.3486289
36 EEG 17.25  0.3470138
37 EEG 17.75  0.3217134
38 EEG 18.25  0.2806634
39 EEG 18.75  0.2613016
40 EEG 19.25  0.2383480
41 EEG 19.75  0.2260711
```
Plotting this:
```
d <- k$PSD$CH_F
plot( d$F , log(d$PSD) , xlab = "Frequency (Hz)" , ylab = "Power" , col="blue" , lwd=2 , type="l" )
```

![img](../../img/r-psd.png)

!!! hint
    You can re-run adding the `max=30` parameter to the `PSD` command to obtain results on the same frequency scale as in the tutorial.


That completes our simple introduction to using _lunaR_.  As noted
above, see the final [tutorial page](../../tut/tut4.md) for a fuller set
of examples of using _lunaR_.  The [next page](ref.md) contains
reference documentation for each _lunaR_ function.


