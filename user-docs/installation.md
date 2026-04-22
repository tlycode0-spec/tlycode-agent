# Installation

## Desktop Application

### Linux (.deb)

Download the `.deb` package from the [Releases page](https://github.com/tlycode0-spec/tlycode-agent/releases) and install:

```bash
sudo dpkg -i TlyCode.Agent_*.deb
```

### macOS (.dmg)

Download the `.dmg` file from the Releases page, open it and drag TlyCode Agent to Applications.

> **Note:** If macOS shows "TlyCode Agent.app is damaged and can't be opened", the app is not yet notarized by Apple. Remove the quarantine flag:
> ```bash
> xattr -cr /Applications/TlyCode\ Agent.app
> ```

### Windows (.exe)

Download the `.exe` installer from the Releases page and run it.

## Server Mode

Download the `tlycode-agent-server` binary for your platform from the Releases page:

- `tlycode-agent-server-linux-amd64` — Linux x86_64
- `tlycode-agent-server-macos-arm64` — macOS Apple Silicon
- `tlycode-agent-server-macos-amd64` — macOS Intel
- `tlycode-agent-server-windows-amd64.exe` — Windows

```bash
chmod +x tlycode-agent-server-linux-amd64
./tlycode-agent-server-linux-amd64 --port 8080
```

The server binary is self-contained — the frontend is embedded inside. No additional files needed.

## Prerequisites

The following tools should be installed on the system where TlyCode Agent runs:

- **Claude CLI** — `npm install -g @anthropic-ai/claude-code`
- **Node.js** — [nodejs.org](https://nodejs.org/)
- **GitHub CLI** (optional) — [cli.github.com](https://cli.github.com/)

The application checks for these tools on first launch (System Check).

## Building from Source

### Desktop

```bash
npm ci
cargo tauri build --bundles deb
```

### Server

```bash
npm ci
npm run build
cd src-tauri
cargo build --release --no-default-features --features server
```

### Both (via Docker)

```bash
./build.sh          # builds both desktop + server
./build.sh desktop  # desktop only
./build.sh server   # server only
```
