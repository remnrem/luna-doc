# Known issues

- although we should be able to read gzipped-compressed plain text
  files as inputs, Luna will truncate the last few lines currently:
  need to fix

- check/document that `SPINDLES` command can use `fwhm` parameterization

- empirical threshold setting for `SPINDLES` appears broken in v0.24

- documnetation: need to document `cstats`, `astats` etc for
  `SIGSTATS` (and `chep` and its interaction with `INTERPOLATE`; need
  to update documentation on various new v0.24 features (e.g. use of
  `file` for `FILTER` etc).  Many of these things will be covered by
  the upcoming _vignettes_ however, and we will add then.



