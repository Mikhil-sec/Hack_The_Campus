# Setup Checklist (do these by hand)

Machine check done 2026-10-02. Already present: uv 0.12.2, Python 3.14.4,
Node 24.18 (OpenTabs needs 22+), npm 11, git, winget, WSL (Ubuntu, stopped),
Chrome (`%LOCALAPPDATA%\Google\Chrome\Application\chrome.exe`).

Not installed because auto-mode blocked installing third-party code. Run these yourself.

## 1. ctf-kit (PowerShell) — DONE

Installed on Windows (v0.1.0) and `ctf tools` works. Only `cyberchef` shows OK;
everything else is missing until step 2.

## 2. Analysis tools inside WSL (Ubuntu)

```bash
sudo apt update
sudo apt install -y file binwalk foremost exiftool tshark sleuthkit hexedit xxd steghide john
pip3 install pwntools pycryptodome z3-solver
```

Then install ctf-kit inside WSL too (needs uv there) so `ctf check` sees the tools.

## 3. OpenTabs — DONE (verified 2026-10-02)

Server v0.0.115, Chrome extension and MCP connection in the Claude desktop app
all work; tab info and console logs verified on `https://example.com`.
Registration is via the repo's `.mcp.json`, which reads the token from the
`OPENTABS_TOKEN` user env var. Do not use `claude mcp add` with a real secret.

The server dies on reboot or when its terminal closes. Restart with:

```powershell
$env:OPENTABS_TELEMETRY_DISABLED = "1"; $env:DO_NOT_TRACK = "1"
opentabs start --background
```

Full steps for teammates: [TEAM-SETUP.md](TEAM-SETUP.md).

## 3b. ctf-kit plugin — DONE (verified 2026-10-02)

Installed with `claude plugin marketplace add C:\Dev\Hack_The_Campus` and
`claude plugin install ctf-kit@hack-the-campus`. The `/ctf-kit:*` commands are
listed in new sessions. The marketplace entry clones over HTTPS (the `github`
source used SSH and failed host-key verification).

## 4. Optional

- browser-use: `uv pip install browser-use` (only if the team will use it)
- PinchTab: follow https://github.com/pinchtab/pinchtab
- Do not commit the OpenTabs auth secret.
