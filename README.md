# MCP Agent Mail — Shared Mailbox Snapshot

This repository hosts a static export of an MCP Agent Mail project. It contains a scrubbed SQLite snapshot plus a self-contained viewer so teammates can browse threads, attachments, and metadata without needing direct database access.

## What's Included

- `mailbox.sqlite3` — scrubbed mailbox database (agent names retained, ack/read state cleared).
- `viewer/` — static web viewer (Alpine.js + Tailwind) with inbox, thread explorer, search, and attachment tooling.
- `manifest.json` — machine-readable metadata (project scope, scrub stats, hosting hints, asset SRI hashes).
- `_headers` — COOP/COEP headers for hosts that support Netlify-style header rules.
- `.nojekyll` — disables GitHub Pages' Jekyll processing so `_headers` and `.wasm` assets are served untouched.
- `HOW_TO_DEPLOY.md` — detailed deployment checklist for GitHub Pages, Cloudflare Pages, Netlify, or generic hosts.

## Live Viewer

The `viewer/index.html` application renders the exported mailbox locally or when deployed to static hosting. If this repo is published via GitHub Pages, the root `index.html` immediately redirects to the viewer.

Detected hosting targets:
- **GitHub Pages** — Deploy the bundle via docs/ or gh-pages branch with coi-serviceworker.js for cross-origin isolation. _(signals: Git remote: https://github.com/Dicklesworthstone/mcp_agent_mail)_

## Quick Start

1. **Install dependencies** (first time only):
   ```bash
   uv sync
   ```
2. **Rebuild or update the export** from the source project:
   ```bash
   uv run python -m mcp_agent_mail.cli share update /path/to/this/repo
   ```
3. **Preview locally**:
   ```bash
   uv run python -m mcp_agent_mail.cli share preview .
   ```
   The command serves the viewer at `http://127.0.0.1:9000/` with hot reload.
4. **Deploy** using GitHub Pages (built into the `share wizard`) or manually follow `HOW_TO_DEPLOY.md`.

## Regenerating a Fresh Snapshot

From the MCP Agent Mail source checkout, run:

```bash
uv run python -m mcp_agent_mail.cli share export \
  --output /path/to/this/repo \
  --project <project-slug> \
  --scrub-preset standard \
  --no-zip
```

This overwrites the bundle (after you clean the repo) with the latest messages while preserving viewer assets.

## Verifying Integrity

- **Signed bundles**: If `manifest.sig.json` is present, verify with:
  ```bash
  uv run python -m mcp_agent_mail.cli share verify . --public-key $(cat signing-*.pub)
  ```
- **SRI hashes**: `manifest.json` records SHA256 digests for viewer assets; static hosts can pin hashes if desired.

## Troubleshooting

- **GitHub Pages shows 404**: confirm Pages is set to the `main` branch and root (`/`). The wizard calls `gh api repos/:owner/:repo/pages` automatically; re-run it if needed.
- **`.wasm` served as text/plain**: ensure `.nojekyll` is present and `_headers` are respected, or configure MIME types manually.
- **Viewer warns about OPFS**: host must send COOP/COEP headers. GitHub Pages requires the bundled `coi-serviceworker.js` to be uncommented (see `HOW_TO_DEPLOY.md`).

## About MCP Agent Mail

MCP Agent Mail is an asynchronous coordination layer for multi-agent coding workflows. It captures messages, attachments, and advisory file reservations so agents can collaborate safely. Static exports make it easy to share audit trails without granting direct database access.
