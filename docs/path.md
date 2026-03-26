# Path

_A practical route through the Luna documentation for first-time users_

If you are new to Luna, do **not** start by reading the full command reference
from top to bottom. Luna has a lot of functionality, and the reference pages are
best used once you already know the basic workflow and are looking up a specific
command, parameter, or output.

Luna can be used in several different ways, but they all sit on top of the
same core library and commands. The command-line interface is usually the best
fit for users who want explicit scripts, batch processing, reproducibility, and
direct access to the full Luna command language. [lunapi](lunapi/index.md) is
usually the best fit for users who prefer Python, notebooks, interactive
analysis, and tighter integration with plotting or downstream data-science
tools. [LunaScope](https://zzz.nyspi.org/lunascope/) is the most approachable
entry point for users who want visual review and point-and-click exploration
before moving to scripts or notebooks. There is also an R interface
([lunaR](ext/R/index.md)), but this page focuses on the main current paths:
command-line Luna, lunapi, and LunaScope.

## Path

### Command Line

For users who want explicit scripts, batch processing, sample lists, and direct
access to the full Luna command language.

1. [Downloads and installation](download/index.md)
   Install the command-line Luna tools.
2. [Quick start](tut/tut1.md)
   Learn the basic mechanics on a small example.
3. [Take two](tut/tut2.md)
   Continue with projects, outputs, and parameterization.
4. [Deeper dive](tut/tut3.md)
   Add masks, hypnograms, artifacts, and more realistic workflows.
5. [Concepts and syntax](luna/args.md)
   Read the core command model once the tutorial gives it context.
6. [Walk-through](https://zzz.nyspi.org/luna-walkthrough)
   Follow the more realistic end-to-end analysis path.
7. [Command reference overview](ref/index.md)
   Use the domain pages and command docs as needed.

### Python

For users who prefer notebooks, interactive analysis, Python scripts, and
integration with plotting or downstream data-science tools.

1. [lunapi overview and installation](lunapi/index.md)
   Install the Python interface and see what it provides.
2. [Quick start](tut/tut1.md)
   Learn the shared Luna concepts on the main tutorial data.
3. [Take two](tut/tut2.md)
   Continue with outputs, projects, and parameterization.
4. [Deeper dive](tut/tut3.md)
   Build the core Luna workflow before switching interfaces.
5. [lunapi getting started](lunapi/index.md#getting-started)
   Move into the Python-specific workflow.
6. [Concepts and syntax](luna/args.md)
   Keep the core command model in view.
7. [Walk-through](https://zzz.nyspi.org/luna-walkthrough)
   Use the main walk-through for the overall analysis logic.
8. [lunapi reference](lunapi/ref.md)
   Look up Python details when needed.

### LunaScope

For users who want point-and-click visual review and an easier first entry
point before moving into scripts or notebooks.

1. [LunaScope](https://zzz.nyspi.org/lunascope/)
   Start with the GUI and overall orientation.
2. [Downloads and installation](download/index.md)
   Install the relevant Luna components.
3. [Quick start](tut/tut1.md)
   Learn the shared Luna ideas on a small example.
4. [Take two](tut/tut2.md)
   Continue with projects, outputs, and annotations.
5. [lunapi overview](lunapi/index.md)
   Understand the Python layer that LunaScope builds on.
6. [Concepts and syntax](luna/args.md)
   Learn the underlying Luna model behind the GUI.
7. [Walk-through](https://zzz.nyspi.org/luna-walkthrough)
   See the fuller end-to-end analysis workflow.
8. [scope viewer](lunapi/scope.md)
   Explore the related interactive viewer documentation.

## Tutorial and Walk-through

The tutorial and the walk-through are the main guided entry points into Luna.
They introduce the basic workflow and the concepts that the reference pages
assume:

- sample lists and projects
- signals, annotations, epochs, and masks
- commands and parameters
- output tables and stratification
- the overall analysis loop of inspect, analyze, extract, iterate

In practice:

- the [tutorial](tut/tut1.md) is the best first-pass introduction
- the [walk-through](https://zzz.nyspi.org/luna-walkthrough) is the best
  practical next step after that

## Concepts

After the tutorial, read these pages before diving into individual commands:

- [Key concepts & syntax](luna/args.md)
- [Command reference overview](ref/index.md)
- [Output and destratification](luna/destrat.md)

These three pages explain how Luna thinks about:

- sample lists and projects
- commands and parameters
- epochs, masks, annotations, and signals
- output tables and stratification

Without that framing, the detailed command pages can feel much denser than they
actually are.

## Reference

The reference pages are excellent when you already know roughly what you want to
do. They are less effective as a first read.

Use them when one of these is true:

- you know the command name and need parameters or outputs
- you want to compare related commands
- you are debugging an analysis and need exact field definitions
- you are working in one domain, such as artifacts, hypnograms, or spectra

Useful starting points:

- [All commands and domains](ref/index.md)
- [Annotations](ref/annotations.md)
- [Masks](ref/masks.md)
- [Artifacts](ref/artifacts.md)
- [Hypnograms](ref/hypnograms.md)
- [Spectral analyses](ref/power-spectra.md)
