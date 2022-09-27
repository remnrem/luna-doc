# CANONICAL

_Generates canonical signals for harmonized EDFs, given a set of rules_

The `CANONICAL` command is designed to help when harmonizing multiple
sets of EDFs that have different labels and conventions.  We're
currently using this in the context of the [NSRR
harmonization](https://gitlab-scm.partners.org/zzz-public/nsrr/-/blob/master/common/harm-principles.md)
efforts.  That document also gives examples that motivates the need
for the `CANONICAL` command in multi-cohort contexts.


| Command | Description |
| -----  | ----- | 
| [Canonical definitions](#canonical-signal-definitions) | Syntax for defining canonical signal rules |
| [`CANONICAL`](#canonical) | Generate _canonical_ signals |


## Canonical signal definitions

Many algorithms expect particular types of signals (e.g.  for
automated sleep staging, EEG, EOG and/or EMG ) as well as certain
conventions to hold, with respect to channel names, units, sampling
rates and/or referencing schemes.  Nonetheless, there may often be a
degree of exchangeability between similar inputs: for example,
a `C3 - M2` channel may be acceptable instead of `C4 - M1`. 

The `CANONICAL` command is designed to faciliate this type of
preprocessing, to generate datasets that adhere to a _canonical_ set
of conventions, and across studies, will therefore be uniform for
subsequent processing.

These _conventions_ are defined in one or more plain-text files.
Other than comment lines (starting with `%` or `#` characters),
canonical signal files have two elements:

 - __variable assignments__: starting with a `let` statement, these allow shortcuts and ensure consistency when specifying _rules_

- __rules__: these define the canonical channels, and can include requirements as well as setting EDF header values (e.g. units, transducer type)


<h5>Variables</h5>

Variables are merely shortcuts to make writing rules simpler. They start with the `let` statement: e.g.
```
let left_mastoid=M1,A1,M1-Ref,A1-Ref
```

This means that if `left_mastoid` is used in any rule, it will be expanded into `M1,A1,M1-Ref,A2-Ref`, i.e.
as if that full statement was typed instead of `left_mastoid`. 

Note that if we repeat the `let` assignment with the same variable name, new values are __appended__
(i.e. they do not overwrite the values). For example: 

```
let left_mastoid=M1,M1-Ref
let left_mastoid=A1,A1-Ref
```
implies `left_mastoid` equals `M1,M1-Ref,A1,A1-Ref`.

!!! hint "Canonical signal file versus standard variables"
    Note that _variables_ in the context of a canonical signal definition file are completely distinct from Luna's normal
    usage of [variables](../luna/args.md#variables). Standard variables (e.g. defined on the Luna command line) cannot be used
    in canonical definition files.   Also, unlike standard Luna variables, which are invoked with the `${var}` format, variables in
    canonical files are simply written without extra syntax, i.e. `var`. 


<h5>Rules</h5>

Rules are the meat of canonical signal definitions.  They define the
canonical channels, and can include requirements as well as setting
EDF header values (e.g. units, transducer type).  Rules use
a strict _space-indentation_ to structure each rule.  Lines that start with are:

 - _no indentation_ : name a canonical channel, i.e. that a _target channel_ we are trying to create; this also indicates the start of a new _rule_
 - _one space_ : a sub-section of that rule that is always one of these keywords: `unless:` `group:` `req:` or `set:`
 - _two spaces_ : the specific features that define the rule

As a toy example, the following constitutes one _rule_ to make the dummy channel `canonical_label`:
```
canonical_label

 unless:
  label2

 group: 
  group1
  group2
 
 req:
  sig = sig1,sig2
  ref = ref1,ref2
  trans = thermistor,therm
  sr-min = 10
  sr-max = 100
  unit = uV,micro-volts
  unit = mV,milli-volts
  scale = AC  

 set:
  unit = uV
  sr = 100
```

There are four sections (not all of which are needed) to a rule. As noted, each
each should start with a __single space character__ and must be written after a
canonical label line (i.e. they relate to that label/channel).

 - `group:` if this is a group-specific rule, list the groups here
 - `unless:` a list of other canonical signals - this rule with be ignored if this channel exists already
 - `req:` any _requirements_ for the signal (see below)
 - `set:` options to set the sample rate and/or units of the new canonical signal

All arguments for each of these four sections must start on subsequent lines that start with __with exactly two spaces__.

All requirements (`req`) must be met for a rule to be enacted; the following requirements
are valid terms:

 - `sig:` one or more _primary signals_ (required)
 - `ref:` optionally, any reference signals 
 - `trans:` optionally, a transducer field 
 - `unit:` optionally, required units
 - `scale:`  optionally, range of the signal (EDF header min/max)
 - `sr-min:` optionally, a min. required sample rate (of the primary signal)
 - `sr-max:` optionally, a max. allowed sample rate (of the primary signal)

The `sig`, `ref`, `trans` and `unit` fields can accept comma-delimited
lists and can appear on multiple lines (i.e. will append new values,
as for variable definitions above).


In practice, rules are likely to be shorter than the example above:
for example, here are two rules for the same target channel `C4_M1`,
that also illustrate the use of _variables_:

```
let volt  = V,volt
let mvolt = mV,millivolt,milli-volt,mvolt,m-volt
let uvolt = uV,microvolt,micro-volt,uvolt,u-volt

C4_M1
 req:
  sig = C4_M1,C4_A1,EEG_C4_M1,EEG_C4_A1
  unit = uvolt
  unit = mvolt
  unit = volt
 set:
  unit = uV
  sr = 128

C4_M1
 req:
  sig = C4,EEG_C4
  ref = A1,M1,EEG_A1,EEG_M1
  unit = uvolt
  unit = mvolt
  unit = volt
 set:
  unit = uV
  sr = 128
```

Note that in the above, `uvolt`, `mvolt` and `volt` are _variables_
(i.e. previously defined with a `let` statement) and so are expanded
into the full text: i.e. writing:

```
  unit = uvolt
```
is as if one had instead written:
```
  unit = uV,microvolt,micro-volt,uvolt,u-volt
```

<h5>Special behaviours</h5>

The canonical signal file syntax allows for a few special behaviours:

 - `ref` requirements: if comma-delimited values are in quotes, this implies an average reference of those signals. For example 
       `"A1,A2"` this indicates a linked mastoid reference, which in turn implies that both `A1` and `A2` must exist, if that rule is to be enacted.
       
 - `unit` and `trans` requirements: for both of these, the first element on each line will be the _preferred_ label, over any subsequent values on the same line, i.e. following the
 same type of symntax for [signal aliases](../luna/args.md#aliases).  That is, given:
 ```
   unit: V,volt
   unit: uV,micro-volt
 ```
 The above will set the unit of the new channel to `V` if it is either `V` or `volt` in the
 original; it will set it to `uV` if it is either `uV` or `micro-volt`.  If
 the unit doesn't match either of these requirements, the rule will
 not be enacted, if these are the only two `unit` requirements.  (If there is no `unit` requirement, then
 the physical dimension/unit of the EDF will be ignored.)

 - `unit` and `trans` requirements: a `.` character denotes an _empty field_ in the EDF (either null, or all spaces).  This provides a means of matching empty fields: i.e.,
 to supply a default label, say of `unknown` instead, one might write:
 ```
   trans: unknown,.
 ```
 which would match any empty field and set it to `unknown` when
 creating the new canonical channel. A period cannot be the first
 entry in the list of values however (i.e. `.` cannot be the _
 preferred_ unit label.

 - `unit` and `trans` requirements: a `*` character means a wildcard that matches anything.  This can be used to effectively set the transducer field 
 ```
  trans:
   thermistor,therm
  trans: unknown,.
   trans: invalid,*
 
(nb. this is
an exception to using the 'set:' part of the rule



%   i.e. this group of requirements would set anything that isn't
%        'thermistor', 'therm' or an empty field to 'invalid'.  Note
%        that by including a * wild card, this effectively stops the
%        trans field from being a requirement per se (i.e. it will
%        always match).  However, it is done this way (rather than
%        having a simple 'set:' trans field, to allow more flexibility
%        (i.e. some pre-existing terms can be handled separately
%        within a single rule, as in the above example)


Note that all arguments use case-insensitive matching (i.e. 'volt', 'Volt' and 'VOLT' are all identical)

% Note: typically, we can assume that all spaces are converted to
%    underscore (_) as per Luna default rule for reading channel
%    labels

% If a group has been specified on the Luna command line, if we
% encouter a rule with that group specified, subsequent generic rules
% (i.e. those w/out a group section) will not be enacted (i.e. even
% if the prior group-specific rules were not enacted either)




------------



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


## CANONICAL

_Apply a set of rules from one or more canonical signal definition files_

This command is set up with a typical PSG in mind, where a single EEG
or single ECG is often all that is required, e.g. for a given staging
or HRV analysis, etc.  In principle, this command can be extended to
define more canonical types (e.g. a _canonical frontal EEG_, or
_canonical oxygen desaturation signal_, etc), which we may do in
future releases.

<h3>Parameters</h3>

| Parameter | Example | Description |
| --- | --- | --- |
| `file`    | `file=rules.txt,extra.txt` | One or more files that define canonical signals |
| `group`   | `SHHS` | Optionally, specify a _group_ to use |
| `verbose` |  | (optional) verbose output in the console log |
| `drop-originals` | | (optional) original channels are dropped from the EDF on completion | 

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

