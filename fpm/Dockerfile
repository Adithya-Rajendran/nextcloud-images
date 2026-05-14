# syntax=docker/dockerfile:1
# Nextcloud PHP-FPM variant. Pairs with a separate web container (typically
# nginx) that proxies *.php to 127.0.0.1:9000 over loopback.
#
# Same extras as the apache variant — only the base image differs.
ARG NEXTCLOUD_VERSION=33.0.3
FROM nextcloud:${NEXTCLOUD_VERSION}-fpm

# Pull Debian security updates the upstream image may be lagging on, then
# install our extras. The dist-upgrade is what closes Debian-package CVEs
# without waiting for Nextcloud to rebuild.
RUN apt-get update \
 && apt-get -y dist-upgrade \
 && apt-get install -y --no-install-recommends \
      ffmpeg \
      libmagickcore-6.q16-6-extra \
      smbclient \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/* /tmp/*

# Build-time smoke test
RUN ffmpeg  -version >/dev/null \
 && ffprobe -version >/dev/null \
 && smbclient --version >/dev/null

# php-fpm.conf in the upstream image already listens on :9000 and runs as
# www-data; we don't override either.
