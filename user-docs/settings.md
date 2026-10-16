# Settings

![Settings](./images/settings-view.png)

## Application Language

Select the language for the UI. Available: Czech (Čeština) and English.

## Agent Language

Select the language Claude will use in responses. This is independent of the UI language.

## Agent Permission Mode

Controls how Claude handles file modifications and command execution:

- **Default** — Claude asks for permission before each action (interactive approval via the chat UI)
- **Plan** — Claude creates a plan first, then asks for approval before executing
- **Automatic (Fully automatic)** — Claude runs without any permission checks. A warning is shown — use with caution as Claude will modify files and run commands without confirmation.

## CLI Agent

Select which CLI agent to use for AI operations. Default is **Claude CLI** (`claude`). Other supported agents (if installed) include Gemini CLI, Codex CLI, and Copilot CLI.

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
