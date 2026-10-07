# DayanVPN v2 — VLESS (WS + XHTTP) on Railway with landing page & subscription

Single repo, deployable to Railway (and any Docker host). nginx fronts Xray:

```
Browser  -->  GET /       -->  nginx  -->  DayanVPN landing page
Client   -->  WS  /ray    -->  nginx  -->  Xray VLESS+WS   (port 10086)
Client   -->  POST /xray  -->  nginx  -->  Xray VLESS+XHTTP (port 10087)
Client   -->  GET /sub    -->  nginx  -->  subscription file (base64)
```

## Files

| File | Purpose |
|------|---------|
| `config.json` | Xray: two VLESS inbounds (WS + XHTTP), ad/tracker blocklist |
| `nginx.conf` | Routes `/`, `/ray`, `/xray`, `/sub` |
| `entrypoint.sh` | Injects UUID into page, builds `/sub` from `SERVERS` env, aligns PORT |
| `index.html` | Landing page (UUID + subscription auto-shown) |
| `Dockerfile` | nginx + latest Xray-core on Alpine |

## Deploy (Railway)

1. Push all files to a **private** GitHub repo (root of repo, not a subfolder).
2. Railway → New Project → Deploy from GitHub repo → select the repo.
3. Networking → Generate Domain → open it → DayanVPN page should appear.
4. Add env var **on every service**:
   ```
   SERVERS = us-domain.up.railway.app:US,eu-domain.up.railway.app:EU
   ```
   (format `domain:TAG`, comma-separated; redeploy triggers automatically)
5. Subscription URL: `https://any-domain/sub` — import in v2rayNG (Settings → Online subscriptions).

Each server produces **two configs** in the subscription:
- `DayanVPN-US` — classic WebSocket (works everywhere)
- `DayanVPN-US-X` — newer XHTTP (faster/stealthier, needs a recent client; if it doesn't connect, just ignore it)

## Features

- **Ad/tracker blocking** server-side (`config.json` routing rules → blackhole)
- **Multi-server subscription** via `SERVERS` env var
- Landing page with copy/QR/subscription buttons
- UUID lives only in `config.json`; page + links are generated from it at startup

## Changing the UUID

Edit `config.json` → `"id"` value → commit → all Railway services rebuild automatically.
Generate with `[guid]::NewGuid()` (PowerShell) or `uuidgen`.

## Troubleshooting

- **Subscription still shows old links after a change:** the build log's `COPY` lines say `cached` → the new file never reached GitHub. Verify file content on github.com (open it, Ctrl+F a unique string from the new version).
- **XHTTP links don't connect:** client too old — keep using the plain links.
- **Railway PORT:** entrypoint rewrites nginx `listen` from the `PORT` env automatically.
