# Quant-UX — Local + Share (Cloudflare Tunnels) Quick Start

Spin up the full Quant-UX stack (frontend + Java backend + WebSocket + MongoDB) on your machine with Docker, then share it publicly using **Cloudflare Quick Tunnels** — no cloud account, no DNS.

> **You'll get**
> - Local app at `http://localhost:8082`
> - Two public URLs (one for the app, one for the WebSocket)
> - Containers only; easy to start/stop

---

## 0) Prerequisites

### Docker + Compose
- **macOS**: Install Docker Desktop.
- **Ubuntu/Linux**:
  ```bash
  sudo apt-get update
  sudo apt-get install -y docker.io docker-compose-plugin
  sudo usermod -aG docker $USER && newgrp docker
  ```

### cloudflared (the Cloudflare Tunnel client)

**macOS (Homebrew):**
```bash
brew update && brew install cloudflared
```

**Ubuntu (if repo package not available, grab the .deb from Cloudflare's site):**
```bash
sudo apt-get update
sudo apt-get install -y cloudflared || true
```

## 1) Clone the repo

```bash
git clone https://github.com/KlausSchaefers/quant-ux
cd quant-ux/docker
```

## 2) Start the stack (Docker)

```bash
docker compose up -d
docker compose ps
```

### What should be running
- Frontend on 8082
- WebSocket on 8086
- Backend on 8080
- MongoDB in the compose network

### Sanity check
```bash
curl -I http://localhost:8082
# Expect HTTP/1.1 200/301-ish
```

Open `http://localhost:8082` in your browser (local only for now).

## 3) Create two Quick Tunnels (two terminals)

Quick Tunnels print a random `https://<something>.trycloudflare.com` URL and stay alive as long as the command runs.

### Terminal A — Frontend
```bash
cloudflared tunnel --hello-world   # quick sanity check; should print a URL
cloudflared tunnel --url http://localhost:8082
# Copy the printed https://<FE>.trycloudflare.com
```

### Terminal B — WebSocket
```bash
cloudflared tunnel --url http://localhost:8086
# Copy the printed https://<WS>.trycloudflare.com
```

Keep both terminals running.

> **Note:** If a command looks "stuck" or doesn't show a URL, hit Ctrl+C and use the Troubleshooting section below.

## 4) Point the frontend at the public WS URL

By default the UI tries `ws://127.0.0.1:8086` which only works on your machine. Change it to the public `wss://` URL from Terminal B.

1. Open `quant-ux/docker/docker-compose.yml`
2. Find the `qux-fe` service → `environment:` and set:

```yaml
QUX_WS_URL: "wss://<WS>.trycloudflare.com"  # paste your WS tunnel URL
TZ: "America/Los_Angeles"                   # optional but recommended
QUX_JWT_PASSWORD: "<long-random-string>"    # optional but recommended
```

Leave `QUX_PROXY_URL` as-is (container-internal).

3. Restart to pick up env changes:
```bash
docker compose down && docker compose up -d
```

## 5) Share it

Send friends the frontend URL from Terminal A:
```
https://<FE>.trycloudflare.com
```

The app will open a WebSocket to:
```
wss://<WS>.trycloudflare.com    # what you set in QUX_WS_URL
```

## 6) Verify it's live

1. Open the frontend public URL.
2. Browser DevTools → Network → WS:
   - You should see `101 Switching Protocols` to `wss://<WS>…`
   - If it still tries `ws://127.0.0.1:8086`, you didn't update `QUX_WS_URL`.
   - If WS shows 400/403/closed, make sure the WS tunnel (Terminal B) is running.

## 7) Keep tunnels alive (background options)

Quick Tunnels die when you stop the command.

### nohup
```bash
nohup cloudflared tunnel --url http://localhost:8082 > /tmp/qux-fe.log 2>&1 &
nohup cloudflared tunnel --url http://localhost:8086 > /tmp/qux-ws.log 2>&1 &
tail -f /tmp/qux-fe.log
tail -f /tmp/qux-ws.log
```

### tmux
```bash
tmux new -s qux-fe 'cloudflared tunnel --url http://localhost:8082'
# detach: Ctrl+B, then D
tmux new -s qux-ws 'cloudflared tunnel --url http://localhost:8086'
```

### Stop them later
```bash
pkill -f 'cloudflared tunnel --url http://localhost:8082'
pkill -f 'cloudflared tunnel --url http://localhost:8086'
```

## 8) Troubleshooting

### A) cloudflared prints cert errors or hangs with no URL

1. Kill it (Ctrl+C).
2. Ensure no default config is confusing it:
   ```bash
   ls -la ~/.cloudflared /usr/local/etc/cloudflared 2>/dev/null
   ```
3. If present, temporarily move them:
   ```bash
   mv ~/.cloudflared ~/.cloudflared.bak 2>/dev/null || true
   sudo mv /usr/local/etc/cloudflared /usr/local/etc/cloudflared.bak 2>/dev/null || true
   ```
4. Force a clean run (ignore any config):
   ```bash
   : > /tmp/empty.yml
   cloudflared tunnel --config /tmp/empty.yml --url http://localhost:8082
   ```

### B) Using Cloudflare WARP/VPN?

Turn it off and retry Quick Tunnels (they can conflict on some setups).

### C) Frontend loads but features fail

Check containers:
```bash
docker compose ps
docker compose logs -f --tail=200
```

Hard restart:
```bash
docker compose down -v
docker compose up -d
```

### D) "Address already in use" on 8082/8086

```bash
lsof -iTCP:8082 -sTCP:LISTEN
lsof -iTCP:8086 -sTCP:LISTEN
kill <PID>
```

### E) Emails (invites/reset) don't send

Set SMTP env vars in the backend service if you need email. Without them, everything else works; email won't.

## 9) Optional: one persistent (named) tunnel with two hostnames

Requires a Cloudflare account and your domain on Cloudflare. Cleaner hostnames; not necessary for a quick share.

```bash
# 1) Auth (opens browser)
cloudflared tunnel login

# 2) Create tunnel
cloudflared tunnel create qux
```

Create `~/.cloudflared/config.yml`:
```yaml
tunnel: qux
credentials-file: /Users/<you>/.cloudflared/<UUID>.json

ingress:
  - hostname: qux.example.com
    service: http://localhost:8082
  - hostname: qux-ws.example.com
    service: http://localhost:8086
  - service: http_status:404
```

Add DNS CNAMEs for `qux` and `qux-ws` to the tunnel in Cloudflare, then run:
```bash
cloudflared tunnel run qux
```

Update compose:
```yaml
QUX_WS_URL: "wss://qux-ws.example.com"
```

Restart Docker.