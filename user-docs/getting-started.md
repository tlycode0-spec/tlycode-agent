# Getting Started

## System Check

On first launch, TlyCode Agent checks that required tools are installed:

- **Claude CLI** — the AI engine (or other supported CLI agents)
- **Node.js** — required by Claude CLI
- **GitHub CLI** — for Git authentication (optional)

The system check also detects Gemini CLI, Codex CLI, OpenCode CLI, and Copilot CLI if installed. OpenCode CLI is optional and never blocks the check; sign in with `opencode auth login` in a terminal. If all required tools are present and authenticated, the check passes automatically. Otherwise, you can authenticate directly from the check screen or click **Skip**.

## Selecting a Project

After the system check, you'll see the **Project Selector**:

![Project Selector](./images/project-selector.png)

### Hosting Projects

At the top of the project selector, you can connect to **TlyCode Hosting** via OAuth. After authenticating:

- Browse your hosting projects from the dropdown
- **Create new** hosting projects with a name and description
- **Open** a hosting project locally (clones the repository)
- GitHub CLI authentication is required for deploy operations

See [OAuth & Hosting](./oauth-hosting.md) for details.

### Recent Projects

Below the hosting section, your recently opened projects are listed (up to 10). Click any project to open it immediately. Use the **X** button to remove a project from the list.

### Open Project

Click **Open Project** to browse the file system:

- Use the **breadcrumb navigation** at the top to jump to any parent directory
- Click a **folder** to navigate into it
- Click the **Home** button to go to your home directory
- Type a path directly into the breadcrumb bar
- Click **Select this folder** to open the current directory as your project

## Main Interface

After selecting a project, you'll see the main application interface:

![Main Interface](./images/chat-view.png)

The top toolbar contains:

- **Project name** — click to switch projects
- **Tab navigation** — Chat, Kanban, Git, Docs, Skills, Scheduler
- **Settings** (gear icon) — application configuration
- **Theme** (sun/moon icon) — switch between light, dark, and system theme
- **Report a Bug** — link to GitHub issues
- **New window** (desktop) — opens another app window with the current project

### Multiple Windows (desktop)

You can work in several app windows at once — for example two projects on two monitors, or the kanban in one window and the chat in another:

- **New window** button in the toolbar, **New window** in the project menu, or **Cmd/Ctrl + Shift + N** — opens a window with the current project.
- Right-click a project tab → **Open in new window** — opens a window with that project.

Each window has its own active project, project tabs and view; switching the project in one window doesn't change the others. Chat messages, streaming answers, the kanban and runs update live in every window showing the same project. A message draft typed in one window shows up in another window with the same chat once saved, unless you are typing there too.

Closing a window leaves the others running. The first (main) window is only hidden when closed — the app keeps running in the tray / Dock and **Show** brings back the last used window. Extra windows aren't restored after restarting the app.
