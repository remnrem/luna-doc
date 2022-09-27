# Luna: software for the analysis of sleep signal data

Luna is an open-source C/C++ software package for manipulating and
analyzing polysomnographic recordings, with a focus on the sleep EEG.
Primarily oriented around command-line scripting
([_lunaC_](luna/args.md)), we are developing various _extensions_,
including a package for the [R](https://www.r-project.org/)
statistical package ([_lunaR_](ext/R/index.md)). Luna is _actively under
development_ and we welcome feedback.  

The
__current release is v0.27__ (27-Sep-2022): see [here](updates.md) for
a list of changes/additions.   
    
## Getting started
 
After [downloading](download/index.md) Luna, the best place to start
is the [tutorial](tut/tut1.md).  Then work your way through the pages
listed in the left-hand side menu.  (On devices with smaller screens
this may be minimized: if so, click the top left three horizontal bars
icon.)  In particular, the [_lunaC_](luna/args.md) describes many key
concepts and conventions (many of which are also relevant for the R
package).

The Luna package comprises a number of components, primarily:

- [_lunaC_](luna/args.md), a command-line interface to the Luna C/C++ library

- [_lunaR_](ext/R/index.md), a package for the [R statistical software](https://www.r-project.org/)

Both _lunaC_ and _lunaR_ are based on the same underlying Luna
library: see the [reference overview](ref/index.md) for
detailed descriptions of Luna's commands.

!!! hint "Luna and the National Sleep Research Resource"
    If you want to use Luna to work with data from the [NSRR](http://sleepdata.org), 
    please see [this page](nsrr.md).

## Installation options

You can install Luna in a number of ways.  The easiest approach is to
download [binary executables](download/exec.md); you
can also compile [from source](download/source.md)):

![img](img/install1.png)

An alternative is to run Luna in a [Docker container](download/docker.md).
If you can install Docker on your machine, this may be a good route to
test-drive Luna.  We've generated two Docker containers: both include
_lunaC_ and _lunaR_, either in the classic R environment, or
via [RStudio](https://www.rstudio.com):

![img](img/install2.png)
    
## Things Luna aims to do

The [reference](ref/index.md) pages list all currently-supported
commands. Main areas are summarized below.
 
!!! success "Primary use cases"    
    * Read, manipulate and write large sets of EDF and EDF+ signals 
    * Filter, resample and re-reference signals
    * Generate a variety of (per-epoch) summary statistics
    * Statistical artifact detection for EEG channels
    * Annotate and mask/filter epochs
    * Estimate key features of sleep macro-architecture
    * Spectral analyses
    * Spindle and slow oscillation detection
    * Coherence and cross-frequency coupling 
    * Multi-channnel, topographical analyses

## Things Luna _doesn't_ aim to do

Luna was originally designed to work with the large number of
polysomnograms at the [NSRR](http://sleepdata.org/), with a focus on
intersecting sleep EEG signals with other annotations.  As such, some
areas are not well supported, or effectively outside of Luna's scope.

!!! failure "Areas outside of Luna's primary focus"

    * _Visual data exploration_: Luna is not a highly interactive,
    _point-and-click_ tool
    
    * _Methods development platform_: although the R extension can support
    methods development, other tools (including general purpose Matlab
    packages such as [EEGLAB](https://sccn.ucsd.edu/eeglab/index.php))
    will naturally be better suited for expert users interested in
    flexibly altering and developing new analyses
    
    * _Online signal processing_: Luna is set up for all analyses
      being done offline, i.e. on the entire recording

    * _Support for multiple formats_: currently, Luna is mainly based
      around EDF and EDF+ files (as well as plain text)  
 
    * _Analyses of cardiac and respiratory events_: most of Luna's specialized sleep analyses are
    currently focused on EEG signals (e.g. spindles and slow oscillations)

