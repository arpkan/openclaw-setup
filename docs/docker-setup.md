# Docker Setup

## Images

### `openclaw-super-abby:v1`
Built from `/home/arpkan/openclaw/Dockerfile`.

Includes: Chromium, Xvfb, x11vnc, noVNC, LibreOffice, LinkedIn skill dependencies.

Contains a patch for a Node 22 / gaxios bug — see [Troubleshooting](troubleshooting.md#gaxios--node-fetch-esm-error-with-google-vertex).

```bash
# Rebuild
cd /home/arpkan/openclaw
docker build -t openclaw-super-abby:v1 .
```

### `openclaw-beta:v1`
Built from `/home/arpkan/openclaw/Dockerfile.beta`. Extends `openclaw-super-abby:v1`.

**Important:** must be rebuilt after rebuilding Abby, since it inherits Abby's `node_modules` including the gaxios patch.

```bash
# Rebuild (always rebuild Abby first)
cd /home/arpkan/openclaw
docker build -f Dockerfile.beta -t openclaw-beta:v1 .
```

## Key files

| File | Purpose |
|---|---|
| `/home/arpkan/openclaw/docker-compose.yml` | Defines all containers |
| `/home/arpkan/openclaw/Dockerfile` | Abby image build |
| `/home/arpkan/openclaw/Dockerfile.beta` | Beta image build |
| `/home/arpkan/openclaw/start-abby.sh` | Abby startup script (Xvfb, VNC, noVNC, gateway) |
| `/home/arpkan/openclaw/start-beta.sh` | Beta startup script |
| `/home/arpkan/openclaw/.env` | Secrets injected at runtime |
| `/home/arpkan/openclaw/gcp-credentials.json` | Google Cloud service account |

## docker-compose.yml notes

Both containers use `restart: always` so they come back up automatically after crashes or reboots.

The `command` uses explicit `/bin/sh` to avoid a startup bug where the entrypoint script incorrectly runs `.sh` files through Node.js:
```yaml
command: ["/bin/sh", "/app/start-abby.sh"]
```

Beta has additional env vars so its internal CLI can connect back to its own gateway:
```yaml
OPENCLAW_GATEWAY_URL: ws://localhost:18889
OPENCLAW_GATEWAY_TOKEN: <token>
```

## Common commands

```bash
# Start / stop / restart everything
sudo systemctl start openclaw
sudo systemctl stop openclaw
sudo systemctl restart openclaw

# Restart a single container
docker compose restart openclaw-beta

# View logs
docker logs openclaw-openclaw-gateway-1
docker logs openclaw-openclaw-beta-1
docker logs --tail 50 --follow openclaw-openclaw-beta-1

# Check container status
docker ps

# Run a CLI command inside Beta
docker exec \
  -e OPENCLAW_GATEWAY_URL=ws://localhost:18889 \
  -e OPENCLAW_GATEWAY_TOKEN=<token> \
  openclaw-openclaw-beta-1 \
  node /app/dist/index.js cron list
```

## Updating OpenClaw

```bash
cd /home/arpkan/openclaw

# Stash local changes first (Dockerfile and docker-compose.yml may conflict)
git stash

# Pull latest
git pull --rebase

# Restore local changes
git stash pop

# Rebuild images (Abby first, then Beta)
docker build -t openclaw-super-abby:v1 .
docker build -f Dockerfile.beta -t openclaw-beta:v1 .

# Restart containers to use new images
sudo systemctl restart openclaw
```
