# Module 3 Reference Guide
### Claude Code — Desktop App: Sessions, Remote Workflows, and Atomic Shipping

Use this guide alongside the Module 3 videos.

---

## Table of Contents

1. [Claude Code Desktop App](#claude-code-desktop-app)
2. [Practice Project: Parallel Sessions Test](#practice-project-parallel-sessions-test)
3. [Remote Control Feature](#remote-control-feature)
4. [Atomic Shipping and the Logic Surgeon Model](#atomic-shipping-and-the-logic-surgeon-model)

---

## Core Concepts

**The container changes, not the engine.** The desktop app shares `CLAUDE.md`, settings, MCP servers, hooks, plugins, voice configuration, and session history with the CLI.

**The Code tab is the working surface.** Chat has no project file access; Cowork runs background work; Code provides local coding sessions, diffs, terminal access, preview, and parallel branches.

**Parallel sessions are physically isolated.** Each session has its own conversation, branch, and git worktree. Agents do not share uncommitted files or context.

**The contract is the integration point.** Parallel work is reviewed and verified independently against the same contract before branches converge.

**Atomic means reviewable.** Build, verify, understand, and commit the smallest meaningful unit before starting the next.

---

## Claude Code Desktop App

### Move an active CLI session

```text
/desktop
```

This saves the current terminal session, opens it in the desktop app, and exits the CLI on macOS and Windows.

### Code session setup

Configure before sending the first prompt:

- Environment — Local for direct access to the project on your machine
- Project folder — the intended repository, not its parent directory
- Model — choose based on the task and change it when needed
- Permission mode — Ask, Auto accept edits, Plan, or Bypass permissions

### Workspace panes

| Pane | Use |
|---|---|
| Chat | Prompts, slash commands, `@filename` references, Plan Mode |
| Diff | Side-by-side changes; accept, reject, or comment per file/line |
| Terminal | Run project commands inside the app; open with `Ctrl+`` |
| File | Inspect or directly edit files opened from a diff or response |

Cycle transcript views with `Ctrl+O`:

- Normal — collapsed tool activity and full responses
- Verbose — every file read, command, and intermediate action
- Summary — final responses and resulting changes

### CONTRIBUTING.md prompt

> "Add a CONTRIBUTING.md to this project. It should cover: how to set up the development environment from scratch, the commit message convention we use, how to run the test suite and what a passing run looks like, the verify-app command and when to run it, and the rule about not committing with failing tests. Base it on what's already in CLAUDE.md and the project structure."

Inline diff comment:

> "This step should mention running npm install first before npm run dev."

### Accept, verify, and commit

```text
/verify-app
```

```bash
git add CONTRIBUTING.md
git commit -m "docs: add CONTRIBUTING.md"
```

Use inline comments for surgical corrections. Review Code is useful for compile errors, logic errors, security problems, and obvious bugs; it does not replace your contract or visual judgment.

---

## Practice Project: Parallel Sessions Test

### Scaffold the demo repository

```bash
mkdir lesson-16-demo
cd lesson-16-demo
npm init -y
npm install express
mkdir public
git init
```

### Initial app prompt

> "Scaffold a minimal Todo app. Create a server.js with an Express server on port 3000 with three endpoints: GET /tasks returns an array of task objects with id, title, and priority fields with some sample data. POST /tasks adds a new task. DELETE /tasks/:id removes a task. Serve the public/ directory as static files. Create public/index.html with a simple interface that fetches and renders the task list on load, shows each task's title and priority, and has a form to add new tasks with a title input and a priority selector — low, medium, high. Vanilla JS only, no frameworks, no build step."

```bash
node server.js
git add .
git commit -m "init: minimal todo app"
```

### Preview configuration prompt

> "Create .claude/launch.json that tells Claude Code to start the preview by running node server.js with the preview URL set to http://localhost:3000."

Visible preview change:

> "Add colour coding to the task list based on priority. High priority tasks should have a red left border, medium yellow, low green."

```bash
git add .
git commit -m "feat: priority colour coding"
```

### Parallel session layout

Create two sessions against the same repository:

- `Todo — Frontend` on `main`
- `Todo — Backend` on `add-validation`

Open both side by side with `Cmd`-click on Mac or `Ctrl`-click on Windows.

Backend prompt:

> "Add priority field validation to the POST /tasks endpoint. Reject requests where priority is not one of: low, medium, high. Return a 400 with a clear error message."

Frontend prompt:

> "Add a filter bar above the task list with three buttons: All, High, Medium, Low. Clicking a filter shows only tasks with that priority. All is selected by default."

Open a side chat with `Cmd+;`, `Ctrl+;`, or `/btw` when a question should not influence the main working thread.

### Review and commit independently

Backend session:

```bash
git add server.js
git commit -m "feat: priority validation on POST /tasks"
git push -u origin add-validation
```

Frontend session:

```bash
git add public/index.html
git commit -m "feat: priority filter bar"
```

### PR monitoring modes

- Monitor — report CI state without taking action
- Auto-fix — diagnose CI failures and push corrections
- Auto-merge — merge after all checks pass; use only after diff review and with comprehensive CI

### Execution environments

| Environment | Behaviour |
|---|---|
| Local | Runs against local files; best for active review and preview |
| Remote | Runs in Anthropic infrastructure and survives closing the app/laptop |
| SSH | Runs on a remote machine with its environment and hardware |

---

## Remote Control Feature

### Start Remote Control

Fresh terminal session:

```bash
claude --remote-control
```

Multiple concurrent sessions from one server process:

```bash
claude remote-control
```

From an existing desktop, terminal, or extension session:

```text
/rc
```

Scan the QR code or open the displayed URL in the Claude mobile app. The machine running the local session must remain awake and connected.

### Push notifications

```text
/config
```

Enable **Push when Claude decides**, or request one directly:

> "Notify me when the tests finish."

### Mobile demonstration prompt

> "Look at the git log and tell me what's been built in this project over the last week — give me a one-line summary of each commit."

### Choose the right remote workflow

| Workflow | Best for |
|---|---|
| Remote Control | Steering a session already running on your machine |
| Dispatch | Starting new work from your phone through Desktop |
| Channels | Reacting to external messages or CI events |
| Remote environment | Long work that must continue after your local machine sleeps |

Use mobile for status, short decisions, and small corrections. Compose complex plans and review large diffs at the desktop.

---

## Atomic Shipping and the Logic Surgeon Model

### The atomic unit

An atomic unit is the smallest meaningful change that can be:

1. Built completely
2. Checked against the contract
3. Tested without failures
4. Confirmed not to regress earlier work
5. Understood structurally
6. Recorded as one coherent commit

### Verification gate

Before moving on, confirm:

- Request, response, and error shapes match `CONTRACT.md` exactly.
- Tests for the unit pass.
- The full suite still passes.
- You can explain the architecture and locate the change.
- The commit contains only this unit.

### Parallel-agent rule

```text
Agent A verifies against CONTRACT.md ─┐
                                    ├─ Merge only when both sides pass
Agent B verifies against CONTRACT.md ─┘
```

Neither session merges merely because it finished first. The shared contract, not one branch's assumptions, is the synchronization point.

### Logic Surgeon posture

- Understand the architecture and protocols before opening build sessions.
- Know the intervention sequence and completion criteria.
- Stop and revise the plan when unexpected evidence appears.
- Use the agent as an instrument; do not outsource the system model in your head.
- Keep every unverified surface area small enough to review honestly.

### Handoff checklist before the backend project

- Planning and contract files are approved.
- Build sequence is explicit.
- Parallel ownership boundaries are written down.
- Verification commands exist before implementation begins.
- Branches and worktrees have clean baselines.
- The director understands OAuth2, Gmail API sync, token refresh, Pub/Sub, and the intended database boundary well enough to evaluate agent decisions.
