# Manipulations

_Commands to alter basic properties of the EDF and the signals therein_

| Command | Description |
| -----  | ----- | 
|[`SIGNALS`](#signals) | Retain/remove specific EDF channels |
|[`COPY`](#copy) | Duplicate one or more EDF channels |
|[`RESAMPLE`](#resample) | Resample signal(s) | 
|[`REFERENCE`](#reference) | Re-reference signals |
|[`CANONICAL`](#canonical) | Generate _canonical_ signals |
|[`MINMAX`](#minmax) | Set digital/physical min/max across channels |
|[`uV`](#uv) | Rescale units to uV |
|[`mV`](#mv) | Rescale units to mV |
|[`FLIP`](#flip) | Flip polarity of signal | 
|[`EDF`](#edf) | Force EDF (versus EDF+) state |
|[`TIME-TRACK`](#time-track) | Add a time-track to an EDF |
|[`RECORD-SIZE`](#record-size) | Change EDF record size |
|[`ALIGN`](#align) | Realign EDF records, annotations and epochs |
|[`ANON`](#anon)       | Strip ID information from EDF header |
|[`SIGGEN`](#siggen) | Generate/spike in artificial test signals |

## SIGNALS

The command requires one of two options: _either_ `keep` _or_ `drop`.
Each expects a comma-delimited list of channel names (or
[_aliases_](../luna/args.md#aliases)), which are either retained or
removed from the in-memory dataset.

<h3>Parameters</h3>

| Option | Example | Description | 
| ---- | ----- | ----- | 
| `drop` | `drop=EMG,ECG`  | Drop channels `EMG` and `ECG` |
| `keep` | `keep=C3,C4` | Drop all channels _except_ `C3` and `C4` |

<h3>Outputs</h3>

Other than modifying the _in-memory_ representation of the EDF, there
is no further output (except some notes written to the log).

<h3>Example</h3>

For an EDF with 6 signals, including `EMG`, `EOG-L` and `EOG-R`, this command would drop these three signals:

```		
luna s.lst -s "SIGNALS drop=EMG,EOG-L,EOG-R & DESC"
```
as shown by the relevant lines in the output from `DESC`:
```
Number of signals : 3
Signals           : EEG1[256] EEG2[256] EEG3[256]
```
In contrast, the `keep` option with the same arguments: 
```
luna s.lst -s "SIGNALS keep=EMG,EOG-L,EOG-R & DESC"
```
yields the expected output:
```
Number of signals : 3
Signals           : EOG-L[256] EOG-R[256] EMG[256]
```

## COPY

_Duplicates one or more EDF channels_

Because some Luna commands modify a channel
(e.g. [`FILTER`](fir-filters.md#filter)), it can be desirable to first
make a copy of the original channel.  New channels are written out
with the [`WRITE`](outputs.md#write) command.

Although multiple signals can be duplicated at the same time
(i.e. will all be given the same tag), only data channels (i.e. not
EDF Annotation channels in EDF+) are duplicated.

<h3>Parameters</h3>

| Parameter | Example | Description | 
| --- | --- | --- | 
| `sig` | `sig=C3,C4` | List of channels to duplicate |
| `tag` | `tag=DELTA` | A required option, this is added to make the new channel name, e.g. `C3` becomes `C3_DELTA` | 

<h3>Output</h3>

One or more new channels are created in the _in-memory_ representation
of the EDF.  Aside from a note in the log, there is no formal
(destrat-based) output for this command.

<h4>Example</h4>

To extract one channel (`EEG`) from an original EDF, and then duplicate it:

```
luna s.lst 2 sig=EEG -s 'DESC & COPY sig=EEG tag=V2 & DESC'
```

As expected, the first `DESC` output shows a single channel:
```
EDF filename      : edfs/learn-nsrr02.edf
ID                : nsrr02
Clock time        : 21:18:06 - 07:15:36
Duration          : 09:57:30
# signals         : 1
Signals           : EEG[125]
```

After the `COPY` command has been executed, there are now two channels: `EEG` and `EEG_V2`:
```
EDF filename      : edfs/learn-nsrr02.edf
ID                : nsrr02
Clock time        : 21:18:06 - 07:15:36
Duration          : 09:57:30
# signals         : 2
Signals           : EEG[125] EEG_V2[125]
```


## RESAMPLE

_Changes the sampling rate of a signal_

Uses functions from `libsamplerate` to upsample or downsample signals.
Within a maximum upsampling/downsampling of 256, there are no
constraints on the new sample rate (i.e. the ratio of old and new
sample rates need not be a rational number).

<h3>Parameters</h3>

| Parameter | Example | Description | 
| --- | --- | --- | 
| `sig` | `sig=C3,C4` | Signal list |
| `sr` | `sr=100` | New sampling rate (Hz) | 

<h3>Output</h3>

No output other than a message to the log (and altering the in-memory
signal).

<h3>Example</h3>

To create a new EDF with the `EEG` channel resampled to 100 Hz:

```
luna s.lst -s 'RESAMPLE sig=EEG sr=100 & WRITE edf-tag=resample edf-dir=edfs/ sample-list=s2.lst'
```


## REFERENCE

_Re-references signals with respect to one or more other signals_

<h3>Parameters</h3>

| Parameter | Example | Description | 
| --- | --- | --- |
| `sig` | `sig=C3,C4` | Signal(s) to re-reference | 
| `ref` | `ref=A1,A2` | Signal(s) to provide the reference | 

Both `sig` and `ref` are required parameters. If more than one
channel is given as the reference (in a comma-delimited list), the
average of those channels is used as the reference value.

<h3>Output</h3>

No output, other than a note to the log.  In memory, the updated
`sig` channels will contain the re-referenced values.

## CANONICAL

_Generates so-called canonical signals given a set of rules_

Particular algorithms may often expect a particular type of signal:
for example, automated sleep staging may expect an EEG, EOG and/or EMG
channel.  Those downstream algorithms may require certain naming
conventions, units, sampling rates and/or referencing schemes to hold.
On the other hand, those algorithms may also allow for alternate
choices, e.g. if a C4/M1 channel is not present, then use C3/M2.  The
intention of the `CANONICAL` command is to faciliate this type of
pre-processing, to generate signals that are _canonical_ in the sense
that they constitute a uniform set for further processing.

The key behind this command is a text file that defines the canonical
signals.  This should be a plain-text file; comments can be included,
by starting the line with a `%` character (the same of Luna command
scripts).  Uncommented lines group should contain either six or seven
tab-delimited columns:

| Column | Description |
| --- | --- |
| _GROUP_ | Matches the `group` option on the CANONICAL command |
| _TYPE_ | Currently one of: `EEG`, `LOC`, `ROC`, `EMG` or `ECG` |
| _CHANNEL_ | The channel that represents the canonical form |
| _REFERNECE_ | If needed, the reference channel to achieve canonical form (or `.` if not needed) |
| _SR_ | The desired sampling rate for the canonical channel |
| _UNIT_ | The desired physical units for the canonical channel (or `.` if no change needed) |
| _NOTES_ | Optionally, a seventh (tab-delimited) field that contains notes on this rule, which will be recorded in the output |

Canonical signals can currently only be one of the following types:
`EEG`, `LOC`, `ROC`, `EMG` and `ECG`.  Note: _canonical signals_ are
not to be confused with [channel
types](#../luna/args.md#channel-types).  A canonical signal represents
a _new_ signal that is generated from existing signals according to a
set of rules.  A _channel type_ is simply a label or annotation that
is given to all channels, based on their channel name.  The reason for
the `CANONICAL` command is that often datasets (such as those in the
NSRR) can EDFs with mixtures of conventions and montages.  Thus, this
command is designed to be able to, e.g. "extract a central EEG" across
multiple studies.

To make things more concrete: consider this example __tab-delimited__ plain text file `cs.txt`:
```
TEST  EEG  C4,EEG2 .      100  uV   If 'C4' not present, look for 'EEG2' instead
TEST  EMG  Lchin   Rchin  100  uV 
TEST  LOC  LOC     M2,A2  100  uV   If 'M2' not present, look for 'A2' instead
TEST  ROC  E2,ROC  M2,A2  100  uV   If 'E2' not present, look for 'ROC' instead
TEST  ECG  ECG     .      100  mV
```

Here we see five rules that define five canonical channels.  If run
with the option `prefix=cs` as above, these channels will be added to
the (in-memory) EDF as `csEEG`, `csEMG`, etc, and can be included in
any analysis, or writen to a new EDF (with the
[`WRITE`](outputs.md#write) command).

!!! Note "Channel label clashses"
    If there is already a channel called `EEG`, then Luna will not
    overwrite it with a new canonical channel.  In this instance, use the
    `prefix` option to specify a new canonical name: e.g. `prefix=cs` will
    make the new channel `csEEG` rather than just `EEG`, which can be used
    to avoid naming clashes.

The first channel, which is the _canonical EEG_ is defined either `C4`
or `EEG2`.  That is, if `C4` is not present, then `EEG2` will be used
to generate `csEEG`, if present.  The `.` indicates that no
re-referencing is needed; the `100` indicates that `EEG` will be
resampled to 100 Hz, if needed.  Finally, the sixth column has an optional note
describing this rule. The second line shows the rule of the _canonical EMG_, which is
defined as the channel `Lchin` re-referenced against the channel
`Rchin`. The other lines continue in this manner: for example, `csLOC` will
reflect `LOC` either re-referenced against `M2` or `A2`.

It is possible to have different rules for the same canonical signal,
by listing them on different lines; Luna will select the first rule
that matches. For example, if within a given cohort some individuals
have left/right EMG as separate channels, but other individuals have a
single channel that is already left-right referenced.  For a
_canonical_ EMG in the first instance you'd want to take left and
right as two channels and perform the re-referencing; in the second
instance, you'd want to take the already re-referenced channel as is.
Therefore:

```
TEST    EMG     lchin   rchin   100   uV
TEST    EMG     chin    .       100   uV
```

That is, if Luna can't find `lchin` and `rchin` in the EDF, it will
search for for `chin` instead.  In this way, one can handle a
heterogeneous set of EDFs but select a single examplar/canonical
channel to reflect a given type of signal. Note: having different
rules (lines) for a given canonical signal implies substantively
different channels (location/referencing); within a rule, the
comma-delimited list only implies superficial labelling differences.

   
Some further notes:

 * Using a comma-delimited list on the same rule only means to take
   the first available channel of that label; in that case, you cannot have a `.`
   included as part of a reference channel list though:
```
TEST EEG  C4   M1,.  100      <--- not allowed
```
   This should be written as two rules:
```
 TEST EEG  C4   M1  100
 TEST EEG  C4   .   100
```
  which implies that, _if_ `M1` is present in the EDF, then `C4` should
  be re-referenced against it; otherwise, just take `C4` (i.e. assuming
  that `C4` has already been re-referenced).


* Do not assume that values are _paired_ for signal and reference 
  comma-delimited lists: i.e. 
  ```  
  TEST EEG C4,C3  M1,M2   100
  ```
  does *not* imply that Luna will take either `C4/M1`, or else `C3/M2`, 
  (or nothing, if none of these exist).
  Rather, here, it would take `C3` referenced against `M1`, if say, only `C3` and `M1`
  were present in the EDF.  To specify the logic of `C4/M1` or `C3/M2` 
  instead use two different rules:
```
TEST EEG C4  M1   100  uV
TEST EEG C3  M2   100  uV
```

 * Do not replicate channel aliases in these files; i.e. this command is intended to 
 be used with standard Luna channel aliases, e.g. from `@include` files.  For example, if `EEG2` 
 is already an alias for `EEG(sec)`, `EEG 2`, `EEG (sec)` and `EEG sec`, e.g. if `sig.alias` is a file
 containing:
```
 alias  EEG2|EEG(sec)|"EEG 2"|"EEG (sec)"|"EEG sec" 
```
  then when using `CANONICAL` we can just write:
```
 TEST EEG  EEG2 .  100  uV
```
and it will match `EEG2` to any of those aliases in the standard manner.  That is, 
there no need to add all the aliases in again, e.g. when running the following:
```
luna s.lst @sig.alias -s 'CANONICAL file=cs.txt group=TEST'
```

 * This command is set up with a typical PSG in mind, where a
   single EEG or single ECG is often all that is required, e.g. for a
   given staging or HRV analysis, etc.  In principle, this command
   can be extended to define more canonical types (e.g. a _canonical
   frontal EEG_, or _canonical oxygen desaturation signal_, etc), which
   we may do in future releases.

<h3>Parameters</h3>

| Parameter | Example | Description |
| --- | --- | --- |
| `sig` | `sig=C3,C4` | Signals (two or more) to set group min/max values |
| `file` | `file=cs.txt` | Specify the definitions of canonical signals |
| `group` | `group=SHHS` | Specifying the group to use |
| `cs` | `cs=EEG,EMG` | Only create these canonical channels |
| `prefix` | `prefix=cs` | Adds an optional prefix to canonical signals | 

<h3>Output</h3>

The primary action of this command is to add new signals to the in-memory EDF, e.g. `csEEG`.

Canonical signal information (strata: `CS`)

| Variable | Description |
| --- | --- |
| `DEFINED`  | 0/1 for whether this canonical signal could be created for this EDF |
| `SIG` | Primary signal used |
| `REF` | Reference signal used (if any) |
| `SR` | Sampling rate of canonical signal |
| `UNITS` | Physical dimension of the canonical signal |
| `NOTES` | Any notes (from the definition file) for the rule applied |

<h3>Example</h3>

Consider the following (admittedly, not particularly compelling) example of how to use `CANONICAL`.
We have a file `cs.txt` that defines the canonical signals for this group:

```
G1	EEG	EEG	.	100	uV
G1	EMG	EMG	.	100	uV
G1	LOC	EOG(L)	.	100	uV	
G1	ROC	EOG(R)	.	100	uV
G1	ECG	ECG	.	100	mV
```
We apply it to a test EDF:
```
luna s.lst 1 -o out.db -s 'CANONICAL file=cs.txt group=G1 prefix=cs & DESC'
```
This reports what is being done in the log:

```
  generating canonical signal csEEG from EEG/.
  resampling channel csEEG from sample rate 125 to 100
  generating canonical signal csLOC from EOG(L)/.
  resampling channel csLOC from sample rate 50 to 100
  generating canonical signal csROC from EOG(R)/.
  resampling channel csROC from sample rate 50 to 100
  generating canonical signal csEMG from EMG/.
  resampling channel csEMG from sample rate 125 to 100
  generating canonical signal csECG from ECG/.
  resampling channel csECG from sample rate 250 to 100
```
And we can see the output of `DESC` (also in the log) that confirms new channels have been created:

```
Signals : SaO2[1] PR[1] EEG(sec)[125] ECG[250] EMG[125] EOG(L)[50]
          EOG(R)[50] EEG[125] AIRFLOW[10] THOR_RES[10] ABDO_RES[10] POSITION[1]
          LIGHT[1] OX_STAT[1] csEEG[100] csLOC[100] csROC[100] csEMG[100]
          csECG[100]
```
We can also see the summary of what was done in the output:

```
destrat out.db +CANONICAL -r CS
```
```
ID       CS      DEFINED  NOTES   REF   SIG     SR    UNITS
nsrr01   csEEG  1        .       .     EEG     100   uV
nsrr01   csLOC  1        .       .     EOG(L)  100   uV
nsrr01   csROC  1        .       .     EOG(R)  100   uV
nsrr01   csEMG  1        .       .     EMG     100   uV
nsrr01   csECG  1        .       .     ECG     100   mV
```

## MINMAX

Set digitial and physical minimum and maximum values in the EDF header
to be equal across multiple channels.  This can be necessary to enable
other software to be able to work with an EDF, by making it better
conform to the EDF specification.  Signals specified here must be
comparable, e.g. all EEG and EOG with a common amplifier and ADC, and
so are expected to have similar scaling and sensitivity (unit/bit) in
the EDF.

<h3>Parameters</h3>

| Parameter | Example | Description |
| --- | --- | --- |
| `sig` | `sig=C3,C4` | Signals (two or more) to set group min/max values |


<h3>Output</h3>

No formal output is given. The channels are rescaled internally.  Any
subsequent commands (i.e. including `WRITE` to write a new EDF) will
therefore based based on these new header values.

<h3>Example</h3>

Here we have an EDF with channels C3, C4, F3, F4, O1, O2, A1 and A2. The `HEADERS`
command shows that the channels have different physical min/max values, and therefore
different `SENS` values (scaling of micro-volts per bit in the EDF):

```
luna id01.edf -o out.db -s HEADERS
```
```
destrat out.db +HEADERS -r CH
```
```
ID        CH  DMAX   DMIN    PDIM  PMAX    PMIN     SENS    SR   TYPE
id01.edf  F3  32767  -32768  uV    1574.8  -3276.8  0.07403 500  EEG
id01.edf  F4  32767  -32768  uV    1720.9  -3276.8  0.07626 500  EEG
id01.edf  C3  32767  -32768  uV    3276.7  -3276.8  0.1     500  EEG
id01.edf  C4  32767  -32768  uV    2034.5  -3276.8  0.08104 500  EEG
id01.edf  O1  32767  -32768  uV    3276.7  -3276.8  0.1     500  EEG
id01.edf  O2  32767  -32768  uV    3276.7  -3276.8  0.1     500  EEG
id01.edf  A1  32767  -32768  uV    3238.6  -3276.8  0.09941 500  EEG
id01.edf  A2  32767  -32768  uV    3255.3  -3276.8  0.09967 500  EEG
```

After running the `MINMAX` command, we see that the `SENS` values are now
set to be equal across all channels.  This command will not fundamentally
change the underlying signal data, only the scaling in the EDF header.

```
luna id01.edf -o out.db -s 'MINMAX & HEADERS'
```
```
ID        CH  DMAX   DMIN    PDIM  PMAX    PMIN     SENS  SR   TYPE
id01.edf  F3  32767  -32768  uV    3276.7  -3276.8  0.1   500  EEG
id01.edf  F4  32767  -32768  uV    3276.7  -3276.8  0.1   500  EEG
id01.edf  C3  32767  -32768  uV    3276.7  -3276.8  0.1   500  EEG
id01.edf  C4  32767  -32768  uV    3276.7  -3276.8  0.1   500  EEG
id01.edf  O1  32767  -32768  uV    3276.7  -3276.8  0.1   500  EEG
id01.edf  O2  32767  -32768  uV    3276.7  -3276.8  0.1   500  EEG
id01.edf  A1  32767  -32768  uV    3276.7  -3276.8  0.1   500  EEG
id01.edf  A2  32767  -32768  uV    3276.7  -3276.8  0.1   500  EEG
```

Note, if the EDF contained other signals that you did not want
included in the `MINMAX` procedure (e.g. respiratory channels, which
have different scaling from EEG channels), you would need to add `sig`
after `MINMAX` to specify, e.g. only the EEG channels.  This command
will skip any EDF+ Annotation channels automatically.


## uV 

_Converts a signal to uV units_ 

Checks the `unit` (physical dimension) field of the EDF header for either `V`, `mV` or `uV`
and rescales the signal appropriately.  If the header specifies some
other unit, or none, then no action is taken.

<h3>Parameters</h3>

| Parameter | Example | Description | 
| --- | --- | --- |
| `sig` | `sig=C3,C4` | Signal(s) to convert |

If `sig` is not specified, this command is applied to all channels.

<h3>Output</h3>

No output, other than updating the in-memory signal.


## mV

_Converts a signal to mV units_ 

Checks the `unit` (physical dimension) field of the EDF header for either `V`, `mV` or `uV`
and rescales the signal appropriately.  If the header specifies some
of unit, or none, then no action is taken.

<h3>Parameters</h3>

| Parameter | Example | Description | 
| --- | --- | --- |
| `sig` | `sig=C3,C4` | Signal(s) to convert |

If `sig` is not specified, this command is applied to all channels.

<h3>Output</h3>

No output, other than updating the in-memory signal.

## TIME-TRACK

_Adds a time-track, which implicitly converts an EDF into an EDF+_

This command is only used internally, currently.  


## FLIP

_Flips the polarity of a signal_ 

Multiplies every sample value of a signal by -1.

<h5>Parameters</h5>

| Parameter | Example | Description |
| --- | --- | --- |
| `sig` | `sig=C3,C4` | Signals to flip |

<h3>Output</h3>

No output, other than a message to the log and an updated in-memory signal.

<h3>Example</h3>

This next command takes the first 10 epochs of the `C3` signal,
outputs the original signal to a file (`f1`), then flips the signal,
and re-outputs it (to `f2`): 


```
luna me.lst sig=C3 -s 'EPOCH & MASK epoch=1-10 & \
                       RESTRUCTURE & MATRIX file=f1 & \
                       FLIP & MATRIX file=f2'
```

!!! note
    In the above, we used the end-of-line `\` character (with no trailing whitespace) 
    to continue the command on multiple lines, as many shells allow.

Comparing the original signals (looking at just the first 10 rows of output) ...

```
head f1
```
```
ID       E   S   SP T             C3
id001    1   0   0  0             3.43407
id001    1   0   1  0.00390625    2.06044
id001    1   0   2  0.0078125    -0.0763126
id001    1   0   3  0.0117188    -1.60256
id001    1   0   4  0.015625     -2.21306
id001    1   0   5  0.0195312    -2.21306
id001    1   0   6  0.0234375    -2.21306
id001    1   0   7  0.0273438    -2.06044
id001    1   0   8  0.03125      -2.06044
```

... to the new signals, we see the values have been flipped, albeit
not as exactly as one might expect.  (This is due to the encoding
used by EDFs; see the note below for more details).


```
head f2
```
```
ID       E   S   SP T             C3
id001    1   0   0  0            -3.43865
id001    1   0   1  0.00390625   -2.06244
id001    1   0   2  0.0078125     0.0728122
id001    1   0   3  0.0117188     1.59799
id001    1   0   4  0.015625      2.20806
id001    1   0   5  0.0195312     2.20806
id001    1   0   6  0.0234375     2.20806
id001    1   0   7  0.0273438     2.05909
id001    1   0   8  0.03125       2.05909
```

!!! warn "Floating point accuracy" 
    EDFs store data as 2-byte
    integers: in contrast, floating point numbers as used
    in Luna typically take up 4 or 8 bytes in memory.  This relatively
    low resolution of EDF introduces slight numerical differences so
    that the values are clearly different from -1 times the original:
    i.e. `3.43407` is not minus `-3.43865`.  As noted in the EDF spec,
    practically this limit on resolution is not a real issue for most
    biosignals, if they are recorded with sensible physical and
    digital min/max values to reflect the dynamic range of the signal.

## EDF

_Sets/forces EDF status for a EDF+_

This command downcasts an EDF+ to a standard EDF file.  This means that

 - any `EDF Annotations` channels will be dropped

 - if the file is a `EDF+D` (i.e. a potentially discontinuous EDF), then it is converted to a standard EDF if it _is actually_ continuous (i.e. is a EDF+D but does not actually contain any gaps)

 - if the `force` option is given, then the conversion to EDF is done _even if the EDF+D actually contains gaps_.

<h5>Parameters</h5>

| Parameter | Description
| ---- | ---- |
| `force` | Change a EDF+D to EDF even if it actually contains gaps |

<h5>Output</h5>

No output other than changing the internal status of the EDF

<h5>Example</h5>

_to be added_



## RECORD-SIZE

_Alters the record size of an EDF_

This command changes the low-level encoding of data in an EDF, which is
something that you should not normally need to change.  Often, EDFs
have a record size (i.e. the size of the _blocks_ in which the data
are stored) of 1 second or so.  Why might you want to change this?  

 - as the smallest `EPOCH` size is limited by the EDF record size, if
   the EDF record size is relatively large (e.g. 10 seconds), it will
   not be possible to specify smaller epochs (e.g. 5 seconds). 

 - if the EDF record size is very small (e.g. 100 milliseconds), this
   can reduce performance when reading the EDF from disk


There are a number of points that should be borne in mind:

 - no subsequent commands can be issued after a `RECORD-SIZE` command;
   rather, a new EDF will be written to disk
 - you should ensure that the new record size contains an integer number of samples for all signals
 - currently, you can only change the record size of EDF, not EDF+ files
 - as only whole records are written to disk, the final part of an EDF
   (that is shorter than the new record size) may be truncated

<h3>Parameters</h3>

| Parameter | Example | Description |
| --- | --- | --- |
| `dur` | `dur=1` | New EDF record/block size |
| `edf-dir` | `edf-dir=edfs/` | Folder for writing new EDFs |
| `edf-tag` | `edf-tag=rec1` | Tag added to new EDFs |
| `sample-list` | `sample-list=s2.lst` | Generate a sample-list pointing to the new EDFs |

That is, while `RECORD-SIZE` itself only takes `dur` as the single
option, one must also specify all options for
[`WRITE`](outputs.md#write), as `RECORD-SIZE` automatically triggers
`WRITE` after changing the record size of the in-memory
representation. (That is, as always, the original EDF file is left
untouched.)

<h3>Output</h3>

No output, other than message to the log and an updated in-memory signal.

<h3>Example</h3>

Focusing only on the signals `PR` and `EEG` in the first
[tutorial](../tut/tut1.md) EDF, we see that this EDF has a record size
of 1 second:

```
luna s.lst 1 sig=PR,EEG -s "SUMMARY" 
```
```
# signals      : 2
# records      : 40920
Duration       : 1
```

That is, the EDF has 40,920 records, each of duration 1 second.
Looking at the two signals, because the record duration is 1 second,
this implies a sample rate of 1 Hz and 125 Hz respectively for `PR`
and `EEG`.


```
Signal 1 : [PR]
       # samples per record : 1
...
Signal 2 : [EEG]
       # samples per record : 125
...
```

To generate a new EDF (which contains only these two signals) with an
altered record size (in this example, 50 seconds):

```
luna s.lst 1 sig=PR,EEG -s "RECORD-SIZE dur=50 edf-tag=r50" 
```

(Note that setting a 50-second record size would be unusual, this is
done here purely for illustrative purposes.)  After running this
command, you'll see the following messages in the log:

```
 saved new EDF, edfs/learn-nsrr01-r50.edf
 **warning: the PROBLEM flag was set, skipping to next EDF...
```

The warning message is expected, this is just Luna's way of ensuring
that no further commands can be run after `RECORD-SIZE` command.
Running `SUMMARY` on the new EDF, we see that the record size has been
changed:

``` 
luna edfs/learn-nsrr01-r50.edf -s SUMMARY
```

```
# records      : 818
Duration       : 50
...
Signal 1 : [PR]
       # samples per record : 50
...
Signal 2 : [EEG]
       # samples per record : 6250
...
```

That is, instead of 40,920 records of 1 second we have 818 records of
50 seconds. Correspondingly, there are now 50 times the number of
samples per record compared to the original EDF (the sample rate in Hz
is obviously the same as before).

!!! note 
    Because 40,920 is not evenly divisible by 50, the last 20
    seconds has been truncated (i.e. the log will indicate a total
    duration of `11:21:40` instead of the original `11:22:00`).

## ALIGN

_Alters the EDF record structure to perfectly align records with annotations and epochs_

This is a rather specialized command to fix certain issues that can arise when working
with EDFs and epoch-defining annotations when the annotations are not "aligned" with EDF records.

Consider the following (which occurs in the NSRR National Children's Hospital dataset):

 - all EDFs start on a clock-time with an exact, integer number of seconds (i.e. EDF spec. does not allow otherwise), e.g. `23.03.07`

 - sample rate is 256 Hz for all EEG channels, and the EDF record size is 4 seconds

 - annotations that define sleep stages are all in exactly 30-second epochs, but they start at a fractional offset:
   ```   
   Stage   Onset (elapsed seconds from EDF start)
   N1 	   4110.9609375 
   N2 	   4140.9609375 
   N2 	   4170.9609375 
   ```

This means that annotations and EDF records not in exact temporal
alignment.  In theory, there is of course absolutely nothing _wrong_
per se with this.  In practice, it can make some Luna commands, or
more generally, working with EDFs (that have a fixed block size) a
little challenging.  That is, the EDF record is an atomic unit: one
must either write a whole record, or none of it, when constructing a
new EDF.  If epochs only partially span EDF records, this naturally
poses problems when thinking about how to write a new EDF that is
structured by those epochs, i.e. only NREM sleep epochs, etc.

Here we break this down three distinct issues, only the third of which
is more challenging and requires the `ALIGN` command.

<h6>Offsets for EDF annotations</h6>

Specifically, when counting epochs, by default Luna starts at 0
seconds elapsed (i.e. the EDF start) and proceeds forwards: 0-30,
30-60, etc.  If sleep stages start at some different offset (other
than starting at 0 or a multiple of 30 seconds) this means that some
Luna-defined epochs may contain more than one implied stage.  For example,
consider that sleep stages start at 4 seconds past the EDF start:

```
      Elapsed seconds from EDF start:
      0000000000 1111111111 2222222222 3333333333 4444444444 5555555555 666...
      0123456789 0123456789 0123456789 0123456789 0123456789 0123456789 012...

Luna: |---epoch 1----------------------|
                                       |---epoch 2----------------------|

Actual:   |---epoch 1-(W)------------------|
                                           |---epoch 2-(N1)------------------|
```

As in the above example, the second Luna epoch (30-60) will span two
distinct annotations, which is clearly a mistake.  This means that
commands that use sleep stages will report (in this example) a missing
(`?`) annotation, as there is a conflict in that _epoch 2_ has both a
wake (`W`) and N1 (`N1`) annotation.   Luna will also flag in the log that
conflicts were found, e.g.:

```
 CMD #1: HYPNO
   options: sig=*
  set epochs to default 30 seconds, 1186 epochs
  *** found 37 epoch(s) of 1186 with conflicting spanning annotations
  *** check that epochs and annotations align as intended
  *** see EPOCH 'start-annot' or 'offset' options
```
and write `CONF` variable to the standard output.

The solution to this is simply to supply an offset to the epoch command, which can either be done
manually:
```
luna s.lst -s 'EPOCH offset=4 & HYPNO`
```
or automatically with respect to a set of annotations (i.e. making the offset equal to the start of the first of these annotions encountered):
```
luna s.lst -s 'EPOCH start-annot=N1,N2,N3,R,W,? & HYPNO'
```

<h6>EDF record size and epoch duration</h6>

Another issue arises when epochs may span EDF records only
partially. The core data structure (of EDF, and internally of Luna) is
a block/record based system: the underlying data must correspond to
whole records. If epochs span partial EDF records, this presents a
problem if we want to restructure the data, e.g. remove W epochs and
write a new EDF

In the above example, with an EDF record duration of 4 seconds, one
cannot straightforwardly output a new EDF on an epoch-by-epoch basis,
as for 30 second epochs), an epoch is not perfectly divisible by 4. In
Luna, this can be fixed by using the `RECORD-SIZE` command to change
the EDF record size, e.g. to 1 second say.  Now, Luna would be
outputting only whole EDF records (i.e. 30 times 1 second) rather than
having to worry about a partial record (i.e. 7 times 4 seconds, plus
an additional two seconds, which would be half an EDF record).

<h6>Fractional offsets</h6>

Finally, a more tedious issue arises if the annotations/epochs _start_ at
arbitray, fractional time points within an EDF.  In the example above, we, for example, 
an annotation starting:
```
   N1      4110.9609375
```
In this example, with a sample rate of 256 Hz, this offset corresponds exactly to 246 (= 0.9609375 * 256 ) samples
past the start of each second.  Although one solution would be to have a very short EDF record sizes (i.e. the length of
one sample, 1/256 seconds), this may for technical reasons make data access suboptimal for many readers; further, we consdier the
more general case, where the offset is not even exactly aligned with a sample point. 

The `ALIGN` command attempts to handle this issue, by creating a new
EDF in which the epochs are perfectly aligned to EDF records.

One specifies a list of annotations that will define epochs: these annotations must be a) non-overlapping, and b)
an exact multiple of the EDF record size.  i.e. if we have changed the above data to have 1 second records, then we can
specify 30-second epochs.  The `ALIGN` command then finds all sample points within an annotation that fits completely within the observed duration
(i.e. only ‘complete’ annotations, to skip those spans discontinuities/borders) and maps these into a new set of records
(e.g. 30 new 1 second records) for each annotation (i.e. which may be mapping fractionally across original records). The end result is a new EDF
with in which annotations, epochs & EDF records are all aligned.

To make this concrete: consider (for ease of presentation) a 5 Hz signal and an EDF record size of 1 second:

```
    RECORD   SAMPLE    TIME
    1        1         0.0
    1        2         0.2
    1        3         0.4
    1        4         0.6
    1        5         0.8

    2        1         1.0
    2        2         1.2
    2        3         1.4
    2        4         1.6
    2        5         1.8

    3        1         2.0
    3        2         2.2
    3        3         2.4
    3        4         2.6
    3        5         2.8
```

Also for simplicity of presentation, assume here we have 1 second epoch (rather than 30 seconds)

If we were to have epoch-defining annotations starting with an offset
of 0.25 seconds, (e.g. 0.25 - 1.25, 1.25 - 2.25, etc.)  The challenge
would be, for example: how to extract those epochs into a new EDF.
Naturally, this involves changing the internal structure of the EDF.

We first collect all the new data-points, making new 5-sample EDF records from the new data, starting at the first point after 0.25 seconds:

```
    RECORD   SAMPLE    TIME           NEW RECORD  SAMPLE    TIME 
    1        1         0.0      xxx
    1        2         0.2      xxx
    1        3         0.4      --->      1       1         0.0 
    1        4         0.6      --->      1       2         0.2 
    1        5         0.8      --->      1       3         0.2 
    2        1         1.0      --->      1       4         0.6
    2        2         1.2      --->      1       5         0.8

    2        3         1.4      --->      2       1         1.0
    2        4         1.6      --->      2       2         1.2
    2        5         1.8      --->      2       3         1.4
    3        1         2.0      --->      2       4         1.6
    3        2         2.2      --->      2       5         1.8
    3        3         2.4      xxx          
    3        4         2.6      xxx          
    3        5         2.8      xxx       
```

If the signal were 100 Hz, we can obviously select an exact sample
point (the 25th) in an EDF record to correspond to an offset of 0.25
seconds.  In the case of our hypothetical 5 Hz signal, as above, we
have to select the closest next signal, so there will be a slight
shift in the timings of signals.  If this matters, you can first
resample all signals to align to the new offset perfectly.

<h5>Parameters</h5>

Main arguments

| Parameter | Example | Description |
| ---- | ---- | ---- |
| `align` | `align=N1,N2,N3,R,W,?` | Annotations to align epochs by |
| `annot-out` | `annot-out=annots/^.annot` | Write (realigned) annotations | 

Other arguments as required by the [`WRITE` command](outputs.md#write)

| Parameter | Description |
| ---- |  ---- |
| `edf-tag` | New EDF tag |
| `edf-dir` | New EDF directory |


<h5>Output</h5>

The `ALIGN` command forces a `WRITE` of the EDF; also, annotation files can be output, with the
annotations correctly altered to specify onset relative to the new EDF start.

<h5>Example</h5>

_to be added_


## ANON

Sets the _in memory_ EDF header fields `Patient ID` and `Start Date` 
fields to missing (a `.` character).  If a new EDF is
generated with the `WRITE` command, it will have those fields blanked.
As with all Luna commands, this does not alter the original EDF.

!!! note 
    This command does not alter the ID specified in the
    [_sample-list_](../luna/args.md) (i.e. the first column).  That ID,
    which is used to track all output, etc, is distinct from the EDF
    header `Patient ID` field, and may or may not be similar.

<h3>Parameters</h3>

No parameters.

<h3>Output</h3>

No output other than message to the log, and altering the in-memory
representation of the EDF header.

<h3>Example</h3>

A typical EDF with identifying information in the header (showing only
relevant rows from the `SUMMARY` output):

```
luna my.edf -s "SUMMARY" | head 
```
```
EDF filename   : my.edf
Patient ID     : id00001
Recording info : 
Start date     : 07.06.16
Start time     : 23:07:56

... (cont'd) ...
```

Here we see how the `ANON` command effectively wipes this information:

```
luna my.edf -s "ANON & SUMMARY" | head 
```
```
EDF filename   : my.edf
Patient ID     : .
Recording info : 
Start date     : .
Start time     : 23:07:56
```

This next command takes all EDFs in a project (defined by `s.lst`) and
creates a set of new EDFs with the [`WRITE`](outputs.md#write) command
(in the folder `edfs/`, and with the new sample list `s2.lst`) that
are identical except they have the `Patient ID` and `Start Date` fields 
set to missing:

```
luna s.lst -s "ANON & WRITE edf-dir=edfs/ edf-tag=anon sample-list=s2.lst" 
```

## SIGGEN

_Generate, or add-in, artificial test signals_

This is a simple command to generate test signal data (on top of an existing EDF).  Currently, it
only generates sine wave signals.

<h5>Parameters</h5>

| Parameter | Example | Description |
| ---- | ----- | ----- |
| `sig` | `sig=C3,C4` | Signals to be modified |
| `sine` | `sine=10,20` | Generate a sine wave with specified frequency (10 Hz), amplitude (20 units) and optionally phase |
| `clear` | | If present, clear the signal before adding in this component |


<h5>Output</h5>

No new output, this command just modifies the internal signal data.

<h5>Example</h5>


To generate a sine wave in the signal `C3` (first clearing that signal):

```
luna s.lst -o out.db -s ' MASK ifnot=NREM2 & RE
                          SIGGEN sig=C3 clear sine=10,100 
                          MTM sig=C3 '
```

Plotting the output of `MTM`:

![img](../img/siggen.png)

