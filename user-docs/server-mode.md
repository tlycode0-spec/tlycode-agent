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
