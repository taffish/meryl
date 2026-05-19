# meryl

TAFFISH wrapper for [Meryl](https://github.com/marbl/meryl), a genomic k-mer
counter and sequence utility used by Merqury and related assembly-evaluation
workflows.

This app packages upstream Meryl `1.4.1` from the official
`meryl-1.4.1.Linux-amd64.tar.xz` release asset.

Release `1.4.1-r2` is a help-only TAFFISH update. It keeps the upstream
software, Dockerfile, runtime dependencies, smoke tests, platform declaration,
and command behavior unchanged from `1.4.1-r1`, and refreshes the terminal
`taf-meryl --help` text.

Package metadata:

```text
name: meryl
command: taf-meryl
version: 1.4.1-r2
kind: tool
image: ghcr.io/taffish/meryl:1.4.1-r2
upstream release: v1.4.1
upstream runtime version: meryl 1.4.1
```

## Install

```sh
taf install meryl
```

Then run:

```sh
taf-meryl --help
taf-meryl --version
taf-meryl -- --version
taf-meryl meryl --version
```

`taf-meryl --help` shows TAFFISH wrapper help. Use `--` when the first upstream
argument starts with `-`, for example `taf-meryl -- --version`. Use command mode
for commands inside the same container, for example `taf-meryl meryl-lookup`.

## Commands

The container includes the public commands from the official Meryl Linux-amd64
binary release:

- `meryl`
- `meryl-lookup`
- `meryl-import`
- `meryl-analyze`
- `meryl-simple`
- `position-lookup`

The default command is `meryl`:

```sh
taf-meryl -- k=21 memory=64 threads=8 count reads.fastq.gz output reads.meryl
taf-meryl -- histogram reads.meryl > reads.hist
taf-meryl -- statistics reads.meryl > reads.stats
taf-meryl -- print reads.meryl > reads.kmers
```

The same calls can be written in explicit command mode:

```sh
taf-meryl meryl k=21 memory=64 threads=8 count reads.fastq.gz output reads.meryl
taf-meryl meryl histogram reads.meryl
taf-meryl meryl statistics reads.meryl
taf-meryl meryl print reads.meryl
```

Helper commands are exposed through command mode:

```sh
taf-meryl meryl-lookup
taf-meryl meryl-import
taf-meryl meryl-analyze
taf-meryl meryl-simple
```

`position-lookup` is included because it is part of the official binary release,
but it expects valid positional inputs and is not useful as a no-argument help
probe.

## Data Model

Modern Meryl databases are directories, not the old `.mcdat` and `.mcidx` files
from Canu 1.8-era Meryl. Keep the whole `.meryl` directory together when moving
or mounting results.

Typical count output:

```sh
taf-meryl meryl k=21 memory=64 count reads.fastq.gz output reads.meryl
```

The output directory is overwritten only as allowed by upstream Meryl. Choose a
fresh output path for repeatable runs.

## Platform

This release is native `linux/amd64` only. The upstream Meryl `1.4.1` release
ships a Linux-amd64 binary asset and Darwin-amd64 asset, but no Linux-arm64
binary asset. Source/native arm64 builds are not claimed here because the
current upstream source path still depends on x86 SSE code in parts of the
alignment utility code.

For Docker and Podman, the TAFFISH wrapper declares `--platform linux/amd64` in
`src/main.taf`, so arm64 hosts can run it through amd64 emulation when the
container backend supports that. This is emulated execution, not native
`linux/arm64` support. Apptainer behavior on non-amd64 hosts depends on the
site runtime and is not declared as supported by this app.

## Smoke Coverage

The app smoke tests check:

- packaged version marker and `meryl --version`;
- main help and helper command usage output;
- presence of all official binary commands;
- runtime dynamic dependency on `libgomp`;
- compression utilities `gzip`, `bzip2`, and `xz`;
- a tiny real `meryl count`, followed by `histogram`, `statistics`, and `print`.

These checks verify packaging and a minimal functional path. They do not replace
large-scale biological validation, memory sizing tests, or project-specific
quality-control interpretation.

## Boundaries

This app packages Meryl itself. It does not include Merqury, GenomeScope,
assembly plotting/report workflows, or large example datasets. Use the separate
`taf-merqury` app for the Merqury assembly-evaluation workflow.

## Upstream

- Upstream repository: <https://github.com/marbl/meryl>
- Release: <https://github.com/marbl/meryl/releases/tag/v1.4.1>
- License: public domain notice in upstream `README.licenses`, with additional
  notices as indicated by upstream
- Citation: Rhie et al. 2020, Genome Biology 21, 245
- DOI: <https://doi.org/10.1186/s13059-020-02134-9>
- PMID: 32928274
