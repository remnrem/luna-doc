# _Lunapi_ Reference

This page describes the _high-level_ interface to Luna functions.
[This notebook](https://github.com/remnrem/luna-api-notebooks/blob/74ab1745ac524c7e9ec5ea86bc032b33b32d0644/99_reference.ipynb)
also contains a few further details.  Below, we assume the `lunapi`
package will always be aliased as `lp`, i.e.:

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
functions](#helpers) and d) the [Moonbeam](#moonbeam) utility to pull
NSRR data directly into Python.  Below, we'll first tabulate all commands in the next section,
and then give some more details on usage in the following sections.


## Command tables

Major commands are described in __bold text__.  Below:

 - the _lunapi_ package is aliased as `lp`

 - the `proj` class is assumed to also be called `proj` (i.e. from `proj = lp.proj()`)

 - any `inst` class is assumed to be called `p` (i.e. from `p = proj.inst(x)`)

 - any `moonbeam` class is assumed to be called `mb`
 

### Projects 

Projects are _singleton classes_ that organize Luna functions and objects within a single Python session. 

<h5>Project/instance creation</h5>

| Command | Description |
|----|-----|
| [__`lp.proj()`__](#lpproj) | __Initiate/reference a _lunapi_ project__ |
| [__`proj.inst()`__](#projinst)  | __Create a new instance__ |
| [`proj.empty_inst()`](#projempty_inst) | Create an empty instance | 
| [`proj.retire()`]() | Retire a project | 

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

<h5>Executing Luna commands</h5>

| Command | Description |
|----|-----|
| __[`proj.proc()`](#projproc)__ | __Evaluate Luna commands on all sample-list individuals__ |
| __[`proj.strata()`](#projstrata)__ | __Return a list of command/strata pairs from a prior eval() run__ |
| __[`proj.table()`](#projtable)__ | __Return a table as a dataframe from a prior eval() run__ |
| [`proj.commands()`](#projcommands) | Return a list of commands executed from a prior `proc()` run | 
| [`proj.variables()`](#projvariables) | Return the variables (table header) for a specific command/strata pair from a prior eval() run |
| [`proj.empty_result_set()`](#projempty_result_set) | Indicate whether there are any results in the project results cache |

<h5>Variables</h5>

| Command | Description |
|----|-----|
| __[`proj.var()`](#projvar)__ | __Get/set project-wide variable__ |
| [`proj.varmap()`](#projvarmap) | Get/set project-wide variables (via a `dict()`) |
| [`proj.vars()`](#projvars) | Gets/sets project-wide options/variables |
| [`proj.clear_var()`](#projclear_var) | Clears a project-wide option/variable  |
| [`proj.clear_vars()`](#projclear_vars) | Clears all project-wide options/variables | 
| [`proj.clear_ivars()`](#projclear_ivars) | Clears all individual-specific variables (for all individuals) | 

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
| __[`inst.annots()`](#instannots)__ | Returns a list of current annotation classes |
| __[`inst.channels()`](#instchannels)__ | Returns a list of current channels |
| [`inst.stat()`](#inststat) | Returns information on the attached data |
| [`inst.has_channels()`](#insthas_channels) | Returns a list of true/false values for whether certain channels exist |
| [`inst.has_annots()`](#insthas_annots) | Returns a list of true/false values for the presence of certain annotation classes |
| [`inst.stages()`](#inststages) | Returns a list of current sleep stages |
| [`inst.has_staging()`](#insthas_staging) | Returns true/false for whether stage annotations are available |

<h5>Executing Luna commands</h5>

| Command | Description |
|----|-----|
| __[`inst.eval()`](#insteval)__ | Evaluate arbitrary Luna commands |
| __[`inst.strata()`](#inststrata)__ | Reports on the contents of the current result cache |
| __[`inst.table()`](#insttable)__ | Displays a table from the current result cache |
| [`inst.proc()`](#instproc) | Evaluate arbitrary Luna commands and directly return all results |
| [`inst.empty_result_set()`](#instempty_result_set) | Indicates whether the results cache is currently non-empty |

<h5>Individual-level variables</h5>

| Command | Description |
|----|-----|
| [`inst.ivar()`](#instivar) | Set or get an individual variable |
| [`inst.ivars()`](#instivars) | Returns all individual variables |

<h5>Extracting signals & annotations</h5>

| Command | Description |
|----|-----|
| __[`inst.data()`](#instdata)__ | __Returns an array of signal/annotation data__ |
| [`inst.slice()`](#instslice) | Returns an array of merged signal/annotation data based on selected intervals (slices) |
| [`inst.slices()`](#instslices) | Returns an array of individual signal/annotation data based on selected intervals (slices) |
| [`inst.e2i()`](#inste2i) | Helper function to convert epochs to intervals |
| [`inst.s2i()`](#insts2i) | Helper function to convert epochs to intervals |

<h5>Updating signals & annotations</h5>

| Command | Description |
|----|-----|
| __[`inst.insert_signal()`](#instinsert_signal)__ | __Inserts a new signal into the in-memory EDF__ |
| [`inst.update_signal()`](#instupdate_signal) | Updates an existing signal in the in-memory EDF |
| [`inst.insert_annot()`](#instinsert_annot) | Insert/append annotation events |

<h5>Models</h5>

| Command | Description |
|----|-----|
| [`inst.pops()`](#instpops) | Run POPS stager for a single individual |
| [`inst.predict_SUN2019()`](inst#predict_sun2019) | Fit Sun et al (2019) brain-age prediction model |

<h5>Plotting</h5>

| Command | Description |
|----|-----|
| [`inst.hypno()`](#insthypno) | Plot a hypnogram given sleep stage data | 
| [`lp.hypno_density()`](#lphypno_density) | Make a hypno-density (posterior stage probabilites) | 
| [`inst.psd()`](#instpsd) | Calculate and plot a PSD curve |
| [`inst.spec()`](#instspec) | Calculate and plot a spectrogram heatmap |
| [`lp.topo_heat()`](#insttopo_heat) | Topo-plot |


### Helpers

<h5>Utility functions</h5>

| Command | Description |
|----|-----|
| [`lp.cmdfile()`](#lpcmdfile) | Loads and parses a Luna command file |
| [`lp.include()`](#lpinclude) | Reads and sets project variables based on a parameter file |

### Scope

| Command | Description |
|----|-----|
| __[`lp.scope()`](#lpscope)__ | __Initiate the Scope viewer__ |


### Moonbeam


| Command | Description |
|----|-----|
| __[`mb = lp.moonbeam()`](#lpmoonbeam)__ | __Initiate a Moonbeam object (`mb`)__ |
| __[`mb.cohorts()`](#mbcohorts)__ | __List available cohorts__ |
| __[`mb.cohort()`](#mbcohort)__ | __Set/list current cohort__ |
| __[`mb.inst()`](#mbinst)__ | __Beam/create an individual from the current cohort__ | 
| __[`mb.pheno()`](#mbpheno)__ | __Beam any phenotypes information for an individual__ |
| [`mb.set_cache()`](#mbset_cache) | Explicitly set cache |
| [`mb.cached()`](#mbcached) | Check whether a file is cached |
| [`mb.pull()`](#mbpull) | Pull (all files for) an individual |
| [`mb.pull_file()`](#mbpull_file) | Pull a single file |


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
 proj()

    Args:
      none

    Returns:
      reference to a proj object

    Example:
      proj.retire()

```

This closes an existing _project_ and frees all resources.


### proj.build()

_Traverses folders (recursively) to generate a lunapi sample-list_

```
 build( args )

    Args:
      args (list)  a list of strings, either folders or special options

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
 sample_list(x, path, df )

    Args:
      x (str, optional)     filename of the sample-list
      path (str, optional)  path to prepend to all relative file paths when building the sample-list
      df (boolean)          return a pandas dataframe rather than python list (default=True)

    Returns:
      nothing, if reading a sample-list
      a dataframe/list of (ID,EDF,set(annotation files))-tuples, if x is None

    Example:
      proj.sample_list( '/tutorial/s.lst' )
      proj.sample_list()

```

If the sample list uses relative paths that are not appropriate for your current directory,
you can set the `path` argument to add a prefix to all relative paths in the sample list.  Alternatively 
(and equivalently), you can set the project variable `path` before calling `sample_list(x)` to 
acheve the same result: `proj.var( 'path' , '/path/to/data/' )`.   For example, if the current working folder
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
 nobs()

    Args:
      none

    Returns:
      a table of IDs, filenames and a flag for valid/invalid status

    Example:
      proj.sample_list( 's.lst' )
      proj.validate()
```

This provides the same functionality as the `--validate` option of
Luna, which is described [here](../ref/helpers.md#-validate).


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
      EDF filename (str), if x is a valid index
      NoneType, if x is not a valid index

    Example:
      proj.get_edf(0)

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


### proj.proc()

_Evaluate Luna commands on all sample-list individuals_

```
 eval( cmdstr )

    Args:
      cmdstr (str)  a valid Luna command script 

    Returns:
      dict of command/strata (str) keys to dataframe values

    Example:
      proj.eval( 'HEADERS' )

```

`eval()` also populates the `proj` results cache, i.e. that can be subsequently queried with `proj.strata()`, `proj.table()`, etc.

The `lp.cmdfile()` utility function can be used to pass a file-based Luna script to `eval()` (i.e.
which will strip out comments, etc).

Multi-line scripts can be passed by using triple-quotes, e.g.:
```
proj.eval( """
MASK ifnot=N2
RE
STATS sig=${eeg}
""" )
```

Note that `eval()` can used both project-wide and individual-specific
variables in scripts, e.g. as above (`${eeg}`).

Note: a similar form of this command exists `silent_eval()` which has identical syntax but suppresses any console/log output.


### proj.strata()

_Return a list of command/strata pairs from a prior `eval()` run_

```
 strata()

    Args:
      none

    Returns:
      a dataframe of commands and strata pairs in the project-level results cache

    Example:
      proj.eval( 'HEADERS' )
      proj.strata()

```

The project results cache stores the results of	the last successful run	of `proj.eval()`


### proj.table()

_Return a table as a dataframe from a prior `eval()` run_

```
 table( cmd , strata = 'BL' )

    Args:
      cmd (str)   case-sensitive command name in the project results cache
      strata (str, optional, defaults to BL)  case-sensitive stratum in the project results cache

    Returns:
      a dataframe of values associated with a specific command/strata pair in the project-level results cache

    Example:
      proj.eval( 'HEADERS' )
      proj.table( 'HEADERS' )
      proj.table( 'HEADERS' , 'CH' )

```

The project results cache stores the results of the last successful run of `proj.eval()`.  A list of available tables
is given by `proj.strata()`.  All individuals from the sample-list are combined as different rows of the same results
table.

### proj.commands()

_Return a list of commands executed from a prior `eval()` run_

```
 commands()

    Args:
      none

    Returns:
      a dataframe of commands in the project-level results cache

    Example:
      proj.eval( 'HEADERS' )
      proj.commands()

```

The project results cache stores the results of the last successful run of `proj.eval()`



### proj.variables()

_Return the variables (table header) for a specific command/strata pair from a prior `eval()` run_

```
 table( cmd , strata = 'BL' )

    Args:
      cmd (str)	  case-sensitive command name in the project results cache
      strata (str, optional, defaults to BL)  case-sensitive stratum in	the project results cache

    Returns:
      a dataframe of variable names from the table associated with a specific command/strata pair in the project-level results cache

    Example:
      proj.eval( 'HEADERS' )
      proj.variables( 'HEADERS' )
      proj.variables( 'HEADERS' , 'CH' )

```

This is similar to `proj.table()` but only returns table headers (i.e. variable names).


### proj.empty_result_set()

_Indicate whether there are any results in the project results cache_

```
 proj.empty_result_set()

    Args:
      none

    Returns:
      bool, True if the project results cache is empty
      
    Example:
      proj.eval( 'HEADERS' )
      proj.proj.empty_result_set()

```

The project results cache stores the results of the last successful run of `proj.eval()`.


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

### proj.varmap()
_Sets project-wide options/variables_

```
 varmap(d)

    Args:
      d (dict)      key/value pairs

    Returns:
      nothing, if value is not None (it sets the option)

    Example:
      proj.varmap( { 'path': '/tutorial/' , 'annot-file': '/path/to/a.annot' , 'ch': 'EEG1,EEG2' } )
      proj.vars()
```

Special variables (i.e. above `path` and `annot-file` are enacted rather than set (i.e. only `ch` above will appear in `proj.vars()`).

Also note that some special project-wide variables may be automatically set (e.g. `sleep` equals `N1,N2,N3,R`).

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

### proj.clear_var()

_Clears a project-wide option/variable_

```
 clear_var(key)

    Args:
      key (str)  name of the variable

    Returns:
      nothing

    Example:
      proj.clear_var( 'path' )
```

If the variable does not exist, this has no effect.

### proj.clear_vars()

_Clears all project-wide options/variables_

```
 clear_vars()

    Args:
      none

    Returns:
      nothing

    Example:
      proj.clear_vars()
```

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
as `lunapi` uses the `vars`/`ivars` nomenclature to distinguish betwen project-wide and individual-specific
variables.

### proj.pops()

_A project-level wrapper for the POPS stager_

```
 pops( s = None, s1 = None , s2 = None,
       path = None , lib = None ,
       do_edger = True ,
       no_filter = False ,
       do_reref = False ,
       m = None , m1 = None , m2 = None )

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
running `POPS` via `proj.eval()` directly.  See the main Luna pages for details on POPS.


### proj.predict_SUN2019()

_A project-level wrapper for the SUN2019 biological age model_

```
 predict_SUN2019(  cen , th = '3' , path = None )`

    Args:
      cen (str)                comma-delimited list of central EEGs
      th (float, default=3)    SD threshold for missing-value imputation
      path (str, default=None) path to resources for this model  

    Returns:
      populates the project results cache with results

    Example:
      proj.silence()         # turn off logging
      proj.silence( False )  # turn it back on
```

`resources.MODEL_PATH` is set to `/build/luna-models/` (suitable for
the `remnrem/lunapi` Docker image). If `path` is `None`, then it is
set to `resources.MODEL_PATH` instead (i.e. this is the default).

For consistency across differnt lunapi commands, future releases will
allow Python lists as well as comma-delimited strings: i.e. `cen = [
'C3' , 'C4' ]` as well as `cen = 'C3,C4'`.

See the main Luna pages for details on the Sun et al (2019) model, and on
the [PREDICT](http://zzz.bwh.harvard.edu/luna/ref/predict/) command in general.


### proj.silence()

_Turns off the console/log echoing_

```
 silence( x = True , verbose = False )

    Args:
      x (bool, True by default)         silence console output
      verbose( bool, False by default)  report action to console

    Returns:
      nothing

    Example:
      proj.silence()         # turn off logging
      proj.silence( False )  # turn it back on
```


### proj.is_silenced()

_Reports on whether the console/log is silenced_

```
 is_silenced()

    Args:
      none

    Returns:
      True if console/log is silenced, else False

    Example:
      proj.silence()         # turn off logging
      proj.is_silenced()
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
 attach_annot( f )

    Args:
      f (str)  annotation filename

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

The key values that are returned are described in the total below:

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


### inst.has_channels()

_Returns a list of true/false values for whether certain channels exist_

```
 has_channels(x)

    Args:
      x  (str or list(str))   one or more channel labels to query

    Returns:
      a list of boolean values

    Example:
      p.has_channels( 'C3' )
      p.has_channels( [ 'C3' , 'C4' , 'EMG' ] )

```

This function uses Luna channel aliasing when matching: i.e. if
alternate labels have been specified, etc, when determining a match.
Also, matches are case-insensitive.


### inst.has_annots()

_Returns a list of true/false values for the presence of certain annotation classes_

```
 has_annots(x)

    Args:
      x (str or list(str))   one or more annotation class labels

    Returns:
      a list of boolean values

    Example:
      p.has_annots('N2')
      p.has_annots( [ 'N2','N3','R','REM','W','wake' ] )

```
Note that matching is case-sensitive and must be exact.


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

To turn off console logging use `proj.silence()`.   Alternatively, use `silent_eval()` which is similar to `eval()` but suppresses console output.

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



### inst.proc()

_Evaluate arbitrary Luna commands and directly return all results_

```
 proc( cmdstr )

    Args:
      cmdstr (str)   a Luna command script

    Returns:
      dict of stratum->table pairs

    Examples:
      res = p.proc( 'HEADERS' )
      lp.show( res )

```

This is effectively the same function as `eval()` - it just varies in how it returns results: whereas `eval()` returns a table summary of the results generated (command, factor/level), as generated by `inst.strata()`; in contrast, `proc()` returns an object representing all results directly.  As with `eval()`, this also populates the internal _results cache_. which can be queried with `strata()`, `table()`, etc.

To turn off console logging use `proj.silence()`.  Alternatively, use `silent_proc()` which is similar to `proc()` but suppresses console output.


### inst.empty_result_set()

_Indicates whether the results cache is currently non-empty_

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

The result cache may be empty is the previous command failed.


### inst.ivar()

_Set or get an individual variable_

```
  var( key = None , value = None )

    Args:
      key (str)                 variable name
      value (str, optional)     value (if setting)

    Returns:
      if key is None, returns all variables
      if key is a dict, set key/value pairs
      else, if value is None, returns the value of variable key
      else, sets variable 'key' to 'value'

    Example:
      p.var( 'x' , 22 )
      p.var( { 'a':22 , 'b':23 , 'c': 'abc' } )
      p.var( 'x' )   # returns 22
```

Note, this may return automatically set individual-variables (i.e. `${eeg}` based on channel labels).


### inst.ivars()

_Returns all individual variables_

```
 ivars()

    Args:
      none

    Returns:
      a dict of key/value pairs for individual-variables set

    Example:
      p.ivars()
```

Note, this may include automatically set `ivars` (i.e. `${eeg}` based on channel labels).

### inst.data()

_Returns an array of signal/annotation data_

```
 data(  chs = None , annots = None , time = False )

     Args:
      chs (str or list[str] )               channel label(s) 
      annots (str ot list[str], optional)   one or more annotation class labels
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
 slice( intervals, chs , annots , time = False )

     Args:
      intervals                            list of (start,stop)-tuples in time-points
      chs (str or list[str] )              channel label(s)
      annots (str ot list[str], optional)  one or more annotation class labels
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
 slices( intervals, chs , annots , time = False )

     Args:
      intervals                            list of (start,stop)-tuples in time-points
      chs (str or list[str] )              channel label(s)
      annots (str ot list[str], optional)  one or more annotation class labels
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
      a list of (start,top)-tuples (time-points)

    Examples:
      p.e2i( range(1,5) )

```

Intervals are in _time-points_; 1 time-point is 1e-9 seconds.  This is a member function of `inst` as it supports generic epochs
(i.e. epochs need not be 30-second fixed intervals)

### inst.s2i()

_Helper function to convert epochs to intervals_

```
 e2i( secs )

     Args:
      secs      list of (start,stop) float tuples (seconds)

     Returns:
      a list of (start,top)-tuples (time-points)

    Examples:
      p.s2i( range(1,5) )

```

Intervals are in _time-points_;	1 time-point is	1e-9 seconds.

This is a member function of `inst` to provide a similar interface to `inst.e2i()` -
it is a simple conversion from seconds to time-points that does not use `inst` member data at all.

### inst.insert_signal()

_Inserts a new signal into the in-memory EDF_

```
 insert_signal( label , data , sr )

     Args:
      label (str)         channel label (must be unique to EDF)
      data (list[float])  signal data
      sr (int)            sample rate

     Returns:
      nothing

    Examples:
      p.insert_signal( 'P1' , x , sr = 200 )

```

This returns an error if the size of the signal does not match the EDF exactly.  This is
based on the _current_ size of the in-memroy representation, i.e. which may differ from
the on-disk file.

Also, the `label` must not already exist in the EDF.   

### inst.update_signal()

_Inserts a new signal into the in-memory EDF_

```
 update_signal( label , data )

     Args:
      label (str)         channel label (must be unique to EDF)
      data (list[float])  signal data

     Returns:
      nothing

    Examples:
      p.update_signal( 'P1' , x )

```

This returns an	error if the size of the signal	does not match the EDF exactly.	 This is
based on the _current_ size of the in-memroy representation, i.e. which	may differ from	
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

### inst.pops()

e for s1 (in two-channel mode, if do_reref == True )
      m2 (str, optional)  mastoid reference for s2 (in two-channel mode, if do_reref == True )

    Returns:
          this command populates the project results cache with POPS results
	        explicitly returns proj.table( 'POPS' , 'E' )

    Example:
          # single channel example, all defaults
	        p.pops( 'EEG' )
		      # two-channel, without filtering , and applying mastoid references
		            p.pops( s1='C3', m1='M2', s2='C4',m2='M1', no_filter=True )
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
running `POPS` via `p.eval()` directly.  See the main Luna pages for details on POPS.~

### inst.predict_SUN2019()

_Fit Sun et al (2019) brain-age prediction model_

```
 predict_SUN2019( cen , age = None , th = '3' , path = None )

    Args:
      cen (str or list[str])   comma-delimited list of central EEGs
      th (float, default=3)    SD threshold for missing-value imputation
      path (str, default=None) path to resources for this model  

    Returns:
      populates the project results cache with results

    Example:
      proj.silence()         # turn off logging
      proj.silence( False )  # turn it back on
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
the [PREDICT](http://zzz.bwh.harvard.edu/luna/ref/predict/) command in general.

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
      ysize(float)        size if figure (vert.)
      title (str)         plot title

    Returns:
      a hypnogram plot

    Example:
      lp.hypno( p.stages() )

```

Stage labels are assumed to be `N1`, `N2`, `N3`, `R`, `W`, `?` and `L`.   This function exists as well as `inst.hypno()` as it allows for different sets of stages to be plotted (e.g. from manual versus automated staging), as the caller has to supply the stage data directly, unlike
`inst.hypno()` (see above).


### lp.hypno_density()

_Make a hypno-density (posterior stage probabilites)_

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
`POPS` and `SOAP` commands ( in the output stratifed by `E` (epoch), 
as above).

### inst.psd()

_Calculate and plot a PSD curve_

```
 psd( ch,  minf = None, maxf = None, minp = None, maxp = None , xlines = None , ylines = None )

    Args:
      ch (str)         a channel label
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

This is a wrapper to call the Luna `PSD` command and the `lp.spec()` to plot a spectrogram of results.  Currently, it uses 
the Welch method to generate a spectrogram.

### lp.spec()

_Plot a spectrogram heatmap given prior spectral resuts_

```
 spec(df , ch = None , var = 'PSD', mine = None , maxe = None , minf = None, maxf = None, w = 0.025 )`

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
           ths = None, th=0.05,
           topo = None, lmts= None ,
           sz=70, colormap = "bwr", title = "", 
           rimcolor="black", lab = "dB")

    Args:
      chs (list[str])      channel labels (per channel)
      z (list[float])      z-values to plot (per channel)
      ths (list[float])    optional, threshold values (per channel) 
      th (float)           threshold (default 0.05)
      topo (dataframe)     alternative channel locations
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
Cartesian co-ordinates `X` and `Y` scaled betwee -0.5 and +0.5,
i.e. head is a unit circle).

If `ths` is not `None`, then channels with a value for `ths` _below_
`th` (which is by default 0.05) are marked with a thicker rim (i.e. to
indicate significance).


### lp.cmdfile()

_Loads and parses a Luna command file_

`cmdfile( f )`

### lp.include()

_Reads and sets project variables based on a parameter file_

`include( f )`

N.b. `lp.include( 'path/to/param' )` use the same function that the Luna command-line option `@path/to/param` uses, e.g. 
```
luna s.lst @param -o out.db < cmd.txt
```




### lp.scope()

_Initiates the scope viewer for a single instance_ 

See [the scope page](#scope.md) for notes on using this tool in practice.  Basic usuage is

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
   throttle1_sr=100, throttle2_np=15000,
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
     throttle2_np (int)   optional, number of points throttle (defautl: 15,000 max)
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



### lp.moonbeam()

_Initiate a [Moonbeam](../apps/moonbeam.md) object_

```
mb = lp.moonbeam( token )
```

where `token` is your [NSRR](https://sleepdata.org/) token, which
for NSRR users who are logged into the site is available from <https://sleepdata.org/token>.

!!!warn "Prototyping Moonbeam"
    Please note that the backend server support for Moonbeam is under flux and so this service may be temporarily unavailable:  in future releases we'll try to make it more stable and performant.

### mb.cohorts()

_List available cohorts_

```
mb.cohorts()
```

Returns a Pandas data-table of available cohorts for the logged-in user

### mb.cohort()

_Select a cohort and return available individuals_

For example, to select `cfs` (Cleveland Family Study) is that was listed as an available cohort:

```
mb.cohort( 'cfs' )
```

Returns of Pandas data-frame of individuals (ID and files).

### mb.inst()

_Pull an individual (EDF & annotations) from a selected cohort_

```
p = mb.inst( id )
```

Generates and attaches an instance to `p` (i.e. similar to `p = proj.inst()`) assuming that `id` is a string
that matches an ID in the attached cohort.  If it exists locally (cached), that version will be used; otherwise,
_Moonbeam_ will attempt to download it from the server.

### mb.pheno()

_Pull phenotypic data from the selected individual_

```
mb.pheno()
```

Returns	of Pandas data-frame of	phenoytpes for the last pulled individual.


### mb.set_cache()

_Set the cache folder explicitly_ 


### mb.cached()

_Internal function: check whether data are cached_

### mb.pull()

_Internal function: pull an individual_

### mb.pull_file()

_Interal function: pull a file_



