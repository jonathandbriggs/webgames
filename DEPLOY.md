# Publishing games.jdbriggs.xyz

## What you're deploying

Every game is a **single, self-contained HTML file** — inline CSS/JS, no build
step, no dependencies, no server-side code, no external assets. They live in
`public/`, which is the Cloudflare Pages output directory.

| File | Game |
|---|---|
| `public/untangle.html` | Untangle — tap the tangled lines to slide them out |
| `public/arrow-maze.html` | Arrow grid — tap tiles to fly them off a random shape |
| `public/hex-maze.html` | Hex grid maze |

Notes:
- Level progress uses `localStorage`, which is **per-domain per-browser**.
  Progress from `file://` copies won't carry over to the website.
- `public/_headers` sets cache rules for the deployed files.

## Pipeline: Cloudflare Workers Builds, connected to GitHub

This runs on Cloudflare's Workers Builds pipeline (the unified successor to
classic Pages), project name `shy-sun-6603`. One-time setup, then every push
to `main` auto-deploys. No workflow files, no API tokens, no `wrangler.toml`
in this repo — the deploy config lives entirely in the Cloudflare dashboard.

1. Cloudflare dashboard → **Workers & Pages** → project `shy-sun-6603` →
   connected to Git → `jonathandbriggs/webgames`.
2. Build settings (dashboard, not repo):
   - Deploy command: `npx wrangler versions upload`
   - **Assets directory: `public`** — this is the important one; there's no
     `wrangler.jsonc`/`wrangler.toml` committed, so Cloudflare only knows to
     serve `public/` because this is set in the project's dashboard config.
3. On push, the build clones the repo, runs `wrangler versions upload`, which
   reads `public/` as static assets and uploads/deploys them. No build step —
   there's nothing to compile.
4. **Custom domain**: project → Custom Domains → `games.jdbriggs.xyz`. DNS is
   configured automatically since the zone is already on Cloudflare.

After this, `git push` to `main` redeploys automatically. Pull requests get a
preview URL like `<version>-shy-sun-6603.<account>.workers.dev`.

### Known failure mode
If the **assets directory** setting in the dashboard is ever unset (e.g. on a
new/recreated project), `wrangler versions upload` has no entry-point or
assets dir to find and fails immediately with:

```
✘ [ERROR] Missing entry-point to Worker script or to assets directory
```

This happened on the first build of PR #2 (2026-07-04) before the assets
directory was configured; the very next build succeeded once it was set.
Since nothing in this repo controls that setting, if deploys start failing
with this error again, check the dashboard build config first, not the code.

### Cloudflare settings worth checking
- **SSL/TLS mode**: Full (or Flexible — traffic is HTTPS regardless).
- **Always Use HTTPS**: on.
- Caching: `public/_headers` controls this; after a deploy, Cloudflare respects
  those `Cache-Control` values at the edge.

### Updating a game
Edit the `.html` file, commit, push to `main`. That's the entire release
process.

## Alternative: manual deploy without Git integration

If you ever want to push a one-off build without waiting on Git:

```sh
npx wrangler deploy --name=shy-sun-6603 --assets=./public
```

Requires `wrangler login` once. Not needed for normal use — the Git
integration above supersedes this for day-to-day updates.
