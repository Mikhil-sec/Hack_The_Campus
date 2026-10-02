# Hack_The_Campus — Team Reference Repo

Shared learning, syntax cheatsheets, and tooling notes for our 4-hour team CTF
(authorized, lab-environment educational event).

This repo is **reference material only**. Nothing here runs active network
tasks on its own — it documents tools so the team can set them up and use them
efficiently during the event.

## Contents

| File | Topic |
|------|-------|
| [docs/static-doc-lookup.md](docs/static-doc-lookup.md) | Text-only conversion wrappers (PinchTab, WebFetch) for parsing static event docs |
| [docs/local-session-integration.md](docs/local-session-integration.md) | Browser-inspection extensions (OpenTabs) for console/network on an authenticated challenge site |
| [docs/high-level-automation.md](docs/high-level-automation.md) | Structural command abstractions (browser-use) vs. raw E2E scripts |
| [docs/ctf-kit-setup.md](docs/ctf-kit-setup.md) | Installing & initializing MysterionRise/ctf-kit + its benign file-analysis components |
| [docs/TEAM-SETUP.md](docs/TEAM-SETUP.md) | **Start here:** full teammate setup (ctf-kit, plugin, OpenTabs) |
| [docs/TEAM-PROMPT.md](docs/TEAM-PROMPT.md) | Paste-ready prompt that has Claude walk you through the setup |

## Scope & ground rules

- Only operate against the designated lab targets we are authorized to touch.
- Keep credentials, session tokens, and any captured data inside the lab.
- These notes describe publicly-documented, open-source tools. Source links are
  included so anyone can verify against upstream docs.
