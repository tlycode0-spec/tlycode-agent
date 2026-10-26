# MCP Server

TlyCode Agent contains a built-in MCP (Model Context Protocol) server that allows external AI agents (e.g. Claude Code) to control the application — manage chat sessions, kanban tickets, scheduler tasks, skills, git, docs, pipelines and project settings.

## How It Works

The MCP server exposes a JSON-RPC 2.0 endpoint at `/mcp`. When a project is loaded, the server is automatically registered in `.claude/settings.json` under the name **tlycode-agent**, so Claude Code and other MCP-compatible agents can discover and use it immediately.

- **Server mode** — the endpoint runs on the same port as the web UI (default `http://localhost:8080/mcp`)
- **Desktop mode** — a lightweight HTTP listener starts on a configurable port at launch

Each request must include the `X-Project-Path` header so the server knows which project to operate on. This header is set automatically during registration.

## Available Tools (60+)

### Chat (9 tools)

| Tool | Description |
|---|---|
| `list_chat_sessions` | List all chat sessions for the current project |
| `create_chat_session` | Create a new chat session |
| `get_chat_messages` | Get messages from a chat session |
| `add_chat_message` | Add a message (role: user or assistant). A `user` message is also queued for that session's agent and delivered in the prompt of its next run — no run is started (use `send_prompt` for that); pass `deliver_to_agent: false` to only write it into the transcript |
| `add_assistant_attachment` | Upload an image (or other file) into a session so it can be shown in a message. Pass exactly one source: `file_path` — an existing file inside the project or the session's worktree, copied in without its bytes passing through the agent's output (preferred for screenshots and rendered charts) — or `data_base64`. Returns the attachment `id`, shown via `add_chat_message` (`attachment_ids`, or `![alt](tlycode-att://<id>)` inline) |
| `delete_chat_session` | Delete a chat session |
| `archive_chat_session` | Archive a session or bring it back. The kanban cards linked to it follow by default (and leave any flow plans); `archive_linked_ticket: false` puts away only the conversation and leaves the card on the board |
| `search_archived_chat_sessions` | Search archived sessions by title, by message content, or both (`in_title`, `in_messages`). Each hit reports `message_matches`, not the messages themselves; without a query it lists the archive |
| `search_chat_messages` | Find the messages of one session containing a text and return a snippet around each match, not the whole message |
| `star_chat_session` | Star a session (it stays at the top of the session list) or remove the star with `starred: false` |
| `rename_chat_session` | Rename a chat session |

### Kanban (12 tools)

| Tool | Description |
|---|---|
| `get_kanban_board` | Get the board with all columns and tickets |
| `create_kanban_ticket` | Create a new ticket (title, description, system_prompt, skill, and optionally its `custom_fields`) |
| `update_kanban_ticket` | Update ticket properties, including its custom fields (`custom_fields`, an object of field name → text; a name the project does not define becomes a field of that ticket alone; omit the argument and the fields stay as they are) |
| `move_kanban_ticket` | Move a ticket to a different column/position |
| `reorder_kanban_tickets` | Set the order of one column's cards in a single call |
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

### Kanban Flows (18 tools)

| Tool | Description |
|---|---|
| `list_kanban_flows` | List the project's flows with status and progress (archived ones only with `include_archived`) |
| `get_kanban_flow` | One flow with its steps and the state of every ticket |
| `get_kanban_flow_log` | The run log: start/stop/reset, dispatches, failures |
| `get_kanban_flow_usage` | What the run cost: tokens and time per ticket, per step and in total |
| `create_kanban_flow` | Create an empty flow |
| `update_kanban_flow` | Rename a flow, or change its description / stop-on-error |
| `delete_kanban_flow` | Delete a flow (refused while it runs) |
| `archive_kanban_flow` | Archive or restore a flow (hidden from the list, kept with its steps, tickets and log; refused while it runs; an archived flow cannot be started) |
| `create_flow_step` | Add a step (sequential/parallel, own/shared session, the column that finishes a ticket, pause after the step) |
| `update_flow_step` | Rename a step or change how it runs (while the flow runs, only a step it has not got to yet) |
| `delete_flow_step` | Delete a step (while the flow runs, only a step it has not got to yet) |
| `reorder_flow_steps` | Set the order the steps run in (while the flow runs, running and finished steps keep their places) |
| `add_flow_items` | Put tickets into a step |
| `move_flow_item` | Move a ticket inside a step or into another step |
| `remove_flow_item` | Take a ticket out of a step |
| `set_flow_item_status` | Mark one ticket done, take it out of the run, or put it back |
| `start_kanban_flow` | Start or resume the run |
| `stop_kanban_flow` | Stop the run and cancel the turns in flight |
| `reset_kanban_flow` | Put every ticket back to waiting |

The update tools are partial: only the fields you pass change.

A step makes no worktrees: each ticket runs in the directory set on the ticket
(its worktree, or the main project). Giving a ticket a worktree of its own is
the job of the agent profile's skills. Beyond its order and modes, a step has
these settings. `done_column` names the board column a ticket has to reach before the
step counts it as finished — for boards whose columns send prompts of their
own, where the first agent run is only the opening move. `enforce_done_column`
(with `done_column`, Claude and Codex agents) keeps the agent from ending its
turn while the ticket is still in the column it started in: it is reminded to
move it, at most twice, and then the ticket fails at once. `gate_command`
(Claude and Codex agents) is a check such as `cargo check` the app runs in the
ticket's worktree whenever the agent wants to end its turn: while it fails the
agent is sent back with its output, and after `gate_max_attempts` failures
(default 3) the ticket fails. `pause_after` stops
the run once the step is done instead of starting the next one — the way to try
a single step out — and a later start carries on from the step that follows.
`prompt_profile` names an agent profile of the flow's project (`list_agent_profiles`) that replaces
the project's prompts for the step's tickets and their chat sessions while it
works on them.

`move_kanban_ticket` fires the prompt of the column the ticket lands in, the
same as a move on the board does. It goes through the session's pending queue
rather than the single-shot channel: an agent moving its own ticket on is
itself mid-turn, so the prompt queues up behind that turn instead of racing it.
That is also what keeps the column chain a flow step's `done_column` relies on
going. The agent profile's Column prompts switch applies here too.

`set_flow_item_status` decides one ticket's fate instead of the whole run's:
`done` marks it finished without running it, `skipped` takes it out of the run,
`pending` puts it back in the queue. A running ticket has its turn cancelled
first, and a step or flow that had called itself finished reopens for a ticket
put back in.

### Agent Profiles (6 tools)

An agent profile is a named set of the project's prompts: `initial_prompt` (the
first message of a session), `append_prompt` (every prompt),
`system_prompt_before_delete` — each with an optional skill or persona — and
`column_prompts`, a prompt per board column. Profiles belong to a project:
each project has its own, plus the built-in `default`, and two projects may
have a profile of the same name. Every profile tool takes the project's
`project_path`.

| Tool | Description |
|---|---|
| `list_agent_profiles` | List the project's profiles and which one it runs under |
| `get_agent_profile` | Read one profile's fields |
| `create_agent_profile` | Create a profile (fields optional) |
| `copy_agent_profile` | Copy a profile from another project (`from_project_path: ""` copies from the profiles shared before profiles became per project) |
| `update_agent_profile` | Change fields; only the ones you send change |
| `delete_agent_profile` | Delete a profile |
| `set_active_agent_profile` | Make a profile the one the project runs under |

A profile is only a saved preset — what a turn reads is the project's own
settings. `set_active_agent_profile` writes the profile's values into them. A
profile **replaces** the project's prompts rather than layering over them, so
an empty field in it clears the project's value. Editing the active profile
applies it right away; deleting it drops the project back to `default`, the
built-in profile, which is read only.

### Prompt Execution (2 tools)

| Tool | Description |
|---|---|
| `send_prompt` | Send a prompt to a chat session with optional skill |
| `run_ticket` | Run a ticket (sends description + system_prompt + skill/persona to the agent) |

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
