# dmerge

_dmerge_ is a tool for working with derived metrics from Luna (or other
tools that generate individual-level output).  By adhering to a
particular file and folder naming convention, data from diverse
pipelines can be easily compiled and checked across multiple individuals.

## Usage

```
dmerge derived/domains
       studies/study1
       merged/study1.txt
       { domains/groups/exclusions }
       > merged/dictionary1.txt
```

## Primary functions

 - compiles multiple, heterogeneous, stratified datasets across multiple individuals into a single file

 - enforces the use of domain-based, data-dictionaries

 - expands from a long- to wide-format dataset, providing a general
   convention for representing variable meta-data

## Definitions

__FACTOR__: a variable such as channel, frequency or sleep stage, etc.
The level of a factor is its value, e.g. `C3`, `12.5` (Hz) or `N2`
respectively.  Factors stratify the values of typical variables,
rather than represent data by themselves. Factors can appear in the
body of a file (for long-format data) or are otherwise encoded in the
variable name (see variable naming conventions, below).  Factors can
also be specified through file naming conventions (e.g. here the
factor `SS` is set to `N2`: `eeg_spectral_PSD-CH_B_SS-N2.txt`).

__DOMAIN__: the highest grouping of data dictionaries; typically a
domain will correspond to an NSRR investigator, i.e. one person/group
controls the vocabulary within that domain.

__GROUP__: within a domain, different data dictionaries can be
modularized, within each separate data dictionary file representing a
group that belongs to a particular domain.

__TYPE__: all variables are of a specified type, specified in the data
dictionary; types are described below.

__EXPANDED VARIABLES__: wide-format variable names, i.e. whereby
factor and level information for a variable is encoded as part of its
variable name



## Folder naming conventions

- All dictionaries must be in a single folder (passed via the first parameter)

- All individual data should be in individual subfolders within a study folder: e.g.

```
data/study1/ie001/
data/study1/id002/
data/study1/id003/
...
```

-  Can have one or more intermediate subfolders between study and individual, all folders are searched for data recursively, e.g.:

```
data/study1/luna/id001/
data/study1/luna/id002/
data/study1/luna/id003/
data/study1/pupa/id001/
data/study1/pupa/id002/
data/study1/pupa/id003/
...
```

- Additional data that need to be stored (e.g. verbose intermediate
  files) can be put under a special folder `extra` within the
  individual sub-folders, and will be skipped; e.g. below, _file3_ is
  ignored

```
data/study1/id003/{file1}
data/study1/id003/{file2}
data/study1/id003/extra/{file3}
```

## File naming conventions

All data files should be named according to the convention:

```
   {domain}_{group}_{label}{_factor}{_factor}{_factor-level}{.txt}
```

Examples (in the `eeg_spectral` domain/group): 

```
  eeg_spectral_psd_B_CH_SS-N1
  eeg_spectral_psd_B_CH_SS-N2
  eeg_spectral_psd_F_CH_SS-N2
  eeg_spectral_psd_E_F_CH_SS-N2.txt
```

- Domain, group and label are required (i.e. always three
  underscore-delimited terms starting the filename)

- Domain and group must correspond to a data dictionary specified on
  the command line, e.g. then `dict/eeg_spectral.txt` must exist)

- The file extension `.txt`, if present, is ignored; files ending `'~'`
  are ignored completely

- Optional factors in the file name (e.g. `CH` for channel) indicate a
  column in the body of the file that gives levels for that factor

- Optional _{factor-level}_ pairs, e.g. `SS-N2`, set that factor
  (`SS`) to that level (`N2`), i.e. as if a column existed in the body
  of the file where all values of `SS` were set to `N2`. Only the first
  hyphen is used to delimit the factor and level, so `F--2` will set
  factor `F` to `-2`.

## Variable naming conventions

- Root variable names (e.g. `PSD`) can only contain alphanumeric characters and underscores

- Factor names (e.g. `B`) can only contain alphanumeric characters and periods

- In the data output and data-dictionary, root variables are expanded
  to contain any additional factor/level pairs

For example, if the following file 

```
eeg_spectral_psd_B_CH_SS-N2.txt
```

contained the variable `PSD` (which was defined in the `eeg_spectral`
domain/group data dictionary), then we might then find expanded
variables such as:

```
PSD.B_ALPHA_CH_C3_SS_N2
PSD.B_ALPHA_CH_C4_SS_N2
PSD.B_BETA_CH_C3_SS_N2
PSD.B_BETA_CH_C4_SS_N2
...
```

That is, expanded variable names are in the form:
```
   {root}.{fac_lvl}_{fac_lvl}
```

- All variable names will be converted to UPPERCASE; variable names
cannot start with a digit or underscore, and cannot contain a period

- Factor names have the same rules as standard variable names, except
they can contain periods but not underscores

- Levels (i.e. that are _"values"_ rather than _"names"_) can contain both
underscores and periods underscores will be converted to periods
(`C3_A2` --> `C3.A2`) periods are common in level values, e.g. if the
factor is frequency: `F_12.25`

- e.g. this implies power for 12.25 Hz (`F`), for channel `C3_A2` (`CH`), during `N2` sleep (`SS`)
```
PSD.F_12.25_CH_C3.A2_SS_N2
```

All factors should be described in the data dictionary; merge will
auto-generate data-dictionary entries for expanded variables based on
the root variable label, and factors/levels (see examples).

## Data-dictionary format

As well as residing in the folder specified on the command line, data
dictionaries should adhere to the following naming convention:

```
{domain}_{group}.txt
```

e.g.
```
dict/eeg_spectral.txt
dict/macro_stages.txt
```

Possible _domains_ (e.g. broad areas) are:

- `macro`
- `eeg`
- `ecg`
- `resp`
- `actigraphy`


Domain and group names _cannot_ contain underscore characters, as these
are used to delimit domain, group and file tag/factor-lists.  They can
contain periods and hyphens.

To only extract/compile variables from specific domains, add those
domain or domain_group identifiers to the command line, after the
other options. For example, if the folders `dict/` and `study1` exist:

_All domains/groups:_

```
dmerge dict study1 study1.txt 
```

_Only `eeg` domain variables_:

```
dmerge dict study1 study1.txt eeg
```

_Only `eeg_spectral` and `eeg_spindles` groups, and all `macro` domain variables_:

```
dmerge dict study1 study1.txt eeg_spectral eeg_spindles macro
```

Within a dictionary file, we expect 3 tab-delimited columns:

- variable 
- type
- label/description 

Any lines starting with `%` are treated as comments and ignored.

Types are as follows:

| Type | Description | Notes|
| --- | --- | --- |
|`numeric`|Floating point values | Tested as a valid number; can use scientific notation (e or E) | 
|`integer`|Integer values | Tested as an integer rather than float, i.e. digits and minus sign only, no decimal point/period character or scientific notation |
|`text`|Any text | No validation rules |
|`yesno`|Boolean value | Case-insensitive encodings: _true_ = 1, y[es] and t[rue]; _false_ = 0, n[o] and f[alse] |
|`factor`|A factor | Indicates that this is factor rather than a variable |
|`date`|Date | Simple check of XX/YY or XX/YY/ZZ encoding (i.e. 2 or 3 elements, delimited by either / or - characters) |
|`time`|Times | Simple check of HH:MM or HH:MM:SS encoding (i.e. 2 or 3 elements, delimited by either : or . characters) |


## Aliases

Data dictionaries can specify aliases for variable names.  This can be
useful, say, if different files have used different labels for the
same variable.  Alternatively, aliases can be used to resolve name
clashes between domains/data-files, if two different variables have
the same name.  Use the keyword alias in the data dictionary as
follows:

```
CH      factor   Channel
CNAME   alias    CH
```

This sets `CNAME` to be an alias for `CH`, where `CH` is said to be the canonical form.

- Any time `CNAME` is encountered in any data-file of that domain, it
  will be as if `CH` were written there instead, i.e. so `CH` will
  appear in the output.

- The tool will check that a canonical form has first been specified,
  i.e. appears earlier in the data dictionary.

- Also, it will check that the same label does not appear as both an
  alias and a canonical form, or that the same alias is listed
  multiple times for different canonical forms.

- As with all variable names, matches are case-insensitive, but all
  output will standardized for _ALLCAPS_ output.

- You can have multiple aliases for the same canonical form, however:
  e.g. here both `CNAME` and `my.ch` would be remapped to `CH`:
```
CH      factor   Channel
CNAME   alias    CH
my.ch   alias    CH
```

## Data dictionary output format      

The combined data are written to the file specified on the
command line.  The corresponding data-dictionary is written to stdout,
using the following format:

| Variable | Description |
| --- | --- |
|VAR | Variable name (e.g. DENS_F_11_CH_F3_SS_N2)
|BASE | Variable root (e.g. `DENS`) |
|TYPE | Type of variable
|OBS | Number of non-missing observations
|GROUP | Group name
|Factors... |  Additional columns describe all factors encountered, e.g.  band, channel and stage (B, CH and SS respectively), with the row as the corresponding level for that expanded variable
|DOMAIN | Domain name |
|DESC | Description |
|COL | Column number in the data file (0 for factors, i.e. which do not appear as columns/variables in the data file)| 



## Exclusions

- Can exclude particular domains/groups by explicitly listing
  domains/groups to be included on the command line, after the other
  options (see example above)

- Can exclude particular variables by setting name to `-VAR` in the
  data dictionary

- Can exclude particular files by setting a `-tag` or
  '-tag-factorlist' on the command line (`-HYPNO-C` will skip
  `macro_stages_HYPNO_C.txt`)


## Permissible character cheat sheet

Less complicated than it may look at first sight, here's a summary of identifier conventions:

| Identifier| Allowed | Disallowed | Notes | 
| --- | --- | --- | --- |
| __All identifiers below__ | __Alphanumeric, period, hyphen/minus & underscore__ | __All other special characters (including spaces)__ | __These conventions allow identifiers to be straightforwardly represented as both filenames (across different OS) and variable names (e.g. within a package such as R)__ | 
| Data dictionary file | Alphanumeric, period, hyphen and underscore | | An optional `.txt` extension is allowed, and will be ignored; underscores are used to delimit domain and group identifiers from file names. | 
| Data dictionary domain/group | Alphanumeric, period and hyphen | Underscore | As above, underscores are used to delimit domain-group identifiers | 
| Variable | Alphanumeric and underscore | Period, hyphen | Should start with a letter; upper and lowercase allowed, although all output will be transformed to uppercase |
| Factor | Alphanumeric and period | Underscore, hyphen | Should start with a letter; upper and lowercase allowed, although all output will be transformed to uppercase |
| Factor levels (data file body) | Alphanumeric, period, hyphen/minus & underscore | | Underscores are allowed but are transformed to a period in the output (in variable names) | 
| Factor levels (data file name)  | Alphanumeric, period, hyphen/minus | Underscores | Levels specified in this way can contain periods and additional hyphens; they cannot contain underscores |
| Data file | Alphanumeric, period, hyphen/minus & underscore | |  Special format(s): <br> `domain_group_tag` <br> _data w/out factors ('baseline')_ </p> `domain_group_tag_F1_F2_F3` <br> _data w/ 3 factors F1, F2 & F3_ </p> `domain_group_tag_F1_F2-X_F3-Y` <br> _as above, but setting F2 to X and F3 to Y_ </p> i) underscores delimit domain, group, file-tag and optional factor list </p> ii) optionally, first hyphen in `FAC-LVL` sets factor `FAC` to level `LVL` </p> A `.txt` file extension is allowed, and will be removed |

## Misc details

- __All data files must be ASCII plain-text, tab-delimited with a
  single header row__

- __Expects individual/EDF ID is named `ID`__

- Checks that variable names (except factors) are unique within and
  across domains

- Run with option `-v` to produce verbose output

- Run with option `-s` to enfore strict mode: here all
   variables/factors/domains/groups present in the data folders must
   have an exact match in the data dictionary (otherwise an error is
   given and the program halted). Otherwise, in the default
   (non-strict) mode, such things are just skipped over silently (or,
   if `-v` is set, with a message to standard error stream).

- Flag duplicate rows (same ID/factors) in and across datasets

- Creates a data dictionary, with variable base-names (e.g. `PSD`) and
factors (e.g. spectral band) expanded (e.g. `PSD.SIGMA_C3_N2`)

- List the column numbers and count of non-missing observations for
  all individuals

- Each domain can have its own missing data symbol (which are
harmonized to a single value in the output; use a special `MISSING`
variable name and `TYPE` in the data dictionary, where the description
column gives the actual missing value code)

- Merge based on case-insensitive matching; sets all variable numbers
 of UPPERCASE in output

- Spaces and special characters in variable names are removed

- Basic type enforcement checks on values 

- Counts missing data for each variable

- Data-dictionary only needs to specify root (long-format) variables
  (e.g. `PSD` not `PSD.B_SIGMA_CH_C3_SS_N2`)

- Throws an error if a variable is present but not described in the
  dictionary

- Columns can be in any order across datasets

- Even in strict mode, all variables must be in the data dictionary, but different
  individuals need not all have the same variables in each file
  (missing data will be indicated in the output)
