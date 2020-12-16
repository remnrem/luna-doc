# Hypnograms

_Commands to show and summarize elements of sleep macro architecture_ 

These commands require staging information to be present in an
[annotation file](annotations.md).

|Command |Description | 
|---|---|
| [`STAGE`](#stage) | Output sleep stages per epoch |
| [`HYPNO`](#hypno) | Sleep macro-architecture summaries |

## `STAGE`

_Export sleep stage information_

This command simply writes sleep stage information (e.g. as encoded by
an NSRR [XML annotation file](annotations.md#nsrr-xml-files)) to a
database.  Internally, it creates a single annotation class called
`SleepStage`, with instances that correspond to wake, `N1`, `N2`,
`N3`, `REM`, `?` and `L` (_Lights On_).  The [`HYPNO`](#hypno) command
does the same but additionally computes a large number of other
statistics.  Unlike `HYPNO` however, the `STAGES` command can be run
after epochs have been [masked out](masks.md).  In contrast, `HYPNO`
requires the original, entire EDF, in order to produce meaningful
summary statistics on sleep macro architecture. 

!!! warning 
    Currently, Luna and this documentation uses a
    not-always-clearcut mixture of sleep stage terminology, e.g. using
    N2 and NREM2 interchangeably.  Also, if stage NREM4 sleep is
    present, it is summarized explicitly.


<h5>Parameters</h5>

| Parameter | Example | Description |
| ---- | ---- | ---- | 
| `wake`| `wake=W`    | Set the annotation class for _wake_ epochs |
| `N1`  | `N1=NREM1`  | Set the annotaiton class for _N1_ epochs |
| `N2`  | `N2=NREM2`  | Set the annotaiton class for _N2_ epochs |
| `N3`  | `N3=NREM3`  | Set the annotaiton class for _N3_ epochs |
| `REM` | `REM=REM`   | Set the annotaiton class for _REM_ epochs |
| `?`   | `?=UNKNOWN` | Set the annotation class for _unscored/unknown_ epochs |

<h5>Output</h5>

| Variable | Description |
| ---- | ---- |
| `CLOCK_TIME` | Clock time (hh:mm:ss) |
| `MINS`       | Elapsed time from start of EDF (minutes) |
| `STAGE`      | Sleep stage (text value) |
| `STAGE_N`    | Numeric encoding of sleep stage | 


__Numeric stage encoding:__ The `STAGE_N` encoding is designed to help
with quickly plotting a hypnogram: `wake` is 1, `NREM1`, `NREM2` and
`NREM3` are -1, -2 and -3 respectively; `REM` is 0; unknown is 2. As
such, something like the following R command can produce a
hypnogram-like representation of the night:

``` 
plot( d$E , d$STAGE_N ) 
```

<h5>See also</h5>

The [`lstages()`](../ext/R.md#lstages) function in
[_lunaR_](../ext/R.md) provides a quick way to run the `STAGE` command
for a single EDF, returning just a vector of stage names.


## `HYPNO`

_Estimates multiple summary statistics based on the hypnogram_

As [`STAGE`](#stage) does, this command expects manual sleep stages
to be present as an [_annotation_](annotations.md).  `HYPNO` produces
individual-level summary statistics (e.g. total sleep time, percent in
each sleep stage, etc) and epoch-level output (e.g. cumulative elapsed
sleep duration, etc).  In addition, NREM sleep cycles are calculated
based on a modified heuristic following [Feinberg and
Floyd](https://www.ncbi.nlm.nih.gov/pubmed/220659).


__Some definitions:__ If _Lights off_ and _Lights on_ times can be
calculated from the annotations, they are used to calculate sleep
efficiency (`SLP_EFF`), i.e. as total sleep time divided by the total
recording time, where the latter is the duration between _lights off_
and _lights on_.  Any epoch with the stage annotation `L` means that
lights are on, i.e. wake but not part of the recording.  However, many
NSRR annotation files do not have explicit information on when _lights
off/on_ events occurred.  In this case, Luna assumes that _lights off_
corresponds to the start of the EDF, and _lights on_ corresponds to
the end.  This may not be appropriate, however, e.g. if the recording
continued for a long time after the subject woke (as an example, see
the first individual in the [tutorial](../tut/tut1.md) dataset).  Luna
therefore also provides a second estimate of sleep efficiency, as
total sleep time divided by the time from first sleep epoch to final
sleep epoch (`SLP_EFF2`).  Overall, if lights on/off annotations are
not clearly marked, `SLP_EFF2` is likely to be the most robust metric
of sleep efficiency. _Sleep maintenance efficiency_ (`SLP_MAIN_EFF`)
is similar to `SLP_EFF` but the denominator is total recording time
minus sleep latency (i.e. from first sleep epoch to lights on/end of
recording).  _Persistent sleep_ is defined as sleep that follows at
least ten minutes of uninterrupted sleep.



<h5>Parameters</h5>

| Parameter | Example | Description |
| ---- | ---- | ---- | 
| `file` | `file=hypno.txt` | Optionally, read stage information from a file |
| `wake`| `wake=W` | Set the annotation class for _wake_ epochs |
| `N1`  | `N1=NREM1` | Set the annotaiton class for _N1_ epochs |
| `N2`  | `N2=NREM2` | Set the annotaiton class for _N2_ epochs |
| `N3`  | `N3=NREM3` | Set the annotaiton class for _N3_ epochs |
| `REM` | `REM=REM` | Set the annotaiton class for _REM_ epochs |
| `?`   | `?=UNKNOWN` | Set the annotation class for _unscored/unknown_ epochs |


<h5>Output</h5>

Individual-level summary statistics (strata: _none_)

| Variable | Description |
| --- | --- |
| `TST` | Total sleep time |
| `TPST` | Total persistent sleep time |
| `TIB` | Time in bed |
| `TWT` | Total wake time |
| `WASO` | Wake after sleep onset |
| | |
| `LIGHTS_OFF` | Lights off time (hours since midnight) |
| `SLEEP_ONSET` | Sleep onset time (hours since midnight) |
| `SLEEP_MIDPOINT` | Sleep midpoint time (hours since midnight) |
| `LIGHTS_ON` | Lights on time (hours since midnight) |
| `FINAL_WAKE` | Final wake time (hours since midnight) |
| | |
| `SLP_EFF` | Sleep efficiency (see note above) |
| `SLP_EFF2` | Alternate sleep efficiency (see note above) |
| `SLP_MAIN_EFF` | Sleep maintenance efficiency (see note above) |
| `SLP_LAT` | Sleep latency (minutes from lights off) |
| `PER_SLP_LAT` | Persistent sleep latency (mins from lights off) |
| `REM_LAT` | REM latency (minutes from onset of sleep) |
| | |
| `MINS_N1` | Total duration of N1 sleep (mins) |
| `MINS_N2` | Total duration of N2 sleep (mins) |
| `MINS_N3` | Total duration of N3 sleep (mins) |
| `MINS_N4` | Total duration of N4 (NREM4) sleep (mins) |
| `MINS_REM` | Total duration of REM sleep (mins) |
| `PCT_N1` | Proportion N1 of total sleep time |
| `PCT_N2` | Proportion N2 of total sleep time |
| `PCT_N3` | Proportion N3 of total sleep time |
| `PCT_N4` | Proportion N4 (NREM4) of total sleep time |
| `PCT_REM` | Proportion REM of total sleep time |
| | |
| `NREMC` | Number of sleep cycles |
| `NREMC_MINS` | Mean duration of each sleep cycle |


Epoch-level output (strata: `E`)

| Variable | Description |
| --- | --- |
|`CLOCK_HOURS`| Start time of epoch (hours since midnight) |
|`CLOCK_TIME`| Start time of epoch (hh:mm:ss) | 
|`MINS`|  Start time of epoch (minutes since start of EDF) | 
|`STAGE`| Text description of sleep stage |
|`STAGE_N`| Numeric encoding of sleep stage (see `STAGE` example, above) | 
|`PERSISTENT_SLEEP`| Flag to indicate persistent sleep (more than 10 minutes of sleep has elapsed) |
|`WASO`| Flag to indicate wake after sleep onset |
| | |
|`E_N1`| Cumulative elapsed N1 sleep (minutes) |
|`E_N2`| Cumulative elapsed N2 sleep (minutes) |
|`E_N3`| Cumulative elapsed N3 sleep (minutes) |
|`E_REM`| Cumulative elapsed REM (minutes) |
|`E_SLEEP`| Cumulative elapsed sleep (minutes) |
|`E_WAKE`| Cumulative elapsed wake (minutes) |
|`E_WASO`| Cumulative elapsed WASO (minutes) |
| | |
|`PCT_E_N1`| Cumulative elapsed N1 as proportion of total N1 sleep |
|`PCT_E_N2`| Cumulative elapsed N2 as proportion of total N2 sleep |
|`PCT_E_N3`| Cumulative elapsed N3 as proportion of total N3 sleep |
|`PCT_E_REM`| Cumulative elapsed REM as proportion of total REM sleep |
|`PCT_E_SLEEP`| Cumulative elapsed sleep as proportion of total sleep |
| | |
|`FLANKING_SIM` | The number minimum number of similarly-staged epochs going either forwards or backwards | 
|`N2_WGT`| Score to indicate ascending versus descending N2 sleep (see below) | 
|`NEAREST_WAKE`| Number of epochs (forward or backwards) since nearest wake epoch (see below) |
|`NREM2REM`| Number of epochs from this N2 epoch to the N2/REM transition (see below) |
|`NREM2REM_TOTAL`| Total number of contiguous N2 epochs until a REM transition (see below) |
|`NREM2WAKE`|  Number of epochs from this N2 epoch to the N2/Wake transition (see below) |
|`NREM2WAKE_TOTAL`| Total number of contiguous N2 epochs until a Wake transition (see below) |
| | |
|`CYCLE`| Cycle number, if this epoch is in a sleep cycle | 
|`CYCLE_POS_ABS`| Absolute position of this epoch in the current NREM cycle (mins) |
|`CYCLE_POS_REL`| Relative position of this epoch in the current NREM cycle (0-1) | 
|`PERIOD`| Cycle period: `NREMP` or `REMP`, or missing if not in a cycle |


Information on each NREM sleep cycle (strata: `C`)

| Variable | Description |
| --- | --- |
| `NREMC_START` | First epoch number of this NREM cycle |
| `NREMC_MINS` | Total duration of this cycle (mins) |
| `NREMC_NREM_MINS` | Duration of NREM in this cycle (mins) |
| `NREMC_REM_MINS` | Duration of REM in this cycle (mins) |
| `NREMC_OTHER_MINS` | Minutes of wake and unscored epochs |


<h5>Example</h5>

Here we run `HYPNO` on `nsrr02` from the [tutorial](../tut/tut1.md) data:
```
luna s.lst nsrr02 -o out.db -s "HYPNO" 
```

The baseline, individual-level output for `HYPNO` contains summaries
such as total sleep time (`TST`).  Here we use Luna's `behead` utility 
to view the output in a more human-readable fashion:

```
destrat out.db +HYPNO  | behead
```

```
                       ID   nsrr02              
               FINAL_WAKE   5.22667             
                LIGHTS_ON   7.26                
               LIGHTS_OUT   21.3017             
                  MINS_N1   5.5                 
                  MINS_N2   199.5               
                  MINS_N3   92.5                
                  MINS_N4   0                   
                 MINS_REM   60                  
                    NREMC   5                   
               NREMC_MINS   77.1                
                   PCT_N1   0.0153846153846154  
                   PCT_N2   0.558041958041958   
                   PCT_N3   0.258741258741259   
                   PCT_N4   0                   
                  PCT_REM   0.167832167832168   
              PER_SLP_LAT   63                  
                  REM_LAT   43.5                
           SLEEP_MIDPOINT   1.64333             
              SLEEP_ONSET   22.06               
                  SLP_EFF   59.7826086956522    
                 SLP_EFF2   83.1395348837209    
                  SLP_LAT   45.5                
             SLP_MAIN_EFF   64.7058823529412    
                      TIB   597.5               
                     TPST   231                 
                      TST   357.5               
                      TWT   240                 
                     WASO   72.5                
```

As indicated above, this individual had five sleep cycles (`NREMC`).
We can view details about each cycle by querying the `C` stratum of
`out.db`:

```
destrat  out.db +HYPNO -r C
```

```
ID      C  NREMC_MINS NREMC_NREM_MINS NREMC_OTHER_MINS NREMC_REM_MINS NREMC_START
nsrr02  1  72.5       34              12.5             26             92
nsrr02  2  133        118.5           5                9.5            237
nsrr02  3  42         40              2                0              503
nsrr02  4  105.5      72              9                24.5           675
nsrr02  5  32.5       31              1.5              0    	      887
```

Finally, we can view epoch-level data (here just a few rows, for a subset of the available variables):
```
ID       E     CLOCK_TIME  E_SLEEP  STAGE
... skipped ...
nsrr02   89    22:02:06    0        wake
nsrr02   90    22:02:36    0        wake
nsrr02   91    22:03:06    0        wake
nsrr02   92    22:03:36    0        NREM2
nsrr02   93    22:04:06    0.5      NREM2
nsrr02   94    22:04:36    1        wake
nsrr02   95    22:05:06    1        wake
nsrr02   96    22:05:36    1        wake
nsrr02   97    22:06:06    1        wake
nsrr02   98    22:06:36    1        NREM2
nsrr02   99    22:07:06    1.5      NREM2
nsrr02   100   22:07:36    2        NREM2
nsrr02   101   22:08:06    2.5      NREM2
nsrr02   102   22:08:36    3        NREM2
nsrr02   103   22:09:06    3.5      wake
nsrr02   104   22:09:36    3.5      wake
nsrr02   105   22:10:06    3.5      wake
nsrr02   106   22:10:36    3.5      wake
... cont'd ...
```


__Time encoding:__ Several variables use an _hours since midnight_
24-hour encoding, so that 20.5, for example, means 8:30pm, etc.

__Stage nomenclature:__ We use the newer format of N1, N2, and N3
sleep for variable names, although if NREM4 sleep has been scored it
will be reported as NREM (although, for "consistency" the variable
name is `N4` rather than `NREM4`).

__Ascending and descending N2:__ _Ascending_ N2 sleep is defined as N2
sleep that follows N3 and precedes N1, Wake or REM.  In contrast,
_descending_ N2 sleep follows wake, N1 or REM sleep and precedes N3
sleep.  The epoch-level `N2_WGT` variable (which is only defined for
N2 sleep) is an index ranging from -1 to +1 to quantify this.
Specifically, it is calculated by looking at the first _k_ non-N2
epochs that come before and after that N2 epoch.  (By default, _k_ is
10.)  N3 epochs are scored +1 if they are prior to this epoch,
otherwise -1. Wake, N1 and REM epochs are scored -1 if they come
before this epoch, otherwise +1.  The final `N2_WGT` score is the
average of these scores.  Epochs that are, e.g., less than -0.5 might
be called _descending_, whereas epochs that are greater than +0.5
might be called _ascending_.


__Stage transition metrics:__ Luna currently has a set of somewhat
idiosyncratic and incomplete stage transition metrics, that were
implemented for the specific purpose of considering transitions out of
N2 sleep into REM or wake.  As they stand, they are unlikely to be of
use for most users, but are noted here for completeness.  _In the
future we will add more complete epoch-level metrics and individual
summaries (i.e. a basic transition matrix)._ As it stands, for N2
epochs followed by REM, `NREM2REM` shows the number of N2 epochs until
a REM epoch; `NREM2REM_TOTAL` shows the total number of contiguous N2
epochs up until that point.  The N2/wake variables are defined
similarly.

```
 STAGE   NREM2REM   NREM2REM_TOTAL
  ..............
  ... cont'd ...
  ..............
    REM         0                0
  NREM2         5                5
  NREM2         4                5
  NREM2         3                5
  NREM2         2                5
  NREM2         1                5
    REM         0                0
  ..............
  ... cont'd ...
  ..............
```