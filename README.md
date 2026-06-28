# Refund Intelligence Hub

Single-file HTML tool for cross-functional refund analysis (CS, Supply Chain, Logistics) across TeePublic (TPO) and Redbubble (RBO). The entire app is `index.html` — a plain static file, hosted on **Cloudflare Pages** behind **Cloudflare Access** for private, members-only viewing.

## Live URL

```
https://refundintelhub.pages.dev
```

(Hosted on Cloudflare Pages. Access is gated by Cloudflare Access to an allow-list of emails — viewers do **not** need GitHub accounts.)

## Config

Non-secret config is baked into `SITE_CONFIG` at the top of the `index.html` script (the site is private and the repo is private, so this is safe):

- `googleClientId` — OAuth Client ID (public by design).
- `sheetId` — the "RIH Master Data" Sheet (access is permissioned by Google).
- `settingsPasswordHash` — SHA-256 of the in-app Settings password. Generate with:
  ```
  printf '%s' 'YOUR_PASSWORD' | shasum -a 256 | cut -d' ' -f1
  ```
  Paste the hash into `SITE_CONFIG.settingsPasswordHash`. (This only gates the in-app Settings panel; Cloudflare Access is the real perimeter.)

## Deploy (Cloudflare Pages — Direct Upload)

No build step, no GitHub connection required.

1. Cloudflare dashboard → **Workers & Pages → Create → Pages → Upload assets**. Name the project (this sets `https://<project>.pages.dev`) and drag in `index.html`.
2. **Add Cloudflare Access** over the project: Zero Trust → Access → Applications → Add a self-hosted app for the `pages.dev` hostname, with an **Allow policy** listing the permitted emails (one-time PIN login).
3. **Authorize the origin for OAuth.** Google Cloud console → APIs & Services → Credentials → the OAuth client → **Authorized JavaScript origins**, add (origin only, no path/trailing slash):
   ```
   https://<project>.pages.dev
   ```
   Without this, Google Sheets / Drive auth — and therefore data persistence — is rejected on the live site.

To update the site later, re-upload `index.html` to the same project (or use `wrangler pages deploy`).

## Google Sheet (the persistence layer)

"RIH Master Data" must have three tabs (row 1 = headers):

- `refunds_master` — A:M: `refund_id, date, site, team, source, tag, amount, currency, agent, ticket_id, order_id, ingested_at, ingested_by`
- `okr_targets` — A:F: `metric, target, unit, direction, label, updated_at`
- `sources_registry` — A:F: `name, team, site, type, conf_level, last_updated`

## Shared data & retention

The Google Sheet is the single shared source of truth — uploaded data lives there, not in the website. Uploading a file (CSV / Excel / Drive) auto-publishes its rows to `refunds_master`, and the app auto-loads from the Sheet on open, so both users see the same data without re-uploading. Publishing uses **replace-by-source**: re-uploading a given source+site refreshes those rows instead of duplicating them. (Reads/writes require each user's own Google sign-in; concurrent edits are last-write-wins.) Sample/demo data is never published. Note: Deflection Tests and per-user settings are browser-local (sessionStorage), not Sheet-backed.

## Local preview

`index.html` is fully static. Serve the folder with any static server (e.g. `node .claude/serve.js`) and open the page.
