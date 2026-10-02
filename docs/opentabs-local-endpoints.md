# OpenTabs — Local Connection Endpoints

Everything below is loopback-only (`127.0.0.1`). Verified 2026-10-02.

## Local files

| Path | Purpose |
|------|---------|
| `C:\Users\naika\.opentabs\config.json` | OpenTabs config (valid JSON; `telemetry: false`, `version: 3`) |
| `C:\Users\naika\.opentabs\config.json.backup` | Backup written by the CLI |
| `C:\Users\naika\.opentabs\server.log` | MCP server log |
| `C:\Users\naika\.opentabs\extension\` | Browser extension files (load unpacked) |

## MCP server

- Process: `node.exe` running `@opentabs-dev\cli\...\mcp-server\dist\index.js`
- Listening on: `127.0.0.1:9515` (loopback only, not exposed on the LAN)

| Endpoint | Auth | Observed response |
|----------|------|-------------------|
| `http://127.0.0.1:9515/health` | none | `200 {"status":"ok"}` |
| `http://127.0.0.1:9515/mcp` | Bearer secret | `401` without a token (expected) |
| `http://127.0.0.1:9515/` | n/a | `404` |

## Connecting a client

```
claude mcp add --transport http opentabs http://127.0.0.1:9515/mcp \
  --header "Authorization: Bearer <secret>"
```

The secret is not stored in this repo. Don't commit it. See
[local-session-integration.md](local-session-integration.md) for where to get it.

## Quick checks (PowerShell)

```powershell
# Is the server listening, and on which interface?
Get-NetTCPConnection -LocalPort 9515 -State Listen

# Health probe
Invoke-WebRequest http://127.0.0.1:9515/health -UseBasicParsing

# Config parses as JSON?
Get-Content C:\Users\naika\.opentabs\config.json -Raw | ConvertFrom-Json
```
