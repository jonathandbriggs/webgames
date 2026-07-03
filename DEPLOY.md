# Publishing games.jdbriggs.xyz

## What you're deploying

Every game in this folder is a **single, self-contained HTML file** — inline CSS/JS, no build step, no dependencies, no server-side code, no external assets:

| File | Game |
|---|---|
| `amaze-go.html` | Amaze GO remake — tap the tangled lines to slide them out |
| `arrow-maze.html` | Arrow grid — tap tiles to fly them off a random shape |
| `uuid-qr-generator.html` | UUID/QR utility |

Anything that can serve static files can host them. That's the entire requirement.

Notes:
- Level progress uses `localStorage`, which is **per-domain per-browser**. Progress from iCloud/file:// copies won't carry over to the website (and vice versa). No data leaves the browser.
- Rename `amaze-go.html` to `index.html` if you want it at the root URL, or add a small landing page linking to each game.

## Recommended: homelab + Cloudflare Tunnel

No ports forwarded, no public IP exposed, free TLS. Two pieces: a static file server, and `cloudflared` pointing at it.

### 1. Static file server

Copy the `.html` files to a folder on the server (e.g. `/opt/games/site/`). Then any of:

**Docker + nginx** (`docker-compose.yml`):
```yaml
services:
  games:
    image: nginx:alpine
    volumes:
      - /opt/games/site:/usr/share/nginx/html:ro
    ports:
      - "127.0.0.1:8080:80"   # localhost only; the tunnel reaches it internally
    restart: unless-stopped
```

**Or Caddy** (one binary, no config file needed):
```sh
caddy file-server --root /opt/games/site --listen 127.0.0.1:8080
```

**Or nginx already on the box**: a plain `root /opt/games/site;` server block on port 8080.

### 2. Cloudflare Tunnel

Prereq: `jdbriggs.xyz` is on Cloudflare (nameservers pointed there — free plan is fine).

**Dashboard method (easiest):**
1. Cloudflare dashboard → **Zero Trust** → **Networks** → **Tunnels** → *Create a tunnel* (Cloudflared connector).
2. Name it `games`, then run the connector it gives you. Docker version:
   ```yaml
   services:
     cloudflared:
       image: cloudflare/cloudflared:latest
       command: tunnel run --token <TOKEN_FROM_DASHBOARD>
       restart: unless-stopped
       network_mode: host   # so it can reach 127.0.0.1:8080
   ```
3. In the tunnel's **Public Hostname** tab: hostname `games.jdbriggs.xyz`, service `http://127.0.0.1:8080`. (If nginx and cloudflared share a compose file, drop `network_mode: host` and use `http://games:80` instead.)
4. Cloudflare creates the DNS record automatically. Done — HTTPS included.

**CLI method (equivalent):**
```sh
cloudflared tunnel login
cloudflared tunnel create games
cloudflared tunnel route dns games games.jdbriggs.xyz
# ~/.cloudflared/config.yml:
#   tunnel: games
#   credentials-file: /root/.cloudflared/<tunnel-id>.json
#   ingress:
#     - hostname: games.jdbriggs.xyz
#       service: http://127.0.0.1:8080
#     - service: http_status:404
cloudflared service install   # run as a systemd service
```

### 3. Cloudflare settings worth checking

- **SSL/TLS mode**: "Full" (or "Flexible" — tunnel traffic is encrypted regardless).
- **Always Use HTTPS**: on.
- Caching: defaults are fine. After updating a game, purge cache (or just Ctrl-Shift-R) since Cloudflare may cache the HTML at the edge.

### Updating a game

Copy the new `.html` file over the old one in `/opt/games/site/`. That's it — no restart needed, maybe a cache purge.

## Alternative: skip the homelab entirely

If uptime-on-your-hardware isn't the point, **Cloudflare Pages** hosts static files free with zero maintenance:

1. Dashboard → **Workers & Pages** → *Create* → **Pages** → *Upload assets*: drag the `.html` files in.
2. Add `games.jdbriggs.xyz` as a custom domain on the project (automatic since the zone is already on Cloudflare).

Or from the command line: `npx wrangler pages deploy /opt/games/site --project-name=games`.

Same result, no server to keep alive. The homelab tunnel is the better fit if you want everything self-hosted or plan to add server-backed projects later.
