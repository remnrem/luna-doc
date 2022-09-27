# Staging: SOAP

_Self-contained modelling and evaluation of sleep staging_

This page is under development: please see this [vignette](../vignettes/soap-pops.md) for more details.  

| Command | Description | 
| ---- | ------ | 
| [`SOAP`](#soap)     | Single-observation accuracies and probabilities |
| [`REBASE`](#rebase) | Use SOAP to translate between epoch lengths (e.g. 20 to 30 second manual staging) | 
| [`PLACE`](#place)   | Use SOAP to localize "lost" stage annotations |

This suite of commands are based around the _SOAP_ (Single Observation
Accuracies and Probabilities) function in Luna.  In brief, SOAP
generates a battery of epochwise features based on one or more
channels, and then fits a simple (linear discriminant analysis) to
those data, which the observed stages (i.e. passed as annotations) as
the outcomes.  If the signal is a) of sufficient quality, and b) is
related to observed sleep stages via the computed features, then SOAP
should be able to generate a model that explains the observed stages
quite well.  This can be assessed, for example, via the kappa
coefficient between observed and predicted stages.

!!! info The SOAP command requires existing stage annotation,
    i.e. similar to the `HYPNO` command (whether these existing
    annotations are from manual scoring, or from an external,
    automated staging algorithm).  That is, it does _not_ predict
    sleep stages from scratch, given only signal data.  For automated
    staging, see Luna's [`POPS`](pops.md) command.


Overall, SOAP can be viewed in two ways: 1) as a tool to check the
consistency between signals & staging, 2) as a tool to manipulate/use
the existing staging, _on the assumption that stages and signals are
largely consistent_.


## SOAP

_Single observation stage accuracies and probabilities_

<h3>Parameters</h3>

_to be completed_

<h3>Output</h3>

_to be completed_

<h3>Example</h3>

_to be completed_


## REBASE

_Translate existing (manual) staging between different epoch durations (e.g. from 20 second to 30 seconds epoch) using the SOAP model_

<h3>Parameters</h3>

_to be completed_

<h3>Output</h3>

_to be completed_

<h3>Example</h3>

_to be completed_


## PLACE

_Temporally align existing staging signal data_

<h3>Parameters</h3>

_to be completed_

<h3>Output</h3>

_to be completed_

<h3>Example</h3>

_to be completed_

