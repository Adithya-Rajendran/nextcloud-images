# nextcloud-images

Custom Nextcloud Docker images, autobuilt and autotested.

Each subdirectory at the repo root is one image variant. The
[official Nextcloud Docker images](https://hub.docker.com/_/nextcloud) are
deliberately minimal — these add the binaries downstream apps actually need
(ffmpeg, smbclient, imagemagick extras) without the user having to maintain a
Dockerfile.

| Image | Source | Notes |
|---|---|---|
| `ghcr.io/<OWNER>/nextcloud-apache:<NC_VERSION>-apache` | [`apache/Dockerfile`](apache/Dockerfile) | Apache + PHP + ffmpeg, ffprobe, smbclient, libmagickcore-6-extra |

Replace `<OWNER>` with whatever GitHub user/org owns this repo.

## How it stays current

1. Upstream Nextcloud releases (say, `33.0.4-apache`).
2. **Renovate** opens a PR bumping the `ARG NEXTCLOUD_VERSION=` default plus
   the pinned image digest.
3. **CI builds and tests** the new image:
   - binaries present (`ffmpeg`, `ffprobe`, `smbclient`)
   - PHP modules present (`imagick`, `pdo_pgsql`, `gd`, `zip`, `intl`, `bcmath`)
   - `apache2ctl configtest` passes
   - `occ --version` runs
   - container boots end-to-end against a real Postgres, `status.php` returns
     `installed:true`
   - Trivy finds no fixable HIGH/CRITICAL CVEs
4. All green → **the PR auto-merges**.
5. Merge to `main` triggers the final build that pushes
   `ghcr.io/<OWNER>/nextcloud-apache:33.0.4-apache`.
6. You bump `image.tag` in your cluster's values.yaml and `helm upgrade`.

Major version bumps (`33.x → 34.x`) explicitly **do not** automerge — they get
a `nextcloud-major needs-review` label and wait for you.

## Local dev

```bash
docker build -t nextcloud-apache:dev apache/
docker run --rm nextcloud-apache:dev ffmpeg -version
```

## Adding a new variant (e.g. `fpm`)

```bash
mkdir fpm
cat > fpm/Dockerfile <<'EOF'
ARG NEXTCLOUD_VERSION=33.0.3
FROM nextcloud:${NEXTCLOUD_VERSION}-fpm
RUN apt-get update && apt-get install -y --no-install-recommends \
      ffmpeg smbclient && rm -rf /var/lib/apt/lists/*
EOF
```

Then add `"fpm"` to the `matrix.variant` list in
`.github/workflows/build.yml`. That's it — Renovate will track the fpm tag
independently, CI will build+test it on every Nextcloud release.

## Why a monorepo for these

If you later want an fpm variant, an alpine variant, a memories-tuned variant
with QSV/VAAPI drivers, or a pdlib-included variant for face recognition,
each lives in its own subdirectory and shares the build/test/release plumbing.
One repo, one Renovate config, one workflow, N images.
