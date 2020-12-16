# EDFZ: working with compressed EDFs 

Luna supports reading and writing compressed EDFs,
i.e.  with decompression performed on-the-fly.  We call such files
_EDFZ files_.  Specifically, Luna uses the
[BGZF](https://samtools.github.io/hts-specs/SAMv1.pdf) library
(developed to support high throughput sequencing alignment/map files), which
itself takes advantage of the
[gzip](https://en.wikipedia.org/wiki/Gzip) file format's ability to
concatenate compressed files in a way that allows for random access of
small blocks.  This means it is possible to allow random access into
part of a compressed file without having to decompress the entire file.

For large projects, there can be considerable savings in terms of disk
space from working with compressed files.  Importantly, the format is
simply a special case of the gzip format, so an EDFZ file can be
easily decompressed with the standard `gunzip` tool, e.g.: 
``` 
cat my.edfz | gunzip > my.edf 
```

Here we consider two use-cases (PSGs from the NSRR, and a hdEEG
study), to consider the space/time trade-offs in working with EDFZ
instead of EDF.  We'll also consider how this plays with Luna's
[`RECORD-SIZE`](../ref/manipulations.md#record-size) command, which
allows you to change the internals of the EDF format.  Because Luna
reads an EDF one record at a time, different records sizes can have
different impacts on different workflows.  Here we'll consider
streaming over the entire EDF.

These tests were performed on an iMac with a 4 GHz Intel Core i7
processor and 32 GB RAM.


!!! info 
    Naturally, if you are routinely using other tools to work
    with the EDFs alongside Luna, there will be limited value in using
    compressed files, unless those other tools can read gzipped files
    directly.  That is, you would not want to have to retain
    uncompressed copies of the original EDF too... On the other hand,
    if you are moving EDFs to a separate system (e.g. a Linux compute
    cluster) specifically for Luna analyses, this might be a really
    good use-case for creating and migrating EDFZ files only.


## Use-case #1, NSRR PSG

- ~10 hours with 14 signals
- 54Mb file
- two EEG (125 Hz) channels, ECG, EMG, EOG, respiratory and other signals
- original EDF record size of 1 second

From the original EDF, we'll obtain three other files, using
compression and/or a different record-size, using the following
commands.  To write a compressed (EDFZ) file, add the `edfz` option
to [`WRITE`](../ref/outputs.md#write):


```
luna t1.lst -s "WRITE edfz edf-dir=test/ edf-tag=z1 sample-list=z1.lst" 
```

As well as the original 1 second record size, we'll try working with a
30-second record size.  Because epochs cannot be smaller than the EDF
record size, we do not want to set this to be any larger.  To increase
the record size to 30 seconds, but keep an uncompressed EDF, we'd use
the [`RECORD-SIZE`](../ref/manipulations.md#record-size) command
(which also expects the same parameters are `WRITE`, as it forces an
immediate write of the reformatted EDF):

```
luna t1.lst -s "RECORD-SIZE dur=30 edf-dir=test/ edf-tag=t30 sample-list=t30.lst" 
```

Finally, to change both record size and write as an EDFZ:
```
luna t1.lst -s "RECORD-SIZE dur=30 edfz edf-dir=test/ edf-tag=z30 sample-list=z30.lst" 
```

Here are the resulting __file sizes  (in bytes)__ for EDF and EDFZ:
```
          EDF       EDFZ      IDX     EDFZ+IDX  RATIO
RS = 1    54495840  24578092  476909  25055001  46%
RS = 30   54495840  23660147  15873   23676020  43%
```

Using the EDFZ format reduces file size by more than half: from 54Mb
to around 24Mb.  In general, PSG files compress quite well.  As
expected, changing EDF record size has no impact on the EDF file size, 
whereas there is a small difference in the efficiency of EDFZ compression.

!!! note ".edfz.idx files" 
    Saving as an EDFZ also produces an _index_
    file (`IDX` above), i.e. if we save `test/my.edfz` Luna will
    additionally generate a file `test/my.edfz.idx`.  The index files
    needs to be kept with the EDFZ file; they are simple text files
    that specify the EDF total record size (in bytes), and the offsets
    for each record in the EDFZ.  You can safely ignore the contents of these
    files.


But how much of a price do we pay (if any) for having to decompress
on-the-fly, in terms of speed?  In theory, working with compressed
files could actually be _faster_ than uncompressed files, depending on
disk speed, CPU speed and the achieved compression ratio, but in
general we expect it will slow things down a little.

We'll use the [`STATS`](../ref/summaries.md#stats) command to calculate
summary statistics for every channel in the EDF or EDFZ, which ensures
that all data from the EDF is read into memory:

```
luna t1.lst -s STATS
```

The __times taken (in seconds)__ to load the data and complete this command are:

```
          EDF    EDFZ
RS = 1    2.8    10.5
RS = 30   1.9    2.3
```

In this case, we see that working with EDFZ and small record sizes (1
second) incurs noticeable overhead (in relative terms), taking 10.5 instead
of 2.8 seconds.  However, using the larger 30-second	 record size speeds things up in both
cases: in fact, the compressed EDFZ is now _faster_ than the
original EDF, but only using 43% of the disk space.  You do pay a 0.4
second cost per-file on loading, but for any non-trivial analysis,
this will be negligible in comparison to the overall processing time.

Importantly, identical results (from the `STATS` command) were
obtained for all four analyses.  That is, EDFZ is a _lossless_
compression format.  We can also check that after using `gunzip` on
the EDFZ, we obtain an EDF that is _identical_ to the original one
(for the same record size).  For example, if `test/my-t30.edf` was the
original EDF:

``` 
cat test/my-t30.edfz | gunzip > test/my-t30-v2.edf 
```
and we'll find that:
```
diff -q test/my-t30.edf test/my-t30-v2.edf
``` 
yields no differences.


## Use-case #2, sleep hdEEG 

For the second example we consider a significantly larger, different
type of EDF file: a hdEEG sleep dataset more than 70 times the size of
the previous PSG file.

- ~9 hours with 63 signals
- 4 Gb file
- EEG channels sampled at 1000 Hz
- original EDF record size of 1 second

Using the same approach as above, here are the __file sizes (in bytes)__
and the compression ratios, as a function of record size (`RS`):

```
         EDF         EDFZ        IDX     EDFZ+IDX    RATIO
RS = 1   4022440384  2568741043  489936  2569230979  64%
RS = 30  4021936384  2532976251  16321   2532992572  63%
```

These EEG files tend not to compress quite as well as the PSG files:
after all, the brain is a more complex organ and so EEG has a higher
information content.  Nonetheless, if we are saving around 1.45 Gb per EDF, 
in a project with 100s of recordings, these differences will quickly become
non-trivial.

In terms of __speed__, naturally everything takes longer (this is a much larger dataset, to be fair):

```
          EDF   EDFZ
RS = 1    168   201
RS = 30   140   151
```

We see a similar pattern to above, however, in that larger EDF record
sizes increase speed overall, as well as making the speed difference
between EDF and EDFZ files relatively trivial.


## Conclusion


Overall, the combination of large (i.e. epoch-length) record sizes and
EDFZ appears to result in a favourable combination of performance in
terms of both space and time.  If working with large datasets, or
making copies of original EDFs that are used by Luna only, you may
want to consider the `edfz` option of `WRITE` and `RECORD-SIZE`.

Naturally, there may be other considerations that impact
performance. The potential drawbacks are a) using larger record size
currently precludes the ability to look at epochs smaller than the
record size, b) if using other tools on the same set of EDFs that do
not support on-the-fly compression, there is no point in compressing
EDFs (with the caveat that it is easy to decompress with `gunzip`,
e.g. if you only want to occasionally use another tool to look at one
or two EDFs).


## Footnote

We have not performed any type of exhaustive test, but _purely in
terms of time to load an EDF_, Luna seems to compare favorably to the couple
of other tools we've used.  Taking the default (EDF, 1-second record
size) hdEEG dataset above:

- [edfReader](https://cran.r-project.org/package=edfReader) R package: 8 minutes
- [EEGLAB](https://sccn.ucsd.edu/eeglab/index.php) Matlab toolbox: 8 minutes 30 seconds
- _lunaC_: 2 minutes 48 seconds 
- _lunaR_: 2 minutes 45 seconds  

Although not formally checked, EEGLAB (using the Biosig toolbox) also
appeared to use more than double the amount of RAM compared to Luna
and edfReader (peaking at ~45 GB).  

!!! Info "Context for comparisons" 
    These other tools likely have options (e.g. memory mapping
    files in EEGLAB) that might greatly enhance performance: we have
    not investigated any of those options.  Naturally, EEGLAB is a
    wonderfully powerful tool that is quite possibly doing a number of
    other checks, etc, behind the scenes, not to mention the fact that
    it encompasses a whole suite of EEG-based methods that are not even 
    part of Luna, which has a very different and focussed use-case.  The point of these
    comparisons is merely to note that Luna's baseline performance is
    at least comparable with other tools that are designed to analyse
    EDF files, which can be useful if wanting to perform standard
    types of analyses on large numbers of EDFs.
