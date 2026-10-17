# Kanban Board

The Kanban board helps you track tasks and organize your work.

![Kanban Board](./images/kanban-view.png)

## Columns

By default, three columns are created: **To Do**, **In Progress**, **Done**.

- Click **Add Column** to create a new column
- Click a column title to rename it
- Drag columns to reorder them
- Delete a column via its menu (all tickets in it will be removed)

## Tickets

- Click **Add ticket** (+ button) on a column to create a new ticket
- A card's actions are icons — hover one for its name. The pencil opens the
  ticket dialog, the speech bubble jumps to the linked chat session (it spins
  while that session's agent is answering), and the arrow-into-a-box is the
  **Move** menu described below.
- **Drag and drop** tickets between columns to change their status — grab the
  card anywhere, not only by its grip. With a finger the grip is the one place
  that drags, so that a swipe still scrolls the column; the card's move icon
  does the same thing in two taps.

### Ordering cards in a column

The order of the cards inside a column is their priority, and it is kept — drag
a card up or down and it stays where you dropped it.

Dragging stops being practical in a long column, where the other end is off
screen. The move icon on the card does the same thing in two clicks:

- **To top of column** / **To bottom of column** — reorder inside the card's
  own column
- **To column** — hand the card over to another column, at its top

While a search is on, only *to top* is offered: the end of a filtered column is
not the end of the real one.

An agent can order a whole column in one go — see the built-in **Backlog
ordering** skill, which takes the priorities from a backlog (the Jira ranking,
for instance) and writes that order onto the board.

### Ticket Dialog

![Ticket Dialog](./images/kanban-ticket-dialog.png)

The ticket dialog lets you edit all ticket properties:

- **Title** — the ticket name
- **Description** — detailed description of the task (markdown supported)
- **System Prompt** — a custom system prompt that will be sent to Claude when running this ticket
- **Skill** — attach a skill to use as context when running the ticket
- **Custom fields** — metadata on the ticket, e.g. a Jira ticket number. Two kinds sit side by side:
  - fields the project defines for every ticket (the `ticket_fields` project setting, see [Settings](./settings.md#kanban)), listed with their configured name; whether they reach the card is the project's switch
  - fields you add here with **Add field**, which belong to this ticket alone: type a name and a value, flip the switch to put it on the card, and remove it again with the bin. The name is the field's identity, so renaming it carries the value along. After a save they are listed alphabetically by name.

  Fields computed from the linked session (see *Session chips* below) are not listed here — there is nothing to type in.
- **Attachments** — upload file attachments to the ticket (stored as base64)
- **Chat Session** — link to an existing chat session or create a new one
- **Comments** — add comments to the ticket for discussion or notes

### Session chips

A card can also carry two chips computed from its linked chat session, so the numbers that decide whether to work in that session are visible at a glance instead of being typed into the description:

- **tokens in context** — the input side of the session's last turn. Highlighted once it passes the project's threshold, which is where rotating the session into a fresh one starts to pay off.
- **session hotness** — `until 14:32 (12 min)` while the prompt cache is still hot, `cold` afterwards. Waking a cold session re-reads the whole context, so a cold chip is a warning that the next turn is expensive.

Both are off until the project defines a ticket field whose source is the session (`session_tokens` or `session_hot` in `ticket_fields`), together with the two thresholds `session_hot_minutes` and `session_context_warn_tokens` — there is no screen for any of it, see [Settings](./settings.md#kanban). The chips refresh when a turn finishes; the hotness countdown ticks on its own.

### Running a Ticket

When a ticket has a description, system prompt, or skill, you can run it directly — this creates a new chat session with the ticket's context and sends the description as a prompt to Claude.

## Search

Type into the search box on the board toolbar and the cards filter as you type —
you stay on the board, no switching to a list. Each column then shows only its
matches, and the number next to the column title is the number of matches in it.
Press **Esc** or click the ✕ to go back to the whole board.

The sliders button next to the box picks where to look. The three choices are
independent and a card matches if any of them does:

- **Card title**
- **Card description** — handy if you keep notes like "tested Oct 12" there
- **Session title** — the name of the chat session linked to the card

Title and description are on by default, session title is off; the choice is
remembered. At least one has to stay on.

Searching looks for cards whose text *contains* what you typed, ignoring case,
and the whole board is searched — not just the cards already loaded. Archived
and backlogged tickets have their own search in their own views.

While a search is on, cards cannot be dragged: a position inside a filtered
column is not the card's position in the real column, so a drop would reorder
the column wrongly. Use the move icon on the card instead — it puts the
card at the top of the column you choose.

## Backlog

Click the **Backlog** button to manage tickets that aren't on the board yet. Backlog supports search and pagination. Tickets can be moved between backlog and the board.

## Archive

Click **Archived** to view and restore archived tickets. Archive also supports search and pagination. When a ticket is archived, its linked chat session is also archived.

## Flow Manager

Click **Flow manager** on the board toolbar to run a batch of tickets in a
planned order.

A **flow** is a list of named **steps**. Drag tickets from the board panel into
a step, reorder them inside it, or drag them to another step. Steps run in
order: the next one starts only once every ticket of the current one has
finished.

Each step decides how its tickets run:

- **Sequential** — one ticket at a time, in the order you arranged them.
  **Parallel** — all of them at once (optionally capped by *Max at once*).
- **Own session** — every ticket runs in its own chat session.
  **One session** — the whole step runs in a single shared chat session, so the
  agent keeps the context of the previous tickets. Only available for a
  sequential step, because one session handles one turn at a time.
- **Done when the ticket reaches…** — by default a ticket is finished when its
  first agent run ends. Name a board column instead and the step waits until
  the ticket lands there, which is how a board whose columns send prompts of
  their own runs a ticket through several turns. Landing in the column is not
  by itself the end: the ticket counts as finished once it is there *and* its
  chat has gone quiet — no agent working and no prompt waiting its turn. The
  target column's own prompt is queued by the very move that brings the ticket
  there, so without that wait a ticket would be called done and then worked on. The picker needs **Column
  prompts** switched on in the [agent profile](#agent-profile) — with them off
  nothing would move the ticket on and the step would wait forever — so it is
  greyed out until they are.

Use **Start**, **Stop** and **Reset** to control the run:

- **Start** begins the flow, or resumes it after a Stop at the first unfinished
  ticket.
- **Stop** cancels the turns that are in flight and dispatches nothing more;
  those tickets go back to waiting.
- **Reset** puts every ticket back to waiting. The chat sessions and their
  messages are kept, so the record of what the flow did stays in Chat.

Rename a flow from the ⋮ on its row; rename a step by clicking its title in the
step header and typing over it (Enter saves, Escape puts the old one back).

Start, Stop and Reset are the whole run; a single ticket has a ⋮ of its own on
its card:

- **Mark as done** — the ticket counts as finished without being run, for work
  you did yourself. A ticket the agent is working on right now has its turn
  cancelled first.
- **Take out of the run** — the run passes the ticket by, the way it passes by
  one that was archived off the board. The rest of the step carries on.
- **Put back into the run** — the ticket goes back to waiting and the run picks
  it up again. A step and a flow that had already called themselves finished
  reopen for it, so press **Start** if the run was over.

The flow's own menu (the ⋮ on its row) also has **Log** — what the run did, newest
first: when it was started, stopped or reset, which ticket went to the agent
and into which chat session, and what failed. A reset is logged too, so the
counters going back to zero is never a mystery. Each flow keeps its last 500
entries.

If the agent runs out of tokens mid-run, the flow does not fail: it parks as
**Waiting for the limit**, the ticket goes back to waiting, and a one-off task
appears in the **Scheduler** that starts the flow again when the limit should
have reset. You can postpone, disable or delete that task, or press Start
yourself — doing so removes the scheduled one. After five waits with no
success the flow gives up and fails.

Moving a ticket to the **backlog** or **archive** takes it out of every flow it
was in — a plan made of tickets that are not on the board is one you cannot
reason about. Bringing the ticket back does not put it into the flow again.
A ticket the agent is working on right now is the exception: it stays until
that run finishes, so archiving never cancels work silently.

Every ticket and every step shows its state — waiting, working, done, failed or
skipped (an archived or deleted ticket is skipped). A failed ticket stops the
flow unless you turn that off for the flow. The flow never moves tickets
between columns, so your board stays as you left it.

A step that waits for a column has one more state: **Waiting for you**. The
agent has finished its turn and the ticket has not reached the column, so
nothing is running and the run cannot move until somebody puts it there — you,
by dragging the card, or the agent from its own chat. The chip names the column
it is waiting for. The ticket still holds its place in the step, so a sequential
step will not start the next one behind its back.

## Agent Profile

Click **Agent profile** on the board toolbar — the third view next to the board
and the flow manager — to set what the agents of this project are told.

An **agent profile** is a named set of prompts: the first-message prompt, the
prompt appended to every message, the prompt that runs before a session is
deleted, and a prompt per board column. Pick one from the selector at the top
and its values are written into the project's settings, so the whole project
switches at once. The built-in **Default** profile is read only; make your own
with **+** and delete it with the bin.

These prompts apply to every turn, whoever started it — a message you type, or
a ticket the flow manager dispatches. The first-message prompt, together with
the skill or persona set next to it, goes to a chat session nothing has been
said in yet: a ticket whose chat is still empty opens with it on its first run
and not again afterwards. A skill or persona set on the ticket itself is an
explicit choice and keeps precedence over the profile's.

A flow step can run under an agent profile of its own — see the step's settings
in the flow manager. While the step works on a ticket, that profile replaces
the project's prompts for it, the first-message prompt included.

### Column Prompts

The **Column prompts** section of the agent profile configures a prompt and skill for each column. When a ticket is moved to a column that has a prompt configured, the prompt is automatically sent to the ticket's linked chat session.

A column may configure only a skill or persona and no words of its own. The
first time — into a session nothing has been said in — the ticket's own title
and description stand in for the missing prompt, so the agent knows what it is
looking at. After that the session already has the ticket, so later columns
send the skill alone: repeating the ticket on every move would bury the
instruction that matters under context the agent read turns ago. The chat still
shows the chip naming the skill that ran.

A move that sends nothing says why, so it never looks like a broken trigger:
the switch is off, the column has nothing configured, or the ticket has no chat
session. The one worth knowing about: while a flow step with an agent profile
of its own works on a ticket, that profile's column prompts **replace** the
project's — so a board that looks configured can still send nothing, and the
message names the profile responsible.

The prompt is written into the chat as a message from you, with the chip
naming the skill or persona it ran with — a turn nobody typed would otherwise
show up as an answer to nothing. If the session is still empty, the prompt also
carries the profile's first-message opening.

While the agent is responding in a ticket's chat session, that ticket cannot be moved into a column whose prompt would then land in the running session — the board says so and leaves the card where it was. Every other move goes through: reordering inside a column, and moving into a column that has no prompt configured. The switch next to **Column prompts** turns the prompts off altogether, and with them this restriction.

## Cross-Navigation with Chat

- Click the **chat link** on a ticket to navigate directly to its linked chat session
- From a chat session, the linked ticket is shown at the top with a link back to the Kanban view
