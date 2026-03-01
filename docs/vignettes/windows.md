# Luna for Windows users

This vignette gives a practical orientation for running Luna on
Windows, especially if you are following examples that were primarily
written with a Unix-like shell in mind.

The short version is:

 - if you want the simplest command-line route, use the Windows Luna binaries and run them from the classic Command Prompt

 - if you want the most complete environment, use the [Dockerized](../download/docker.md) version of Luna

 - if you want a browser-based interactive environment, consider [_lunapi_](../lunapi/index.md) plus the [`scope`](../lunapi/scope.md) viewer in JupyterLab; if your site already provides JupyterHub, that can often be a convenient way to host the same setup


## First: open a terminal

If you are new to command-line tools on Windows, start by opening a
plain command prompt:

 - press the Windows key

 - type `cmd`

 - open `Command Prompt`

You can then change folders with `cd`, for example:

```text
cd C:\Users\you\work
```

If you installed the Luna executables in the current folder, you can
run:

```text
luna -v
```

or, depending on how your system is configured:

```text
.\luna.exe -v
```

The main point is that Luna examples generally assume a shell that is
comfortable with plain command files, redirection, and quoted command
strings.


## Avoid PowerShell for typical Luna examples

Although PowerShell is useful in many contexts, it is often not the
best choice for standard Luna command-line usage.

In particular, many Luna examples use shell redirection such as:

```bash
luna s.lst < cmd.txt
```

or

```bash
destrat out.db +PSD -r CH B > psd.txt
```

The `>` form is common in many shells, but PowerShell has different
parsing rules and the `<` operator is especially awkward for these
examples.  In practice, if you want to follow Luna documentation
closely, it is safer to use one of:

 - the classic Windows Command Prompt

 - Git Bash

 - WSL

 - a Docker container running Luna

If you do use PowerShell, expect that some examples may need to be
rewritten rather than copied literally.


## Windows command-line differences

Most Luna commands are the same on Windows, macOS and Linux.  The main
differences are usually about the shell, file paths, and quoting.

For example, these are all Luna concepts that stay the same:

```text
luna s.lst 3/10 -o out.3.db < cmd.txt
destrat out.db +PSD -r CH B
luna s.lst sig=C3,C4 -s PSD
```

But on Windows, you should watch for the following:

 - paths often look like `C:\data\study` rather than `/data/study`

 - quoting rules may differ between `cmd.exe`, Git Bash, WSL and PowerShell

 - wildcard expansion can differ by shell

 - examples using Unix tools such as `cat`, `awk`, `grep`, `parallel` or shell loops may need Git Bash, WSL, Docker, or a Windows alternative

If you want the documentation examples to work with minimal
translation, a Unix-like shell environment is usually the easiest
choice.


## Using the Windows binaries

The binary installation route is described [here](../download/exec.md).
This is usually the quickest way to run plain command-line Luna on a
single Windows machine.

The main things to remember are:

 - use the Windows release files from the Luna downloads page

 - keep `luna`, `destrat`, and the accompanying `.dll` together

 - start in Command Prompt, not PowerShell

 - use plain-text editors for command files

For example, create a text file `cmd.txt` containing:

```text
DESC
```

and then run:

```text
luna sample.lst < cmd.txt
```

If that basic pattern works, most other command-line Luna workflows
follow the same structure.


## Docker is often the best Windows environment

The Luna docs already recommend [Docker](../download/docker.md) as the
most complete Windows environment.  This is also the only documented
way to get the full Luna ecosystem on Windows, including the R-based
workflow.

The main advantages are:

 - the environment is much closer to the Linux/macOS examples used throughout the docs

 - Unix shell tools are available inside the container

 - Docker is the documented route for using `_lunaR_` on Windows

 - it is often easier to reproduce the same setup across machines

The Windows-specific Docker notes are [here](../download/docker.md#windows-machines).

A typical Windows Docker command from the docs is:

```text
docker run --rm -it -v D:/luna:/data -v C:/mydata/work:/data1 remnrem/luna /bin/bash
```

Once inside the container, Luna behaves much more like the examples in
the rest of the documentation.


## Git Bash or WSL

If you prefer to stay outside Docker but still want a more Unix-like
shell, Git Bash or WSL can be a better fit than Command Prompt.

These environments can be helpful when you want:

 - shell redirection and quoting that behave more like the examples in the docs

 - basic Unix tools such as `awk`, `sed`, `grep`, and `cat`

 - easier use of shell loops and pipelines

That said, path translation between Windows and Unix-style paths can
itself introduce some complexity, so Docker is often simpler if you
also need R, Jupyter, or a more reproducible environment.


## Interactive environments: `_lunapi_`, `scope`, and JupyterHub

For interactive work, it may be easier not to use the `luna` command
line directly at all.

The [`_lunapi_`](../lunapi/index.md) interface works on Windows, and
the [`scope`](../lunapi/scope.md) viewer provides an interactive
browser-style environment within JupyterLab.

This is especially useful if you want:

 - interactive signal viewing

 - notebook-style analyses

 - a setup that mixes Luna with Python code

The docs note that `scope` requires JupyterLab.  In the Luna source,
the `lunascope`-specific hooks are visible in
`~/luna-base/lunapi/segsrv.cpp` and `~/luna-base/lunapi/segsrv.h`,
which is a good indication that this workflow is a first-class part of
the current codebase.

If your institution already provides JupyterHub, then a practical
approach can be to expose the same JupyterLab + `_lunapi_` workflow in
that environment.  That is an inference from the documented JupyterLab
setup rather than a separately documented Luna deployment recipe, but
in practice it is often a sensible Windows-friendly option because it
moves most environment management to the server side.


## A few practical hints

 - Prefer short paths with no spaces when you are starting out.

 - Keep Luna command files as plain text, for example `cmd.txt`.

 - If a copied example uses `< cmd.txt`, run it in Command Prompt, Git Bash, WSL, or Docker rather than PowerShell.

 - If a workflow uses `awk`, `parallel`, or similar Unix tools, either switch to Docker/Git Bash/WSL or translate that step into a Windows-native tool.

 - If you need the richest environment on Windows, Docker is usually the best documented route.

 - If you need interactive notebooks and visual inspection, `_lunapi_` plus JupyterLab is often a better fit than plain `luna.exe`.


## Bottom line

For Windows users, there are really three practical tiers:

 - use the native Windows binaries plus Command Prompt for straightforward `luna` and `destrat` usage

 - use [Docker](../download/docker.md) if you want the most complete and reproducible Luna environment

 - use [`_lunapi_`](../lunapi/index.md) and [`scope`](../lunapi/scope.md) if you want an interactive notebook-based workflow, potentially via JupyterHub if your site already supports it

If you are unsure where to start, Docker is usually the safest default.
