# Downloads

<p align="right"><em>Current Luna release: <b>v0.28</b> (11-Apr-2023)</em></p>

## Quick links

Latest stable Luna command-line tool (binaries and source):

| Platform | Link |
| ----- | ----- |
| __Source code (all platforms)__ | [https://github.com/remnrem/luna-base/archive/refs/tags/v0.28.tar.gz](https://github.com/remnrem/luna-base/archive/refs/tags/v0.28.tar.gz) |
| macOS (Intel/x86_64) binary executable | [https://github.com/remnrem/luna-base/releases/download/v0.28/mac_luna.tar.gz](https://github.com/remnrem/luna-base/releases/download/v0.28/mac_luna.tar.gz) |
| Windows binary executable | [https://github.com/remnrem/luna-base/releases/download/v0.28/win_luna.zip](https://github.com/remnrem/luna-base/releases/download/v0.28/win_luna.zip) |

<!-- | macOS (Silicon/ARM64) binary executable | [http://zzz.bwh.harvard.edu/dist/luna/macOS-arm64.gz](http://zzz.bwh.harvard.edu/dist/luna/macOS-arm64-v0.28.gz)| -->


Latest development source (Luna and _lunaR_):

| Platform | Link |
| ----- | ----- |
| Luna source code (all platforms) | [https://github.com/remnrem/luna-base/](https://github.com/remnrem/luna-base/) |
| _lunaR_ source code (all platforms) | [https://github.com/remnrem/luna/](https://github.com/remnrem/luna/) |


## Installation options 

Luna is released under the
[GPLv3](https://www.gnu.org/licenses/gpl-3.0.en.html) license,
allowing free sharing and modification of the source code.  There are
three basic ways to obtain Luna:

- ultimately, the best route is to compile from source: see [here](source.md) for _lunaC_ and _lunaR_, along with
  instructions for compilation

- [binary executables](exec.md) for _lunaC_ for Mac OS and Windows

- or as a [Docker container](docker.md), which allows any machine with
  [Docker](http://www.docker.com) installed on it to run Luna (both
  _lunaC_ and _lunaR_) and comes with R (and RStudio) and the tutorial data pre-installed.


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

