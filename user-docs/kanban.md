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
- Every ticket has a short **key** such as `#42`, shown in grey in the card's
  top right corner, in the ticket dialog's header ("Ticket #42 / To Do") and on
  the cards of the Flow Manager. Click it to copy it. The number counts up per
  project and never changes: moving the ticket between columns or boards,
  archiving it or sending it to the backlog keeps it. Use it when you talk to
  an agent: "move #42 to Test" or "update ticket #42" is enough, the agent's
  tools find the ticket by its key. The full ticket ID (UUID) can be copied
  from the **⋯** menu of the ticket dialog.
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
closes (also when you close the browser tab in server mode; the browser then
asks before leaving). Ctrl/Cmd+S saves right away, but you never need it.

Next to "Ticket #N / column" the header shows where saving stands:
**Saving…**, **Saved** (hover for the time), **Not saved · Try again** when a
save failed (hover for the error; it is retried on its own and nothing you
typed is lost) or **Not saved — empty title** while the title is empty (the
other fields still save).

If somebody else — an agent over MCP, another window — changes the ticket while
the dialog is open, the fields you have not touched update in place. A field
you changed too is never overwritten silently: the header says **Changed
elsewhere meanwhile: …** and lets you **Use theirs** or **Keep mine**.

**Header** — the column the ticket is in, the **⋯** menu (*Move to backlog*,
*Archive*, *Delete* — deleting asks for confirmation) and **✕** to close.

**Left column** (scrolls on its own):

- **Title** — shown as a heading; click it to edit, Enter confirms, Escape restores it.
- **Description** — rendered as markdown, the same way the chat renders it (tables, code, mermaid diagrams). Click the text or the pencil to edit the raw markdown; Escape or clicking away goes back to the preview. There is nothing to confirm — the text saves as you type.
- **Acceptance criteria** — a checklist under the description, see *Acceptance criteria* below.
- **Attachments** — **Add** picks files; you can also drop files anywhere on the left column or paste them (e.g. a screenshot) from the clipboard.
- **Comments** — at the very bottom: a box for a new comment (Ctrl/Cmd+Enter sends), then the comments, newest first, rendered as markdown.

**Right column:**

- **Status** — the ticket's column as a coloured dropdown (grey for the first column, green for the last, blue in between). Picking another column moves the ticket there, with the same column prompt as a move on the board. While the agent is responding in the linked session the move is refused.
- **Run** — sends the ticket to its chat session and opens it, see *Running a Ticket*.
- **New session** — shown while the ticket has no chat session: creates one linked to the ticket and opens it with the ticket's text pre-filled in the message input. Nothing is sent — edit the text and send it yourself.
- **Open chat** — shown instead of **New session** once the ticket has a session: opens it. A ticket has at most one chat session.
- **Details** (collapsible; the dialog remembers whether it is collapsed) — one `label | value` row per parameter; an empty value shows a grey *Add…*, click it to fill it in:
  - **Persona / skill** — attach a persona or a skill to use as context when running the ticket
  - **Worktree** — the git worktree the ticket's session works in; **+** creates a new one
  - **Chat session** — the linked session (unlink it with the broken-link button), or link an existing one
  - **Custom fields** — metadata on the ticket, e.g. a Jira ticket number. Two kinds sit side by side:
    - fields the project defines for every ticket (the `ticket_fields` project setting, see [Settings](./settings.md#kanban)), listed with their configured name; whether they reach the card is the project's switch
    - fields you add here with **Add field**, which belong to this ticket alone: click the name and the value to fill them in, flip the switch to put it on the card, and remove it again with the bin. The name is the field's identity, so renaming it carries the value along. After a save they are listed alphabetically by name.

    Fields computed from the linked session (see *Session chips* below) are not listed here — there is nothing to type in.
- Below the panel: when the ticket was created and last updated, and whether it is in the backlog, archived, or has a worktree a flow created for it.

On a narrow window the right column moves under the content and the whole
dialog scrolls as one.

**New ticket** — the same layout with the title and description open for
typing. Acceptance criteria, attachments, comments and the chat session appear
once the ticket exists. The status picks the column the ticket is created in. **Create** (or
Ctrl/Cmd+Enter) creates it.

### Acceptance criteria

A ticket keeps its acceptance criteria as a checklist of their own, not as
bullet points in the description. Each criterion is one line of text that is
either met or not.

- **Add** — type into *Add a criterion…* under the list and press Enter. The box stays open for the next one. Pasting several lines adds each line as its own criterion; list markers such as `-`, `1.` or `[ ]` are dropped.
- **Edit** — click a criterion's text. Enter saves it and opens a new item right below it, Escape cancels. Saving an emptied text deletes the criterion.
- **Tick off** — the checkbox. Done criteria are struck through. A criterion an agent ticked off shows a small robot icon; hover it, or the checkbox, to see when it was done and from which session.
- **Reorder** — drag the handle on the left of a row.
- **Delete** — the bin on the right of a row.

Every change is saved at once and shows up in all open windows and browsers.
The section heading counts the criteria (`3/5`), and the counter turns green
when all of them are met.

**On the card** — a ticket with criteria shows an **AC 3/5** chip on the board
and on its card in the Flow Manager. The chip turns green once every
criterion is met. A ticket without criteria shows no chip.

**Agents** work with the criteria through MCP (`list_ticket_criteria`,
`set_ticket_criteria`, `add_ticket_criterion`, `update_ticket_criterion`,
`delete_ticket_criterion`, see [MCP server](../mcp-server.md)). The built-in
skills tell the agent to write the criteria during analysis (including
criteria found in an older ticket's description), to tick them off while
implementing and testing, and to explain in a ticket comment any it cannot
meet.

Criteria stay with the ticket when it is archived or moved to the backlog,
and are deleted with it.

### Session chips

A card can also carry two chips computed from its linked chat session, so the numbers that decide whether to work in that session are visible at a glance instead of being typed into the description:

- **tokens in context** — the input side of the session's last turn. Highlighted once it passes the project's threshold, which is where rotating the session into a fresh one starts to pay off.
- **session hotness** — `until 14:32 (12 min)` while the prompt cache is still hot, `cold` afterwards. Waking a cold session re-reads the whole context, so a cold chip is a warning that the next turn is expensive.

Both are off until the project defines a ticket field whose source is the session (`session_tokens` or `session_hot` in `ticket_fields`), together with the two thresholds `session_hot_minutes` and `session_context_warn_tokens` — there is no screen for any of it, see [Settings](./settings.md#kanban). The chips refresh when a turn finishes; the hotness countdown ticks on its own.

### Running a Ticket

When a ticket has a description or system prompt, you can run it directly — **Run** in the ticket dialog (or the `run_ticket` MCP tool) sends the description as a prompt with the ticket's persona or skill into the ticket's chat session and opens the chat. A ticket has at most one session: Run uses the one it is linked to and creates it only when there is none. The prompt appears once in the chat history; it is not left in the message input (a text of your own in the input stays). The project's opening prompt goes with it only into a session nothing has been said in yet. Run is off while the agent is responding in the ticket's session.

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

Typing a ticket key — `#42`, or just `42` — also finds the ticket with that key,
whichever of the choices are on. The ticket list of the Flow Manager finds
tickets by their key the same way.

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
Tickets moved to the backlog leave their flows the same way a single ticket
does; archived tickets stay in them (see [Flow Manager](#flow-manager)).

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
- **Enforce the done column** — with a done column set, the agent may not end
  its turn while the ticket is still in the column the step started it in.
  When it tries — most often it simply forgot to move the card — it is told
  where the ticket is and where the step wants it, and carries on in the same
  turn. It gets at most two such reminders. If the ticket still has not moved
  after that, the ticket fails right away with "did not reach the column … even
  after 2 reminders", instead of waiting an hour for you. A ticket the agent
  moves to any other column is let go: that column's prompt takes over, as it
  would without the switch. Off by default, because some runs are meant to wait
  for you to move the card. Works with **Claude** and **Codex** agents (Codex
  not on Windows); with other agents the switch does nothing.
- **Step check** — a command such as `cargo check`, `cargo test -p x` or
  `npm run build`: the step's Definition of Done. Whenever the agent wants to
  end its turn, the app runs the command itself in the ticket's worktree (the
  project when the ticket has none; on a server with the Docker Runner inside
  the turn's container, where the agent's tools are). It is not part of the
  prompt, so the agent cannot talk its way past it:
  - exit code 0 — the agent may stop (and **Enforce the done column**, when on,
    has its say next),
  - anything else — the agent is sent back with the last ~4 KB of the output
    (stdout and stderr) and told to fix it; when it stops again the check runs
    again,
  - after **Max. failed checks** failures in one run of the ticket (3 by
    default, 1–10) the agent is let go and the ticket fails with "The step
    check failed (…): <last lines of the output>" — even if the agent already
    moved the card to the done column.

  A check may run for 10 minutes; one that runs longer is killed (with
  everything it started) and counts as a failure. Every run of the check goes
  to the run log (**Step check passed** / **Step check failed** with the exit
  code, the attempt and the end of the output). If the check cannot be started
  at all (no such directory, the turn was stopped) the agent is let go without
  counting it. The command runs with the project's environment variables and
  your login shell's `PATH`. It runs with your rights, like any command the
  agent runs — whoever may edit the flow may set it, as with the step's
  prompts. The check only sees the exit code: an agent that edits the build
  script or skips tests can still get past it. Works with **Claude** and
  **Codex** agents (Codex not on Windows); other agents ignore it. Empty = no
  check.
- **Run guardrails** — on for every step unless you switch them off (see
  below).

A run is unattended, usually with every tool call auto-approved, so with
**Claude** and **Codex** the app checks each tool call before it runs and
refuses what an unattended run must not do:

- `git push` and `git tag` (listing tags is fine), and `git reset --hard` onto
  the main branch or outside the ticket's worktree — in the shell as well as
  through the MCP tools (`git_push`, `git_tag`, `git_delete_remote_branch`,
  `git_reset`),
- running the project's deploy scripts (anything in `deploy/`, `run-cloud.sh`),
- moving, editing, archiving, backlogging or deleting **another** ticket than the one the run
  is working on (its id, `#key` and number all count as its own),
- writing a file outside the ticket's worktree, when the ticket has one,
- removing the ticket's own worktree (`git_worktree_remove`).

The agent is told why and carries on without the action; the refusal shows up
in the flow's **Log** as *Guardrail refused a tool*, with the tool, a short
version of what it tried and the rule. In the **default** permission mode a
refused call is never put to you as a permission question. Ordinary chats,
the other agents (Gemini, Copilot, Opencode, LM Studio) and a step with the
guardrails switched off behave as before. A step that has to do one of these
things on purpose — a merge step that removes its own worktree, say — needs
the guardrails switched off in its settings; the step then shows *no
guardrails*. The guardrails catch an agent's mistakes, they are no sandbox: a
file written through the shell, or a script that pushes inside, gets through,
and so does everything when the app cannot be reached.

Use **Start**, **Stop** and **Reset** to control the run:

- **Start** begins the flow, or resumes it after a Stop at the first unfinished
  ticket. A ticket that has already talked to the agent is picked up where it
  is: its chat gets a new message that says which column the ticket is in and
  what the step waits for, followed by the ticket's text, and the turn runs with
  the skill (or persona) of that column's prompt — the phase the board is at,
  not the start again. A ticket that is already in the column the step waits
  for counts as done and nothing is sent.
- **Stop** cancels the turns that are in flight and dispatches nothing more;
  those tickets go back to waiting, and whatever their chats had queued is
  dropped so it cannot run ahead of the ticket when the flow resumes. (Stop in
  a single ticket's chat is different: only that ticket stops, see **Waiting
  for you · stopped** below.)
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

**Archiving a flow.** A finished run you want to keep but not see every day —
Phase 1, Phase 2… — goes away with **Archive flow** in the ⋮ on its row. It
leaves the list of flows and shows up under **Archived (N)** below it, with its
steps, tickets and log intact. An archived flow opens read-only: it has no
Start, Stop or Reset, and its steps and tickets cannot be changed or added to.
**Restore from archive** (in its header, or in the ⋮) brings it back to its
old place in the list. A running flow cannot be archived — stop it first; a
flow waiting for the token limit is stopped by archiving it, and its scheduled
restart goes. A scheduled task that starts an archived flow fails until the
flow is restored. When the manager opens on its own it never picks an archived
flow; it takes the first flow that is not.

Start, Stop and Reset are the whole run; a single ticket has a ⋮ of its own on
its card:

- **Mark as done** — the ticket counts as finished without being run, for work
  you did yourself. A ticket the agent is working on right now has its turn
  cancelled first.
- **Take out of the run** — the run passes the ticket by, the way it passes by
  an archived one. The rest of the step carries on.
- **Put back into the run** — the ticket goes back to waiting and the run picks
  it up again. A step and a flow that had already called themselves finished
  reopen for it, so press **Start** if the run was over. An archived ticket has
  to be restored first (the ✎ on its card opens it, and its ⋮ has **Unarchive**).

The flow's own menu (the ⋮ on its row) also has **Log** — what the run did, newest
first: when it was started, stopped or reset, which ticket went to the agent
and into which chat session, which agent was reminded of the done column (and
how many times), which tool the agent asked permission for and what the answer
was (allowed, denied, or no answer), and what failed. A reset is logged too, so the
counters going back to zero is never a mystery. Each flow keeps its last 500
entries.

If the agent runs out of tokens mid-run, the flow does not fail: it parks as
**Waiting for the limit**, the ticket goes back to waiting, and a one-off task
appears in the **Scheduler** that starts the flow again when the limit should
have reset. You can postpone, disable or delete that task, or press Start
yourself — doing so removes the scheduled one. After five waits with no
success the flow gives up and fails.

With Claude the app does not have to guess from the error text: Claude tells it
why a turn ended on an API error. A spent limit or billing problem parks the
flow as above. An **overloaded API** parks it too, but only for a minute at
first (then 2, 5 and 10), because it is usually back soon; the log says
"Agent's API overloaded" rather than "Out of tokens". A failed login fails the
ticket. Waits for the limit and for an overload count together towards the five
attempts.

Moving a ticket to the **backlog** or to another board takes it out of every
flow it was in — a plan made of tickets that are not on the board is one you
cannot reason about. Bringing the ticket back does not put it into the flow
again. A ticket the agent is working on right now is the exception: it stays
until that run finishes, so a move never cancels work silently.

**Archiving** a ticket keeps it in its flows — from the board, with a bulk
action or by archiving its chat. Tickets are mostly archived once they are
done, and the run keeps the record of what it did with them: the card stays in
its step with its state and an **Archived** label. A ticket the run has not got
to yet is skipped when it does (an archived ticket is not run); restore it and
put it back into the run to have it run after all.

Every ticket and every step shows its state — waiting, working, done, failed or
skipped (an archived or deleted ticket that is still waiting is skipped). A failed ticket stops the
flow unless you turn that off for the flow. The flow never moves tickets
between columns, so your board stays as you left it.

A step that waits for a column has one more state: **Waiting for you**. The
agent has finished its turn and the ticket has not reached the column, so
nothing is running and the run cannot move until somebody puts it there — you,
by dragging the card, or the agent from its own chat. The chip names the column
it is waiting for. The ticket still holds its place in the step, so a sequential
step will not start the next one behind its back. To send the agent on without
typing "continue" into its chat, press **Continue** (▶) on the card — a failed
ticket has it too. The ticket goes back in line and, while the flow runs, gets
the same message as after a Stop: where it is, its text and its column's skill.

If you press **Stop** in the chat of a ticket the flow is running, only that
ticket stops: it says **Waiting for you · stopped** and the other tickets of
the flow keep working. The stopped ticket holds its place in the step, so the
run does not go past it — a sequential step does not start the next ticket and
the next step does not begin — until you decide:

- **Continue** (▶ on the card, or the first entry of its ⋮) sends the agent a
  message in the same chat to carry on from where it was stopped, with the
  ticket's text and its column's skill. The message is in the chat history.
- **Continue (N)** in the run's header continues every ticket you stopped.
- **Skip** or **Mark as done** in the card's ⋮ settle the ticket without
  running it again, and the run goes on.
- Typing into the ticket's chat yourself works too: the card turns to
  **Working**, and when that turn ends the ticket is judged like any other.

While nothing else in the run is working, the run itself says **Waiting for
you · stopped**; otherwise a chip next to its state counts the stopped tickets.
**Stop** in the run's header still stops the whole run, as before.

When the agent runs with the **default** permission mode (Claude asks before
using a tool), a ticket whose agent waits for your approval says **Waiting for
approval**. Answer in the ticket's chat; the card goes back to **Working** and
the run carries on. Without an answer the agent gives up on that tool after
5 minutes and carries on without it. The ticket keeps its place in the step and
counts as running: putting it back in the queue cancels the turn, and it cannot
be moved or removed until the turn ends.

In a big run, use the **Show:** row under the run controls to see only the
tickets in some states — say **Failed**, **Waiting for you** and **Waiting for
approval**. Each chip says
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

## Sync from Jira

A board can follow a Jira instance: issue changes come over to TlyCode and
show up on the board's cards. Either Jira sends a **webhook** on every change,
or TlyCode **polls** the Jira REST API itself with your API token (see
[Webhook or API polling?](#webhook-or-api-polling)). It is set up in
**Settings → Kanban** ([Settings → Kanban](settings.md#kanban)), separately for
each board, and works in the desktop app and in server mode (not in the cloud
edition).

A board with a sync shows a **sync icon** next to the board switcher. Its
tooltip lists the syncs with their method, whether they are on and the last
error; the icon turns orange on an error and grey when every sync is off. A
click opens Settings → Kanban on that board.

A board can have **several syncs**, for example two Jira instances or two
teams with different status mappings feeding one board. Each sync has its own
name, sync method (webhook URL and secret, or Jira connection), switch,
events, default column, status mapping and filters; see
[Several syncs on one board](#several-syncs-on-one-board).

What the sync does, each event behind its own checkbox:

| Jira event | On the board |
|---|---|
| Issue created | A new card in the default column (or the board's first column). The card's description starts with a link to the issue; the issue key goes into the `jira` ticket field. |
| Summary / description changed | The card's title and description are rewritten. Its system prompt, skill, persona and worktree stay. |
| Status changed | The card moves to the column mapped to the new status. Without a mapping, to the column with the same name; without one of those, it stays. |
| Issue deleted | The card is archived (it can be restored from the archive). |
| Comment created | A ticket comment headed with the author and a link to the comment. |
| Comment edited / deleted | That comment is rewritten / deleted from the ticket. |

Only issues the sync created a card for are updated. Deliveries arriving twice
or out of order are ignored.

### Filters

Three filters decide which issues get a card. They add up (an issue has to
pass all that are set):

| Filter | Lets through |
|---|---|
| **Only Jira projects** | Issues of the listed project keys (`LMS, SUP`). Empty = every project. |
| **Only assignees** | Issues assigned to one of the listed users: one per line, as accountId, e-mail, user name or display name. In Jira Cloud the e-mail is often hidden, the **accountId** is reliable (it is in the URL of the user's profile). An unassigned issue does not pass. Empty = anyone. |
| **Sprint** | *Any*, *Active sprint only*, or *Selected sprints only* (sprint names, one per line, in any state). An issue without a sprint passes only *Any*. TlyCode finds Jira's sprint field by itself, in Jira Cloud and Jira Server alike. |

The assignee and sprint filters only decide about **new cards**:

- **Coming into the filter creates the card.** An issue that did not pass when
  it was created and passes later (it was assigned, added to the active sprint)
  gets a card with its next update, provided **New issue** is ticked. The
  card goes to the column of the issue's status (status mapping, then the
  column of the same name), otherwise to the default column.
- **Leaving the filter keeps the card.** It is not archived and keeps
  following the issue (title, status, comments).

For this to work Jira has to send **Issue updated** as well, and the JQL filter
of the webhook must not drop the issues before they come into the TlyCode
filter. Any update of a passing issue without a card creates one, so an issue
that passed all along but was created before the webhook gets its card with its
next change.

### Webhook or API polling?

Each sync picks its **Sync method**:

| | Webhook | API polling |
|---|---|---|
| How | Jira posts every change to TlyCode's URL. | TlyCode asks Jira for changed issues every few minutes. |
| Needs | Admin rights in Jira (to create the webhook) and a URL Jira can reach (tunnel on a desktop with Jira Cloud). | Your own API token (Jira Cloud) or Personal Access Token (Server / Data Center). No admin rights, no public URL. |
| Delay | Seconds. | Up to the interval (1–60 minutes, 5 by default). |

Pick the webhook when you administer Jira and TlyCode is reachable from it;
otherwise API polling. The events, default column, status mapping, filters
and column prompts work the same way in both.

### Setting up API polling

1. **Get a token.**
   - *Jira Cloud*: [Atlassian account → Security → API tokens](https://id.atlassian.com/manage-profile/security/api-tokens)
     → **Create API token**. You also need the e-mail of that account.
   - *Jira Server / Data Center*: your profile → **Personal Access Tokens** →
     **Create token**. No e-mail.

   The token sees what your account sees: the sync gets only issues you may
   browse.
2. **In TlyCode**, open Settings → Kanban, pick the board, **Add sync** (or
   open an existing one) and switch **Sync method** to **API polling**. Fill
   in:
   - **Base URL**: `https://acme.atlassian.net`, or the address of your Jira
     Server including its context path (`https://jira.acme.com/jira`).
   - **E-mail**: Jira Cloud only; leave it empty for Server / Data Center.
   - **API token**: after saving it is never shown again, only "Token saved
     (…abcd)". Type a new one to change it, or **Remove token**.
   - **API version**: v3 for Jira Cloud, v2 for older Jira (Server / Data
     Center always uses v2).
   - **Interval**: how often to ask, in minutes.
   - **JQL** (optional): which issues to fetch, e.g. `project = LMS AND type
     != Epic`. Empty = the **Only Jira projects** and **Sprint** filters
     below make the query. The **Only assignees** filter always applies on
     top of it; an `ORDER BY` is ignored.

   When the project has the [Jira MCP preset](settings.md#mcp-servers) set
   up, **Take from the Jira MCP preset** fills in the URL, e-mail, token and
   version from it.
3. **Test connection** logs in with what the form holds (unsaved changes too;
   an empty token field uses the saved token) and shows who you are logged in
   as and how many issues the query finds (for Jira Cloud sometimes "at
   least N"). A failure says what is wrong: rejected token, missing
   permission, no Jira at the URL, Jira not reachable.
4. Switch the sync on and **Save**. Polling needs the URL and the token; a
   switched-off sync can be saved without them.
5. **Polling status** under the connection shows the last fetch, how many
   issues it changed on the board, the last error and when the next fetch is due. **Sync
   now** fetches right away (save your changes first). The card's header shows
   the last fetch or its error.

The webhook URL, the secret and the Webhook server card do not matter for a
polling sync; the form hides the webhook parts in this mode.

### How API polling behaves

- **Start**: the first fetch only notes the time. Changes from then on come
  over, existing issues do not, just like a new webhook. Changing the URL, the
  JQL or the sync method starts over the same way.
- Each fetch asks for the issues changed since the last one, with two minutes
  of overlap so nothing falls between two fetches; the overlap changes
  nothing on the board.
- **Cards you move or rename stay** until the status or the summary really
  changes in Jira. Fetching an issue again changes nothing and fires no column
  prompt.
- **Comments** follow Jira: new, edited and deleted ones. A deleted comment
  shows up with the next change of its issue.
- **Deleted issues** are checked at most once an hour. A card whose issue is
  gone from Jira gets archived. Jira answers "not found" for an issue the token
  may not see, too, so when every checked issue looks gone, nothing is
  archived and the sync shows an error about the token's permissions.
- **Errors** (rejected token, Jira down, rate limit) show on the sync and do
  not hold up other syncs. After a rate limit the next fetch waits as long as
  Jira asked.
- Polling runs in the desktop app and in server mode, not in the cloud
  edition, which has no Jira sync.

### Setting up a webhook

1. **In TlyCode**, open Settings → Kanban, pick the board and click **Add
   sync**. Name it, switch it on, choose the events, the default column and
   the status mapping, and **Save**. The first save creates the **webhook
   URL**: copy it.
2. **Make the URL reachable from Jira** (see below). Swap the host of the
   copied URL for that address and keep the path (`/webhooks/jira/<token>`).
3. **In Jira** (Jira Cloud: *Settings → System → WebHooks → Create a
   WebHook*; Jira Server / Data Center: *Administration → System →
   WebHooks*):
   - **URL**: the webhook URL from step 2.
   - **Secret** (optional, recommended): any random string; put the same one
     into **Webhook secret (HMAC)** in TlyCode and save. Deliveries without a
     valid signature are then refused with `401`.
   - **Events**: *Issue* → created, updated, deleted; *Comment* → created,
     updated, deleted. Only tick what you also ticked in TlyCode.
   - **JQL filter** (optional), e.g. `project = LMS AND type != Epic`. This is
     the best place to narrow down what comes over; the **Only Jira projects**
     field in TlyCode is a coarse second filter by project key.
4. **Check it**: create or change an issue. The **Webhook server** card in
   Settings → Kanban shows the last delivery (HTTP status, result and the sync
   it went to), and each sync shows its own last delivery or error in its
   header. Jira's webhook page also lists failed deliveries.

### Making the URL reachable

- **Desktop + Jira Cloud.** The listener binds to `127.0.0.1:45893`, which
  Jira Cloud cannot reach. Put a tunnel in front of it, for example:
  - `tailscale funnel 45893`,
  - `cloudflared tunnel --url http://127.0.0.1:45893`,
  - `ngrok http 45893`.

  Then use the tunnel's public address with the same path. The tunnel has to
  run whenever Jira should deliver; deliveries missed meanwhile are not
  replayed.
- **Desktop + Jira Server / Data Center on your network.** Switch the bind to
  `0.0.0.0` in the Webhook server card and use the computer's network address
  (`http://192.168.1.20:45893/webhooks/jira/<token>`). The port is then open to
  the whole network, guarded by the token in the URL and the secret.
- **Server mode.** The URL on the server's **main port** works directly
  wherever the server has a public address (behind a reverse proxy too). The
  listener on port `45893` is an alternative if you want webhooks on a port of
  their own.

**Generate a new token** when the URL may have leaked. The old URL stops
working immediately, so update the webhook in Jira.

### Several syncs on one board

Settings → Kanban lists a board's syncs as collapsible cards. **Add sync**
adds one; nothing is stored until its first **Save**, which also creates its
webhook URL. In Jira, each sync gets a webhook of its own with its own URL.

- A delivery uses only the settings of the sync whose URL it came to: its
  events, default column, status mapping, filters and secret.
- **Each sync keeps its own cards.** The same issue key arriving through two
  syncs, say `ABC-1` from two Jira instances, gives two cards, and an update
  through one sync never touches the other's card.
- So keep the filters of syncs fed by **the same** Jira apart (different
  projects, teams or assignees). An issue that passes the filters of two
  syncs gets a card from each, and an issue reassigned from one team's sync
  to the other's gets a second card there, while the first one stays.
- **Delete sync** (behind a confirmation) stops its webhook URL at once and
  forgets which cards it made. The cards stay on the board but are no longer
  updated from Jira. Remove the webhook in Jira too.
- A board that had a sync before keeps it as the sync named **Jira**, with
  its URL and cards unchanged.

### The sync log

Every sync keeps a log. Open it with **Log** at the top of the sync in
Settings → Kanban. The line next to the button sums up the last 24 hours:
how many polls ran (or deliveries came), how many changed something on the
board and how many failed.

- **Webhook**: one line per delivery Jira sent, with what became of it:
  *Applied* (what changed, e.g. the card moved from *To Do* to *In
  Progress*), *Filtered* and why (project key, sprint / assignee filter, an
  event that is switched off), *Unknown issue* (it has no card), *Sync
  switched off*, *Duplicate / older* (Jira sent it twice or out of order) or
  *Error*. A wrong signature or a body that is not a Jira delivery shows as an
  error too. A delivery with an unknown token cannot be told apart from noise,
  so it is not logged.
- **API polling**: one entry per poll, with the issues it found, what it did
  to them, the new cursor and how long it took, or why it failed (wrong
  token, missing permission, rate limit, timeout). Expand it to see each
  issue that changed a card or failed, with its JQL and cursor under
  **Details**. Polls that find nothing are merged into one line ("12× without
  changes"). Issues fetched again without a change only count in the poll's
  summary.
- Filter by **Errors**, **Applied** or **Filtered**, or by part of an issue
  key. The card name opens the card on the board, the issue key opens the
  issue in Jira.
- **Clear log** deletes the sync's entries. Deleting the sync deletes its log
  too.
- Tokens, secrets and delivery bodies are never logged. The log keeps at most
  1000 entries per sync, for 30 days; change that under **Sync logs** below
  the syncs (on the server only an admin can).

Agents read the same log with the MCP tool `get_jira_sync_log`.

### What it deliberately does not do

- **One way only**: Jira → kanban. Moving a card, editing it or commenting on
  it in TlyCode changes nothing in Jira (an agent with the built-in Jira MCP
  server can do that).
- **No backfill**: existing issues do not come over, only changes from the
  moment the webhook exists (or polling starts).
- Attachments, worklogs, sprints, links between issues, assignee, priority and
  Jira's custom fields are not copied to the card (sprint and assignee only
  filter which issues get one).
- **Column prompts are off by default.** Turned on, every card the sync moves
  fires its column's prompt. A bulk move in Jira can start dozens of agents.

## Cross-Navigation with Chat

- Click the **chat link** on a ticket to navigate directly to its linked chat session
- From a chat session, the linked ticket is shown at the top with a link back to the Kanban view
