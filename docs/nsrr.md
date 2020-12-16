# Luna & the National Sleep Research Resource

Luna provides a number of convenience features to support working with
polysomnography data from the National Sleep Research Resource
([NSRR](http://sleepdata.org)), including pre-populated [_sample
lists_](luna/args.md#sample-lists), [_parameter
files_](luna/args.md#parameter-files), channel
[_aliases_](luna/args.md#aliases) and remappings for
[_annotations_](luna/args.md#interval-annotations).

## NSRR sample lists 

After downloading PSGs (via the [NSRR Ruby
Gem](https://github.com/nsrr/nsrr-gem/blob/master/README.md#prerequisites),
as [shown here](https://sleepdata.org/datasets/shhs/files) for SHHS,
for example), you can create
[_sample-lists_](luna/args.md#sample-lists) using Luna's
[`--build`](luna/args.md#build) command.


## Annotation aliases 

Slightly different annotation encodings are sometimes used between and
within different NSRR cohorts.  For example, in some studies a leg
movement may be encoded as `PLM (Left)` whereas in others it might be
`Limb Movement (Left)`.  Arousals, in particular, use an array of
trivially different terms (for example, varying in how they are
formatted or whether upper or lower case is used).  This can make
specifying annotations (e.g. for use in `MASKS`) unnecessarily
difficult.

To simplify NSRR annotations, Luna will, by default, automatically
convert NSRR annotations to a standard, easy-to-parse format
(i.e. with no special characters or spaces).  Luna also defines a
series of _variables_ that represent higher-level groups of
annotations, as tabulated below.  artifact.

To turn off Luna's automatic conversion of NSRR annotations, set
```
nsrr-remap=N
```
on the command line or parameter file.

Below we list the annotation remappings by broad class:
[_arousals_](#arousals), [_respiratory events_](#respiratory-events),
[_sleep staging_](#sleep-staging), [_PLMs_](#plms),
[_artifacts_](#artifacts) and [_miscellaneous_](#misc).  We then list
the pre-defined set of [_variables_](#higher-level-variables) that
group related annotation terms. 

### Arousals

| Remapped term | Original NSRR terms |
| ---- | ---- | 
| `arousal_standard` | `Arousal ()`<br>`Arousal|Arousal ()`<br>`Arousal|Arousal (Arousal)`<br>`Arousal|Arousal (Standard)`<br>`Arousal|Arousal (STANDARD)`<br>`ASDA arousal|Arousal (ADSA)`<br>`ASDA arousal|Arousal (asda)`<br>`ASDA arousal|Arousal (ASDA)`<br>`Arousal (Asda)`<br>`Arousal (ASDA)` | 
| `arousal_spontaneous` |  `Arousal (ARO SPONT)`<br>`Spontaneous arousal|Arousal (apon aro)`<br>`Spontaneous arousal|Arousal (ARO SPONT)`<br>`Spontaneous arousal|Arousal (spon aro)`<br>`Spontaneous arousal|Arousal (SPON ARO)` |
| `arousal_external` | `External arousal|Arousal (External Arousal)` |
| `arousal_respiratory` | `Arousal resulting from respiratory effort|Arousal (ARO RES)`<br>`Arousal resulting from respiratory effort|Arousal (RESP ARO)`<br>`RERA`<br>`Arousal (ARO RES)`<br>`Respiratory effort related arousal|RERA`
| `arousal_cheshire` | `Arousal resulting from Chin EMG|Arousal (Cheshire)`<br>`Arousal resulting from Chin EMG|Arousal (CHESHIRE)` |
| `arousal_plm` | `Arousal (ARO Limb)`<br>`Arousal resulting from periodic leg movement|Arousal (PLM)`<br>`Arousal resulting from periodic leg movement|Arousal (PLM ARO)` |

### Respiratory events

| Remapped term | Original NSRR terms |
| ---- | ---- | 
| `apnea_obstructive` | `Obstructive Apnea`<br>`Obstructive apnea|Obstructive Apnea` |
| `apnea_central` | `Central Apnea`<br>`Central apnea|Central Apnea` | 
| `apnea_mixed` | `Mixed Apnea`<br>`Mixed apnea|Mixed Apnea` |
| `hypopnea` | `Hypopnea`<br>`Hypopnea|Hypopnea` | 
| `periodic_breathing` | `Periodic Breathing`<br>`Periodic breathing|Periodic Breathing` |
| `respiratory_paradox` | `Respiratory Paradox` |
| `desat` | `SpO2 desaturation`<br>`SpO2 desaturation|SpO2 desaturation` |
| `unsure` | `Unsure`<br>`Unsure|Unsure` |

### Sleep staging

| Remapped term | Original NSRR terms |
| ---- | ---- | 
| `NREM1` | `Stage 1 sleep|1` |
| `NREM2` | `Stage 2 sleep|2` |
| `NREM3` | `Stage 3 sleep|3` |
| `NREM4` | `Stage 4 sleep|4` |
| `REM`   | `REM sleep|5` |
| `wake`  | `Wake`<br>`Wake|0` |
| `unscored` | `Unscored`<br>`Unscored|9` |
| `movement` | `Movement|6` |

### PLMs

| Remapped term | Original NSRR terms |
| ---- | ---- | 
| `plm_left` |  `Periodic leg movement - left|PLM (Left)`<br>`PLM (Left)`<br>`Limb Movement (Left)`<br>`Limb movement - left|Limb Movement (Left)` |
| `plm_right` | `Periodic leg movement - right|PLM (Right)`<br> `PLM (Right)`<br> `Limb Movement (Right)`<br> `Limb movement - right|Limb Movement (Right)` |


### Artifacts

| Remapped term | Original NSRR terms |
| ---- | ---- | 
| `artifact_respiratory` | `Respiratory artifact|Respiratory artifact`<br>`Respiratory artifact` |
| `artifact_proximal_pH` | `Proximal pH artifact` | 
| `artifact_distal_pH` | `Distal pH artifact` |
| `artifact_blood_pressure` | `Blood pressure artifact` |
| `artifact_TcCO2` | `TcCO2 artifact` |
| `artifact_SpO2` | `SpO2 artifact`<br>`SpO2 artifact|SpO2 artifact` |
| `artifact_EtCO2` | `EtCO2 artifact`<br>`EtCO2 artifact|EtCO2 artifact` |

### Misc

| Remapped term | Original NSRR terms |
| ---- | ---- | 
| `bradycardia` | `Bradycardia` |
| `tachycardia` | `Tachycardia` |
| `tachycardia_narrowcomplex` | `Narrow Complex Tachycardia`<br>`Narrow complex tachycardia|Narrow Complex Tachycardia` |
| `notes` | `Technician Notes` |


Therefore, one can subsequently write

```
MASK if=apnea_mixed
```

to exclude epochs with a mixed apnea event, for example.


### Higher-level variables

In addition to the remapping above, Luna provides a number of internal variables that group sets of NSRR annotations.  

| Variable definitions |
| ------- | 
| `${arousal} = arousal_standard,arousal_spontaneous,arousal_external,arousal_respiratory,` `arousal_plm,arousal_cheshire` |
| `${apnea} = apnea_obstructive,apnea_central,apnea_mixed,hypopnea` |
| `${artifact} = artifact_respiratory,artifact_proximal_pH,artifact_distal_pH,` `artifact_blood_pressure,artifact_TcCO2,artifact_SpO2,artifact_EtCO2` |
| `${arrhythmia} = bradycardia,tachycardia,tachycardia_narrowcomplex` |
| `${plm} = plm_left,plm_right` |
| `${n1} = NREM1` |
| `${n2} = NREM2` |
| `${n3} = NREM3,NREM4` |
| `${rem} = REM` |
| `${wake} = wake` |
| `${sleep} = NREM1,NREM2,NREM3,NREM4,REM` |

Note that these higher-level _variables_ are conceptually and
practically distinct from the _remappings_ of terms described in the
previous tables.

**Remappings**: these occur whilst the annotation file is being read
into Luna, meaning that different terms can be mapped to the same
annotation, resulting in a single annotation _class_.  For example, if
a project contained both `PLM (Left)` and `Limb Movement (Left)`
terms, it would be as if they were all replaced with a single term
`plm_left`, yielding a single annotation (of _class_ `plm_left`).

**Variables**: in contrast, the variable `${plm}` is just like any
other Luna variable, and so does not alter how annotations are read in
or stored: there are still two potential distinct annotations,
`plm_left` and `plm_right`.  Rather, it is just that writing `${plm}`
is the same as writing `plm_left,plm_right` which means _any_ PLM
(left or right leg).  (Note: in `MASK` statements, a comma-delimited
list `a,b` typically means `a` OR `b`.)

For example, to exclude epochs with a mixed apnea, one might write:

```
MASK if=apnea_mixed
```
which uses the remapped annotation term `apnea_mixed`.  If instead one wished to 
exclude epochs that contained _any_ type of apnea, one could use the higher-level variable `${apnea}` 
```
MASK if=${apnea}
```

Note that we use the `${x}` variable notation in this latter case, as
`${apnea}` is a variable rather than an annotation class name.  That is,
the above is identical to writing:

```
MASK if=apnea_obstructive,apnea_central,apnea_mixed,hypopnea
```

## Channel aliases

Data from NSRR studies were collected in different sites over long
periods of time; as such, there is not always a high degree of
consistency in the naming conventions for channels in the EDFs.  Over
a number of NSRR studies, we enumerated 328 distinct labels, for
example (none of these studies are hdEEG studies).

A number of redundancies are _trivial_ in that they are due to
capitalization issues (although note that many analysis packages
including Luna are case-sensitive).  Even within some cohorts
(e.g. SHHS2) we see multiple versions of the same label:

```
  “EEG2”  “EEG 2”   "EEG sec”  "EEG(sec)”    "EEG(SEC)"
```

We will generate a [_alias file_](luna/args.md#aliases) that you can
`@include` with any NSRR dataset you analyze.

_to be completed_




## Cohort parameter files

Here are some convenience [_parameter-files_](luna/args.md#parameter-files) for these NSRR cohorts:
[ _to be completed_ ]

| Cohort | _N_ EDFs records | Sample list | 
| ----   | --- | ---- | 
| CCSHS  |     | [`ccshs.txt`](http://zzz.bwh.harvard.edu/luna/dist/nsrr/parameter-files/ccshs.txt) |  
| CFS    |     | [`cfs.txt`](http://zzz.bwh.harvard.edu/luna/dist/nsrr/parameter-files/cfs.txt) |  
| CHAT (baseline)       |  | [`chat-baseline.txt`](http://zzz.bwh.harvard.edu/luna/dist/nsrr/parameter-files/chat-baseline.txt) |  
| CHAT (follow-up)      |  | [`chat-followup.txt`](http://zzz.bwh.harvard.edu/luna/dist/nsrr/parameter-files/chat-followup.txt) |  
| CHAT (non-randomized) |  | [`chat-nonrandomized.txt`](http://zzz.bwh.harvard.edu/luna/dist/nsrr/parameter-files/chat-nonrandomized.txt) |  
| MESA   |     | [`mesa.txt`](http://zzz.bwh.harvard.edu/luna/dist/nsrr/parameter-files/mesa.txt) |  
| MrOS   |     | [`mros.txt`](http://zzz.bwh.harvard.edu/luna/dist/nsrr/parameter-files/mros.txt) |  
| SHHS1  |     | [`shhs1.txt`](http://zzz.bwh.harvard.edu/luna/dist/nsrr/parameter-files/shhs1.txt) |  
| SHHS2  |     | [`shhs2.txt`](http://zzz.bwh.harvard.edu/luna/dist/nsrr/parameter-files/shhs2.txt) |  
| SOF    |     | [`sof.txt`](http://zzz.bwh.harvard.edu/luna/dist/nsrr/parameter-files/sof.txt) |  

See the hints in the section above for some notes on using a shell
alias to automatically point to the parameter file for a given project
(i.e. rather than having to retype it every time).
 
The following variables defined (with examples from CFS):

| Variable | CFS Example | Description |
| --- | --- | --- | 
| `${eeg}` | `eeg=C3,C4` | EEG channels | 
| `${emg}` | `emg=EMG` | Primary EMG channel | 
| `${eog}` | `eog=LOC,ROC` | Left and right EOG channels | 
| `${ecg}` | `ecg=ECG1` | Primary ECG channel |




