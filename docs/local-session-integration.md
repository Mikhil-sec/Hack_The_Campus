# Local Session Integration

**Goal:** when a challenge site is already open and authenticated in our own
browser, let an agent inspect what the page is actually doing — console logs,
network requests/responses — without us copy-pasting everything by hand.

Only use this against the lab targets we are authorized to work on.

## OpenTabs (Chrome extension + MCP server)

OpenTabs connects an AI agent to web apps through a Chrome **extension** paired
with an **MCP server**. The key property: it reuses the *existing authenticated
browser session* rather than logging in separately, so the agent sees the app
as we see it — including requests made through that logged-in session.

What this gives an engineering agent, locally:

- **Console log visibility** — read what the page logs (errors, debug output,
  app state) instead of eyeballing DevTools.
- **Network inspection** — see the API calls the page makes and the responses
  it gets back, as MCP tool results the agent can reason over.
- **Authenticated API calls** — execute real calls through the browser's
  session, so the agent works with the same access the signed-in user has.

Source: https://github.com/opentabs-dev/opentabs — "connects AI agents to web
applications via a Chrome extension and MCP server, executing real API calls
through authenticated browser sessions."

## Why this is efficient

- No re-auth flow for the agent to script — it rides the session we already have.
- Console + network come back as structured tool output, not screenshots, so
  they're cheap to read and easy to diff across steps.
- Keeps a human in the loop: we open and authenticate the tab; the agent inspects.

## Setup (from upstream quick-start; requires Node.js 22+ and Chrome)

```bash
npm install -g @opentabs-dev/cli
opentabs start      # first run: creates ~/.opentabs/, generates an auth
                    # secret, installs extension files, prints MCP configs
```

1. **Load the extension:** open `chrome://extensions/`, enable *Developer
   mode*, *Load unpacked*, select `~/.opentabs/extension/`. It connects to the
   server automatically.
2. **Register with Claude Code** (use the secret printed by `opentabs start`):
   ```bash
   claude mcp add --transport http opentabs http://127.0.0.1:9515/mcp \
     --header "Authorization: Bearer YOUR_SECRET_HERE"
   ```
3. **Check it's healthy:** `opentabs status` (server, extension, MCP clients).
4. Open and sign into the challenge site in that Chrome, then have the agent
   use the read tools below before any action tools.

Keep the auth secret out of the repo — don't commit `~/.claude.json` or the
`claude mcp add` command with a real secret in it.

## Console and network tools

- `browser_enable_network_capture` — turn on capture for a tab (the debugger
  records network requests **and** console output).
- `browser_get_console_logs` — read console messages; filter by level
  (`log`, `warn`, `error`, `info`, `debug`, `all`).
- `browser_clear_console_logs` — clear the buffer without disabling capture.

## Auditing what the agent did

```bash
opentabs audit              # recent tool calls, success/failure, duration
opentabs audit --since 1h
opentabs logs --follow      # server logs in real time
```

The audit log persists to `~/.opentabs/audit.log` — handy for our post-event
write-up and for confirming the agent only touched in-scope targets.

## Guardrails

- Scope to the authorized lab target only.
- Treat captured requests/responses and any tokens as sensitive — keep in-lab.
- Prefer read (inspect) tools over action tools until you know what the page does.
