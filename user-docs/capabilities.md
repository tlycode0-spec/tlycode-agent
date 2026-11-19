# Capabilities

A **capability** is a named list of steps kept in the project's library — "read
the file and summarize it", "analyze → change → test". You write it once and run
it in any chat, put it into a ticket's prompt queue, or let a step of another
capability run it.

Open the library with **Capabilities** in the top bar (Cmd/Ctrl+9). The section
has three tabs: **Capabilities**, and the **Apps** and **Scripts** an app or a
script step hands its answer to (see [Apps and scripts](#apps-and-scripts)).

## The library

- **New** — the **+** next to the search, type a name, Enter. Names are unique
  within the project.
- **Search** — finds names, descriptions and the text of the steps.
- **Order** — the arrows next to a capability move it up or down.
- **Edit** — pick a capability: name, description, the switch (a disabled
  capability stays in the library but nothing runs it) and its steps.
- **Delete** — the bin. When tickets or other capabilities run it, the dialog
  names them; their steps stay, and a run reports the capability as missing.

A capability without an enabled step is a **draft**: nothing can run it.

## Steps

The steps are edited like a ticket's prompt queue — the same editor, in the
modal of a ticket and in the library. Type into *Add a step…* (Markdown,
Ctrl+Enter) for a prompt step, click a step to edit it, switch it off, drag to
reorder, delete with the bin.

**Add step ▾** next to the field picks a step by its **kind**: *Prompt*,
*Capability*, *Condition*, *For each item*, *While*, *App*, *Script*. A kind
that needs something the project lacks (a capability that can run, an app, a
script) is greyed out and says what is missing. The kind's capability, app or
script is picked right after; whatever you typed into the field becomes the
step's prompt, question, what to list or what to hand the program.

Every step shows its kind as a chip. **Click the chip to change the kind**: the
step keeps its text and gets what the new kind needs (a condition a yes/no
test, a while a number test, a capability, app or script picked from a menu).
A *Capability* step has no text of its own, so turning a step into one asks
first. Under the text only the fields of the step's kind show: a prompt its
skill or persona, a condition its test and branches, a loop its body and cap,
an app or a script its program.

A step that saved fine but **cannot run** — the capability it runs was deleted,
switched off or has no enabled step, its app or script is gone — is marked
**Draft**, the tooltip says why, and a note above the list counts them. A run
stops at such a step.

**Show answer** (prompt, app and script steps) — a run folds into one group in
the chat and only its last answer shows (see
[Runs in the chat](#runs-in-the-chat)). Switch it on for a step whose answer
you want to see on its own too, a summary halfway through, say.

**Sees** on a prompt step says what the step works with:

- **The whole session** (the default) — the conversation of the chat so far.
- **The previous answer** — only the answer of the step sent before it,
  as *Previous step answered: …* (cut in the middle beyond 8 000 characters).
  The first step of a run is told there was none.
- **Nothing** — only its task.

A step that doesn't see the session gets its task in front of it: the ticket
and its opening message as the Run window composes it, or the capability and
its description.

**Remember** (only for a step that sees the session) says whether the step
stays in the chat's memory. Switched off, the agent runs the step on a branch
of the conversation and the chat goes on as if it hadn't happened. A step that
doesn't see the session never stays in memory. Such steps show in the chat
marked **out of memory**. How each agent does it:

| | LM Studio | Claude, Codex, OpenCode | Gemini, Copilot |
|---|---|---|---|
| Whole session, remembered | as any message | `--resume` | `--resume` |
| Whole session, not remembered | goes on from the chat's response chain, which stays where it was | the session is **forked** (Claude `--fork-session`, Codex `exec fork`, OpenCode `--fork`), the fork is dropped | cannot fork: the step **stays in memory** (the switch is greyed out on a ticket with such an agent) |
| Previous answer / nothing | a new chain with nothing of the chat replayed | a new session without the chat's recap, dropped afterwards | the same |

For a small local model (a Gemma with a 4–16k context) *The previous answer*
and Remember off on the working steps keep the chat short: only what you
want the model to build on stays in it.

A **Capability** step **runs another capability**. It
refers to it rather than copying it: the run looks the capability up only when
it gets to that step, so whatever you changed up to then is what runs. Click
the chip of such a step to point it at another capability. A capability cannot
run itself.

## Conditions

A **Condition** step **decides which capability runs**. The model is asked for one value only — a number, a yes or no, or a
short word — and **the app** compares it. A small model is unreliable when it
has to judge "is this worth a review?", but it reads "how many files changed?"
off the material in front of it reliably, and `17 > 3` is then arithmetic, not
an opinion. Whatever you typed into *Add a step…* becomes the question;
otherwise edit the placeholder. Then pick the capability for **If yes**.

Under the question:

- **Variable** — the name the value is kept under for the rest of the run
  (letters, digits, `_`; empty = `value`).
- **Value kind** — *yes / no* (Czech or English, whole words: "nevím" is not
  a "ne"), *number* (the first number in the answer, decimal comma or point)
  or *text* (the first line, without quotes and punctuation, at most 200
  characters).
- **Operator** — is, is not, contains, is greater than, is less than, is at
  least, is at most, is empty, is not empty. Text compares ignoring case,
  numbers by their value (`17` is `17.0`).
- **Compared value** — what the value is compared with.
- **If yes** / **Otherwise** — the capability each way runs. Without an
  *Otherwise* the run goes on with the next step.

The chat gets one line saying how it went:
`◆ files = „5“ · files is greater than „3“ → true · running „Review“`. A value
that cannot be read, or a number compared with something that is not one, is
**not a "no"**: the run takes the *Otherwise* branch and the line says
`ERROR: …` with the start of the answer.

**Variables.** Write `{files}` into any later step, question or compared value
and the run puts the value in. An unknown name stays as it is. Only the steps'
own text is filled in, not the ticket's opening or the run's note.

**Who answers.** The question never stays in the chat's memory, and **Sees**
says what it sees:

| | LM Studio | Claude, Codex, OpenCode | Gemini, Copilot |
|---|---|---|---|
| Whole session | a short helper call outside the chat's response chain: temperature 0, no tools, the newest messages that fit an eighth of the window | a turn on a **fork** of the session; the agent may use its tools to find the value | a turn in the session itself (it cannot fork), so the question stays in memory |
| Previous answer / nothing | a helper call with that alone | a turn without the chat | the same |

The LM Studio helper call has no tools: to measure something (failing tests,
changed files) put a prompt step in front that finds it, and give the condition
**Sees: The previous answer**. A queue with a condition has to run one step per
turn; a session shared by several tickets cannot run it.

## Loops

Two more kinds of step repeat a capability. Both have a **cap**, the most
items or passes they run, and **Show every item**: the folded run in the chat
shows the answer of every item's last step rather than none of them.

**For each** asks the model for a **list** — files, tickets, points — and runs
the chosen capability for every item, at most the cap (3, 5, 10 or 20; 20 by
default). Whatever you typed into *Add a step…* says what to list. The answer
is read forgivingly: a JSON array (even with words around it or an array of
objects), otherwise one item per line (bullets and numbering dropped), or one
line of commas. The same item twice runs once. Before anything runs the chat
shows the list, and how many items the cap left out:

```
↻ 3 items:
- api.py
- main.py
- notes.md

[2 more were listed and not run: this step's cap is 3]
```

Every step of the capability is told its item (*This is one item of a list,
and the item is: „api.py“*) whatever it sees, and `{item}` puts it into the
step's text. A step that sees *The previous answer* gets the item as the
answer before it. **Items are blind to each other**: every item starts from
where the chat was before the loop, so twenty items don't fill a small model's
window with the work of the nineteen before. The items' messages show in the
chat, marked **out of memory**; the chat's agent session goes back to where it
was after each item:

| LM Studio | Claude, Codex, OpenCode | Gemini, Copilot |
|---|---|---|
| goes on from the chat's response chain as it was before the loop | the item's first turn **forks** the session as it was before the loop, its later steps go on in the fork | cannot fork: the item starts a session of its own, with a recap of the chat before the loop |

**While** asks for a value before every pass, like a condition (variable,
value kind, operator, compared value), and runs the chosen capability as long
as the test holds, at most the cap (3, 5 or 10; 10 by default). Passes build
on each other — that is the point of "repeat while the tests fail". The chat
gets a line for every pass (`↻ Pass 2 of at most 10` and the test), a warning
when the value did not change since the last pass, and one when the cap
stopped a loop whose test still held. A value that cannot be read ends the
loop with `ERROR: …`. The value is a variable of the run and `{item}` too.

**After the loop** the next step gets what the items answered, each under its
heading (`### api.py`, `### Pass 2`): a step that sees *The previous answer*
as that answer, a step that sees the whole session as *The loop before this
step answered: …*. A "save the results" step after a loop saves all of them.

Every question for a list or a value counts against the run's budget of 120
steps, and a loop counts **at its cap** when the tree is checked before a run:
a list and 20 items of a six-step capability (121) is refused before anything
is sent — lower the cap or split the capability. For LM Studio the question
has no tools, as with a condition: for "every file in the diff" put a step in
front that lists them and give the loop *Sees: The previous answer*; a CLI
agent may run `git diff --name-only` itself.

## Apps and scripts

A small model is good at giving data (a JSON list, a few numbers) and bad at
drawing a chart or adding up a column. Two kinds of step leave that to code:

- **App**: the step's answer goes to a **web app** — HTML and JavaScript the
  chat renders in place of the text (a chart, a table, a form). *Full screen*
  opens it over the window; *Input* shows what it was given.
- **Script**: the step's answer goes to a **script** — JavaScript without a
  UI. What the script returns **replaces the step's answer**: the next step
  builds on it, and the chat shows it marked as the script's output.

Both are kept in the **Capabilities** section, on the **Apps** and **Scripts**
tabs: name, description, parameters, code and a sample input. The editor shows
an app's **preview** with the sample input (*Refresh* after an edit) and runs a
script with it (**Run with the sample input**: output, `console.log` lines,
error and time) in the same sandbox a step runs it in.

In a step list, **Add step ▾ → App / Script** adds the step; what you typed in
the field says what to hand the program ("the changed lines of every file as
JSON"), the chip next to it picks the program.

### What the code gets

| | App (HTML) | Script (JavaScript) |
|---|---|---|
| Input | `window.CAPABILITY_INPUT` (text) | `input` (text), also `window.CAPABILITY_INPUT` |
| Result | — (it is drawn) | `output = …` or `window.CAPABILITY_OUTPUT = …`; a string as it is, anything else as JSON, nothing set = empty |
| Where it runs | a sandboxed frame in the chat | the backend (desktop and server alike, with no window open) |
| May | load scripts, styles, fonts and images over https (a chart library from a CDN) | use the language: `JSON`, `Math`, `Date`, `RegExp`… |
| May not | reach the chat, its storage or the app; send data anywhere (`fetch`, forms, WebSocket) | read or write files, use the network, start processes, `import` modules; run longer than **2 s** or take more than **32 MB** |

The input is text: parse it with `JSON.parse` inside a `try`, since a small
model's JSON is not always clean. The app's or script's **description** goes to
the model with the step, so say there what input it expects ("a JSON array of
`{label, value}`").

### Without and with parameters

- **Without parameters** the step runs as a turn of its own (with tools, and
  *Sees* / *Remember* as set), told to answer with the content alone in the
  shape the description asks for. An outer markdown fence (```` ```json ````)
  is taken off.
- **With parameters** (key, label, required, default) the model takes **no
  turn**: it only extracts the values from the step's text, in a question kept
  out of the chat's memory (for LM Studio a helper call), and the input is a
  JSON object `{"city": "Brno", "days": "3"}`. A required one nobody gave stops
  the run.

### Memory, errors

- An app's own answer row stays where the step's *Remember* put it. The row of
  an app with parameters and the script's output row are **out of memory**:
  the step after it gets the output (*Previous step answered* /
  *The script "…" returned*), but a later message in the chat does not.
- A script that throws, runs out of time or memory, a missing required
  parameter, or a program deleted meanwhile **stop the run** with a line in
  the chat. Fix the script and press **Continue**: the step is asked again and
  runs the script as it is now.

## Running a capability in a chat

The **Capabilities** button next to the message box lists the library. Pick one
and its steps go out **one per turn**, each waiting for the end of the previous
turn, like a ticket's queue. Whatever you typed into the message box becomes the
run's **note**: it goes in front of **every** step ("answer in Czech", "only
the `src/` folder"). The bar above the message box shows the step, inside a
nested capability which one and its step, and inside a loop the item or pass;
it stops and continues the run. A run continued inside a loop goes on with the
item it was at; the list is not asked for again.

## Runs in the chat

A run of ten steps with a loop would bury its result in dozens of messages, so
the chat **folds a run into one group**: a header with the capability's (or the
ticket's) name, how far the run is and how many messages it holds. Click it to
unfold the steps — their prompts, the answers, the lines about conditions and
loops (◆, ↻). Outside the group stay:

- the run's **closing answer** — of its last step outside a loop,
- the answers of steps with **Show answer**,
- inside a loop with **Show every item**, the answer of every item's last step.

A run that failed or was stopped shows nothing on its own; its group stays open
while it is the end of the chat, with the error in the header. A running run's
group says live which step it is at (*step 2 / 4*) and inside a loop which item
(*item 3 of 10*); **Stop** and **Continue** are in the bar above the message
box. A message you type while a run goes on splits its group in two — the later
one says *continued*. Searching the chat opens a folded group the hit is in.
Messages of runs from before this version are not folded.

An answer that shows outside the group but came from a step out of the chat's
memory keeps its **out of memory** mark: the agent does not remember it.

## On a ticket

In a ticket's prompt queue, **Add step ▾ → Capability** adds a step that runs a
capability by reference, **Copy** puts its steps in as they are now (nested
references stay references). The **Run** window previews the **first message**
in full and the **steps of the run** as a tree — the queue's steps, a
capability's steps under it, a condition's *yes* / *otherwise*, a loop's body
and cap — with a step that cannot run marked. The token estimate is the first
step's (steps go one at a time), with the largest step below it. On the board
the **Prompts** chip of a queue with conditions or loops carries their icon.

## Guards

A run refuses to:

- run a capability that is already running in it (a **cycle**, A → B → A),
- nest more than **3** capabilities deep,
- run a capability that is **missing** (deleted, another project), **disabled**
  or **empty**,
- hand an answer to an app or a script that is **missing**,
- send more than **120 steps** in one run (nested steps count; a step sent again
  after a failure counts once; a condition counts its question and the longer of
  its branches; a loop counts at its cap, every question included).

The whole tree is checked before a run starts — then nothing is sent at all.
Because a capability can change while a run is going, every nested capability is
checked again when the run gets there: the run stops, a line in the chat says
why, and nothing more is sent. Fix the capability and press **Continue** in the
bar; the run tries that step again.

## Over MCP

Agents use `list_capabilities`, `get_capability`, `save_capability`,
`delete_capability` and `run_capability`; the ticket prompt tools take
`capability` (id or name) for a step that runs one, and `app` / `script` for a
step that hands its answer to one; `condition.revealed` is *Show answer*. Apps and scripts themselves are
`list_programs`, `save_program` and `delete_program`. See
[MCP Server](./mcp-server.md).
