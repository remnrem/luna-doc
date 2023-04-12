# It's about time: specifying annotation intervals

As described [here](../ref/annotations.md), Luna supports a number of formats for specifying the
start/stop times (and dates) of annotations (in either an [`.annot`](../ref/annotations.md#annot-files) or
[`.xml`](../ref/annotations.md#nsrr-xml-files) annotation file). Here we detail these formats and provide a few examples.

## Formats

When specifying formats below, the following abbreviations are used:

 - `x`: elapsed seconds 
 - `hh`: hours (clock 24hr: 00 to 23; clock 12hr: 1-12; elapsed: 0+)
 - `mm`: minutes (0 - 59) _or_ month (1 - 12, or Jan-Dec) 
 - `ss`: seconds (0 to 60) and can include fractional components
 - `dd`: day (1 - 31)
 - `yy`: year (85-99 = 1985 - 1999; 00-84 = 2000 - 2084)
 - `yyyy`: year (from 1985 onwards)


There are four basic ways to specify _start-times_:

 - elapsed seconds past the EDF start, e.g. `30`

 - absolute clock-times (pref. 24hr based as per EDF headers), e.g. `23:30:02` or `11:30:02pm`

 - elapsed times in _hh:mm:ss_ format starting `0+`, e.g. two hours from the EDF start `0+02:00:00` 

 - _epoch-encodings_ starting `e:`, e.g. `e:10` for the 10th epoch

_Stop-times_ can be specified in a similar manner as for _start-times_, but there are two additional options:

 - specifying the _duration of the event_ in seconds, by starting with `+`, e.g. `+30` for a 30-second interval

 - specifying an event lasting _until the start of the next annotation in the file_, by writing an ellipsis (`...`)


For example, if the EDF start time is `22:28:15`, then all the examples below are identical (i.e. a 30-second interval starting
from 2 minutes into the recording until 2 minutes and 30 seconds):

| Start time field | Stop time field  | Example (start, stop) | Description |
| ---- | ---- | ----- | ----|
| `x`  | `x`  | `120`, `150` | Elapsed seconds from EDF start for both start & stop |  
| `x`  | `+x` | `120`, `+30` | Using _duration_ to specify a 30-second interval starting at 120 seconds past EDF start | 
| `hh:mm:ss` | `hh:mm:ss` | `22:30:15`, `22:30:45` | Using clock-time to specify start & stop (note: a `.` delimiter can be used instead of `:`) |
| `hh:mm:ss` | `+x` | `22:30:15`, `+30` | As above, but using a _duration_ rather than explicit stop time |
| `dd-mm-yy-hh:mm:ss` | `dd-mm-yy-hh:mm:ss` | `04-08-18-22:30:15`, `04-08-18-22:30:45` | Explicit date-time strings (date delimiters `-` or `/`); year can be `2015` or `15`; month can be `Aug`, `8` or `08` | 
| `0+hh:mm:ss` | `0+hh:mm:ss` | `0+00:02:00`, `0+00:02:30` | Elapsed _hh:mm:ss_ times from EDF start (rather than clock-times) |
| `x` | `...` | `120`, `...` | Duration is up until the next annotation _in the same file_ (assumed to be at 150s here); this can be used with any format of _start-time_ (i.e. absolute or relative clock-time) | 


!!! note "Delimiters"
    For annotation files, Luna requires a colon
    (`:`) as the delimiter for times, e.g. `hh:mm:ss`.  (For EDF
    headers, either `.` or `:` can be used; and Luna may output times
    with period delimiters in some cases because of this.)  For dates,
    `-` is the preferred delimiter in annotation files, although `/`
    is also accepted: for a date-time string `dd-mm-yy-hh:mm:ss` is
    the canonical format.  (Note that Luna uses _European_ date
    formats with day, then month, then year as per EDF spec.

## Examples

Consider we have an EDF `f.edf` with the following start time and date in the EDF headers:

```
 duration 08.56.46, 32206s | time 21.23.23 - 06.20.09 | date 29.07.16
```
i.e. starting just after 9:23pm on the 29th of July 2016. 

Also consider this four-column `.annot` annotation file:
```
a01   .   e:5                 .
a02   .   e:5                 e:5
a03   .   120                 150
a04   .   120                 +30
a05   .   21:25:23            21:25:53
a06   .   9:25:23pm           9:25:53pm
a07   .   9:25:23 pm          9:25:53 pm
a08   .   21:25:23            +30
a09   .   0+00:02:00          0+00:02:30
a10   .   0+00:02:00          +30
a11   .   29-07-16-21:25:23   29-07-16-21:25:53
a12   .   29-07-16-21:25:23   +30
a13   .   21:25:23            ...
a00   .   21:25:53            21:25:53
a98   .   28-07-16-21:25:23   28-07-16-21:25:53
a99   .   30-07-16-21:25:23   30-07-16-21:25:53
```

Here we have 16 annotations (each of a distinct _class_ labelled `a00`
to `a99`).  Of the main annotations (`a01` to `a13`), __all specify
the identical period in the recording__.

 - `a01` and `a02` use _epoch-encoding_; assuming 30-second epochs starting at the EDF start, the 5th epoch spans from 120 to 150 seconds from the start of the EDF; when specifying a single epoch, the stop field can be left as missing (`a01`)

 - `a03` explicitly gives the start time in seconds (120) past the EDF start; `a04` is similar but specifies a duration `+30` rather than an explicit stop time (`150`)

 - `a05` uses a 24-hr clock-time - as the EDF start is `21:23:23`, then `21:25:23` is two minutes (120 seconds) past the EDF start

 - `a06` and `a07` illustrate the use of 12-hour AM/PM encoding formats

 - `a08` combines absolute clock-time to specify the start, with a duration (in seconds) in the second field

 - `a09` uses elapsed time in _hh:mm:ss_ format, i.e. 120 seconds (`02:00`) and 150 seconds (`02:30`) past the EDF start; `a10` is similar but uses a duration (`+30`) encoding

 - `a11` gives explicit absolute date-time strings in `dd-mm-yy-hh:mm:ss` format for both start and stop times; `a12` is similar but uses the duration encoding of `+30` instead of a stop date-time

 - `a13` uses clock-time encoding combined with an _ellipsis_ to specify the duration as _up to the next observed annotation in the file_ (or EDF end, if this is the last annotation in the file).  We therefore add a dummy event `a00` after it, which is a zero-duration timestamp at 150 seconds past the EDF start (i.e. purely to define `a13`). In practice, this type of encoding might be used for sleep stages, or body positions, where we only record the _change_ of state, and we wish for the entire recording to be covered by some type of these annotations.

The file contains two other events that test the effect of shifting
the date field either a day earlier (`a98`) or later (`a99`).  Thus,
`a98` occurs _before_ the start of the EDF (and so should be
ignored); in contrast, `a99` occurs 24 hours after all the other
events.

To test this, we will load the annotations and use `WRITE-ANNOTS` to output the file using a uniform format (by default, elapsed seconds) called `a2.annot`:

```
luna f.edf annot-file=a1.annot -s WRITE-ANNOTS file=a2.annot
```

Examining the `a2.annot` file (removing extra blank columns and re-ordering rows for clarity) - of the primary events (`a01` through `a13`) we see
they all resolve to the identical interval:
```
a01  . 120.000  150.000
a02  . 120.000  150.000
a03  . 120.000  150.000
a04  . 120.000  150.000
a05  . 120.000  150.000
a06  . 120.000  150.000
a07  . 120.000  150.000
a08  . 120.000  150.000
a09  . 120.000  150.000
a10  . 120.000  150.000
a11  . 120.000  150.000
a12  . 120.000  150.000
a13  . 120.000  150.000
```
As expected, the _dummy_ interval `a00` is at 150 seconds exactly:

```
a00  . 150.000  150.000  .
```
Finally, `a98` is not present - it is ignored as it occurred prior to the EDF start (one day earlier). In contrast, `a99` (starting one day later)
is:
```
a99  . 86520.000  86550.000
```
i.e. which as one day is 24 x 60 x 60 = 86400 seconds, is 120 + 86400 = 86520 and 150 + 86400 = 86550 seconds for start and stop times respectively, as expected.


## Modifying the EDF start 

To further explore and confirm the behavior of Luna's handling of times
and dates, here we perform the same command but first modifying the
EDF headers to 1) set the start date to null, 2) alter the start time,
and in the section below, 3) alter the start dates.  

First, we'll set the EDF start date to null (i.e. `1.1.85`).  Adding `anon=T` to the
command line modifies the EDF header _before_ attaching annotations, and so annotations will be processed differently
depending on whether they have absolute or relative times.  In fact, if the EDF header has a null start date, then Luna will not
allow any dates to be specified in annotation files - i.e. as it would not be possible to correctly place the annotation relative
to the start of the EDF, as the duration between that and the annotation is unknown.

```
luna f.edf annot-file=a1.annot anon=T -s WRITE-ANNOTS file=a2.annot
```
That is, Luna correctly reports an error message:
```
error : cannot specify annotations with date-times if the EDF start date is null (1.1.85)
```

As the message states, specifying explicit dates in an annotation file
is only allowed if the EDF header has a non-null date defined.
If we drop the offending lines from the file (i.e. `a11`, `a12` as well
as `a98` and `a99`) then this will run and give identical results (120
to 150 seconds past the EDF start) for all remaining primary intervals.

Next, we'll alter the start _time_ of the EDF header by setting the `starttime` special variable:

```
luna f.edf annot-file=a1.annot starttime=19:00:00 -s WRITE-ANNOTS file=a2.annot
```

This sets the EDF start to 7:00:00pm instead of 9:23:23pm, i.e. 2
hours, 23 minutes and 23 seconds earlier (or 2 x 3600 + 23 x 60 + 23 =
8603 seconds).  Considering the output, we now see that all
annotations using relative start times (or epoch-encoding) have the
same outputs as before when generating a new file that uses elapsed seconds,
i.e. as these do not depend on absolute times:

```
a01  .  120   150
a02  .  120   150
a03  .  120   150
a04  .  120   150
a09  .  120   150
a10  .  120   150
```

In contrast, annotations that used absolute times are delayed w.r.t. to the EDF start, e.g. below, 8723 = 8603 + 120 seconds.

```
a05  .  8723   8753
a06  .  8723   8753
a07  .  8723   8753
a08  .  8723   8753
a11  .  8723   8753
a12  .  8723   8753
a13  .  8723   8753
```

Note that this is not a problem _per se_ - we're merely altering the times
to check the behavior of Luna is as expected under these different encodings.


## Dates and long recordings

By default, annotations are only specified by times (rather than dates
and times) which can impose some limitations on annotation encoding
for long (>24 hour) recordings.

Specifying annotations in elapsed seconds or elapsed _hh:mm:ss_
formats is acceptable for recordings over 24 hours.  For example, here
we can define two (identical) events that begin at 9:25:23pm (the same
time as all the annotations in the example above, or 120 seconds past
the EDF start) but alter the annotation such that it occurs exactly
_three days later_.  In elapsed seconds, this would be 259320 = 120 +
3 x 86,400 (where 86,400 is the number of seconds in a day):
```
b01 . 259320   259350	
```
Similarly, we can specify the same event in in elapsed _hh:mm:ss_ encoding, i.e. adding 72 hours to 2 minutes:
```
b02 . 0+72:02:00 0+72:02:30 
```

Using the above `WRITE-ANNOTS` command after reading in these events,
then with no further options except `file`, the command emits times in
elapsed seconds, with both events correctly aligned as overlapping:

```
b01  .  259320    259350
b02  .  259320    259350
```

However, when dealing with long recordings, using `hms` clock-time encoding would yield ambiguous/incorrect results
when writing a new annotation file, i.e. as we obviously lose information about the _day_:

```
WRITE-ANNOTS file=a2.annot hms
```
```
b01  .  21:25:23    21:25:53
b02  .  21:25:23    21:25:53
```

In this scenario, if we want to retain absolute clock-times in the created annotation file, we must specify `dhms` instead, to emit date-time strings:
```
WRITE-ANNOTS file=a2.annot dhms
```
```
b01  .  1-8-2016-21:25:23    1-8-2016-21:25:53
b02  .  1-8-2016-21:25:23    1-8-2016-21:25:53
```
As the original EDF start date was 29th July, here we see that these events are correctly assigned to the 1st August, i.e. three days later.

!!! info "Dates in Luna"
    Luna understands calendar conventions for advancing
    _X_ number of days (i.e. correctly counting 3 days from 28th of July to be the 1st August as above, and
    it will respect leap year conventions.  When parsing dates, it is acceptable to use string codes for months
    as well as numbers 1 - 12: e.g. `01-08-2016` equals `01-Aug-2016`.  Month strings are the standard three-letter codes
    and are case insensitive. 

    The _null_ date is `01.01.85` as per the EDF specification
    (equivalently, `1.1.85` or in annotation files `1-Jan-1985`, etc)
    , meaning that no earlier dates can be specified; internally, Luna
    represents dates as the number of days past `1.1.85`.  A value of
    `1.1.85` (i.e. 0 or _null_) is a special case, taken to mean that
    no date is specified (which impacts how Luna parses dates, as
    shown above).  If you want to have multi-day recordings and use
    absolute clock-times/dates in annotations when working with an
    _anonymized_ EDF (i.e. a null start date), we suggest you use
    `2.1.85` (or any other arbitrary date) instead (or simply use
    elapsed time encodings to specify annotations).


!!! info "Internal representation of time and long recordings"
    Internally, all annotations are mapped to elapsed _time-points_
    which are unsigned (positive) integers from 0 (the EDF start) to a
    very large number (8,446,744,073,709,551,615).  Each time-point is
    0.000000001 seconds (1e-9 seconds) which theoretically allows for
    very long recordings (i.e. over 260 years).  The important
    constraint to note is that no _negative_ times can be specified,
    meaning that no annotations can be represented as occurring
    _before_ the EDF start.


To confirm the treatment of dates, we will use `startdate` to force
the EDF header to different dates (in place of the original
`29.07.16`) but keeping the same `a1.annot`.   First, we'll set 
the EDF start time to __one year later__, i.e. 2017:

```
luna f.edf annot-file=a1.annot startdate=29.07.17 -s WRITE-ANNOTS file=a2.annot
```

```
a01   .   120   150
a02   .   120   150
a03   .   120   150
a04   .   120   150
a05   .   120   150
a06   .   120   150
a07   .   120   150
a08   .   120   150
a09   .   120   150
a10   .   120   150
a13   .   120   150
a00   .   150   150
```

Note that in the above output, we're missing `a11` and `a12` (as well
as `a98` and `a99`).  This because those annotations used an absolute
date in 2016, i.e. before the EDF start-date.  When reading the
annotations in, these are therefore skipped automatically.  

Second, we'll now set the EDF start date to __one year earlier__, i.e. 2015:

```
luna f.edf annot-file=a1.annot startdate=29.07.15 -s WRITE-ANNOTS file=a2.annot
```
```
a01  .   120        150
a02  .   120        150
a03  .   120        150
a04  .   120        150
a05  .   120        150
a06  .   120        150
a07  .   120        150
a08  .   120        150
a09  .   120        150
a10  .   120        150
a11  .   31622520   31622550
a12  .   31622520   31622550
a13  .   120        150 
```

Considering the main set of annotations, `a11` and `a12` are now included.  Because those annotations used an absolute 2016 date, Luna
correctly writes the number of seconds as 31622520 = 120 + 3600 x 24 x 366 (as 2016 is a leap year), with a stop time 30 seconds later. 

Likewise, the `a98` and `a99` annotations are one year later (but
either a day behind or a day ahead, i.e. 31536120 = 120 +
3600 x 24 x (366-1) and 31708920 = 120 + 3600 x 24 x (366+1):

```
a98  .   31536120   31536150
a99  .   31708920   31708950
```


!!! hint "Analysis of multi-day recordings"
    Although Luna can represent multi-day recordings, please remember that certain commands (most notably `HYPNO`)
    are not explicitly tailored to this context, and so it would be desirable to create multiple single-day EDFs
    prior to running such analyses. 


## Summary

We've seen that Luna can accept times and dates in a number of
formats.  The `.annot` format is quite flexible: by default, it has header rows and
six tab-delimited fields, but several of these can be omitted, spaces can
be used instead of tabs, and the header rows are optional.  Together with the
alternative options for specifying times, this should enable most text-based
annotation files to be reformatted, with only minimal effort, into a form that Luna can accept.

We've also seen that Luna can handle long (>24 hr) recordings, but in
this case, it is necessary to use elapsed time metrics (seconds or
_hh:mm:ss_) or absolute clock-times that also include dates.  As we've
also seen in other contexts (e.g. exporting EDF+D files or merging
EDFs), working with absolute annotation time-stamps can be convenient
(over times that are relative to the EDF
start) and so the date-time representation support by Luna is useful
here.
