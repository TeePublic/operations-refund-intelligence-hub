# Refund Intelligence Hub

Single-file HTML tool for cross-functional refund analysis (CS, Supply Chain, Logistics) across TeePublic (TPO) and Redbubble (RBO). The entire app is `index.html`; it deploys to GitHub Pages via GitHub Actions.

## Live URL

```
https://sloanewi.github.io/refund-intelligence-hub/
```

(Trailing slash matters — `index.html` is served at that path. The exact URL also appears on each successful workflow run under the `github-pages` environment, and on Settings → Pages.)

## Deploy setup

Deployment runs from `.github/workflows/deploy.yml` on every push to `main`. It injects two repo Secrets into `index.html` at build time so they never live in the repo source (the source only ever holds `__SETTINGS_HASH_PLACEHOLDER__` / `__GOOGLE_CLIENT_ID_PLACEHOLDER__`, and the build hard-fails if a placeholder survives).

To go live, in the GitHub repo:

1. **Add repo Secrets** (Settings → Secrets and variables → Actions):
   - `SETTINGS_PASSWORD` — the settings-panel password (the workflow stores only its SHA-256 hash).
   - `GOOGLE_CLIENT_ID` — the OAuth Client ID.

   The first workflow run fails at the inject step until both exist — that's expected. Re-run after adding them.

2. **Set the Pages source to GitHub Actions** (Settings → Pages → Build and deployment → Source = **GitHub Actions**).
   ⚠️ Do **not** use "Deploy from a branch" — that serves the raw source with placeholders un-injected (broken password + OAuth) and bypasses secret injection.

3. **Authorize the Pages origin for OAuth.** In the Google Cloud console (APIs & Services → Credentials → your OAuth client), add to **Authorized JavaScript origins**:
   ```
   https://sloanewi.github.io
   ```
   Without this, Google Sheets / Drive / Gmail auth is rejected on the live site.

4. **Provision the Google Sheet** ("RIH Master Data") with three tabs the app reads via **☁️ Load from Sheet** (row 1 = headers):
   - `refunds_master` — A:M: `refund_id, date, site, team, source, tag, amount, currency, agent, ticket_id, order_id, ingested_at, ingested_by`
   - `okr_targets` — A:F: `metric, target, unit, direction, label, updated_at`
   - `sources_registry` — A:F: `name, team, site, type, conf_level, last_updated`

## Security note

GitHub Pages sites are publicly reachable by default. No secrets or refund data live in the repo source (data loads client-side via OAuth), and the Settings panel is password-gated — but the app shell itself is open to anyone with the URL. If the whole app needs to be access-restricted, that requires a private Pages site (GitHub Pro/Team/Enterprise) or fronting it with auth.

## Local preview

`index.html` is fully static. Serve the repo root with any static server and open the page — settings and OAuth degrade gracefully when the build placeholders aren't injected.
