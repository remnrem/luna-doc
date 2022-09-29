# CANONICAL

_Generates canonical signals for harmonized EDFs, given a set of rules_

The `CANONICAL` command is designed to help when harmonizing multiple
sets of EDFs that have different labels and conventions.  We're
currently using this in the context of the [NSRR
harmonization](https://gitlab-scm.partners.org/zzz-public/nsrr/-/blob/master/common/harm-principles.md)
efforts (the linked document gives some motivating examples in actual multi-cohort contexts).


| Command | Description |
| -----  | ----- | 
| [Canonical definitions](#canonical-signal-definitions) | Syntax for defining canonical signal rules |
| [`CANONICAL`](#canonical) | Generate _canonical_ signals |


## Canonical signal definitions

The `CANONICAL` command is designed to help producing sets of EDFs
that have harmonized sets of channel labels and conventions. It sequentially
attempts to process a number of _rules_, that will create a new _canonical_ signal
based on existing signals, as long as certain _requirements_ (based on the EDF header fields)
are met.

The types of requirements may include:

 - channel labels
 - the presence of other reference channels
 - certain physical unit types 
 - certain transducer types
 - certain sample rates
 - certain minimum/maximum values

The `CANONICAL` command takes one or more _canonical signal definition
files_ and tries to make as many _canonical signals_ as it can, by
following the set of rules. Rules are followed sequentially, and once
a particular canonical signal has been generated, Luna will skip all
subsequent definitions.

It is also possible to allow for cohort/study-specific _exceptions_ to
a generic set of rules, by specifying a _group_ variable: e.g. if it
is known that a particular study uses an unusual naming convention,
this can be incorporated into the generic body of rules for that
particular study/cohort.

As part of the NSRR harmonization, we are generating a generic set of
harmonized rules, intended to work for most typical PSG studies.  Note
that the purpose of the CANONICAL command is not necessarily to map
every signal in an EDF: rather, it is to guarantee that any newly
generated canonical signals will follow a known set of conventions,
and thus can be more easily combined in cohort-level analyses.  These
will be distributed alongside Luna in future releases.

<h3>Basic file format</h3>

Following all Luna inputs, definition files must be plain-text (ASCII)
files, i.e. generated with a text editor. Other than comment lines
(starting with `%` or `#` characters), canonical signal files have two
main elements (a few exceptions to this are noted below):

 - __variable assignments__: starting with a `let` statement, these
   allow shortcuts and ensure consistency when specifying _rules_

 - __rules__: these define the canonical channels, and can include
   requirements as well as setting EDF header values (e.g. units,
   transducer type)


### Variables

Variables are shortcuts to make writing rules simpler. Any line that
starts with the `let` statement in this form is a variable definition:
e.g.

```
let left_mastoid=M1,A1,M1-Ref,A1-Ref
```

This means that if `left_mastoid` is used in any subsequent rule, it will be expanded into `M1,A1,M1-Ref,A2-Ref`, i.e.
as if that text had been typed in full instead of `left_mastoid`. 

Importantly, note that if we repeat the `let` assignment with the same
variable name, new values are __appended__ (i.e. they do not overwrite
the values). For example:

```
let left_mastoid=M1,M1-Ref
let left_mastoid=A1,A1-Ref
```
implies `left_mastoid` equals `M1,M1-Ref,A1,A1-Ref`. This can make it easier
to format and edit canonical definition files.

Some common variables may be used to list typically encountered unit types, e.g:

```
let volt  = V,volt
let mvolt = mV,millivolt,milli-volt,mvolt,m-volt
let uvolt = uV,microvolt,micro-volt,uvolt,u-volt
```

Then, when specifying rules, one can just write `uvolt` instead of
`uV,microvolt,micro-volt,uvolt,u-volt`

!!! hint "Canonical signal file versus standard variables"
    Note that
    _variables_ in the context of a canonical signal definition file
    are distinct from Luna's normal usage of
    [variables](../luna/args.md#variables). Standard variables
    (e.g. defined on the Luna command line) cannot be used in
    canonical definition files.  Also, unlike standard Luna variables,
    which are invoked with the `${var}` format, variables in canonical
    files are simply written without extra syntax, i.e. `var`.


### Rules

Rules are the meat of canonical signal definitions.  A rule follows a
specific syntax that defines certain _requirements_ for a canonical
signal, and optionally, may also _set_ some values for the newly
generated signal (primarily unit and or sample rate).


Rules follow a strict _space-indentation_ format.  Lines that start with:

 - _no indentation_ : name of a canonical channel, i.e. a _target
   channel_ we are trying to create; this also indicates the start of
   a new _rule_

 - _one space_ : a sub-section of that rule that is always one of
   these keywords: `unless:` `group:` `req:` or `set:`

 - _two spaces_ : the specific features that define the sub-section

As a toy example, the following constitutes one _rule_ to make the dummy channel `canonical_label`:

```
canonical_label
 req:
  sig = sig1,sig2
  ref = ref1,ref2
  trans = type1,T1
  trans = unknown,.
  unit = uV,micro-volts,uvolt
  unit = mV,milli-volts,mvolt
 set:
  unit = uV
  sr = 100
```

This rule creates a new signal called `canonical_label`, _if the following conditions are met_:

 - the EDF has a channel `sig1` or `sig2` (if it has both, the first listed will be used)
 - the EDF also has a channel `ref1` or `ref2` (again, if both are present, the first will be used)
 - the primary signal selected (either `sig1` or `sig2`) has an EDF transducer field that is `type1` or `T1` (first line)
 or it is `unknown` or an empty field (the `.`) (second line)
 - the EDF physical unit is one of `uV`, `micro-volts`, etc, or (on the second line) `mV`, etc
 
If these conditions are not met for each term (`sig`, `ref`, `unit`
and `trans`), the rule will not be enacted.  If these conditions are
met, then when running the `CANONICAL` command, Luna will generate a
new channel with the following properties:

 - the new label is `canonical_label`
 - the primary signal (either `sig1` or `sig2`) will be _re-referenced_ by either `ref1` or `ref2`
 - the new signal's transducer field will be set to _either_ `type1` or `unknown` (as these are the __first__ entries
 for each `trans:` line above)
 - likewise, the new signal's unit field will initially be set to _either_ `uV` or `mV` (as these are the first entries, i.e. the preferred values) for that set
 - because the rule also has a `set:` section, two additional operations will be performed:
  - first, if the unit is `mV`, then the channel will be rescaled to be in `uV` and then EDF header will be updated to reflect that
  - second, if the original sample rate (i.e. for `sig1` or `sig2`) is not 100 Hz, then the channel will be resampled to be 100 Hz

That is, (as well as the re-referencing step), the two `set:`
operations will actually change the numeric values of the channel data
themselves; all other options (i.e. for transducer fields and units)
are simply changing the text fields in the EDF headers, and do not
touch the actual signal data.


As above, the two main sections are `req:` and `set:`, to specify
the _requirements_ for the signal to be generated, and then any
special things that must be _set_ in the new signal.  These start with
__a single space character__ as noted above, and refer to the
canonical signal previously defined above (i.e. the last line that
started without any space indentation):

 - `group:` if this is a group-specific rule, list the groups here
 - `unless:` a list of other canonical signals - this rule with be ignored if this signal has already been made
 - `req:` any _requirements_ for the signal (see below)
 - `set:` options to set the sample rate and/or units of the new canonical signal

As noted, requirements (`req`) must be met for a rule to be enacted;
the following requirements are valid: (each starting with exactly two spaces):

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

For `unit` and `trans` only, if a line has more than one value listed,
then the first _of that particular_ line, is the _preferred_ value,
i.e. the EDF field for any generated canonical signal will be set to
that string value.  For example, the line as part of a `req:`
```
  unit = uV,mV,micro-volts,milli-volts
```
would be incorrect, as any of these four values would be changed to `uV`.  In contrast, this
would appropriately set the field to either `uV` or `mV` depending on the matched values:
```
  unit = uV,micro-volts
  unit = mV,milli-volts
```
(Remember that specifying _preferred_ unit labels as the first in a list of requirements does not
actually change/rescale the data themselves, unlike the `set:` sub-section, which allows the one special
case of converting between volts, milli-volts and micro-volts - i.e. actually changing the signal, not just
the text label in the EDF header.)

In the case of voltage units, using the _variables_ we defined above:
```
let volt  = V,volt
let mvolt = mV,millivolt,milli-volt,mvolt,m-volt
let uvolt = uV,microvolt,micro-volt,uvolt,u-volt
```
one would just write
```
  unit = uvolt
  unit = mvolt
  unit = volt
```
as part of a _requirement_.  Because each line is a different unit (i.e. not just different labels for the same thing), they
need to be on different lines.  Note that this still leads to the same _preferred_ units (`uV`, `mV`, `V`).  


Here is an example rule for an EEG target channel: `C4_M1`, i.e. a
right central electrode with contralateral mastoid reference.  Using the unit
variables defined above, we might write: 

```
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
  sig = C4,EEG_C4,C4_REF,EEG_C4_REF
  ref = A1,M1,EEG_A1,EEG_M1,
  ref = A1_REF,M1_REF,EEG_A1_REF,EEG_M1_REF
  unit = uvolt
  unit = mvolt
  unit = volt
 set:
  unit = uV
  sr = 128
```

Here we have two rules for the same channel.  The `CANONICAL` command
would first try to make the top one, which assumes a suitable
referenced channel already exists (but allowing for some variations on
the naming, e.g. `A1` instead of `M1` etc).   Thus there is no `ref` requirement.

If the EDF did not contain such a channel, but did contain _two_ channels matching one of the `sig` and `ref`
lists (e.g. `C4_REF` and `M1_REF`) then it would try to make the `C4_M1` channel, explicitly re-referencing `C4_REF`
against `M1_REF`).  In this example, the `sig` channel would also be required to be of some flavor of voltage units.

The `set:` sub-section in the last command would then change the
sample rate of 128 Hz if it was not already that; also, if the channel
(based on `sig` only) was `mV` or `V`, it would rescale that variable
to be in `uV` units (and update the EDF header field appropriately for
the new channel).


### Special behaviours

The canonical signal file syntax allows for a few special behaviours.

#### Missing values/wildcards

For the `unit` and `trans` requirement fields only:

 - a period character (`.`) means to match an empty field (null, all spaces, or already a period character).
 - an asterisk (`*`) means to match any non-empty field

In this way, on can provide defaults for missing or otherwise unrecognized values.   For example:
```
 req:
  unit = degrees
  unit = missing,.
  unit = unknown,*  
```

As the order matters, Luna would first match on the term `degrees`
(or, if that is a variable with a comma-delimited list, on any of the
aliases for that concept); failing that, if the unit field was empty,
it would be set to the text string `missing`; otherwise, it will be
set to `unknown` - i.e. the last line with the wild-card acts as a catch-all.

Note that if this is the only form for a `unit` or `trans` requirement:
```
  trans = EMG,.,*
```
it effectively sets the field to that first value (`EMG`) no matter what the original value is,
either missing or non-missing.

Note that the missing field `.` cannot ever be the _first_ item listed
for a `unit` or `trans` requirement - i.e. the preferred name must
always be non-null.

If `*` is the preferred term, this means keep it as is.  Thus, by itself, 
```
  unit = *
```
is equivalent to not specifying any `unit` requirement - i.e. everything matches, and nothing is changed.  However, this
could be useful in the context of having specified some prior requirement: e.g.
```
  unit = unknown,?,.
  unit = *
```
This basically says: make `?` or empty fields as `unknown`, otherwise leave the unit as is.    Without the last line, this rule
would _require_ that the unit if either empty, `?` or `unknown`. 

#### Average referencing

For the `ref` field, if comma-delimited values are in quotes, this
implies an _average reference_ of those signals. For example `"A1,A2"`
this indicates a linked mastoid reference, which in turn implies that
both `A1` and `A2` must exist, if that rule is to be enacted.


#### Sanitization of labels

Luna will _sanitize_ all labels coming in, meaning that spaces and
most other non-alphanumeric characters are translated to an underscore
(`_`).  When writing rules, it is therefore best to write the rule with that in mind:
e.g. if the original value is `EEG F3-M2`, this will be sanitized to `EEG_F3_M2`.  Therefore,
write the rule with this latter label:
```
  sig = EEG_F3_M2
```
When writing out new unit/transducer fields, these will also be sanitized as needed.

Note that Luna will match in a case-insenstive manner: therefore you can just write 
```
  sig = airflow
```
instead of something like:
```
  sig = airflow,AIRFLOW,Airflow,AirFlow
```

### Unless sections

An `unless:` section in a rule means that it will not be processed if
another canonical label has already been created.  For example, if a
channel may be a thermistor or nasal pressure cannula, this would
first try to make those specific channels, e.g. where
`thermistor_names` is a variable list of labels known to match to
thermistor channels (in practice, the first two rules would specify
other requirements, e.g. based on units), etc.  As both thermistor and
cannuala are measures of airflow generally, you'd only want to make
the last channel if it was not possible to make either of the first
(which are more specific).  That is, here `airflow` acts as a catch
all to describe _some type of airflow channel_ but where the precise
form is not known.

```
therm
 req:
  sig = thermistor_names

nasal_pres
 req:
  sig = nasal_pres_names

airflow
 unless:
  therm nasal_pres
 req:
  sig = airflow_names
```

Another example: you may only wish to make linked-mastoid referenced EEG channels if the matched
contralateral-mastoid referenced EEG could not be made: (again, in practice, these rules would likely
contain other features, e.g. units, sample rates etc)
```
C4_M1
 req:
  sig = C4
  ref = M1

C4_LM
 unless:
  C4_M1
 req:
  sig = C4
  ref = LM
  ref = "M1,M2"
```

### Group sections

As noted below, the `CANONICAL` command can optionally called with one or more `group` options set:

```
luna s.lst -s CANONICAL file=defs.txt group=study1,study2
```

When processing the rules, if groups have been set, then Luna will
watch out for group-specific rules.  If a rule has a `group:`
attached, then it will only be processed if one of those groups (here
`study1`) was specified when running `CANONICAL`.  Group labels can be
any _sanitized_ text label (i.e avoid whitespace, etc)


```
C4_M1
 group:
  study1
  study3
 req:
  sig = EEG2
 ... etc ...

CZ_LM
 group:
  study2
 req:
  sig = EEG2
 ... etc ...

```

This can be useful when constructing _generic_ signal definition files
- i.e. rules that are expected to match most studies, i.e. where the
labels are more or less self-explanatory.  In this context,
intrinsically ambiguous or confusing labels should not be included in
any generic set.  Here, they can be partitioned out into separate
rules/files, and only applied when running `CANONICAL` for those
studies/groups.  In the above example, say you have datasets where
`EEG2` generically means `C4-M1` in some studies, but `CZ-(M1+M2)/2`
in another.  Especially as the label `EEG2` is ambiguous without
further context, these should be group-specific rules.  In the above
example, `EEG2` will be mapped to `CZ_LM` if run with the
`group=study2` option, for example.

A more general case may be where you wish to combine group-specific and generic rules.  As Luna
processes the files/rules in the order they are specified, you'd want to add group-specific rules
first, when running for a particular group.  For example, in the general case:
```
luna s.lst -s CANONICAL file=generic-defs.txt
```
whereas if we also have group-specific rules in a separate file `study2-defs.txt`, then
you would invoke the command as follows:
```
luna s.lst -s CANONICAL file=study2-defs.txt,generic-defs.txt group=study2
```

As a concrete (if contrived) example, you may know that this study (`study2`)
mislabelled channel `C3` as `F3`:   so in `study2-defs.txt` you may have 

```
C3_M2
 group:
  study2
 req:
  sig = F3
  ref = M2
```
whereas in the generic file (`generic-defs.txt`) there is the rule in the form:
```
C3_M2
 req:
  sig =	C3
  ref =	M2
```

Here, Luna would correctly make `C3_M2` (using the mislabelled
channel) in this special case.  As it processes the group-specific
rule first (based on the order of the `file=` option), when it comes
to the second generic rule, it would already have been satisifed.

In this context, it is often useful to tell Luna to stop trying to make a given
channel - for example, if you know that only group-specific rules apply, and never
want a generic rule to be attempted.    For example, we'd want to avoid generic rules
involving `F3` from being run for this particular study (i.e. if we knew this was a mislablled channel).

This can be achieved with the special keyword `closed:` at the start of a line, followed by one or more
canoical labels, i.e. placed in a leading group-specific definition file:

```
closed: F3_M2 F3_LM
```

This means that subsequently Luna would not attempt to make `F3_M2` or
`F3_LM` (for example), even if were otherwise able to do so.

In the general case, Luna will attempt to satisfy generic rules if
group-specific rules failed, if when a `group` has been specified on
the command line.


### Simple aliasing

A common form of rule that does not involve other requirements may be in the form:
```
channel1
 sig:
  req = channel1,ch1,c1,ch-n1,ch_A
```

i.e. this is doing nothing more than selecting the first of
`channel1`, `ch1`, `c1`, `ch-n1` and `ch_A` and making a new channel
called `channel1`.

You can make a shortcut for this by using the special `<-` operator when
writing the canonical label:

```
channel1 <- channel1 ch1 c1 ch-n1 ch_A
```
i.e. this is identical to writing the more verbose form above.

Note that it is possible that the canonical channel already exists, as
in this case. Here it simply means "keep as is".  

### Multiple/interchangeable canonicals channels

This is an advanced feature. Sometimes one EDF may contain more than
one type of canonical signal.  It is possible to make different
canonical signals and then relabel them as follows, with the `<<-`
operator:

```
canon <<- canon1 canon2 canon3
```
where we expect `canon1`, `canon2` and `canon3` to have been defined by previous rules.  In
this case, Luna will relabel the channels as `canon`.  If more than one or these exists in
a given EDF, then would be relabelled following Luna's standard approach to making channel labels
unique (i.e. adding `.1`, `.2` etc):  i.e. `canon`, `canon.1` and `canon.2`. 

!!! hint "Canonical signals and channel _types_"
    _Canonical signals_
    are not to be confused with [channel
    types](#../luna/args.md#channel-types).  A canonical signal
    represents a _new_ signal that is generated from existing signals
    according to a set of rules.  A _channel type_ is simply a label
    or annotation that is given to all channels, based on their
    channel name.  The reason for the `CANONICAL` command is that
    often datasets (such as those in the NSRR) can EDFs with mixtures
    of conventions and montages.  Thus, this command is designed to be
    able to more simply, e.g., "extract a central EEG" across multiple studies.
   

## CANONICAL

_Apply a set of rules from one or more canonical signal definition files_

See the logic of canonical signal definitions and rules above.  To process one
ore more definition files on a set of PSGs, use the `CANONICAL` command as below.


<h3>Parameters</h3>

| Parameter | Example | Description |
| --- | --- | --- |
| `file`    | `rules.txt,extra.txt` | One or more files that define canonical signals |
| `group`   | `SHHS` | Optionally, specify a _group_ to use |
| `verbose` |  | (optional) verbose output in the console log |
| `drop-originals` | | (optional) original channels are dropped from the EDF on completion | 
| `inc` | `C3_M2,C4_M1` | Only attempt to make these canonincal signals |
| `exc` | `C3_M2,C4_M1` | Do not attempt to make these canonincal signals |
| `prefix` | `/path/to/files/` | Add this folder prefix to all `file` values |
| `prefiltering` | | Retain pre-filtering field information; otherwise this EDF header is wiped |

Note that files are processed in the order they are listed; within a
file, rules are processed in the order they are listed.


<h3>Output</h3>

The primary action of this command is to add new signals to the in-memory EDF, as defined in
the canonical file definitions.

Baseline output (strata: _none_)

| Variable | Description |
| --- | --- |
| `CS_NOT`  | Number of canonical signals not defined |
| `CS_SET`  | Number of canonical signals defined |
| `UNUSED_CH` | Number of EDF channels not used in one or canonical signal |
| `USED_CH` | Number of EDF channels used in one or canonical signal |

EDF channel-specific information (stratum: `CH`)

| Variable | Description |
| --- | --- |
| `USED` | 0/1 for whether this EDF channel was used
| `DROPPED` | 0/1 for whether this EDF channel was dropped |

Canonical signal information (stratum: `CS`)

| Variable | Description |
| --- | --- |
| `DEFINED` | 0/1 for whether this canonical channel was defined |
| `SIG` | The primary EDF signal used to define this canonical channel | 

<h3>Example</h3>

```
luna s.lst -o out.db -s CANONICAL file=~/nsrr/common/resources/canonical/harm.txt drop-originals
```

```
 CMD #1: CANONICAL
   options: file=~/nsrr/common/resources/canonical/harm.txt sig=*
  read 81 rules from /Users/smp37/nsrr/common/resources/canonical/harm.txt
  in total, read 81 rules and 93 variables

  26 signals from EDF
  + generating canonical signal C3_M2 from existing signal(s) C3 / M2
  + generating canonical signal C4_M1 from existing signal(s) C4 / M1
  + generating canonical signal C3_LM from existing signal(s) C3 / M1,M2
  + generating canonical signal C4_LM from existing signal(s) C4 / M1,M2
  + generating canonical signal LOC from existing signal(s) LOC / M2
  + generating canonical signal ROC from existing signal(s) ROC / M2
  + generating canonical signal EMG from existing signal(s) EMG2 / EMG1
  + generating canonical signal ECG from existing signal(s) ECG2 / ECG1
  + generating canonical signal abdomen from existing signal(s) ABDO_EFFORT
  + generating canonical signal thorax from existing signal(s) THOR_EFFORT
  + generating canonical signal sum from existing signal(s) SUM
  + generating canonical signal nas_pres from existing signal(s) NASAL_PRES
  + generating canonical signal airflow from existing signal(s) AIRFLOW
  + generating canonical signal SpO2 from existing signal(s) SPO2
  + generating canonical signal HR from existing signal(s) HRATE
  + generating canonical signal pleth from existing signal(s) PLETHWV
  + generating canonical signal pulse from existing signal(s) PULSE
  + generating canonical signal LAT from existing signal(s) L_LEG
  + generating canonical signal RAT from existing signal(s) R_LEG
  + generating canonical signal pos from existing signal(s) POSITION
  + generating canonical signal snore from existing signal(s) SNORE
```

We see that all but 3 EDF channels were harmonized:
```
destrat out.db +CANONICAL
```
```
ID       CS_NOT   CS_SET  UNUSED_CH  USED_CH
id0001   20       21      3          23 
```

```
ID        CH          DROPPED  USED
id0001    C3          1        1
id0001    C4          1        1
id0001    M1          1        1
id0001    M2          1        1
id0001    LOC         0        1
id0001    ROC         0        1
id0001    ECG2        1        1
id0001    ECG1        1        1
id0001    EMG1        1        1
id0001    EMG2        1        1
id0001    EMG3        1        0
id0001    L_Leg       1        1
id0001    R_Leg       1        1
id0001    AIRFLOW     0        1
id0001    THOR_EFFORT 1        1
id0001    ABDO_EFFORT 1        1
id0001    SNORE       0        1
id0001    SUM         0        1
id0001    POSITION    1        1
id0001    OX_STATUS   1        0
id0001    PULSE       0        1
id0001    SpO2        0        1
id0001    NASAL_PRES  1        1
id0001    PlethWV     1        1
id0001    Light       1        0
id0001    HRate       1        1
```
