# Helpers

_Miscellaneous functions designed to help with Luna workflows_

|Command | Description | 
|---|---|
| [`--build`](#-build)  | Generate a sample list automatically |
| [`--validate`](#-validate) | Validate all EDFs and annotation files in a project | 
| [`--repath`](#-repath) | Change root filepaths in a sample list |
| [`--merge`](#-merge)  | Merge (concatenate) multiple EDFs |
| [`--bind`](#-bind) | Merge (column/channel bind) multiple EDFs |
| [`--xml`](#-xml) | View NSRR XML annotation files |
| [`--xml2`](#-xml2) | View NSRR XML annotation files (verbose) |
| [`--otsu](-otsu) | Calculate thresholds based on Otsu's method (external data) |
| [`OTSU`](otsu) | Calculate thresholds based on Otsu's method (internal channel) |

## --build

_Automatically compile a sample list_

See [here](../luna/args.md#-build-option) for details.

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
through](https://zzz.bwh.harvard.edu/luna-walkthrough/p1/valid) for
some examples of using `--validate`.

If invalid files are found, Luna writes a message to the console
indicating the nature of the problem.  Per-individual flags are sent
to the standard output mechanism to indicate whether the EDF and any
annotation files were valid also.

<h3>Parameters</h3>

| Parameter | Example | Description |
| --- | --- | --- |
| `slist` | `s.lst` | Sample list | 

<h3>Output</h3>

Primary output (strata: _none_)

| Variable | Description |
| --- | --- |
| `EDF` | 0/1 flag for whether EDF was valid (1=valid) |
| `ANNOTS` | 0/1 flag for whether annotation file(s) were valid (1=all valid) |

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

<h3>Parameters</h3>

| Parameter | Example | Description |
| --- | --- | --- |
| _first arg_ | `/old/path/` | Part of sample list to be replaced ( or `.`) |
| _second arg_ | `/new/path/` | Replacement |

<h3>Output</h3>

A new sample list written to the standard output stream.

<h3>Example</h3>

If the working folder have changed from `Users/js7/data` to
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

_to be added_

<h3>Parameters</h3>

_to be added_

<h3>Output</h3>

_to be added_

<h3>Example</h3>

_to be added_


## --xml2

_Dump any XML file_

Also see `--xml`

_to be added_

<h3>Parameters</h3>

_to be added_

<h3>Output</h3>

_to be added_

<h3>Example</h3>

_to be added_


## --otsu

_Derive Otsu optimal binary threshold for values from an external file_

_to be added_



## OTSU

_Derive Otsu optimal binary threshold for values from an internal EDF channel_

_to be added_


