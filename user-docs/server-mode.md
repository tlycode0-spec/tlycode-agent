# Server Mode

TlyCode Agent can run as a web server, making it accessible from any browser on the network.

## Starting the Server
```bash
# Using the server binary (auto-detects server mode from binary name)
./tlycode-agent-server --port 8080

# Or using the main binary with --server flag
./tlycode-agent --server --port 8080

# With custom data directory
./tlycode-agent-server --port 8080 --data-dir /path/to/data
```

The server starts and prints:

```
Starting TlyCode Agent server on http://0.0.0.0:8080
WebSocket events available at ws://0.0.0.0:8080/ws/events
Frontend is embedded in the binary
```

Open `http://localhost:8080` in your browser to use the application.

## Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `--port` | 8080 | HTTP port to listen on |
| `--data-dir` | `~/.local/share/com.tlycode.agent` | Database and data directory |
| `--server` | — | Force server mode (not needed if binary is named `*server*`) |

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `TLYCODE_LMSTUDIO_URL` | `http://127.0.0.1:1234/v1` | Fallback LM Studio endpoint for the LM Studio agent, the model picker and the system check. Given with `/v1`; the agent derives the native `/api/v1` from it |
| `LM_API_TOKEN` | — | LM Studio's API token, when its API authentication is on. A project's env variable or a group secret takes precedence |

The LM Studio agent runs inside the server process, so it reaches LM Studio
over the server's own network. When the server does not share a loopback with LM
Studio, point it at the right host:

```bash
# Server in a container, LM Studio on the host
TLYCODE_LMSTUDIO_URL=http://host.docker.internal:1234/v1 ./tlycode-agent-server --port 8080

# LM Studio on another machine (enable "Serve on Local Network" in LM Studio)
TLYCODE_LMSTUDIO_URL=http://192.168.1.10:1234/v1 ./tlycode-agent-server --port 8080
```

### LM Studio requirements

- LM Studio **0.4 or newer** — the agent uses its native chat API `POST /api/v1/chat`.
- **Allow per-request MCPs** on in LM Studio's server settings, so the model gets its tools.
- With LM Studio's API authentication on, an API token in `LM_API_TOKEN` or in the group secret.

Everything else about the agent (tools, limits, stored conversations) is in
[Settings → LM Studio](./settings.md#lm-studio).

### Per-project host and port

**Settings → Agent** has a *Model host* and *Port* pair which takes precedence
over `TLYCODE_LMSTUDIO_URL`. The env variable stays the deployment-wide
fallback — useful for a container that should work before anyone opens the
settings — while the fields are what an individual user reaches for.

The host field is forgiving: a bare host (`spark.local`), `host:port` or a whole
URL (`http://10.0.0.5:1234/v1`) all resolve. Empty fields mean `127.0.0.1:1234`.

In server mode both values are per-user (`USER_SCOPED_KEYS`), so each member can
point at their own machine, and an admin can pin them for a group under
**Settings → Admin → Groups**.

### What a remote endpoint cannot do

Only LM Studio's HTTP API travels over the network. Model management
(start server, load / unload, download) goes through the `lms` CLI, which drives
LM Studio on the machine running tlycode and nothing else. With a non-loopback
host configured:

- the model picker lists what the remote server reports over `GET /v1/models`
  (i.e. its *loaded* models) instead of what `lms ls` finds locally,
- the `lms`-only controls are hidden, and models are loaded in LM Studio on that
  host,
- the agent runs **without tools**. LM Studio itself calls the tool endpoint of
  the tlycode server (`http://127.0.0.1:<port>/mcp/agent-tools`), which only
  works when both run on the same machine — not from a container's host or
  another computer,
- the system check still probes the env / default endpoint, because it runs
  before a project — and therefore a per-project setting — is known.

## Architecture

The server binary is self-contained:

- **Frontend** (React) is embedded in the binary via `rust-embed` — no external files needed
- **REST API** — all backend operations via `POST /api/<command>` endpoints
- **WebSocket** — real-time streaming at `ws://<host>:<port>/ws/events`
- **Database** — SQLite stored in the data directory

## Differences from Desktop

| Feature | Desktop | Server |
|---------|---------|--------|
| System notifications | Native OS notifications | Disabled |
| Open in VS Code | Available | Hidden |
| File dialog | Native OS dialog | Built-in folder browser |
| System tray | Yes | No |
| Autostart | Available | Not applicable |
| Multiple users | Single user | Can serve multiple browsers |

## WebSocket Events

The WebSocket connection receives JSON messages in the format:

```json
{ "event": "<event-name>", "payload": { ... } }
```

Events:

| Event | Description |
|-------|-------------|
| `claude-stream-<session_id>` | Real-time streaming of Claude's response (phase, content, actions) |
| `permission-request-<session_id>` | Claude requests permission for a tool use |
| `scheduler-task-executed` | A scheduler task was automatically executed |

## Deployment

### Standalone

Copy the single binary to your server and run it. No dependencies needed (except Claude CLI, Node.js on the host for agent operations).

### Docker

```dockerfile
FROM debian:bookworm-slim
COPY tlycode-agent-server /usr/local/bin/
RUN apt-get update && apt-get install -y nodejs npm git && \
    npm install -g @anthropic-ai/claude-code
EXPOSE 8080
CMD ["tlycode-agent-server", "--port", "8080"]
```

### Docker Runner

When a project runs its prompts inside the Docker runner image, the CLI agents (Claude, Gemini, Codex, OpenCode, Copilot) come preinstalled in the image. Credentials live in named Docker volumes so they survive container restarts:

| CLI | Volume | Mounted at |
|-----|--------|------------|
| Claude | `tlycode-runner-auth` | `/root/.claude` |
| Gemini | `tlycode-runner-gemini` | `/root/.gemini` |
| Codex | `tlycode-runner-codex` | `/root/.codex` |
| OpenCode | `tlycode-runner-opencode-data` | `/root/.local/share/opencode` (credentials, sessions) |
| OpenCode | `tlycode-runner-opencode-config` | `/root/.config/opencode` (config, MCP servers) |

Seed the credentials once with an interactive login inside a disposable container:

```bash
sh scripts/tlycode-runner-login.sh            # Claude
sh scripts/tlycode-runner-login.sh opencode   # OpenCode (runs `opencode auth login`)
```

Remove a volume with `docker volume rm <name>` to reset the login.

### Reverse Proxy (nginx)

```nginx
server {
    listen 443 ssl;
    server_name tlycode.example.com;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
    }
}
```

The `Upgrade` and `Connection` headers are required for WebSocket support.
