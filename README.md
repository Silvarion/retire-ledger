# retire-ledger

A dividend/retirement-income ledger tool (retire-ledger.silvarion.org). A
single hand-authored `html/index.html` — no build step, no framework —
deployed to Cloudflare Workers as static assets.

See [`AGENT.md`](./AGENT.md) for the guiding principles this project
follows.

## History

This was previously deployed via a docker-compose stack (a small Python
companion server, `ledger-server.py`, persisting the saved fund-table
config server-side to a bind-mounted `config.json`) under
`infrastructure/homelab/containers/docker-compose/retire-ledger`. That
stack is **retired** in favor of this repo deployed straight to Cloudflare
Workers — a static deploy has no persistent container to run that
companion server in. Concretely, that meant:

- **Config persistence moved from server-side to this browser's
  `localStorage`** (see `saveConfigToLocalStorage()`/
  `loadConfigFromLocalStorage()` in `html/index.html`). The saved fund
  table no longer syncs across devices/browsers — each browser keeps its
  own copy. Use the existing Download/Upload buttons to carry a config
  from one browser/device to another.
- The Eulerpool API key was **already** kept in-memory only (never sent
  to this app's own server, never persisted) — that behavior is unchanged
  by this migration; it's what let this become a pure static site with no
  backend at all despite calling a keyed third-party API client-side.

The old docker-compose directory has **not** been deleted — it's left in
place for reference/rollback. The cutover plan (all manual, outside this
repo/this session — no access to any of these hosts):

1. Deploy this repo to Cloudflare (see below).
2. Re-point `retire-ledger.silvarion.org`'s DNS/routing at the new
   Cloudflare deployment (see "Custom domain / DNS cutover" below) and
   confirm it actually serves the new site.
3. Only then decommission the docker-compose stack: remove the
   `retire-ledger` entry from `dockge_stacks` in
   `infrastructure/homelab/proxmox/srv-proxmox01/base/lxc/dockge/core`'s
   group_vars, redeploy that host, then remove the old stack directory —
   and update the Uptime Kuma monitor/Prometheus blackbox target for
   `retire-ledger.silvarion.org` to the `{inf_tech: cloudflare-workers,
   inf_category: external}` labels resume-silvarion's own entry already
   uses, since the target is no longer a `dockge`-fronted service.

## Requirements

None beyond a browser. No build step — `html/index.html` is served as-is.

## Local preview

```sh
npx serve html
# or: python3 -m http.server 8000 -d html
```

## Deploying to Cloudflare (Workers Builds)

1. Push this repo to a Git provider Cloudflare can read from.
2. In the Cloudflare dashboard: **Workers & Pages → Create → Connect to
   Git**, pick this repo.
3. Build settings (**Settings → Build** on the created Worker):
   - **Build command:** *(leave empty — no build step)*
   - **Deploy command:** `npx wrangler deploy` (the default)
   - **Version command:** `npx wrangler versions upload` (the default —
     used for preview builds on non-production branches)
4. `wrangler.toml` defines an assets-only Worker (`[assets] directory =
   "html"`, no Worker script) — Cloudflare serves `html/index.html`
   directly.

### Direct upload with Wrangler (no Git integration)

```sh
npx wrangler deploy
```

### Custom domain / DNS cutover

`retire-ledger.silvarion.org` currently reaches the docker-compose stack
via **opnsense-caddy** (the internet-facing OPNsense firewall's own Caddy
instance, `10.48.10.250`) forwarding straight to the container — this
routing is *not* managed by the `silvarion.webservers.caddy` role or any
repo in this workspace (confirmed: it has no `retire-ledger` entry in
`infrastructure/homelab/proxmox/srv-proxmox01/base/lxc/caddy/core`'s
`caddy_sites`), so it can't be changed from here.

To cut over:

1. In the Cloudflare dashboard, on this Worker: **Settings → Domains &
   Routes → Add → Custom Domain** → `retire-ledger.silvarion.org`. Since
   the `silvarion.org` zone is already on Cloudflare, this
   creates/updates the DNS record to point at the Worker automatically.
2. That DNS change alone doesn't remove the *existing* opnsense-caddy
   forwarding rule for this hostname — whichever one Cloudflare/DNS
   resolution actually routes to first wins. Remove or repoint that rule
   on the OPNsense box once the Custom Domain is confirmed live, so
   traffic isn't split or flapping between the old and new deployments.
3. Verify with `curl -sI https://retire-ledger.silvarion.org` — a
   `server: cloudflare` response with no `via: ... Caddy` header means
   it's hitting the Worker directly, not the old chain.

## Project structure

```
html/index.html   The entire app — markup, inline CSS, inline JS
html/_headers     Security headers / CSP for Cloudflare
wrangler.toml     Cloudflare Workers Builds config (assets-only, no Worker script)
```
