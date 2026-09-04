# brainwave-uptime

External uptime monitoring for **brainwavegenerator.us** and
**brainwavegenenator.com**, running on GitHub's infrastructure.

## Why a second monitor

The primary check is a systemd timer on the server itself
(`deploy/uptime-check.sh` in the site repo), which runs every minute and
catches nginx failures, wrong content, an accidentally re-enabled staging lock,
certificate expiry and a full disk.

It cannot report that the server is dead. This workflow can: it runs every
ten minutes on GitHub's machines and fails the job — which emails the repository
owner — when either site stops answering correctly.

GitHub's scheduler runs late under load, so treat this as a ~15 minute safety
net rather than the primary monitor.

## What it checks

| Check | Failure means |
|---|---|
| `/` returns 200 and contains the site name | The site is down or serving something else |
| `/sitemap.xml` returns 200 | The generated files are missing |
| `/noise/brown/` returns 200 with its heading | Deep pages are broken while the homepage is fine |
| `brainwavegenenator.com` returns 200 | The older site is down (it once was, for 139 days) |
| Certificate has more than 10 days left | Renewal has stopped working |
| HTTP 401 on any check | The staging lock was left on — invisible to Google |

## Alerts

- **Email:** GitHub notifies the repository owner when a scheduled workflow fails.
- **Push:** if the `NTFY_TOPIC` secret is set, an alert goes to that
  [ntfy.sh](https://ntfy.sh) topic — the same one the server-side monitor uses.

## Note on the heartbeat

GitHub disables scheduled workflows in repositories with no activity for 60
days. The workflow commits a timestamp once a day so the monitor does not
quietly switch itself off.
