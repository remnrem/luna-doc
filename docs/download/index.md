# Downloads

<p align="right"><em>Current stable Luna release: <b>v1.8.0</b></em></p>



## Quick links

The current stable release is **1.8.0** for `luna-base`, `luna-api`
(the `lunapi` Python package), LunaScope, _lunaR_ (the R package `luna`),
and the three Docker images below. Choose the pinned route to keep this
release fixed, or the unpinned route to follow updates. Unpinned package
and binary routes follow releases; unpinned Git routes follow development.

### Binaries and packages

| Component | Pinned: 1.8.0 | Unpinned |
| ----- | ----- | ----- |
| macOS (Intel/x86_64) binaries | [v1.8.0](https://github.com/remnrem/luna-base/releases/download/v1.8.0/mac_luna.tar.gz) | [Latest](https://github.com/remnrem/luna-base/releases/latest/download/mac_luna.tar.gz) |
| macOS (Silicon/ARM64) binaries | [v1.8.0](https://github.com/remnrem/luna-base/releases/download/v1.8.0/macos_arm64_luna.tar.gz) | [Latest](https://github.com/remnrem/luna-base/releases/latest/download/macos_arm64_luna.tar.gz) |
| Windows binaries | [v1.8.0](https://github.com/remnrem/luna-base/releases/download/v1.8.0/win_luna.zip) | [Latest](https://github.com/remnrem/luna-base/releases/latest/download/win_luna.zip) |
| _lunapi_ (`luna-api`) | `python -m pip install lunapi==1.8.0` | `python -m pip install --upgrade lunapi` |
| LunaScope with _lunapi_ | `python -m pip install lunascope==1.8.0 lunapi==1.8.0` | `python -m pip install --upgrade lunascope lunapi` |
| R package `luna` | `pak::pak("remnrem/lunaR@v1.8.0")` | `pak::pak("remnrem/lunaR")` |

Install `pak` first with `install.packages("pak")`. The unpinned R route
tracks the development branch. See [R installation](lunar.md) for
first-use setup and pinning the Python engine too.
For GUI setup, see [LunaScope installation](#lunascope).

### Source

Git tags use the `v` prefix. Unpinned clones track each repository's
default development branch, which may be newer than the stable release.
Choose one clone command per repository, then follow its build instructions.

| Repository | Pinned: v1.8.0 | Unpinned: development |
| ----- | ----- | ----- |
| `luna-base` | `git clone --branch v1.8.0 https://github.com/remnrem/luna-base.git` | `git clone https://github.com/remnrem/luna-base.git` |
| `luna-api` | `git clone --branch v1.8.0 https://github.com/remnrem/luna-api.git` | `git clone https://github.com/remnrem/luna-api.git` |
| `lunascope` | `git clone --branch v1.8.0 https://github.com/remnrem/lunascope.git` | `git clone https://github.com/remnrem/lunascope.git` |
| R package `luna` (`lunaR` repository) | `git clone --branch v1.8.0 https://github.com/remnrem/lunaR.git` | `git clone https://github.com/remnrem/lunaR.git` |

The `luna-base` source archive is also available as a
[pinned v1.8.0 download](https://github.com/remnrem/luna-base/archive/refs/tags/v1.8.0.tar.gz);
the [latest release page](https://github.com/remnrem/luna-base/releases/latest)
provides the source archive for the latest stable release.

### Docker

| Image | Pinned: 1.8.0 | Unpinned: latest |
| ----- | ----- | ----- |
| Luna + _lunaR_ / RStudio | `docker pull remnrem/luna:1.8.0` | `docker pull remnrem/luna:latest` |
| + JupyterLab & _lunapi_ | `docker pull remnrem/lunapi:1.8.0` | `docker pull remnrem/lunapi:latest` |
| Lightweight command-line Luna | `docker pull remnrem/lunalite:1.8.0` | `docker pull remnrem/lunalite:latest` |

Use the same tag in `docker run`; pulling `:1.8.0` does not change which
image an untagged run uses. See [Docker](docker.md) for paired examples.
 
## Installation options 

Luna is released under the
[GPLv3](https://www.gnu.org/licenses/gpl-3.0.en.html) license,
allowing free sharing and modification of the source code.  There are
multiple ways to obtain Luna:

- compile the command-line _lunaC_ from [source](source.md)

- install the R package `luna` directly from `remnrem/lunaR` using
  [pak](lunar.md); the R interface requires no compilation

- download [binary executables](exec.md) for _lunaC_ for Mac OS and Windows

- install the Python package with `python -m pip install lunapi==1.8.0`
  (pinned) or `python -m pip install --upgrade lunapi` (unpinned)

- or pull a [Docker container](docker.md), which allows any machine
  with [Docker](http://www.docker.com) installed on it to run Luna
  (both _lunaC_ and either _lunapi_ or _lunaR_) and comes with JupyterLab (or R and
  RStudio) and with the tutorial data pre-installed.


!!! hint "Advice for Linux & macOS users"
    For command-line Luna on up-to-date Linux and macOS distributions, you
    can pull the [source](source.md) and compile directly. For R, use the
    [R package installation](lunar.md).
 
!!! hint "Advice for Windows" 
    For Unix-style command-line workflows on Windows, consider
    [Docker](docker.md). The R package `luna` can be installed directly
    on Windows x86_64 using the [same R commands](lunar.md) as on
    macOS and Linux; Docker is optional.

!!! hint "Quick start w/ Docker"
    If you have trouble installing Luna via the above methods, using a Docker container may instead prove to be a relatively easy
    way to get a full Luna environment up and running.


## R package `luna`

See [lunaR installation](lunar.md) for pinned 1.8.0 and unpinned routes,
automatic Python setup, and installation from a local source checkout.

## LunaScope

In an activated Python virtual environment, choose one route:

```bash
# Pinned stable release (including the Python engine)
python -m pip install lunascope==1.8.0 lunapi==1.8.0

# Unpinned: install or upgrade to the latest releases
python -m pip install --upgrade lunascope lunapi
```

For GUI setup and usage, see the dedicated
[LunaScope install instructions](https://zzz-luna.org/lunascope/install/).
    

## Data resources

The [data](data.md) page contains a number of resources that can be
used with Luna on any platform, including the [tutorial](../tut/tut1.md) and [walk-through](https://zzz-luna.org/luna-walkthrough)
data.


## Changelog and known issues

For recent changes and additions in the latest version, see the [CHANGELOG](../updates.md) page.

For any known issues impacting the current release, see [this page](misc.md).
