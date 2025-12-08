# HOW_TO_DEPLOY

## Quick Local Preview

1. Run `uv run python -m mcp_agent_mail.cli share preview ./` from this bundle directory.
2. Open the printed URL (default `http://127.0.0.1:9000/`).
3. Press Ctrl+C to stop the preview server when finished.

## Detected Hosting Targets

- **GitHub Pages**: Deploy the bundle via docs/ or gh-pages branch with coi-serviceworker.js for cross-origin isolation. _(signals: Git remote: https://github.com/Dicklesworthstone/mcp_agent_mail)_

## GitHub Pages (detected)

- Copy `viewer/`, `manifest.json`, and `mailbox.sqlite3` into your `docs/` folder or gh-pages branch.
- Add a `.nojekyll` file so `.wasm` assets are served with correct MIME types.
- **CRITICAL**: Edit `viewer/index.html` and uncomment the line `<script src="./coi-serviceworker.js"></script>` (around line 63).
- This service worker enables Cross-Origin-Isolation (COOP/COEP headers) required for OPFS caching and optimal sqlite-wasm performance.
- GitHub Pages does not support the `_headers` file, so the service worker intercepts requests and adds the required headers.
- Commit and push, then enable GitHub Pages for your repository branch in repository settings.
- On first visit, the page will reload automatically once the service worker activates (this is normal behavior).
- Verify isolation: open browser DevTools console and check that `window.crossOriginIsolated === true`.

## Cloudflare Pages (guide)

- Ensure `wrangler.toml` references the bundle directory (or upload the ZIP directly via the dashboard).
- The included `_headers` file will automatically apply `Cross-Origin-Opener-Policy: same-origin` and `Cross-Origin-Embedder-Policy: require-corp` headers.
- These headers are required for OPFS caching and optimal sqlite-wasm performance.
- Verify isolation: open browser DevTools console and check that `window.crossOriginIsolated === true`.
- For attachments >25 MiB, push them to R2 and reference the signed URLs in the manifest.

## Netlify (guide)

- The included `_headers` file will automatically apply `Cross-Origin-Opener-Policy: same-origin` and `Cross-Origin-Embedder-Policy: require-corp` headers.
- These headers are required for OPFS caching and optimal sqlite-wasm performance.
- Deploy the bundle directory (or ZIP) via CLI or the Netlify UI.
- Verify isolation: open browser DevTools console and check that `window.crossOriginIsolated === true`.
- Verify `.wasm` assets are served with `application/wasm` using Netlify's response headers tooling.

## Amazon S3 / Generic S3-Compatible (guide)

- Upload the bundle directory to your bucket (e.g., via `aws s3 sync`).
- Set `Content-Type` metadata: `.wasm` → `application/wasm`, SQLite files → `application/octet-stream`.
- When fronted by CloudFront, configure response headers for COOP/COEP and caching policies.

## Generic Static Hosts

- Serve the directory via any static host that honours `Content-Type` metadata (e.g., nginx, Vercel static, Firebase Hosting).
- Ensure `.wasm` files return `application/wasm` and SQLite databases return `application/octet-stream` or `application/vnd.sqlite3`.
- For optimal performance, enable Cross-Origin-Isolation (COOP/COEP headers). The included `_headers` file is automatically applied by Cloudflare Pages and Netlify.
- If your host doesn't support `_headers` (e.g., GitHub Pages), uncomment the `coi-serviceworker.js` script in `viewer/index.html` to enable isolation via service worker.
- If cross-origin isolation is unavailable, the viewer will show a warning banner with platform-specific instructions and fall back to streaming mode.
- Verify isolation: open browser DevTools console and check that `window.crossOriginIsolated === true`.

Review `manifest.json` before publication to confirm the included projects, hashing, and scrubbing policies.