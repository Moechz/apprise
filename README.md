# apprise-for-tos

> TerraMaster TOS 7 App Center — **Docker application** package for
> [Apprise API](https://github.com/caronc/apprise-api), delivered as the
> unmodified LinuxServer.io image `linuxserver/apprise-api:2.1.0`.

Current package version: **2.1.0-2** (dual architecture: `x86_64`, `aarch64`).

## What this is

A four-file TOS Docker application archive (`apprisedocker.tar.gz`) that installs
Apprise API on TOS 7 through the App Center:

- `config.ini` — TOS application metadata (`application_type: docker`, id `apprisedocker`)
- `apprisedocker.lang` — 23-language superset (TOS requires 14)
- `apprisedocker.svg` — application icon (3921 bytes, 6 SVG elements)
- `docker-compose.yml` — one service, image `linuxserver/apprise-api:2.1.0`

## Repository layout

```
.
├── config.ini                     TOS metadata: id, version, help, official,
│                                  publisher, port 8108, DockerEngine dependency
├── apprisedocker.lang             UI strings for 23 languages (name, auth,
│                                  descript, release_note, important)
├── apprisedocker.svg              App Center icon (viewBox 0 0 512 512)
├── docker-compose.yml             one non-root service + HTTP Basic Auth
├── PRIVACY.md                     privacy policy: local data, outbound
│                                  endpoints, retention, rights, security
├── README.md                      this file
├── CHANGELOG.md                   per-version package changes
├── src/                           files copied into the archive
├── scripts/
│   ├── build.sh                   single-architecture build (self-check)
│   └── build-dual.sh              both architectures + official `-c` check
├── test/install-and-verify.sh     on-device install/verify helper (needs root)
├── tools/verify.py                review-standards self-check used by build.sh
└── TOSAppSelfTestingTool          official TerraMaster package checker
```

The published GitHub release carries `apprisedocker_x86_64.tar.gz` and
`apprisedocker_aarch64.tar.gz` (plus `.sha256`); the two architectures are
submitted as two App records with the same `app_id`.

## How it works

| | |
|---|---|
| Container image | `linuxserver/apprise-api:2.1.0` (Docker Hub, used unmodified) |
| Open | `http://<NAS-IP>:8108/` (container port 8000, HTTP Basic Auth) |
| Runs as | non-root, `user: "1000:1000"` |
| Persistent data | `/config` → `/Volume1/DockerAppData/apprisedocker/config` |
| Logs | `docker logs apprisedocker` (stdout/stderr) |

`/config` (mounted from `DockerAppData/apprisedocker/config`) holds the saved
notification destinations, one file per Apprise URL — for example
`/config/b3/<sha256>.text`; with `APPRISE_STATEFUL_MODE=simple` they are
`/config/{KEY}.cfg` or `/config/{KEY}.yml`. Notification state and reminders
live under `/config/store`.

### Authentication

Apprise API stores third-party credentials (bot tokens, webhook secrets, SMTP
passwords), so this package enables its built-in HTTP Basic Auth
(`APPRISE_AUTH_REQUIRED=yes`). Upstream ships no built-in account, so the
package wrapper generates a random administrator password on first start and
writes it to
`/Volume1/DockerAppData/apprisedocker/config/admin-credentials.txt` (mode 600,
username `admin`). Delete that file and restart the application to generate a
new password. Unauthenticated requests receive `401`; individual configurations
can still be shared with their own credentials, or marked `public` for tagged
`/notify` calls, through the web interface.

## Design highlights

- **Non-root.** `user: "1000:1000"`; verified on real hardware (the container reports uid 1000).
- **Fixed image tag, never `:latest`**, Docker Hub only, no `privileged`, no `network_mode: host`, and no Docker socket, no `cap_add`.
- **Single published port** (8108 → 8000), no host networking.
- **HTTP Basic Auth on by default**, random password generated on first start.
- **Health check** probing the service on container port 8000.
- **Deterministic build**: GNU tar with uid/gid 0 and zeroed timestamps, gzip without a timestamp, plus the review-standards self-check.
- **One endpoint, many services.** Every destination is a URL; tags group them for scripts.

## Runtime file manifest

Everything the application writes on the NAS lives inside its own data
directory; the container-internal `/tmp` is discarded with the container.

| Path | Purpose | Created when | Growth bound | Lifecycle |
|---|---|---|---|---|
| `/Volume1/DockerAppData/apprisedocker/config` | Apprise API application data (everything the container writes outside /tmp) | on first start / on demand | bounded by your own data | persistent; kept on uninstall unless you ask to delete it |
| `…/config/admin-credentials.txt` | generated `admin` password for the web interface (mode 600) | first start | one short text file | survives restarts; delete + restart to rotate |
| `…/config/.web_auth_secret` | signing key for the browser login session | first start | one 64-byte file | persistent |
| `…/config/<id>/*.text` (or `{KEY}.cfg` / `{KEY}.yml`) | notification destinations and their credentials | when you save a configuration | your data | delete a key in the UI, or the whole directory |
| `…/config/store` | optional persistent notification state (reminders, delivery state) | on demand | pruned after 30 days by default | automatic |

## Privacy

See [PRIVACY.md](PRIVACY.md) for the full policy. In short:

- Apprise API runs entirely on your NAS; the package has **no telemetry** and
  sends nothing to the package author or TerraMaster.
- Message text and the credentials you save are sent **only** to the
  notification services you configure (Telegram, Discord, Slack, SMTP,
  webhooks, ntfy, Gotify, Mailgun, …). Each delivery goes to that provider in
  the region the provider operates; the package does not pin a region or route
  data through an intermediary. The full endpoint/data/destination table is in
  [PRIVACY.md §3](PRIVACY.md#3-data-sent-to-third-parties-api-disclosure).
- Configurations, credentials and the generated admin password stay under
  `/Volume1/DockerAppData/apprisedocker/config`; you can read, export
  (`POST /cfg/{KEY}`) or delete (`POST /del/{KEY}`, or delete the directory)
  them at any time.
- The web interface is HTTP on the LAN, protected by Basic Auth. For remote
  access use your own TLS reverse proxy and do not expose port 8108 directly.

## Build

```bash
scripts/build.sh 2.1.0-2                  # single arch (tosapp.json platform)
scripts/build.sh 2.1.0-2 aarch64          # explicit architecture
bash scripts/build-dual.sh 2.1.0-2        # both arches + official check
```

The build stages the four files, substitutes the version/platform placeholders
and the embedded entrypoint, removes CRLF/BOM/AppleDouble, then runs
`tools/verify.py`, a review-standards self-check (archive layout, JSON validity,
required fields, 14+ language coverage, version consistency, Docker-Hub-only
images, fixed tag, reserved/recommended host ports, non-root user, per-service
healthcheck/TZ/restart, data path below the application data root, never the
`/Volume*` wildcard, comment-free sequence blocks, `x-app-meta` position, no
literal secrets and the SVG size/element limits).

## Local verification

```bash
./TOSAppSelfTestingTool -c out/apprisedocker_x86_64.tar.gz   # official spec check
./TOSAppSelfTestingTool -c out/apprisedocker_aarch64.tar.gz
bash test/install-and-verify.sh                  # on-device E2E (needs root)
```

> Docker applications cannot be side-loaded through the App Center's manual
> install page (it accepts `.deb` only); the official self-testing tool's `-i`
> mode or `docker compose` are the only ways to install one for testing.

## Store submission checklist

1. Public GitHub repo (code + README + PRIVACY).
2. Release **tag `v2.1.0-2`**, assets `apprisedocker_x86_64.tar.gz` /
   `apprisedocker_aarch64.tar.gz` + `.sha256` files.
3. Developer platform → Add Application: ID `apprisedocker`, package type Docker,
   repository URL, architecture `x86_64` and `aarch64`.
4. Version Management → Add Version `2.1.0-2` → automated validation → review.

## License / branding

Apprise API is distributed by its upstream authors under **MIT**. The container image is the LinuxServer.io rebuild from Docker Hub (`linuxserver/apprise-api:2.1.0`) and is used unmodified; no upstream source code is redistributed here. This packaging is an independent community submission by Moechz and is not affiliated with the upstream project or TerraMaster.

## Limitations

- HTTP Basic Auth is enabled, but the transport is plain HTTP on the LAN; use a TLS reverse proxy for remote access.
- Delivering to a service depends on outbound network access from the NAS and on the third-party service being reachable.
- No image vulnerability scan / SBOM is bundled with this package; the image is the unmodified LinuxServer.io build, and scan output must be produced on a host that has trivy/grype.

---

Packaged for TOS 7 by Moechz. Upstream image used unmodified from Docker Hub.
