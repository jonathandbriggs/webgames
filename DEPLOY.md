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
to `main` auto-deploys.

1. Cloudflare dashboard → **Workers & Pages** → project `shy-sun-6603` →
   connected to Git → `jonathandbriggs/webgames`.
2. Build settings (dashboard):
   - Root directory: `/public`
   - Version command (used for PR/preview builds): `npx wrangler versions upload`
   - Deploy command (used for `main`): `npx wrangler deploy`
3. **`public/wrangler.jsonc`** declares the assets directory (`"."`, i.e.
   `public/` itself, since the dashboard root directory is already
   `/public`). This used to be a dashboard-only "Assets directory" setting
   with no config committed to the repo — that setting silently disappeared
   at least once (see "Known failure mode" below), so it's now pinned in
   the repo instead of depending on dashboard state.
4. On push, the build clones the repo, cds into `public/`, and runs
   `wrangler versions upload`/`wrangler deploy`, which reads `wrangler.jsonc`
   and uploads `public/` as static assets. No build step — there's nothing
   to compile.
5. **Custom domain**: project → Custom Domains → `games.jdbriggs.xyz`. DNS is
   configured automatically since the zone is already on Cloudflare.

After this, `git push` to `main` redeploys automatically. Pull requests get a
preview URL like `<version>-shy-sun-6603.<account>.workers.dev`.

### Known failure mode
Without `public/wrangler.jsonc` (or an equivalent dashboard "Assets
directory" setting), `wrangler versions upload`/`wrangler deploy` has no
entry-point or assets dir to find and fails immediately with:

```
✘ [ERROR] Missing entry-point to Worker script or to assets directory
```

This happened on the first build of PR #2 (2026-07-04), fixed at the time by
setting a dashboard-only "Assets directory" option — but that setting isn't
version-controlled and reverted/vanished, causing the same failure to
resurface on PR #4. `public/wrangler.jsonc` now makes this config part of
the repo instead of invisible dashboard state, so it should survive project
reconfigurations going forward.

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
