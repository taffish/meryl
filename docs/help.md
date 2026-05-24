taf-meryl 1.4.1-r3

Meryl is a genomic k-mer counter and sequence utility.

Usage:
  taf-meryl [-h | --help]
  taf-meryl [-v | --version]
  taf-meryl [MERYL_ARGS...]
  taf-meryl -- [MERYL_ARGS...]
  taf-meryl COMMAND [ARGS...]
  taf-meryl --compile [ARGS...]

Wrapper options:
  -h, --help       Show this TAFFISH app help
  -v, --version    Show TAFFISH package version
  --compile        Print generated shell code instead of running it
  --               Stop wrapper option parsing and pass following args to meryl

Default upstream command:
  taf-meryl -- --version
  taf-meryl -- -h
  taf-meryl -- k=21 memory=64 threads=8 count reads.fastq.gz output reads.meryl
  taf-meryl -- histogram reads.meryl
  taf-meryl -- statistics reads.meryl
  taf-meryl -- print reads.meryl

Command mode:
  taf-meryl meryl --version
  taf-meryl meryl k=21 memory=64 threads=8 count reads.fastq.gz output reads.meryl
  taf-meryl meryl-lookup
  taf-meryl meryl-import
  taf-meryl meryl-analyze
  taf-meryl meryl-simple

Packaged commands:
  meryl, meryl-lookup, meryl-import, meryl-analyze, meryl-simple,
  position-lookup.

Notes:
  Modern Meryl databases are directories, not old .mcdat/.mcidx pairs.
  This app packages Meryl 1.4.1 from the official Linux-amd64 binary tarball.
  gzip, bzip2, xz, and bash are included for common Meryl input and helper
  paths.
  Native support is linux/amd64 only. Docker and Podman declare
  --platform linux/amd64 in the app wrapper, so arm64 hosts use amd64
  emulation when supported.

License:
  TAFFISH app packaging: Apache-2.0.
  Upstream software: Public Domain.
  Bundled components, data, models, and external resources keep their
  own license terms.

Boundaries:
  This app packages Meryl only. Merqury, GenomeScope, plotting/report workflows,
  and large example datasets are not bundled.
