# Epochs

_Commands to define epochs for an EDF, and to attach annotations after loading an EDF_

| Command   | Description |
|------|---|
| [`EPOCH`](#epoch) | Specify epochs for an EDF |
| [`EPOCH-ANNOT`](#epoch-annot) | Load epoch-wise annotations from a [`.eannot`](annotations.md#eannot-files)-format file | 

## `EPOCH`

_Divides the time series into equally-sized (possibly overlapping) epochs_

Epoching an EDF is often a first a step that is required for many
other Luna functions that work with epoched data.  Although many
commands that require epoched data will automatically set epochs (with
a default 30-second duration), it is always good practice to set them
explicitly in a script.

<h3>Parameters</h3>

Without parameters the default is 30-seconds, with no overlap
(i.e. 30-second increments, equivalent to `len=30` or `epoch=30`).  In
general, if the increment is not specified, it defaults to the epoch
length (i.e. no overlap between epochs).

By default, epochs start at 0 seconds (start of EDF).   The 

| Parameter | Example |Description |
| --- | --- | --- | 
| `len`  | 30 | Epoch length (seconds), defaults to 30 |
| `inc`  | 30 | Epoch increment (seconds), defaults to `len` (i.e. no overlap) |
| `epoch` | 30,15 | Epoch length{,increment} (seconds), defaults to 30,30 |
| `require` | 10 | Stop processing that EDF if there are not at least _N_ epochs | 
| `verbose` |  | Output epoch-level information |
| `align` | `N1,N2,N3,R,W,?` | Align epoch starts with the first of these annotations |
| `offset` | 10 | Explicitly set start time for first epoch (seconds) |
| `clear` | | Reset any defined epochs |
| `min` | | Minimal output: writes the number of epochs to standard output | 

Note: if `align` is not given an explicit argument, it defaults to `N1,N2,N3,R,W,?,L,U,M` - i.e.
a list of common stage annotations.


<h3>Outputs</h3>

Basic summary information (strata: _none_)

| Variable | Description |
| --- | --- |
| `DUR` | Epoch duration (as per the input) |
| `INC` | Epoch increment (as per the input) |
| `NE` | Number of epochs in the EDF, given `DUR` and `INC` |

Per-epoch interval information (option: `verbose`, strata: `E`)

| Variable | Description |
| --- | --- |
|`E1`       | Current epoch number (which may differ from `E` if the EDF has been restructured) |
|`HMS`      | Clock-time for epoch start (hh:mm:ss) | 
|`INTERVAL` | String label of epoch interval (seconds) |
|`MID`      | Midpoint of epoch (seconds elapsed from EDF start) |
|`START`    | Start of epoch (seconds elapsed from EDF start) | 
|`STOP`     | Stop of epoch (seconds elapsed from EDF start) |


<h3>Example</h3>

To set epochs of 10-seconds, each with a 5-second overlap (output
showing the first six epochs):

```
luna my.edf -o out.db -s "EPOCH len=10 inc=5 verbose" 
```

To get the number of epochs set:
```
destrat out.db +EPOCH
```
```
ID       DUR   INC   NE
nsrr01   10    5     8183
```
To get the exact location of each epoch: 	
```
destrat out.db +EPOCH -r E
```

```
ID      E   E1   HMS        INTERVAL       MID  START  STOP
nsrr01  1   1    21:58:17   0.00->10.00    5    0      10
nsrr01  2   2    21:58:22   5.00->15.00    10   5      15
nsrr01  3   3    21:58:27   10.00->20.00   15   10     20
nsrr01  4   4    21:58:32   15.00->25.00   20   15     25
nsrr01  5   5    21:58:37   20.00->30.00   25   20     30
nsrr01  6   6    21:58:42   25.00->35.00   30   25     35
... (etc) ...
```

If instead running this command, which first applies a [`MASK`](masks.md#mask) 
and [`RESTRUCTURE`](masks.md#restructure)s the data, the 
```
MASK epoch=10-14 & RE & EPOCH verbose
```
then the output would be as follows (i.e. note the difference between `E` and `E1`):
```
ID      E    E1   HMS        INTERVAL         MID   START  STOP
nsrr01  10   1    22:02:47   270.00->300.00   285   270    300
nsrr01  11   2    22:03:17   300.00->330.00   315   300    330
nsrr01  12   3    22:03:47   330.00->360.00   345   330    360
nsrr01  13   4    22:04:17   360.00->390.00   375   360    390
nsrr01  14   5    22:04:47   390.00->420.00   405   390    420
```

!!! info
    The above command implicitly ran an `EPOCH` command prior to the `MASK`; as we only specified `verbose` for the second command, the 
    output will be from the second `EPOCH`.  In general, if you use the same command more than once in a script, it is a good idea
    to use [`TAG`](summaries.md#tag)s to keep track of the output, as Luna will otherwise overwrite previous output from that command.  For example:
    ```
    MASK epoch=10-14 & RE & TAG R/2 & EPOCH verbose   
    ```
    Output from the first (untagged) `EPOCH`:
    ```
    destrat out.db +EPOCH 
    ```
    ```
    ID      DUR    INC    NE
    nsrr01  30     30     1364
    ```
    Output from the second (tagged) `EPOCH` (tagged with `R` set to `2`):
    ```
    destrat out.db +EPOCH -r R
    ```
    ```
    ID       R   DUR   INC    NE
    nsrr01   2   30    30     5
    ```
    i.e. note the difference in `NE`.


## `EPOCH-ANNOT`

_Attach epoch-level annotations from a file, to an epoched EDF_

This command reads a __plain-text__ file from disk, expecting the
[`.eannot`](annotations.md#eannot-files) format.  It is possible to
apply multiple sets of annotations to epochs with multiple
`EPOCH-ANNOT` commands.

The behavior of this command similar but not identical to what would
happen if the `.eannot` file were directly specified in the
[_sample-list_](../luna/args.md#sample-lists) (i.e. as any other XML,
`.annot` annotation file).  As this command can be performed
after the EDF has been loaded and manipulated (e.g. via
[`RESTRUCTURE`](manipulations.md#restructure)), the number of epochs
(i.e. rows in the file) should _match exactly_ the number of epochs
given the _current state_ of the in-memory EDF (i.e. after any
restructuring, and with the current epoch definitions).  
	       
That is, if `EPOCH len=30` generates 1022 epochs for a given EDF, then
the annotation file must be exactly 1022 epochs. Luna will give an
error if the `EPOCH` command has not been performed prior to
`EPOCH-ANNOT`.  

_In contrast_, when attaching an `.eannot` file via the sample-list,
Luna assumes the `.eannot` file corresponds to the entire _on-disk_
EDF (typically with the default 30-second epochs).


!!! danger "Make sure to use plain-text files"
    As with all input to Luna, except the EDF itself, it is
    critical that the annotation file be a simple, plain-text file
    with no special encodings, not RTF (rich text format, etc).
    Please see [this FAQ](../faq.md#windows-line-endings).


<h3>Parameters</h3>

| Parameter | Example | Description |
| --- | --- | --- |
| `file`   | `file=annots/id1.epochs` | File path/name to read annotations from (required) |
| `recode` | `recode=NREM1=N1,NREM2=N2` | Optional, comma-delimited list of recodings (_from_=_to_) |

An optional `recode` parameter can be used to translate annotations on-the-fly.

<h3>Outputs</h3>

This command does not produce any output _per se_, other than noting the number of epoch/annotations read in the log/console, e.g.

```
 CMD #2: EPOCH-ANNOT
 mapping 2 distinct epoch-annotations (1022 in total) from annots/test.eannot
```

<h3>Example</h3>

Here we attach epoch-annotations to an epoched EDF and use them to
[_mask_](masks.md) out certain epochs.  For example, here we have a
dummy annotation file, that contains an annotation of either `T` or
`A` for each epoch.  This particular EDF has 1022 30-second epochs,
and so the file `epoch1.eannot` is the same length:

``` 
$ sort epoch1.eannot | sort | uniq -c 
102 A 
920 T
```

The file `epoch1.eannot` is a simple text-file, with either an
`A` or a `T` on each line, and nothing else.

!!! note 
    Annotations are not restricted to `A` or `T` or single
    characters: they can be any text string, but should not contain
    spaces or special characters.  A common use may be if sleep
    staging has been performed, so each row indicates _N1_, _N2_,
    _N3_, _REM_ or _Wake_, etc.


To attach this file, and use it to select only `A` epochs:

```
luna my.edf -s "EPOCH & EPOCH-ANNOT file=epoch1.eannot & MASK ifnot=A & RE"
```

```
 ..................................................................
 CMD #1: EPOCH
 set epochs, length 30 (step 30), 1022 epochs
 ..................................................................
 CMD #2: EPOCH-ANNOT
 mapping 2 distinct epoch-annotations (1022 in total) from epoch1.eannot
 ..................................................................
 CMD #3: MASK
 set masking mode to 'mask' (default)
 based on A 102 epochs match;  920 newly masked, 0 unmasked, 102 unchanged
 total of 102 of 1022 retained
 ..................................................................
 CMD #4: RESTRUCTURE
 restructuring as an EDF+ : keeping 3060 records of 30664, resetting mask
 retaining 102 epochs
```

That is, we've used an external annotation to select a set of epochs
from an EDF.  In real applications, one would subsequently apply other
analysis commands, or write out a new EDF (see
[`WRITE`](outputs.md#write)), for example.

!!! hint "Working with multiple EDFs"
    When using [sample-lists](../luna/args.md#sample-lists) to
    work with multiple EDFs, it is often convenient to use the `^`
    special character in a script, which is replaced with the ID (from
    the first column of the sample-list, not the EDF header) for that
    individual, which enables the same script to point to different
    annotations files for different individuals (assuming of course,
    that those annotation files are named to include that ID).
   
Here is an example of using a script (instead of the `-s` option) for
achieve the same result as above, but here using a sample list and
renaming the annotation file to, for example,
`annots/id001.epoch.eannot`.  We might specify the sample list `s.lst`
as follows, to assign the ID `id001` to the EDF:

``` 
id001    my.edf 
```

We would then write a Luna script file, say called `extract.txt`,
which also illustrates using a [_variable_](../luna/args.md#variables)
(to specify more flexibly which annotation we want to select,
e.g. `A`, or `T` or something else, if another annotation file were
used) and writing a new EDF (with the [`WRITE`](outputs.md#write)
command):

```
% Epoch the data (default 30 sec)

EPOCH

% Attach the matching file in the annots/ folder for this individual
% i.e. requires that files are named 

EPOCH-ANNOT file=annots/^.eannot

% Set this mask, where the value of ${x} will be specified 
% on the command line

MASK ifnot=${x}

% Apply the mask, actually restructuring the internal EDF
% (Note, RESTRUCTURE can be abbreviated as RE)

RESTRUCTURE

% Write a new EDF back to disk

WRITE edf-dir=edfs/            % write new EDFs in this directory/folder
      edf-tag=extract-${x}     % add this tag to the filename for each EDF
      sample-list=s-${x}.lst   % make a new sample-list to point to the new EDFs
```

Note how the `^` character is used to specify the individual/EDF
ID. We can then run Luna, setting the `x` variable to `A`:

```
luna s.lst x=A < extract.txt 
```

When run for `id001`, for example, Luna looks for
`annots/id001.eannot`, and attaches it.  It then masks epochs that do
not match the value specified by the variable `${x}`, which in
this case is set to `A`. After `RESTRUCTURE`-ing the internal EDF to
remove the masked epochs, we `WRITE` it back out to a file,
`edfs/my-extract-A.edf` and create a new sample list `s-A.lst` that
points to this.  As a sanity check, we can run a `DESC` command on the
new project/sample-list:

```
luna s-A.lst -s DESC
```
which correctly shows a new EDF of 51 minutes (i.e. 102 30-second epochs).
```
EDF file          : edfs/my-extract-A.edf
ID                : id001
Duration          : 00:51:00
Number of signals : 6
Signals           : EOG-L[256] EOG-R[256] EMG[256] EEG1[256] EEG2[256] EEG3[256]
```


Naturally, the advantage of this approach is that it will work if a
project (i.e. the `s.lst`) contains multiple EDFs:

```
id001   subj-1.edf
id002   subj-2.edf
id003   subj-3.edf
... (etc) ...
```
and the `annots/` folder contains the corresponding annotation files for each subject:
```
annots/id001.eannot
annots/id002.eannot
annots/id003.eannot
```

That is, all EDFs can still be processed (and multiple new EDFs
generated) with a single Luna command and without altering or copying
the `extract.txt` Luna script.
