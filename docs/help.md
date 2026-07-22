taf-meryl 1.4.2-r1

Purpose:
  Count, store, combine, filter, and query genomic k-mers with Meryl 1.4.2.

Usage:
  taf-meryl -- --version
  taf-meryl -- -h
  taf-meryl meryl k=21 memory=64 threads=8 count reads.fastq.gz output reads.meryl
  taf-meryl COMMAND [ARGS...]

Common workflows:
  taf-meryl meryl histogram reads.meryl > reads.hist
  taf-meryl meryl statistics reads.meryl > reads.stats
  taf-meryl meryl print reads.meryl > reads.kmers.tsv
  taf-meryl meryl ploidy reads.meryl > reads.ploidy.tsv
  taf-meryl meryl-analyze -mers reads.meryl -prefix reads -gt

Packaged commands:
  meryl             Main count, database, filter, set, and report command.
  meryl-lookup      Lookup and annotation helper.
  meryl-import      Import helper.
  meryl-analyze     GC, GA, and GT motif analysis helper.
  meryl-simple      Simplified helper interface.
  position-lookup   Positional lookup helper requiring data arguments.

Upstream help and version:
  taf-meryl -- -h
  taf-meryl -- --version
  taf-meryl meryl --version

Command-mode note:
  Use "taf-meryl meryl ..." for positional Meryl commands such as count,
  histogram, and ploidy. Use "taf-meryl -- ..." only when the first upstream
  argument starts with "-", such as -h or --version.

Inputs:
  FASTA/FASTQ       Plain, gzip, bzip2, lzip, xz, or zstd sequence records.
  SAM/BAM/CRAM      Alignment records read through bundled HTSlib.
  .meryl directory  Complete modern Meryl database directory.
  histogram file    Frequency/count text accepted by ploidy reporting.

Key outputs:
  *.meryl/          Directory-form Meryl database from a count command.
  stdout            Histogram, statistics, print, and ploidy text reports.
  PREFIX.*          Files written by meryl-analyze in the working directory.

Platform and resources:
  Native linux/amd64 and linux/arm64 images are supported.
  Set memory= and threads= explicitly for production counting jobs.
  No external database is required for ordinary local Meryl use.

Remote and CRAM inputs:
  HTTP(S) and S3 paths are contacted only when explicitly supplied.
  Network access, credentials, and CRAM references are user-managed.

Boundaries:
  Merqury, GenomeScope, reports, large datasets, and references are not bundled.
  Experimental meryl2 is not part of the upstream public default build.
  Large-data performance and remote services are outside the smoke scope.

Detailed documentation:
  https://github.com/taffish/meryl
  https://github.com/marbl/meryl/tree/v1.4.2

Wrapper options:
  taf-meryl --help       Show this TAFFISH help.
  taf-meryl --version    Show TAFFISH wrapper version.
  taf-meryl --compile    Print generated shell code.
  taf-meryl -- --help    Pass option-leading args to the default command.

Notes:
  Modern Meryl databases are directories, not legacy .mcdat/.mcidx pairs.
  The exact upstream source and utility commits are pinned and checksummed.
  TAFFISH packaging is Apache-2.0; upstream and bundled code keep their terms.
