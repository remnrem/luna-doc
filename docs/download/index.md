# Downloads

<p align="right"><em>Current Luna release: <b>v1.00</b> (31-May-2024)</em></p>

## Quick links

Latest stable Luna command-line tool (binaries and source):

!!! note
    The latest release (v1.0) is only available via the [Github repo](http://github.com/remnrem/luna-base) and [Docker images](https://hub.docker.com/r/remnrem/lunapi);  binaries for v1.0 will be posted in due course.

| Platform | Link |
| ----- | ----- |
| __Source code (all platforms)__ | [https://github.com/remnrem/luna-base/archive/refs/tags/v0.99.tar.gz](https://github.com/remnrem/luna-base/archive/refs/tags/v0.99.tar.gz) |
| macOS (Intel/x86_64) binary executable | [https://github.com/remnrem/luna-base/releases/download/v0.99/mac_luna.tar.gz](https://github.com/remnrem/luna-base/releases/download/v0.99/mac_luna.tar.gz) |
| macOS (Silicon/ARM64) binary executable | [http://zzz.bwh.harvard.edu/dist/luna/macOS-arm64-v0.99.gz](http://zzz.bwh.harvard.edu/dist/luna/macOS-arm64-v0.99.gz)|
| Windows binary executable | [https://github.com/remnrem/luna-base/releases/download/v0.99/win_luna.zip](https://github.com/remnrem/luna-base/releases/download/v0.99/win_luna.zip) |

The binary distributions each contain a folder with the main executables
(`luna`, `destrat`, etc) and some necessary libraries).  All files
must be kept in the same folder. Add this folder to your system's path environment variable,
e.g. on macOS:
```
export PATH=$PATH:/Users/john/downloads/luna-v0.99/
```
so that you can just type `luna` in any location and it will run the executable here. 
For macOS, see [these notes](exec.md##macos-installation-notes) about how to get rid of
harmless security warnings that will be shown the first time you try
to run the code.  Also note that these are _command-line tools_ -
i.e. use the terminal/command prompt on your machine rather than
clicking on the icons.  

Latest development source (Luna and _lunaR_):

| Platform | Link |
| ----- | ----- |
| Luna source code (all platforms) | [https://github.com/remnrem/luna-base/](https://github.com/remnrem/luna-base/) |
| _lunaR_ source code (all platforms) | [https://github.com/remnrem/luna/](https://github.com/remnrem/luna/) |


## Installation options 

Luna is released under the
[GPLv3](https://www.gnu.org/licenses/gpl-3.0.en.html) license,
allowing free sharing and modification of the source code.  There are
multiple ways to obtain Luna:

- compile from source: see [here](source.md) for _lunaC_ and _lunaR_,
  along with instructions for compilation

- download [binary executables](exec.md) for _lunaC_ for Mac OS and Windows

- install the Python package with `pip install lunapi` (on macOS and Linux) 

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


## Data resources

The [data](data.md) page contains a number of resources that can be
used with Luna on any platform, including the [tutorial data](../tut/tut1.md).


## Changelog and known issues

For recent changes and additions in the latest version, see the [CHANGELOG](../updates.md) page.

For any known issues impacting the current release, see [this page](misc.md).

