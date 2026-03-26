# About

Background and project information for Luna.

## What

Luna is a C/C++ library focused on the analysis of sleep studies, primarily
encoded as EDF/EDF+ files. This is a free, open-source project. The main
interfaces are the command-line tool ([_lunaC_](luna/args.md)), the Python
interface ([_lunapi_](lunapi/index.md)), and the interactive GUI
([_LunaScope_](https://zzz.nyspi.org/lunascope/)). There is also an R
interface ([_lunaR_](ext/R/index.md)).

## Version

The current version is __v1.3.4 (27-Feb-2026)__.
Use `luna -v` to display the specific build date/time.

## Where

Luna is developed at [Columbia University Irving Medical
Center](https://www.cuimc.columbia.edu/) and the [New York State
Psychiatric Institute](https://www.nyspi.org/), New York, NY, United States.

## Who

Luna was primarily developed by Shaun Purcell, with input from
a number of colleagues:

- Tejas Karkera for code maintenance, algorithmic testing, and related development support
- Lorcan Purcell for work on the LunaScope GUI codebase
- Senthil Pananivelu for maintaining distributions and work on Moonlight
- Nataliia Kozhemiako for input into multiple EEG analytic components and the revised artifact detection workflows
- Shyamal Agarwal for work on automating the build distribution and NSRR's Automated Pipeline (NAP) built around Luna
- Michael Rueschman for input on Moonlight
- Alexander Kent for testing and feedback
- Susan Redline and her team developing the [National Sleep Research Resource](http://sleepdata.org)
- Dennis Dean for sharing his original SpectralTrainFig code-base
- Sara Mariani and Charmaine Demanuele for input on several EEG and ECG analysis components

Interested to contribute (either as a colleague or as a job)? Please
[contact me](http://zzz.nyspi.org/index.html#contact).

## Support

Luna development is indirectly supported via a number of NIH grants: NHLBI
R01HL146339 (PI Purcell), NHLBI R21HL145492 (PI Purcell), NIMH R03
MH108908 (PI Purcell), as well as NHLBI R35HL135818 (PI
Redline) and NHLBI R24HL114473 (PI Redline).

## Why

The primary aim of Luna was to provide a platform for bringing methods and
models from cognitive neuroscience into the setting of large, sometimes noisy,
polysomnographic datasets.

My own background is primarily in
[psychiatric genetics](http://zzz.nyspi.org/publications.html), and the
development of Luna has tracked with my learning curve in how to think about
sleep signal data. I built it using the tools I knew best, namely
[C/C++](https://en.wikipedia.org/wiki/C%2B%2B) and
[R](http://www.r-project.org), rather than the more typical in-house
[Matlab](https://www.mathworks.com/products/matlab.html) script. At the same
time, developing Luna has only increased my appreciation for how powerful the
Matlab ecosystem can be for electrophysiological analysis.

What still seemed missing, however, was a robust way to work with sleep data in
[thousands of individuals](https://www.ncbi.nlm.nih.gov/pubmed/28649997), such
as from the [NSRR](http://sleepdata.org). In many settings, the core analysis
itself might be one or two well-established signal-processing steps, but the
practical scaffolding around that analysis was often brittle, error-prone, and
poorly documented. Luna grew out of a desire to make that surrounding workflow
more structured, scalable, and transparent.

Luna began as a personal library for my own research, but I decided to
document and distribute it for a few reasons:

- _to make the tool better:_ documenting and distributing code usually improves the underlying tool, even if only a small number of people ever use it
- _accessibility and transparency:_ open tools and open formats make it easier for others to see exactly what was done and to reuse or adapt it
- _community:_ others can build on the work; with tools such as [PLINK](https://www.cog-genomics.org/plink/1.9/), I have seen how an open user and developer community can extend a project far beyond its original scope

For both large and small projects, I think that document-and-distribute model
is worth following whenever practical.

## Acknowledgments

Luna uses a number of excellent open-source components, in particular:

- [FFTW](http://www.fftw.org) library
- [SQLite](https://www.sqlite.org/index.html) embedded database
- [R Project for Statistical Computing](http://www.r-project.org)
- [Python](http://python.org) and the [JupyterLab](http://jupyter.org) framework
- [Eigen](https://eigen.tuxfamily.org/index.php?title=Main_Page) C/C++ matrix/linear algebra library
- [LightGBM](https://lightgbm.readthedocs.io/en/v3.3.5/index.html) gradient boosting, tree-based learning algorithm library
- Chapters and example code from [Mike X Cohen](http://mikexcohen.com)'s book: _Analyzing neural time series data_
- Lees, J. M. and J. Park (1995): Multiple-taper spectral analysis: A stand-alone C-subroutine: Computers & Geology: 21, 199
- Laurent Condat (2013) A Direct Algorithm for 1-D Total Variation Denoising. IEEE Signal Processing Letters, 20:11
- [Multi-scale entropy (MSE) algorithm](https://physionet.org/physiotools/mse/) by Madalena Costa et al. (Costa M., Goldberger A.L., Peng C.-K. Multiscale entropy analysis of biological signals. Phys Rev E 2005;71:021906.)
