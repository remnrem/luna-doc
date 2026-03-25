# Using Luna at scale

This vignette gives a few practical hints for running Luna on larger
sample lists, especially on shared servers or compute clusters.
The main topics are:

 - choosing between [`-o`](../luna/args.md#output-databases) output databases and [`-t`](../luna/args.md#text-tables)

 - combining output across multiple Luna runs

 - splitting a sample list into slices with `m/n`

 - using Luna with tools such as GNU `parallel`, and in LSF or SLURM batch environments


## `-o` versus `-t`

Luna has two main output modes for larger analyses:

 - [`-o`](../luna/args.md#output-databases): write output to a SQLite database

 - [`-t`](../luna/destrat.md#text-tables): write output as one rectangular text file per table, per individual

In general:

 - use `-o` when the output is modest in size and you want convenient downstream extraction with [`destrat`](../luna/destrat.md)

 - use `-t` when commands produce very large output and you would rather write flat files directly

For example, this is often convenient with command output that is
already reasonably compact:

```bash
luna s.lst -o out.db < cmd.txt
```

whereas this may be preferable for larger outputs:

```bash
luna s.lst -t tout < cmd.txt
```

One practical rule is:

 - for summary-style output, `-o` is usually simpler

 - for very large outputs, especially when each job writes many large tables, `-t` is often safer and faster


## A note on multiple jobs

If you are running multiple Luna jobs in parallel, do not have all jobs
write to the same output target at the same time.

That is:

 - do not have multiple jobs append to the same database with `-a`

 - do not have multiple jobs write into the same text-table folder unless you are certain they are writing to disjoint paths

Instead, write one output per job, for example:

```bash
luna s.lst 1/10 -o out.1.db < cmd.txt
luna s.lst 2/10 -o out.2.db < cmd.txt
...
luna s.lst 10/10 -o out.10.db < cmd.txt
```

or

```bash
luna s.lst 1/10 -t tout.1 < cmd.txt
luna s.lst 2/10 -t tout.2 < cmd.txt
...
luna s.lst 10/10 -t tout.10 < cmd.txt
```


## Combining multiple databases with `destrat`

If each job writes a separate database, you can extract the same table
from all of them in one step by passing multiple database files to
[`destrat`](../luna/destrat.md).

For example:

```bash
destrat out.*.db +PSD -r CH B > psd.txt
```

or equivalently with an explicit shell expansion:

```bash
destrat out.1.db out.2.db out.3.db +PSD -r CH B > psd.txt
```

This is often the easiest way to combine output from many Luna shards.

If all jobs produced the same tables, then `destrat` can be used to
merge those tables across the databases directly, rather than first
dumping one file per database and then concatenating them manually.


## Aggregating across text tables

If you used `-t`, Luna will create one folder per individual, each
containing the same table names.  In that case, a simple shell command
can often combine the same table across individuals.

For example, if every individual folder contains `PSD-B_CH.txt`, then:

```bash
awk '( FNR == 1 && NR == 1 ) || FNR > 1' tout/*/PSD-B_CH.txt > psd.txt
```

keeps the header from the first file only, and then appends the data
rows from all remaining files.

The same pattern works for any text-table output:

```bash
awk '( FNR == 1 && NR == 1 ) || FNR > 1' tout/*/FILE_X_Y.txt > out.txt
```

If you ran multiple shards, each with its own text-table root, the same
approach applies, just with a wider file pattern, for example:

```bash
awk '( FNR == 1 && NR == 1 ) || FNR > 1' tout.*/*/PSD-B_CH.txt > psd.txt
```


## Splitting a sample list with `m/n`

Luna can process the `m`th slice of `n` total slices of a sample list
directly from the command line:

```bash
luna s.lst 3/10 -o out.3.db < cmd.txt
```

This means:

 - take sample list `s.lst`

 - split it into 10 slices

 - process the 3rd slice only

This is a simple way to parallelize a large sample list without having
to create 10 separate sample-list files manually.


## GNU `parallel`

If GNU `parallel` is available, one convenient pattern is:

```bash
seq 1 10 | parallel --progress 'luna s.lst {}/10 -o out.{}.db < cmd.txt'
```

This launches 10 jobs, each processing one slice of the sample list,
and each writing to its own output database.

The same idea works with text tables:

```bash
seq 1 10 | parallel --progress 'luna s.lst {}/10 -t tout.{} < cmd.txt'
```


## LSF and SLURM

In a batch environment, the scheduler-specific submission syntax will
vary, but the Luna-side pattern is usually the same:

 - use the scheduler's array index as `m`

 - set the total number of array jobs as `n`

 - write one output target per job

For example, in LSF-style notation:

```bash
luna s.lst ${LSB_JOBINDEX}/${NJOBS} -o out.${LSB_JOBINDEX}.db < cmd.txt
```

and in SLURM-style notation:

```bash
luna s.lst ${SLURM_ARRAY_TASK_ID}/${NJOBS} -o out.${SLURM_ARRAY_TASK_ID}.db < cmd.txt
```

The exact `bsub`, `sbatch`, or array-job wrapper commands will differ
across sites, but typically the Luna command itself does not need to
change much.


## Checking failures

On clusters, some jobs will occasionally fail because of bad input
files, unexpected EDF issues, path problems, or resource limits.

When that happens:

 - check the job's `.err` file first, as Luna writes log and error output to `stderr`

 - if a job produced no `.db` file or no text-table output, inspect the scheduler log before re-running

 - keep one output per shard, as this makes it much easier to identify which slice failed

In practice, it is often easier to re-run only the failed slice, for example:

```bash
luna s.lst 7/10 -o out.7.db < cmd.txt
```

rather than restarting the full analysis.


## Summary

For larger projects:

 - use `m/n` slices to split the sample list cleanly

 - write one output target per job

 - prefer `-o` for moderate-sized summary output

 - prefer `-t` for very large table-heavy output

 - combine database output with `destrat`

 - combine text-table output with shell tools such as `awk`

 - on clusters, inspect `.err` files when jobs fail
