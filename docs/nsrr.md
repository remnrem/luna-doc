# Luna & the National Sleep Research Resource

Luna reads polysomnography recordings and annotation files distributed by the
[National Sleep Research Resource (NSRR)](https://sleepdata.org). This page
explains how to build a sample list, enable optional NSRR annotation remappings,
and select events using the resulting labels. See the
[annotation reference](ref/annotations.md#nsrr-xml-files) for the NSRR XML format.

## NSRR sample lists

After downloading EDFs and their annotation files, create a
[sample list](luna/args.md#sample-lists) with [`--build`](ref/helpers.md#-build):

```bash
luna --build /path/to/nsrr-data -nsrr > s.lst
```

The `-nsrr` option restricts annotation matching to the `-nsrr.xml` filename
suffix, matching, for example, `subject01.edf` with `subject01-nsrr.xml`.
The default builder already recognizes this suffix alongside other supported
annotation extensions; `-nsrr` is useful when you want only the NSRR XML
files attached. It controls filename matching, not annotation remapping or
harmonization. Enable event remapping separately with `nsrr-remap=T`.

Some newer NSRR cohorts include original data that have not yet been
harmonized to the usual NSRR formats and naming conventions. Their annotation
files may use different names or formats, which `-nsrr` does not automatically
handle. Check the cohort's files, then use the appropriate builder options or
prepare the sample list manually. Unsupported annotation formats need to be
converted to a format Luna can read; changing the sample list alone does not
convert them.

Review the generated list to check that each EDF has the intended annotation
files attached. See the builder reference for naming rules and additional
options. Each row contains an ID, an EDF path, and optional annotation paths.

## Annotation aliases

Annotation names vary between NSRR cohorts and scoring systems. Luna provides
two sets of built-in annotation remappings:

- **Default mappings** normalize common sleep-stage labels, such as
  `Stage 2 sleep|2` to `N2`. These are enabled automatically.
- **Optional NSRR mappings** normalize additional event labels, such as
  `Mixed Apnea` to `apnea:mixed`. These are **off by default**; enable them
  with `nsrr-remap=T`.

For example, load the first recording with NSRR event mappings enabled and
inspect its annotations:

```bash
luna s.lst 1 nsrr-remap=T -s 'ANNOTS'
```

Remapping occurs while annotations are loaded. Subsequent commands must use
the resulting labels. The mappings do not change the original files on disk.
The same label sanitization and
[space-handling settings](luna/args.md#spaces-in-channel-and-annotation-names)
apply to registered aliases and incoming annotation labels. Unrecognized
nonempty labels are retained after this normalization.
The tables below list every alias-to-target definition in `annot/nsrr-remap.cpp`,
grouped by purpose and separated into default and optional mappings.
Inspect your recording's actual labels with [`ANNOTS`](ref/annotations.md#annots)
before writing selection rules.

To use only the default mappings, omit `nsrr-remap=T`. Setting
`annot-remap=F` clears all annotation remappings, including default sleep-stage
mappings and any previously added NSRR mappings.
`annot-remap=F nsrr-remap=T` clears the defaults and then adds NSRR mappings;
`nsrr-remap=T annot-remap=F` clears both sets. `nsrr-remap=F` does not undo
mappings already enabled earlier in the options. See
[remapping annotations](luna/args.md#remapping-annotations) for custom mappings.

### Sleep staging

These mappings are defined in the default annotation setup and do not require
`nsrr-remap=T`. Legacy stage 4 is combined with stage 3 as `N3`.

| Configured target | Defined input aliases |
| --- | --- |
| `N1` | `NREM1`<br>`NREM1 sleep`<br>`N1 sleep`<br>`Stage 1 sleep|1`<br>`Sleep stage N1`<br>`Sleep stage 1`<br>`Stage N1`<br>`StageN1`<br>`Stage 1`<br>`Stage1`<br>`Stage NREM1`<br>`StageNREM1`<br>`S1` |
| `N2` | `NREM2`<br>`NREM2 sleep`<br>`N2 sleep`<br>`Stage 2 sleep|2`<br>`Sleep stage N2`<br>`Sleep stage 2`<br>`Stage N2`<br>`StageN2`<br>`Stage 2`<br>`Stage2`<br>`Stage NREM2`<br>`StageNREM2`<br>`S2` |
| `N3` | `NREM3`<br>`NREM3 sleep`<br>`N3 sleep`<br>`Stage 3 sleep|3`<br>`Sleep stage N3`<br>`Sleep stage 3`<br>`Stage N3`<br>`StageN3`<br>`Stage 3`<br>`Stage3`<br>`Stage NREM3`<br>`StageNREM3`<br>`S3`<br>`N4`<br>`NREM4`<br>`NREM4 sleep`<br>`N4 sleep`<br>`Stage 4 sleep|4`<br>`Sleep stage N4`<br>`Sleep stage 4`<br>`Stage N4`<br>`StageN4`<br>`Stage 4`<br>`Stage4`<br>`Stage NREM4`<br>`S4` |
| `R` | `REM`<br>`REM sleep`<br>`REM sleep|5`<br>`REM|5`<br>`Sleep stage REM`<br>`Sleep stage R`<br>`Stage R`<br>`StageR`<br>`Stage REM`<br>`StageREM` |
| `W` | `Wake`<br>`Wake|0`<br>`Sleep stage Wake`<br>`Sleep stage W`<br>`Stage W`<br>`StageW`<br>`Stage Wake`<br>`Wake stage` |
| `NR` | `Sleep stage N`<br>`Sleep stage NR`<br>`Sleep stage NREM`<br>`Stage N`<br>`Stage NR`<br>`Stage NREM`<br>`StageN`<br>`StageNR`<br>`StageNREM`<br>`NREM`<br>`NREM sleep`<br>`NR sleep` |
| `U` | `Unscored`<br>`Unscored|9` |
| `?` | `Unknown`<br>`Sleep stage ?`<br>`Stage ?`<br>`missing` |
| `M` | `Movement`<br>`Movement|6` |

`NR` denotes unspecified NREM sleep; `U` denotes unscored intervals, `?`
unknown stages, and `M` movement-time annotations. These are distinct from
the five standard stage labels `N1`, `N2`, `N3`, `R`, and `W`.

Additional cohort-specific stage aliases require `nsrr-remap=T`:

| Configured target | Defined input aliases |
| --- | --- |
| `W` | `SRO:Wake`<br>`SDO:WakeState` |
| `N1` | `SRO:Stage1Sleep`<br>`SDO:NonRapidEyeMovementSleep-N1` |
| `N2` | `SRO:Stage2Sleep`<br>`SDO:NonRapidEyeMovementSleep-N2` |
| `N3` | `SRO:Stage3Sleep`<br>`SRO:Stage4Sleep`<br>`SRO:Stage34Sleep`<br>`SDO:NonRapidEyeMovementSleep-N3`<br>`SDO:NonRapidEyeMovementSleep-N4` |
| `R` | `SRO:RapidEyeMovement`<br>`SDO:RapidEyeMovementSleep` |

### Lights and gaps

These mappings are also part of the default setup:

| Configured target | Defined input aliases |
| --- | --- |
| `L` | `Lights`<br>`lights`<br>`Light` |
| `lights_on` | `Lights On`<br>`LightsOn` |
| `lights_off` | `Lights Off`<br>`LightsOff` |
| `G` | `GAP`<br>`-` |

### Arousals

The following event mappings require `nsrr-remap=T`. All aliases defined
in the source are listed.

| Configured target | Defined input aliases |
| --- | --- |
| `arousal` | `Arousal ()`<br>`Arousal|Arousal ()`<br>`Arousal|Arousal`<br>`Arousal|Arousal (Standard)`<br>`Arousal_(STANDARD)`<br>`Arousal|Arousal_(Arousal)`<br>`ASDA arousal|Arousal (ADSA)`<br>`ASDA arousal|Arousal (ASDA)`<br>`Arousal (ASDA)`<br>`Arousal_(Asda)`<br>`EEG arousal` |
| `arousal:spontaneous` | `Arousal (ARO SPONT)`<br>`Spontaneous arousal|Arousal (apon aro)`<br>`Spontaneous arousal|Arousal (ARO SPONT)`<br>`Spontaneous arousal|Arousal (SPON ARO)` |
| `arousal:respiratory` | `Arousal resulting from respiratory effort|Arousal (ARO RES)`<br>`RERA`<br>`Arousal (ARO RES)`<br>`Arousal resulting from respiratory effort|Arousal (RESP ARO)`<br>`Respiratory effort related arousal|RERA` |
| `arousal:external` | `External arousal|Arousal (External Arousal)`<br>`Arousal_(External_Arousal)` |
| `arousal:cheshire` | `Arousal resulting from Chin EMG|Arousal (Cheshire)`<br>`Arousal_(CHESHIRE)` |
| `arousal:lm` | `arousal_lm`<br>`lml_arousal`<br>`lmr_arousal`<br>`lmb_arousal`<br>`Arousal_(ARO_Limb)` |
| `arousal:plm` | `arousal_plm`<br>`Arousal_resulting_from_periodic_leg_movement|Arousal_(PLM)`<br>`Arousal_resulting_from_periodic_leg_movement|Arousal_(PLM_ARO)` |

### Respiratory events

| Configured target | Defined input aliases |
| --- | --- |
| `apnea` | `Apnea` |
| `apnea:obstructive` | `Obstructive apnea|Obstructive Apnea`<br>`Obstructive Apnea`<br>`apnea_obstructive`<br>`Obstructive_apnea|APNEA-OBSTRUCTIVE` |
| `apnea:central` | `Central Apnea`<br>`apnea_central`<br>`Central apnea|Central Apnea`<br>`Central_apnea|APNEA-CENTRAL` |
| `apnea:mixed` | `Mixed Apnea`<br>`apnea_mixed`<br>`Mixed apnea|Mixed Apnea`<br>`Mixed apnea|APNEA-MIXED` |
| `hypopnea` | `Hypopnea|Hypopnea` |
| `hypopnea:obstructive` | `hypopnea_obstructive`<br>`Obstructive_Hypopnea` |
| `hypopnea:central` | `hypopnea_central` |
| `periodic_breathing` | `Periodic Breathing`<br>`Periodic breathing|Periodic Breathing` |
| `respiratory_paradox` | `Respiratory Paradox` |
| `snoring` | `Snoring` |
| `cheynestokes_breathing` | `cheynestokes_breathing` |
| `desat` | `SpO2 desaturation`<br>`SpO2 desaturation|SpO2 desaturation`<br>`SpO2 desaturation|DESAT` |

### PLMs

Periodic leg movements (`PLM`) and other limb movements (`LM`) have
separate targets, including left/right variants.

| Configured target | Defined input aliases |
| --- | --- |
| `PLM` | `Periodic leg movement`<br>`Periodic leg movement|PLM` |
| `PLM:left` | `Periodic leg movement - left|PLM (Left)`<br>`PLM (Left)` |
| `PLM:right` | `Periodic leg movement - right|PLM (Right)`<br>`PLM (Right)` |
| `LM` | `Limb Movement`<br>`Limb movement|Limb Movement` |
| `LM:left` | `Limb Movement (Left)`<br>`Limb movement - left|Limb Movement (Left)` |
| `LM:right` | `Limb Movement (Right)`<br>`Limb movement - right|Limb Movement (Right)` |

### Artifacts

| Configured target | Defined input aliases |
| --- | --- |
| `artifact` | `Signal artifact|SIGNAL-ARTIFACT` |
| `artifact:respiratory` | `Respiratory artifact`<br>`Respiratory artifact|Respiratory artifact` |
| `artifact:proximal_pH` | `Proximal pH artifact`<br>`Proximal_pH_artifact|Proximal_pH_artifact` |
| `artifact:distal_pH` | `Distal pH artifact` |
| `artifact:pH` | `Proximal_pH|Distal_pH_artifact` |
| `artifact:blood_pressure` | `Blood pressure artifact`<br>`Blood_pressure_artifact|Blood_pressure_artifact` |
| `artifact:TcCO2` | `TcCO2 artifact`<br>`TcCO2 artifact|TcCO2 artifact` |
| `artifact:SpO2` | `SpO2 artifact`<br>`SpO2 artifact|SpO2 artifact` |
| `artifact:EtCO2` | `EtCO2 artifact`<br>`EtCO2 artifact|EtCO2 artifact` |
| `artifact:body_temperature` | `Body_temperature_artifact|Body_temperature_artifact` |

### Misc

| Configured target | Defined input aliases |
| --- | --- |
| `unsure` | `Unsure|Unsure|Unsure` |
| `movement` | `Movement` |
| `position:left` | `Body position change to left|POSITION-LEFT` |
| `position:right` | `Body position change to right|POSITION-RIGHT` |
| `position:prone` | `Body position change to prone|POSITION-PRONE` |
| `position:supine` | `Body position change to supine|POSITION-SUPINE` |
| `position:upright` | `Body position change to upright|POSITION-UPRIGHT` |
| `arrhythmia:bradycardia` | `Bradycardia`<br>`Bradycardia|Bradycardia` |
| `arrhythmia:tachycardia` | `Tachycardia`<br>`Tachycardia|Tachycardia` |
| `arrhythmia:narrow_complex_tachycardia` | `Narrow Complex Tachycardia`<br>`Narrow complex tachycardia|Narrow Complex Tachycardia` |
| `notes` | `Technician Notes` |

With the optional mappings enabled, plain `Movement` maps to the event label
`movement`, overriding its default mapping to `M`. The default mapping of
`Movement|6` to `M` remains available.

For NSRR XML input, labels such as `apnea:mixed` are annotation class names
and can be used directly in commands. When loading the six-column `.annot`
format, the default `class-instance-delimiter=:` can instead split such a
label into class `apnea` and instance ID `mixed`. Check the loaded labels
when changing annotation formats; masks can also match
[instance IDs](ref/masks.md#matching-instance-ids).

### Selecting events

With NSRR event mappings enabled, exclude epochs overlapping a mixed apnea:

```bash
luna s.lst 1 nsrr-remap=T -s 'EPOCH & MASK if=apnea:mixed'
```

To exclude epochs overlapping any apnea or hypopnea class, use trailing
wildcards:

```bash
luna s.lst 1 nsrr-remap=T -s 'EPOCH & MASK if=apnea*,hypopnea*'
```

This matches `apnea` and its subtype labels, plus `hypopnea` and its subtypes.
A comma-separated annotation list uses OR logic: an epoch is masked if it
overlaps any matching class. A mask marks epochs for subsequent analyses;
it does not remove the annotation events themselves. See
[`MASK`](ref/masks.md#mask) for other selection rules.

### Higher-level variables

Luna defines `${sleep}` as `N1,N2,N3,R` during default annotation setup.
For example, to retain only epochs overlapping scored sleep:

```bash
luna s.lst 1 -s 'EPOCH & MASK ifnot=${sleep}'
```

Other event groups can be defined with ordinary
[Luna variables](luna/args.md#variables). For example, define a study-specific
respiratory-event group and use it in a mask:

```bash
luna s.lst 1 nsrr-remap=T resp_events='apnea*,hypopnea*' -s 'EPOCH & MASK if=${resp_events}'
```

Quote the variable value so that the shell passes the `*` characters to Luna
instead of treating them as filename patterns. The separate single quotes
around the command keep the shell from expanding `${resp_events}` before
Luna reads it. Defining a variable groups labels for substitution;
it does not rename or merge annotation classes. Earlier documentation listed
predefined `${apnea}`, `${arousal}`, `${artifact}`, `${arrhythmia}`, `${plm}`,
and individual-stage variables; these are not populated by the current
NSRR remapping code. Define any such groups explicitly for your analysis.
