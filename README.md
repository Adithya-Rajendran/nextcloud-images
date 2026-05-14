# nextcloud-images

Custom container images for the Nextcloud stack, autobuilt and autotested.
Each top-level directory is one image variant with its own `Dockerfile` and
its own GitHub Actions workflow under `.github/workflows/`.

The [official `nextcloud/docker`](https://github.com/nextcloud/docker) and
[`collabora/code`](https://hub.docker.com/r/collabora/code) images are
deliberately minimal. These derivatives add the things downstream apps
actually need (ffmpeg for Memories, English fonts for Collabora) so the
runtime doesn't have to apt-get install on every pod start.

## Images

| Image | Source dir | Workflow | Notes |
|---|---|---|---|
| `ghcr.io/<OWNER>/nextcloud-apache:<NC_VERSION>-apache` | [`apache/Dockerfile`](apache/Dockerfile) | [`nextcloud-apache.yml`](.github/workflows/nextcloud-apache.yml) | Apache + PHP + ffmpeg, ffprobe, smbclient, libmagickcore-6-extra. `apt dist-upgrade` runs at build time. |
| `ghcr.io/<OWNER>/nextcloud-fpm:<NC_VERSION>-fpm` | [`fpm/Dockerfile`](fpm/Dockerfile) | [`nextcloud-fpm.yml`](.github/workflows/nextcloud-fpm.yml) | PHP-FPM only (no web server). Pairs with a separate nginx (or apache) container. Same extras as apache. |
| `ghcr.io/<OWNER>/collabora-fonts:<COL_VERSION>` | [`collabora/Dockerfile`](collabora/Dockerfile) | [`collabora.yml`](.github/workflows/collabora.yml) | Collabora Online + MS Core Fonts + Carlito/Caladea + Liberation/DejaVu/Noto/Roboto. |

Replace `<OWNER>` with whatever GitHub user/org owns this repo (the workflow
lowercases the owner before tagging — GHCR requires lowercase image names).

## How releases happen

1. Upstream releases — `nextcloud:33.0.4-{apache,fpm}` or `collabora/code:24.04.12.1.1`.
2. Renovate opens a PR bumping `ARG ..._VERSION=` in the affected Dockerfile.
3. The corresponding workflow runs — build → tests → (advisory) Trivy.
4. Tests pass → PR auto-merges.
5. Merge to main triggers the final build and push to GHCR.

Each image has an independent workflow, so an `fpm` regression doesn't block
an `apache` release and vice-versa.

## Tests

### `nextcloud-apache`

- Binaries: `ffmpeg`, `ffprobe`, `smbclient`
- PHP modules: `imagick`, `pdo_pgsql`, `gd`, `zip`, `intl`, `bcmath`
- Apache config: `apache2ctl configtest`
- Nextcloud version readable from `/usr/src/nextcloud/version.php`
- End-to-end boot against a real Postgres 16 — `/status.php` returns `installed:true`

### `nextcloud-fpm`

- Same binary + PHP-module checks as apache variant
- FPM config syntax: `php-fpm -t`
- Nextcloud version readable from `/usr/src/nextcloud/version.php`
- End-to-end boot via fpm + sidecar nginx pair against a real Postgres 16

### `collabora-fonts`

- Fonts present: Times New Roman, Carlito, Caladea, Liberation Sans, DejaVu
  Sans, Noto, Roboto
- Boot test: `/hosting/discovery` answers and the response XML contains
  the expected WOPI elements

Trivy runs on all three, **advisory only** — CVE findings appear in the run
logs but don't gate the merge. (Trade-off: Renovate auto-merge actually
works. The phpseclib CVE in Nextcloud's bundled `3rdparty/` cleared
automatically when this policy was relaxed.)

## Local dev

```bash
# Nextcloud-apache
docker build -t nextcloud-apache:dev apache/
docker run --rm nextcloud-apache:dev ffmpeg -version

# Nextcloud-fpm
docker build -t nextcloud-fpm:dev fpm/
docker run --rm nextcloud-fpm:dev php-fpm -t

# Collabora-fonts
docker build -t collabora-fonts:dev collabora/
docker run --rm collabora-fonts:dev sh -c 'fc-list | grep -i carlito'
```

## Adding a new image variant

```bash
mkdir my-variant
cat > my-variant/Dockerfile <<'EOF'
ARG FOO_VERSION=1.2.3
FROM upstream/foo:${FOO_VERSION}
RUN apt-get update && apt-get install -y --no-install-recommends bar \
 && rm -rf /var/lib/apt/lists/*
EOF
```

Then drop a `.github/workflows/my-variant.yml` modeled on the existing
three (copy + sed names). Renovate picks up the `FROM` line automatically.
Branch protection will need the new `build / my-variant` and
`test / my-variant` contexts added.

## Branch protection — required status checks

After adding a new image+workflow, add the new check-context names to the
`main` branch protection rule:

```powershell
$OWNER = gh api user --jq .login

@'
{
  "required_status_checks": {
    "strict": true,
    "contexts": [
      "build / apache",    "test / apache",
      "build / fpm",       "test / fpm",
      "build / collabora", "test / collabora"
    ]
  },
  "enforce_admins": false,
  "required_pull_request_reviews": null,
  "restrictions": null,
  "allow_force_pushes": false,
  "allow_deletions": false
}
'@ | gh api -X PUT "/repos/$OWNER/nextcloud-images/branches/main/protection" `
  -H "Accept: application/vnd.github+json" `
  --input -
```
