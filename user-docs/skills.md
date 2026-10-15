# Skills

Skills are reusable prompts/instructions that can be attached to chat messages to give Claude specific context.

![Skills List](./images/skills-view.png)

## What is a Skill?

A Skill is a `SKILL.md` file stored in your project's `.claude/skills/<skill-name>/` directory. When you select a skill in the chat, its content is sent as a system prompt to Claude.

Skills can contain:

- Code style guidelines
- Project-specific instructions
- Task templates
- Domain knowledge

## Skill Editor

![Skill Editor](./images/skill-editor.png)

The editor has three tabs:

### Metadata

The metadata tab shows a form for YAML frontmatter fields:

| Field | Description |
|-------|-------------|
| **Name** | Display name of the skill |
| **Description** | Short description |
| **Allowed Tools** | Comma-separated list of tools Claude can use |
| **Model** | Specific Claude model to use (optional) |
| **Context** | `inline` (default) or `fork` — how the skill runs |
| **Agent** | Agent to use (optional) |
| **User Invocable** | Whether users can invoke this skill directly |
| **Disable Model Invocation** | Prevent the model from auto-invoking this skill |
| **Argument Hint** | Hint text for the skill argument |

### Markdown

Raw markdown editor for the skill body (the actual prompt content).

### Preview

Rendered preview of the full SKILL.md file including frontmatter.

## Skill Attachments

Skills can have file attachments stored in `.claude/skills/<skill-name>/attachments/`:

- Click **Attach File** to upload a file (stored as base64)
- Attachments are listed with their file name and size
- Delete attachments you no longer need

Attachments provide additional context that Claude can reference when the skill is used.

## Managing Skills

- Click **New skill** to create a new skill
- Click a skill in the list to view/edit it
- Use the **Refresh** button to reload the skill list
- Right-click a skill to rename or delete it

## Full ticket cycle (built-in)

The built-in **Full ticket cycle** skill takes one ticket from start to finish
and keeps the board in sync while it works:

1. moves the ticket to **In Progress**,
2. writes an analysis and an implementation plan into `docs/analyses/`,
3. implements it,
4. moves the ticket to **Test** and runs the project's tests, staying there
   until they pass,
5. commits (merging the worktree branch first, if there is one), moves the
   ticket to **Done**, then deletes the ticket's worktree and clears its
   `worktree_path`.

It knows which ticket it is working on because a chat linked to a ticket says
so in its system prompt. Column ids come from `get_kanban_board` and are
matched by title, so renamed columns are reported rather than guessed, and a
failed step leaves the ticket where it is with the reason in a ticket comment.

If you have a **column prompt** configured for one of those columns, moving the
ticket there fires it — the cycle would then run alongside whatever that prompt
starts. Use one or the other for a given column.

## Using Skills

### In Chat

1. Open a chat session
2. Click the **Skill** dropdown above the message input
3. Select a skill
4. Type your message and send

The skill content is sent as a system prompt via `--append-system-prompt`.

### In Kanban Tickets

Assign a skill to a ticket in the ticket dialog. When the ticket is run, the skill provides context for Claude.

### In Scheduler Tasks

Select a skill when creating a scheduler task. The skill provides context for each scheduled execution.

### In Column Prompts

In Settings, assign a skill to a Kanban column prompt. When a ticket moves to that column, the skill is used as context.
