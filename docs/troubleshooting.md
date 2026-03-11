# Troubleshooting

Known issues and fixes encountered during setup and operation.

---

## Abby crash-loops with `ERR_UNKNOWN_FILE_EXTENSION: .sh`

**Symptom:** Abby container restarts repeatedly. Logs show:
```
TypeError [ERR_UNKNOWN_FILE_EXTENSION]: Unknown file extension ".sh" for /app/start-abby.sh
```

**Cause:** The `docker-entrypoint.sh` in the image prepends `node` to any command that isn't executable. If the `.sh` file loses its executable bit in the image, Node's ESM loader tries to run it as JavaScript.

**Fix:** Use `/bin/sh` explicitly in `docker-compose.yml`:
```yaml
command: ["/bin/sh", "/app/start-abby.sh"]
```
Apply the same fix to `openclaw-beta` using `start-beta.sh`.

---

## Beta Telegram polling stalls / `sendMessage failed: Network request failed`

**Symptom:** Beta stops responding on Telegram. Logs show polling stall detected and repeated `sendMessage failed` errors.

**Cause:** Transient network issue causing Telegram connection to drop. Beta gets stuck sending typing indicators for a previous message.

**Fix:** Restart Beta:
```bash
docker compose restart openclaw-beta
```

---

## Beta: `Tool cron not found`

**Symptom:** Beta tries to schedule a task but gets `Tool cron not found`.

**Cause:** The `cron` tool is `ownerOnly` and must be explicitly allowed in config.

**Fix:** Add `cron` to `tools.alsoAllow` in `/home/arpkan/.openclaw-beta/openclaw.json`:
```json
"tools": {
  "alsoAllow": ["llm-task", "cron"]
}
```
Then restart Beta.

---

## Beta can't schedule tasks (`scheduler` skill missing)

**Symptom:** Beta reports it cannot schedule tasks or create cron jobs.

**Cause:** The `scheduler` skill was not enabled in Beta's config (unlike Abby).

**Fix:** Add to `skills.entries` in `/home/arpkan/.openclaw-beta/openclaw.json`:
```json
"skills": {
  "entries": {
    "scheduler": { "enabled": true }
  }
}
```

---

## `gaxios` / `node-fetch` ESM error with google-vertex

**Symptom:** Google Vertex calls fail with `Cannot convert undefined or null to object`.

**Cause:** `gaxios@7.x` tries `await import('node-fetch')` in Node 22. `node-fetch@3.3.2` is ESM-only and fails in Node 22's CJS-to-ESM translator.

**Fix:** `patches/fix-gaxios-fetch.cjs` patches `gaxios` to use `globalThis.fetch` instead. Applied in the Dockerfile via:
```dockerfile
RUN node patches/fix-gaxios-fetch.cjs
```
Must rebuild both Abby and Beta images after any `pnpm install`.

---

## Google Vertex API rate limits

**Symptom:** Beta logs show repeated `API rate limit reached` on Gemini models.

**Fix:** Switch Beta to a different model via Telegram:
```
/model MiniMax
/model Sonnet
```
Or wait — Vertex rate limits typically reset within minutes to an hour.

---

## Session tool list stale after config change

**Symptom:** After adding a new tool to `tools.alsoAllow`, the agent still reports the tool is not found.

**Cause:** The tool list is baked into the session's system prompt when the session starts. Existing sessions don't pick up config changes.

**Fix:** Reset the active session:
```bash
docker exec \
  -e OPENCLAW_GATEWAY_URL=ws://localhost:18889 \
  -e OPENCLAW_GATEWAY_TOKEN=<token> \
  openclaw-openclaw-beta-1 \
  node /app/dist/index.js agent --agent main --message "/reset"
```
Note: this clears conversation history for that session.
