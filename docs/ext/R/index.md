# lunaR 

_lunaR_ is a package for the [R statistical software
environment](https://www.r-project.org), which aims to provide a
simple interface for the Luna library.  Primary use cases are 1) for
working with individual sleep studies/EDFs, and 2) as a convenient way
to work with [_destrat_](../../luna/args.md#destrat) output databases.
For larger projects, the command-line version of Luna
([_lunaC_](../../luna/args.md)) will likely be a better option.

!!! hint
    To install _lunaR_, see the main [installation pages](../../download/index.md), or try using the
    [Dockerized](../../download/docker.md) version of Luna.


## Overview

This page gives an overview of the core [functions](#functions)
provided by this package, followed by [examples](example.md) and
descriptions of some additional [convenience
functions](#convenience-functions).  All examples on the page use the
[tutorial dataset](../../tut/tut1.md), especially the second individual
(`nsrr02`).  The Luna tutorial also includes [a
section](../../tut/tut4.md) in which all steps are performed in _lunaR_
as well as _lunaC_.

_lunaR_ is simply an interface to the identical C/C++ library that
underlies the _lunaC_ command line tool: as such, _lunaC_ and _lunaR_
are fundamentally similar in most core respects.  The different
interfaces do mean that some aspects of the workflow differ between the
two tools, however.  For example, like _lunaC_, _lunaR_ operates on a
single EDF at a time; unlike the command line tool, however, _lunaR_
is more interactive, in the sense that the modifiable, _in-memory_
representation of the EDF typically persists between different _lunaR_
commands.  We discuss these differences below in more detail.

!!! note 
    In this documentation, we refer to Luna's R interface as
    _lunaR_ to distinguish it from the base Luna library and
    command-line tool, which we're calling _lunaC_.  The actual R package is named `luna`
    however (as a `lunar` package already exists in
    [CRAN](https://cran.r-project.org/)), as is the command line
    you'll actually type (i.e. `luna`) to run _lunaC_.


The _L_-functions (i.e. all starting with the letter `l` for Luna) can
be roughly grouped as follows:

- __attaching an EDF:__ [`lsl()`](#lsl), [`lattach()`](#lattach),
  [`ledf()`](#ledf), [`lstat()`](#lstat), [`ldrop()`](#ldrop),
  [`lrefresh()`](#lrefresh)

- __extracting data:__ [`lchs()`](#lchs), [`ldata()`](#ldata),
  [`ldata.intervals()`](#ldataintervals)  

- __working with annotations:__ [`lannots()`](#lannots),
  [`lepoch()`](#lepoch), [`letable()`](#letable),
  [`lstages()`](#lstages), [`ladd.annot()`](#laddannot),
  [`ladd.annot.file()`](#laddannotfile)

- __running commands:__ [`leval()`](#leval), [`lcmd()`](#lcmd),
  [`leval.project()`](#levalproject), 
  [`literate()`](#literate)

- __working with output:__ [`ldb()`](#ldb), [`lx()`](#lx),
  [`lx2()`](#lx2), [`lid()`](#lid), [`ltxttab()`](#ltxttab)

- __setting variables:__ [`lset()`](#lset), [`lvar()`](#lvar),
  [`lreset()`](#lreset)

- __convenience/misc:__ [`lstgcols()`](#lstgcols), [`lbands()`](#lbands), [`llog()`](#llog),
  [`le2i`](#le2i), [`lsanitize()`](#lsanitize), [`ldenoise()`](#ldenoise), 

- __plotting functions__: [`lheatmap()`](#lheatmap), [`ltopo.heat()`](#ltopo.heat), [`ltopo.rb()`](#ltopo.rb) ,
  [`ltopo.xy()`](#ltopo.xy) ,  [`ltopo.heat2()`](#ltopo.heat2) , [`ltopo.topo()`](#ltopo.topo) ,
   [`ltopo.conn()`](#ltopo.conn) , [`ltopo.dconn()`](#ltopo.dconn) 


The index below gives a listing of all major _lunaR_ functions:

| Function     | Description |
| ----         | ---- |  
| [`lsl()`](ref.md#lsl)           | Loads a sample list | 
| [`lattach()`](ref.md#lattach)   | Attaches an EDF from a sample list |
| [`ledf()`](ref.md#ledf)         | Attaches an EDF directly |
| [`lstat()`](ref.md#lstat)        | Reports on the currently attached EDF |
| [`ldrop()`](ref.md#ldrop)       | Detaches the current EDF |
| [`lrefresh()`](ref.md#lrefresh) | Reverts to the original attached EDF |
| [`lchs()`](ref.md#lchs)         | Returns a vector of channel names for the attached EDF |
| [`ldata()`](ref.md#ldata)       | Extracts signal (and annotation) data from an EDF for one or more epochs |
| [`ldata.intervals()`](ref.md#ldataintervals)  | Extracts signal (and annotation) data from an EDF for one or more intervals |
| [`lannots()`](ref.md#lannots)   | Returns a vector of annotation class names for the attached EDF |
| [`lepoch()`](ref.md#lepoch)     | Epochs the data |
| [`letable()`](ref.md#letable)   | Returns an _epoch-table_ with information about epochs, masks and annotations |
| [`lstages()`](ref.md#lstages)   | Return vector of sleep stages from annotations |
| [`ladd.annot()`](ref.md#laddannot) | Attaches an new annotation from an R interval list |
| [`ladd.annot.file()`](ref.md#laddannotfile) | Attaches an new annotation from an file |
| [`leval()`](ref.md#leval)       | Evaluates an arbitrary set of Luna commands |
| [`leval.project()`](ref.md#levalproject)       | Evaluates an arbitrary set of Luna commands for all members of a project |
| [`lcmd()`](ref.md#lcmd)         | Reads and parses a Luna command script from a file |
| [`literate()`](ref.md#literate) | Applies an arbitrary R function to signal data, one epoch or interval at a time |
| [`ldb()`](ref.md#ldb)           | Imports data from a [_lunout_ database](../../luna/destrat.md) as an R list object |
| [`lx()`](ref.md#lx)             | Extracts table(s) from objects returned by `ldb()`, `leval()` or `leval.project() |
| [`lx2()`](ref.md#lx2)           | Like `lx()` but for lists of returned objects |
| [`lid()`](ref.md#lid)           | Extract a particular individual from a returned data frame |
| [`ltxttab()`](ref.md#ltxttab)   | Loads and conconcatenates multiple text-table output files |
| [`lstgcols()`](ref.md#lstgcols) | Maps typical stage labels to colors (for plotting) |
| [`lbands()`](ref.md#lands)      | Splits a signal into five band-pass filtered signals |
| [`llog()`](ref.md#llog)         | Turns Luna's typical console messages on/off | 
| [`le2i()`](ref.md#le2i)         | Converts epochs to intervals |
| [`lsanitize()`](ref.md#lsanitize) | Cleans syntax of command, factor/level and channel names |
| [`ldenoise()`](ref.md#ldenoise) | 1D total variation denoiser |
| [`lheatmap()`](viz.md#lheatmap) | Plots heatmaps, e.g. for spectrograms | 
| [`ltopo.heat()`](viz.md#ltopoheat) | Topoplot (sensor-level) |
| [`ltopo.rb()`](viz.md#ltoporb) | Topoplot (sensor-level) with red-white-blue |
| [`ltopo.xy()`](viz.md#ltopoxy) | Topoplot (sensor-level) X-Y plots |
| [`ltopo.heat2()`](viz.md#ltopoheat2) | Topoplot (sensor-level) of X-Y-Z heatmaps |
| [`ltopo.conn()`](viz.md#ltopoconn) | Topoplot (sensor-level) of pairwise connectivity |
| [`ltopo.dconn()`](viz.md#ltopodconn) | Topoplot (sensor-level) of single-sensor seeded connectivity |
| [`ltopo.topo()`](viz.md#ltopotopo) | Topoplot (sensor-level) of topoplots |
| [`ltopo.topo()`](viz.md#ltopotopo) | Topoplot (sensor-level) of topoplots |
| [`ltopo.topo()`](viz.md#ltopotopo) | Topoplot (sensor-level) of topoplots |
| [`ldefault.xy()`](viz.md#ldefaultxy) | Default XY channel locations for plotting |
| [`ldefault.coh.xy()`](viz.md#ldefaultcohxy) | Default XY channel locations for coherence/connectivity plots |
