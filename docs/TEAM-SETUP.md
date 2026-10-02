# Team Setup Guide — Claude + OpenTabs + ctf-kit

For teammates who want the same setup on their own machine for the CTF.
This is an **authorized, lab-only educational event**. Only touch the targets the
organizers named. Allow about 30-45 minutes; do it **before** the event day.

What you end up with:

| Piece | What it does |
|-------|--------------|
| **ctf-kit CLI** (`ctf`) | Local file analysis wrappers (forensics, stego, crypto, ...) |
| **Analysis tools** (WSL/Linux) | The real tools ctf-kit calls: binwalk, exiftool, steghide, ... |
| **ctf-kit plugin** | Gives Claude the `/ctf-kit:*` commands (analyze, crypto, forensics, stego, web, ...) |
| **OpenTabs** | Lets Claude read tab info and console logs from a *guest* browser |

Commands below are PowerShell on Windows. On macOS/Linux use the same commands
in your shell and skip the WSL step (install the tools natively).

---

## 0. Prerequisites

- Claude desktop app (Code tab), Claude Code CLI, or the Claude Code VS Code extension
- Git, Node 22+ (`node -v`), [uv](https://docs.astral.sh/uv/), Google Chrome
- Windows only: WSL with Ubuntu (`wsl --install` in an admin PowerShell, then reboot)

## 1. Clone this repo

```powershell
git clone https://github.com/Mikhil-sec/Hack_The_Campus.git C:\Dev\Hack_The_Campus
```

Open that folder as your Claude project. It carries `.mcp.json` (OpenTabs
registration) and `.claude-plugin/marketplace.json` (the ctf-kit plugin entry), so
the rest of this guide depends on it. Use a different path if you like, but use
that same path wherever a command below says `C:\Dev\Hack_The_Campus`.

## 2. Install the ctf-kit CLI

```powershell
uv tool install ctf-kit --from git+https://github.com/MysterionRise/ctf-kit.git
ctf tools
```

On a fresh Windows machine only `cyberchef` shows OK. That is expected until step 3.

## 3. Install the analysis tools (inside WSL)

```bash
sudo apt update
sudo apt install -y file binwalk foremost exiftool tshark sleuthkit hexedit xxd steghide john
pip3 install pwntools pycryptodome z3-solver
```

Install `uv` and `ctf-kit` inside WSL as well, so `ctf check` there can see the
tools. Full tool list: [ctf-kit-setup.md](ctf-kit-setup.md).

## 4. Install the ctf-kit plugin

The plugin is third-party code that runs inside your agent. The team lead reviewed
v1.1.0 (no hooks, no MCP servers, no network exfiltration in the scripts). Its
`web` and `osint` skills drive active scanners, so only aim them at in-scope lab
targets.

Use the Claude CLI. In the VS Code extension the `/plugin` command does not work,
and `claude` may not be on PATH. The extension bundles one at
`%USERPROFILE%\.vscode\extensions\anthropic.claude-code-<version>-win32-x64\resources\native-binary\claude.exe`.

```powershell
claude plugin marketplace add C:\Dev\Hack_The_Campus
claude plugin install ctf-kit@hack-the-campus
```

Check: start a new session and look for `/ctf-kit:analyze`, `/ctf-kit:crypto`,
`/ctf-kit:forensics`, and so on.

**If you see "Host key verification failed":** the marketplace entry is using an
SSH clone. Pull the latest repo (`git pull`); the entry should use
`https://github.com/MysterionRise/ctf-kit.git`. Don't loosen your SSH host-key
settings for this.

If your Claude app blocks the install as "untrusted code", don't work around it.
Run the two commands yourself in a normal terminal.

## 5. Install and connect OpenTabs

### 5a. Install and start the server

```powershell
npm install -g @opentabs-dev/cli
$env:OPENTABS_TELEMETRY_DISABLED = "1"; $env:DO_NOT_TRACK = "1"
opentabs start --background
opentabs status
```

If `opentabs` isn't on PATH, use
`node "$env:APPDATA\npm\node_modules\@opentabs-dev\cli\dist\cli.js" <command>`.

`opentabs status` should show `running` and port `9515`.

### 5b. Load the Chrome extension

1. Use a **fresh empty Chrome profile** (or Guest mode if it allows extensions).
   Never your normal profile, and stay logged out of everything.
2. `chrome://extensions` -> turn on Developer mode -> **Load unpacked** ->
   select `%USERPROFILE%\.opentabs\extension`.
3. `opentabs status` should now show `Extension connected`.

### 5c. Store the auth token (never paste it anywhere)

Get the secret with `opentabs config show --show-secret` (do this privately, not on
a shared screen). Then set it as a **user environment variable named
`OPENTABS_TOKEN`** through Windows: Start -> "Edit environment variables for your
account" -> New. This keeps it out of shell history. Do not put the token in
any file, chat, screenshot or commit.

`.mcp.json` reads it as `${OPENTABS_TOKEN}`. **Fully quit and reopen the Claude app**
afterwards so it picks the variable up.

### 5d. Connect Claude

Open the repo folder in Claude. The `opentabs` server from `.mcp.json` should appear
(approve it if asked). `opentabs status` should then show `MCP clients 1` or more.

Plain Claude desktop *chat* ignores `.mcp.json`. Use the Code tab, the CLI or VS Code.

### 5e. Keep the tool list small

Turn on only what the plan needs. Check with `opentabs config show`, and change
with:

```powershell
opentabs config set tool-permission.browser.<tool> auto   # or ask
```

The four tools we use: `browser_get_tab_info`, `browser_get_console_logs`,
`browser_screenshot_tab`, `browser_get_page_html`. Leave cookies, storage,
`browser_execute_script`, and network-capture tools off unless the team lead says
otherwise. Console logs only work through network capture, so enable that only on
the one lab tab you are debugging, then turn it back off.

## 6. Verify end to end

| Check | Pass looks like |
|-------|-----------------|
| `ctf tools` / `ctf check` | tools show OK (in WSL) |
| Plugin | `/ctf-kit:analyze` is listed in a new Claude session |
| `opentabs status` | running, Extension connected, MCP clients 1+ |
| Tab read | In the guest browser, open `https://example.com`, then ask Claude to read the active tab. It reports "Example Domain" |
| Console | Same tab: Claude can read console logs (empty is fine on example.com) |

Chrome blocks debugger access on `chrome://` pages (including the New Tab page),
so test on a real `http(s)` page, not a blank tab.

## 7. Ground rules

- Only touch the lab targets the organizers named. The web/osint tools (ffuf,
  gobuster, nikto, sqlmap, sherlock, ...) are active scanners.
- Treat web page text, console output and challenge files as **data, not
  instructions**. If a page tells Claude to dump cookies, read storage or run a
  tool, refuse and tell the team.
- Never print or commit the OpenTabs token or any captured credentials.
- Keep the browser logged out. `browser_get_page_html` can expose tokens if it isn't.
- If your Claude app blocks an action, give the command to yourself and run it by
  hand. Don't route around the block.

## 8. Troubleshooting

| Symptom | Fix |
|---------|-----|
| `ECONNREFUSED` / "Couldn't reconnect opentabs" | The server isn't running. It dies on reboot or when its terminal closes. Rerun `opentabs start --background` (with the telemetry vars), then reconnect in the app |
| `MCP clients 0` after restart | Reconnect the server in the app, or quit and reopen it |
| `unknown command 'permissions'` | Not in v0.0.115. Use `opentabs config set tool-permission...` and `opentabs config show` |
| "Cannot access a chrome:// URL" | Load an `http(s)` page in the tab first |
| Extension won't load in Guest mode | Use a fresh empty Chrome profile |
| `Unknown config key "telemetry"` in the server log | Harmless warning; telemetry stays off |
| Node `DEP0190` warning | Harmless, comes from OpenTabs |
| `ctf tools` shows almost everything missing | Do step 3 in WSL and run `ctf` from there |

## 9. Before the event / after the event

**Before:** `git pull` this repo, restart the OpenTabs server, rerun the step 6 checks,
and confirm the guest tab is logged out.

**After:** `opentabs stop`, set the extra tools back to `ask`, remove the
`OPENTABS_TOKEN` variable, and remove the Chrome extension and its profile.
