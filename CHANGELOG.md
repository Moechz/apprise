# Changelog

All notable user-visible changes to the Apprise API TOS application package.

## 2.1.0-2 — 2026-11-21

Remediation of the App Center listing review (aarch64 on-hold, tracking
`TOS-APP-CL-20261121-01`). Upstream base is unchanged: still
`linuxserver/apprise-api:2.1.0` (no newer upstream release is published, so
there is nothing to bump for T1; the image is used unmodified).

### Fixed
- **C10 — dead help link.** The Help entry pointed at
  `https://github.com/Moechz/appriseapi/issues`, which returns HTTP 404. It now
  opens the public privacy policy at
  `https://github.com/Moechz/apprise/blob/main/PRIVACY.md` (both
  `config.ini.help` and `tosapp.json.help`).
- **C10 / C02 — metadata mismatch and redundant key.** `config.ini.official`
  pointed at `caronc/apprise-api` while the package ships the LinuxServer.io
  rebuild; it now points at `https://github.com/linuxserver/docker-apprise-api`
  and the duplicate `Official` key was removed. The description states that
  this is the LinuxServer.io rebuild.
- **C3 — missing privacy policy.** Added `PRIVACY.md`: what is stored under
  `/config`, which endpoints receive data, retention, user rights (access /
  export / deletion), security measures and the deletion channel.
- **C8 — third-party API disclosure.** `PRIVACY.md` carries an explicit
  endpoint → data → destination/region table (Telegram, Discord, Slack, Teams,
  SMTP, webhooks, ntfy, Gotify, Mailgun, …); the app description summarises
  that message data and saved credentials go only to the services the user
  configures.
- **C9 — wrong `auth` field.** Every language section of `apprisedocker.lang`
  declared `auth = "caronc"` (the upstream author) although the publisher is
  Moechz; all 23 sections now declare `auth = "Moechz"`.
- **C9 — wrong configuration storage path.** The description claimed the
  configuration is stored in `config.yml`, a file that does not exist. It now
  says the configuration lives under `/config` (one file per saved destination,
  e.g. `/config/<id>/<sha256>.text`; `{KEY}.cfg` / `{KEY}.yml` with
  `APPRISE_STATEFUL_MODE=simple`).
- **S4 — unauthenticated web interface holding third-party credentials.**
  HTTP Basic Auth is now enabled by default (`APPRISE_AUTH_REQUIRED=yes`).
  Because the image has no built-in account, a first-start wrapper generates a
  random administrator password and stores it in
  `/config/admin-credentials.txt` (mode 600, username `admin`); deleting that
  file and restarting rotates it. Only one port stays published, and the
  description/README state that the UI is plain HTTP for LAN use and needs a
  TLS reverse proxy for remote access.

### Changed
- `README.md`: added **Repository layout** and **Privacy** sections, updated the
  runtime file manifest (configuration layout, admin credentials, web-session
  key, notification state), the authentication and verification instructions.
- `tosapp.json` / `apprisedocker.lang`: new `descript`, `important` and
  `release_note` for all 23 languages (en, zh-CN, zh-HK translated; the rest
  fall back to English as the TOS guide allows).
- Package version `2.1.0-1` → `2.1.0-2`; both architectures built and
  resubmitted.

## 2.1.0-1 — 2026-11-14

### Added
- First release. Apprise API (`linuxserver/apprise-api:2.1.0`, Docker Hub official image, unmodified)
  packaged as a TOS 7 Docker application, published on port 8108 and running
  as a non-root user (1000:1000).
- Persistent data is kept under `/Volume1/DockerAppData/apprisedocker/`.
