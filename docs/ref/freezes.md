# Data freezes and caches

_Saving/reverting to snapshots of the dataset and derived metrics_


| Command | Description | 
| ---- | ------ | 
| [`FREEZE`](#freeze) | Make a named freeze of the current datatset |
| [`THAW`](#thaw) | Revert to a previous data freeze |
| [`CLEAN-FREEZER`](#clean-freezer) | Empty the freezer | 
| [`CACHE`](#cache) | Cache operations |


_Freezes_

As Luna processes an EDF, it first reads the header, followed by
some number of records and/or channels from the body of the EDF, on an
_as needed_ basis.  Subsequently, the signal data can be transformed
or augmented _in memory_, e.g.  by resampling, filtering and other
transformations, adding or dropping channels, and masking/dropping certain epochs.
  
There are two key commands to 1) take a snapshot of the current state
of the dataset (`FREEZE`) and then 2) later revert back to that state
(`THAW`). This effectively enables _rewind_ or _undo_
functionality after changes have been made to an _in-memory_ EDF.
That is, `FREEZE`/`THAW` allows for more efficient single-run
pipelines (i.e.  to avoid unnecessarily having to reload the same EDF
from disk multiple times, say if restricting to N2 epochs only with a
mask, but then reverting to the full dataset and subsequently
restricting to N3 epochs only, etc.)

_Caches_

Although conceptually different, this page also describes _caches_, a
second internal mechanism that tracks state inside a single Luna run.
In this case case, the cache tracks either values generated internally
by a particular command (e.g. `SPINDLE cache-peaks`) or generic Luna
outputs.  The most obvious use-case for working with the cache is the [`PREDICT`](predict.md#predict)
command.


## FREEZE

_Make a freeze (snapshot) of the current dataset_

This saves the current state of the in-memory EDF (signal data),
including the index of all interval-based annotations, and epochs set
and the status of any epoch-based masks.  Freezes are retained for the duration of
operations on the currently loaded EDF - i.e. when running Luna on multiple EDFs,
freezes are not saved after the last command has finished for that EDF.


<h3>Parameters</h3>

| Option | Example | Description |
| ---- | ----- | ----- |
| `tag` | `tag=f1`  | Make freeze called `f1` |
| `preserve-cache` | Retain any caches from the current freeze when thawing an prior freeze | 

Alternatively, `tag` can be dropped and the freeze name is specified directly after `FREEZE`.

By default, `THAW` wipes any changes to the cache made after the
paired `FREEZE`, unless `preserve-cache` is specified.

<h3>Output</h3>

None (other than adding a freeze to the freezer).

<h3>Example</h3>

To save a freeze at any point in time and label it, for example, `f1`:
``` 
    FREEZE tag=f1
```
or omitting the `tag` option, simply:

``` 
    FREEZE f1
```
You can make multiple freezes with different labels, and then revert to a particular freeze with
the `THAW` command:
```
    THAW tag=f1
```
or equivalently:
```
    THAW f1
```

To give a fuller example:

```
% original signal C3, all epochs -
MATRIX sig=C3 file=a0.txt
 
% save as F1
FREEZE F1
 
% change stuff (drop some epochs, add a new signal):
%  output both signals, new C3_V2, a copy of C3
%  but starting from 'epoch 200' of the original

MASK epoch=200-250
RE
COPY sig=C3 tag=V2
DESC
MATRIX sig=C3,C3_V2 file=a1.txt
 
% save this changed version as F2

FREEZE F2
 
% bring back the original:
%  the "C3_V2" does not exist here,
%  so 'a2.txt' should only have one col

THAW F1
DESC
MATRIX sig=C3,C3_V2 file=a2.txt
 
% finally, bring back the second (smaller) version F2
%  this has C3_V2, so this should be output by MATRIX
THAW F2
DESC
MATRIX sig=C3,C3_V2 file=a3.txt
```

When running, we see the key steps logged in the console, starting with
making the first freeze `F1`:

``` 
 CMD #2: FREEZE
   options: sig=* tag=F1
  freezing state, with tag F1
  copied 33710 records
  currently 1 freeze(s): F1
```
Making the second (smaller freeze) `F2`:
```
 CMD #8: FREEZE
   options: sig=* tag=F2
  freezing state, with tag F2
  copied 1530 records
  currently 2 freeze(s): F1 F2
```
and then reverting to the first freeze:
```
 CMD #9: THAW
   options: sig=* tag=F1
  thawing previous freeze F1
  old dataset   : 1530 records, 41 signals, 13 annotations
  thawed freeze : 33710 records, 39 signals, 13 annotations
  copied 33710 records
```
and finally, back to the second:
```
 CMD #12: THAW
   options: sig=* tag=F2
  thawing previous freeze F2
  old dataset   : 33710 records, 39 signals, 13 annotations
  thawed freeze : 1530 records, 41 signals, 13 annotations
  copied 1530 records
```

The outputs are as expected (i.e. `a0.txt` and `a2.txt` have a similar row count, as do `a1.txt` and `a3.txt`):

``` 
wc -l a*.txt
```
```
 8624641 a0.txt
  391681 a1.txt
 8624641 a2.txt
  391681 a3.txt
```

Looking at the dumped `MATRIX` outputs:
```
head a0.txt a2.txt
```
```
==> a0.txt <==
ID    E    S    SP   T          C3
id001 1    0    0    0          -117.634607462
id001 1    0    1    0.00390625 -117.306477455
id001 1    0    2    0.0078125  -129.181658656
id001 1    0    3    0.01171875 -125.400350958
id001 1    0    4    0.015625   -129.759792477
id001 1    0    5    0.01953125 -127.197253376
id001 1    0    6    0.0234375  -133.447348745
id001 1    0    7    0.02734375 -131.212939651
id001 1    0    8    0.03125    -129.978545815
  
==> a2.txt <==
ID    E    S    SP   T          C3
id001 1    0    0    0          -117.634607462
id001 1    0    1    0.00390625 -117.306477455
id001 1    0    2    0.0078125  -129.181658656
id001 1    0    3    0.01171875 -125.400350958
id001 1    0    4    0.015625   -129.759792477
id001 1    0    5    0.01953125 -127.197253376
id001 1    0    6    0.0234375  -133.447348745
id001 1    0    7    0.02734375 -131.212939651
id001 1    0    8    0.03125    -129.978545815
```

In contrast, `a1.txt` and `a3.txt` (based on the second `F2` freeze) start later and show different values and
have the extra `C3_V2` channel:
```
head a1.txt a3.txt
```
```
==> a1.txt <==
ID    E    S    SP   T               C3              C3_V2
id001 200  5970 0    5970            -43.7584802014  -43.7584802014
id001 200  5970 1    5970.00390625   -47.6804150454  -47.6804150454
id001 200  5970 2    5970.0078125    -48.5866788739  -48.5866788739
id001 200  5970 3    5970.01171875   -50.2898298619  -50.2898298619
id001 200  5970 4    5970.015625     -52.8367437247  -52.8367437247
id001 200  5970 5    5970.01953125   -58.8680857557  -58.8680857557
id001 200  5970 6    5970.0234375    -56.6493018997  -56.6493018997
id001 200  5970 7    5970.02734375   -57.2743114366  -57.2743114366
id001 200  5970 8    5970.03125      -57.7899443046  -57.7899443046

==> a3.txt <==
ID    E    S    SP   T               C3              C3_V2
id001 200  5970 0    5970            -43.7584802014  -43.7584802014
id001 200  5970 1    5970.00390625   -47.6804150454  -47.6804150454
id001 200  5970 2    5970.0078125    -48.5866788739  -48.5866788739
id001 200  5970 3    5970.01171875   -50.2898298619  -50.2898298619
id001 200  5970 4    5970.015625     -52.8367437247  -52.8367437247
id001 200  5970 5    5970.01953125   -58.8680857557  -58.8680857557
id001 200  5970 6    5970.0234375    -56.6493018997  -56.6493018997
id001 200  5970 7    5970.02734375   -57.2743114366  -57.2743114366
id001 200  5970 8    5970.03125      -57.7899443046  -57.7899443046
```

 
In practice, one might use `FREEZE` and `THAW` to apply a uniform set of manipulations
to the whole, continuous sample (e.g. resampling or filtering) and then extract out different
subsets for analyses:

```
% run common operations on whole, continuous recording
RESAMPLE sig=C3,C4 sr=128
FILTER sig=C3,C4 bandpass=0.3,35 tw=0.5 ripple=0.02  

% drop all wake eppchs, keep only C3 and C4
MASK if=W
RE
SIGNALS keep=C3,C4

% make a freeze
FREEZE tag=F1
 
% ---- N2 epochs
TAG STG/N2
MASK ifnot=N2 & RE
CHEP-MASK ep-th=3,3
CHEP epochs & RE
PSD spectrum dB
 
% ---- N3 epochs, i.e. THAWing F1 first
THAW tag=F1
TAG STG/N3
MASK ifnot=N3 & RE
CHEP-MASK ep-th=3,3
CHEP epochs & RE
PSD spectrum dB
 
% ---- REM, i.e. re-THAWing F1
THAW tag=F1
TAG STG/R
MASK ifnot=R & RE
CHEP-MASK ep-th=3,3
CHEP epochs & RE
PSD spectrum dB
```

Note that the `TAG` commands are necessary to differentiate the output of
`PSD` (or any other commands) from the different freezes, i.e.  here
`PSD` output is stratified by `CH` x `F` x `STG`, for example, not
just `CH` x `F`:

```
destrat out.db +PSD -r F CH -c STG -v PSD  | head
```

```
ID    CH   F    PSD.STG_N2           PSD.STG_N3           PSD.STG_R
id001 C3   0.5  26.4753553410174     29.7908867326006     24.2826264205783
id001 C3   0.75 26.9078210841686     31.4015320540544     23.8294058549926
id001 C3   1    26.0094772619657     31.518945161215      22.35240483615
id001 C3   1.25 24.9041682579044     30.9866797641856     21.0973533542852
id001 C3   1.5  23.4953988040375     29.5269909464068     19.9228334881858
id001 C3   1.75 21.7463729204331     27.7504946194862     18.9047617988682
id001 C3   2    20.1703821679443     26.0024225614884     17.7828403799221
id001 C3   2.25 19.0448488790284     24.7055650800498     16.9523484034449
id001 C3   2.5  17.9078051859294     23.4408530496513     16.3539538209027
```

!!! info
    One thing you can’t do is `FREEZE`, change EDF record size and then `THAW`, as Luna forces a `WRITE`
    after `RECORD-SIZE` and moves to the next EDF.


## THAW

_Revert to a previous data freeze_

If the named freeze does not exist (i.e. from a prior `FREEZE` command), Luna will give an error.

If you add the option `remove`, this will delete the freeze after
retrieving it, meaning that it cannot be used again, which can help to
save memory usage. Typically, this will not be necessary (note that
all freezes are deleted after moving to process the next EDF in any
case).  Still, because of this option, it is not possible to name a
freeze as the label `remove`, as it is a special, reserved keyword in
this context.

!!! hint
    It is not necessary to manually use `THAW remove` at the end of a
    string of commands - i.e.  as noted above, all freezes are deleted
    when moving to the next EDF.

<h3>Parameters</h3>

| Option | Example | Description |
| ---- | ----- | ----- |
| `tag` | `tag=F1`  | Revert to a freeze called `F1` |
| `remove` |   | Delete the freeze after reverting to it |

<h3>Output</h3>

None (other than changing the state of the current in-memory EDF).

<h3>Example</h3>

See the example above given for the `FREEZE` command.

## CLEAN FREEZER

_Empty the freezer_

The `CLEAN-FREEZER` command clears all data freezes from the freezer.
Typically, you should not need to call this (as it is called
implicitly when one is finished processing a given file).  If you are
working with _extremely_ large files and memory becomes an issue, then
this command might be useful, i.e. to manage intermediate memory storage.

<h3>Parameters</h3>

None.

<h3>Output</h3>

None (other than changing the state of the current in-memory EDF).


## CACHE

_Use/display an internal cache for inter-command communication_

!!! warning
    This is an advanced command used primarily in conjunction with other specific Luna commands: most
    users can likely safely ignore it.  The information here is primarily for our own internal reference.
    
Internally, Luna uses a _cache_ mechanism to store information in
several ways, such that it can be shared between different commands
running on the same dataset/EDF. One exemplar use of the cache mechanism is
when working with the [`PREDICT`](predict.md#predict) command, which expects
_features_ to be derived from one or more Luna commands (e.g. spectral
power) which are then combined with a _model_ (read by the `PREDICT`
command) to make a prediction.

<h3>Parameters</h3>

| Option | Description |
| ----- | ----- |
| `cache` | The cache name |
| `record` | Set up the cache to track certain output, using a `command,variable,{strata}` form |
| `clear` | Clears the cache |
| `import` | Imports a cache from long-format output for the current individual |
| `factors`, `v` | Used with `import`, see below |
| `dump` |  Dump cache contents to the console (with `num`, `int`, `text` or `bool`) |

Various other commands use the cache indirectly: for example, the
`SPINDLES` command has a `cache-peaks` option to store the sample
points of spindle peaks (see below for an example).

Cache types are either _numeric_ (`num`), _integer_ (`int`), _textual_
(`text`) or true/false (`bool`).  Caches from `record` are always
_numeric_, but caches set by other commands may use different types.

<h3>Outputs</h3>

No formal outputs, except from `dump` which prints to the console.

<h3>Example</h3>

To illustrate how the cache can capture (`record`) specific outputs and store them in the cache, consider this script:
```
CACHE cache=p1 record=HEADERS,CH,SR
HEADERS
CACHE dump num=p1 
```

It first sets up a cache called `p1` and instructs it to record any
`SR` variables emitted from a subsequent `HEADERS` command; as `SR` is
a channel-specific variable, it always is stratified by `CH`
(channel), as described in the [`HEADERS`](summaries.md#headers)
documentation.  That is, the `record` option takes at least two
arguments (the command, the variable name) followed by any
"stratifying" factors that define that variable (in this case just
`CH`). Finally, it dumps the cache to the console, as seen here:

```
 CMD #4: CACHE
   options: dump num=p1 sig=*
cache: p1[num]
strata: CH=A1
value: HEADERS:SR=128
strata: CH=A2
value: HEADERS:SR=128
strata: CH=ABDO_EFFORT
value: HEADERS:SR=32
strata: CH=AIRFLOW
value: HEADERS:SR=32
strata: CH=C3
value: HEADERS:SR=128
...
```

Above we see that `AIRFLOW` has a sample rate of 32 Hz, whereas `C3`
has a sample rate of 128 Hz, for example. The same information can be
extracted from the output of the above run, assuming it was saved to
`out.db`:

```
destrat out.db +HEADERS -r CH  > o.1
```
```
head o.1
```
```
ID   CH     DMAX    DMIN  PDIM  PMAX   PMIN  POS      SENS   SR   TRANS TYPE
id1  C3    32767  -32768    mV  0.51  -0.51    1  1.55e-05  128  EEG C3  EEG
id1  C4    32767  -32768    mV  0.51  -0.51    2  1.55e-05  128  EEG C4  EEG
id1  A1    32767  -32768    mV  0.51  -0.51    3  1.55e-05  128  EEG A1  REF
id1  A2    32767  -32768    mV  0.51  -0.51    4  1.55e-05  128  EEG A2  REF
id1  LOC   32767  -32768    mV  0.51  -0.51    5  1.55e-05  128 EEG LOC  EOG
id1  ROC   32767  -32768    mV  0.51  -0.51    6  1.55e-05  128 EEG ROC  EOG
id1  ECG2  32767  -32768    mV  1.02  -1.02    7  3.11e-05  256     ECG  ECG
...
```

This "long-format" output can be _imported_ back into the cache as follows:
```
luna s.lst -s ' CACHE cache=p1 import=o.1 factors=CH v=SR & CACHE dump num=p1 '
```
Note that if you did not specify the appropriate `factors` or `v` (variable) options,
it would 1) report for all variables, not just `SR`, but also 2) not track which
values belong to which channels, and so inputs would overwrite each other, with the
final dump just showing the final row of `o.1` :
```
luna s.lst -s ' CACHE cache=p1 import=o.1 & CACHE dump num=p1 '
```
```
value: DMAX=32767
value: DMIN=-32768
value: PMAX=120
value: PMIN=40
value: POS=28
value: SENS=0.00122072
value: SR=1024
```
where the last row happens to be:
```
ID      CH   DMAX    DMIN  PDIM  PMAX  PMIN  POS     SEN  S   SR   TRANS TYPE
id1  HRate  32767  -32768   bpm   120    40   28  0.00122   1024    Off   HR
```

Note that only numeric values are included in the cache here.

For the `SPINDLES` example, the `cache-peaks` option generates a integer cache:
```
luna s.lst -o out.db -s 'MASK ifnot=N2 & RE & SPINDLES sig=C3 cache-peaks=p1 & CACHE dump int=p1 ' 
```

```
 CMD #4: CACHE
   options: dump int=p1 sig=*
cache: p1[int]
strata: CH=C3
strata: F=13.5
value: (695 element vector)
```

The is not directly manipulated, but rather may be passed to other Luna commands, e.g. `TLOCK`.  


!!! info "Preserving the cache"
    If using the cache to store derived metrics, you may often want to stop a [`RESTRUCTURE`](masks.md#restructure)
    or [`THAW`](#thaw) operation from wiping the cache, which is the default behavior.  The reason for this is that
    some cache values may point to intervals of signals using sample-points as the index, which could potentially
    be invalidated after either of these two operations if the number of epochs changes.  However, for metrics
    that will not be impacted by this (e.g. storing outputs for `PREDICT`), then you can preserve the cache
    by adding `preserve-cache` to any `RE` or `THAW` command.
    