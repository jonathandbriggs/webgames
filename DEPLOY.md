# Publishing games.jdbriggs.xyz

## What you're deploying

Every game is a **single, self-contained HTML file** — inline CSS/JS, no build
step, no dependencies, no server-side code, no external assets. They live in
`public/`, which is the Cloudflare Pages output directory.

| File | Game |
|---|---|
| `public/amaze-go.html` | Amaze GO remake — tap the tangled lines to slide them out |
| `public/arrow-maze.html` | Arrow grid — tap tiles to fly them off a random shape |
| `public/hex-maze.html` | Hex grid maze |

Notes:
- Level progress uses `localStorage`, which is **per-domain per-browser**.
  Progress from `file://` copies won't carry over to the website.
- `public/_headers` sets Cloudflare Pages cache rules for the deployed files.

## Pipeline: Cloudflare Pages, connected to GitHub

One-time setup, then every push to `main` auto-deploys. No workflow files, no
API tokens in the repo.

1. Cloudflare dashboard → **Workers & Pages** → **Create** → **Pages** →
   **Connect to Git** → select `jonathandbriggs/webgames`.
2. Build settings:
   - Framework preset: **None**
   - Build command: *(leave empty)*
   - Build output directory: **`public`**
3. Deploy. Cloudflare builds nothing (there's no build step) and just serves
   `public/` as-is.
4. **Custom domain**: project → Custom Domains → add `games.jdbriggs.xyz`.
   DNS is configured automatically since the zone is already on Cloudflare.

After this, `git push` to `main` redeploys automatically. Pull requests get a
preview URL (`<hash>.<project>.pages.dev`) for free.

### Cloudflare settings worth checking
- **SSL/TLS mode**: Full (or Flexible — Pages traffic is HTTPS regardless).
- **Always Use HTTPS**: on.
- Caching: `public/_headers` controls this; after a deploy, Cloudflare respects
  those `Cache-Control` values at the edge.

### Updating a game
Edit the `.html` file, commit, push to `main`. That's the entire release
process.

## Alternative: manual deploy without Git integration

If you ever want to push a one-off build without waiting on Git:

```sh
npx wrangler pages deploy public --project-name=games
```

Requires `wrangler login` once. Not needed for normal use — the Git
integration above supersedes this for day-to-day updates.
