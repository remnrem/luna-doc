# _Lunapi_ Reference

This page describes the _high-level_ interface to Luna functions.
[This notebook](https://github.com/remnrem/luna-api-notebooks/blob/74ab1745ac524c7e9ec5ea86bc032b33b32d0644/99_reference.ipynb)
also contains a few further details.  Below, we assume the `lunapi`
package will always be aliased as `lp`, i.e.:

An auto-generated API reference is also available [here](https://remnrem.github.io/luna-api/). This page is intended to be a slightly more readable guide to the same interface.

```
import lunapi as lp
```

Using the package generally involves first initiating a _project_ class, which we'll call `proj`:

```
proj = lp.proj()
```

The _project_ is the primary "engine" that brings together various
Luna concepts including _sample lists_, _output databases_, _special
variables_, etc, alongside the ability to evaluate and execute Luna
functions. Given a _project_, you can create one or more data _instances_, that
represent signal and/or annotation data for a single observation.  We'll generally assume the
instance is called `p`:

```
p = proj.inst(x)
```

The sections below describe the suite of functions for a)
[manipulating projects](#projects), b) [working with
instances](#instances) as well as c) other [helper
functions](#helpers).  Below, we'll first tabulate all commands in the next section,
and then give some more details on usage in the following sections.


## Command tables

Major commands are described in __bold text__.  Below:

 - the _lunapi_ package is aliased as `lp`

 - the `proj` class is assumed to also be called `proj` (i.e. from `proj = lp.proj()`)

 - any `inst` class is assumed to be called `p` (i.e. from `p = proj.inst(x)`)

### Projects 

Projects are _singleton classes_ that organize Luna functions and objects within a single Python session. 

<h5>Project/instance creation</h5>

| Command | Description |
|----|-----|
| [__`lp.proj()`__](#lpproj) | __Initiate/reference a _lunapi_ project__ |
| [__`proj.inst()`__](#projinst)  | __Create a new instance__ |
| [`proj.empty_inst()`](#projempty_inst) | Create an empty instance | 
| [`proj.retire()`](#projretire) | Retire a project | 

<h5>Sample lists</h5>

| Command | Description |
|----|-----|
| __[`proj.build()`](#projbuild)__ | __Traverses folders (recursively) to generate a lunapi sample-list__ |
| __[`proj.sample_list()`](#projsample_list)__ | __Reads a sample-list from a file, or returns an existing sample-list__ | 
| [`proj.validate()`](#projvalidate) | Validate all files in a sample list |
| [`proj.nobs()`](#projnobs) | Returns of number of observations in the current sample-list |
| [`proj.get_n()`](#projget_n) | Returns of index (row) of an observations in the current sample-list | 
| [`proj.get_id()`](#projget_id) | Returns the ID of a sample-list observation |
| [`proj.get_edf()`](#projget_edf) | Returns the EDF filename of a sample-list observation |
| [`proj.get_annots()`](#projget_annots) | Returns the annotation filename(s) of a sample-list observation |
| [`proj.clear()`](#projclear) | Clears the current sample-list |
| [`proj.desc()`](#projdesc) | Returns descriptive information for all sample-list individuals |

<h5>Executing Luna commands</h5>

| Command | Description |
|----|-----|
| __[`proj.proc()`](#projproc)__ | __Evaluate Luna commands on all sample-list individuals__ |
| __[`proj.proc_parallel()`](#projproc_parallel)__ | __Evaluate Luna commands on all sample-list individuals using worker processes__ |
| [`proj.procn()`](#projprocn) | Convenience alias for `proj.proc_parallel()` |
| [`proj.silent_proc()`](#projsilent_proc) | Evaluate Luna commands silently |
| [`ProcResult`](#procresult-live-view) | Result object returned by all proc calls; dict-like interface; `.copy()` for snapshots |
| __[`proj.strata()`](#projstrata)__ | __Return a list of command/strata pairs from the project results cache__ |
| __[`proj.table()`](#projtable)__ | __Return a table as a dataframe from the project results cache__ |
| [`proj.commands()`](#projcommands) | Return a list of commands in the project results cache | 
| [`proj.variables()`](#projvariables) | Return the variables (table header) for a specific command/strata pair from the project results cache |
| [`proj.empty_result_set()`](#empty_result_set) | Indicate whether the project results cache is empty |

<h5>Variables</h5>

| Command | Description |
|----|-----|
| __[`proj.var()`](#projvar)__ | __Get/set project-wide variable__ |
| [`proj.vars()`](#projvars) | Gets/sets project-wide options/variables |
| [`proj.clear_vars()`](#projclear_vars) | Clears one, some, or all project-wide options/variables |
| [`proj.clear_ivars()`](#projclear_ivars) | Clears all individual-specific variables (for all individuals) | 
| [`proj.include()`](#projinclude) | Reads and sets project variables from a parameter file |
| [`proj.aliases()`](#projaliases) | Returns signal and annotation aliases |

<h5>Models</h5>

| Command | Description |
|----|-----|
| __[`proj.pops()`](#projpops)__ | __A project-level wrapper for the POPS stager__ |
| [`proj.predict_SUN2019()`](#projpredict_sun2019) | A project-level wrapper for the SUN2019 biological age model |

<h5>Misc.</h5>

| Command | Description |
|----|-----|
| [`proj.silence()`](#projsilence) | Turns off the console/log echoing |
| [`proj.is_silenced()`](#projis_silenced) | Reports on whether the console/log is silenced |
| [`proj.reset()`](#projreset) | Drops the Luna problem flag |
| [`proj.reinit()`](#projreinit) | Re-initializes the project |
| [`proj.flush()`](#projflush) | Flushes the cached output buffer |
| [`proj.import_db()`](#projimport_db) | Imports a prior Luna output database from a file |


### Instances 

Instances are generally created by the `proj.inst()` command; if the individual exists
in the project's sample list, then any EDF and annotation files are automatically attached.

<h5>Creation</h5>

| Command | Description |
|----|-----|              
| __[`inst.attach_edf()`](#instattach_edf)__ | __Explicitly attach an EDF__ |
| __[`inst.attach_annot()`](#instattach_annot)__ | __Attach an annotation file__ |
| [`inst.refresh()`](#instrefresh) | Drop any changes made to the attached data |

<h5>Basic summaries</h5>

| Command | Description |
|----|-----|
| __[`inst.headers()`](#instheaders)__ | Returns a dataframe of channel-header information | 
| [`inst.id()`](#instid) | Returns the current instance identifier |
| [`inst.desc()`](#instdesc) | Returns a descriptive summary of the attached record |
| __[`inst.annots()`](#instannots)__ | Returns a list of current annotation classes |
| __[`inst.channels()`](#instchannels)__ | Returns a list of current channels |
| [`inst.chs()`](#instchs) | Alias for `inst.channels()` |
| [`inst.stat()`](#inststat) | Returns information on the attached data |
| [`inst.has_channels()`](#insthas_channels) | Returns a list of true/false values for whether certain channels exist |
| [`inst.has()`](#insthas) | Alias for `inst.has_channels()` |
| [`inst.has_annots()`](#insthas_annots) | Returns a list of true/false values for the presence of certain annotation classes |
| [`inst.has_annot()`](#insthas_annot) | Alias for `inst.has_annots()` |
| [`inst.fetch_annots()`](#instfetch_annots) | Returns annotation events as a dataframe |
| [`inst.fetch_fulls_annots()`](#instfetch_fulls_annots) | Returns full annotation events as a dataframe |
| [`inst.stages()`](#inststages) | Returns a list of current sleep stages |
| [`inst.has_staging()`](#insthas_staging) | Returns true/false for whether stage annotations are available |

<h5>Executing Luna commands</h5>

| Command | Description |
|----|-----|
| __[`inst.eval()`](#insteval)__ | Evaluate arbitrary Luna commands |
| [`inst.eval_dummy()`](#insteval_dummy) | Evaluate commands in dummy mode and return backend log text |
| [`inst.eval_lunascope()`](#insteval_lunascope) | Evaluate commands for the LunaScope viewer and return log text |
| __[`inst.strata()`](#inststrata)__ | Reports on the contents of the current result cache |
| __[`inst.table()`](#insttable)__ | Displays a table from the current result cache |
| [`inst.proc()`](#instproc) | Evaluate arbitrary Luna commands and directly return all results |
| [`inst.silent_proc()`](#instsilent_proc) | Evaluate arbitrary Luna commands silently |
| [`inst.silent_proc_lunascope()`](#instsilent_proc_lunascope) | Internal silent evaluation helper for LunaScope |
| [`inst.variables()`](#instvariables) | Return the variables for a specific command/strata pair |
| [`inst.empty_result_set()`](#instempty_result_set) | Indicates whether the results cache is currently empty |

<h5>Individual-level variables</h5>

| Command | Description |
|----|-----|
| [`inst.var()`](#instvar) | Set or get an individual variable |
| [`inst.vars()`](#instvars) | Returns or sets individual variables |
| [`inst.clear_vars()`](#instclear_vars) | Clears one, some, or all individual variables |

<h5>Extracting signals & annotations</h5>

| Command | Description |
|----|-----|
| __[`inst.data()`](#instdata)__ | __Returns an array of signal/annotation data__ |
| [`inst.slice()`](#instslice) | Returns an array of merged signal/annotation data based on selected intervals (slices) |
| [`inst.slices()`](#instslices) | Returns an array of individual signal/annotation data based on selected intervals (slices) |
| [`inst.e2i()`](#inste2i) | Helper function to convert epochs to intervals |
| [`inst.s2i()`](#insts2i) | Helper function to convert epochs to intervals |
| [`inst.mask()`](#instmask) | Applies one or more mask expressions and rebuilds epochs |
| [`inst.segments()`](#instsegments) | Runs [`SEGMENTS`](../ref/outputs.md#segments) and returns the `SEGMENTS: SEG` table |
| [`inst.epoch()`](#instepoch) | Runs [`EPOCH`](../ref/epochs.md#epoch) with optional arguments |
| [`inst.epochs()`](#instepochs) | Returns a compact epoch summary table |

<h5>Updating signals & annotations</h5>

| Command | Description |
|----|-----|
| __[`inst.insert_signal()`](#instinsert_signal)__ | __Inserts a new signal into the in-memory EDF__ |
| [`inst.update_signal()`](#instupdate_signal) | Updates an existing signal in the in-memory EDF |
| [`inst.insert_annot()`](#instinsert_annot) | Insert/append annotation events |
| [`inst.freeze()`](#instfreeze) | Saves the current timeline mask to a freezer tag |
| [`inst.thaw()`](#instthaw) | Restores a saved freezer tag |
| [`inst.empty_freezer()`](#instempty_freezer) | Clears all freezer tags for this instance |

<h5>Models</h5>

| Command | Description |
|----|-----|
| [`inst.pops()`](#instpops) | Run POPS stager for a single individual |
| [`inst.predict_SUN2019()`](#instpredict_sun2019) | Fit Sun et al (2019) brain-age prediction model |

<h5>Plotting</h5>

| Command | Description |
|----|-----|
| [`inst.hypno()`](#insthypno) | Plot a hypnogram given sleep stage data | 
| [`lp.hypno_density()`](#lphypno_density) | Make a hypno-density (posterior stage probabilities) | 
| [`inst.psd()`](#instpsd) | Calculate and plot a PSD curve |
| [`inst.spec()`](#instspec) | Calculate and plot a spectrogram heatmap |
| [`inst.tfview()`](#insttfview) | Plot an MTM spectrogram view for a selected interval |
| [`lp.topo_heat()`](#lptopo_heat) | Topo-plot |


### Helpers

<h5>Utility functions</h5>

| Command | Description |
|----|-----|
| [`lp.cmdfile()`](#lpcmdfile) | Loads and parses a Luna command file |
| [`proj.include()`](#projinclude) | Reads and sets project variables based on a parameter file |
| [`lp.fetch_doms()`](#lpfetch_doms) | Lists all Luna command domains |
| [`lp.fetch_cmds()`](#lpfetch_cmds) | Lists all commands for a domain |
| [`lp.fetch_params()`](#lpfetch_params) | Lists parameters for a command |
| [`lp.fetch_tbls()`](#lpfetch_tbls) | Lists output tables for a command |
| [`lp.fetch_vars()`](#lpfetch_vars) | Lists variables for a command/table |
| [`lp.fetch_desc_dom()`](#lpfetch_desc_dom) | Returns the description for a domain |
| [`lp.fetch_desc_cmd()`](#lpfetch_desc_cmd) | Returns the description for a command |
| [`lp.fetch_desc_param()`](#lpfetch_desc_param) | Returns the description for a command parameter |
| [`lp.fetch_desc_tbl()`](#lpfetch_desc_tbl) | Returns the description for a command table |
| [`lp.fetch_desc_var()`](#lpfetch_desc_var) | Returns the description for a command variable |
| [`lp.strata()`](#lpstrata) | Lists command/strata pairs from a raw results object |
| [`lp.table()`](#lptable) | Converts one command/strata pair to a dataframe |
| [`lp.tables()`](#lptables) | Converts all raw results into dataframes |
| [`lp.show()`](#lpshow) | Displays a set of result tables |
| [`lp.subset()`](#lpsubset) | Subsets rows and columns of a result table |
| [`lp.concat()`](#lpconcat) | Concatenates matching tables across result collections |
| [`lp.version()`](#lpversion) | Returns the `lunapi` and Luna versions |

<h5>Output database reader</h5>

| Command | Description |
|----|-----|
| [__`lp.destrat()`__](#lpdestrat) | __Read one or more Luna output databases__ |
| [`lp.list_text_tables()`](#lplist_text_tables) | List tables in a Luna text-output folder |
| [`lp.read_text_table()`](#lpread_text_table) | Read a concatenated text-output table from a Luna text-output folder |

<h5>EDF utilities</h5>

| Command | Description |
|----|-----|
| [`lp.merge_edfs()`](#lpmerge_edfs) | Concatenate EDFs in time (mirrors `luna --merge`) |
| [`lp.bind_edfs()`](#lpbind_edfs) | Bind EDFs by adding channels (mirrors `luna --bind`) |
| [`lp.overlap()`](#lpoverlap) | Multi-sample annotation overlap / enrichment analysis (mirrors `luna --overlap`) |

<h5>BioData Catalyst</h5>

| Command | Description |
|----|-----|
| [`lp.BDCClient`](#biodata-catalyst) | Authenticated browser/downloader for Gen3/BioData Catalyst files |
| [`lp.bdc`](#biodata-catalyst) | Alias for `BDCClient` |

### Scope

| Command | Description |
|----|-----|
| __[`lp.scope()`](#lpscope)__ | __Initiate the Scope viewer__ |


---


## Projects

### lp.proj()

_Constructor for proj class_

```
 proj()
    
    Args:
      none

    Returns:
      reference to a proj object

    Example:
      proj = lp.proj() 
```

This calls `lunapi.lunapi0.inaugurate()`, which constructs a singleton
instance of a Luna _project_.  A session can only contain one
_project_: subsequent calls to `proj()` will return a reference to
the same _project_.


### proj.inst()

_Creates a new instance (individual)_

```
 inst( x )

    Args:
      x (str)    If there is an attached sample-list with individual ID 'x'
                 then this individual is attached (EDF & annotations)
                 Otherwise, an empty instance with ID 'x' is created

      x (int)    Assuming a sample-list is attached, generate a new
                 instance and attach the EDF/annotations; uses 1-based indexing

    Returns:
      reference to an inst object

    Example:
      p = proj.inst('id1')

```

This is the primary way to create a new _instance_ in `lunapi`.
If `x` is a string that doesn't match an entry in the attached sample-list (from `sample_list()`), 
then a new, empty `inst` object is created with that ID.  Data can subsequently be attached
with the instance-level `attach_edf()` and `attach_annot()` functions.

Alternatively, if a _sample-list_ exists within the _project_ class and `x`
indexes (1-based) the entry in the list, then as well as creating an `inst` with the ID
from the _sample-list_, this variant of `inst()` will also automatically attach the specified
EDF (via `attach_edf()`) and any annotation files also specified (via `attach_annot()`).  Likewise, 
if `x` is a string that matches an ID in the _sample-list_, the associated EDF/annotations will be attached.
Otherwise, an empty instance will be created, as above.

### proj.empty_inst()

_Create an empty EDF_

This provides a means to create an empty EDF, similar to the command-line Luna interface [as described here](../luna/args.md#empty-edfs).


```
 empty_inst( id , nr , rs ,
             startdate = '01.01.00', starttime = '00.00.00' )

    Args:
      id (str)         ID for the new (empty/zero-channel) instance
      nr (int)         number of EDF records
      rs (float)       duration of each EDF record (in seconds)
      startdate (str)  EDF format date-string for date
      starttime (str)  EDF format time-string for start
      
    Returns:
      reference to an empty inst object

    Example:
      p = proj.empty_inst('id1' , 3600 , 1 )

```

This creates an empty EDF with fixed duration of `nr` * `rs` seconds.
Signals can be added via using `inst.eval()` and Luna commands (e.g. for
simulating signals), or directly from Python objects via
[inst.insert_signal()](#instinsert_signal).


### proj.retire()

_Retire a project_

```
 retire()

    Args:
      none

    Returns:
      backend status value

    Example:
      proj.retire()

```

This closes an existing _project_ and frees all resources.


### proj.build()

_Traverses folders (recursively) to generate a lunapi sample-list_

```
 build( args )

    Args:
      args (str or list[str])  one or more folders and/or special options

    Returns:
      nothing; it creates an internal sample-list

    Example:
      proj.build( [ '/tutorial/' , '/path/to/more/data' , '-ext=-profusion.xml' ] )
	  
```

Internally, this function calls the same code used by the `--build`
option for the `luna` command-line tool.  If a specified folder does
not exist, this function returns a RuntimeError.


### proj.sample_list()

_Reads a sample-list from a file, or returns an existing sample-list_

```
 sample_list( filename = None , path = None , df = True )

    Args:
      filename (str, optional)  filename of the sample-list
      path (str, optional)      path to prepend to all relative file paths when building the sample-list
      df (boolean)              return a pandas dataframe rather than python list (default=True)

    Returns:
      nothing, if reading a sample-list
      a dataframe/list of (ID,EDF,set(annotation files))-tuples, if filename is None

    Example:
      proj.sample_list( '/tutorial/s.lst' )
      proj.sample_list()

```

If the sample list uses relative paths that are not appropriate for your current directory,
you can set the `path` argument to add a prefix to all relative paths in the sample list.  Alternatively
(and equivalently), you can set the project variable `path` before calling `sample_list(filename)` to
achieve the same result: `proj.var( 'path' , '/path/to/data/' )`.   For example, if the current working folder
is `/home/joe/work1/` and the data are in `/data/proj1/`

```
  /data/proj1/
  /data/proj1/s.lst
  /data/proj1/edfs/
  /data/proj1/annots/
```
If the sample-list `s.lst` is in the form, e.g.:
```
id1   edfs/id1.edf   annots/id1.annot
id2   edfs/id2.edf   annots/id2.annot
id3   edfs/id3.edf   annots/id3.annot
```
then after attaching the sample-list via `sample_list( '/data/proj1/s.lst' )`, Luna would look for the data (e.g. for `id1`) in `/home/joe/edfs/ed1.edf`, etc.    Instead, running `sample_list( '/data/proj1/s.lst' , '/data/proj1/' )` would create an internal sample-list as follows:
```
id1   /data/proj1/edfs/id1.edf   /data/proj1/annots/id1.annot
id2   /data/proj1/edfs/id2.edf   /data/proj1/annots/id2.annot
id3   /data/proj1/edfs/id3.edf   /data/proj1/annots/id3.annot
```
and so Luna would be able to look in the correct locations.


### proj.validate()

_Validate all files in a sample list_

```
 validate()

    Args:
      none

    Returns:
      a dataframe with `ID`, `Filename`, and `Valid` columns

    Example:
      proj.sample_list( 's.lst' )
      proj.validate()
```

This provides the same functionality as the `--validate` option of
Luna, which is described [here](../ref/helpers.md#-validate).

### proj.reset()

_Drop the Luna problem flag_

```
 reset()

    Args:
      none

    Returns:
      nothing
```

### proj.reinit()

_Re-initialize the project_

```
 reinit()

    Args:
      none

    Returns:
      nothing
```

This resets project-level variables and state in the underlying Luna engine without replacing the project object itself.


### proj.nobs()

_Returns of number of observations in the current sample-list_

```
 nobs()

    Args:
      none

    Returns:
      number of observations (int)

    Example:
      proj.nobs()
```

### proj.get_n()

_Returns of index (row) of an observations in the current sample-list_

```
 get_n(id)

    Args:
      id (str)  ID corresponding to first field of the sample-list

    Returns:
      0-based index (int), it the ID exists in the sample-list
      NoneType, if the ID does not exist in the sample-list

    Example:
      proj.nobs()
```


### proj.get_id()

_Returns the ID of a sample-list observation_

```
 get_id(n)

    Args:
      n (int)  a 0-based index for the sample-list

    Returns:
      ID (str), if n is a valid index
      NoneType, if n is not a valid index

    Example:
      proj.get_id(0)

```

### proj.get_edf()

_Returns the EDF filename of a sample-list observation_

```
 get_edf(x)

    Args:
      x (int)  a 0-based index for the sample-list
      x (str)  an ID matching a sample-list observation

    Returns:
      annotation filename(s) (str), if x is a valid index
      NoneType, if x is not a valid index

    Example:
      proj.get_annots(0)

```

If `x` is a `str`, this calls the function is `get_n(x)`.


### proj.get_annots()

_Returns the annotation filename(s) of a sample-list observation_

```
 get_annots(x)

    Args:
      x (int)  a 0-based index for the sample-list
      x (str)  an ID matching a sample-list observation
      
    Returns:
      EDF filename (str), if x is a valid index
      NoneType, if x is not a valid index

    Example:
      proj.get_edf(0)

```

If `x` is a `str`, this calls the function is `get_n(x)`.

### proj.clear()

_Clears the current sample-list_


```
 clear()

    Args:
      none

    Returns:
      nothing; will only clear an existing sample-list

    Example:
      proj.clear()

```

### proj.desc()

_Return descriptive information for all sample-list individuals_

```
 desc()

    Args:
      none

    Returns:
      a dataframe of per-individual descriptive information
```


### proj.proc()

_Evaluate Luna commands on all sample-list individuals_

```
 proc( cmdstr )

    Args:
      cmdstr (str)  a valid Luna command script 

    Returns:
      ProcResult

    Example:
      res = proj.proc( 'HEADERS' )
      res.table( 'HEADERS' )

```

`proc()` populates the project results cache and returns a `ProcResult` object.  Results can be accessed
either through the returned object or directly via `proj.strata()`, `proj.table()`, etc.:

```
res = proj.proc( 'HEADERS' )
res.table( 'HEADERS' )      # via the returned ProcResult
proj.table( 'HEADERS' )     # equivalently, directly from the project cache
```

!!! note "ProcResult is a live view, not a snapshot"
    `ProcResult` is a lightweight wrapper that delegates all table queries to the
    **project results cache**.  It does not hold copies of the DataFrames itself.
    This means that calling `proc()` again overwrites the cache, and any previously
    returned `ProcResult` will now reflect the new results.

    This is a common source of confusion when calling `proc()` in a loop:

    ```python
    # BUG: all entries in res point to the same live cache
    res = {}
    for stage in ['N1', 'N2', 'N3', 'R']:
        proj.var('stage', stage)
        res[stage] = proj.silent_proc(cmdstr)   # each iteration overwrites the cache
    res['N1']['STATS: CH_STAGE']   # silently returns R-stage results, not N1
    ```

    To capture a true snapshot, call `.copy()` on the returned `ProcResult`.
    This produces a frozen `ProcResult` that owns its DataFrames independently
    of the cache and supports the full `ProcResult` interface:

    ```python
    res = {}
    for stage in ['N1', 'N2', 'N3', 'R']:
        proj.var('stage', stage)
        res[stage] = proj.silent_proc(cmdstr).copy()   # frozen snapshot
    res['N1']['STATS: CH_STAGE']       # correct N1 results
    res['N1'].table('STATS', 'CH_STAGE')
    res['N1'].strata()
    ```

The `lp.cmdfile()` utility function can be used to pass a file-based Luna script to `proc()` (i.e.
which will strip out comments, etc).

Multi-line scripts can be passed by using triple-quotes, e.g.:
```
proj.proc( """
MASK ifnot=N2
RE
STATS sig=${eeg}
""" )
```

Note that `proc()` can use both project-wide and individual-specific
variables in scripts, e.g. as above (`${eeg}`).

Note: a similar form of this command exists, `proj.silent_proc()`, which has identical syntax but suppresses console/log output.


### proj.proc_parallel()

_Evaluate Luna commands on all sample-list individuals using worker processes_

```
 proc_parallel( cmdstr, workers=None, batch_size=None, params=None, param_file=None,
               strict=False, progress=True, out_db=None, out_text=None, in_memory=None,
               n1=None, n2=None, ids=None, skip=None )

    Args:
      cmdstr (str)       a valid Luna command script
      workers (int)      number of worker processes; if omitted, a conservative default is used
                         (min(10, cpu_count // 2)).  Values are capped at the available CPU
                         count and the number of selected records.
      batch_size (int)   number of sample-list rows sent to each worker task
      params (dict)      optional Luna variables to pass explicitly to workers
      param_file (str)   optional parameter file of Luna variables to pass to workers
      strict (bool)      raise an error if any row or worker fails
      progress (bool)    show a progress bar while worker tasks complete (default: True)
      out_db (str)       base path for per-worker Luna output database files.
                         Each worker writes to '{out_db}-{slice_idx}.db'.  Results are
                         NOT held in memory; table queries raise FileOutputModeError.
                         Mutually exclusive with out_text.
      out_text (str)     path to a folder for per-individual plain-text output (equivalent
                         to the luna -t flag).  Each individual's tables are written to
                         '{out_text}/{id}/CMD_FACTORS.txt'.  Mutually exclusive with out_db.
      in_memory (bool)   explicitly select in-memory mode.  Defaults to True unless out_db
                         or out_text is supplied; it cannot be combined with file output.
      n1 (int)           first sample-list row to process (1-based, inclusive).
                         Mirrors the luna n1 option.
      n2 (int)           last sample-list row to process (1-based, inclusive).
                         Mirrors the luna n2 option.
      ids (str or list)  process only the individuals named here; a plain string
                         is split on whitespace.  Mirrors luna id=.
      skip (str or list) exclude the individuals named here; a plain string
                         is split on whitespace.  Mirrors luna skip=.

    Returns:
      ProcResult

    Example:
      res = proj.proc_parallel( 'HEADERS', workers=4 )
      res.ok
      res.table( 'HEADERS' )
```

`proc_parallel()` is the process-based version of `proj.proc()`.  It uses separate
Python worker processes rather than threads, which keeps each worker's Luna state
isolated and avoids shared static resources.  After all workers complete, results are
injected into the project's results cache, so `proj.table()` and `proj.strata()` work
identically regardless of whether `proc()` or `proc_parallel()` was used.

The returned `ProcResult` is a lightweight wrapper that delegates all table queries
back to the project cache — no separate copy of the data is held.

For example, for a command with multiple strata:

```
res = proj.proc_parallel( """
EPOCH len=30
PSD sig=C4 spectrum dB
""" , workers=4 , batch_size=5 )

res.ok
res.strata()
psd = res.table( 'PSD' , [ 'CH' , 'F' ] )
```

The list form for `strata` is order-insensitive, so `[ 'CH' , 'F' ]` and
`[ 'F' , 'CH' ]` refer to the same table.  Results can also be accessed with
a string key: `res[ 'HEADERS: BL' ]`.  To see what tables are available:

```
res.strata()              # DataFrame of Command/Strata pairs
res.commands()            # unique commands
res.table_index()         # dict: { cmd: [[factor_list], ...] }
res.has_table( 'PSD' , 'CH_F' )
```

Numeric columns are automatically coerced to numeric types after all rows are
collated.  The `ID` column is always kept as a string.

If `strict=False` (the default), failed rows are collected in `res.errors` and
successful rows are still returned.  `res.ok` is `True` only when there were no
per-row errors.  If `strict=True`, any failure raises `ProcError`; the exception
has a `result` attribute containing the partial `ProcResult`.

Worker processes do not implicitly inherit project variables from the parent
process.  Pass variables explicitly with `params`, `param_file`, or both.  If the
same key appears in both places, the value in `params` takes precedence.

**Workers on cloud/HPC nodes**: when `workers` is `None`, the default is
`min(10, cpu_count // 2)` — a conservative cap suitable for shared laptops and
notebooks. An explicit value is limited by the available CPU count and by the
number of selected records.

**File-output mode** (`out_db` / `out_text`): output is written directly to disk
rather than accumulated in the project results cache.  This is useful when the
combined in-memory result would be too large to hold at once.

- `out_db` — each worker slice writes a Luna output database file named
  `{out_db}-{slice_idx}.db`.  These can later be read with `proj.import_db()`
  or `lp.destrat()`.
- `out_text` — each individual's tables are written as tab-delimited text files
  under `{out_text}/{id}/CMD_FACTORS.txt`, mirroring the `luna -t` convention.
  Use `lp.list_text_tables()` and `lp.read_text_table()` to read the results.

The two modes are mutually exclusive.  In either case the returned `ProcResult`
carries error and records metadata but raises `FileOutputModeError` if you try to
call `.table()` or `.strata()` on it.

```python
# Write per-worker .db files
res = proj.proc_parallel('PSD sig=EEG spectrum', workers=8, out_db='out/run')
# → writes out/run-1.db, out/run-2.db, ...
db = lp.destrat('out/run-*.db')
df = db.get('PSD', r=['B', 'CH'])

# Write plain-text tables
res = proj.proc_parallel('HEADERS', workers=4, out_text='out/txt')
lp.list_text_tables('out/txt')
df = lp.read_text_table('out/txt', 'HEADERS', factors='CH')
```

```
res = proj.proc_parallel( ' EPOCH len=30 & PSD sig=${s} spectrum dB ' ,
                          workers=4, params={ 's': 'C4' } )
```

A parameter file follows the usual Luna convention of one key/value pair per row,
using either whitespace/tab separation or `key=value` syntax, for example:

```
sig C4
th=3
```

**Row and ID filtering** mirrors the Luna command-line `n1`, `n2`, `id=`, and `skip=`
options.  All four can be combined:

```python
# process rows 1–50 of the sample list
res = proj.proc_parallel('HEADERS', workers=4, n1=1, n2=50)

# process specific individuals
res = proj.proc_parallel('HEADERS', workers=4, ids=['subj01', 'subj02'])

# skip a list of individuals
res = proj.proc_parallel('HEADERS', workers=4, skip=['subj99', 'subj100'])

# combine row-range and ID exclusion
res = proj.proc_parallel('HEADERS', workers=4, n1=1, n2=200, skip='subj99')
```

`n1`/`n2` are 1-based, inclusive row numbers that refer to the sample-list order.
`ids` and `skip` accept a list of strings or a whitespace-separated string.
Filtering happens before workers are dispatched, so only the matching rows consume
worker capacity.

Because Python process creation has overhead, especially in notebooks,
`proc_parallel()` is most useful when each individual command run is non-trivial.
Larger `batch_size` values reduce scheduling overhead but update progress less often.


### proj.procn()

_Convenience alias for `proj.proc_parallel()`_

```
 procn( cmdstr, workers=None, batch_size=None, params=None, param_file=None,
        strict=False, progress=True, out_db=None, out_text=None, in_memory=None,
        n1=None, n2=None, ids=None, skip=None )

    Args:
      see proj.proc_parallel()

    Returns:
      ProcResult

    Example:
      res = proj.procn( 'HEADERS', workers=4 )
```

`procn()` is a shorter name for `proj.proc_parallel()`; the behavior and return
object are identical.


### proj.silent_proc()

_Evaluate Luna commands on all sample-list individuals without console/log output_

```
 silent_proc( cmdstr )

    Args:
      cmdstr (str)  a valid Luna command script

    Returns:
      ProcResult

    Example:
      proj.silent_proc( 'HEADERS' )
      proj.table( 'HEADERS' )
```

Identical to `proj.proc()` but suppresses console/log output for the duration of the call.


### ProcResult { #procresult-live-view }

All `proc()`, `silent_proc()`, `proc_parallel()`, and `procn()` calls return a `ProcResult`
object.  It supports a dict-like interface over the results:

```python
res = proj.proc( 'EPOCH len=30 & PSD sig=EEG spectrum' )

'PSD: B_CH' in res              # True/False
res['PSD: B_CH']                # DataFrame
res.table('PSD', 'B_CH')        # same
res.strata()                    # DataFrame of Command/Strata pairs
res.commands()                  # DataFrame of unique commands
res.has_table('PSD', 'B_CH')    # True/False
res.ok                          # True if no per-row errors
res.errors                      # DataFrame of errors
res.keys()                      # all 'CMD: STRATA' keys
res.items()                     # (key, DataFrame) pairs
```

**`ProcResult` is a live view.**  It holds a reference to the project (or instance)
results cache and delegates every table query there at call time.  It contains
no copies of the data itself.  Calling `proc()` again replaces the cache, so any
previously returned `ProcResult` immediately reflects the new results.

**`.copy()` — frozen snapshot**

Call `.copy()` to produce a self-contained `ProcResult` that owns its DataFrames
independently of the cache.  This is essential when accumulating results across
multiple `proc()` calls, for example looping over conditions:

```python
# Without .copy() — WRONG: all entries silently share the same cache
res = {}
for stage in ['N1', 'N2', 'N3', 'R']:
    proj.var('stage', stage)
    res[stage] = proj.silent_proc(cmdstr)       # live view, cache overwritten each iter
res['N1']['STATS: CH_STAGE']    # returns R-stage data, not N1

# With .copy() — correct
res = {}
for stage in ['N1', 'N2', 'N3', 'R']:
    proj.var('stage', stage)
    res[stage] = proj.silent_proc(cmdstr).copy()   # frozen snapshot
res['N1']['STATS: CH_STAGE']    # N1 data
res['N2'].table('STATS', 'CH_STAGE')
res['N3'].strata()
```

A frozen `ProcResult` returned by `.copy()` supports the full interface above —
`[]`, `.table()`, `.strata()`, `.commands()`, `in`, `.items()`, etc. — but reads
from its own internal copy of the DataFrames rather than the live cache.


### proj.strata()

_Return a list of command/strata pairs from the project results cache_

```
 strata()

    Args:
      none

    Returns:
      a dataframe of commands and strata pairs in the project-level results cache

    Example:
      proj.proc( 'HEADERS' )
      proj.strata()

```

The project results cache is populated by `proj.proc()`, `proj.proc_parallel()`, and `proj.procn()`.


### proj.table()

_Return a table as a dataframe from the project results cache_

```
 table( cmd , strata = 'BL' )

    Args:
      cmd (str)                           case-sensitive command name
      strata (str or list, optional)      stratum label (default: 'BL'), or a list of
                                          factor names resolved order-independently, e.g. ['CH','F']

    Returns:
      a dataframe of values associated with a specific command/strata pair in the project-level results cache

    Example:
      proj.proc( 'HEADERS' )
      proj.table( 'HEADERS' )
      proj.table( 'HEADERS' , 'CH' )
      proj.table( 'PSD' , [ 'CH' , 'F' ] )

```

The project results cache is populated by `proj.proc()`, `proj.proc_parallel()`, and `proj.procn()`.
A list of available tables is given by `proj.strata()`.  All individuals from the sample-list are
combined as different rows of the same results table.

The `strata` argument accepts either a string label (e.g. `'CH_F'`) or a list of factor names
(e.g. `['CH', 'F']`); the list form is order-insensitive.

### proj.commands()

_Return a list of commands in the project results cache_

```
 commands()

    Args:
      none

    Returns:
      a dataframe of commands in the project-level results cache

    Example:
      proj.proc( 'HEADERS' )
      proj.commands()

```

The project results cache is populated by `proj.proc()`, `proj.proc_parallel()`, and `proj.procn()`.



### proj.variables()

_Return the variables (table header) for a specific command/strata pair from a prior `proc()` run_

```
 variables( cmd , strata = 'BL' )

    Args:
      cmd (str)	  case-sensitive command name in the project results cache
      strata (str, optional, defaults to BL)  case-sensitive stratum in	the project results cache

    Returns:
      a dataframe of variable names from the table associated with a specific command/strata pair in the project-level results cache

    Example:
      proj.proc( 'HEADERS' )
      proj.variables( 'HEADERS' )
      proj.variables( 'HEADERS' , 'CH' )

```

This is similar to `proj.table()` but only returns table headers (i.e. variable names).


### empty_result_set()

_Indicate whether the project results cache is empty_

```
 proj.empty_result_set()

    Args:
      none

    Returns:
      bool, True if the project results cache is empty
      
    Example:
      proj.proc( 'HEADERS' )
      proj.empty_result_set()

```

The project results cache stores the results of the last successful run of `proj.proc()`.


### proj.var()
_Sets or gets a project-wide option/variable_

```
 var(key=None,value=None)

    Args:
      key (str, optional)    variable name
      value (str, optional)  value to set variable to 

    Returns:
      nothing, if value is not None (it sets the option)
      the value of the variable, if value is None but key is not
      vars(), if key and value are both None 

    Example:
      proj.var( 'path' , '/tutorial/')
      proj.var( 'path' )
```

### proj.vars()

_Gets/sets project-wide options/variables_

```
 vars( key = None , value = None )

  Usage 1: return all variables  
    Args:
      none
    Returns:
      a `dict` of `variable: value` pairs

  Usage 2: return a specific variable
    Args:
      key (str) 
    Returns:
      a `str` value for that variable (or `None` if it does not exist)

  Usage 3: set a single key/value pair
    Args:
      key (str)                   variable name
      value (str, int or float)   variable value (scalar, that can be cast to a str)
    Returns:
      nothing

  Usage 4: set multiple key/value pairs
    Args:
      key (dict)     dictionary of key/value pairs to set
    Returns:
      nothing

    Example:
      proj.vars( 'a', 2 )    # sets ${a} to 2
      proj.vars( { 'a': 2 , 'stage': 'N2' } )
      proj.vars( 'stage' )   # returns 'N2'
      proj.vars()            # returns all variables (including presets)
```

Special variables (e.g. `path` or `annot-file`) are enacted rather than simply stored, and some project-wide variables may be set automatically.

### proj.clear_vars()

_Clears one, some, or all project-wide options/variables_

```
 clear_vars( key = None )

    Args:
      key (str or list[str], optional)  variable(s) to clear; if omitted, clears all

    Returns:
      nothing

    Example:
      proj.clear_vars( 'path' )
      proj.clear_vars( [ 'path' , 'annot-file' ] )
      proj.clear_vars()
```

If `key` is omitted, all project-level variables are cleared.

### proj.clear_ivars()

_Clears all individual-specific variables (for all individuals)_

```
 clear_ivars()

    Args:
      none

    Returns:
      nothing

    Example:
      proj.clear_ivars()
```

This clears any previously attached individual-specific variables.  Typically, these will be
attached through Luna's `vars` special variable, i.e. `proj.var( 'vars' , 'path/to/ivar.txt' )`.
(Note that although the Luna variable is `vars` it would perhaps have been better called `ivars`,
as `lunapi` uses the `vars`/`ivars` nomenclature to distinguish between project-wide and individual-specific
variables.

### proj.include()

_Include options and variables from a parameter file_

```
 include( f )

    Args:
      f (str)  parameter file to read

    Returns:
      backend return value from the Luna wrapper
```

This is the project-level equivalent of using `@file` on the Luna command line.

### proj.aliases()

_Return a table of signal and annotation aliases_

```
 aliases()

    Args:
      none

    Returns:
      a dataframe with alias type and preferred/alias labels
```

### proj.pops()

_A project-level wrapper for the POPS stager_

```
 pops( s = None, s1 = None , s2 = None,
       path = None , lib = None ,
       do_edger = True ,
       no_filter = False ,
       do_reref = False ,
       m = None , m1 = None , m2 = None,
       lights_off = '.' , lights_on = '.',
       ignore_obs = False, args = '' )

    Args:
      s (str, optional)   central EEG channel (in single-channel mode)
      s1 (str, optional)  first central EEG channel (in two-channel mode)
      s2 (str, optional)  second central EEG channel (in two-channel mode)

      path (str,default = lp.resources.POPS_PATH )    path to POPS training model folder
      lib (str,default = lp.resources.POPS_LIB )      library name of POPS training model

      do_edger (bool, default=True)    perform EDGER cleaning prior to POPS
      no_filter (bool, default=False)  assume EEG are pre-filtered, if True

      do_reref (bool, default=False)   perform re-referencing of the EEG before POPS
      m (str, optional)   mastoid reference (in single-channel mode, if do_reref == True )
      m1 (str, optional)  mastoid reference for s1 (in two-channel mode, if do_reref == True )
      m2 (str, optional)  mastoid reference for s2 (in two-channel mode, if do_reref == True )
      lights_off (str, default='.')   lights-off clock time/marker passed to POPS
      lights_on (str, default='.')    lights-on clock time/marker passed to POPS
      ignore_obs (bool, default=False) ignore observation-level issues
      args (str, default='')          extra POPS arguments

    Returns:
      this command populates the project results cache with POPS results
      explicitly returns proj.table( 'POPS' , 'E' ) 

    Example:
      # single channel example, all defaults 
      proj.pops( 'EEG' )
      # two-channel, without filtering , and applying mastoid references 
      proj.pops( s1='C3', m1='M2', s2='C4',m2='M1', no_filter=True )
```

The default locations of `lp.resources.POPS_PATH` and
`lp.resources.POPS_LIB` are set to be appropriate for the
remnrem/lunapi docker image,
i.e. `/build/nsrr/common/resources/pops/`.  If you are running `lunapi` outside of
the Docker context, you will need to edit these prior to running `pops()`, e.g.:
```
lp.resources.POPS_PATH = '/home/george/data/pops/'
proj.pops( s='C3_M2' )
```

Currently, the default (and only) model is `s2`; more models should be added soon.

More than two channels can be used (as _equivalence channels_) by
running [`POPS`](../ref/pops.md#pops-prediction) via `proj.proc()` directly.  See the main Luna pages for details on POPS.


### proj.predict_SUN2019()

_A project-level wrapper for the SUN2019 biological age model_

```
 predict_SUN2019( cen , th = '3' , path = None )

    Args:
      cen (str or list[str])   comma-delimited list of central EEGs
      th (float, default=3)    SD threshold for missing-value imputation
      path (str, default=None) path to resources for this model  

    Returns:
      populates the project results cache with results

    Example:
      proj.predict_SUN2019( 'C3,C4' )
```

`resources.MODEL_PATH` is set to `/build/luna-models/` (suitable for
the `remnrem/lunapi` Docker image). If `path` is `None`, then it is
set to `resources.MODEL_PATH` instead (i.e. this is the default).

For consistency across different lunapi commands, future releases will
allow Python lists as well as comma-delimited strings: i.e. `cen = [
'C3' , 'C4' ]` as well as `cen = 'C3,C4'`.

See the main Luna pages for details on the Sun et al (2019) model, and on
the [PREDICT](http://zzz.nyspi.org/luna/ref/predict/) command in general.


### proj.silence()

_Turns off the console/log echoing_

```
 silence( b = True , verbose = False )

    Args:
      b (bool, True by default)         silence console output
      verbose (bool, False by default) report action to console

    Returns:
      nothing

    Example:
      proj.silence()         # turn off logging
      proj.silence( False )  # turn it back on
```


### proj.is_silenced()

_Reports on whether the console/log is silenced_

```
 is_silenced( b = True )

    Args:
      b (bool, optional)  ignored; retained for API compatibility

    Returns:
      True if console/log is silenced, else False

    Example:
      proj.silence()         # turn off logging
      proj.is_silenced()
```

### proj.flush()

_Flush the cached output buffer_

```
 flush()

    Args:
      none

    Returns:
      nothing
```

### proj.import_db()

_Imports a prior Luna output database from a file_

```
 import_db(f,s=None)

    Args:
      f (str)                    filename of luna database
      s (set of str, optional)   set of individual IDs to include 

    Returns:
      a list of str, of the individual ID(s) imported
      
    Example:
      proj.import_db( 'out.db' )
      proj.import_db( 'out.db' , s = { 'nsrr01', 'nsrr03' } ) 
      proj.strata()
```

Note that `lunapi` does not create `destrat`-style databases - these are only generated
by the `luna` command-line program.   Any imported data are placed within the _results cache_ of
the `proj` class, and therefore accessible via `proj.strata()`, `proj.table()`, etc.

## Instances

### inst.attach_edf()

_Attach an EDF_

```
 attach_edf( f )

    Args:
      f (str)  EDF filename

    Returns:
      True if attachment was a success, else False

    Example:
      p = proj.inst( 'id1' ) 
      p.attach_edf( '/path/to/id1.edf' )
```

You cannot attach a new EDF to an existing instance.   If you want to reuse the same variable, you must first `del` the instance: e.g. 
```
      p.attach_edf( 'f1.edf' )
      del p
      p.attach_edf( 'f2.edf' )
```


### inst.attach_annot()

_Attach an annotation file_

```
 attach_annot( annot )

    Args:
      annot (str)  annotation filename

    Returns:
      True if attachment was a success,	else False

    Example:
      p.attach_annot( '/path/to/id1.annot' )
```


### inst.refresh()

_Drop any changes made to the attached data_

```
 refresh()

    Args:
      none

    Returns:
      nothing

    Example:
      p.refresh()
```

This drops and then reattaches any currently attached EDFs and
annotation files: i.e. all internal modifications to the attached
instance are cleared.

### inst.headers()

_Returns a dataframe of channel-header information_

```
 headers()

    Args:
      none

    Returns:
      a Pandas dataframe of channel header information

    Example:
      p.headers()
```

### inst.id()

_Return the current instance identifier_

```
 id()

    Args:
      none

    Returns:
      the current instance ID
```

### inst.desc()

_Return a descriptive summary of the attached record_

```
 desc()

    Args:
      none

    Returns:
      a dataframe showing ID, timing, duration, signal counts, annotation counts, and signals
```

### inst.annots()

_Returns a list of current annotation classes_

```
 annots()

    Args:
      none

    Returns:
      a Pandas dataframe of annotation classes

    Example:
      p.annots()
```


### inst.stat()

_Returns information on the attached data_

```
 stat()

    Args:
      none

    Returns:
      a Pandas dataframe of summary values (see below)

    Example:
      p.stat()
```

The key values that are returned are described in the table below:

| Variable | Description |
|----|----|
| `id` | EDF ID |
| `edf_file` | EDF filepath |
| `annotation_files` | Annotation filepath(s) |
| `ns` | Current number of signals |
| `nt` | Original, total number of signals in EDF header |
| `na` | Number of annotation classes |
| `duration` | EDF duration (hh:mm:ss) |
| `ne` | Number of epochs (if epoched) |
| `nem` | Number of masked epochs (if epoched ) |
| `elen` | Epoch duration (for standard, not _generic_, epochs) | 
| `state` | Status flag: 1=attached, 0=empty, -1=problem |


### inst.channels()

_Returns a list of current channels_

```
 channels()

    Args:
      none

    Returns:
      a Pandas dataframe of channels

    Example:
      p.channels()
```

This function can also be called as `chs()` instead of `channels()`.

### inst.chs()

_Alias for `inst.channels()`_

```
 chs()
```


### inst.has_channels()

_Returns a list of true/false values for whether certain channels exist_

```
 has_channels(ch)

    Args:
      ch (str or list(str))   one or more channel labels to query

    Returns:
      a list of boolean values

    Example:
      p.has_channels( 'C3' )
      p.has_channels( [ 'C3' , 'C4' , 'EMG' ] )

```

This function uses Luna channel aliasing when matching: i.e. if
alternate labels have been specified, etc, when determining a match.
Also, matches are case-insensitive.

### inst.has()

_Alias for `inst.has_channels()`_

```
 has(ch)
```


### inst.has_annots()

_Returns a list of true/false values for the presence of certain annotation classes_

```
 has_annots(anns)

    Args:
      anns (str or list(str))   one or more annotation class labels

    Returns:
      a list of boolean values

    Example:
      p.has_annots('N2')
      p.has_annots( [ 'N2','N3','R','REM','W','wake' ] )

```
Note that matching is case-sensitive and must be exact.

### inst.has_annot()

_Alias for `inst.has_annots()`_

```
 has_annot(anns)
```

### inst.fetch_annots()

_Return annotation events as a dataframe_

```
 fetch_annots( anns , interp = -1 )

    Args:
      anns (str or list[str])         one or more annotation class labels
      interp (float, optional)        interpolation value for sample-level expansion

    Returns:
      a dataframe with `Class`, `Start`, and `Stop` columns
```

### inst.fetch_fulls_annots()

_Return full annotation events as a dataframe_

```
 fetch_fulls_annots( anns , add_keys = False )

    Args:
      anns (str or list[str])        one or more annotation class labels
      add_keys (bool, optional)      include encoded annotation keys in the returned table

    Returns:
      a dataframe with `Class`, `Instance`, `Channel`, `Meta`, `Start`, and `Stop` columns
```


### inst.stages()

_Returns a list of current sleep stages_

```
 stages()

    Args:
      none

    Returns:
      a list of sleep stages

    Example:
      ss = p.stages()
```

This is a wrapper around `inst.eval( 'STAGE' )`.


### inst.has_staging()

_Returns true/false for whether stage annotations are available_

```
 has_staging()

    Args:
      none

    Returns:
      boolean 

    Example:
      p.has_staging()
```

This tests whether annotations exist that can be mapped to Luna's generic forms (i.e. `N1`, `N2`, `N3`, `R`, `W`, `L` and `?`);  also, valid staging requires that not all epochs have unknown (`?`) stage assignments.


### inst.eval()

_Evaluate arbitrary Luna commands_

```
 eval( cmdstr )

    Args:
      cmdstr (str)   a Luna command script 

    Returns:
      inst.strata()

    Examples:
      p.eval( 'HEADERS' )
      p.eval( lp.cmdfile( 'cmd/s1.txt' ) )

```

This populates the internal _results cache_ which can be queried with `strata()`, `table()`, etc.

To turn off console logging use `proj.silence()`. Alternatively, use `inst.silent_proc()`, which is similar to `proc()` but suppresses console output.

### inst.eval_dummy()

_Evaluate commands in dummy mode and return backend log text_

```
 eval_dummy( cmdstr )

    Args:
      cmdstr (str)  a Luna command script

    Returns:
      backend status/log text
```

### inst.eval_lunascope()

_Evaluate commands for the LunaScope viewer and return backend log text_

```
 eval_lunascope( cmdstr )

    Args:
      cmdstr (str)  a Luna command script

    Returns:
      backend status/log text
```

### inst.strata()

_Reports on the contents of the current result cache_

```
 strata()

     Args:
      none

    Returns:
      a Pandas dataframe of command/stratum pairs from the last command

    Examples:
      res = p.proc( 'HEADERS' )
      p.strata()
```

This is returned by `eval()`.


### inst.table()

_Displays a table from the current result cache_

```
 table( cmd , strata = 'BL' )

     Args:
      cmd (str)                command name
      strata (str, optional)   if missing, defaults to BL (baseline)

    Returns:
      a Pandas dataframe of the results table

    Examples:
      res = p.proc( 'HEADERS' )
      p.table( 'HEADERS' , 'CH' )
```

### inst.variables()

_Return the variables for a specific command/strata pair_

```
 variables( cmd , strata = 'BL' )

    Args:
      cmd (str)                command name
      strata (str, optional)   stratum label, defaulting to `BL`

    Returns:
      a list of variable names for that output table
```



### inst.proc()

_Evaluate arbitrary Luna commands and return a ProcResult_

```
 proc( cmdstr )

    Args:
      cmdstr (str)   a Luna command script

    Returns:
      ProcResult

    Examples:
      res = p.proc( 'HEADERS' )
      res.table( 'HEADERS' )
      p.table( 'HEADERS' )     # equivalent: query the instance cache directly

```

Like `eval()`, this populates the instance results cache.  The returned `ProcResult` is a
lightweight wrapper that delegates all table queries back to the instance cache — results can
be accessed either through the returned object or directly via `p.strata()`, `p.table()`, etc.

See the note on [live views and `.copy()`](#procresult-live-view) in the project `proc()` section — the same behaviour applies at the instance level.

To turn off console logging use `proj.silence()`. Alternatively, use `inst.silent_proc()`, which is similar to `proc()` but suppresses console output.

### inst.silent_proc()

_Evaluate Luna commands silently_

```
 silent_proc( cmdstr )

    Args:
      cmdstr (str)  a Luna command script

    Returns:
      ProcResult
```

Identical to `inst.proc()` but suppresses console/log output for the duration of the call.
Results are accessible via the returned `ProcResult` or directly via `p.strata()`, `p.table()`, etc.

### inst.silent_proc_lunascope()

_Internal silent evaluation helper for LunaScope_

```
 silent_proc_lunascope( cmdstr )
```


### inst.empty_result_set()

_Indicates whether the results cache is currently empty_

```
 empty_result_set()

     Args:
      none

    Returns:
      True if the results cache is empty

    Examples:
      res = p.proc( 'HEADERS' )
      if p.empty_result_set(): print( 'no results' )
```

The result cache may be empty if the previous command failed.

### inst.clear_vars()

_Clear one, some, or all individual variables_

```
 clear_vars( keys = None )

    Args:
      keys (str, list[str], or set[str], optional)  variable(s) to clear

    Returns:
      nothing

    Example:
      p.clear_vars( 'x' )
      p.clear_vars( [ 'a' , 'b' ] )
      p.clear_vars()
```

### inst.var()

_Set or get an individual variable_

```
 var( key = None , value = None )

    Args:
      key (str or dict, optional)   variable name or dictionary of key/value pairs
      value (str, optional)         value, if setting a single variable

    Returns:
      all variables if `key` is `None`, a single value if getting one variable, or nothing if setting

    Example:
      p.var( 'x' , 22 )
      p.var( { 'a': 22 , 'b': 23 , 'c': 'abc' } )
      p.var( 'x' )
```

### inst.vars()

_Return or set individual variables_

```
 vars( key = None , value = None )

    Args:
      key (str or dict, optional)   variable name or dictionary of key/value pairs
      value (str, optional)         value, if setting a single variable

    Returns:
      a dictionary of current individual variables if `key` is omitted
```

This may include automatically set individual variables such as `${eeg}` based on channel labels.

### inst.data()

_Returns an array of signal/annotation data_

```
 data( chs , annots = None , time = False )

     Args:
      chs (str or list[str] )               channel label(s) 
      annots (str or list[str], optional)   one or more annotation class labels
      time (optional, default = False)      add a time column to the output (seconds)

     Returns:
      a tuple of i) list[str] of labels, ii) numpy array of samples by signals

    Examples:
      d = p.data( ['EEG'] , ['N2'] , time = True)

```

All signals must have the same sample rate, as this returns a regular, i.e. rectangular array.

The presence/absence of annotations is represented as a 0/1 variable with the same sample rate.

### inst.slice()

_Returns an array of merged signal/annotation data based on selected intervals (slices)_

```
 slice( intervals, chs , annots = None , time = False )

     Args:
      intervals                            list of (start,stop)-tuples in time-points
      chs (str or list[str] )              channel label(s)
      annots (str or list[str], optional)  one or more annotation class labels
      time (optional, default = False)     add a time column to the output (seconds)

     Returns:
      a tuple of i) list[str] of labels, ii) numpy array of samples by signals

    Examples:
      d = p.slices( p.e2i( range(2,3) ), ['EEG'] , ['N2'] , time = True)

```

Intervals can be defined using convenience functions `inst.e2i()` and `inst.s2i()`
to convert epochs or seconds to _time-points_ (i.e. 1 time-point is 1e-9 seconds.

Unlike `inst.slices()`, this function merges all requested samples into a single return array.

### inst.slices()

_Returns an array of individual signal/annotation data based on selected intervals (slices)_

```
 slices( intervals, chs , annots = None , time = False )

     Args:
      intervals                            list of (start,stop)-tuples in time-points
      chs (str or list[str] )              channel label(s)
      annots (str or list[str], optional)  one or more annotation class labels
      time (optional, default = False)     add a time column to the output (seconds)

     Returns:
      a list of tuples: i) list[str] of labels, ii) numpy array of samples by signals

    Examples:
      d = p.slices( p.e2i( range(2,3) ), ['EEG'] , ['N2'] , time = True)

```

Intervals can be defined using convenience functions `inst.e2i()` and `inst.s2i()`
to convert epochs or seconds to _time-points_ (i.e. 1 time-point is 1e-9 seconds.

Unlike `inst.slice()`, this function does not merge all requested samples into a single return array,
but returns a list where each element is one interval from the `intervals` input list.

### inst.e2i()

_Helper function to convert epochs to intervals_

```
 e2i( epochs )

     Args:
      epochs    int or list[int] of base-1 epochs

     Returns:
      a list of (start,stop)-tuples (time-points)

    Examples:
      p.e2i( range(1,5) )

```

Intervals are in _time-points_; 1 time-point is 1e-9 seconds.  This is a member function of `inst` as it supports generic epochs
(i.e. epochs need not be 30-second fixed intervals)

### inst.s2i()

_Helper function to convert seconds to intervals_

```
 s2i( secs )

     Args:
      secs      list of (start,stop) float tuples (seconds)

     Returns:
      a list of (start,stop)-tuples (time-points)

    Examples:
      p.s2i( range(1,5) )

```

Intervals are in _time-points_;	1 time-point is	1e-9 seconds.

This is a member function of `inst` to provide a similar interface to `inst.e2i()` -
it is a simple conversion from seconds to time-points that does not use `inst` member data at all.

### inst.mask()

_Apply one or more Luna mask expressions and rebuild epochs_

```
 mask( f = None )

    Args:
      f (str or list[str], optional)  one or more mask expressions/files to apply

    Returns:
      nothing
```

This runs [`MASK`](../ref/masks.md#mask) for each supplied expression and then issues `RE` to rebuild epochs.

### inst.segments()

_Run [`SEGMENTS`](../ref/outputs.md#segments) and return the `SEGMENTS: SEG` table_

```
 segments()

    Args:
      none

    Returns:
      the `SEGMENTS: SEG` dataframe
```

### inst.epoch()

_Run [`EPOCH`](../ref/epochs.md#epoch) with optional arguments_

```
 epoch( f = '' )

    Args:
      f (str, optional)  additional `EPOCH` arguments

    Returns:
      nothing
```

### inst.epochs()

_Return a compact epoch summary dataframe_

```
 epochs()

    Args:
      none

    Returns:
      an `EPOCH: E` dataframe restricted to `E`, `E1`, `LABEL`, `HMS`, `START`, `STOP`, and `DUR`
```

### inst.insert_signal()

_Inserts a new signal into the in-memory EDF_

```
 insert_signal( label , data , sr )

     Args:
      label (str)         new channel label (must be unique to EDF)
      data (list[float])  signal data
      sr (int)            sample rate

     Returns:
      nothing

    Examples:
      p.insert_signal( 'P1' , x , sr = 200 )

```

This returns an error if the size of the signal does not match the EDF exactly.  This is
based on the _current_ size of the in-memory representation, i.e. which may differ from
the on-disk file.

Also, the `label` must not already exist in the EDF.   

### inst.update_signal()

_Updates an existing signal in the in-memory EDF_

```
 update_signal( label , data )

     Args:
      label (str)         existing channel label
      data (list[float])  signal data

     Returns:
      nothing

    Examples:
      p.update_signal( 'P1' , x )

```

This returns an	error if the size of the signal	does not match the EDF exactly.	 This is
based on the _current_ size of the in-memory representation, i.e. which	may differ from	
the on-disk file.

`label` must already exist in the EDF.


### inst.insert_annot()

_Insert/append annotation events_

```
 insert_annot( label , intervals, durcol2 = False )

     Args:
      label (str)                            annotation class 
      intervals (list[(start,stop)-tuples])  events (secs)
      durcol2 (bool, optional )              if True, 2nd col. is duration, not stop     

     Returns:
      nothing

    Examples:
      p.insert_annot( 'A1' , [ (10,12) , (45,46.5) ] )

```

Note, unlike `slice()` and `slices()`, intervals are input in _seconds_ here, not time-points.

If the annotation class already exists, events are appended; otherwise, a new class is created.

### inst.freeze()

_Persist the current timeline mask to a freezer tag_

```
 freeze( f )

    Args:
      f (str)  freezer tag name

    Returns:
      nothing
```

### inst.thaw()

_Restore a previously saved freezer tag_

```
 thaw( f , remove = False )

    Args:
      f (str)                 freezer tag name
      remove (bool, optional) if `True`, remove the tag after thawing

    Returns:
      nothing
```

### inst.empty_freezer()

_Clear all persisted freezer tags for this instance_

```
 empty_freezer()

    Args:
      none

    Returns:
      nothing
```

### inst.pops()

_Run the POPS stager_

```
 pops( s = None, s1 = None , s2 = None,
       path = None , lib = None ,
       do_edger = True ,
       no_filter = False ,
       do_reref = False ,
       m = None , m1 = None , m2 = None ,
       lights_off = '.' , lights_on = '.' ,
       ignore_obs = False ,
       args = '' )

    Args:
      s (str, optional)   central EEG channel (single-channel mode)
      s1 (str, optional)  first central EEG channel (two-channel mode)
      s2 (str, optional)  second central EEG channel (two-channel mode)
      path (str, optional) path to POPS resources
      lib (str, optional)  POPS library name
      do_edger (bool)      perform EDGER cleaning before POPS
      no_filter (bool)     assume EEGs are already filtered
      do_reref (bool)      apply mastoid rereferencing
      m, m1, m2 (str, optional)  mastoid references
      lights_off, lights_on (str) optional light markers
      ignore_obs (bool)    ignore observation-level issues
      args (str)           extra POPS arguments

    Returns:
      populates the instance results cache and returns the POPS output table

    Example:
      p.pops( 'EEG' )
      p.pops( s1='C3', m1='M2', s2='C4', m2='M1', no_filter=True )
```

This is the `inst`-analog of `proj.pops()` (i.e. fits to a
single individual rather than all individuals in the `proj`
sample-list).

The default locations of `lp.resources.POPS_PATH` and
`lp.resources.POPS_LIB` are set to be appropriate for the
remnrem/lunapi docker image,
i.e. `/build/nsrr/common/resources/pops/`.  If you are running `lunapi` outside of
the Docker context, you will need to edit these prior to running `pops()`, e.g.:
```
lp.resources.POPS_PATH = '/home/george/data/pops/'
p.pops( s='C3_M2' )
```

Currently, the default (and only) model is `s2`; more models should be added soon.

More than two channels can be used (as _equivalence channels_) by
running [`POPS`](../ref/pops.md#pops-prediction) via `p.eval()` directly.  See the main Luna pages for details on POPS.

### inst.predict_SUN2019()

_Fit Sun et al (2019) brain-age prediction model_

```
 predict_SUN2019( cen , age = None , th = '3' , path = None )

    Args:
      cen (str or list[str])   comma-delimited list of central EEGs
      age (int or float, optional) chronological age for this individual
      th (float, default=3)    SD threshold for missing-value imputation
      path (str, default=None) path to resources for this model  

    Returns:
      populates the instance results cache with results

    Example:
      p.predict_SUN2019( 'C3,C4', age = 55 )
```

This is the `inst`-analog of `proj.predict_SUN2019()` (i.e. fits to a
single individual rather than all individuals in the `proj`
sample-list).

`resources.MODEL_PATH` is set to `/build/luna-models/` (suitable for
the `remnrem/lunapi` Docker image). If `path` is `None`, then it is
set to `resources.MODEL_PATH` instead (i.e. this is the default).

Channels (`cen`) can be specified as either Python lists as well as
comma-delimited strings: i.e. `cen = [ 'C3' , 'C4' ]` as well as `cen
= 'C3,C4'`.

See the main Luna pages for details on the Sun et al (2019) model, and on
the [PREDICT](http://zzz.nyspi.org/luna/ref/predict/) command in general.

### inst.hypno()

_Plot a hypnogram given sleep stage data_

```
 hypno()

    Args:
      none

    Returns:
      a hypnogram image 

    Example:
      p.hypno()
```

This assumes that valid staging annotations exist for the attached individual (as can be queried by `inst.has_staging()`).  This function works by calling the project-level `lp.hypno()` function, and is equivalent to calling `lp.hypno( p.stages() )`


## lp.hypno()

_Plot a hypnogram given sleep stage data_

```
 hypno( ss , e = None , xsize = 20 , ysize = 2 , title = None )

    Args:
      ss (list[str])      sleep stage annotations (per-epoch)
      e (list[float])     optional epoch times (seconds)
      xsize (float)       size of figure (horiz.)
      ysize(float)        size of figure (vert.)
      title (str)         plot title

    Returns:
      a hypnogram plot

    Example:
      lp.hypno( p.stages() )

```

Stage labels are assumed to be `N1`, `N2`, `N3`, `R`, `W`, `?` and `L`.   This function exists as well as `inst.hypno()` as it allows for different sets of stages to be plotted (e.g. from manual versus automated staging), as the caller has to supply the stage data directly, unlike
`inst.hypno()` (see above).


### lp.hypno_density()

_Make a hypno-density (posterior stage probabilities)_

```
 hypno_density( probs , e = None , xsize = 20 , ysize = 2 , title = None )

    Args:
      probs (array[float])  posterior probabilities
      e (list[float])       optional epoch times (seconds)
      xsize (float)         size of figure (horiz.)
      ysize(float)          size if figure (vert.)
      title (str)           plot title

    Returns:
      a hypno-density plot

    Example:
      p.pops( 'EEG' )
      stgs = p.table( 'POPS' , 'E' )
      lp.hypno_density( stgs )

```

This function expects columns `PP_N1`, `PP_N2`, etc, as returned by
[`POPS`](../ref/pops.md#pops-prediction) and [`SOAP`](../ref/soap.md#soap) commands ( in the output stratified by `E` (epoch), 
as above).

### inst.psd()

_Calculate and plot a PSD curve_

```
 psd( ch, var = 'PSD', minf = None, maxf = 25, minp = None, maxp = None , xlines = None , ylines = None )

    Args:
      ch (str)         a channel label
      var (str, optional)      power variable to plot (default 'PSD')
      minf (float, optional)   minimum frequency (x-axis)
      maxf (float, optional)   maximum frequency (x-axis)
      minp (float, optional)   minimum power (y-axis)
      maxp (float, optional)   maximum power (y-axis)
      xlines (float)           one or more x-axis lines 
      ylines (float)           one or more y-axis lines 
      
    Returns:
      a PSD plot

    Example:
      p.psd( 'C4' )

```


### inst.spec()

_Calculate and plot a spectrogram heatmap_

```
 spec( ch, mine = None , maxe = None , minf = None, maxf = None, w = 0.025 )

    Args:
      ch (str)         a channel label
      mine (int, optional)   minimum epoch number (x-axis)
      maxe (int, optional)   maximum epoch number (x-axis)
      minf (float, optional) minimum frequency (y-axis)
      maxf (float, optional) maximum frequency (y-axis)
      w (float, optional)    winsorsize threshold (0-0.4)

    Returns:
      a spectrogram plot

    Example:
      p.spec( 'EEG' )

```

This is a wrapper to call the Luna [`PSD`](../ref/power-spectra.md#psd) command and the `lp.spec()` to plot a spectrogram of results.  Currently, it uses 
the Welch method to generate a spectrogram.

### inst.tfview()

_Generate an MTM spectrogram view for a selected interval_

```
 tfview( ch , e = None , t = None , a = None ,
         tw = 2 , sec = 2 , inc = 0.1 ,
         f = ( 0.5 , 30 ) , winsor = 0.025 ,
         anns = None , norm = None ,
         traces = True ,
         xlines = None , ylines = None ,
         silent = True , pal = 'turbo' )

    Args:
      ch (str)                 main channel
      e (int or list[int], optional)   one epoch or an epoch range
      t (list[float], optional)        start/stop times in seconds
      a (optional)              reserved for future use
      tw (float)                MTM time-bandwidth parameter
      sec (float)               segment length in seconds
      inc (float)               segment increment in seconds
      f (tuple[float,float])    frequency range
      winsor (float)            winsorization fraction for power values
      anns (list[str], optional) annotations to overlay
      norm (str, optional)      normalization mode
      traces (bool)             whether to draw the raw trace above the spectrogram
      xlines, ylines            optional guide lines
      silent (bool)             suppress console output from the underlying Luna call
      pal (str)                 matplotlib palette name

    Returns:
      an MTM spectrogram plot
```

This helper runs [`MTM`](../ref/power-spectra.md#mtm) internally over the selected interval, extracts the `CH_F_SEG` and `CH_SEG` tables, and then plots the result.

### lp.spec()

_Plot a spectrogram heatmap given prior spectral resuts_

```
 spec(df , ch = None , var = 'PSD', mine = None , maxe = None , minf = None, maxf = None, w = 0.025 )

    Args:
      df (dataframe)         CH/E/F-level output from PSD or MTM
      ch (str, optional)     select output for this channel only
      var (str, optional)    power variable (default 'PSD')
      mine (int, optional)   minimum epoch number (x-axis)
      maxe (int, optional)   maximum epoch number (x-axis)
      minf (float, optional) minimum frequency (y-axis)
      maxf (float, optional) maximum frequency (y-axis)
      w (float, optional)    winsorsize threshold (0-0.4)
      
    Returns:
      a spectrogram

    Example:
      p.eval( 'PSD sig=EEG dB epoch-spectrum' )
      df = p.table( 'PSD' , 'CH_E_F' )
      lp.spec( df )
```


### lp.topo_heat()

_Make a topo-plot_

```
 topo_heat(chs, z,
           ths = None, th = 0.05,
           topo = None, lmts = None,
           sz = 70, colormap = 'bwr', title = '',
           rimcolor = 'black', lab = 'dB')

    Args:
      chs (list[str])      channel labels (per channel)
      z (list[float])      z-values to plot (per channel)
      ths (list[float])    optional, threshold values (per channel) 
      th (float)           threshold (default 0.05)
      topo (dataframe)     alternative channel locations
      lmts (tuple, optional) color scale limits
      sz (float)           point size
      colormap (str)       optional, default color-map (default: blue,white,red)
      title (str)          optional title
      rimcolor (str)       optional, color of rim (above threshold channels)
      lab (str)            optional, label (default 'dB')

    Returns:
      a topo-plot

    Example:
      p.eval( 'PSD sig=${eeg} dB ' )
      df = p.table( 'PSD' , 'B_CH' )
      df = p.table( 'PSD' , 'B_CH' )
      df_sigma = df[ df[ 'B' ] == "SIGMA" ] 
      lp.topo_heat( df['CH'] , df['PSD' ] , title = 'Sigma', sz=200 )
```

The default channel locations are from `lp.default_xy()`.  You can
specify your own: see the format of `lp.default_xy()` output for
details (i.e. a Pandas dataframe with three columns: `CH`, with 2D
Cartesian coordinates `X` and `Y` scaled between -0.5 and +0.5,
i.e. head is a unit circle).

If `ths` is not `None`, then channels with a value for `ths` _below_
`th` (which is by default 0.05) are marked with a thicker rim (i.e. to
indicate significance).


### lp.cmdfile()

_Loads and parses a Luna command file_

`cmdfile( f )`

Unlike the command-line helper, the generated API docs define `cmdfile()` as reading the file contents verbatim so that multi-line control statements are preserved.

### Parameter Files

_Reads and sets project variables based on a parameter file_

`include( f )`

This is the documented API entry point for reading a parameter file into the current project, analogous to using `@param` on the Luna command line, e.g.
```
luna s.lst @param -o out.db < cmd.txt
```

### lp.fetch_doms()

_Fetch all Luna command domains_

`fetch_doms()`

### lp.fetch_cmds()

_Fetch all commands in a domain_

`fetch_cmds( dom )`

### lp.fetch_params()

_Fetch all parameters for a command_

`fetch_params( cmd )`

### lp.fetch_tbls()

_Fetch all output tables for a command_

`fetch_tbls( cmd )`

### lp.fetch_vars()

_Fetch all variables for a command/table_

`fetch_vars( cmd , tbl )`

### lp.fetch_desc_dom()

_Return the description for a domain_

`fetch_desc_dom( dom )`

### lp.fetch_desc_cmd()

_Return the description for a command_

`fetch_desc_cmd( cmd )`

### lp.fetch_desc_param()

_Return the description for a command parameter_

`fetch_desc_param( cmd , param )`

### lp.fetch_desc_tbl()

_Return the description for a command table_

`fetch_desc_tbl( cmd , tbl )`

### lp.fetch_desc_var()

_Return the description for a command/table variable_

`fetch_desc_var( cmd , tbl , var )`

These helpers mirror Luna's internal command registry and are useful for building interactive tooling or checking command metadata programmatically.

### lp.strata()

_List command/strata pairs from a raw results object_

`strata( ts )`

### lp.table()

_Convert one command/strata pair from a raw results object to a dataframe_

`table( ts , cmd , strata = 'BL' )`

### lp.tables()

_Convert all raw results to dataframes_

`tables( ts )`

### lp.show()

_Display a collection of result tables_

`show( dfs )`

### lp.subset()

_Subset rows and columns of a result table_

`subset( df , ids = None , qry = None , vars = None )`

This can subset by `ID`, by a pandas query string, and by a selected list of columns.

### lp.concat()

_Extract and concatenate matching tables across result collections_

`concat( dfs , tlab , vars = None , add_index = None , ignore_index = True )`

This is useful when a higher-level workflow has produced multiple result dictionaries and you want to stack one named table across them.

### lp.version()

_Return the `lunapi` and Luna versions_

`version()`


---

## Output database reader

`lp.destrat` is a pure-Python, in-process reader for Luna output databases (`.db`
files produced by the `luna` command-line tool with `-o`).  It replicates the functionality
of the `destrat` command-line tool without spawning a subprocess, and supports querying
multiple databases simultaneously via glob patterns.

### lp.destrat()

_Open one or more Luna output databases_

```
 destrat( pattern )

    Args:
      pattern (str or list of str)
          A glob pattern, single file path, or list of paths/patterns.
          Examples: 'out.db', 'results/run-*.db', ['a.db', 'b.db']

    Returns:
      a destrat object

    Raises:
      FileNotFoundError if no files are found

    Example:
      db = lp.destrat('out.db')
      db = lp.destrat('results/run-*.db')
      db = lp.destrat(['run1.db', 'run2.db'])
```

When more than one file is matched, `attaching N databases` is printed on
construction.  All subsequent calls transparently query across all files and
merge the results.

Metadata (factors, strata, variables, individual IDs) is loaded once from each
file at construction time and cached in memory.  This means `tables()`, `vars()`,
and `get()` do not re-open the schema tables on every call.

#### destrat.tables()

_Summary of available command/strata/variable combinations_

```
 tables()

    Args:
      none

    Returns:
      pandas.DataFrame with columns CMD, FACTORS, N_VARS, VARIABLES

    Example:
      db = lp.destrat('out.db')
      db.tables()
```

Each row describes one (command, factor-set) combination present in the database(s).
`FACTORS` is a comma-separated list of row-stratifying factors, e.g. `B,CH`.  The
special factor `E` indicates that the table has per-epoch rows (from the Luna
timepoints table).  Baseline output (no stratification) appears as an empty
`FACTORS` field.

```
   CMD FACTORS  N_VARS                     VARIABLES
   PSD    B,CH       3               MTM,PSD,RSPEC
   PSD   E,B,CH      3               MTM,PSD,RSPEC
  STATS             13       KURT,MAX,MEAN,MIN,RMS,...
```

#### destrat.vars()

_List variables present in the database(s)_

```
 vars( cmd=None )

    Args:
      cmd (str, optional)   filter to a single command; leading '+' or '#' is stripped

    Returns:
      pandas.DataFrame with columns CMD, VAR

    Example:
      db.vars()
      db.vars('PSD')
      db.vars('+PSD')
```

#### destrat.get()

_Extract data and return a tidy DataFrame_

```
 get( cmd, r=None, v=None, ids=None, c=None )

    Args:
      cmd (str)
          Command name.  '+PSD', '#PSD', and 'PSD' are all accepted.

      r (str, list, or dict, optional)
          Row stratifiers — each unique combination of factor levels becomes a
          separate row.  Three equivalent forms are accepted:

            Space-separated string:  r='B CH'
            With level subset:       r='B/ALPHA,SIGMA CH'
            List (all levels):       r=['B', 'CH']
            Dict (explicit levels):  r={'B': ['ALPHA', 'SIGMA'], 'CH': None}

          Use r='E' to add per-epoch rows (values joined from the timepoints
          table).  Factors not in r become implicit — they must still be
          present in the strata for the command to match, but they are not
          expanded into separate rows.

      c (str, list, or dict, optional)
          Column stratifiers — each level combination is pivoted into its own
          set of columns named VAR.FAC_LEVEL (matching destrat's -c behaviour).
          Accepts the same forms as r.  A factor cannot appear in both r and c;
          attempting this raises ValueError.

      v (str or list of str, optional)
          Variable name(s) to return.  A space-separated string is accepted.
          None (default) returns all variables.

      ids (str or list of str, optional)
          Individual IDs to include.  A space-separated string is accepted.
          None (default) returns all individuals.

    Returns:
      pandas.DataFrame.
        Without c: columns are ID, row-factor columns, then variable columns.
        With c:    variable columns are named VAR.FAC_LVL for each col-strata.
        Missing combinations (individuals absent from some databases) yield NaN.

    Raises:
      ValueError if the same factor appears in both r and c

    Examples:
      # baseline (no stratification)
      df = db.get('STATS')

      # all PSD variables, all bands and channels
      df = db.get('+PSD', r=['B', 'CH'])

      # destrat-style string with level subset
      df = db.get('+PSD', r='B/ALPHA,SIGMA CH', v=['PSD'])

      # dict-style
      df = db.get('+PSD', r={'B': ['ALPHA','SIGMA'], 'CH': None}, v='PSD')

      # per-epoch PSD for specific individuals
      df = db.get('+PSD', r='E B CH', v='PSD', ids=['id1', 'id2'])

      # column pivot (wide format, one column per channel)
      df = db.get('+PSD', r='B', c='CH', v='PSD')
      # → columns: ID, B, PSD.CH_C3, PSD.CH_C4, ...
```

**Performance note**: `get()` queries only the rows that match the requested
strata.  It does not scan entire tables and does not round-trip through text
serialisation.  For large databases, this is substantially faster than the
`destrat` command-line tool, which reads and holds all data in memory before
filtering.

**Multiple databases**: glob patterns or lists of files are transparently merged.
Individual IDs present in only some databases produce `NaN` for the variables
from the missing files.  Unlike the `destrat` command-line tool, the `c=` column
pivot works correctly even when querying multiple databases.

#### destrat.files

_List of resolved file paths_

```python
db = lp.destrat('results/run-*.db')
db.files          # ['results/run-1.db', 'results/run-2.db', ...]
len(db)           # number of attached database files
```

---

### lp.list_text_tables()

_List tables available in a Luna text-output folder_

```
 list_text_tables( path, id=None )

    Args:
      path (str or Path)   root folder produced by proc_parallel(out_text=...) or luna -t
      id (str, optional)   individual ID (subdirectory) to inspect; defaults to the first
                           subdirectory in sorted order

    Returns:
      pandas.DataFrame with columns command, strata, file

    Raises:
      FileNotFoundError if the folder or the individual subdirectory does not exist

    Example:
      res = proj.procn('PSD sig=EEG spectrum', workers=4, out_text='out/txt')
      lp.list_text_tables('out/txt')
```

Luna's text output (`-t`) stores each individual's results as tab-delimited files
named `CMD_FACTOR1_FACTOR2.txt` under a per-individual subdirectory.  This function
inspects one subdirectory and returns a summary — useful for discovering what tables
are available before calling `read_text_table()`.

```
    command  strata           file
    HEADERS      BL      HEADERS.txt
    HEADERS      CH   HEADERS_CH.txt
        PSD    B_CH   PSD_B_CH.txt
```

---

### lp.read_text_table()

_Read a concatenated text-output table from a Luna text-output folder_

```
 read_text_table( path, cmd_or_file, factors=None )

    Args:
      path (str or Path)   root folder produced by proc_parallel(out_text=...) or luna -t

      cmd_or_file (str, tuple, or list)
          One of:
            - bare command name:  'HEADERS'              (baseline strata)
            - filename:           'HEADERS_CH.txt'
            - sequence:           ('HEADERS', 'CH')
                                  ['PSD', ['B', 'CH']]

      factors (str or list of str, optional)
          Factor(s) when cmd_or_file is a plain command name.
          Ignored when a filename or sequence is given.

    Returns:
      pandas.DataFrame — all individuals concatenated, with an ID column

    Raises:
      FileNotFoundError if no matching files are found

    Example:
      df = lp.read_text_table('out/txt', 'HEADERS')
      df = lp.read_text_table('out/txt', 'HEADERS', factors='CH')
      df = lp.read_text_table('out/txt', ('PSD', ['B', 'CH']))
      df = lp.read_text_table('out/txt', 'PSD_B_CH.txt')
```

Finds every per-individual file matching the requested command/strata combination
and concatenates them into a single DataFrame.  Factor ordering in the filename is
handled automatically; both `.txt` and `.txt.gz` files are supported.

This is a convenient complement to `proc_parallel(out_text=...)`: run with
`out_text=` to write results to disk without holding everything in memory, then
use `read_text_table()` to load just the table(s) you need.

```python
# Run and write to disk
proj.procn('EPOCH & PSD sig=EEG spectrum dB', workers=16, out_text='out/txt')

# Inspect what was written
lp.list_text_tables('out/txt')

# Load one table
df = lp.read_text_table('out/txt', 'PSD', factors=['B', 'CH'])
```

---

## EDF utilities

These functions wrap the Luna command-line tools for EDF-level operations that are
not directly exposed through the Python engine bindings.  They require that the
`luna` binary is available on the system PATH (or supplied via `luna_bin=`).

### lp.merge_edfs()

_Concatenate EDFs in time (row-bind)_

```
 merge_edfs( files, edf='merged.edf', id='merged',
             slist=None, fixed=False, luna_bin=None )

    Args:
      files (list)       EDF file paths to merge
      edf (str)          output EDF filename (default: 'merged.edf')
      id (str)           EDF record ID for the output file (default: 'merged')
      slist (str)        if given, write a one-row sample-list to this path
      fixed (bool)       if True, ignore file timestamps and concatenate in list order
                         (default: False — sort by embedded start time)
      luna_bin (str)     path to the luna binary; defaults to 'luna' on PATH

    Returns:
      pathlib.Path       path to the written output EDF

    Example:
      lp.merge_edfs(['night1.edf', 'night2.edf'], edf='both_nights.edf', id='subj1')
```

Mirrors `luna --merge`.  If there are gaps between the recordings the output is
written as EDF+D (discontinuous).  Use `fixed=True` to override timestamp ordering
and concatenate in the exact order supplied.

```python
# Merge two nights in timestamp order
out = lp.merge_edfs(['night1.edf', 'night2.edf'], edf='combined.edf', id='p01')

# Force a specific order (ignore timestamps)
out = lp.merge_edfs(['seg1.edf', 'seg2.edf', 'seg3.edf'],
                    edf='full.edf', id='p01', fixed=True)
```

---

### lp.bind_edfs()

_Bind EDFs by adding channels (column-bind)_

```
 bind_edfs( files, edf='merged.edf', id='merged',
            slist=None, luna_bin=None )

    Args:
      files (list)       EDF file paths to bind
      edf (str)          output EDF filename (default: 'merged.edf')
      id (str)           EDF record ID for the output file (default: 'merged')
      slist (str)        if given, write a one-row sample-list to this path
      luna_bin (str)     path to the luna binary; defaults to 'luna' on PATH

    Returns:
      pathlib.Path       path to the written output EDF

    Example:
      lp.bind_edfs(['eeg.edf', 'eog.edf', 'emg.edf'], edf='psg.edf', id='subj1')
```

Mirrors `luna --bind`.  All source EDFs must share the same start time and record
count, but channels may have different sample rates.

```python
# Add EEG, EOG and EMG channels from separate files into one EDF
out = lp.bind_edfs(['eeg.edf', 'eog.edf', 'emg.edf'], edf='psg.edf', id='subj1')
```

---

### lp.overlap()

_Multi-sample annotation overlap / enrichment analysis_

```
 overlap( files, seed, other=None, bg=None, nreps=1000,
          event_perm=False, event_perm_w=None,
          w=None, out=None, luna_bin=None, **kwargs )

    Args:
      files              per-individual annotation files; accepted forms:
                           dict   {'id1': 'id1.annot', 'id2': 'id2.annot', ...}
                           list   [('id1', 'path'), ...] or plain list of file paths
                                  (filename stem used as ID)
                           DataFrame  first col = ID, second = annot file
                           str    path to a Luna sample-list (ID / EDF / annot columns)
                                  or a glob pattern for annotation files
      seed (str or list) annotation class(es) to use as seeds — the events being tested
      other (str or list) annotation class(es) to measure overlap against; defaults to
                          all annotations present other than the seeds
      bg (str or list)   background annotation class(es) defining the regions within
                         which permutations are performed; required unless event_perm=True
      nreps (int)        number of permutations (default: 1000)
      event_perm (bool)  use event-based permutation instead of background-region shuffling
      event_perm_w (float) neighbourhood window in seconds for event permutation (default 5)
      w (float)          window size in seconds for distance-based calculations
      out (str)          path for the output database; if omitted, a temporary file is used
      luna_bin (str)     path to the luna binary; defaults to 'luna' on PATH
      **kwargs           any additional luna --overlap parameters (e.g. edges=5, pileup='T')

    Returns:
      lp.destrat         output database reader; call .tables() and .get() to extract results

    Example:
      db = lp.overlap({'id1': 'id1.annot', 'id2': 'id2.annot'},
                      seed='spindle', bg='NREM', nreps=1000)
      db.tables()
      db.get('OVERLAP', r='SEED')
```

Mirrors `luna --overlap`.  Individual annotation events are pooled across all
supplied individuals into a shared timeline, then tested for non-random overlap with
`seed` annotations by permutation.

Either `bg=` or `event_perm=True` is required.  `bg=` defines the regions within
which seed events are randomly shuffled on each permutation; `event_perm=True`
instead shuffles the seed event positions within a local neighbourhood window.

```python
# Basic enrichment: are spindles co-occurring with slow oscillations more
# than expected under random shuffling within NREM sleep?
db = lp.overlap(
    {'id1': 'id1.annot', 'id2': 'id2.annot', 'id3': 'id3.annot'},
    seed='spindle',
    bg='NREM',
    other='SO',
    nreps=1000,
)
db.tables()
df_seed  = db.get('OVERLAP', r='SEED')    # per-seed-class stats
df_other = db.get('OVERLAP', r='OTHER')   # per-other-class stats

# From a sample list (tab-delimited: ID / EDF / annot)
db = lp.overlap('cohort.lst', seed='spindle', bg='NREM', other='SO')

# Glob of annotation files (filename stem becomes the ID)
db = lp.overlap('annots/*.annot', seed='spindle', bg='NREM')

# Event permutation mode
db = lp.overlap(files_dict, seed='spindle', event_perm=True, event_perm_w=5)

# Save the output database for later
db = lp.overlap(files_dict, seed='spindle', bg='NREM', out='overlap_results.db')
```

Additional Luna `--overlap` parameters can be passed as keyword arguments using
Python underscores or their original hyphenated names:

```python
db = lp.overlap(files, seed='spindle', bg='NREM',
                pileup=True, edges=5, max_shuffle=30)
```

---

## BioData Catalyst

`lunapi` includes a small client for browsing and downloading files from a
Gen3/BioData Catalyst commons. It is intended for obtaining recordings and
their sidecar files for subsequent Luna processing; it does not replace the
commons' own access-control or consent procedures.

### BDCClient

Create a client with the commons endpoint and either a BioData Catalyst
API-key pair or an access token:

```python
from lunapi import BDCClient

bdc = BDCClient(
    endpoint='https://api.sb.biodatacatalyst.nhlbi.nih.gov/v2',
    key_id='YOUR_KEY_ID',
    api_key='YOUR_API_KEY',
)
```

The client authenticates lazily when a request is first made. An existing
token can instead be supplied with `token=`. The main operations are:

```python
bdc.projects()                         # projects visible to the credentials
files = bdc.files('my-project')        # normalized file-index records
groups = bdc.recording_groups('my-project')
                                      # EDFs paired with matching sidecars

edf = groups[0]['recording']
path = bdc.download(edf, 'data')      # preserves relative paths
```

`files()` returns normalized dictionaries with fields including `guid`,
`name`, `path`, `size`, `md5`, `project`, `is_edf`, and `is_sidecar`, while
retaining the source record under `raw`. `recording_groups()` identifies EDF,
EDF.GZ, and EDFZ recordings and pairs them with common annotation/XML/TSV
sidecars. `download()` skips a complete existing file unless `force=True`,
can resume partial downloads when supported by the server, and verifies the
expected size and optional MD5 checksum. A `progress(done, total)` callback
can be supplied for download progress.

The shorthand `lp.bdc` is an alias for `BDCClient`:

```python
client = lp.bdc(endpoint='https://example-commons/v2', token='TOKEN')
```

Keep API keys and access tokens out of notebooks that will be shared. Access
to a project remains subject to the commons' authorization and the relevant
study's data-use requirements.


### lp.scope()

_Initiates the scope viewer for a single instance_ 

See [the scope page](scope.md) for notes on using this tool in practice. [LunaScope](https://zzz.nyspi.org/lunascope/) is a standalone desktop application built on top of _lunapi_ and is generally a better, more full-featured viewer. In contrast, `lp.scope()` is a smaller embedded viewer intended mainly for use inside JupyterLab notebooks. Basic usage is

```
lp.scope( p )
```
where `p` is an _instance_ (i.e. a single recording).

Of the other options above, the most likely to be of immediate use is
`chs`, which restricts the viewer to a subset of channels, e.g. :

```
lp.scope( p , chs = [ 'Fz', 'Cz', 'Pz' , 'Oz' ] ) 
```

This can be useful as scope performs some pre-processing of the
signals prior to creating the viewer window; for large datasets with
many channels and high sample rates, this can take a few seconds or
more, and so if you're only interested in a subset of channels, it can
be useful to add these options (or even for a single channel: e.g. in
the form `lp.scope(p,'Cz')`.

The full set of options are given below (most are internal options
that most users can ignore):

```
 lp.scope(
   p, chs=None,
   bsigs=None, hsigs=None, anns=None,
   stgs=['N1', 'N2', 'N3', 'R', 'W', '?', 'L'],
   stgcols={'N1': 'blue', 'N2': 'blue', 'N3': 'navy', 'R': 'red', 'W': 'green', '?': 'gray', 'L': 'yellow'},
   stgns={'N1': -1, 'N2': -2, 'N3': -3, 'R': 0, 'W': 1, '?': 2, 'L': 2},
   sigcols=None, anncols=None,
   throttle1_sr=100, throttle2_np=5 * 30 * 100,
   summary_mins=30, height=600, annot_height=0.15,
   header_height=0.04, footer_height=0.01 )

   Args:
     p (lp.inst)          lunapi instance   
     chs (list[str])      optional, list of channels to show (default: all)
     bsigs (list[str])    optional, channels to calculate band power for (default, likely EEG)
     hsigs (list[str])    optional, channels to calculate Hjorth parameters for (default, likely EEG)
     anns (list[str])     optional, annotations to show (default:all)
     stgs (list[str])     optional, stage labels (default: `['N1', 'N2', 'N3', 'R', 'W', '?', 'L']`)
     stgcols (dict)       optional, stage colors 
     stgns (dict)         optional, stage y-axis values
     sigcols (dict)       optional, channel:color mappings
     anncols (dict)       optional, annotation:color mappings
     throttle1_sr (int)   optional, sample rate throttle (default: 100 Hz max)
     throttle2_np (int)   optional, number of points throttle (default: 15,000 max)
     summary_mins (int)   optional, duration of summary stats (currently not used)
     height (int)         optional, height of viewer in pixels
     annot_height (float) optional, height of annotations as proportion (default: 0.15)
     header_height (float)optional, height of header as proportion (default: 0.04)
     footer_height (float)optional, height of footer as proportion (default: 0.01)

    Returns:
      an interactive scope viewer widget

    Example:
      proj.sample_list( 's.lst' ) 
      p = proj.inst( 1 )
      lp.scope( p ) 

```    
