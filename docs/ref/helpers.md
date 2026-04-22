# Helpers

_Miscellaneous functions designed to help with Luna workflows_

This page covers utility commands that support project-level workflows rather than signal analysis. `--build` automatically generates a Luna sample-list by traversing a directory tree. `--validate` checks that all EDFs and annotation files in a project are readable. `--repath` updates file paths in an existing sample-list. `--merge` concatenates multiple EDFs end-to-end; `--bind` combines EDFs by adding channels from one into another. `--xml` and `--xml2` parse and display NSRR XML annotation files. [`OTSU`](helpers.md#otsu) and `--otsu` apply Otsu's thresholding method to EDF channels or external data files, respectively. [`REPORT`](helpers.md#report) controls output visibility and can force additional variables into text-table output, and [`REQUIRES`](helpers.md#requires) lets scripts enforce a minimum Luna version.

|Command | Description | 
|---|---|
| [`--build`](#-build)  | Generate a sample list automatically |
| [`--validate`](#-validate) | Validate all EDFs and annotation files in a project | 
| [`--repath`](#-repath) | Change root filepaths in a sample list |
| [`--merge`](#-merge)  | Merge (concatenate) multiple EDFs |
| [`--bind`](#-bind) | Merge (column/channel bind) multiple EDFs |
| [`--xml`](#-xml) | View NSRR XML annotation files |
| [`--xml2`](#-xml2) | View NSRR XML annotation files (verbose) |
| [`--otsu`](#-otsu) | Calculate thresholds based on Otsu's method (external data) |
| [`OTSU`](#otsu) | Calculate thresholds based on Otsu's method (internal channel) |
| [`REPORT`](#report) | Control output visibility and forced reporting |
| [`REQUIRES`](#requires) | Require a minimum Luna version |

## --build

_Automatically compile a sample list_

See [here](../luna/args.md#-build-option) for details.

<h3>Methods</h3>

`--build` traverses the specified directory (and optionally its subdirectories) looking for EDF files, then attempts to pair each EDF with annotation files sharing the same base name. Matching is performed by stripping extensions and comparing base filenames; the `-nsrr` flag activates NSRR-specific filename conventions for pairing. The resulting sample list (with one row per EDF, containing the subject ID, EDF path, and any matched annotation paths) is written to standard output.

<h3>Parameters</h3>

| Parameter | Example | Description |
| --- | --- | --- |
| `-nsrr` |  | Assume NSRR file names |
| `-ext` | | Include this extension |
| `-nospan` | | Do not span folders when name-matching |

<h3>Output</h3>

A sample list written to the standard output stream.



## --validate

_Validate all EDFs and annotation files in a project_

This command checks that files can be opened correctly - e.g. spotting
EDFs with corrupt headers or other issues.  See the [Luna walk
through](https://zzz.nyspi.org/luna-walkthrough/p1/valid) for
some examples of using `--validate`.

If invalid files are found, Luna writes a message to the console
indicating the nature of the problem.  Per-individual flags are sent
to the standard output mechanism to indicate whether the EDF and any
annotation files were valid also.

<h3>Methods</h3>

`--validate` iterates over each entry in the sample list and attempts to open the EDF by reading and parsing its header; it then checks that all signals listed in the header are readable and have consistent record counts. Each annotation file listed for that individual is similarly opened and parsed. Any file that cannot be opened or parsed correctly generates a console warning with the nature of the failure; otherwise, per-individual pass/fail flags are written to the output database.

<h3>Parameters</h3>

| Parameter | Example | Description |
| --- | --- | --- |
| `slist` | `s.lst` | Sample list | 

<h3>Output</h3>

Primary output (strata: _none_)

| Variable | Description |
| --- | --- |
| `EDF` | 0/1 flag for whether EDF was valid (1=valid) |
| [`ANNOTS`](annotations.md#annots) | 0/1 flag for whether annotation file(s) were valid (1=all valid) |

<h3>Example</h3>

Testing the tutorial data:

```
luna --validate -o out.db --options slist=s.lst
```
```
  validating files in sample list s.lst

  all good, no problems detected in 3 observations scanned
```

Getting the individual output:
```
destrat out.db +VALIDATE 
```
```
ID	ANNOTS	EDF
nsrr01	1	1
nsrr02	1	1
nsrr03	1	1
```


## --repath

_Swap out file paths in a sample list_

Reads a sample list from standard input, and (ignoring the first ID
column) changes the starting path.  This can be useful if you have
moved the EDFs, and so need a new path.  Alternatively, if you are
working with files mounted via a network drive, or from a Docker
container, then the paths may be different from the local paths.

This command accepts `.` as the first argument, meaning always append the
second argument if (and only if) the sample list has a relative path.

<h3>Methods</h3>

`--repath` reads a sample list line by line from standard input. For each line, the first (ID) column is passed through unchanged. For all subsequent columns (file paths), `--repath` replaces the first occurrence of the specified old path prefix with the new path prefix. If the old path is `.`, the new path is prepended to any relative (non-absolute) path. The modified sample list is written to standard output.

<h3>Parameters</h3>

| Parameter | Example | Description |
| --- | --- | --- |
| _first arg_ | `/old/path/` | Part of sample list to be replaced ( or `.`) |
| _second arg_ | `/new/path/` | Replacement |

<h3>Output</h3>

A new sample list written to the standard output stream.

<h3>Example</h3>

If the working folder has changed from `Users/js7/data` to
`home/jsmith/cfs/`, for example.  The original sample list (4 files
from CFS):

```
cat s.lst
```
```
cfs-800002	/Users/js7/data/cfs-800002.edf	/Users/js7/data/cfs-800002.xml
cfs-800010	/Users/js7/data/cfs-800010.edf	/Users/js7/data/cfs-800010.xml
cfs-800011	/Users/js7/data/cfs-800011.edf	/Users/js7/data/cfs-800011.xml
cfs-800017	/Users/js7/data/cfs-800017.edf	/Users/js7/data/cfs-800017.xml
```

Using repath to fix the sample list:

```
luna --repath /Users/js7/data /home/jsmith/cfs < s.lst > s2.lst
```
```
cat s2.lst
```
```
cfs-800002	/home/jsmith/cfs/cfs-800002.edf	/home/jsmith/cfs/cfs-800002.xml
cfs-800010	/home/jsmith/cfs/cfs-800010.edf	/home/jsmith/cfs/cfs-800010.xml
cfs-800011	/home/jsmith/cfs/cfs-800011.edf	/home/jsmith/cfs/cfs-800011.xml
cfs-800017	/home/jsmith/cfs/cfs-800017.edf	/home/jsmith/cfs/cfs-800017.xml
```



## --merge

_Create a single EDF from multiple partial EDFs_

Some systems may export EDFs in small segments (e.g. each 1 hour or 5
minutes in duration).  The `--merge` command can take these and create
a single EDF for analysis.
    
This command only provides a simple way to concatenate multiple EDFs
that must have identical header structures (in terms of the number of
channels, their labels and samples rates). Segments cannot be
overlapping, but otherwise they can be of different sizes, and there
can also be gaps between segments (in which case, `--merge` will
generate an EDF+D).

!!!hint "Horizontal merges"
    The `--merge` command combines EDFs with the same channel that span different time periods.  In contrast,
    the [`--bind` command](#-bind) combines EDFs with different channels but that span the same time period.

<h3>Methods</h3>

`--merge` reads each input EDF in order, verifying that all share identical channel counts, labels, and sample rates. The segments are concatenated chronologically; if consecutive segments are contiguous in time (within a small tolerance), they are joined seamlessly into a standard continuous EDF. If gaps exist between segments, the output is written as an EDF+D (discontinuous EDF+), with EDF+ Annotations channel records encoding the start time of each segment. The merged EDF header uses the start date and time of the first input segment, and the total record count spans from the start of the first to the end of the last segment.

<h3>Parameters</h3>

| Parameter | Example | Description |
| ---- | ---- | ---- |
| `id` | `id001` | ID to be specified in the resulting EDF's header |
| `edf` | `merged.edf` | Filename for the resulting EDF |
| `sample-list` | `s.lst` | Write the resulting ID/EDF pair to a sample list |
| `*` | `f1.edf f2.edf ...` | Two or more EDFs to be merged |

<h3>Output</h3>

No output other than message to the log and writing a new, merged EDF
(and optionally, writing to a sample list).

<h3>Example</h3>

```
echo "id=id01 edf=id01.edf sample-list=s.lst data/*.edf" | luna --merge
```

This command will read all the EDFs in the folder `data/` and attempt
to concatenate them, and save a new EDF called `id01.edf`.  The
resulting EDF will have the start time set to the earliest start time/date
observed in the whole set.  EDFs will be concatenated (as a single,
continuous EDF) in the order in which they are specified on the
command line.  Therefore, be careful if using wildcards as per the
above example (i.e. if ordered files are listed as `block1.edf`,
`block10.edf`, `block11.edf`, ..., `block2.edf`).  In this example,
use, e.g. `block01.edf`, `block02.edf`, etc to ensure correct sorting.

## --bind

_Horizontally merge EDFs_

This special command takes a list of EDFs and merges them:

 - all EDFs must have compatible headers in terms of the start times, duration and number of records

 - we assume the EDFs each contain one or more unique channels that span the same period

 - channels can have different sample rates

<h3>Methods</h3>

`--bind` reads each input EDF and verifies that all share compatible headers: identical recording start times, identical total durations, and identical record counts (though channels may differ in label and sample rate). It then constructs a new EDF whose signal data is the column-wise union of all input channels, preserving each channel's original sample rate. Channels from each input EDF are appended in order; if a channel label already exists in the merged EDF, a numeric suffix is appended to ensure uniqueness. The output is written as a single EDF (or EDF+) to the specified filename.

<h3>Parameters</h3>

The `--bind` command takes a series of EDF files. If an argument
has an `=` sign in it, Luna will check for the three special values:

| Parameter | Example | Description |
| ---- | ---- | ---- |
| `id` | `id=id001` | ID to be specified in the resulting EDF's header |
| `edf` | `edf=id001.edf` | Filename for the resulting EDF (instead of `merged.edf`) |
| `sample-list` | `sample-list=s.lst` | Append the ID/EDF names to `s.lst` |

For example (the order of arguments doesn't matter):
```
luna --bind id=id001 edf=id001.edf f1.edf f2.edf f3.edf
```

<h3>Outputs</h3>

A single EDF will be written.

<h3>Example</h3>

Some systems such as the ZMax emit multiple EDFs, each one containing
a single channel.  To create a single EDF:

```
luna --bind "EEG L.edf" "EEG R.edf" LIGHT.edf BATT.edf \
            "BODY TEMP.edf"  "NASAL L.edf" "NASAL R.edf" \
	    NOISE.edf OXY_DARK_AC.edf  OXY_DARK_DC.edf OXY_IR_AC.edf \
	    OXY_IR_DC.edf OXY_R_AC.edf OXY_R_DC.edf RSSI.edf \
	    dX.edf dY.edf dZ.edf
```

The console output will show the following:
```
  in total, attached 18 EDFs
  writing bound data:
     ID           : merged1
     EDF filename : merged.edf

  good, all EDFs have bind-compatible headers
  expecting 18 signals (each of 25200 records of 1 sec) in the new EDF
  adding timeline; adding 25200 empty records...

  compiling channels from EDF #1: EEG_L
  compiling channels from EDF #2: EEG_R

  ...
  
  writing merged EDF as merged.edf
  data are not truly discontinuous
  writing as a standard EDF
  writing 18 channels
  saved new EDF, merged.edf
```

We can confirm the new EDF has all 18 channels:

```
luna merged.edf -s DESC
```
```
EDF filename      : merged.edf
ID                : merged
Clock time        : 09.53.51 - 16.53.51
Duration          : 07:00:00  25200 sec
# signals         : 18
Signals           : EEG_L[256] EEG_R[256] LIGHT[256] BATT[256] BODY_TEMP[256]
                    NASAL_L[256] NASAL_R[256] NOISE[256] OXY_DARK_AC[256]
		    OXY_DARK_DC[256] OXY_IR_AC[256] OXY_IR_DC[256] OXY_R_AC[256]
		    OXY_R_DC[256] RSSI[256] dX[256] dY[256] dZ[256]
```


 

## --xml

_Dump XML annotation files_

Also see `--xml2`

`--xml` is a lightweight command-line helper to inspect a single sleep-annotation
XML file without loading an EDF. Internally, Luna parses the XML and prints a
compact, chronologically ordered summary of the events it finds.

The implementation auto-detects standard NSRR-style XML versus Profusion-style
XML. For each scored event it prints start/stop time, duration, event type (if
present), concept/label, and notes (if present). For Profusion XML, Luna also
expands the `SleepStages` block into epoch-wise stage rows using the XML
`EpochLength`.

<h3>Methods</h3>

`--xml` parses a single XML annotation file using Luna's internal XML reader (TinyXML-based). The file format is auto-detected as either NSRR-style or Profusion-style based on the root element structure. For NSRR-style files, each `ScoredEvent` node is extracted and printed in chronological order with start time, duration, event type, concept, and notes fields. For Profusion-style files, the `SleepStages` element is additionally decomposed into per-epoch stage labels using the recorded epoch length. Output is written directly to standard output as a plain-text listing.

<h3>Parameters</h3>

| Parameter | Example | Description |
| --- | --- | --- |
| `{xml}` | `study.xml` | A single XML file to inspect |

!!! note
    This helper is invoked directly from the command line, e.g. `luna --xml file.xml`.
    It does not require an EDF, sample list, or output database.

<h3>Output</h3>

No Luna output tables are created. Output is printed directly to standard
output as a compact text listing.

Typical rows have the form:

```text
start - stop    (duration secs)    EventType    EventConcept    Notes
```

If the XML contains an `EpochLength` field, Luna also prints that near the
start of the listing. For Profusion XML, sleep stages are rendered as
`SleepStage` rows.

<h3>Example</h3>

```bash
luna --xml shhs1-200001-nsrr.xml
```


## --xml2

_Dump any XML file_

Also see `--xml`

`--xml2` is the verbose counterpart to `--xml`. Instead of extracting a compact
event list, it dumps the parsed XML tree itself so that element names,
attributes, and nested text nodes can be inspected directly.

This is mainly useful when working with unfamiliar XML variants, debugging field
names, or checking how Luna is seeing a given file before writing import logic
or annotation mappings.

<h3>Methods</h3>

`--xml2` parses the specified XML file using Luna's internal TinyXML-based reader and then performs a recursive depth-first traversal of the document tree, printing each node (document, element, or text node) with its type, name, attribute key-value pairs, and text content. No semantic interpretation of sleep events is applied; the output represents the raw parsed tree structure of the XML file.

<h3>Parameters</h3>

| Parameter | Example | Description |
| --- | --- | --- |
| `{xml}` | `study.xml` | A single XML file to dump verbosely |

<h3>Output</h3>

No Luna output tables are created. Output is written directly to standard
output as a recursive tree dump from TinyXML, showing:

- document / element / text node structure
- element names
- attributes and attribute values
- text payloads

The output is intentionally low-level and mirrors the parsed XML structure more
than the semantic sleep-event content.

<h3>Example</h3>

```bash
luna --xml2 shhs1-200001-nsrr.xml
```


## --otsu

_Derive Otsu optimal binary threshold for values from an external file_

`--otsu` is the command-line helper form of Otsu thresholding. It reads a single
numeric vector from standard input, evaluates candidate thresholds across the
observed value range, and reports the cut-point that maximizes between-class
variance under Otsu's method.

This helper does **not** operate on EDF channels. Use [`OTSU`](#otsu) for the
in-EDF version.

Internally, Luna reads whitespace-delimited numeric values from `stdin`, so the
input can be one number per line or any whitespace-separated stream of numbers.

<h3>Methods</h3>

`--otsu` reads a stream of whitespace-delimited numeric values from standard input and applies Otsu's method to identify an optimal binary threshold. The observed range of values is divided into `k` equally spaced candidate cut-points. For each candidate threshold, the between-class variance is computed as the product of the two class weights (proportions of observations below and above the threshold) and the squared difference between the two class means. The threshold that maximizes between-class variance — equivalent to minimizing within-class variance — is selected as the Otsu threshold. The selected threshold and its empirical percentile within the input distribution are reported.

<h3>Parameters</h3>

| Parameter | Example | Description |
| --- | --- | --- |
| `k` | `k=100` | Number of candidate threshold bins / cut-points to evaluate |

!!! note
    The current implementation reads values from standard input. Although older
    helper conventions sometimes mention a sample list, `--otsu` does not use an
    EDF or sample list in this code path.

<h3>Output</h3>

Top-level helper summary:

| Variable | Description |
| --- | --- |
| `EMPTH` | Estimated Otsu threshold |
| `EMPF` | Empirical percentile at the selected threshold |

Threshold-scan output (strata: `TH`)

| Variable | Description |
| --- | --- |
| `SIGMAB` | Between-class variance at that candidate threshold |
| `F` | Empirical percentile at that candidate threshold |

The helper also prints a short console summary reporting the selected threshold
and percentile.

<h3>Example</h3>

```bash
cat values.txt | luna --otsu k=200
```

Or equivalently:

```bash
awk '{print $3}' data.txt | luna --otsu
```


## OTSU

_Derive Otsu optimal binary threshold for values from an internal EDF channel_

[`OTSU`](helpers.md#otsu) applies the same Otsu threshold scan directly to one or more EDF signals.
For each requested channel, Luna extracts the whole-trace values, evaluates
candidate thresholds, and reports the cut-point that maximizes between-class
variance.

This is useful when you want a data-driven binary cut-point for an existing
signal channel without first exporting the values.

<h3>Methods</h3>

[`OTSU`](helpers.md#otsu) applies Otsu's method to the full-trace sample values of each specified EDF signal. The observed value range is divided into `k` equally spaced candidate cut-points, and for each candidate the between-class variance is computed as the product of the two class weights (proportions of samples below and above the threshold) and the squared difference between the two class means. The threshold maximizing between-class variance is selected. This threshold and its empirical percentile within the signal's distribution, together with the full threshold-scan profile, are reported in the output.

<h3>Parameters</h3>

| Parameter | Example | Description |
| --- | --- | --- |
| `sig` | `sig=C3,C4` | One or more signals on which to estimate Otsu thresholds |
| `k` | `k=100` | Number of candidate threshold bins / cut-points to evaluate |
| `verbose` | `verbose` | Accepted by the implementation; current output is driven by the standard tables |

<h3>Output</h3>

Channel-level summary (strata: `CH`)

| Variable | Description |
| --- | --- |
| `EMPTH` | Estimated Otsu threshold for this channel |
| `EMPF` | Empirical percentile at the estimated threshold |

Threshold scan by channel (strata: `CH × TH`)

| Variable | Description |
| --- | --- |
| `SIGMAB` | Between-class variance at that candidate threshold |
| `F` | Empirical percentile at that candidate threshold |

<h3>Example</h3>

```bash
luna s.lst -o out.db -s 'OTSU sig=EMG k=200'
```

To inspect the selected threshold per channel:

```bash
destrat out.db +OTSU -r CH
```

To inspect the full threshold scan:

```bash
destrat out.db +OTSU -r CH TH
```

## REPORT

_Control output visibility and forced reporting_

[`REPORT`](helpers.md#report) is a utility command for controlling what Luna emits to its output
streams. It can hide or show whole commands, individual tables, or selected
variables, and it can also ensure that additional variables are emitted in
text-table output even if they were not part of the original command metadata.
It also supports marking text-table output as compressed.

<h3>Methods</h3>

[`REPORT`](helpers.md#report) modifies Luna's internal command-definition registry for the current
run. In `hide`/`show` mode, it suppresses or restores output at the command
level, table level, or variable level depending on whether `cmd`, `fac`, and
`vars` are specified. `hide-all` and `show-all` globally suppress or restore
all output definitions. In `ensure` mode, [`REPORT`](helpers.md#report) creates or augments a table
definition for a given command/table combination so that named variables will be
logged in text-table output even if that output was not already declared in the
standard metadata. In `compress` mode, it marks text-table output for a command
or table as compressed.

<h3>Parameters</h3>

| Parameter | Example | Description |
| --- | --- | --- |
| `cmd` | `cmd=PSD` | Command whose output definition to modify |
| `fac` | `fac=CH,E` | Optional table / factor specification within that command |
| `vars` | `vars=PSD,RELPSD` | Optional comma-delimited list of variables within that table |
| `hide` |  | Hide the specified command, table, or variables |
| `show` |  | Show the specified command, table, or variables |
| `hide-all` |  | Hide all output definitions |
| `show-all` |  | Show all output definitions |
| `ensure` |  | Ensure the specified variables are defined for output |
| `compress` |  | Mark text-table output as compressed |

Notes:

- `cmd` is required unless you are using `hide-all` or `show-all` alone.
- If `vars` is given, `fac` must also be given.
- `ensure` cannot be combined with `hide`, `show`, `hide-all`, `show-all`, or `compress`.
- `compress` cannot be combined with `ensure`.
- `fac=BL` or `fac=.` can be used to refer to the baseline table.

<h3>Output</h3>

No formal tabular output. [`REPORT`](helpers.md#report) changes Luna's internal output-definition
state for the current run and may emit notes to the console log.

<h3>Examples</h3>

Hide an entire command's output:

```bash
luna s.lst -s 'REPORT hide cmd=PSD'
```

Hide selected variables from one table:

```bash
luna s.lst -s 'REPORT hide cmd=PSD fac=CH vars=PSD,RELPSD'
```

Ensure additional variables are logged for text-table output:

```bash
luna s.lst -s 'REPORT ensure cmd=MYCMD fac=CH vars=VAR1,VAR2'
```

## REQUIRES

_Require a minimum Luna version_

[`REQUIRES`](helpers.md#requires) is a guard command for scripts and pipelines. It stops processing if
the running Luna binary is older than the version you specify, allowing a
workflow to fail early with a clear message rather than run under an
incompatible build.

<h3>Methods</h3>

[`REQUIRES`](helpers.md#requires) compares the running Luna version against the requested minimum
version string. If the installed version is sufficiently recent, processing
continues normally. Otherwise, Luna halts before later commands are run.

<h3>Parameters</h3>

| Parameter | Example | Description |
| --- | --- | --- |
| `version` | `version=1.3.4` | Minimum required Luna version |

<h3>Output</h3>

No formal tabular output. The command either allows processing to continue or
halts execution with an explanatory error.

<h3>Example</h3>

```bash
luna s.lst -s 'REQUIRES version=1.5.1 & PSD sig=C3'
```
