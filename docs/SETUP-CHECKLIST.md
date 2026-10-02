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

## 3. OpenTabs — still to do (npm install was blocked again)

```powershell
npm install -g @opentabs-dev/cli
opentabs start      # copy the printed auth secret
```

1. Chrome: `chrome://extensions` -> Developer mode -> Load unpacked -> `%USERPROFILE%\.opentabs\extension`
2. Register with Claude Code (the `claude` CLI is not on PATH in this shell, so run it where it is available):
   `claude mcp add --transport http opentabs http://127.0.0.1:9515/mcp --header "Authorization: Bearer <secret>"`
3. `opentabs status` should show the extension connected.

## 4. Optional

- browser-use: `uv pip install browser-use` (only if the team will use it)
- PinchTab: follow https://github.com/pinchtab/pinchtab
- Do not commit the OpenTabs auth secret.
