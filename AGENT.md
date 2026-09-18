# AGENT.md — retire-ledger.silvarion.org

Guidelines for working on this site. Every decision must satisfy four principles:

1. **Keep it simple** — prefer the obvious solution; avoid abstractions that aren't needed yet.
2. **Keep it clear** — names, structure, and config must be readable without prior context.
3. **Keep it working** — never break the deployed site; sanity-check `html/index.html` in a browser before declaring done.
4. **Keep it secure** — no secrets in files, CSP-friendly embeds only, no credentials in client-side JS.

---

## Project overview

- **Site:** A dividend/retirement-income ledger tool, retire-ledger.silvarion.org
- **Stack:** A single hand-authored `html/index.html` (inline CSS/JS) — no
  build step, no framework, no static site generator.
- **Deployment:** Cloudflare Workers (static assets), via `wrangler.toml`.
  Formerly deployed via a docker-compose stack with a small Python
  companion server (`ledger-server.py`) persisting the saved fund-table
  config server-side — retired in favor of Cloudflare Workers, since a
  static deploy can't run a persistent backend. See README.md.
- **Local preview:** any static file server pointed at `html/`, e.g.
  `npx serve html` or `python3 -m http.server -d html`.

## API keys (Eulerpool, Twelve Data)

The page fetches live fund/dividend data client-side directly from
`api.eulerpool.com` and `api.twelvedata.com`, using API keys the visitor
pastes into the page. Neither key is ever sent anywhere besides that
provider's own API — no third-party server, no this-app's-own backend
(there isn't one). By explicit user request (2026-09-18), clicking "Save
as new defaults" now also persists both keys to this browser's own
localStorage (a separate key, `retireLedgerApiKeys`, from the shareable
config under `retireLedgerConfig`) so a returning visitor on the SAME
browser doesn't have to re-paste them — deliberately kept out of
`collectConfig()`'s output specifically so they can never end up in a
Download/Upload file or the printable report, both of which are meant to
be shareable. Anyone with access to that browser profile (or its
localStorage) can read the keys back out — a real tradeoff, accepted
for convenience over the previous "keeps in this tab's memory only, ask
again every session" behavior.
