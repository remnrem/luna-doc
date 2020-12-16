# Interval annotations

_Here we describe the formats of annotation files, how to attach them
to EDFs, and how to view and summarize their contents._


| Command  | Description |
|---|---|
| [Luna annotations](#luna-annotations) | Overview of annotations in Luna | 
| [NSRR XML files](#nsrr-xml-files) | Format of NSRR XML annotation files |
| [`--xml`](#-xml) | Quickly view an NSRR XML annotation file |
| [.annot files](#annot-files) | `.annot` file format |
| [.eannot files](#eannot-files) | `.eannot` file format |
| [FTR files](#ftr-files) | Format of FTR annotation files |
| [`ANNOTS`](#annots)       | Tabulate all annotations |
| [`SPANNING`](#spanning)   | Report on _coverage_ of annotations |

## Luna annotations

Annotations are a key part of Luna, and in combination with the
[`MASK`](masks.md#mask) command and so-called [_eval_
expressions](evals.md), provide a flexible way to manipulate EDFs.
 
Luna can represent a number of different types of annotation.
Annotations may be represented as arbitrary intervals of time (defined
with respect to the start of the EDF), or can be specified at the
per-epoch level in a number of ways.  First, we introduce some terminology:

A _class_ of annotations is a set of arbitrary time intervals
(e.g. such as "spindles")

Each _instance_ of a _class_ (e.g. one particular spindle) is defined
by a time interval and an identifier (which may or may not be unique).

Optionally, each _instance_ can also have associated meta-data, stored
as typed key/value pairs.

This table summarizes these levels of annotation:

| Type | Level | Example concept | Specific example |
| ---  | ---- | --- | --- | 
| _class_ name | Generic set of annotations   | All spindles |     `spindles` |
| _instance_ | Specific instance of a _class_<br> defined by a time interval | A single spindle |  873.2 to 874.1 seconds | 
| _instance_ names | ID (or non-unique label) for each instance | Spindle number <br>(or type of spindle) | `spindle-1, spindle-2`, ... <br>(or `fast-spindle`, `slow-spindle`) | 
| _instance meta data_ | Associated information | Duration, amplitude, etc,<br> for an individual spindle | `dur=0.88` `amp=12.2` `frq=11.1` |


Annotations can be represented in a number of different file formats, all described below:

- [NSRR-format XML](#nsrr-xml-files) files, where each _ScoredEvent_ parent node in one XML is treated as a distinct annotation _class_
- [`.annot`](#annot-files) files, where each `name` in the header specifies a distinct annotation _class_
- [`.eannot`](#eannot-files) files containing simple per-epoch labels, where each distinct label is treated as a distinction annotation _class_ 
- [EDF+ Annotations](#edf-annotations-channel), where annotations are embedded in the EDF as a separate channel
- [FTR](#ftr) files, where each FTR file is a single annotation _class_, within which the instance IDs are the row-level labels in that file

!!! note 
    Currently masks only operate on _class_ and _instance_ level
    data, not _meta-data_. That is, annotation _meta-data_ is currently
    not used by Luna.
    Also, including or excluding certain annotations with the
    [`annot`](../luna/args.md#annot) option works at the _class_ level
    only.  That is, either all _instances_ of a given _class_ are
    loaded in, or none are.


## NSRR XML files

Luna accepts [XML](https://en.wikipedia.org/wiki/XML) annotation
files, as used by the [National Sleep Research
Resource](http://sleepdata.org).  These are based on Compumedics
Profusion files, as described
[here](https://github.com/nsrr/edf-editor-translator/wiki/Compumedics-Annotation-Format).
Any files that end in a `.xml` or `.XML` extension are assumed to be
in this format, and Luna will attempt to read it as an annotation
file. Luna maps each _ScoredEvent_ node to a single annotation
_class_.


## `--xml`

To quickly view the _Scored Events_ in a single NSRR `XML` file (sorted by clock time):

```
luna --xml my-annotations.xml
```
```
.    .	    EpochLength	30
0 - 30	    (30 secs)	SleepStage	Wake
30 - 60	    (30 secs)	SleepStage	Wake
60 - 90	    (30 secs)	SleepStage	Wake
90 - 120    (30 secs)	SleepStage	Wake
120 - 150   (30 secs)	SleepStage	Wake
150 - 180   (30 secs)	SleepStage	NREM1
180 - 210   (30 secs)	SleepStage	Wake
210 - 240   (30 secs)	SleepStage	Wake
... (etc) ...
```

Alternatively, use `--xml2` to report _all_ entries in any XML, along with the full, original XML document tree structure:

## EDF+ Annotations channel


for -s ' MASK ... '
  note c( 'aa' , 'bb' )   is same as c( {aa} , {bb} ) "
  can use { and } instead of ', when can be convenient if using -s on the command line

Using a test EDF+ file that contains some annotations
(one posted on Teunis van Beelen's website, which can be accessed [here](https://www.teuniz.net/edf_bdf_testfiles/)), 
Luna will automatically extract the annotations (by convention, from channels named `EDF Annotations`) and include 
these alongside any other annotations (e.g. from XML or `.annot` files).  For this example EDF+ file:

```
luna ma0844az_1-1+.edf -o out.db  -s ANNOTS
```
The console output notes that an `EDF Annotations` channel is present, and is being extracted:
```
 signals: 38 (of 38) selected in an EDF+ file:
  EEG_FP1 | EEG_FP2 | EEG_F3 | EEG_F4 | EEG_C3 | EEG_C4 | EEG_P3 | EEG_P4
  EEG_O1 | EEG_O2 | EEG_F7 | EEG_F8 | EEG_T3 | EEG_T4 | EEG_T5 | EEG_T6
  EEG_FZ | EEG_CZ | EEG_PZ | EEG_E | EEG_PG1 | EEG_PG2 | EEG_A1 | EEG_A2
  EEG_T1 | EEG_T2 | EEG_X1 | EEG_X2 | EEG_X3 | EEG_X4 | EEG_X5 | EEG_X6
  EEG_X7 | DC01 | DC02 | DC03 | DC04 | EDF Annotations
  extracting 'EDF Annotations' track
```
Further, the log also notes the number of annotations, 41 in this case:

```
 annotations:
  edf_annot (x41)
```
Note that all annotations from an EDF+ file are assigned the _annotation class_ of `edf_annot`; the _annotation instance_ ID is assigned to whatever value is 
present in the EDF+ (annotation instance IDs need not be unique).  See above for a description of annotation _class_ versus _instances_. 

If the `verbose` special variable is set to true, then (surprisingly...) you'd see more verbose output
in the log, that lists some of these _instance_ IDs too:
```
 annotations:
  [edf_annot] 41 instance(s) (from ma0844az_1-1+.edf)
   36 instance IDs:  A1+A2_OFF HVT_00:30 HVT_01:00 HVT_01:30 HVT_02:00 ...
```

The `ANNOTS` command can be used to tabulate these annotations.  

```
luna ma0844az_1-1+.edf -o out.db  -s ANNOTS
```

```
destrat out.db +ANNOTS -r ANNOT INST T
```

(nb. below, omitting some columns for clarity of output, and just listing the first three annotations:)
```
ANNOT     INST              START START_ELAPSED_HMS START_HMS
edf_annot PAT_IIB_EEG       0     00.00.00          10.18.42
edf_annot REC_START_IIB_CAL 0     00.00.00          10.18.42
edf_annot A1+A2_OFF         1     00.00.01          10.18.43
...
```

For each annotaton that _start_ (and _stop_) times are given: in seconds elapsed since start of the EDF (`START`), 
the same information but in _hh:mm:ss_ format (`START_ELAPSED_HMS`) and as clock-time (`START_HMS`).

!!! note "Spaces in annotation labels"
    In the original EDF+, the annotation labels are actually `PAT IIB
    EEG`, `REC START IIB CAL`, etc.  Note that Luna has autoamtically
    converted spaces to underscore characters (`_`), to make working
    with these annotations easier on the command line (i.e. `PAT_IIB_EEG`, etc).   If for some reason this was 
    not desired, set the special variable `keep-annot-spaces=T` (or `keep-spaces=T` to do
    this for both channel and annotation labels.

Luna [masks](masks.md#masks) typically refer to annotation _class_ labels to work, but can also use annotation _instance_ identifiers, which
will typically be necessary with EDF+ annotations (i.e. unless every annotation in the EDF meant the same thing, in which case the `edf_annot` 
class label could be used). 

For example, we see overall this EDF+ is just over 30 minutes in duration, so 60 30-second epochs:
```
Processing: ma0844az_1-1+.edf [ #1 ]
 duration: 00.30.18 ( clocktime 10.18.42 - 10.49.00 )
```

We'll use two of the annotations in this file to extract only the
epochs containing those annotations: `HVT_00:30` and `HVT_01:00`,
which occur at `00.21.32` and `00.22.02` (elapsed time) respectively,
as one can see from looking at the full output as the above `ANNOTS`
command, for example.


To mask epochs based on these two labels, we can use the following, using the 
```
 class[instance|instance]
```
form (i.e. with instance IDs are specified as a `|`-delimited list, within square brackets `[` and `]` following the class ID).

``` 
luna ma0844az_1-1+.edf -o out.db  -s  ' MASK mask-ifnot=edf_annot[HVT_00:30|HVT_01:00] '
```


!!! note "Using eval expressions instead"
    This is a digression, but to illustrate a difference way to use Luna masks in this instance, 
    and to point to a common problem/solution.  Instead of the above form, to mask epochs based on these two labels, we could also use the following:

    ```
    luna ma0844az_1-1+.edf 
       -o out.db  
       -s ' MASK expr=" edf_annot =~ c( {HVT_00:30} , {HVT_01:00} ) " '
    ```
    ```
     CMD #1: MASK
     options: expr=" edf_annot =~ c( {HVT_00:30} , {HVT_01:00} ) " sig=*
     set epochs, to default length 30, 60 epochs
      set masking mode to 'force'
       based on eval expression [edf_annot =~ c( {HVT_00:30} , {HVT_01:00} )]
       2 true, 58 false and 0 invalid return values
       2 epochs match; 2 newly masked, 0 unmasked, 58 unchanged
        total of 58 of 60 retained
    ```

    Expressing the above `MASK` command on the command line in this way, we used `-s` to write it out.  To stop the (bash) shell from interpreting special characters that
    can often occur (e.g. `&` to separate Luna commands) we
    placed all text within single quotes (`'`).  We also had to place the eval expression with double quotes, so that Luna could correctly parse the entire expression.
    This means we could not also use single quotes easily to specify string literals on the command line (e.g. `HVT_00:30`).  We therefore used Luna's alternate way of
    specifying string literals in eval expressions, using braces `{` and `}` -- so we have:    
    ```
    edf_annot =~ c( {HVT_00:30} , {HVT_01:00} )     
    ```
    instead of:
    ```
    edf_annot =~ c( 'HVT_00:30' , 'HVT_01:00' )     
    ```
    Note that if the whole command was in a separate file (rather than following `-s` and therefore already enclosed in single quotes), 
    we would be able to use this second (more readable) form.   Neither quotes nor braces are needed in the simpler format first given above.


## .annot files 

_Generic annotation files_

Text-based, tab-delimited files that can contain either
_interval-level_ annotations: that is, internally, annotations that
reflect an interval of time along with certain meta-data.

!!! note 
    `.annot` files should be plain-text but need not have the
    `.annot` file extension.  Any annotation file that is neither
    `.xml`, `.ftr` nor `.eannot` is assumed to be of generic `.annot`
    file format.

An `.annot` file can describe one or more annotations _classes_. Each
class can have one or more _instances_, where each _instance_
corresponds to an interval of time and a single row of the `.annot`
file.  `.annot` files require header rows that first define the
classes in that file.  Each header row starts with a `#` character and
contains between one to three `|`-delimited fields, which are class
_name_, _description_ and _meta-data types_ respectively. For example

```
# a1 
# a2 | Unlike the first, this annotation has a description field
# a3 | This annotation also specifies meta-data types | val1[txt] val2[num] val3[bool]
```

In the contrived example above, there are three annotation classes:
`a1`, `a2` and `a3`.  Only `a2` and `a3` have description fields
(these are currently not used, but will feature in Scope as labels for
visualizations).  The `a3` class also expects some _meta-data_ for
each _instance_: three variables named `var1`, `var2` and `var3`, each with a specified _type_. 

| Type | Description |
| ---- | ---- | 
| `num` | Numeric (i.e. any floating point number) |
| `int` | Integer |
| `bool` | Boolean yes/no, true/false (with values `y`, `yes`, `Y` or `1` versus `n`, `N`, `no` or `0`) |
| `txt` | Any text string |

Subsequent _data_ rows of the `.annot` file specify instances of one
of these three classes, along with the necessary meta-data in the case
of `a3`. For example:

```
a1	i1	10.00	15.00	
a1	i2	92.10	105.22	
a1	i3	108.5	123.11
a2	.	e:2
a2	.	e:7
a2	.	e:10	e:12
a3	A	0	30	W	0.88	Y
a3	A	30	60	W	0.98	Y
a3	B	60	90	N1	0.23	N
```

That is, the each _data_ row must be at least three tab-delimited columns:

- first column: _class name_ that must match one of the header rows (i.e. `a1`, `a2` or `a3` in this example)
- second column: _instance ID_ that can be unique or not with respect to its annotation class (or even missing, as for `a2`)
- third (and fourth) column(s): these define the interval for this annotation instance, as described below
- remaining columns: if the header specified _meta-data_ for that annotation _class_, these values must be listed here, in the same order as specified in the header row

As in the example above, there are three different ways to define
intervals in an `.annot` file:

- start and stop _times in seconds_ in columns 3 and 4 (as for `a1` above)
- a single _epoch code_ in column 3, starting with `e:` (as for `a2` above)
- a pair of _epoch codes_ in columns 3 and 4, both starting with `e:` (as for `a3` above)

You can mix and match these different formats with the same annotation
class. In the example above, we map them to `a1`, `a2`, and `a3`
simply to make the example clearer.

<h5>Interval encoding</h5>

Defining intervals by start and stop times in seconds is the most
flexible: these annotations can take on any values that can be smaller
than an epoch, or can span multiple epochs.

Intervals are defined to be inclusive of the _start_ but exclusive of
the _stop_, i.e. the interval from _a_ to _b_ is _[a,b)_.  In other
words, _b_ is the first point past the end of the interval. Interval
duration is therefore defined as _b-a_.  This means that two 30-second
epochs specified below are non-overlapping epochs: rather, they are
contiguous despite `30.00` appearing in both definitions:

``` 
class1	interval1   0.00 30.00 
class1 	interval2  30.00 60.00 
``` 

<h5>Epoch encoding</h5>

_Epoch encoding_ is provided as a convenience feature as many
annotations are in fact specified in terms of regular-sized epochs and
it might be awkward to always have to list the start and stop times of
each epoch.  These codes (that start with the characters `e:` to
distinguish them from times in seconds) are converted to the
equivalent _interval_ when reading the file. 
 
If the value in the third or fourth column starts with the characters
`e:`, then Luna assumes that epoch-notation is being used to specify the
interval.  By default, Luna assumes non-overlapping 30-second epochs,
whereby `e:1`, `e:2`, `e:3`, etc, refer to the first, second, third,
etc, epochs.   If the fourth column also starts `e:` then Luna assumes the interval is 
from the start for the first epoch to the end of the second epoch.   

For example, the following two lines specify identical intervals:
```
class1       instance1       0        30
class1       instance1       e:1
```
as do these two lines:
```
class1       instance1       30       120
class1       instance1       e:2      e:4
```

Different epochs definitions can be specified by explicitly appending the epoch
duration (in seconds) and increment (in seconds) as colon-delimited
values, as shown in the Table below.  

| Example | Description | Implied interval (sec) |
| ---- | ---- | -----|
| `e:2` | Second epoch; non-overlapping 30-second epochs | _[30.0,60.0)_ |
| `e:2:20` | Second epoch; non-overlapping 20-second epochs | _[20.0,40.0)_ |
| `e:2:30:15` | Second epoch; 50% overlapping 30-second epochs | _[15.0,45.0)_ |

If not specified, the increment is assumed to be the same as the epoch
duration, i.e. no overlap of consecutive epochs.

!!! note 
    As epochs are defined within the `.annot` file itself, the
    actual EDF need not be epoched (i.e. from the
    [`EPOCH`](epochs.md#epoch) command). In fact, the EDF may even
    have epochs of a different duration specified. Also, unlike
    [`.eannot`](#eannot-files) files, not every epoch needs to be
    specified in an `.annot` file, i.e. if there are 1200 epochs, you
    do not need to have exactly 1200 rows in this file. This is
    because instances specified with epoch-encoding are automatically
    converted to interval-encoding upon loading.

## .eannot files 

This is the simplest format for epoch-level annotations.  Epoch
annotations in `.eannot` files are simple labels attached to
individual epochs.  The format is as follows:

- one row per epoch
- each row contains a single label, that is attached to that epoch
- for each distinct label in the file, a new annotation _class_ is generated
- each _instance_ is assigned the same ID as the label name (i.e. same as the _class_ name)

When an `.eannot` is specified in the
[_sample-list_](#../luna/args.md#sample-lists), it is attached prior
to loading the EDF.  By default, Luna assumes epochs are 30-seconds in
duration and do not overlap when using `.eannot` files.

!!! hint
    To work with `.eannot` files but use different epoch definitions, you have three options: 

    1. use the [`EPOCH`](epochs.md#epoch) and
    [`EPOCH-ANNOT`](epochs.md#epoch-annot) commands to attach the file
    _after_ initially attaching the EDF (i.e. instead of specifying
    the `.eannot` file in the sample-list)

    2. Set the special variable
    [`epoch-len`](../luna/args.md#epoch-len) variable if the `.eannot`
    is specified in the sample list (i.e. and so loaded when the EDF
    is first attached, prior to running any `EPOCH` command)
    
    3. Use the `e:` epoch encoding notation in a generic `.annot` file instead, as described above.


Unlike other types of annotation, these can be loaded via a Luna
command, [`EPOCH-ANNOT`](epochs.md#epoch-annot), potentially _after_
the EDF has been loaded and manipulated.  In that case, i.e. when using the
`EPOCH-ANNOT` command, the number of rows (epochs) in the `.eannot`
file must match the number of epochs that currently exist in the
in-memory representation of the EDF (i.e. which may be different from
the on-disk version).

A common use of an `.eannot` file could be to store manually-scored sleep stages: 

```
wake
wake
N1
N1
wake
N1
N1
N2
N2
... (etc) ...
```
where the number of rows of this file corresponds to the number of 30-second epochs in the EDF.  One could then use these annotations in a [`MASK`](masks.md#mask) command, such as:
```
MASK if=wake
RESTRUCTURE
```
to exclude `wake` epochs from analysis.


## FTR files

_Feature_ files are a simple annotation format generated by Luna 
after certain commands (e.g. the `ftr` option of the 
[`SPINDLES`](spindles-so.md#spindles)).  Their filenames must conform to a special format:

```
id_{indiv_id}_feature_{label}.ftr 
``` 

where `{indiv_id}` should be replaced by the individual/EDF ID, and
 `{label}` should be replaced by the name of the feature. (Note, IDs
 and feature labels can contain underscore characters, but they should
 not contain the phrase `_feature_`.)  

For example:


```
id_subj00001_feature_spindles11hz.ftr 
```

means that the contents of this file are the annotation class
`spindles11hz` for the individual/EDF with ID `subj00001`.

If the sample-list points to a folder, e.g. 
```
id001	/path/to/id001.edf     /path/to/annots/
```
or a global [`annot-folder`](../luna/args.md#annotations) 
folder is specified via the command-line or parameter file, e.g. 
```
luna s.lst annot-folder=/path/to/annots/ < command.txt 
```

then all FTR files that match the ID of the EDF being processed are
loaded.  

!!! Hint
    To turn off automatic loading of all FTR files, add the
    following to the command line

    ```
    luna s.lst ftr=N < commands.txt
    ```
    or set that variable in a [parameter file](../luna/args.md#parameter-files)
    ```
    ftr    N
    ```

FTR files have the following format:

- tab-delimited plain-text
- no header line
- creates a single annotation _class_ with the name from the FTR filename (i.e. after `_feature_`)
- columns 1 and 2 are interval start and stop in [_time-point_](../luna/args.md#time-points) units of each _instance_
- column 3 is a text label that specifies the _instance_ ID
- further columns are _key=value_ pairs become instance _meta-data_ (all encoded as string values)

For example, for the file `id_subj00001_feature_spindles.ftr`:
```
20733175781250       20733707031250       sp-1   amp=3.34   dur=0.53   frq=11.21   nosc=6
21057488281250       21058015625000       sp-2   amp=4.59   dur=0.53   frq=11.29   nosc=6
21139898437500       21140558593750       sp-3   amp=3.59   dur=0.66   frq=10.54   nosc=7
... (etc) ...
```

These lines list information on detected spindles; here only three
spindles are listed, labeled `sp-1`, `sp-2`, etc.  Each spindle has
some arbitrary information encoded, on spindle amplitude (`amp`),
duration (`dur`), frequency (`frq`) and number of oscillations
(`nosc`).  Currently, these additional fields (column 4 onwards) are
not used internally by Luna.  They are, however, displayed when
looking at annotations in [Scope](../ext/scope.md) for example,
and are accessible when using the Luna [R extension library](../ext/R.md).

When loaded in, the annotation _class_ will be `spindles`.  There will
be as many _instances_ as rows in the FTR file, and the instance IDs
will be the third column of the FTR file (`sp-1`, `sp-2`, etc).  As
noted, instance IDs need not be unique, and they can be included in [`MASK`](masks.md#mask) 
commands.

!!! note 
    Although not currently used, there are two reserved keywords
    for FTR meta-data that will have special meanings, i.e. in future releases of
    Scope. These _key_ values are `_rgb_` (which expects a RGB value,
    encoded 0..255) and `_value`, which expects a floating-point
    number that will be taken to be a primary value, e.g. the one that
    will be used if plotting annotations in certain contexts.
    ```
    _rgb=255,255,255
    ```
    ```
    _value=0.234
    ```

## ANNOTS

_Tabulate and summarize annotation information_ 

Produces information about the number and total duration of
annotations in an EDF, at the whole-file level and optionally per-epoch.

By default, `ANNOTS` will only show annotations that span at least one
unmasked epoch. The definition of whether or not an annotation
instance is masked or not can be varied.  Depending on the context,
this can be useful to generate different types of summaries, e.g. the
number of respiratory events in REM versus NREM sleep.

<h3>Parameters</h3>

|  Parameter &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; | Example &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; | Description |
| --- | --- | --- |
| `epoch` | `epoch` | Show epoch-level summaries | 
| `show-masked` | `show-masked` | Show masked annotations (default is not to do so) | 
| `any` | `any` | Keep annotations that have _any overlap_ with one or more unmasked epochs (default) |
| `all` | `all` | Only keep annotations that are _completely within_ unmasked epochs |
| `start` | `start` | Keep annotations that _start_ in an unmasked epoch  |

The `epoch` option lists, for each epoch, all the annotations
that overlap with that epoch, given the specified overlap definition
(`any`, `all` or `start`).  If the `show-mask` option is given as well,
all epochs and annotations are shown; otherwise, only unmasked epochs
and annotations are shown.  The output contains two variables
(`EPOCH_MASK` and `ANN_MASK`) that indicate whether a given annotation
instance is masked or not.

<h3>Output</h3>

_Class-level_ annotation summary (strata: `ANNOT`)

| Variable | Description |
| --- | --- |
| `COUNT` | Number of instances of that annotation class |
| `DUR` | Combined duration (seconds) of all instances of that annotation class (does not account for potential overlap) |

_Instance-level_ annotation summary (strata: `ANNOT` x `INST`)

| Variable | Description |
| --- | --- |
| `COUNT` | Number of instances of that annotation class and instance ID |
| `DUR` | Combined duration (seconds) of all instances of that annotation class and instance ID (does not account for potential overlap) |
 
_Instance-level_ annotation tabulation (strata: `ANNOT` x `INST` x `T`) 

| Variable | Depends on | Description |
| --- | --- | ---- | 
| `START`         |   |   Start time (seconds) of this instance | 
| `STOP`          |  | Stop time (seconds) of this instance |
| `VAL`           |  | The _meta-data_ for this instance, if any exists (otherwise missing `NA`) |
| `ALL_MASKED`    | `show-masked` | |
| `ALL_UNMASKED`  | `show-masked` |  |
| `SOME_MASKED`   | `show-masked` | |
| `SOME_UNMASKED` | `show-masked` | |
| `START_MASKED`  | `show-masked` | |


Per-epoch _instance-level_ annotation tabulation (strata: `E` x `INTERVAL` x `INST`)

| Variable &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; | Depends on | Description |
| --- | ---- | ---- | 
| `ANNOT_MASK` | `epoch` | Flag whether this annotation instance is included or excluded (`1` means _masked_ or excluded) 
| `EPOCH_MASK` | `epoch` | Flag whether this epoch is included or excluded (`1` means _masked_ or excluded) 


<h3>Example</h3>

_to be added_


## SPANNING

_Summarize the coverage of an EDF by one or more annotations_

Produces information about the coverage of an EDF by a group of one or
more annotations.  This can be used to check whether sleep stage
annotations cover the whole recording or not; it will also report on
annotations that are outside the duration of the EDF, and whether any
of the annotations in the group overlap each other.

The report treats all annotations in the group as identical, asking
what proportion of the EDF is spanned by _one or more_ of the
specified annotations.

!!! note 
    Currently, this command can only be applied to continuous,
    unmasked EDFs (i.e. not discontinuous EDF+ files, not EDFs after
    running the `MASK` or `RE` commands.

<h3>Parameters</h3>

|  Parameter | Example | Description |
| --- | --- | --- |
| `annot` | `annot=N1,N2,N3,R,W` | Annotation(s) to group and report on |


<h3>Output</h3>

_Class-level_ annotation summary (strata: _none_ )

| Variable | Description |
| --- | --- |
|`REC_HMS` |  EDF recording duration (hh:mm:ss)
|`REC_SEC` |   EDF recording duration (seconds)
|`ANNOT_N` |  Number of annotations in group
|`ANNOT_SEC` | Total (potentially overlapping) annotation duration (secs)
|`ANNOT_HMS` | Total (potentially overlapping) annotation duration (hh:mm:ss)
|`ANNOT_OVERLAP` | Do any annotations in group overlap w/ one another (0/1)?
|`VALID_N` |  Number of valid annotations, ANNOT_N - INVALID_N
|`INVALID_N` | Number of annotations that over-extend EDF duration
|`INVALID_SEC` |  Total duration of all annotation beyond EDF end
|`SPANNED_SEC` |  Duration of EDF spanned by 1+ of these annotations (secs)
|`SPANNED_HMS` |  Duration of EDF spanned by 1+ of these annotations (hh:mm:ss)
|`SPANNED_PCT` |  % of EDF spanned by 1+ of these annotations
|`UNSPANNED_SEC` |  Duration of EDF unspanned by 1+ of these annotations (secs)
|`UNSPANNED_HMS` | Duration of EDF unspanned by 1+ of these annotations (hh:mm:ss)
|`UNSPANNED_PCT` |  % of EDF unspanned by 1+ of these annotations


List of _invalid_ annotations (strata: `N` )

| Variable | Description |
| --- | --- |
| `ANNOT` | Annotation class |
| `INST` | Annotation instance ID |
| `START`| Start (seconds) |
| `STOP` | Stop (seconds) |


<h3>Example</h3>

Here we use `SPANNING` on the first [tutorial](../tut/tut1.md)
individual.  For example, given the sleep stage annotations, we might
want to check that all epochs are spanned by one (and only one) stage
annotation:

```
luna s.lst 1 -o out.db -s SPANNING annot=NREM1,NREM2,NREM3,REM,wake
```
```
destrat out.db +SPANNING | behead
```
```
                       ID   nsrr01              
                ANNOT_HMS   11:21:30            
                  ANNOT_N   1363                
            ANNOT_OVERLAP   0                   
                ANNOT_SEC   40890               
                INVALID_N   0                   
              INVALID_SEC   0                   
                  REC_HMS   11:22:00            
                  REC_SEC   40920               
              SPANNED_HMS   11:21:30            
              SPANNED_PCT   99.9266862170088    
              SPANNED_SEC   40890               
            UNSPANNED_HMS   00:00:30            
            UNSPANNED_PCT   0.0733137829912023  
            UNSPANNED_SEC   30.0                
                  VALID_N   1363                
```

The above tells us that 99.93% (`SPANNED_PCT`) of the EDF is spanned
by one of these annotations.  Furthermore, there is no overlap among
this set of annotations (`ANNOT_OVERLAP` is 0).  See do we do see
there is 30 seconds (i.e. one epoch) not covered by any of these
annotations (`UNSPANNED_SEC`, i.e. 0.07% as noted by `UNSPANNED_PCT`).

Looking at the console/log output, we can see why this is the case: we omitted `NREM4` 
which is also present as a staging annotation in this case:
```
  [NREM1] 109 instance(s) (from edfs/learn-nsrr01-profusion.xml)
  [NREM2] 523 instance(s) (from edfs/learn-nsrr01-profusion.xml)
  [NREM3] 16 instance(s) (from edfs/learn-nsrr01-profusion.xml)
  [NREM4] 1 instance(s) (from edfs/learn-nsrr01-profusion.xml)
  [REM] 238 instance(s) (from edfs/learn-nsrr01-profusion.xml)
```

Re-running with this included now shows that these annoations a)
completely span the EDF (`SPANNED_PCT` is 100%) without any overlap
(`ANNOT_OVERLAP` is 0):

```
luna s.lst 1 -o out.db -s SPANNING annot=NREM1,NREM2,NREM3,NREM4,REM,wake
```
```
                       ID   nsrr01              
                ANNOT_HMS   11:22:00            
                  ANNOT_N   1364                
            ANNOT_OVERLAP   0                   
                ANNOT_SEC   40920               
                INVALID_N   0                   
              INVALID_SEC   0                   
                  REC_HMS   11:22:00            
                  REC_SEC   40920               
              SPANNED_HMS   11:22:00            
              SPANNED_PCT   100                 
              SPANNED_SEC   40920               
            UNSPANNED_HMS   00:00:00            
            UNSPANNED_PCT   0                   
            UNSPANNED_SEC   0                   
                  VALID_N   1364                
```

As an alternative (and to demonstrate using Luna in different ways), we
could have identified the "unspanned" epoch and its annotations as
follows, using the `ANNOTS` command (described [above](#annots)) on the
EDF after excluding epochs with one of the above five annotations:

```
luna s.lst 1 -o out.db -s 'MASK mask-if=NREM1,NREM2,NREM3,REM,wake & RE & ANNOTS'
```
Looking at the output, we see a `NREM4` annotation:
```
destrat out.db +ANNOTS -r ANNOT 
```

``` 
ID      ANNOT     COUNT   DUR
nsrr01  NREM4     1       30
nsrr01  hypopnea  1       15.3
```

