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

## Setup shape (verify against upstream before the event)

1. Install the OpenTabs Chrome extension.
2. Run the OpenTabs MCP server and register it with the agent/client.
3. Open and sign into the challenge site in the extension's browser.
4. Point the agent at the tab; use its read tools for console/network first
   before any write/action tools.

## Guardrails

- Scope to the authorized lab target only.
- Treat captured requests/responses and any tokens as sensitive — keep in-lab.
- Prefer read (inspect) tools over action tools until you know what the page does.
