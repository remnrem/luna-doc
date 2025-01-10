# Luna + Python = _lunapi_

_lunapi_ is a Python module that provides an interface to Luna. It accesses the C/C++ Luna library directly, meaning
all core Luna commands described [here](../ref/index.md) have similar syntax and performance;
many of the fundamental concepts described [here](../luna/args.md) apply here too.

## Installation 

To obtain _lunapi_ (macOS, Linux or Windows) use `pip`:

```
pip install lunapi
```

Alternatively, you can pull the [lunapi Docker
image](#https://github.com/remnrem/luna-api-notebooks?tab=readme-ov-file#docker-installation)
which also provides a Jupyter lab environment (as well as the
command-line Luna and R-based _lunaR_ tools) in a single package.


## Getting started

 - Follow the [example](https://github.com/remnrem/luna-api-notebooks/blob/main/00_overview.ipynb) and [_lunapi_ tutorial](https://github.com/remnrem/luna-api-notebooks/blob/main/tutorial.ipynb) notebooks from [this repository](https://github.com/remnrem/luna-api-notebooks/)

 - See the [primary reference](ref.md) and [scope viewer](scope.md) pages

## Known issues

 - [Jupyter Lab](https://jupyter.org/) is required for the `scope` viewer

 - Using `ctrl-D` or `ctrl-C` to escape from long-running Luna
   processes may be slow

 - On some platforms, commands may run more slowly under the Jupyter
   Lab environment compared to a plain Python environment (which gives
   comparable performance to the command-line Luna). This may be due
   to suboptimal configuration settings, but it is beyond the scope of
   this documentation to advise for specific cases.  In general, the
   notebooks are best suited for smaller, interactive jobs rather than
   more intensive processing.
 


