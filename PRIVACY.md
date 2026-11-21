# Privacy Policy — Apprise API for TerraMaster TOS 7

This policy covers the TOS 7 application package **Apprise API** (App Center
ID `apprisedocker`, published by **Moechz**). It describes what the packaged
application stores on your NAS, what it sends over the network, and how you can
inspect or delete it.

The application itself is the open-source **Apprise API** project
(https://github.com/caronc/apprise-api, MIT). The container image is the
unmodified **LinuxServer.io** rebuild `linuxserver/apprise-api:2.1.0`
(https://github.com/linuxserver/docker-apprise-api). This package adds only
TOS metadata, the data directory mapping and a first-start wrapper.

Last updated: 2026-11-21 (package version 2.1.0-2).

---

## 1. What the application does

Apprise API is a notification gateway that runs on your own NAS:

- it accepts an HTTP request (`POST /notify/<tag>`) from a script or another
  application on your network;
- it forwards the message body and title to the notification services that
  **you** configured (Telegram, Discord, Slack, e-mail/SMTP, ntfy, Gotify,
  generic webhooks and 100+ other Apprise services);
- it serves a web interface on `http://<NAS-IP>:8108/` where you add, test and
  delete those destinations.

There is no Apprise account and no vendor cloud: the whole configuration lives
on your NAS.

## 2. Data stored on your NAS

All persistent data is kept below `/Volume1/DockerAppData/apprisedocker/config`
(container path `/config`):

| What | Where | Why |
|---|---|---|
| Notification destinations (Apprise URLs, tags) | `/config/<id>/…` — with `APPRISE_STATEFUL_MODE=simple` instead `/config/{KEY}.cfg` or `/config/{KEY}.yml` | so `/notify/<tag>` knows where to deliver |
| Service credentials embedded in those URLs (bot tokens, webhook secrets, SMTP user/password, API keys) | same files, plain text, directory mode `700` | the services require them to authenticate the delivery |
| Web-interface password (HTTP Basic Auth) | `/config/admin-credentials.txt`, mode `600`, generated on first start | to keep other hosts on the LAN out of the configuration |
| Browser-session signing key | `/config/.web_auth_secret` | signs the web login session |
| Optional persistent notification state (reminders, delivery state) | `/config/store` | retries and reminders survive a restart; pruned after 30 days by default |
| Container logs | `docker logs apprisedocker` (stdout/stderr) | troubleshooting |

Nothing is written outside this directory. The image default
`APPRISE_ATTACH_SIZE=0` disables attachment storage, so files are not kept on
disk; the container-internal `/tmp` is discarded with the container.

**No telemetry.** The package contains no analytics, no crash reporting, no
update check and no account registration. Neither the package author nor
TerraMaster receives any data from the running application.

## 3. Data sent to third parties (API disclosure)

Apprise API sends data **only** to the destinations you configure. There is no
default destination and nothing is delivered until you add one. The table
below lists the most common endpoints, the data that is transmitted and where
it goes. "Region" is the region of the provider you chose — the package does
not pin a region and does not route through any intermediary.

| Destination (endpoint called) | Data sent | Destination / region |
|---|---|---|
| Telegram Bot API — `api.telegram.org` | message text, title, optional attachment, bot token | Telegram FZ-LLC servers; the data centre is chosen by Telegram (worldwide) |
| Discord — `discord.com/api/webhooks/…` | message text, title, optional attachment, webhook ID and token | Discord Inc., United States / global |
| Slack — `slack.com/api/chat.postMessage` or an incoming webhook | message text, title, channel, bot/OAuth token | Slack Technologies (Salesforce), United States / global |
| Microsoft Teams — `*.webhook.office.com` | message text, title, webhook token | Microsoft, region of your Microsoft 365 tenant |
| E-mail / SMTP — the host you enter | message body, subject, sender/recipient addresses, SMTP username and password | the mail server you specify (your own, your ISP or a provider) |
| Generic webhook / JSON / form — the URL you enter | message body, title and the fields you configure | the server you specify, in whatever country it runs |
| ntfy — `ntfy.sh` or your own server | message text, title, topic, optional access token | ntfy.sh (Germany) or your own server |
| Gotify — your server | message text, title, application token | your own server |
| Mailgun — `api.mailgun.net` | message body, subject, recipients, API key | Mailgun (Sinch), United States / EU depending on your account |
| Pushover, Matrix, Mattermost, Rocket.Chat, and 100+ others | message body/title plus the credential fields of that service | the service you configured |

The complete list of supported services and their URL formats is visible in
the running web interface under **Details** (`/details`), and upstream at
https://github.com/caronc/apprise-api.

**What leaves your NAS:** the notification text you send and the credentials
needed by the service you selected. **What does not:** your other
configurations, your NAS account, file listings, logs or telemetry.

## 4. Retention

- Configurations and credentials stay until you delete the configuration in
  the web interface (or call `POST /del/{KEY}`), or until the data directory is
  removed.
- The generated administrator password stays until you delete
  `admin-credentials.txt` and restart the application (a new password is then
  generated).
- Persistent notification state under `/config/store` is pruned automatically
  after 30 days (`APPRISE_STORAGE_PRUNE_DAYS`, up to 30 days by default).
- Container logs follow Docker's log rotation and disappear when the container
  is removed.
- Uninstalling the application from App Center keeps the data directory by
  default; choosing to delete the data on uninstall removes it.

No copy of your data is retained by the package author.

## 5. Your rights

Because the data is on your own device, you exercise these rights directly:

- **Access / portability** — read the files under
  `/Volume1/DockerAppData/apprisedocker/config`, or export one configuration
  with `POST /cfg/{KEY}`.
- **Rectification** — edit or replace the destination in the web interface.
- **Erasure** — delete a configuration with `POST /del/{KEY}`; delete
  `admin-credentials.txt` to rotate the password; delete the whole
  `DockerAppData/apprisedocker` directory (or uninstall and choose to delete
  the data) to erase everything.
- **Objection / restriction** — stop the application, or remove the
  destinations you no longer want to notify.
- **Complaint** — contact the publisher through
  https://github.com/Moechz/apprise/issues.

## 6. Security measures

- The container runs as the unprivileged user `1000:1000`; it has no
  `privileged` flag, no host network, no added capabilities and no access to
  the Docker socket.
- Only one TCP port is published (host `8108` → container `8000`).
- The web interface is protected by **HTTP Basic Auth**, enabled by default.
  The random administrator password is generated on first start (see
  `admin-credentials.txt` above); every unauthenticated request receives
  `401`.
- Configuration files are written with owner-only permissions inside a
  directory that is not reachable from the network.
- The interface speaks plain HTTP. On a LAN this is the same trust level as
  other local services; **for remote access put it behind your own TLS
  reverse proxy with authentication** (for example the Nginx Basic Auth setup
  documented upstream) and do not expose port 8108 directly to the internet.
- Deleting `admin-credentials.txt` and restarting rotates the password;
  deleting a configuration removes its stored credentials.

## 7. Contact

Publisher: **Moechz** — https://github.com/Moechz/apprise/issues

Upstream project: https://github.com/caronc/apprise-api
Container image: https://github.com/linuxserver/docker-apprise-api

## 8. Changes to this policy

Material changes are noted in `CHANGELOG.md` and in the release notes of the
package version that introduces them. The current version of this document is
always at
https://github.com/Moechz/apprise/blob/main/PRIVACY.md.
