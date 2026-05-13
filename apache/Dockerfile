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
    && apt-get -y upgrade \
    apt-get install -y --no-install-recommends ffmpeg smbclient; \
    magick_extra=$(apt-cache search --names-only '^libmagickcore-[0-9]+\.q16(hdri)?-[0-9]+(t64)?-extra$' \
                   | awk '{print $1}' | sort -Vr | head -n1); \
    [ -n "$magick_extra" ] || { echo "no libmagickcore-*-extra package found" >&2; exit 1; }; \
    apt-get install -y --no-install-recommends "$magick_extra"; \
    rm -rf /var/lib/apt/lists/* /tmp/*

# Build-time smoke test — if any of these fail, the build fails.
RUN ffmpeg  -version >/dev/null \
 && ffprobe -version >/dev/null \
 && smbclient --version >/dev/null
