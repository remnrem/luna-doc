# Luna + Python = _lunapi_

_lunapi_ is a Python module that provides an interface to Luna. It accesses the C/C++ Luna library directly, meaning
all core Luna commands described [here](../ref/index.md) have similar syntax and performance;
many of the fundamental concepts described [here](../luna/args.md) apply here too.

!!! hint "LunaScope"
    [LunaScope](https://zzz.nyspi.org/lunascope/) is a standalone desktop viewer built on top of _lunapi_. For interactive visual review, it is generally the better and more full-featured tool; the `scope` utility described here is a smaller embedded viewer intended for use inside JupyterLab notebooks.

!!! tip "Luna In Your Browser"
    You can also try `_lunapi_` in your browser, linked to example data and
    interactive notebooks, via this [Binder-hosted cloud
    instance](https://mybinder.org/v2/gh/remnrem/luna-api-notebooks/HEAD?urlpath=%2Fdoc%2Ftree%2F00_overview.ipynb).

## Installation 

To obtain _lunapi_ (macOS, Linux or Windows) use `pip`:

```
pip install lunapi
```

## Command-line use

The `lunapi` package also provides `luna.py`, a Python command-line variant
for the standard sample-list processing workflow. It uses the same underlying
Luna implementation as the Python API and accepts a sample list or a single
EDF, Luna variables, parameter files, and a command string. It is intended as
a drop-in replacement for common `lunaC` jobs; use `lunaC` for standalone
options that are not part of this interface (for example, `--xml`, `--merge`,
or `--append`).

```bash
luna.py s.lst -o out.db sig=EEG @params.txt -s "EPOCH len=30 PSD"
```

The command string can be read from standard input instead of using `-s`:

```bash
cat commands.txt | luna.py s.lst -o out.db
```

To process one EDF directly:

```bash
luna.py recording.edf -o out.db -s "HEADERS"
```

Rows can be selected by 1-based row number, inclusive range, Luna-style slice,
or sample-list ID:

```bash
luna.py s.lst 3 -o out.db -s "HEADERS"
luna.py s.lst 10-20 -o out.db -s "STATS"
luna.py s.lst 2/5 subject_001 -o out.db -s "DESC"
```

The helper modes `--build` and `--validate` use Luna's native sample-list
builder and validator:

```bash
luna.py --build /data/study > s.lst
luna.py --validate s.lst
```

Run `luna.py --help` for the complete list of supported options. The command
is installed alongside `lunapi` by `pip install lunapi` and is named
`luna.py` so that it does not conflict with the native `lunaC` executable.

The package also installs `destrat.py`, a Python/SQLite reader for Luna output
databases. It accepts one or more database paths, glob patterns, a command
selector, row and column stratifiers, variable and individual filters, and
level restrictions. For example:

```bash
destrat.py out.db
destrat.py out.db +PSD -r CH F -v PSD
destrat.py out/run-*.db +PSD -r F/11,15 CH -c B -v PSD
```

Results are written as tab-delimited text to standard output. Use
[`lp.destrat()`](ref.md#lpdestrat) when the same queries should be performed
inside Python and returned as pandas dataframes.

Alternatively, you can pull the [lunapi Docker
image](https://github.com/remnrem/luna-api-notebooks?tab=readme-ov-file#docker-installation)
which also provides a Jupyter lab environment (as well as the
command-line Luna and R-based _lunaR_ tools) in a single package.


## Getting started

 - Follow the [example](https://github.com/remnrem/luna-api-notebooks/blob/main/00_overview.ipynb) and [_lunapi_ tutorial](https://github.com/remnrem/luna-api-notebooks/blob/main/tutorial.ipynb) notebooks from [this repository](https://github.com/remnrem/luna-api-notebooks/)

 - See the [primary reference](ref.md) and [scope viewer](scope.md) pages
 - For a standalone desktop viewer built on top of _lunapi_, see [LunaScope](https://zzz.nyspi.org/lunascope/)

## Known issues

 - [Jupyter Lab](https://jupyter.org/) is required for the `scope` viewer
 - For most interactive signal viewing tasks, [LunaScope](https://zzz.nyspi.org/lunascope/) is a better choice than the notebook-embedded `scope` widget

 - Using `ctrl-D` or `ctrl-C` to escape from long-running Luna
   processes may be slow

 - On some platforms, commands may run more slowly under the Jupyter
   Lab environment compared to a plain Python environment (which gives
   comparable performance to the command-line Luna). This may be due
   to suboptimal configuration settings, but it is beyond the scope of
   this documentation to advise for specific cases.  In general, the
   notebooks are best suited for smaller, interactive jobs rather than
   more intensive processing.
 
