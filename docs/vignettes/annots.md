# Working with discontinuous EDFs and annotation data

_How Luna handles discontinuous EDFs and best practices for working with annotations_

## EDF types 

Luna can work with three types of EDF:

| Format | Description |
| ---- | ---- |
| EDF | Assumes continuous recording, starting at the EDF header start time as 0 seconds elapsed time |
| EDF+C | EDF+ (that may contain `EDF Annotations`) but is a continuous recording, i.e. as for a standard EDF, a single interval that starts at the EDF header start time as 0 seconds elapsed time |
| EDF+D | EDF+ (that may contain `EDF Annotations`) but is (potentially) a discontinuous recording, i.e. containing more than one non-contiguous segment.  Here, the time of each record is tracked as an `EDF Annotations` channel, and so each record _knows_ when it occurs (relative to the EDF header start time) |

## Casting between EDF types

Internally, after _any_ restructuring (i.e. removing epochs, dropping channels) Luna will
represent the data as an EDF+D, in that each record has an associated
time that tracks when it occurs (in seconds past the EDF start time).
However, we make a distinction between restructuring that results in
_truly discontinuous_ recordings (i.e. more than one non-contiguous
segment) versus those that might still be a single, contiguous segment
(i.e. if only the first and/or last epochs are removed).  In the latter
case, the data can still be represented in a _continuous_ manner
without any loss of information (i.e. EDF or EDF+C rather than EDF+D).
This can be beneficial, as it takes time on loading an EDF+ to first
extract and parse all the `EDF Annotations` tracks (and the time track is
encoded as one of these in EDF+).

On writing a new EDF with the [`WRITE`
command](../ref/outputs.md#write), Luna will see whether the file can
be represented as an EDF, EDF+C or EDF+D, as follows (here, assuming the original input
was an EDF):

| Condition | Output format | EDF start-time |
| ----- | ----- | ----- | 
| No prior restructuring with `RE` | Standard EDF | Same as original EDF start-time |
| Restructured but only one segment | Standard EDF if no other `EDF Annotations`, otherwise EDF+C | Time of first observed record | 
| Restructured with more than one segment (i.e. truly discontinuous) | EDF+D | Same as original EDF start-time |

These default behaviors of `WRITE` can be modified by various options:

- `force-edf` will drop any `EDF Annotations` and write as a continuous, standard EDF, i.e. ignoring any potential true discontinuities in the data
- `EDF+D` forces `WRITE` to output a EDF+D rather than EDF+C or EDF, no matter whether there are actual discontinuities or not

## Toy dataset

For this vignette, consider a standard EDF `test01.edf` with a
matching annotation file `test01.xml` that contains sleep stage
annotations (this is an XML in [NSRR format](../nsrr.md)).  Assuming
these files are present in the working folder, we'll first create a
new sample list `s.lst`, e.g:

```
luna --build . > s.lst
```
```
cat s.lst
```
```
test01	./test01.edf	./test01.xml
```

We can confirm that this original file is a standard EDF in a number of ways.  First, on reading Luna will note the format in the console:
```
luna s.lst -s DESC
```
```
Processing: test01 [ #1 ]
 duration: 10.28.30 | 37710 secs ( clocktime 20.08.36 - 06.37.05 )

 signals: 26 (of 26) selected in a standard EDF file:
```
Alternatively, if working with a large number of files, the `HEADERS` command track the file type:
```
luna s.lst -o out.db -s HEADERS
```
```
destrat out.db +HEADERS | behead
```
```
                       ID   test01              
                   EDF_ID   .                   
                 EDF_TYPE   EDF                 
                       NR   37710               
                       NS   26                  
                   NS_ALL   26                  
                  REC_DUR   1                   
               START_DATE   01.01.85            
               START_TIME   20.08.36            
              TOT_DUR_HMS   10:28:30            
              TOT_DUR_SEC   37710               
```
We see the `EDF_TYPE` variable is set to `EDF` (rather than `EDF+C` or `EDF+D`).

## Time-tracking in Luna

Here we use the `WRITE` command to output the same EDF under different conditions, as either EDF, EDF+C or EDF+D depending
on the state of the internal EDF (reflecting whether epochs have been masked/removed).

First, we simply write the whole EDF with any changes: this will
always generate a standard EDF as output:

```
luna s.lst -s 'WRITE edf-tag=v0'
```

Second, we remove only the first epoch via a `MASK` command, and then restructure (via `RE`).  Internally, the EDF will
be represented as discontinuous by the time Luna executes the `WRITE` command.  However, on writing Luna will see that
this EDF contains only a single, continuous interval of time (i.e. from epoch 2 to the final epoch) and so will write
as a continuous EDF (as there are no other EDF Annotations channel present in the EDF, other than the _time-track_ annotation
that restructuring implicitly adds).  Luna will change the EDF header in the new file, to represent the new start time
for the continuous EDF (i.e. which is at the second epoch in the original):

```
luna s.lst -s 'MASK mask-epoch=1 & RE & WRITE edf-tag=v1'
```

To force Luna to write the same data as EDF+D, add the `EDF+D` option.  The data are the same as the previous `v1` condition,
except now, the file is EDF+, the EDF header start time stays the same as the original, and the time of each EDF record is tracked
via the EDF Annotations channel, as per EDF+ spec:

```
luna s.lst -s 'MASK mask-epoch=1 & RE & WRITE edf-tag=v2 EDF+D'
```

Next, instead of dropping the first epoch, we can drop one in the middle of the recording. This makes the recording inherently discontinuous (i.e. more than
one segment is present).  Luna therefore writes as an EDF+D:

```
luna s.lst -s 'MASK mask-epoch=666 & RE & WRITE edf-tag=v3'
```

This example is conceptually similar to the previous one, except now three intervals are removed, leaving three contiguous series of epochs
in the output data (i.e. which will be reflected as an EDF+D):

```
luna s.lst -s 'MASK mask-epoch=1-8,666,777 & RE & WRITE edf-tag=v4'
```

The final two examples demonstrate using the `force-edf` option to force writing as a standard EDF, i.e.
dropping any EDF Annotations and the time-track, so that the resulting output is a continuos recording that
ignores any potential discontinuities.

Luna still checks whether the recording is _truly_ discontinuous (i.e. has more than one segment), and handles the new EDF header start time accordingly.
If the new EDF is a continuous segment, it adjusts (or keeps as is) the EDF start time to match the new observed start, noting this in the log:


```
luna s.lst -s 'MASK mask-epoch=1 & RE & WRITE edf-tag=v5 force-edf'
```
```
resetting EDF start time from 20.08.36 to 20.09.06
```

However, if the output is truly discontinous, this means that clock-times are effectively invalid for that file when it is forced to be a standard EDF.
In this scenario, Luna
notes it is setting the EDF start to a null value (midnight), which effectively makes all times into elapsed hh:mm:ss past the EDF start time,
ignoring any gaps:
```
luna s.lst -s 'MASK mask-epoch=666 & RE & WRITE edf-tag=v6 force-edf'
```
```
setting EDF starttime to null (00.00.00)
```


### SEGMENTS

We can use the `SEGMENTS` command to describe the new files.   We'll first compile a list of the newly created EDFs using the `--build` command
but extracting only EDFs matching the new pattern (i.e. `test-v*.edf`):


```
 luna --build . | grep "test01-v" > s2.lst
```

This gives an `s2.lst` file as follows:

```
test01-v0	./test01-v0.edf
test01-v1	./test01-v1.edf
test01-v2	./test01-v2.edf
test01-v3	./test01-v3.edf
test01-v4	./test01-v4.edf
test01-v5	./test01-v5.edf
test01-v6	./test01-v6.edf
```

To examine the structure of these EDFs:

```
luna s2.lst -o out.db -s 'SEGMENTS & EPOCH verbose'
```

First, we can check the console reports from each file, we see the expected descriptions.  That is,
the unaltered standard EDF with the original start time and duration: 

```
Processing: test01-v0 [ #1 ]
 duration: 10.28.30 | 37710 secs ( clocktime 20.08.36 - 06.37.05 )
 signals: 26 (of 26) selected in a standard EDF file:
```

The `v1` file is an EDF+C with a delayed start time, as the first
epoch was dropped.  Note that it contains 27 signals, not 26 as above,
as there is a new EDF Annotations channel:

```
Processing: test01-v1 [ #2 ]
 duration: 10.28.00 | 37680 secs ( clocktime 20.09.06 - 06.37.05 )
 signals: 27 (of 27) selected in an EDF+C file:
```

The `v2` file contains the same data, but as an EDF+D, and so has the original start-time
of `20.08.36`, not `20.09.06` as above:

```
Processing: test01-v2 [ #3 ]
 duration: 10.28.00 | 37680 secs ( clocktime 20.08.36 - 06.37.05 )
 signals: 27 (of 27) selected in an EDF+D file:
```

The `v3` file is truly discontinuous, as an EDF+D:

```
Processing: test01-v3 [ #4 ]
 duration: 10.28.00 | 37680 secs ( clocktime 20.08.36 - 06.37.05 )
 signals: 27 (of 27) selected in an EDF+D file:
```

The `v4` file is also an EDF+D; note the overall duration is shorter
than the previous file (37410s versus 37680s), as more epochs were
removed.  The clock-time shows the same start and stop times however,
as the final epoch is present in both (i.e. `20.08.36` to `06.37.05`).

```
Processing: test01-v4 [ #5 ]
 duration: 10.23.30 | 37410 secs ( clocktime 20.08.36 - 06.37.05 )
 signals: 27 (of 27) selected in an EDF+D file:
```

The `v5` file was similar to `v3` but was generated with the `force-edf` command and so is written as
a standard EDF:  

```
Processing: test01-v5 [ #6 ]
 duration: 10.28.00 | 37680 secs ( clocktime 20.09.06 - 06.37.05 )
 signals: 26 (of 26) selected in a standard EDF file:
```

Finally, the `v6` file was similar to `v4` but was generated with the
`force-edf` option too.  As above (`v5`) this is written as a standard
EDF.  However, note that because this is a truly discontinuous file,
Luna resets the EDF clock time to a _null_ value of `00.00.00`
(i.e. midnight), and notes this in the log.  Luna does this as
clock-time information is no longer valid if a truly discontinous
EDF+D is converted to a standard EDF.

```
Processing: test01-v6 [ #7 ]
 duration: 10.28.00 | 37680 secs ( clocktime 00.00.00 - 10.27.59 )
 signals: 26 (of 26) selected in a standard EDF file:
```

__As a summary:__

| File | Epochs | Option | Format | Duration | Clock-time |
| --- | --- | ---| --- | --- | --- |
| `test01-v0.edf` | [ start-end] | _none_ | EDF   | 37710s | 20.08.36 - 06.37.05 |
| `test01-v1.edf` | [ 2-end ] |  _none_ | EDF+C | 37680s| 20.09.06 - 06.37.05 |
| `test01-v2.edf` | [ 2-end ] | `EDF+D` |EDF+D | 37680s  | 20.08.36 - 06.37.05 |
| `test01-v3.edf` | [ start-665, 667-end ] | _none_ | EDF+D | 37680s | 20.08.36 - 06.37.05 |
| `test01-v4.edf` | [ 9-665, 667-776, 778-end ] | _none_ | EDF+D | 37410s | 20.08.36 - 06.37.05 |
| `test01-v5.edf` | [ 2-end ] | `force-edf` | EDF | 37680s | 20.09.06 - 06.37.05  |
| `test01-v6.edf` | [ start-665, 667-end ] | `force-edf` | EDF | 37680s | 00.00.00 -  10.27.59 |


Next, we can view the output of the `SEGMENTS` command, which gives the number of segments in each:

```
destrat out.db +SEGMENTS
```
```
ID	NSEGS
test01-v0	1
test01-v1	1
test01-v2	1
test01-v3	2
test01-v4	3
test01-v5	1
test01-v6	1
```

We can view the detailed SEGMENT information, which is given for each segment (here
adding spaces between different EDFs / IDs for clarity):

```
destrat out*.db +SEGMENTS -r SEG
```
```
ID        SEG   DUR_HR  DUR_MIN  DUR_SEC   START  START_HMS    STOP  STOP_HMS
test01-v0   1   10.475    628.5    37710       0   20.08.36   37710  06.37.06

test01-v1   1   10.466      628    37680       0   20.09.06   37680  06.37.06

test01-v2   1   10.466      628    37680    30.0   20.09.06   37710  06.37.06

test01-v3   1    5.541    332.5    19950       0   20.08.36   19950  01.41.06
test01-v3   2    4.925    295.5    17730   19980   01.41.36   37710  06.37.06

test01-v4   1    5.475    328.5    19710   240.0   20.12.36   19950  01.41.06
test01-v4   2    0.916       55     3300   19980   01.41.36   23280  02.36.36
test01-v4   3        4      240    14400   23310   02.37.06   37710  06.37.06

test01-v5   1   10.466      628    37680       0   20.09.06   37680  06.37.06

test01-v6   1   10.466      628    37680       0   00.00.00   37680  10.28.00
```


### EPOCHs

The `verbose` option of the `EPOCH` command will list the times of epochs in the new set of files. 


For the copy of the unaltered standard EDF (`v0`) everything looks
as expected: i.e. `START` is epoch start in elapsed seconds from the EDF header start-time
of `20:08:36`: (only showing first and last two lines)

```
destrat out.db +EPOCH -r E -i test01-v0
```

```
ID            E    E1       HMS            INTERVAL   MID   START   STOP
test01-v0     1     1  20:08:36         0.00->30.00    15       0     30
test01-v0     2     2  20:09:06        30.00->60.00    45      30     60
...
test01-v0  1256  1256  06:36:06  37650.00->37680.00  37665  37650  37680
test01-v0  1257  1257  06:36:36  37680.00->37710.00  37695  37680  37710
```

The next file `v1` skips the first epoch: note the later `HMS` time of the first epoch,
and the final epoch count is one less (1256 versus 1257), and the final `STOP`
time is different (37680 vesus 37710), although as expected, the `HMS`
time of the last epoch is the same as above (`06:36:36`):

```
destrat out.db +EPOCH -r E -i test01-v1
```
```
ID            E    E1       HMS            INTERVAL   MID   START   STOP
test01-v1     1     1  20:09:06         0.00->30.00    15       0     30
test01-v1     2     2  20:09:36        30.00->60.00    45      30     60
...
test01-v1  1255  1255  06:36:06  37620.00->37650.00  37635  37620  37650
test01-v1  1256  1256  06:36:36  37650.00->37680.00  37665  37650  37680
```

The `v2` example is a EDF+D and note here the first epoch starts are 30 seconds, not 0 seconds, elapsed time,
as the EDF+D preserves the original EDF start-time: that is, all elapsed seconds times are relative to the
EDF start-time, not the first observed record:

```
destrat out.db +EPOCH -r E -i test01-v2
```
```
ID            E    E1       HMS            INTERVAL    MID  START   STOP
test01-v2     1     1  20:09:06        30.00->60.00     45     30     60
test01-v2     2     2  20:09:36        60.00->90.00     75     60     90
...
test01-v2  1255  1255  06:36:06  37650.00->37680.00  37665  37650  37680
test01-v2  1256  1256  06:36:36  37680.00->37710.00  37695  37680  37710
```

The `v3` file is also an EDF+D that is discontinuous, as the epoch numbered
666 in the original file was removed.  Here we also show the intermediate rows
spanning what was the 666th epoch.  Note that there is still an epoch numbered 666
in this file: this is as expected, as epochs are simply numbered sequentially when created
for a given file (the column `E`). However, note how between 665 and 666 the time difference is
1 minute, with one epoch ending at 19950 elapsed seconds, but the next not starting until 19980.  That
is, the EDF+D structure has tracked the fact that this epoch was sliced out (from 19950 to 19980). 

```
ID            E    E1       HMS            INTERVAL    MID  START   STOP
test01-v3     1     1  20:08:36         0.00->30.00     15      0     30
test01-v3     2     2  20:09:06        30.00->60.00     45     30     60
...
test01-v3   665   665  01:40:36  19920.00->19950.00  19935  19920  19950
test01-v3   666   666  01:41:36  19980.00->20010.00  19995  19980  20010
test01-v3   667   667  01:42:06  20010.00->20040.00  20025  20010  20040
...
test01-v3  1255  1255  06:36:06  37650.00->37680.00  37665  37650  37680
test01-v3  1256  1256  06:36:36  37680.00->37710.00  37695  37680  37710
```

As as separate point: _within_ the operations for the same attached EDF, Luna does map
how the new epochs map onto the "original" epochs: i.e. if rather than writing `v3` to a
new EDF and then running `EPOCH` we combined the steps:

```
luna s.lst -o out.db -s 'MASK mask-epoch=666 & RE & EPOCH verbose'
```
```
ID         E    E1       HMS            INTERVAL    MID  START   STOP
test01     1     1  20:08:36         0.00->30.00     15      0     30
test01     2     2  20:09:06        30.00->60.00     45     30     60
test01     3     3  20:09:36        60.00->90.00     75     60     90
...
test01   664   664  01:40:06  19890.00->19920.00  19905  19890  19920
test01   665   665  01:40:36  19920.00->19950.00  19935  19920  19950
test01   667   666  01:41:36  19980.00->20010.00  19995  19980  20010
test01   668   667  01:42:06  20010.00->20040.00  20025  20010  20040
test01   669   668  01:42:36  20040.00->20070.00  20055  20040  20070
...
test01  1255  1254  06:35:36  37620.00->37650.00  37635  37620  37650
test01  1256  1255  06:36:06  37650.00->37680.00  37665  37650  37680
test01  1257  1256  06:36:36  37680.00->37710.00  37695  37680  37710
```

Now note how the original epoch number is preserved in `E` whereas
`E1` shows the current sequential numbering: these diverge after epoch
666, which Luna knows is missing in the internal EDF.   Note that, internally,
Luna always represents data as a EDF+D after any restructuring.


Returning to the generated EDFs, `v4` has a greater number of epochs removed
and is an EDF+D. Note how the first epoch starts at 240 seconds (at 20:12:36).
Also note that although Luna correctly reported the duration of the EDF as 37410
seconds in the log, as noted above, the final elapsed second time is 37710, as this is
relative to the EDF start time.  That is, for a discontinuous EDF, the total duration (sum
of intervals present) will be numerical smaller than the final time-point in elapsed seconds
past the EDF start-time.

```
ID            E    E1       HMS            INTERVAL    MID  START   STOP
test01-v4     1     1  20:12:36      240.00->270.00    255    240    270
test01-v4     2     2  20:13:06      270.00->300.00    285    270    300
...
test01-v4  1246  1246  06:36:06  37650.00->37680.00  37665  37650  37680
test01-v4  1247  1247  06:36:36  37680.00->37710.00  37695  37680  37710
```

The final two examples converted EDF+ files to standard EDFs using the `force-edf` option.  The
epoch start clock-time start has been changed here (to reflect the dropping of the first epoch)
but the first epoch seen here (i.e. was what epoch 2 in the original file) is now at 0 elapsed seconds,
as this is a continuous recording:

```
ID            E    E1       HMS            INTERVAL   MID   START   STOP
test01-v5     1     1  20:09:06         0.00->30.00    15       0     30
test01-v5     2     2  20:09:36        30.00->60.00    45      30     60
...
test01-v5  1255  1255  06:36:06  37620.00->37650.00  37635  37620  37650
test01-v5  1256  1256  06:36:36  37650.00->37680.00  37665  37650  37680
```

Finally, here we forced a truly discontinuous EDF+D file (with multipe segments) to be
written as a standard EDF.  This violates the use of clocktime (or indeed any meaningful tracking of
when the segments occurred with respect to the original recording).  Luna therefore purposely
denotes this fact by arbitrarily forcing the EDF start time to be `00:00:00`.

```
ID            E    E1       HMS            INTERVAL   MID   START   STOP
test01-v6     1     1  00:00:00         0.00->30.00    15       0     30
test01-v6     2     2  00:00:30        30.00->60.00    45      30     60
...
test01-v6  1255  1255  10:27:00  37620.00->37650.00  37635  37620  37650
test01-v6  1256  1256  10:27:30  37650.00->37680.00  37665  37650  37680
```


### MASKs

Naturally, now "epoch 1222" (for example) may point to different intervals of time in the
new EDFs, compared to the original.   For example, we can use the following command to pull out
the clock-time of "epoch 1222" for each of the new EDFs:

```
luna s2.lst -o out.db -s 'MASK epoch=1222 & RE & EPOCH verbose' 
```

```
destrat out.db +EPOCH -r E
```
```
ID            E  E1       HMS            INTERVAL    MID  START   STOP
test01-v0  1222   1  06:19:06  36630.00->36660.00  36645  36630  36660
test01-v1  1222   1  06:19:36  36630.00->36660.00  36645  36630  36660
test01-v2  1222   1  06:19:36  36660.00->36690.00  36675  36660  36690
test01-v3  1222   1  06:19:36  36660.00->36690.00  36675  36660  36690
test01-v4  1222   1  06:24:06  36930.00->36960.00  36945  36930  36960
test01-v5  1222   1  06:19:36  36630.00->36660.00  36645  36630  36660
test01-v6  1222   1  10:10:30  36630.00->36660.00  36645  36630  36660
```

As expected, the `HMS` clock-time varies between studies.  Correspondingly,
statistics based on the nominally similar epoch number will of course produce
different results, being based on different stretches of time: e.g. pulling the _skewness_
statistic for each epoch for the C3 channel:

```
luna s2.lst -o out.db -s 'MASK epoch=1222 & RE & STATS sig=C3'
```
```
destrat out.db +STATS -r CH -v SKEW
```
```
ID         CH               SKEW
test01-v0  C3  -0.31842721246534
test01-v1  C3  -2.71543137965712
test01-v2  C3  -2.71543137965712
test01-v3  C3  -2.71543137965712
test01-v4  C3  -0.45894829218807
test01-v5  C3  -2.71543137965712
test01-v6  C3  -2.71543137965712
```

In contrast, using clock-time (via the `hms` option) in a mask will work in all cases where
clock time has been preserved in the new studies.   As noted above, we know that the final `v6`
EDF does __not__ preserve clocktime, as a truly discontinuous EDF+D was forced to be a standard EDF.  As
such, we'll ignore this file below, only looking at `v0` to `v5`. 


```
destrat out.db +EPOCH -r E
```

```
luna s2.lst 1 6 -o out.db -s 'MASK hms=06:19:06-06:19:36 & RE & STATS sig=C3'
```

```
destrat out.db +STATS -r CH  -v SKEW 
```
```
ID         CH                SKEW
test01-v0  C3  -0.318427212465345
test01-v1  C3  -0.318427212465345
test01-v2  C3  -0.318427212465345
test01-v3  C3  -0.318427212465345
test01-v4  C3  -0.318427212465345
test01-v5  C3  -0.318427212465345
```

Note, the mask `06:19:06-06:19:36` only pulls out a single epoch, as
intervals are defined _up to_ the stop (i.e. `06:19:36` implies just
past the end of this section and so we do not match the next epoch
that starts at `06:19:36`).

Similarly, as expected, the times of the selected epochs now align
(where the actual epoch number `E` now varies between files):

```
luna s2.lst 1 6 -o out.db -s 'MASK hms=06:19:06-06:19:36 & RE & EPOCH verbose'
```

```
ID            E  E1       HMS            INTERVAL    MID  START   STOP
test01-v0  1222   1  06:19:06  36630.00->36660.00  36645  36630  36660
test01-v1  1221   1  06:19:06  36600.00->36630.00  36615  36600  36630
test01-v2  1221   1  06:19:06  36630.00->36660.00  36645  36630  36660
test01-v3  1221   1  06:19:06  36630.00->36660.00  36645  36630  36660
test01-v4  1212   1  06:19:06  36630.00->36660.00  36645  36630  36660
test01-v5  1221   1  06:19:06  36600.00->36630.00  36615  36600  36630
```

More generally, one will use annotation files to mask regions of an EDF rather than the `hms` option of the mask.  But the
same message applies: when dealing with discontinuous EDFs, it is better to use clock-time based annotation files, as illustrated below.


## .eannot and discontinuous EDFs

Luna supports several types of annotation file format, as described
[here](../ref/annotations.md#luna-annotations).  Here, we consider
some limitations of using the simple `.eannot` file format when
dealing with discontinuous/restructured EDFs.

To create an `.eannot` file from the existing hypnograms in the
original file, we can use the `eannot` option of the `STAGE` command,
which simply dumps out sleep stages into the file `a.eannot`:

```
luna s.lst -s STAGE eannot=a.eannot
```
 
```
head a.eannot
```
```
wake
wake
wake
wake
wake
wake
wake
wake
```

These annotations can be attached back to the original `test01.edf`
without issue (note: here we specify `test01.edf` directly rather than
`s.lst`, to ensure that the XML annotations are not automatically
attached):

```
luna test01.edf annot-file=a.eannot -s DESC
```
```
 annotations:
  NREM1 (x27) | NREM2 (x357) | NREM3 (x263) | REM (x158)
  wake (x452)
```

!!! info "Note on the annotation instance counts"
    Note that the number of each class of annotation is different here
    (e.g. 27 `NREM1` annotations versus what is listed in the log (only 15)
    when attaching the orignial
    XML).  This simply reflects that fact that NSRR XML annotations
    may contain intervals longer than 30 seconds, i.e. if contiguous
    epochs have the same annotation.  Any output that actually uses these
    annotations on a per-epoch level would give identical results.
 

Naturally, we cannot attach this `a.eannot` file to any of the derived / altered EDFs, however, e.g.
`test01-v1.edf` which has the first epoch missing.  Luna would spot there is a different number of epochs
implied and complain:

```
luna test01-v1.edf annot-file=a.eannot -s DESC
```
``` 
 error : expecting 1256 epoch annotations, but found 1257
```

In fact, Luna does not even let .eannot files be attached to EDF+D
files, even if the number of epochs matches. This behavior is done
purposefully, as it is generally error-prone practice to mix the
simple .eannot format with discontinuous files:

```
luna test01-v4.edf annot-file=a.eannot -s DESC
```
``` 
 error : cannot use .eannot files with discontinuous (EDF+) files
```

## Clock-time annotations

The solution is to instead use an `.annot` file that tracks the hh:mm:ss time
and so can always be (safely) interpreted w.r.t. the original recording.


``` 
luna test01.edf annot-file=a.eannot -s WRITE-ANNOTS hms file=a.annot

```
The `a.annot` file is as follows (some lines removed for clarity):
```
class  instance  channel        start       stop    meta 
 wake      wake        .     20:08:36   20:09:06       .
 wake      wake        .     20:09:06   20:09:36       .
 wake      wake        .     20:09:36   20:10:06       .
 wake      wake        .     20:10:06   20:10:36       .
```
Note that the `hms` option for `WRITE-ANNOTS` makes the .annot file use clock-time (hh:mm:ss) rather than
elapsed seconds.   

Now we can attach this to any of the derived EDFs that preserved clock-time.  Again, ignoring the `v6`
file which did not preserve clock-time:
 
```
luna s2.lst 1 6 annot-file=a.annot -o out.db -s HYPNO
```
Pulling out the implied stage durations in minutes:
```
destrat out.db +HYPNO -v MINS_N1 MINS_N2 MINS_N3 MINS_REM TWT
```
```
ID        MINS_N1  MINS_N2  MINS_N3  MINS_REM    TWT
test01-v0    13.5    178.5    131.5        79    226
test01-v1    13.5    178.5    131.5        79  225.5
test01-v2    13.5    178.5    131.5        79  225.5
test01-v3    13.5    178      131.5        79    226
test01-v4    13.5    177.5    131.5        79    222
test01-v5    13.5    178.5    131.5        79  225.5
```

Relative to the original (`v0`), `v1`, `v2`and `v5` have one fewer wake
epoch (i.e. `TWT` reduced by half a minute).   `v3` has one fewer NREM2 epoch, and `v4` has eight
fewer wake epochs (4 mins) and two fewer NREM2 epochs (1 min).

Recalling the epochs dropped above (tabulated here):

| File | Dropped epoch numbers (relative to original file) |
|---|---|
| `v0` | _none_ |
| `v1` | 1 |
| `v2` | 1 |
| `v3` | 666 |
| `v4` | 1-8, 666 & 777 |
| `v5` | 1 | 

We can confirm that these are as expected, by querying the original EDF's annotation as follows:

```
luna s.lst -o out.db -s 'MASK epoch=1-8,666,777 & RE & STAGE'
```

```
destrat out.db +STAGE -r E
```
```
ID       E     CLOCK_TIME  MINS    STAGE  STAGE_N
test01   1     20.08.36    0       W      1
test01   2     20.09.06    0.5     W      1
test01   3     20.09.36    1       W      1
test01   4     20.10.06    1.5     W      1
test01   5     20.10.36    2       W      1
test01   6     20.11.06    2.5     W      1
test01   7     20.11.36    3       W      1
test01   8     20.12.06    3.5     W      1
test01   666   01.41.06    332.5   N2     -2
test01   777   02.36.36    388     N2     -2
```
 

## Start-time offsets

Consider an EDF recording starts at 21:04:56 but that staging (in 30
second epochs) started after a brief pause of 4 seconds at exactly
21:05:00.

One could have an annotation file, `a1.annot` as follows, with the first two rows:

```
W      .      .     4     34     .
N1     .      .     34    64     .
...
```

Although there are a few workarounds possible here (e.g. setting
epoch duration equal to 1 second, removing the first 4 seconds and
writing a new EDF, etc), the easiest remedy is to use the new `offset`
or `align` options for the `EPOCH` command.  Rather than always
having epochs start at the first observed record (i.e. 0 seconds in a
continuous EDF) this starts defining epochs at some later time, still
shifting each epoch forward by the `inc` parameter (which by
default equals the `dur` parameter, which is 30 seconds by
default).

For example, to start defining epochs at 4 seconds:
```
luna s.lst -o out.db -s ‘EPOCH offset=4 verbose & HYPNO’
```
```
destrat out.db +EPOCH -r E
```
```
ID      E  E1       HMS        INTERVAL  MID  START  STOP
test01  1   1  20:08:40     4.00->34.00   19      4    34
test01  2   2  20:09:10    34.00->64.00   49     34    64
test01  3   3  20:09:40    64.00->94.00   79     64    94
test01  4   4  20:10:10   94.00->124.00  109     94   124
test01  5   5  20:10:40  124.00->154.00  139    124   154
test01  6   6  20:11:10  154.00->184.00  169    154   184
test01  7   7  20:11:40  184.00->214.00  199    184   214
test01  8   8  20:12:10  214.00->244.00  229    214   244
test01  9   9  20:12:40  244.00->274.00  259    244   274
...
```

Alternatively, the `align` option will do the same as `offset` but will automatically select the offset
to be the start time of the first instance of any of the listed annotations: i.e. here will be 4.0 when `W` occurs:

```
luna s.lst -s 'EPOCH align=W,N1,N2,N3,R,U verbose & HYPNO'
```

## EDF record alignment

<NCH-SDB example>




