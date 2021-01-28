# dmerge example

Below is a toy example of using `dmerge`, specifically designed to
illustrate some features and the design logic.  That is, for didactic
reasons, this example is written to _not_ work "out of the box", but
rather to flag the types of things that might be encountered on real
data.

## Data dictionaries

In this example we assume a single _domain_ (just called `d1`) and
two _groups_ (here `g1` and `g2`).  The data dictionaries are under a
top-level folder `dict/`:

```
ls dict/
```
```
d1_g1.txt       d1_g2.txt
```

The first group `g1` has two variables `V1` and `V2` along with two
stratifying _factors_ (i.e. effectively defining the repeated measures
of those variables for each individual):

```
cat dict/d1_g1.txt
```
```
F1      factor  Factor 1
F2      factor  Factor 2
V1      num     Var 1
V2      num     Var 2
```

The second group, `g2`, also has variables named `V1` and `V9`, along with factors `F1`, `F2` and also `SS`

```
cat dict/d1_g2.txt
```
```
SS      factor  Sleep stage
F1      factor  Factor 1
F2      factor  Different label
V1      num     Var 1
V9      int     More variables
```


!!! hint
    Factors can be common across domains and groups,
    as these will contain common elements (channel, sleep stage, etc).

    As we'll see below, variables (i.e. corresponding to the actual
    information) are defined as being specific to a domain/group, and it
    would be a problem to have the same label e.g. `N` for count appear in
    different contexts (i.e. different domains/groups) if it means
    different things (e.g. number of OSA events, number of arousals,
    number of spindles, etc).  Thus, as we'll see, `dmerge` will spot
    this potential problem.


## The data

In this example we have two individuals, `id1` and `id2`.  We require
that each individual has one (or more) subfolders under a root
_project_ folder.


```
$ ls studies/
id1     id2
```

The first individual has two data files:

```
ls studies/id1
```
```
d1_g1_s1_F1_F2.txt      d1_g2_s1_SS-N2.txt
```

The second individual only has one:

```
ls studies/id2/
```
```
d1_g1_s1_F1_F2.txt
```

The filename convention tells us:

- which domain/group each dataset belongs, i.e. `d1_g1` or `d1_g2`

- the tag is just set to `s1` in these examples, as there is only a single file per domain/group in this trivial example

- additional underscore-delimited terms reflect the _factors_ that apply for that dataset, e.g. `F1`, `F2`

- in one case, a level value `N2` is also ascribed to a factor (`SS`), implying that all rows should be assigned to this stratum, and that there is no column `SS` in the datafile itself;  i.e. in real data, other files might imply different strata
 
Looking at the actual data:

```
$ cat studies/id1/d1_g1_s1_F1_F2.txt
```
```
ID      F1      F2      V1      V2
id1     A       X       1       2
id1     A       Y       3       4
id1     B       Y       5       U
```

Note here that `V2` has a missing value set as a nonstandard value
`U`.  Certain values (`.`, `?`, `NA`, `NaN`, etc) are all treated as
missing, but (given that `V2` has a numeric _type_ defined, this value
will cause a problem downstream, as we'll see.


```
$ cat studies/id1/d1_g2_s1_SS-N2.txt
```
```
ID      V1      V9      V10
id1     10      .       11
id1     NA      12      13
id1     14      15      ?
```

In the second data file for `id1` (above), we have only standard missing value codes.  Note a
variable `V10` that was not defined in the corresponding data
dictionary. Also note that `V1` is present here, but has different
values from the first file.  Also note that in this case, there are no
stratifying variables included, but multiple (different) values of the
same variable for the same individual, which is also clearly a
problem (i.e. somebody forgot to include the relevant stratifiers in this output
to distinguish rows 1, 2 and 3.


Finally, here we data for the second individual.
```
$ cat studies/id2/d1_g1_s1_F1_F2.txt
```
```
ID      F1      F2      V1      V2
id2     A       X       7       8
id2     B       Y       9       10
```




## Running dmerge

Here the various runs needed to get a final results are done to illustrate a couple of features of `dmerge`.

### First run (namespaces of factors and variables)

This is the initial run of `dmerge` - which just points to the data dictionary folder (`dict/`) and the study data folder (`studies/`) and gives a file to be created, for the output to be collated in, `s1.txt`:

```
dmerge dict studies s1.txt
```
```
 ++ adding domain d1::g1 (4 variables)
 ++ adding domain d1::g2 (5 variables)

*** error : inconsistent label for factor F2 across data dictionaries
```

We get an error as `dmerge` notices that the `F2` factor is defined different (different descriptions) across dictionaries.  In trying to harmonize data-files, this is obviously a problem: which should be used?    This error therefore alerts the user to this issue, which has to be fixed.   Looking up the relevant lines (which might not be trivial in a large project) here with `grep`:

```
grep F2 dict/*
```
```
dict/d1_g1.txt:F2       factor  Factor 2
dict/d1_g2.txt:F2       factor  Different label
```

So, either in one or the other of these files, you must make the
description label identical, to enforce consistency across datafiles.
This avoids, for example, `S` meaning _signal_ in one set of outputs,
but _sleep stage_ in a second, which would lead to downstream
problems.

### Second run (introducing aliases)

Trying again, we get a new error:

```
dmerge dict studies s1.txt
```
```
 ++ adding domain d1::g1 (4 variables)
 ++ adding domain d1::g2 (5 variables)

*** error : V1 is duplicated across data dictionaries
```

Looking up this variable across dictionaries:
```
grep V1 dict/*
```
```
dict/d1_g1.txt:V1       num     Var 1
dict/d1_g2.txt:V1       num     Var 1
```

As noted above, although the labels are identical, this is
purposefully not allowed by the tool: variables are _by definition_
specific to a domain and the names should be unique to a domain.

!!! hint
    The same variable name can exist in diffferent data files within the same domain/group.
    e.g. `PSD` might exist in two files

    `eeg_spec_avg_B.txt`

    `eeg_spec_avg_B_SS.txt`

    meaning that this measure is stratified by either _band_ (`B`) or
    by both _band_ and _sleep stage_ (`SS`).  The variable `PSD` would
    only feature once in the data dictionary `eeg_spec.txt` (along
    with factor definitions for `B` and `SS`), and the the program
    would correctly pull these together.

It would be burdensome to have to go back to the original files (which
may have been generated by different tools/people, and may large and
not easy to edit, etc) and so `dmerge` allows for _aliases_ to be
defined on the data dictionaries. For one of these domain/groups, we
can effectively relabel the variable `V1` to something else when
harmonizing.   Here we edit `dict/d1_g2.txt`, to change the line:

```
V1     num   Var 1
```
to become two lines
```
V1b   num     A new var 1
V1    alias   V1b
```

That is, we first define a new variable `V1b`, which is specific to
this domain, and then define an _alias_ for `V1b` which is `V1`.  This
means that for any `g1_d2` datafile, any instance of `V1` is treated
as if it were written `V1b`, and this avoids any potential naming
conflicts.


### Third run (missing data codes)

Having fixed the above, we re-run:
```
dmerge dict studies s1.txt > s1.dict
```
```
 ++ adding domain d1::g1 (4 variables)
 ++ adding domain d1::g2 (5 variables)
 ++ read 3 rows from data-file studies/id2/d1_g1_s1_F1_F2.txt
      domain    [ d1 ]
      group     [ g1 ]
      file-tag  [ s1 ]
      variables [ V1 | V2 ]
      factors   [ F1 | F2 ]

*** error : invalid value [U] for V2 (type Numeric)
	in: studies/id1/d1_g1_s1_F1_F2.txt
```

As noted above, this illustrates the simple type-checking features,
spotting that `U` is not a valid numeric value.  If we know it is a
missing code used in the data file, we can add a line to the
dictionary `dict/d1_g1.txt`: (here just two tab-delimited cols):
```
  missing       U
```

### Fourth run (multiple conflicting values / missing strata) 

Running again, we now see a new error:

```
dmerge dict studies s1.txt
```
```
 ++ adding domain d1::g1 (4 variables)
 ++ adding domain d1::g2 (5 variables)
 ++ read 3 rows from data-file studies/id2/d1_g1_s1_F1_F2.txt
      domain    [ d1 ]
      group     [ g1 ]
      file-tag  [ s1 ]
      variables [ V1 | V2 ]
      factors   [ F1 | F2 ]
 ++ read 4 rows from data-file studies/id1/d1_g1_s1_F1_F2.txt
      domain    [ d1 ]
      group     [ g1 ]
      file-tag  [ s1 ]
      variables [ V1 | V2 ]
      factors   [ F1 | F2 ]

*** error : multiple values for id1 V9.SS_N2
```

This flags the issue we spotted above: in the data file. (The tool will spot if there are duplicate discordant values spread across multiple files also.)  In this instance, it is clear this is due to a stratifying factor not being included in the file:

```
cat studies/id1/d1_g2_s1_SS-N2.txt
```
```
ID      V1      V9      V10
id1     10      .       11
id1     NA      12      13
id1     14      15      ?
```

If we were to go back and correct the original data, which would be necessary here, say we instead have this:

```
cat studies/id1/d1_g2_s1_SS-N2.txt
```
```
ID      V1      V9      V10     F2
id1     10      .       11      X
id1     NA      12      13      Y
id1     14      15      ?       Z
```

i.e. we've made each row unique by adding the missing factor, `F2`,
and so there should be no conflicts now.  However, if we were to
re-run as is, we'd still get the same error.  Why?  This is because
the filename convention has not specified that `F2` is a factor for
this file.

!!! note
    Yes, that `F2` is a factor could be inferred from
    `dict/d1_g2.txt`, but the tool purposefully has the model that the
    dictionaries must contain a complete representation of _the truth_
    of the data, but also requires a second level of consistency (here
    that filenames match). The design logic is that, at the cost of a
    marginally more involved set-up, it makes it more robust
    downstream, and less likley to have subtle errors when merging
    across different datafiles.
    

We therefore would need to also change the name of the datafile as
well as the contents to reflect the status of `F2`:
```
mv studies/id1/d1_g2_s1_SS-N2.txt studies/id1/d1_g2_s1_F2_SS-N2.txt
```

### Fifth run: validation

The fifth run will now work.    Again, that it "failed" the first four times is not reflecting
problems with the tool -- rather, think of it as giving feedback to enfore a set of conventions that
help for data harmonization.

```
dmerge dict studies s1.txt > s1.dict
```
```
 ++ adding domain d1::g1 (4 variables)
 ++ adding domain d1::g2 (5 variables)
 ++ read 3 rows from data-file studies/id2/d1_g1_s1_F1_F2.txt
      domain    [ d1 ]
      group     [ g1 ]
      file-tag  [ s1 ]
      variables [ V1 | V2 ]
      factors   [ F1 | F2 ]
 ++ read 4 rows from data-file studies/id1/d1_g1_s1_F1_F2.txt
      domain    [ d1 ]
      group     [ g1 ]
      file-tag  [ s1 ]
      variables [ V1 | V2 ]
      factors   [ F1 | F2 ]
 ++ read 4 rows from data-file studies/id1/d1_g2_s1_F2_SS-N2.txt
      domain    [ d1 ]
      group     [ g2 ]
      file-tag  [ s1 ]
      variables [ V1B | V9 | V10 (skipped) ]
      factors   [ F2 | SS = N2 ]

finished: processed 2 individuals across 3 files, yielding 12 (expanded) variables
```

Here we've also saved the data dictionary to a file `s1.dict` as well as the actual data, in `s1.txt`.

For reference, the final data dictionaries and files are:

```
cat dict/d1_g1.txt
```
```
F1      factor  Factor 1
F2      factor  Factor 2
V1      num     Var 1
V2      num     Var 2
missing U
```

```
$ cat dict/d1_g2.txt
```
```
SS      factor  Sleep stage
F1      factor  Factor 1
F2      factor  Factor 2
V1b     num     A new var 1
V1      alias   V1b
V9      int     More variables
```

```
cat studies/id1/d1_g1_s1_F1_F2.txt
```
```
ID      F1      F2      V1      V2
id1     A       X       1       2
id1     A       Y       3       4
id1     B       Y       5       U
```


```
cat studies/id1/d1_g2_s1_F2_SS-N2.txt
```
```
ID      V1      V9      V10     F2
id1     10      .       11      X
id1     NA      12      13      Y
id1     14      15      ?       Z
```


```
cat studies/id2/d1_g1_s1_F1_F2.txt
```
```
ID      F1      F2      V1      V2
id2     A       X       7       8
id2     B       Y       9       10
```


We can look at the `s1.txt` file (here using Luna's `behead` utility to make it more human readable):

```
cat s1.txt | behead
```
```
                       ID   id1
             V1.F1_A_F2_X   1
             V1.F1_A_F2_Y   3
             V1.F1_B_F2_Y   5
           V1B.F2_X_SS_N2   10
           V1B.F2_Y_SS_N2   NA
           V1B.F2_Z_SS_N2   14
             V2.F1_A_F2_X   2
             V2.F1_A_F2_Y   4
             V2.F1_B_F2_Y   NA
            V9.F2_X_SS_N2   NA
            V9.F2_Y_SS_N2   12
            V9.F2_Z_SS_N2   15

                       ID   id2
             V1.F1_A_F2_X   7
             V1.F1_A_F2_Y   NA
             V1.F1_B_F2_Y   9
           V1B.F2_X_SS_N2   NA
           V1B.F2_Y_SS_N2   NA
           V1B.F2_Z_SS_N2   NA
             V2.F1_A_F2_X   8
             V2.F1_A_F2_Y   NA
             V2.F1_B_F2_Y   10
            V9.F2_X_SS_N2   NA
            V9.F2_Y_SS_N2   NA
            V9.F2_Z_SS_N2   NA
```

Note that `V10` is not present.  This was noted in the console output above:
```
      variables [ V1B | V9 | V10 (skipped) ]
```

i.e. as `V10` was not described in any data dictionary (or it was
commented out, or the domain/group was skipped on the command line):

We can run `dmerge` in _strict mode_, where there must be an exact one-to-one matching between the contents of the data dictionaries and the contents of the data files, using the `-s` option:

```
dmerge dict studies s1.txt -s > s1.dict
```
This would indeed yield the error:
```
*** error : V10 not specified in data-dictionary for studies/id1/d1_g2_s1_F2_SS-N2.txt
```

## Generated data-dictionaries

`dmerge` generates a bespoke data dictionary to exactly match the
output `s1.txt`.  We saved it as `s1.dict`, it is a tab-delimited
tabular file, e.g. that can easily be loaded into R, etc.

```
COL     VAR     BASE    OBS     DOMAIN  GROUP   TYPE    DESC    F1      F2      SS
0       F1      .       .       .       .       Factor  Factor 1        .       .       .
0       F2      .       .       .       .       Factor  Factor 2        .       .       .
0       SS      .       .       .       .       Factor  Sleep stage     .       .       .
1       ID      .       2       .       .       ID      Individual ID   .       .       .
2       V1.F1_A_F2_X    V1      2       d1      g1      Numeric Var 1 (F1=A, F2=X)      A       X       .
3       V1.F1_A_F2_Y    V1      1       d1      g1      Numeric Var 1 (F1=A, F2=Y)      A       Y       .
4       V1.F1_B_F2_Y    V1      2       d1      g1      Numeric Var 1 (F1=B, F2=Y)      B       Y       .
5       V1B.F2_X_SS_N2  V1B     1       d1      g2      Numeric Var 1 (F2=X, SS=N2)     .       X       N2
6       V1B.F2_Y_SS_N2  V1B     0       d1      g2      Numeric Var 1 (F2=Y, SS=N2)     .       Y       N2
7       V1B.F2_Z_SS_N2  V1B     1       d1      g2      Numeric Var 1 (F2=Z, SS=N2)     .       Z       N2
8       V2.F1_A_F2_X    V2      2       d1      g1      Numeric Var 2 (F1=A, F2=X)      A       X       .
9       V2.F1_A_F2_Y    V2      1       d1      g1      Numeric Var 2 (F1=A, F2=Y)      A       Y       .
10      V2.F1_B_F2_Y    V2      1       d1      g1      Numeric Var 2 (F1=B, F2=Y)      B       Y       .
11      V9.F2_X_SS_N2   V9      0       d1      g2      Integer More variables (F2=X, SS=N2)    .       X       N2
12      V9.F2_Y_SS_N2   V9      1       d1      g2      Integer More variables (F2=Y, SS=N2)    .       Y       N2
13      V9.F2_Z_SS_N2   V9      1       d1      g2      Integer More variables (F2=Z, SS=N2)    .       Z       N2
```

Each column in `s1.txt` (`COL`) is described here, with the matching
_expanded_ variable names (`VAR`).  The descriptions are also
expanded, and columns in `s.dict` are added to correspond to _factors_
from the data (`F1`, `F2`, etc) with values that match the level
correspondiong to that variable.  That is, variables are created by
combining the _base_ (`BASE`) along with the factor/level pairs, in
the form: `BASE.FAC_LVL_FAC_LVL`

### Other options

We can check that expanded variables names do not get too long (e.g. if some stats programs have a hard limit, say 32 characters).  Here we specify a max of 8 characters:

```
dmerge dict studies s1.txt -ml=8 > s1.dict
```
```
 ** variable name exceeds 8 characters: V1.F1_A_F2_X
 ** variable name exceeds 8 characters: V1.F1_A_F2_Y
 ** variable name exceeds 8 characters: V1.F1_B_F2_Y
 ** variable name exceeds 8 characters: V1B.F2_X_SS_N2
 ** variable name exceeds 8 characters: V1B.F2_Y_SS_N2
 ** variable name exceeds 8 characters: V1B.F2_Z_SS_N2
 ** variable name exceeds 8 characters: V2.F1_A_F2_X
 ** variable name exceeds 8 characters: V2.F1_A_F2_Y
 ** variable name exceeds 8 characters: V2.F1_B_F2_Y
 ** variable name exceeds 8 characters: V9.F2_X_SS_N2
 ** variable name exceeds 8 characters: V9.F2_Y_SS_N2
 ** variable name exceeds 8 characters: V9.F2_Z_SS_N2

*** error : variables too long... options:
 - change max. allowed length with -ml=999 option
 - do not show factors with -nofac
 - use aliases in data dictionaries
 - or use numeric strata codes with -ns
```

Of the suggestions above, one is to omit the factor names from expanded variable names, to make them shorter. 

```
dmerge dict studies s1.txt -nofac > s1.dict
```

Instead of:
```
V1.F1_A_F2_X
V1.F1_A_F2_Y
V1.F1_B_F2_Y

```
for example, this now gives:
```
V1.A_X
V1.A_Y
V1.B_Y
```

Alternatively, we can just encode strata numerically, with the `-ns` option:

```
dmerge dict studies s1.txt -ns > s1.dict
```
in which case the (arbitrary) uniqifying numbers are as follows:
```
 V1.1
 V1.3
 V1.5
```

Here, the codes may vary from run to run, so it would be important to
match the particular data dictionary (`s1.dict`) with this file,
i.e. so that the "meaning" of `V1.3` can be tracked.

### Working with generated data dictionaries

As noted, the generated data dictionary is designed to be easily
analyzable, i.e. to use `s1.dict` hand-in-hand to help analyse the
data in `s1.txt`.  For example, loading it in R:

```
dd <- read.table( "s1.dict" , header=T , stringsAsFactors=F , sep="\t" ) 
```
along with the actual data:
```
d <- read.table("s1.txt" , header=T, stringsAsFactors=F ) 
```

One can imagine querying `dd` to pull out the expanded variable names (`dd$VAR`) which match the column names in `d`.  For example, to get the `V1` variable(s) where the `F1` is level `A`:

```
v <- dd$VAR[ dd$BASE == "V1" & d$F1 == "A" ]
v
```
```
[1] "V1.1" "V1.5"
```
e.g. to pull out these columns:

```
d[ , v ]
```
```
V1.1 V1.5
1    1    3
2    7   NA
```

One can imagine writing simple convenience functions to assist with this: e.g.
```
fvars <- function( dd , bases = NULL , facs = NULL )
{
 inc <- rep( T , dim(dd)[1] )
 if ( ! is.null( bases ) )
  inc[ ! dd$BASE %in% bases ] <- F
 if ( is.list( facs ) )
  for (f in names(facs) )
   inc[ ! dd[,f] %in% facs[[f]] ] <- F
 dd$VAR[ inc ]
}
```

All `V1` variables:
```
fvars( d , "V1" )
```
```
[1] "V1.1" "V1.3" "V1.5"
```

`V1` variables where `F1` is `A` _and_ `F2` is `X` _or_ `Y`:
```
fvars( d , "V1" , list( F1="A", F2=c("X","Y") ) )
```
```
[1] "V1.1" "V1.5"
```
etc.  (note: in practice, such as convenience function might also give
greater control over the and/or logic of selecting specific variables,
this is just for illustration).
