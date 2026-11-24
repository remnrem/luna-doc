# lunaR installation

The R package **`luna` 1.8.0** uses
[lunapi](../lunapi/index.md) through reticulate and follows its
project/instance API. Install it from
[`remnrem/lunaR`](https://github.com/remnrem/lunaR), then load it with
`library(luna)`.

## Install and start Luna

Use **R 4.1 or later**. You do not need a compiler, FFTW, or a separate
Luna executable.

In R, run:

```r
install.packages("pak")
pak::pak("remnrem/lunaR")
library(luna)
p <- lp$proj()
luna_config()
```

On first use, reticulate sets up Python and lunapi automatically. Keep
an internet connection available and allow extra time for this first
run. Later sessions reuse the downloaded environment.

If `luna_config()` shows `status : ok`, you are ready to use Luna.
It also shows the Python path and versions in use. If setup fails, see
[Fixing a missing or outdated lunapi](#fixing-a-missing-or-outdated-lunapi).

This installs the current GitHub version. For a fixed 1.8.0 installation,
see [Specifying particular versions](#specifying-particular-versions).

Supported platforms are macOS Intel and Apple Silicon, Linux x86_64
with glibc 2.28 or later, and Windows x86_64. Python 3.9–3.14 is supported.

## Already have Python or lunapi?

If you do not need to reuse an existing Python environment, use the
commands above and let reticulate handle setup.

If you already have lunapi in a virtual environment, tell reticulate
where it is using one of the methods below. If that environment lacks
lunapi, follow the [installation fix below](#fixing-a-missing-or-outdated-lunapi).

## Using an existing virtual environment

**Restart R first.** Select your environment before `library(luna)` or
any other Python use. R can use only one Python per session.

These examples use `~/.venv` on macOS/Linux. Replace it with your own
virtual environment's path.

### Use it every time

Add this line to `~/.Renviron` (create the file if needed), then restart R:

```text
RETICULATE_PYTHON=~/.venv/bin/python
```

Then run:

```r
library(luna)
luna_config()  # python should show the full path to ~/.venv/bin/python
p <- lp$proj()
```

To change environments later, edit or remove that line and restart R.

### Use it for one session

In a fresh R session, run:

```r
reticulate::use_virtualenv("~/.venv", required = TRUE)
library(luna)
luna_config()
p <- lp$proj()
```

Or select the Python executable directly, as shown in Luna's error message:

```r
reticulate::use_python("/path/to/venv/bin/python", required = TRUE)
library(luna)
luna_config()
p <- lp$proj()
```

Choose one method. If `RETICULATE_PYTHON` is already set in `.Renviron`,
change or remove it before selecting a different environment for one session.

### Starting R from a terminal or RStudio

Starting R from a terminal after activating your virtual environment
usually works too. RStudio launched from the macOS Dock does not normally
see that terminal's activated environment, so use the `.Renviron` setting
for a consistent setup.

On Windows, use the environment's `Scripts/python.exe` path in
`.Renviron`, or pass its directory to `use_virtualenv()`.

When you select your own Python, Luna expects lunapi to be installed
there. It will not automatically add lunapi to that environment.

## Fixing a missing or outdated lunapi

If you see **"lunapi is not available in the Python that R is using"**,
check the Python path in the error. That is the environment that needs
lunapi, even if you have already installed it somewhere else.

Choose one fix:

1. **Use a Python that already has lunapi.** Restart R and select it with
   `use_python()` or `.Renviron`, as shown above.
2. **Install lunapi into the Python shown in the error.** In the R session
   that produced the error, run:

```r
install_lunapi()
```

When installation finishes, restart R and run:

```r
library(luna)
luna_config()
p <- lp$proj()
```

Use the same fixes if Luna reports that lunapi is too old.

If you prefer to install from a terminal, use the Python path shown in
the error. For the example `~/.venv` environment, choose one:

```bash
# Fixed version
~/.venv/bin/python -m pip install lunapi==1.8.0

# Latest compatible version
~/.venv/bin/python -m pip install --upgrade 'lunapi>=1.8.0,<2'
```

Then restart R and check again with `luna_config()`.

If you want to turn off automatic setup, set
`options(luna.auto_install = FALSE)` or `LUNA_AUTO_INSTALL=false` before
loading `luna`. Select your Python environment and install lunapi there
using the steps above.

## Install from a local source checkout

Choose one command in a terminal:

```bash
# Fixed release
git clone --branch v1.8.0 https://github.com/remnrem/lunaR.git

# Current development version
git clone https://github.com/remnrem/lunaR.git
```

Then install the checkout from R:

```r
install.packages("pak")
pak::local_install("lunaR")
```

Replace `"lunaR"` with the checkout's full path if it is not in your
current R working directory. Then follow the startup steps above.

## Specifying particular versions

!!! info "Install version 1.8.0"
    To install the R package at a fixed version:

    ```r
    pak::pak("remnrem/lunaR@v1.8.0")
    ```

    To also use lunapi 1.8.0 with automatic Python setup, restart R and run:

    ```r
    reticulate::py_require("lunapi==1.8.0")
    library(luna)
    p <- lp$proj()
    luna_config()
    ```

    If you selected your own Python environment, use the fixed-version
    pip command above to install lunapi 1.8.0 there instead.

## Upgrading from the old R interface

Install from `remnrem/lunaR`, restart R, and keep using `library(luna)`.
The API uses `p <- lp$proj()` and methods such as `p$inst()` and
`i$eval()`. For scripts written before 1.8.0, see the
[migration guide](../ext/R/ref.md#migrating-old-r-scripts).

For current examples, see the [R worked example](../ext/R/example.md),
[R reference](../ext/R/ref.md), and R help (`?lp`, `?proj`, `?inst`,
`?ProcResult`). The [/R/epeat tutorial](../tut/tut4.md) covers the full
tutorial workflow in R. The [lunapi reference](../lunapi/ref.md) describes
the corresponding Python methods; in R, use `$` in place of `.`.

To uninstall, run `remove.packages("luna")`.
