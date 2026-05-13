# syntax=docker/dockerfile:1
ARG NEXTCLOUD_VERSION=33.0.3
FROM nextcloud:${NEXTCLOUD_VERSION}-apache

# Extras the upstream image deliberately omits to keep the base small.
#
# - ffmpeg / ffprobe        : Memories video thumbnails + HLS transcoding
# - libmagickcore-6.q16-6-extra : PDF/HEIC/SVG support inside the already-loaded
#                             PHP-Imagick extension (no PECL rebuild needed)
# - smbclient               : SMB/CIFS external-storage backend
RUN apt-get update \
 && apt-get install -y --no-install-recommends \
      ffmpeg \
      libmagickcore-6.q16-6-extra \
      smbclient \
 && rm -rf /var/lib/apt/lists/* /tmp/*

# Build-time smoke test — if any of these fail, the build fails.
RUN ffmpeg  -version >/dev/null \
 && ffprobe -version >/dev/null \
 && smbclient --version >/dev/null
