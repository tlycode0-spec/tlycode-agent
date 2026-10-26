# Pipelines

A pipeline chains personas and skills into a sequence of steps the agent works through in one chat session. You build it on a canvas in the **Pipelines** view and start it from the chat (the context picker), or let a **trigger** start it by itself when something happens on the kanban board or in a chat session.

## Building a pipeline

1. Open **Pipelines** and create a pipeline in the sidebar (**+**).
2. Drag blocks from the palette on the left onto the canvas:
   - **Triggers** — what starts the pipeline by itself (see below),
   - **Personas** and **Skills** — the steps.
3. Connect the blocks: drag from the bottom handle of one block to the top of the next one. A block with several outgoing edges branches. **Decision** mode lets the agent pick one branch, **Parallel** mode runs each branch in its own chat session.
4. Click a block to configure it. Every change is saved automatically.

**Automatic mode** in the palette decides whether the agent may ask you questions between steps.

## Triggers

A trigger is a block that is not a step: it starts the pipeline when an event matches its filters. It has only an output, so nothing can lead into it. Connect it to the first step. Two triggers in one pipeline work as OR, and one event starts the pipeline once.

| Trigger | Reacts to |
|---|---|
| **Kanban board** | ticket created, moved, edited, archived / unarchived, sent to / restored from the backlog, comment added |
| **Chat session** | session created, user message, agent finished a turn, agent turn failed, agent waits for you (tool permission or a question), session archived |

A trigger dropped on the canvas is **switched off**. Pick its events and filters first, then switch it on in its dialog. The block on the canvas shows a summary ("Ticket moved → Test · board Main"), whether it is off, the time and result of its last run, and any problem.

### Kanban trigger

- **Events** — at least one.
- **Board** and **Target column**: the column the ticket is in after the change. Columns are stored by id, so renaming a column keeps the filter.
- **Source column**: where the ticket came from. Only for *Ticket moved* and *Ticket restored from backlog*, since other events have no source column.
- **Custom ticket fields**: *is set*, or *equals* one of the given values.
- **Origin** — who made the change: user, agent, Jira, flow, system.

### Chat trigger

- **Events** — at least one.
- **Message text** (only for *User message*): equals / starts with / contains / regex, optionally case sensitive. For example *starts with* `/review`.
- **Session title**, **turn outcome** (completed / cancelled, optionally only when nothing else waits in the queue), **waiting for** (tool permission / an answer).
- **Agent**, **persona / skill**, **linked ticket** (only sessions with / without a ticket, the ticket's board), **origin** (user, scheduler, flow, pipeline, trigger, MCP, column prompt, Remote Control).

### Where the run goes

- **New session** (recommended): every run gets its own chat session, so the steps do not mix with your work.
- **Ticket's session** (kanban) / **Same session** (chat): the steps join that session's queue and wait for the running turn to end. They share the queue with your work, and a failed step drops your waiting messages too.

**Extra instruction** is appended to the event description that the first step receives. The agent gets the ticket's key, title and id, the board and columns, or the session, the last messages and the error.

### Examples

- **Ticket moved to Test → run the tests**: Kanban trigger, event *Ticket moved*, board *Main*, target column *Test*, origin *user*. Connect it to a testing skill.
- **`/review` in a chat → review pipeline**: Chat trigger, event *User message*, message text *starts with* `/review`, target *Same session*.
- **Agent finished in a ticket's session → summary**: Chat trigger, event *Agent finished a turn*, linked ticket *only sessions with a ticket*, *only when nothing waits in the queue*.

## Safeguards against loops

- **One run per ticket / session** at a time. Another event on the same subject is skipped (*already running*).
- **Max. concurrent runs** per trigger (default 3). An event over the limit is skipped (*limit*).
- **A run never starts its own pipeline again.** A change made by the pipeline's agent (moving the ticket, finishing a turn) is skipped (*loop*). A change of a ticket an agent makes over MCP while runs are active counts as caused by them.
- **Chain depth**: trigger → agent → event → another trigger goes at most 3 levels deep (*depth*).
- **Switches**: every trigger has its own on / off switch. **Triggers on** in the palette switches off all triggers of the pipeline at once. Starting the pipeline from the chat always works.
- A **copy** of a pipeline with triggers starts with its triggers switched off, so one event does not start both.

## Column prompts and triggers

A column can also have a **column prompt** in the agent profile (Settings → Prompts). When a kanban trigger watches such a column, the trigger dialog says so: both the column prompt and the pipeline run. With *New session* two agents may work in the ticket's worktree at once. With *Ticket's session* both go to the ticket's queue in no guaranteed order.

## Run history

**Run history** in the palette (or in the pipeline's ⋮ menu in the sidebar) lists the latest runs started by triggers. For each run it shows when it started and how long it took, which trigger started it, the event, the subject (a link to the ticket or the session), a link to the session the run ran in, and the status: running, done, failed, cancelled, interrupted (the app restarted), or skipped with its reason. **Started only** hides the skipped ones. The list and the badges on the canvas update live.

## Notes

- Triggers work on the desktop and in server mode. On a server a run runs as the pipeline's owner, only for tickets and sessions the owner can see.
- Cloud projects do not run triggers. The palette there has no Triggers section.
- The canvas is view-only on a phone. Triggers are configured on the desktop.
