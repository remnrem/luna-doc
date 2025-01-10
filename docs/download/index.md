# Downloads

<p align="right"><em>Current Luna release: <b>v1.2.0</b> (03-Jan-2025)</em></p>



## Quick links

Latest stable Luna command-line tool (binaries and source):

| Platform | Link/command |
| ----- | ----- |
| __Executables__ | |
| macOS (Intel/x86_64) binaries | [https://github.com/remnrem/luna-base/releases/download/v1.2.0/mac_luna.tar.gz](https://github.com/remnrem/luna-base/releases/download/v1.2.0/mac_luna.tar.gz) |
| macOS (Silicon/ARM64) binaries | [https://github.com/remnrem/luna-base/releases/download/v1.2.0/macos_arm64_luna.tar.gz](https://github.com/remnrem/luna-base/releases/download/v1.2.0/macos_arm64_luna.tar.gz) |
| Windows binaries | [https://github.com/remnrem/luna-base/releases/download/v1.2.0/win_luna.zip](https://github.com/remnrem/luna-base/releases/download/v1.2.0/win_luna.zip) |
| __Python__ | |
| _lunapi_ Python package only | `pip install lunapi` |
| __Source__| |
| Stable Luna (all platforms)| [https://github.com/remnrem/luna-base/archive/refs/tags/v1.2.0.tar.gz](https://github.com/remnrem/luna-base/archive/refs/tags/v1.2.0.tar.gz)
| Latest (unstable) Luna| [https://github.com/remnrem/luna-base/](https://github.com/remnrem/luna-base/) |
| Latest (unstable) _lunaR_ | [https://github.com/remnrem/luna/](https://github.com/remnrem/luna/) |
| __Docker__ | | 
| Core Luna | `docker pull remnrem/luna` |
| + JupyterLab & _lunapi_ | `docker pull remnrem/lunapi` |
| lightweight Luna | `docker pull remnrem/lunalite` |
 
## Installation options 

Luna is released under the
[GPLv3](https://www.gnu.org/licenses/gpl-3.0.en.html) license,
allowing free sharing and modification of the source code.  There are
multiple ways to obtain Luna:

- compile from source: see [here](source.md) for _lunaC_ and _lunaR_,
  along with instructions for compilation

- download [binary executables](exec.md) for _lunaC_ for Mac OS and Windows

- install the Python package with `pip install lunapi`

- or pull a [Docker container](docker.md), which allows any machine
  with [Docker](http://www.docker.com) installed on it to run Luna
  (both _lunaC_ and either _lunapi_ or _lunaR_) and comes with JupyerLab (or R and
  RStudio) and with the tutorial data pre-installed.


!!! hint "Advice for Linux & macOS users"
    For most up-to-date Linux and macOS distributions, we recommend pulling the
    [source](source.md) and compiling directly.
 
!!! hint "Advice for Windows" 
    For Windows, we recommend using the
    [Dockerized](docker.md) version of Luna to provide a complete
    environment in which Luna can operate best.  Currently, it is the only way that you can use _lunaR_ on Windows too.

!!! hint "Quick start w/ Docker"
    If you have trouble installing Luna via the above methods, using a Docker container may instead prove to be a relatively easy
    way to get a full Luna environment up and running.
    

## Data resources

The [data](data.md) page contains a number of resources that can be
used with Luna on any platform, including the [tutorial](../tut/tut1.md) and [walk-through](https://zzz.bwh.harvard.edu/luna-walkthrough)
data.


## Changelog and known issues

For recent changes and additions in the latest version, see the [CHANGELOG](../updates.md) page.

For any known issues impacting the current release, see [this page](misc.md).

