# Source code

## Preliminaries

To compile luna from source, you'll need a system capable of
configuring and compiling C/C++ code, and for the dependent libraries,
ideally some kind of package manager.  The instructions below refer to
Unix-like operating systems, namely Linux and Mac OS.  

!!! hint "Please note..."
    We're still in the process of optimizing the installation
    process, especially for _lunaR_ which is not guaranteed to work
    seamlessly on all platforms....  If you are having trouble
    installing _lunaC_ or _lunaR_, try working with the 
    [Docker container](docker.md) for now.

Before attempting to compile Luna, please ensure that you have the
following (all of which can be obtained for free):

- a __C/C++ compiler__: on Mac OSX, this can be achieved by installing
  Apple's XCode Command Line Tools.  Open a `Terminal` window and type
  (you may get an error if they are already installed; to test whether
  they are already installed.):

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

- optionally, a __package manager__: to obtain dependent libraries
  for Luna, a tool like [Homebrew](https://brew.sh) is very useful;
  follow the instructions on the linked page to install Homebrew

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

Alternatively, obtain the source from the project website: (https://zlib.net)


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

These options should install FFTW so that it is accessible
system-wide, so no special steps will be necessary when compiling
Luna.


__Option 2) Alternatively, compile FFTW from source__

Go to the [FFTW download page](http://www.fftw.org/download.html) to
pull the latest source.  The current version used to compile Luna is
3.3.8, which you can directly download from this link:
[`http://www.fftw.org/fftw-3.3.8.tar.gz`](http://www.fftw.org/fftw-3.3.8.tar.gz).

Let's say you download `fftw-3.3.8.tar.gz` to the directory
`/home/joe/fftw/`.  The following should compile and install FFTW _in
that directory_ (i.e. rather than system-wide):

```
cd /home/joe/fftw/

tar -xzvf fftw-3.3.8.tar.gz

cd fftw-3.3.8

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
- On a Mac OSX platform:
```
make ARCH=MAC
```

__To specify a non-standard location of the Luna dependencies__, i.e. if these were installed 
in `/home/joe/fftw/`, then set the `FFTW` variable equal to that location:

- On a Linux platform
```
make FFTW=/home/joe/fftw/
```
- On a Mac OSX platform
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
===================================================================
+++ luna | v0.9, 12-Feb-2019 | starting process 2019-02-21 14:57:44
===================================================================
usage: luna [sample-list|EDF] [n1] [n2] [@param-file] [sig=s1,s2] [v1=val1] < command-file
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
developed on version 3.5.1.  To install, if you have the `devtools`
package, you might try this:

```
library(devtools)
```

<!---
Next, set an environment variable to `LUNA_BASE` point to the location of `luna-base`:
```
Sys.setenv( LUNA_BASE="/home/joe/luna-base/" ) 
```
--->


Optionally, if you manually installed FFTW in a nonstandard location, 
then set a `FFTW` environment variable too:

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
** lunaR v0.24.1 28-Aug-2020
** luna v0.24.1 28-Aug-2020
```
If you specified an alternate library location, you'll need to add that too here:
```
library(luna,lib.loc="~/my-Rlib")
```


!!! note

    Alternatively, (e.g. if you have trouble with `devtools`), you can obtain source code directly:
    ```
    git clone https://github.com/remnrem/luna.git
    ```

    Install the package, where if needed, `FFTW` is set to the location where you
    compiled FFTW as follows:
    ```
    FFTW=/Users/joe/fftw R CMD INSTALL luna
    ```
    This should then allow you to attach the lunaR library in R:
    ```
    library(luna)
    ```
    As above, if you don't have permission to write to the main R library, try creating a separate folder as follows:
    ```
    mkdir ~/Rlib
    
    FFTW=/Users/joe/fftw R CMD INSTALL --library=~/my-Rlib luna
    ```    
    Then, when in R, you'll need to call:
    ```
    library(luna, lib.loc="~/my-Rlib")
    ```    