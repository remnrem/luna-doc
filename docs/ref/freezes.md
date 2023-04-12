# Data freezes

_Saving/reverting to snapshots of the dataset_

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


| Command | Description | 
| ---- | ------ | 
| [`FREEZE`](#freeze) | Make a named freeze of the current datatset |
| [`THAW`](#thaw) | Revert to a previous data freeze |

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

Alternatively, `tag` can be dropped and the freeze name is specified directly after `FREEZE`.

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


