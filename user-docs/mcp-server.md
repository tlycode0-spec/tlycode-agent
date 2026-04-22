# MCP Server

TlyCode Agent contains a built-in MCP (Model Context Protocol) server that allows external AI agents (e.g. Claude Code) to control the application — manage chat sessions, kanban tickets, scheduler tasks, skills, git, docs, pipelines and project settings.

## How It Works

The MCP server exposes a JSON-RPC 2.0 endpoint at `/mcp`. When a project is loaded, the server is automatically registered in `.claude/settings.json` under the name **tlycode-agent**, so Claude Code and other MCP-compatible agents can discover and use it immediately.

- **Server mode** — the endpoint runs on the same port as the web UI (default `http://localhost:8080/mcp`)
- **Desktop mode** — a lightweight HTTP listener starts on a configurable port at launch

Each request must include the `X-Project-Path` header so the server knows which project to operate on. This header is set automatically during registration.

## Available Tools (60+)

### Chat (6 tools)

| Tool | Description |
|---|---|
| `list_chat_sessions` | List all chat sessions for the current project |
| `create_chat_session` | Create a new chat session |
| `get_chat_messages` | Get messages from a chat session |
| `add_chat_message` | Add a message (role: user or assistant) |
| `delete_chat_session` | Delete a chat session |
| `rename_chat_session` | Rename a chat session |

### Kanban (12 tools)

| Tool | Description |
|---|---|
| `get_kanban_board` | Get the board with all columns and tickets |
| `create_kanban_ticket` | Create a new ticket (title, description, system_prompt, skill) |
| `update_kanban_ticket` | Update ticket properties |
| `move_kanban_ticket` | Move a ticket to a different column/position |
| `delete_kanban_ticket` | Delete a ticket |
| `archive_kanban_ticket` | Archive or unarchive a ticket |
| `set_ticket_backlog` | Move a ticket to/from backlog |
| `create_ticket_chat_session` | Create a chat session from a ticket |
| `link_ticket_chat` | Link a ticket to an existing chat session |
| `create_kanban_column` | Create a new column |
| `create_ticket_comment` | Add a comment to a ticket |
| `get_ticket_comments` | Get all comments for a ticket |

### Ticket Attachments (4 tools)

| Tool | Description |
|---|---|
| `list_ticket_attachments` | List attachments on a ticket |
| `add_ticket_attachment` | Upload a file attachment (base64) |
| `delete_ticket_attachment` | Delete an attachment |
| `read_ticket_attachment` | Download an attachment (base64) |

### Prompt Execution (2 tools)

| Tool | Description |
|---|---|
| `send_prompt` | Send a prompt to a chat session with optional skill |
| `run_ticket` | Run a ticket (sends description + system_prompt + skill to Claude) |

### Skills (9 tools)

| Tool | Description |
|---|---|
| `list_skills` | List all skills in the project |
| `read_skill` | Read full skill content (YAML frontmatter + markdown) |
| `save_skill` | Create or update a skill |
| `delete_skill` | Delete a skill |
| `rename_skill` | Rename a skill |
| `list_skill_attachments` | List attachments on a skill |
| `add_skill_attachment` | Upload a file attachment (base64) |
| `delete_skill_attachment` | Delete a skill attachment |
| `read_skill_attachment` | Download a skill attachment (base64) |

### Scheduler (5 tools)

| Tool | Description |
|---|---|
| `list_scheduler_tasks` | List all scheduled tasks |
| `create_scheduler_task` | Create a recurring task (skill + message + interval) |
| `update_scheduler_task` | Update a task |
| `delete_scheduler_task` | Delete a task |
| `toggle_scheduler_task` | Enable or disable a task |

### Git (14 tools)

| Tool | Description |
|---|---|
| `git_status` | Get staged, unstaged, untracked files and current branch |
| `git_log` | Get commit log (with optional max_count and skip) |
| `git_branches` | List all branches |
| `git_diff` | Get diff (staged or unstaged, optionally per file) |
| `git_stage` | Stage files |
| `git_unstage` | Unstage files |
| `git_commit` | Create a commit |
| `git_push` | Push to remote |
| `git_pull` | Pull from remote |
| `git_checkout` | Checkout a branch |
| `git_create_branch` | Create a new branch |
| `git_fetch` | Fetch from remote |
| `git_show_files` | List files changed in a commit |
| `git_show_file_diff` | Show diff for a specific file in a commit |

### Documentation (6 tools)

| Tool | Description |
|---|---|
| `list_docs` | List all documentation files (tree structure) |
| `read_doc` | Read a documentation file |
| `save_doc` | Save/update a documentation file |
| `create_doc` | Create a new empty doc file |
| `delete_doc` | Delete a doc file |
| `rename_doc` | Rename/move a doc file |

### Pipelines (5 tools)

| Tool | Description |
|---|---|
| `list_pipelines` | List all pipelines |
| `get_pipeline` | Get a pipeline with nodes and edges |
| `create_pipeline` | Create a new pipeline |
| `update_pipeline_meta` | Update pipeline name and description |
| `delete_pipeline` | Delete a pipeline |

### Project Settings (6 tools)

| Tool | Description |
|---|---|
| `get_project_settings` | Get project settings (language, permission mode, CLI, prompts) |
| `save_project_setting` | Save a project setting |
| `get_recent_projects` | List recently opened projects |
| `get_env_variables` | List environment variables |
| `set_env_variable` | Set an environment variable |
| `delete_env_variable` | Delete an environment variable |

## Common Parameter

All tools accept a `project_path` parameter — the absolute path to the project directory. Use `get_recent_projects` to discover available projects.

## Manual Usage

You can test the MCP server directly with curl:

```bash
# Initialize
curl -X POST http://localhost:8080/mcp \
  -H "Content-Type: application/json" \
  -H "X-Project-Path: /path/to/project" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{}}'

# List tools
curl -X POST http://localhost:8080/mcp \
  -H "Content-Type: application/json" \
  -H "X-Project-Path: /path/to/project" \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/list","params":{}}'

# Call a tool
curl -X POST http://localhost:8080/mcp \
  -H "Content-Type: application/json" \
  -H "X-Project-Path: /path/to/project" \
  -d '{"jsonrpc":"2.0","id":3,"method":"tools/call","params":{"name":"get_kanban_board","arguments":{"project_path":"/path/to/project"}}}'
```

## Auto-Registration

When a project is loaded in TlyCode Agent, the MCP server is automatically added to `.claude/settings.json`:

```json
{
  "mcpServers": {
    "tlycode-agent": {
      "type": "http",
      "url": "http://localhost:8080/mcp",
      "headers": {
        "X-Project-Path": "/path/to/project"
      }
    }
  }
}
```

This means any Claude Code session running in the same project directory will automatically have access to all TlyCode Agent tools without any manual configuration.
