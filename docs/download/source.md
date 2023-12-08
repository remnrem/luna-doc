# Source code

## Preliminaries

To compile luna from source, you'll need a system capable of
configuring and compiling C/C++ code, and for the dependent libraries,
ideally some kind of package manager.  The instructions below refer to
Unix-like operating systems, namely Linux and macOS.  

!!! hint "Please note..."
    If you are having trouble
    installing _lunaC_ or _lunaR_, you might try working with the 
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
  possible that you may also need to __autotools__ installed to
  compile some dependent libraries; e.g. on macOS with Homebrew
  installed, these can be obtained by typing:
  ```
  brew install automake autoconf libtool
  ```


## zlib

Many systems will have the zlib compression library installed by
default.  If not, you'll see an error message about a missing `zlib.h` file
when trying to compile Luna.

Get the development version (i.e. which includes the source headers),
either via a package manager such as (depending on your platform)
`homebrew` or `apt-get`: e.g.

```
brew install zlib
```
or
```
apt-get install zlib1g-dev
```

Alternatively, obtain the source from the project website: [https://zlib.net](https://zlib.net)


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
system-wide, so no special steps will be necessary when compiling
Luna.   If not, you will need to add an extra argument when compiling Luna.  For
example, newere versions of Homebrew place files under `/opt/homebrew/` which will
not, by default, by in the system path (i.e. check by typing
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

You'll then need to set the `FFTW` variable when compiling Luna (and
_lunaR_ too, if compiling that from source), as noted in the next
section.

## Luna base

Pull the latest Luna source code from the
[`luna-base`](<https://github.com/remnrem/luna-base>) Github repository:

```
git clone https://github.com/remnrem/luna-base.git
```

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


## lunaR

!!! warning
    LunaR is not currently supported for Windows.  Use the Docker version instead.

Make sure you are running a relatively recent version of R (which can
be [downloaded here](https://www.r-project.org)): _lunaR_ was
developed on version 3.5.1.

You can obtain source code directly:
```
git clone https://github.com/remnrem/luna.git
```

Install the package by running:
```
R CMD INSTALL luna
```

If needed, you can set `FFTW` to the location where you
compiled FFTW as follows:
```
FFTW=/Users/joe/fftw R CMD INSTALL luna
```
If the above works, this will allow you to attach the _lunaR_ library in R:
```
library(luna)
```
If you don't have permission to write to the main R library, try creating a separate folder as follows:
```
mkdir ~/Rlib

FFTW=/Users/joe/fftw R CMD INSTALL --library=~/my-Rlib luna
```    
Then, when in R, you'll need to call:
```
library(luna, lib.loc="~/my-Rlib")
```    


!!! note "Installing with `devtools`"

    Alternatively, you can try using the `devtools` R package to install _lunaR_.  
    ```
    library(devtools)
    ```
    If you manually installed FFTW to a nonstandard location, 
    then set the `FFTW` environment variable too:

    ```
    Sys.setenv( FFTW="/home/joe/fftw/" ) 
    ```

    Then run the following command to pull the package source from a
    [Github](http://github.org) and compile it:

    ```
    devtools::install_github( "remnrem/luna" ) 
    ```

    If there is an issue with permissions, you might need to write this to
    an alternate local library, e.g. assuming the `~/my-Rlib` folder exists:

    ```
    devtools::install_github( "remnrem/luna" , lib="~/my-Rlib" ) 
    ```

    Assuming that works, then once installed you should subsequently be able to run only this command to start the library:
    ```
    library(luna)
    ```
    ```
    ** lunaR v0.25.5 31-Mar-2021
    ** luna v0.25.5 31-Mar-2021
    ```
    If you specified an alternate library location, you'll need to add that too here:
    ```
    library(luna,lib.loc="~/my-Rlib")
    ```


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


### LightGBM and lunaR

Assuming LightGBM is in your system path, you can build _lunaR_ with
LGBM support by adding the `LGBM=1` flag (so that the base Luna library will be
compiled with LGBM support) and `EXTRA_PKG_LIBS` (so that the R library will link to
the LGBM library):

```
LGBM=1 EXTRA_PKG_LIBS="-l_lightgbm" R CMD INSTALL luna
```

Otherwise, you can specify additional flags using `EXTRA_PKG_CPPFLAGS`
and `EXTRA_PKG_LIBS` to point to the LighGBM headers and libraries.  As a full example:

 - if FFTW has been manually installed at `/Users/mary/src/fftw3-3.3.10`

 - if LGBM has been manually installed at `/Users/mary/src/LightGBM`

 - to correctly build the base Luna library (the prerequisite for
 install the R package) we need to pass Luna information on the
 location of FFTW (via `FFTW`) and to include LightGBM (`LGBM=1`) and
 also point to its location (`LGBM_PATH`)

 - we also,separately need to tell the R library compilaton step about the locations of both libraries, via `EXTRA_PKG_LIBS`

In all:

```
FFTW="/Users/albert/fftw-3.3.10/" \
 LGBM=1 \
 LGBM_PATH="/Users/albert/src/LightGBM/" \
 EXTRA_PKG_LIBS="-L/Users/albert/src/LightGBM/ -L/Users/albert/src/fftw-3.3.10/ -l_lightgbm" \
 R CMD INSTALL luna
```

As a second example: if FFTW is avaialable system wide, but say LGBM
has been installed outside of the system path (say from `brew install`
on newer Macs):

```
LGBM=1 \
 LGBM_PATH="/opt/homebrew/" \
 EXTRA_PKG_LIBS="-L/opt/homebrew/lib -l_lightgbm" \
 R CMD INSTALL luna
```
