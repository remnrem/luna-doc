# Helpers

_Miscellaneous functions designed to help with Luna workflows_

|Command | Description | 
|---|---|
| [`--build`](#-build)  | Generate a sample list automatically |
| [`--repath`](#-repath) | Change root filepaths in a sample list |
| [`--merge`](#-merge)  | Merge (concatenate) multiple EDFs |
| [`--xml`](#-xml) | View NSRR XML annotation files |
| [`--xml2`](#-xml2) | View NSRR XML annotation files (verbose) |
| [`--otsu](-otsu) | Calculate thresholds based on Otsu's method (external data) |
| [`OTSU](otsu) | Calculate thresholds based on Otsu's method (internal channel) |

## --build

_Automatically compile a sample list_

_to be added_

<h3>Parameters</h3>

| Parameter | Example | Description |
| --- | --- | --- |
| `-nsrr` |  | Assume NSRR file names |
| `-ext` | | Include this extension |

<h3>Output</h3>

A sample list written to the standard output stream.

<h3>Example</h3>

_to be added_


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
x
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

Future versions of this command will be able to merge "horizontally"
(i.e. EDFs with different channels) as well as handling overlapping
regions and/or gaps.  As it stands, this command only provides a
simple way to concatenate multiple EDFs that must have identical
header structures (in terms of the number of channels, their labels
and samples rates).  That is, currently we assume that all segments
specified are contiguous, and without gaps.  They do not need to be
of similar duration, however.

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



## --build

_Automatically compile a sample list_

See [here](../luna/args.md#-build-option) for details.

<h3>Parameters</h3>

| Parameter | Example | Description |
| --- | --- | --- |
| `-nsrr` |  | Assume NSRR file names |
| `-ext` | | Include this extension |

<h3>Output</h3>

A sample list written to standard output.



## --otsu

_Derive Otsu optimal binary threshold for values from an external file_

_to be added_



## OTSU

_Derive Otsu optimal binary threshold for values from an internal EDF channel_

_to be added_


