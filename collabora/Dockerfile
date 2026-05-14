# syntax=docker/dockerfile:1
# Collabora Online Development Edition (the "code" image) with English fonts
# baked in. Replaces an initContainer that did apt-get install on every pod
# start (~45s, ~150 MB pulled each restart).
#
# Tag note: Collabora's apt repo's "latest" pointer moves frequently; pin to
# a specific version below and let Renovate bump it.
ARG COLLABORA_VERSION=25.04.9.4.1
FROM collabora/code:${COLLABORA_VERSION}

USER root

# Microsoft Core Fonts (Debian non-free, EULA auto-accepted via DEBIAN_FRONTEND=noninteractive)
#   ttf-mscorefonts-installer — Arial, Times New Roman, Courier New, Verdana, etc.
# Metric-compatible substitutes for Office formats that don't include MS fonts:
#   fonts-crosextra-carlito  — Calibri replacement (Google Carlito)
#   fonts-crosextra-caladea  — Cambria replacement (Google Caladea)
# Common OSS English fonts:
#   fonts-dejavu, fonts-liberation, fonts-liberation2, fonts-noto-core, fonts-roboto
RUN apt-get update \
 && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
      ttf-mscorefonts-installer \
      fonts-crosextra-carlito \
      fonts-crosextra-caladea \
      fonts-dejavu \
      fonts-liberation \
      fonts-liberation2 \
      fonts-noto-core \
      fonts-roboto \
 && fc-cache -f \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/* /tmp/*

# Build-time smoke test: representative fonts from each group must be visible.
RUN fc-list | grep -qi "Times New Roman"  \
 && fc-list | grep -qi "Carlito"          \
 && fc-list | grep -qi "Caladea"          \
 && fc-list | grep -qi "Liberation"       \
 && fc-list | grep -qi "DejaVu"

USER cool
