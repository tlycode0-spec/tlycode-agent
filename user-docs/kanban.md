# Kanban Board

The Kanban board helps you track tasks and organize your work.

![Kanban Board](./images/kanban-view.png)

## Boards

A project can have several boards — say one for features, one for bugs and one
for the cloud edition. Each board has its own columns and tickets, its own
**archive**, its own **flows** and its own **agent profile**. The **backlog** is
shared by all boards of the project.

- Pick the board in the selector at the left of the board toolbar. The last
  board you looked at is remembered per project.
- The ⋮ next to the selector creates a board (with the default columns, or with
  the same column titles as the current board), renames it, moves it left or
  right, or deletes it.
- A board can only be deleted once it is empty — no tickets on it, in its
  archive or in the backlog — and while none of its flows is running. The
  project's last board cannot be deleted.
- The first board in the list is the **default board**. Agents and integrations
  that do not name a board work with it.

Search looks through the board on screen.

### Moving a ticket to another board

On a card, open the move menu and choose **Move to board…**; in the ticket
dialog it is in the ⋯ menu. Pick the board and the column. The ticket keeps its
chat, comments, attachments and worktree. It leaves the flows of its old board;
while a flow is working on it, the move is refused — wait for the turn to end
or stop the flow. The target column's prompt runs with the **target** board's
agent profile.

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

The ticket dialog is laid out like a Jira issue: the ticket's content on the
left, its status and parameters on the right. Changes save on their own — there
is no Save button; whatever is still being typed is saved when the dialog
closes.

**Header** — the column the ticket is in, the **⋯** menu (*Move to backlog*,
*Archive*, *Delete* — deleting asks for confirmation) and **✕** to close.

**Left column** (scrolls on its own):

- **Title** — shown as a heading; click it to edit, Enter confirms, Escape restores it.
- **Description** — rendered as markdown, the same way the chat renders it (tables, code, mermaid diagrams). Click the text or the pencil to edit the raw markdown; **Done**, Escape or clicking away goes back to the preview.
- **Attachments** — **Add** picks files; you can also drop files anywhere on the left column or paste them (e.g. a screenshot) from the clipboard.
- **Comments** — at the very bottom: a box for a new comment (Ctrl/Cmd+Enter sends), then the comments, newest first, rendered as markdown.

**Right column:**

- **Status** — the ticket's column as a coloured dropdown (grey for the first column, green for the last, blue in between). Picking another column moves the ticket there, with the same column prompt as a move on the board. While the agent is responding in the linked session the move is refused.
- **Run** — starts the ticket in a new chat session and opens it, see *Running a Ticket*.
- **Open chat** — opens the linked chat session.
- **Details** (collapsible; the dialog remembers whether it is collapsed) — one `label | value` row per parameter; an empty value shows a grey *Add…*, click it to fill it in:
  - **Persona / skill** — attach a persona or a skill to use as context when running the ticket
  - **Worktree** — the git worktree the ticket's session works in; **+** creates a new one
  - **Chat session** — the linked session, or link an existing one / **+** create a new one
  - **Custom fields** — metadata on the ticket, e.g. a Jira ticket number. Two kinds sit side by side:
    - fields the project defines for every ticket (the `ticket_fields` project setting, see [Settings](./settings.md#kanban)), listed with their configured name; whether they reach the card is the project's switch
    - fields you add here with **Add field**, which belong to this ticket alone: click the name and the value to fill them in, flip the switch to put it on the card, and remove it again with the bin. The name is the field's identity, so renaming it carries the value along. After a save they are listed alphabetically by name.

    Fields computed from the linked session (see *Session chips* below) are not listed here — there is nothing to type in.
- Below the panel: when the ticket was created and last updated, and whether it is in the backlog, archived, or has a worktree a flow created for it.

On a narrow window the right column moves under the content and the whole
dialog scrolls as one.

**New ticket** — the same layout with the title and description open for
typing. Attachments, comments and the chat session appear once the ticket
exists. The status picks the column the ticket is created in. **Create** (or
Ctrl/Cmd+Enter) creates it.

### Session chips

A card can also carry two chips computed from its linked chat session, so the numbers that decide whether to work in that session are visible at a glance instead of being typed into the description:

- **tokens in context** — the input side of the session's last turn. Highlighted once it passes the project's threshold, which is where rotating the session into a fresh one starts to pay off.
- **session hotness** — `until 14:32 (12 min)` while the prompt cache is still hot, `cold` afterwards. Waking a cold session re-reads the whole context, so a cold chip is a warning that the next turn is expensive.

Both are off until the project defines a ticket field whose source is the session (`session_tokens` or `session_hot` in `ticket_fields`), together with the two thresholds `session_hot_minutes` and `session_context_warn_tokens` — there is no screen for any of it, see [Settings](./settings.md#kanban). The chips refresh when a turn finishes; the hotness countdown ticks on its own.

### Running a Ticket

When a ticket has a description or system prompt, you can run it directly — **Run** in the ticket dialog (or the `run_ticket` MCP tool) creates a new chat session linked to the ticket, sends the description as the first prompt with the ticket's persona or skill, and opens the chat. Run is off while the agent is responding in the ticket's linked session.

## Search

Type into the search box on the board toolbar (or press **Ctrl+F** / **⌘F** to
jump there from anywhere on the board, even from another text field) and the cards filter as you type —
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
and backlogged tickets have their own search in their own views; Ctrl+F / ⌘F
jumps into that one there. **Esc** in a search box clears the board search and
puts the cursor back where it was.

While a search is on, cards cannot be dragged: a position inside a filtered
column is not the card's position in the real column, so a drop would reorder
the column wrongly. Use the move icon on the card instead — it puts the
card at the top of the column you choose.

## Backlog

Click the **Backlog** button to manage tickets that aren't on the board yet. Backlog supports search and pagination. Tickets can be moved between backlog and the board.

The backlog is shared by every board of the project: it lists the backlog tickets of all of them, each with its board and column. **Move to board** asks which board and column the ticket goes to (with a single board it goes straight back to the column it came from).

## Archive

Click **Archived** to view and restore archived tickets. Archive also supports search and pagination. When a ticket is archived, its linked chat session is also archived.

The archive belongs to the board: it shows the archived tickets of the board on screen, and restoring puts a ticket back into the column it came from.

## Bulk actions

**A whole column.** The column menu (⋮ next to its title) has **All to
backlog** and **All to archive**. They take every card the column shows on the
board — including cards further down that haven't been scrolled into view yet —
after you confirm the count. Cards already in the backlog stay where they are.
While the board is searched both items are disabled: the column then shows only
the matches, and "all" would mean more than you see.

**Picked tickets.** In the backlog and the archive every ticket has a
checkbox, and the checkbox in the table header (on a phone, the one above the
list) picks the whole page. The toolbar above the list then offers:

- Archive: **Back to board**.
- Backlog: **Move to board** and **Archive**.

"Select all" means the current page (20 tickets). The selection is cleared when
you change the page, the search or the view.

**Back to board** returns a ticket to the board even when it was archived out of
the backlog, and unarchives its chat. A ticket goes back to the column and place
it left.

**A running agent.** Archiving skips a ticket whose agent is answering right
now and tells you how many were skipped; archive it once the agent is done.
Tickets moved to the backlog or archive leave their flows the same way a single
ticket does (see [Flow Manager](#flow-manager)).

## Flow Manager

Click **Flow manager** on the board toolbar to run a batch of tickets in a
planned order. Flows belong to the board on screen and take only its tickets.

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

You do not have to stop a run to rework the steps it has not got to yet. While
the flow runs, a step that is still **Waiting** can have its settings changed,
be deleted, or be dragged to another place among the other waiting steps — the
run reads a step's settings when it gets there. You can also add new steps and
add, remove or move tickets in a waiting step. A step that is running or has
finished is locked until the run stops: its settings are read-only, it cannot
be deleted, and it keeps its place in the order, so no waiting step can be put
in front of it. Renaming always works. Delete the last step that is still
waiting and the run ends once the current one is done.

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

In a big run, use the **Show:** row under the run controls to see only the
tickets in some states — say **Failed** and **Waiting for you**. Each chip says
how many tickets of the whole run are in that state; click one or more to
filter, **Clear filter** to see everything again. The filter only hides cards:
the step and flow progress still count every ticket, and as tickets change
state during the run they appear and disappear on their own. A step with no
matching ticket is dimmed but stays where it is, so you can still drop tickets
into it — a ticket dropped between visible cards lands in the right place among
the hidden ones too. A ticket just added to a step is **Waiting**, so with a
filter that leaves that state out it disappears from view until you clear it.
Each flow remembers its own filter.

The **Board tickets** list on the right has a filter of its own: one chip per
board column, with how many tickets the column holds. Pick one or more columns
— say **To Do** — so finished tickets do not bury the ones still to plan;
**Clear filter** lists every column again. The search works within the chosen
columns, and the project remembers the choice.

## Agent Profile

Click **Agent profile** on the board toolbar — the third view next to the board
and the flow manager — to set what the agents of this project are told.

An **agent profile** is a named set of prompts: the first-message prompt, the
prompt appended to every message, the prompt that runs before a session is
deleted, and a prompt per board column. Pick one from the selector at the top
and its values are written into the project's settings, so the whole project
switches at once.

Each board can also run under a profile of its own. The **Profile of board …**
selector above the profile picks it: **Same as the project** (the default)
keeps the board on the project's profile; picking a profile makes the board's
tickets — their column prompts, first message, appended prompt and the prompt
before deleting their chat — use that profile instead. While a board has its
own profile, the edits on this screen change that profile only and leave the
project's settings alone. A flow step with a profile still wins over the
board's. The built-in **Default** profile is read only; make your own
with **+** and delete it with the bin.

Profiles belong to the project. The selector lists only this project's
profiles and **Default**, two projects may each have a profile of the same
name, and creating, editing or deleting one here leaves every other project
alone. To reuse a profile from another project, click the copy button next to
**+**, pick the project and the profile, and optionally give it a new name —
the copy is this project's own from then on. Profiles that were shared by all
projects before profiles became per project are offered there as **Shared
profiles from before**; a project that was using one of them as its active
profile, or in a flow step, got its own copy automatically.

These prompts apply to every turn, whoever started it — a message you type, or
a ticket the flow manager dispatches. The first-message prompt, together with
the skill or persona set next to it, goes to a chat session nothing has been
said in yet: a ticket whose chat is still empty opens with it on its first run
and not again afterwards. A skill or persona set on the ticket itself is an
explicit choice and keeps precedence over the profile's.

A flow step can run under an agent profile of its own, one of the project's —
see the step's settings in the flow manager. While the step works on a ticket, that profile replaces
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
