# ctf-kit — Local Setup & Benign File-Analysis Components

ctf-kit (MysterionRise/ctf-kit) is an AI-assisted toolkit that integrates with
AI coding agents and common security tools to help **analyze, solve, and
document** CTF challenges. These notes cover installing it locally and the benign
file-analysis components it wraps. No active external network tasks here — this
is local install + reference.

Repo: https://github.com/MysterionRise/ctf-kit

## 1. Install the CLI

ctf-kit installs globally via `uv` and integrates into an existing repo without
changing its structure.

```bash
uv tool install ctf-kit --from git+https://github.com/MysterionRise/ctf-kit.git
```

(Install `uv` first if needed — see the astral-sh/uv docs.)

## 2. Initialize in a challenge folder

Run inside any challenge directory. It creates a hidden `.ctf` folder for
workspace data and leaves the original challenge files untouched.

```bash
ctf init                          # initialize a challenge folder
ctf init --category forensics     # ...optionally with a category
ctf init --repo                   # one-time init for a whole repo
ctf new <name> --category <cat>   # create a new challenge folder
```

Day-to-day commands (from the upstream README):

```bash
ctf analyze <path> [--verbose]    # analyze challenge files
ctf check [--category <cat>]      # which tools are installed
ctf tools                         # list all tools and their status
ctf run <tool> [args...]          # run a wrapped tool directly
ctf writeup [--format md|html]    # generate a writeup
ctf here                          # set competition context for this directory
ctf status                        # show challenge/competition status
ctf flag                          # submit a flag for the current challenge
ctf competition                   # manage competitions
```

Verified locally with `ctf tools` (v0.1.0): 33 wrappers across categories
forensics, stego, crypto, encoding, misc, reversing, pwn, osint and web. The
web/osint wrappers (ffuf, gobuster, nikto, sqlmap, shodan, sherlock,
theharvester) are active scanners, so point them only at the in-scope lab
targets. The file-analysis ones used in this doc (file, strings, exiftool,
binwalk, cyberchef) are local and passive. Only `cyberchef` showed as OK on a
fresh Windows install.

## 3. Install the underlying analysis tools

ctf-kit orchestrates standard, open-source CLI tools. On Ubuntu/Debian the
project documents this baseline (install only what you need):

> **Windows note:** these are Linux packages. On our Windows machines, run
> everything in this section inside **WSL (Ubuntu)**, including `uv` and
> `ctf-kit` itself, so `ctf check` can see the tools.

```bash
# file ID, carving, metadata  | network       | disk forensics | hex
sudo apt update
sudo apt install -y \
    file binwalk foremost exiftool \
    tshark wireshark \
    sleuthkit \
    hexedit xxd \
    gdb radare2 \
    hashcat john \
    steghide

# Python tooling
pip3 install volatility3 pwntools z3-solver pycryptodome

# Ruby stego helper
gem install zsteg
```

Check what ctf-kit can see:

```bash
ctf check --category stego      # list available tools in a category
```

## 4. Benign file-analysis components (what it wraps)

### File metadata & identification
- **file** — identify type via magic bytes.
- **exiftool** — read embedded metadata (EXIF, author, timestamps, comments).
- **binwalk** — scan for embedded files / appended data.
- **xxd / hexedit** — raw hex inspection.

### Encoding / string detection
- **strings** — pull printable strings out of a binary/image.
- Structured workflow (from ctf-kit's forensics notes): identify file type via
  magic bytes → extract with binwalk/strings → inspect for encoded content
  (base64/hex and similar) in common hiding spots.

### Steganography analysis
ctf-kit's `StegoAnalyzer` orchestrates several tools over an image:

```
IMAGE_TOOLS = ['exiftool', 'zsteg', 'steghide', 'stegsolve', 'binwalk']
AUDIO_TOOLS = ['exiftool', 'sonic-visualizer', 'deepsound']
```

Common commands it drives:

```bash
zsteg image.png          # LSB analysis (PNG/BMP)
zsteg -a image.png       # all combinations
steghide extract -sf image.jpg
exiftool image.jpg       # metadata
binwalk image.png        # embedded files
xxd image.png | head -50 # hex peek
```

### Forensics / archives
- **sleuthkit** — disk & filesystem examination.
- **volatility3** — memory-image analysis.
- **tshark / wireshark** — pcap inspection.
- Encrypted-ZIP workflow: list entries → known-plaintext attack where
  applicable → fall back to hash extraction + `john` for password cracking.

### Category slash-commands (agent integration)

With the Claude plugin installed (`ctf-kit@hack-the-campus`, see
[TEAM-SETUP.md](TEAM-SETUP.md) step 4), ctf-kit exposes these as `/ctf-kit:*`
commands (verified against plugin v1.1.0):

```
/ctf-kit:analyze     triage unknown files, detect category
/ctf-kit:here        set context for the current challenge folder (.ctf/)
/ctf-kit:status      challenge / competition progress
/ctf-kit:flag        save and validate a flag
/ctf-kit:crypto  /ctf-kit:forensics  /ctf-kit:stego  /ctf-kit:web
/ctf-kit:pwn     /ctf-kit:reverse    /ctf-kit:osint  /ctf-kit:misc
/ctf-kit:team-solve  /ctf-kit:compete    multi-agent modes (need
                                         CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1)
```

The older `/ctf.forensics` style names from the upstream README are gone. The
`web` and `osint` skills drive active scanners: lab targets only. Most skills
call the Linux tools from section 3, so run them where those are installed.

## 5. Orchestration patterns (from the project plan)

ctf-kit documents two useful shapes:

- **Sequential pipeline** — e.g. encrypted ZIP: inspect → try known-plaintext →
  fall back to cracking.
- **Parallel analysis** — e.g. stego image: run `zsteg`, `exiftool`, `binwalk`
  concurrently and collect results by tool.

## Notes

- Everything above is local analysis of files we're given. Keep it scoped to the
  authorized lab materials.
- Verify commands against the upstream README/`docs/` before the event — the
  tool list evolves.
