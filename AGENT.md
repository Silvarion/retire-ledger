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

## The Eulerpool API key

The page fetches live fund/dividend data client-side directly from
`api.eulerpool.com`, using an API key the visitor pastes into the page
each session. That key is never sent anywhere else and never persisted
(not even to localStorage) — this was already true before the Cloudflare
migration and must stay true: it's the only reason this page can be a
plain static site with no backend at all despite calling a keyed API.
