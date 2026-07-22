# meryl

`meryl` packages [Meryl](https://github.com/marbl/meryl) for TAFFISH. Meryl
counts, stores, filters, combines, and queries genomic k-mers, and is also used
by assembly-evaluation workflows such as Merqury.

Package identity:

- name: `meryl`
- command: `taf-meryl`
- kind: `tool`
- version: `1.4.2-r1`
- license: Apache-2.0 for TAFFISH packaging
- upstream: Meryl `v1.4.2`

## What This App Packages

The image builds Meryl `1.4.2` from the exact upstream source commit
`dbacc3d0fd26bcd4247dcaf5b7e70ffdf5f69831` and its pinned `meryl-utility`
commit `f800fc4dada365a701122b5ac1ecb946eb402cb4`. Source archives are verified by
SHA-256 during the build. This source path is used because the upstream
`v1.4.2` tag has no public downloadable binary asset at packaging time.

This update adds upstream `1.4.2` behavior, including native Linux arm64
support, SAM/BAM/CRAM sequence input through bundled HTSlib, the `meryl ploidy`
report, and the `meryl-analyze -gt` motif report.

## Scope

This app supports:

- counting k-mers from FASTA, FASTQ, SAM, BAM, and CRAM inputs;
- reading gzip, bzip2, lzip, xz, and Zstandard-compressed sequence files;
- Meryl database statistics, histograms, printing, filtering, set operations,
  arithmetic operations, and lookup/import helper commands;
- local files plus upstream HTSlib HTTP(S) and S3 paths when explicitly used;
- native `linux/amd64` and `linux/arm64` containers.

This app does not:

- bundle Merqury, GenomeScope, plotting/report workflows, or reference data;
- enable the experimental upstream `meryl2` target, which upstream does not
  include in its public default build;
- automatically access the network, configure cloud credentials, or provide a
  CRAM reference.

## Container Contents

- `meryl`: default k-mer database command
- `meryl-lookup`: lookup and annotation helper
- `meryl-import`: import helper
- `meryl-analyze`: GC, GA, and GT motif analysis helper
- `meryl-simple`: simplified helper interface
- `position-lookup`: positional lookup helper
- `gzip`, `bzip2`, `lzip`, `xz`, and `zstd`: compressed-input helpers used by upstream code

## Usage

Count canonical 21-mers and create a Meryl database directory:

```sh
taf-meryl meryl k=21 memory=64 threads=8 count reads.fastq.gz output reads.meryl
```

Inspect the database:

```sh
taf-meryl meryl histogram reads.meryl > reads.hist
taf-meryl meryl statistics reads.meryl > reads.stats
taf-meryl meryl print reads.meryl > reads.kmers.tsv
taf-meryl meryl ploidy reads.meryl > reads.ploidy.tsv
```

Use a helper executable through command mode:

```sh
taf-meryl meryl-analyze -mers reads.meryl -prefix reads -gt
taf-meryl meryl-lookup
taf-meryl meryl-import
```

Access upstream identity and help:

```sh
taf-meryl -- --version
taf-meryl -- -h
taf-meryl meryl --version
```

`taf-meryl --help` shows this TAFFISH app manual. Meryl's normal positional
syntax begins with words such as `count` or `histogram`, so use the explicit
`meryl` executable as shown above. Use `--` only to pass option-leading
arguments such as `--version` or `-h` to the default command. Other packaged
executables such as `meryl-analyze` use automatic command mode.

## Inputs

| Input | Meaning | Notes |
| --- | --- | --- |
| FASTA/FASTQ | Sequence records to count | Plain, `.gz`, `.bz2`, `.lz`, `.xz`, and `.zst` inputs are supported |
| SAM/BAM/CRAM | Alignment records used as sequence input | HTSlib is embedded; CRAM may require an accessible reference |
| `.meryl` directory | Modern Meryl database | Keep the complete directory together |
| Histogram text | Frequency/count histogram | Accepted by reports such as `meryl ploidy` |

Relative paths under the current working directory are available inside the
TAFFISH container. Remote HTTP(S) or S3 paths are contacted only when supplied
to an upstream command. S3 credentials and remote-access policy remain the
user's responsibility.

## Output Notes

Modern Meryl databases are directories, not the legacy `.mcdat`/`.mcidx` file
pairs. A count command writes the requested database directory. Reporting
commands write text to standard output, so redirect them to files when needed.
`meryl-analyze` writes files derived from its `-prefix` argument in the current
working directory.

Upstream controls overwrite behavior. Use a fresh output path or remove an old
database deliberately before rerunning a count.

## Resources, Databases, and Platform

The app is built natively for `linux/amd64` and `linux/arm64`; no forced amd64
emulation is declared in `src/main.taf`. Counting cost depends on k-mer size,
input size, selected memory limit, and thread count. Set `memory=` and
`threads=` explicitly for production jobs.

No external database is required for ordinary Meryl use. Large sequence,
Meryl, or CRAM-reference files remain outside the image and should be supplied
through paths available to the container.

## Boundaries

The image preserves all commands built by the upstream public default target
and the runtime libraries/helpers they call. It does not promise every
project-specific Merqury pipeline, large-data performance profile, remote S3
configuration, or CRAM reference arrangement. Use the separate `taf-merqury`
app when the complete Merqury workflow is required.

## Troubleshooting

- If a `.meryl` input cannot be opened, confirm that the entire database
  directory is present and readable rather than only one internal file.
- If a CRAM or remote input fails, first verify the reference, credentials,
  network policy, and path visibility outside Meryl.
- `position-lookup` expects positional data inputs and is not useful as a
  no-argument help probe.

## Testing

The independent offline smoke suite checks:

- exact source/version provenance and all six packaged commands;
- ordinary help plus `ploidy` and `meryl-analyze -gt` discovery;
- dynamic libraries and compression helpers;
- real tiny FASTA and SAM counts;
- gzip, bzip2, lzip, xz, and Zstandard input paths;
- histogram, statistics, print, and GT motif outputs.

These tests verify packaging and small functional paths. They do not replace
large-scale biological validation, production memory sizing, remote-access
testing, or project-specific interpretation.

## License and Citation

TAFFISH app packaging is Apache-2.0. Upstream Meryl is distributed under its
public-domain notice, while bundled third-party code retains its own notices.
The image preserves both upstream `README.licenses` files.

Upstream asks users to cite Rhie et al. (2020), Genome Biology 21, 245:
[DOI 10.1186/s13059-020-02134-9](https://doi.org/10.1186/s13059-020-02134-9),
[PMID 32928274](https://pubmed.ncbi.nlm.nih.gov/32928274/).

- upstream repository: <https://github.com/marbl/meryl>
- upstream tag: <https://github.com/marbl/meryl/tree/v1.4.2>
