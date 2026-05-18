ARG MERYL_VERSION=1.4.1
ARG MERYL_LINUX_AMD64_SHA256=fe5e81f97994461bd30adab98359b17aebcd6853afd1afe27c609fa6d27ec9ec
ARG DEBIAN_MIRROR=http://deb.debian.org/debian
ARG DEBIAN_SECURITY_MIRROR=http://deb.debian.org/debian-security

FROM debian:12-slim AS fetcher

ARG MERYL_VERSION
ARG MERYL_LINUX_AMD64_SHA256
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
            ca-certificates \
            curl \
            xz-utils; then \
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
      "https://github.com/marbl/meryl/releases/download/v${MERYL_VERSION}/meryl-${MERYL_VERSION}.Linux-amd64.tar.xz" \
      -o meryl.tar.xz; \
    echo "${MERYL_LINUX_AMD64_SHA256}  meryl.tar.xz" | sha256sum -c -; \
    mkdir -p /opt/meryl; \
    tar -xJf meryl.tar.xz --strip-components=1 -C /opt/meryl; \
    test -x /opt/meryl/bin/meryl; \
    test -x /opt/meryl/bin/meryl-lookup; \
    test -x /opt/meryl/bin/meryl-import; \
    test -x /opt/meryl/bin/meryl-analyze; \
    test -x /opt/meryl/bin/meryl-simple; \
    test -x /opt/meryl/bin/position-lookup

FROM debian:12-slim

ARG MERYL_VERSION
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
            bash \
            bzip2 \
            ca-certificates \
            gzip \
            libgcc-s1 \
            libgomp1 \
            libstdc++6 \
            xz-utils; then \
        break; \
      fi; \
      if [ "$attempt" = "6" ]; then exit 1; fi; \
      apt-get clean; \
      rm -rf /var/lib/apt/lists/*; \
      sleep 5; \
    done; \
    rm -rf /var/lib/apt/lists/*

COPY --from=fetcher /opt/meryl /opt/meryl

ENV PATH=/opt/meryl/bin:${PATH}

RUN set -eux; \
    mkdir -p /usr/local/share/doc/meryl; \
    install -m 0644 /opt/meryl/README.md /opt/meryl/README.licenses /usr/local/share/doc/meryl/; \
    printf "%s\n" "${MERYL_VERSION}" > /usr/local/share/doc/meryl/VERSION; \
    test "$(cat /usr/local/share/doc/meryl/VERSION)" = "${MERYL_VERSION}"; \
    meryl --version 2>&1 | grep -F "meryl ${MERYL_VERSION}" >/dev/null; \
    meryl -h 2>&1 | grep -F "COMMANDS:" >/dev/null; \
    meryl-lookup 2>&1 | grep -F "usage:" >/dev/null; \
    meryl-import 2>&1 | grep -F "usage:" >/dev/null; \
    meryl-simple 2>&1 | grep -F "usage:" >/dev/null; \
    meryl-analyze 2>&1 | grep -F "usage:" >/dev/null; \
    command -v position-lookup; \
    command -v gzip; \
    command -v bzip2; \
    command -v xz; \
    ldd "$(command -v meryl)" | grep -F "libgomp" >/dev/null

RUN set -eux; \
    mkdir -p /tmp/meryl-check; \
    cd /tmp/meryl-check; \
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
    rm -rf /tmp/meryl-check

WORKDIR /root

ENV TAFFISH_ENV=TAFFISH
ENV TAFFISH_NAME=meryl
