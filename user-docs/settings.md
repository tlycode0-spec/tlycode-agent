# Settings

![Settings](./images/settings-view.png)

## Application Language

Select the language for the UI. Available: Czech (Čeština) and English.

## UI Scale

Text and controls can be made larger or smaller — 80 % to 200 % in 10 % steps, 100 % by default. There are two levels:

- **App scale (Settings → Application):** the slider, the −/+ buttons or **Default (100 %)** set the scale of every app window. It applies immediately and is kept after a restart.
- **Window scale (top bar):** the magnifier icon next to the keyboard shortcuts changes only the current window (−/+ in a small popup). **Use app scale (X %)** returns the window to the app scale. A window with its own scale ignores later changes of the app scale. The main window keeps its own scale after a restart; other windows keep it while they're open.
- **Shortcuts:** Cmd/Ctrl + `=` (or `+`) and Cmd/Ctrl + `-` change the current window's scale, Cmd/Ctrl + `0` returns it to the app scale. The numpad `+`, `−` and `0` keys work too. Shortcuts follow the physical key position like all app shortcuts, so on a Czech layout zoom in is the `´` key and zoom out the `=` key (or use the numpad).
- **Webview windows:** their toolbar follows the app scale; the web page inside is left at 100 %.
- **Server mode (browser):** the app doesn't scale itself there — use the browser's own zoom (Cmd/Ctrl + `+`/`-`/`0`), which the browser remembers for the address.

## Agent Language

Select the language Claude will use in responses. This is independent of the UI language.

## Agent Permission Mode

Controls how Claude handles file modifications and command execution:

- **Default** — Claude asks for permission before each action (interactive approval via the chat UI)
- **Plan** — Claude creates a plan first, then asks for approval before executing
- **Automatic (Fully automatic)** — Claude runs without any permission checks. A warning is shown — use with caution as Claude will modify files and run commands without confirmation.

## CLI Agent

Select which CLI agent to use for AI operations. Default is **Claude CLI** (`claude`). Other supported agents (if installed) include Gemini CLI, Codex CLI, OpenCode CLI, and Copilot CLI. **LM Studio** runs a local model without any CLI — see [LM Studio](#lm-studio) below.

### OpenCode CLI

[OpenCode](https://opencode.ai) is a provider-agnostic CLI agent. Notes specific to it:

- **Sign-in** — OpenCode manages provider credentials itself. Run `opencode auth login` in a terminal; TlyCode Agent does not ask for API keys.
- **Model** — entered as `provider/model` (e.g. `anthropic/claude-sonnet-4-5`). The field suggests models reported by `opencode models` but accepts any value. Leave it empty to use the default model from your OpenCode configuration.
- **Permission mode** — **Plan** runs the OpenCode `plan` agent (read-only); every other mode runs with `--auto`. Rules denied in your OpenCode config stay denied.
- **MCP servers** — servers configured in TlyCode Agent are also synced to `~/.config/opencode/opencode.json` (section `mcp`); other keys in that file are preserved.
- **Streaming** — response text arrives in chunks per agent step, not token by token.

### LM Studio

The **LM Studio** agent chats with a model served by [LM Studio](https://lmstudio.ai) through its native chat API (`POST /api/v1/chat`). LM Studio runs the tool loop itself; TlyCode Agent hands it the tools over MCP and streams the answer token by token.

- **Requirements**
  - LM Studio **0.4 or newer** with its server running.
  - **Allow per-request MCPs** switched on in LM Studio's server settings — without it the model gets no tools.
  - A model that supports tool calling. The model picker only lists such models.
  - The **System Check** reports a missing server, an old version, the per-request MCP switch and a missing API token.
- **Model, host and port** — the model comes from LM Studio and is loaded on the first request if it is not loaded yet. LM Studio runs on `127.0.0.1:1234` unless you set *Model host* and *Port* — see [Per-project host and port](./server-mode.md#per-project-host-and-port).
- **API token** — when LM Studio requires an API token, put it into the `LM_API_TOKEN` environment variable of the project (or of the process running TlyCode Agent). In server mode an admin can also store it as a group secret.
- **Tools** — the model can read, write and search files, run shell commands and keep a todo list. It also gets a selection of the built-in tlycode-agent MCP tools (kanban tickets and comments, docs, skills, everyday git). Both are served by TlyCode Agent's own MCP server and **LM Studio calls them itself**, so:
  - tools only work when LM Studio runs **on the same machine** as TlyCode Agent (the desktop app or the server process),
  - with a remote host the agent is a plain chat without tools, and its first answer says so.
  - There is no "ask the user" tool — the model asks in its answer and you reply in the next message.
- **Permission mode** — **Plan** offers read-only tools only. Every other mode gives the model full file and shell access **without per-call approvals**, so use it in trusted projects only.
- **Limits**
  - A single tool call may currently take about **60 seconds** — LM Studio's fixed timeout. TlyCode Agent stops a tool after 50 s and returns its partial output. Run long commands (builds, test suites) in the background with their output redirected to a file (`cmd > /tmp/out.log 2>&1 &`) and let the model read the file later.
  - **Context management cannot help within a single turn.** LM Studio runs the tool loop, so when one message fills the context with tool output, nothing can be taken out of it. The defence is small tool output: TlyCode Agent cuts the output of its own tools to fit the window; servers from `~/.lmstudio/mcp.json` are out of its reach. When it happens, LM Studio ends the turn without an error of its own and the chat shows that the response ended without a result or came back empty; the next message then starts a new chain.
  - A turn stops after **50 tool calls** (setting `agent_local_max_iterations`, `0` = no limit). LM Studio has no limit of its own and a model can loop on tools until its context overflows.
- **Conversation** — after the first message only the new message is sent; LM Studio continues the conversation it stored (`previous_response_id`). The history is sent again as a new chain when that is not possible: after a change of model, host, context length, persona, skill or language, after you delete or edit a message, after a failed or empty turn, when LM Studio no longer knows the conversation, or when the chain has filled the context window past the threshold.
- **Context management** — LM Studio doesn't compress the context itself; a full chain returns empty answers. So TlyCode Agent watches how full the chain is and, past **Compaction threshold** (`agent_local_context_threshold`, default 75 %), starts the next message in a new chain. What that chain gets is up to **Context management** (`agent_local_context_policy`):
  - **Compact** (default) — the system prompt, a summary of the older part of the conversation and the newest messages. The summary is a separate request to the same model (not stored in LM Studio, no tools); the next compaction builds on it rather than summarizing everything again. If the summary fails, *Truncate middle* is used instead.
  - **Truncate middle** — the system prompt, your first message (the assignment), a note how many messages were left out and the newest messages.
  - **Rolling window** — the system prompt and the newest messages.
  - **Off** — the chain grows until the window is full; the chat only warns once it passes the threshold.

  The history of a new chain includes the model's tool calls, with each tool output cut to about 1 500 characters (shell output keeps its end), and takes at most 40 % of the window. Every step shows in the chat as a grey line between the messages — compacted (with the summary to unfold), shortened, new chain with its reason, or the warning — and stays there after a reload. These lines are never sent to a model. A new chain means one full prompt prefill (about 16 s for 16k tokens on gemma-4-e4b); compaction adds the summary request on top.
- **Context fill** — each answer shows how full it left the window (`ctx 16.1k / 16.4k (98%)`), in the warning colour past the threshold; the session footer shows the current chain.
- **Privacy** — LM Studio stores every conversation, including tool output, **for 30 days** on the disk of the machine it runs on, in `~/.lmstudio/.internal/api-prediction-history`.
- **Images** — image attachments are sent to the model directly, so a vision model can see them.

## Launch at Startup

Enable to start TlyCode Agent automatically when the system boots. Desktop only.

## System Check

Click **Run System Check** to re-check the installation status of Claude CLI, Node.js, GitHub CLI, and other supported tools.

## Prompts — moved

The project's prompts (append prompt, first-message prompt, prompt before
deleting a session and the per-column prompts) are no longer a settings tab.
They live on the Kanban board under **Agent profile**, the third view next to
the board and the flow manager — see [Kanban Board](./kanban.md#agent-profile).

The one thing worth repeating here: the **Column prompts** switch is per
project and lives in `.tlycode-settings.json` (`column_prompts_enabled`). Agent
profiles carry prompt *content*, so switching a profile does not change that
switch.

## Kanban

The tab is empty: the Kanban board has no settings of its own at the moment.
More will land here.

### Custom ticket fields, without the editor

Custom ticket fields — a Jira ticket number, an estimate, whatever the project
needs — used to be defined on this tab. The fields themselves are still there
and a board that has them keeps showing them; what is gone is the screen that
edited the project-wide ones. Two ways in are left:

- **Per ticket** — the ticket dialog's own **Add field**. A field added there
  belongs to that ticket alone and has its own switch for the card. That covers
  everything a single ticket needs.
- **For the whole project** — the `ticket_fields` key in
  `.tlycode-settings.json` (in server mode, the project's settings in the
  database), which an agent writes over MCP with `save_project_setting` and
  reads with `get_project_settings`. Each entry has a `key` (fixed, it is what
  the values hang on), a `label`, a `show_on_card` switch and a `source`:
  `manual`, `session_tokens` or `session_hot`.

Deleting a field does **not** erase the values stored on the tickets: add the
field back and they reappear.

### Chips computed from the session

A field whose source is the session is not typed in at all — it is read off the
linked session's latest turn:

- **Tokens in session context** (`session_tokens`) — how much the session has in
  its context, i.e. the input side of its last turn (new input + cache read +
  cache write). This is *not* the cumulative "Σ tokens" the chat shows: that one
  counts the whole history and is much larger. The last turn's output is not
  counted either — it only enters the context when the next request sends it
  back.
- **Session hotness** (`session_hot`) — until when the session's prompt cache
  stays hot, as `until 14:32 (12 min)`, and `cold` once the window is over.
  Waking a cold session re-reads the whole context, so this is what tells you
  whether to pick the session up now or let it go.

A computed field has no value to type in, so the ticket dialog does not offer
it. A ticket with no linked session — or one whose agent reports no token usage
— shows no computed chip.

Two project settings decide when such a chip is highlighted. They have no editor
either; they are read the same way as `ticket_fields` and fall back to the
defaults below.

- **`session_hot_minutes`** — how long after the last turn the session counts as
  hot. No agent reports this (Claude has both a 5-minute and a 1-hour prompt
  cache), so the chip is only as honest as this number. Default 60.
- **`session_context_warn_tokens`** — the context size at which rotating the
  session pays off. Above it the token chip is highlighted; nothing is rotated
  automatically. Default 160000.

## Log Level

Set the verbosity of debug logs:

- **Error** — only errors
- **Warning** — errors and warnings
- **Info** — general information
- **Debug** — detailed debug output
- **Trace** — everything, including throttling and scroll events

## Environment Variables

Define environment variables that will be set when Claude runs commands:

- Click **Add** to create a new variable
- Variables are synchronized with the `.env` file in the project directory
- Useful for API keys, custom paths, configuration values
- Changes are saved immediately

## MCP Servers

Configure Model Context Protocol servers that extend Claude's capabilities:

- **Local (stdio)** — a command that runs locally (e.g. `npx -y @modelcontextprotocol/server-name`)
- **HTTP** — a remote MCP server URL with optional headers
- **HTTP with OAuth** — remote servers that require OAuth authentication

Servers are stored in `~/.claude.json`:
- Global servers (like "tlycode") are in the root scope
- Project-specific servers are under `projects.<project_path>`

To add a server:

1. Choose transport type (Local or HTTP)
2. Enter the command/URL and arguments
3. Optionally add environment variables or headers
4. Click **Add Server**

Servers can be edited or deleted from the list.

### Built-in tlycode-agent Server

TlyCode Agent automatically registers its own MCP server named **tlycode-agent** when a project is loaded. This server provides 60+ tools for controlling chat, kanban, scheduler, skills, git, docs, pipelines and settings — allowing external agents like Claude Code to interact with the application.

See [MCP Server](./mcp-server.md) for full documentation.
