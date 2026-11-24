# Source code

The C/C++ build instructions on this page are for `luna-base`. The R
package `luna` 1.8.0 uses lunapi through reticulate and needs no compilation;
see [R installation](lunar.md).

## Preliminaries

To compile luna from source, you'll need a system capable of
configuring and compiling C/C++ code, and for the dependent libraries,
ideally some kind of package manager.  The instructions below refer to
Unix-like operating systems, namely Linux and macOS.  

!!! hint "Please note..."
    If you are having trouble
    compiling _lunaC_, you might try working with the
    [Dockerized version](docker.md) instead.

Before attempting to compile Luna, please ensure that you have the
following (all of which can be obtained for free):

- a __C/C++ compiler__: on macOS, this can be achieved by installing
  Apple's XCode Command Line Tools.  Open a `Terminal` window and type
  (you may get an error message if they are in fact already installed):
  ```
  xcode-select --install
  ```

- a __text editor__: ensure you have a text editor that works with
  plain-text files (i.e. __not__ a word processor), either old-school
  (e.g. `emacs`, `nano`, both of which are in the luna Docker container)
  or new-school (e.g. [Atom](https://atom.io/), which is free, or
  [Sublime Text](https://www.sublimetext.com/), which is not)

- __R__ : the [R project for statistical
  computing](https://www.r-project.org/) is obviously necessary to use
  _lunaR_ -- but even if you don't plan to use _lunaR_, it will help when
  working with Luna output

- optionally, a __package manager__: to obtain dependent libraries for
  Luna, a tool like [Homebrew](https://brew.sh) is very useful; follow
  the instructions on the linked page to install Homebrew. It is
  possible that you may also need __autotools__ installed to
  compile some dependent libraries; e.g. on macOS with Homebrew
  installed, these can be obtained by typing:
  ```
  brew install automake autoconf libtool
  ```


## FFTW3 

Beyond the preliminaries mentioned above, Luna
requires [FFTW3](<http://www.fftw.org>), a C subroutine library for
computing the discrete Fourier transform (DFT).  If FFTW is not
installed on your system, choose one of the following routes to obtain
it:

__Option 1) Use a package manager__ 

On a system with a package manager, e.g. such as
[`apt`](https://wiki.debian.org/Apt) or [Homebrew](https://brew.sh),
you can run the following to first update the required dependencies
(possibly prefixing each command with `sudo` if needed, or installing
locally):

- On a Linux system using `apt-get`, first make sure everything is up-to-date:
```
apt-get update 
```
and then obtain FFTW:
```
apt-get install libfftw3-dev
```

- On a Mac using the [Homebrew](https://brew.sh) package manager, you might try:
```
brew install fftw 
```

If these options install FFTW so that it is accessible
system-wide, no special steps will be necessary when compiling
Luna.   If not, you will need to add an extra argument when compiling Luna.  For
example, newer versions of Homebrew place files under `/opt/homebrew/` which will
not, by default, be in the system path. Check by typing:
```
echo $PATH
```
You can either add that folder to the path, or add `FFTW=/opt/homebrew/` when running `make` as shown below.


__Option 2) Compile FFTW from source__

Alternatively, go to the [FFTW download page](http://www.fftw.org/download.html) to
pull the latest source.  The current version used to compile Luna is
3.3.10, which you can directly download from this link:
[`http://www.fftw.org/fftw-3.3.10.tar.gz`](http://www.fftw.org/fftw-3.3.10.tar.gz).

Let's say you download `fftw-3.3.10.tar.gz` to the directory
`/home/joe/fftw/`.  The following should compile and install FFTW _in
that directory_ (i.e. rather than system-wide):

```
cd /home/joe/fftw/

tar -xzvf fftw-3.3.10.tar.gz

cd fftw-3.3.10

./configure --enable-shared --prefix=/home/joe/fftw/

make && make install
```

You'll then need to set the `FFTW` variable when compiling `luna-base`,
as noted in the next section.

## Luna base

Choose one source route from the
[`luna-base`](https://github.com/remnrem/luna-base) repository:

```bash
# Pinned stable release
git clone --branch v1.8.0 https://github.com/remnrem/luna-base.git

# Unpinned: current development branch
git clone https://github.com/remnrem/luna-base.git
```

The current stable release is **1.8.0**. An unpinned clone may contain
unreleased changes. The compilation instructions below apply to the
checkout you selected.

Enter the `luna-base` folder:

```
cd luna-base
```

__If FFTW libraries are accessible system-wide, to build luna:__

- On a Linux platform:
```
make
```
- On macOS:
```
make ARCH=MAC
```

__To specify a non-standard location of the Luna dependencies__, i.e. if these were installed 
in `/home/joe/fftw/`, then set the `FFTW` variable equal to that location:

- On a Linux platform
```
make FFTW=/home/joe/fftw/
```
- On a macOS platform
```
make ARCH=MAC FFTW=/home/joe/fftw/
```

_Hopefully_, this will result in a ready-to-run executable, `luna`, as
well as `destrat` and some other files.  Test these by typing
`./luna`, which should give something like:

```
./luna 
```

```
usage:
 luna [sample-list|EDF] [n1] [n2] [@parameters] [sig=s1,s2] [v1=val1] < commands
```

and

```
$ ./destrat 
```
```
error : usage: destrat stout.db {-f|-d|-s|-v|-i|-r|-c|-n|-e}
```

You can copy `luna` and `destrat` to any folder, e.g. likely one that
is in your path, such as `/usr/local/bin/`, so that you can
subsequently run luna by typing `luna` in any folder.


## LightGBM

To use the [`POPS`](../ref/pops.md) and other commands, you need to
enable the [LightGBM](https://lightgbm.readthedocs.io) library when
compiling Luna.  First obtain the LightGBM library on your system, following the
steps described in the documentation link above.   On many systems, a package manager
may be able to achieve this in a single step: e.g. on macOS

```
brew install lightgbm
```

Determine where this is installed and whether it is in your path: e.g. newer versions of homebrew on macOS place
these files in `/opt/homebrew/` which needs to be explicitly passed to `LGBM_PATH=/opt/homebrew/` below.


Then, when compiling Luna from source, add the `LGBM=1` flag: e.g.
```
make -j 4 ARCH=MAC LGBM=1 
```

If LightGBM is installed in a nonstandard location, you can point to
it with the `LGBM_PATH` variable.  For example, to install on Linux in
a local folder, following the documentation
[here](https://lightgbm.readthedocs.io/en/latest/Installation-Guide.html#linux):

```
mkdir ~/tmp
cd ~/tmp
git clone --recursive https://github.com/microsoft/LightGBM
cd LightGBM
mkdir build
cd build
cmake ..
make -j4
```

Then, when building Luna, add, for example:

```
cd ~/src/luna-base/
make -j4 LGBM=1 LGBM_PATH=~/tmp/LightGBM
```


## ONNX Runtime support

ONNX Runtime (ORT) support is optional. The instructions below build a
shared, CPU-only ORT library with the operators and features needed by the
Luna ORT interface. These instructions use ORT version 1.29.0; use the same
ORT version when building on each platform.

The ORT source tree is only used to build the library. Luna needs the final
shared library and headers arranged as follows:

```
onnxruntime-dist/
  include/onnxruntime/
  lib/
```

### macOS

Install the build prerequisites:

```
brew install cmake python git
```

Download the ORT source:

```
git clone --branch v1.29.0 --depth 1 \
  https://github.com/microsoft/onnxruntime.git ~/onnxruntime
cd ~/onnxruntime
```

Build the shared CPU library:

```
python3 tools/ci_build/build.py \
  --build_dir build/MacOS/ReleaseShared \
  --config Release \
  --build_shared_lib \
  --skip_tests \
  --parallel 0 \
  --disable_contrib_ops \
  --disable_ml_ops \
  --no_telemetry \
  --cmake_extra_defines onnxruntime_BUILD_UNIT_TESTS=OFF \
  --target onnxruntime \
  --cmake_generator "Unix Makefiles" \
  --update \
  --build
```

Stage the library and headers:

```
mkdir -p ~/onnxruntime/dist/cpu-shared/lib
mkdir -p ~/onnxruntime/dist/cpu-shared/include
cp build/MacOS/ReleaseShared/Release/libonnxruntime.dylib \
  ~/onnxruntime/dist/cpu-shared/lib/
cp -R include/onnxruntime \
  ~/onnxruntime/dist/cpu-shared/include/
install_name_tool -id '@rpath/libonnxruntime.dylib' \
  ~/onnxruntime/dist/cpu-shared/lib/libonnxruntime.dylib
```

Build Luna with ORT support:

```
cd ~/luna-base
make ARCH=MAC ORT=1 \
  ORT_PATH=$HOME/onnxruntime/dist/cpu-shared
```

### Linux

On Debian or Ubuntu, install the build prerequisites:

```
sudo apt-get update
sudo apt-get install -y build-essential cmake python3 git
```

Download the ORT source:

```
git clone --branch v1.29.0 --depth 1 \
  https://github.com/microsoft/onnxruntime.git ~/onnxruntime
cd ~/onnxruntime
```

Build the shared CPU library:

```
python3 tools/ci_build/build.py \
  --build_dir build/Linux/ReleaseShared \
  --config Release \
  --build_shared_lib \
  --skip_tests \
  --parallel 0 \
  --disable_contrib_ops \
  --disable_ml_ops \
  --no_telemetry \
  --cmake_extra_defines onnxruntime_BUILD_UNIT_TESTS=OFF \
  --target onnxruntime \
  --cmake_generator "Unix Makefiles" \
  --update \
  --build
```

Stage the library and headers:

```
mkdir -p ~/onnxruntime/dist/cpu-shared/lib
mkdir -p ~/onnxruntime/dist/cpu-shared/include
cp build/Linux/ReleaseShared/Release/libonnxruntime.so \
  ~/onnxruntime/dist/cpu-shared/lib/
cp -R include/onnxruntime \
  ~/onnxruntime/dist/cpu-shared/include/
```

Build Luna with ORT support:

```
cd ~/luna-base
make ORT=1 ORT_PATH=$HOME/onnxruntime/dist/cpu-shared
```

### Windows (MinGW)

The Windows `luna-base` build uses MinGW. Build ORT with MinGW as well; an
ORT library built with MSVC is not compatible with the MinGW Luna build.
Run these commands from an MSYS2 MinGW shell with GCC, CMake, Python, and Git
available:

```
git clone --branch v1.29.0 --depth 1 \
  https://github.com/microsoft/onnxruntime.git C:/onnxruntime
cd /c/onnxruntime
```

Build the shared CPU library:

```
python tools/ci_build/build.py \
  --build_dir build/Windows/ReleaseShared \
  --config Release \
  --build_shared_lib \
  --skip_tests \
  --parallel 0 \
  --disable_contrib_ops \
  --disable_ml_ops \
  --no_telemetry \
  --cmake_extra_defines onnxruntime_BUILD_UNIT_TESTS=OFF \
  --target onnxruntime \
  --cmake_generator "MinGW Makefiles" \
  --update \
  --build
```

Stage the library, import library, and headers from the Release output:

```
mkdir -p /c/onnxruntime/dist/cpu-shared/lib
mkdir -p /c/onnxruntime/dist/cpu-shared/include
cp build/Windows/ReleaseShared/Release/onnxruntime.dll \
  /c/onnxruntime/dist/cpu-shared/lib/
cp build/Windows/ReleaseShared/Release/onnxruntime.lib \
  /c/onnxruntime/dist/cpu-shared/lib/
cp -R include/onnxruntime \
  /c/onnxruntime/dist/cpu-shared/include/
```

Build Luna with ORT support from the MinGW shell:

```
cd /c/path/to/luna-base
make ARCH=WINDOWS ORT=1 \
  ORT_PATH=C:/onnxruntime/dist/cpu-shared
```
