ARG MERYL_VERSION=1.4.2
ARG MERYL_COMMIT=dbacc3d0fd26bcd4247dcaf5b7e70ffdf5f69831
ARG MERYL_SOURCE_SHA256=97b484060681aef41b5ee963d9e3849b50b5ed063e4e6782667d45673f9c0792
ARG MERYL_UTILITY_COMMIT=f800fc4dada365a701122b5ac1ecb946eb402cb4
ARG MERYL_UTILITY_SOURCE_SHA256=e930270630d28f12152fdc1e38e94ec776ec50ab38d3830558b0e6422d15019b
ARG DEBIAN_MIRROR=http://deb.debian.org/debian
ARG DEBIAN_SECURITY_MIRROR=http://deb.debian.org/debian-security

FROM debian:12-slim AS builder

ARG MERYL_VERSION
ARG MERYL_COMMIT
ARG MERYL_SOURCE_SHA256
ARG MERYL_UTILITY_COMMIT
ARG MERYL_UTILITY_SOURCE_SHA256
ARG DEBIAN_MIRROR
ARG DEBIAN_SECURITY_MIRROR

ENV DEBIAN_FRONTEND=noninteractive

RUN set -eux; \
    sed -i \
      -e "s|http://deb.debian.org/debian-security|${DEBIAN_SECURITY_MIRROR}|g" \
      -e "s|http://deb.debian.org/debian|${DEBIAN_MIRROR}|g" \
      /etc/apt/sources.list.d/debian.sources; \
    for attempt in 1 2 3 4 5 6; do \
      if apt-get -o Acquire::Retries=3 update \
       && apt-get -o Acquire::Retries=3 install -y --no-install-recommends \
            build-essential \
            ca-certificates \
            curl \
            git \
            gzip \
            libbz2-dev \
            libcurl4-openssl-dev \
            liblzma-dev \
            libssl-dev \
            perl \
            pkg-config \
            zlib1g-dev; then \
        break; \
      fi; \
      if [ "$attempt" = "6" ]; then exit 1; fi; \
      apt-get clean; \
      rm -rf /var/lib/apt/lists/*; \
      sleep 5; \
    done; \
    rm -rf /var/lib/apt/lists/*

WORKDIR /tmp/build

RUN set -eux; \
    curl --retry 5 --retry-delay 2 --retry-all-errors -fsSL \
      "https://codeload.github.com/marbl/meryl/tar.gz/${MERYL_COMMIT}" \
      -o meryl.tar.gz; \
    echo "${MERYL_SOURCE_SHA256}  meryl.tar.gz" | sha256sum -c -; \
    curl --retry 5 --retry-delay 2 --retry-all-errors -fsSL \
      "https://codeload.github.com/marbl/meryl-utility/tar.gz/${MERYL_UTILITY_COMMIT}" \
      -o meryl-utility.tar.gz; \
    echo "${MERYL_UTILITY_SOURCE_SHA256}  meryl-utility.tar.gz" | sha256sum -c -; \
    mkdir -p meryl/src/utility; \
    tar -xzf meryl.tar.gz --strip-components=1 -C meryl; \
    tar -xzf meryl-utility.tar.gz --strip-components=1 -C meryl/src/utility; \
    grep -Fx "VERSION      := release v${MERYL_VERSION}" meryl/src/main.mk >/dev/null; \
    make -C meryl/src -j "$(nproc)"; \
    test -x meryl/build/bin/meryl; \
    test -x meryl/build/bin/meryl-lookup; \
    test -x meryl/build/bin/meryl-import; \
    test -x meryl/build/bin/meryl-analyze; \
    test -x meryl/build/bin/meryl-simple; \
    test -x meryl/build/bin/position-lookup; \
    for binary in meryl/build/bin/*; do \
      strip --strip-unneeded "$binary"; \
    done; \
    rm -rf meryl/build/lib meryl/build/obj; \
    mkdir -p /opt/meryl; \
    cp -a meryl/build/bin /opt/meryl/; \
    if [ -d meryl/build/share ]; then cp -a meryl/build/share /opt/meryl/; fi; \
    install -m 0644 meryl/README.md meryl/README.licenses /opt/meryl/; \
    install -m 0644 meryl/src/utility/README.licenses /opt/meryl/README.meryl-utility.licenses

FROM debian:12-slim

ARG MERYL_VERSION
ARG MERYL_COMMIT
ARG MERYL_SOURCE_SHA256
ARG MERYL_UTILITY_COMMIT
ARG MERYL_UTILITY_SOURCE_SHA256
ARG DEBIAN_MIRROR
ARG DEBIAN_SECURITY_MIRROR
ARG TARGETARCH

ENV DEBIAN_FRONTEND=noninteractive

RUN set -eux; \
    sed -i \
      -e "s|http://deb.debian.org/debian-security|${DEBIAN_SECURITY_MIRROR}|g" \
      -e "s|http://deb.debian.org/debian|${DEBIAN_MIRROR}|g" \
      /etc/apt/sources.list.d/debian.sources; \
    for attempt in 1 2 3 4 5 6; do \
      if apt-get -o Acquire::Retries=3 update \
       && apt-get -o Acquire::Retries=3 install -y --no-install-recommends \
            bash \
            bzip2 \
            ca-certificates \
            gzip \
            libbz2-1.0 \
            libcurl4 \
            libgcc-s1 \
            libgomp1 \
            liblzma5 \
            libssl3 \
            libstdc++6 \
            lzip \
            xz-utils \
            zlib1g \
            zstd; then \
        break; \
      fi; \
      if [ "$attempt" = "6" ]; then exit 1; fi; \
      apt-get clean; \
      rm -rf /var/lib/apt/lists/*; \
      sleep 5; \
    done; \
    rm -rf /var/lib/apt/lists/*

COPY --from=builder /opt/meryl /opt/meryl

ENV PATH=/opt/meryl/bin:${PATH}

RUN set -eux; \
    mkdir -p /usr/local/share/doc/meryl; \
    install -m 0644 \
      /opt/meryl/README.md \
      /opt/meryl/README.licenses \
      /opt/meryl/README.meryl-utility.licenses \
      /usr/local/share/doc/meryl/; \
    printf "%s\n" "${MERYL_VERSION}" > /usr/local/share/doc/meryl/VERSION; \
    { \
      printf "upstream_repo=https://github.com/marbl/meryl\n"; \
      printf "upstream_tag=v%s\n" "${MERYL_VERSION}"; \
      printf "upstream_commit=%s\n" "${MERYL_COMMIT}"; \
      printf "source_sha256=%s\n" "${MERYL_SOURCE_SHA256}"; \
      printf "meryl_utility_commit=%s\n" "${MERYL_UTILITY_COMMIT}"; \
      printf "meryl_utility_source_sha256=%s\n" "${MERYL_UTILITY_SOURCE_SHA256}"; \
      printf "target_arch=%s\n" "${TARGETARCH}"; \
    } > /usr/local/share/doc/meryl/SOURCE; \
    test "$(cat /usr/local/share/doc/meryl/VERSION)" = "${MERYL_VERSION}"; \
    meryl --version 2>&1 | grep -Fx "meryl ${MERYL_VERSION}" >/dev/null; \
    meryl -h 2>&1 | grep -F "COMMANDS:" >/dev/null; \
    meryl -h 2>&1 | grep -F "ploidy" >/dev/null; \
    meryl-lookup 2>&1 | grep -F "usage:" >/dev/null; \
    meryl-import 2>&1 | grep -F "usage:" >/dev/null; \
    meryl-simple 2>&1 | grep -F "usage:" >/dev/null; \
    meryl-analyze 2>&1 | grep -F -- "-gt" >/dev/null; \
    command -v position-lookup >/dev/null; \
    command -v gzip >/dev/null; \
    command -v bzip2 >/dev/null; \
    command -v lzip >/dev/null; \
    command -v xz >/dev/null; \
    command -v zstd >/dev/null; \
    if ldd "$(command -v meryl)" | grep -F "not found"; then exit 1; fi; \
    ldd "$(command -v meryl)" | grep -F "libgomp" >/dev/null; \
    ldd "$(command -v meryl)" | grep -F "libcurl" >/dev/null

RUN set -eux; \
    tmp="/tmp/meryl-build-check"; \
    rm -rf "$tmp"; \
    mkdir -p "$tmp"; \
    cd "$tmp"; \
    printf ">toy\nACGTACGTACGTACGTACGTACGTACGTACGT\n" > seq.fa; \
    meryl k=21 memory=0.1 threads=1 count seq.fa output seq.meryl >/dev/null 2> meryl.log; \
    test -d seq.meryl; \
    meryl histogram seq.meryl > seq.hist; \
    grep -E "^6[[:space:]]+2$" seq.hist >/dev/null; \
    meryl statistics seq.meryl > seq.stats; \
    grep -F "distinct                    2" seq.stats >/dev/null; \
    meryl print seq.meryl > seq.print; \
    grep -F "ACGTACGTACGTACGTACGTA" seq.print >/dev/null; \
    cd /; \
    rm -rf "$tmp"

WORKDIR /root

ENV TAFFISH_ENV=TAFFISH
ENV TAFFISH_NAME=meryl
