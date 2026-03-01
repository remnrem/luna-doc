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
 
