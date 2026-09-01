# Module 2 Reference Guide
### Claude Code — Fundamentals in the Terminal: Commands, Prompts, and Configuration

Use this guide alongside the Module 2 videos.

---

## Table of Contents

1. [Introduction to Claude Code](#introduction-to-claude-code)
2. [Setting Up the Agentic Development Environment](#setting-up-the-agentic-development-environment)
3. [Slash Commands and Modes Steering](#slash-commands-and-modes-steering)
4. [Plan Mode and Memory Architecture](#plan-mode-and-memory-architecture)
5. [Memory in Practice](#memory-in-practice)
6. [Extended Memory: Ogham MCP & Context7](#extended-memory-ogham-mcp--context7)
7. [Practice Repository: Architecture Refactor](#practice-repository-architecture-refactor)
8. [Permissions, YOLO Mode and Autonomous Execution](#permissions-yolo-mode-and-autonomous-execution)
9. [Verification Loops](#verification-loops)
10. [Routines](#routines)
11. ['/Goal' Command](#goal-command)
12. [Plugins](#plugins)
13. [Install a Plugin](#install-a-plugin)
14. [Vercel (CLI vs. MCP)](#vercel-cli-vs-mcp)

---

## Core Concepts

**Claude Code is a scalpel.** It reads the actual repository, makes targeted edits, runs commands, and works with git. The quality of the result depends on Plan Mode, durable context, verification, and atomic shipping.

**Four environments, one engine.** The standalone terminal, IDE extension, IDE-integrated terminal, and desktop app share `CLAUDE.md`, MCP servers, hooks, skills, settings, and conversation history.

**Plan before broad changes.** A plan is approved before implementation. Use clarifying questions to expose assumptions while changes are still cheap.

**Memory has scopes.** Session context supports the current conversation; `CLAUDE.md` and project context files travel with the repository; Ogham carries durable cross-session context; Context7 supplies current library documentation.

**Autonomy requires an escape hatch.** Start from a committed baseline, work on a branch, state exact scope, and define a mechanical verification step.

**Use the lightest effective tool interface.** Prefer a CLI for known operations, a thin CLI wrapper for an API without a CLI, and MCP when discovery or a uniquely structured action justifies its context cost.

---

## Introduction to Claude Code

### Four working environments

| Environment | Reach for it when |
|---|---|
| Standalone terminal | Automation, scripting, CI/CD, quick focused sessions, maximum control |
| VS Code/Cursor extension | Active development, graphical diffs, checkpoints and rewind |
| IDE integrated terminal | You want CLI behaviour plus editor diffs and diagnostics through `/ide` |
| Desktop app | Parallel sessions, visual diff review, preview, PR monitoring, remote work |

### Environment decision rule

- Use the terminal for automation and direct CLI control.
- Use the extension when checkpoints and tight editor integration matter.
- Use the desktop app when coordinating multiple sessions or reviewing broader changes visually.
- Move between surfaces as the task changes; the project context travels with you.

---

## Setting Up the Agentic Development Environment

### Prerequisites and install

```bash
node --version
npm install -g @anthropic-ai/claude-code
claude --version
```

Node.js 18 or newer is required for the npm installation. Install the latest LTS release if needed.

### Start and resume

```bash
claude
claude --resume
```

Exit an interactive session with `/exit` or `Ctrl+C`.

### Connect the terminal session to the IDE

```text
/ide
```

### Voice dictation

```text
/voice
/voice tap
```

User settings file (`~/.claude/settings.json`):

```json
{
  "voice": {
    "enabled": true,
    "mode": "tap"
  }
}
```

Voice requires Claude.ai authentication and local execution. It is unavailable with direct API-key, Bedrock, Vertex, Foundry, remote, or SSH sessions.

### Setup verification

- `claude --version` returns a version.
- Authentication completes and persists.
- A session can summarize files in a real project directory.
- `/ide` connects to the intended editor window.
- Voice tap mode records and submits a coding phrase correctly.

---

## Slash Commands and Modes Steering

### Essential commands

```text
/help
/init
/status
/context
/compact
/re
/clear
/exit
/voice tap
```

### What they do

| Command | Purpose |
|---|---|
| `/init` | Inspect the repository and generate a first-draft `CLAUDE.md` |
| `/status` | Show model, token use, context use, and session state |
| `/context` | Show what currently occupies the context window |
| `/compact` | Compress the conversation while preserving useful state |
| `/re` | Rewind to an earlier turn and redirect the session |
| `/clear` | Discard the current conversation context |

Enter Plan Mode with `Shift+Tab` twice from the session prompt. Exit it before asking Claude to implement an approved plan.

### Custom command location

```text
.claude/commands/<command-name>.md
```

Treat custom slash commands as repeatable workflows. Their instructions should name the steps, stop conditions, and expected report.

---

## Plan Mode and Memory Architecture

### Plan Mode opening pattern

```text
Ask me clarifying questions one at a time until you are 95% confident you
understand the task. Then produce a detailed plan for review. Do not edit any
files until I approve the plan.
```

Review a plan for:

- Exact files and boundaries
- Order of operations and dependency direction
- Assumptions surfaced by the agent
- Verification after each meaningful unit
- Rollback or recovery path

### The memory stack

| Layer | Scope | Best content |
|---|---|---|
| Session context | Current conversation | Active task, recent diffs, command output |
| `CLAUDE.md` | Repository | Identity, stack, hard constraints, pointers |
| Project context files | Repository/subdirectory | API conventions, testing standards, local workflows |
| `/memory` and Ogham | Personal/cross-tool | Durable decisions, lessons, preferences |
| Context7 | Current external knowledge | Live library and framework documentation |

### CLAUDE.md content

- What the project is
- Stack and essential commands
- Folder and ownership conventions
- Never-do rules
- Verification policy
- Pointers to supporting context files

Keep it compact. If detailed guidance belongs in a supporting file, point to it rather than duplicating it.

---

## Memory in Practice

### Generate the first draft

```text
/init
```

### CLAUDE.md refinement prompts

> "The project overview is accurate but add that this codebase has intentional structural problems that will be addressed in a refactor — Claude should understand the current state and avoid reinforcing the bad patterns."

> "Add a Never Do These Things section with the following rules: never add new route logic to routes.js — it is already too large and any new routes belong in dedicated route files, never add new utility functions to utils.js without first checking whether they belong in a more specific module, never import from files in the misc/ directory — that code is dead and scheduled for removal."

> "Add a note that test files should follow the naming pattern resource.test.js — the current inconsistent naming in the tests/ directory is a known problem that will be standardised."

```bash
git add CLAUDE.md
git commit -m "docs: add CLAUDE.md with project context and conventions"
```

### Project context files

```bash
mkdir -p context
```

> "Create context/api-conventions.md. This skill file should instruct Claude on how to add a new API endpoint to the Taskr API. The instructions should specify: new routes belong in a dedicated file in src/routes/ after the refactor, each route file should export a named Express Router, the filename should match the resource name in lowercase plural, every route handler should call a corresponding function in src/services/ rather than containing business logic directly, errors should be passed to Express's next function with a status code and message object, and all routes should be registered in src/index.js using app.use with the resource path and the imported router."

> "Create context/testing-standards.md. This skill file should instruct Claude on how to write tests for the Taskr API. The instructions should specify: test files should follow the naming convention resource.test.js in lowercase, tests should use Jest and supertest against an in-memory SQLite database so no external setup is required, each test file should cover the happy path and at least two error cases for each endpoint, test descriptions should be plain English describing what the endpoint does rather than the implementation detail, and the test suite should be runnable with npm test with no other setup required."

> "Add a Context Files section to CLAUDE.md that lists context/api-conventions.md for any task involving API routes or endpoints, and context/testing-standards.md for any task involving tests or test coverage."

```bash
git add context/ CLAUDE.md
git commit -m "docs: add skill files for API conventions and testing standards"
```

### Inspect and compact context

```text
/status
/context
/compact
```

Post-compaction verification prompt:

> "Based on what we just discussed about the project structure, what would be the most impactful first step in a refactor?"

---

## Extended Memory: Ogham MCP & Context7

### Prerequisites

- A Supabase project URL and secret/service-role key
- `uv` and Python 3.13+
- Ollama running locally with `embeddinggemma`

```bash
uv --version
curl -LsSf https://astral.sh/uv/install.sh | sh
ollama pull embeddinggemma
ollama list
```

### Run the Ogham setup wizard

```bash
uvx --from ogham-mcp ogham init
```

Wizard choices used in the course:

1. Ollama at `http://localhost:11434`
2. Supabase with the secret/service-role key
3. `uvx` execution mode
4. Apply the supplied `sql/schema.sql` in Supabase SQL Editor
5. Configure Claude Code and any other clients you use
6. Require successful Supabase and Ollama connection tests

If the wizard creates `.env`, immediately confirm it is ignored by git.

### Ogham store prompt

> "Remember the following about the practice repository for this course: it is called agentic-engineering-practice and contains the Taskr API, a Node.js Express REST API using SQLite via better-sqlite3. The codebase currently has intentional structural problems — mixed naming conventions, route logic in a single routes.js file, utility functions scattered across utils.js without domain separation, and dead code in the misc/ directory. These will be addressed in a refactor. The test files also have inconsistent naming that will be standardised."

Close the session, open a new session in another directory, and retrieve:

> "What is the practice repository for this course and what are its current structural problems?"

Useful memory habit:

> "Remember that the Taskr API uses supertest with an in-memory SQLite database for tests — no database reset is needed between test runs."

### Context7 install and verification

```bash
claude mcp add context7 -- npx -y @upstash/context7-mcp
```

Restart Claude Code, then ask:

> "Use Context7 to find the current documentation for Express.js — specifically how to register middleware."

---

## Practice Repository: Architecture Refactor

### Clean baseline

```bash
cd agentic-engineering-practice
git status
git checkout -b refactor/architecture
git add .
git commit -m "docs: CLAUDE.md and skill files"
```

### Plan Mode prompt

> "Ask me clarifying questions until you're 95% confident you understand the full scope of this refactor, then produce a detailed plan. The goal is a clean architecture with consistent kebab-case naming throughout, a src/ directory with routes/, services/, db/queries/, middleware/, and utils/ subdirectories, logic separated so routes call services and services call queries, dead code in misc/ removed entirely, and the test files renamed to follow the resource.test.js convention. The plan should cover the exact file renames, what moves where, and the order of operations. Do not edit anything yet."

### Execute and verify

```bash
git diff --stat
npm test
npm pkg set scripts.dev="node --watch src/index.js"
npm pkg set scripts.start="node src/index.js"
npm run dev
```

### Update durable context

> "Update the CLAUDE.md to reflect the refactored structure. The folder structure section should describe the new src/ layout — routes call services, services call queries, queries call the database connection. The naming conventions section should state that all files use kebab-case. Remove any references to misc/, UserController.js, get-tasks.js, or the old structure. Update the dev and start script commands to reference src/index.js."

### Commit and merge

```bash
git add .
git commit -m "refactor: clean architecture, consistent naming, src/ structure"
git checkout main
git merge refactor/architecture
git push
```

If the refactor needs to be abandoned, return to the clean branch rather than fixing forward through an untrusted state. Inspect the failure, add the missing constraint to the plan, and rerun on a fresh branch.

---

## Permissions, YOLO Mode and Autonomous Execution

### Project permission file

`.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      "npm test",
      "npm run build",
      "npm run dev",
      "git status",
      "git diff",
      "git add",
      "git commit",
      "git push",
      "gh pr create"
    ],
    "deny": [
      "rm -rf",
      "curl",
      "wget"
    ]
  }
}
```

Use `/permissions` for session-only changes.

### Autonomous execution

```bash
claude --dangerously-skip-permissions
```

The three non-negotiable rules:

1. Commit first.
2. Work on a dedicated branch.
3. Scope the task tightly.

### Course YOLO task

```bash
git checkout -b auto/add-jsdoc
git status
claude --dangerously-skip-permissions \
  "Add JSDoc comments to every exported function in the utils/ directory. Each comment should describe what the function does, its parameters, and its return value. Do not touch any files outside utils/. Do not change any function implementations."
```

Review the diff, run verification, then commit only the intended files:

```bash
git add utils/
git commit -m "docs: add JSDoc comments to utils functions"
```

Run broad autonomous changes in two phases: approve the design in Plan Mode, then execute the approved plan autonomously from a recoverable baseline.

---

## Verification Loops

### verify-app command prompt

> "Create a file at .claude/commands/verify-app.md. The command should run the full verification suite for this project after any significant change. The steps should be: run npm test with the runInBand flag to run the full test suite serially. The test suite uses supertest and exercises all API endpoints directly — if tests pass, the server and all endpoints are working correctly. If any tests fail, fix the implementation and run the tests again until all pass. Report the result clearly: either all tests passing and all endpoints verified, or the specific failures that need addressing."

### CLAUDE.md verification policy prompt

> "Add a Verification section to CLAUDE.md. It should state that after completing any significant change, run /verify-app before committing. Do not commit if any tests are failing. The test suite uses supertest and covers all API endpoints — a passing run confirms the full application is working correctly."

### Fast Stop hook

Use `.claude/settings.local.json` for a local per-turn activity log. Keep the Stop hook fast; the full test suite belongs in `/verify-app`.

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "echo \"$(date '+%Y-%m-%d %H:%M:%S') — files modified: $({ git diff --name-only; git ls-files --others --exclude-standard; } | tr '\n' ' ')\" >> .claude/hook.log"
          }
        ]
      }
    ]
  }
}
```

Hook test prompt:

> "Add a comment to the top of the main entry point file describing what it does."

```bash
cat .claude/hook.log
/verify-app
git add .claude/ CLAUDE.md
git commit -m "setup: verify-app command, Stop hook, and CLAUDE.md verification policy"
```

Route mechanical test execution to a smaller model; keep architectural simplification and judgment on a stronger model.

---

## Routines

### Trigger choice

| Trigger | Use it for |
|---|---|
| Scheduled | Nightly health checks, weekly drift checks, recurring triage |
| API | Deploy verification or external systems that can POST to an endpoint |
| GitHub webhook | PR review, merge follow-up, cross-repository automation |
| `/loop` | Lightweight polling while the current local session remains open |

### Local loop example

Start the app in another terminal:

```bash
npm run dev
```

Then run:

```text
/loop "GET http://localhost:3000/tasks. Log status code and healthy or down. Every 30 Seconds."
```

### Nightly test routine prompt

> "Run the full test suite for this repository using `npm test -- --runInBand`. When the run completes: if all tests pass, post a one-line summary stating the number of tests that passed and confirming the suite is clean. If any tests fail, post a structured report identifying which tests failed, which endpoints they cover, and what the failure message indicates about the likely cause. Do not attempt to fix anything — report only."

Routine prompts must be self-contained and define what to do when there is nothing actionable, input is unexpected, or the task would leave scope.

---

## '/Goal' Command

Start the session with permission prompts bypassed so the multi-turn loop can run unattended:

```bash
claude --dangerously-skip-permissions
```

### Course goal

```text
/goal all JSDoc comments across every file in src/ are present and complete — every exported function has @param tags for each parameter, a @returns tag, and a one-sentence @description. Prove it by running grep -r "@description" src/ and showing the output covers every exported function. No implementation files are modified, only documentation. Stop after 15 turns.
```

### Goal controls

```text
/goal          # show condition, turn count, usage, evaluator reason
/goal clear    # stop and clear an active goal
```

A strong goal contains:

1. One measurable true-or-false end state
2. A stated command or artefact that proves completion
3. Explicit scope constraints
4. A turn limit for bounded unattended work

When complete, review the diff and verify before committing:

```text
/verify-app
```

```bash
git add src/
git commit -m "docs: complete JSDoc coverage across all exported functions"
```

---

## Plugins

### What a plugin can contain

- Skills — instructions for specific tasks
- Rules — always/never behavioural constraints
- Agents — reusable specialist roles
- Hooks — actions triggered by lifecycle events
- MCP servers — external tool connections
- LSP configuration — language-aware tooling

`CLAUDE.md` describes this project. A plugin captures how you or your team work across projects.

### Strong skill and rule criteria

- Specific enough to trigger at the right time
- Operational steps rather than aspirations
- Constraints that can be checked in output
- Explanations for important prohibitions
- Narrow scope and minimal always-loaded context

Skills teach a procedure. Rules govern behaviour. Context should activate only where it applies.

---

## Install a Plugin

### Install Superpowers and observe behaviour

```text
/plugin install superpowers@claude-plugins-official
```

Open a fresh session and ask:

> "I want to add a tagging feature — users should be able to add tags to tasks and filter tasks by tag."

Inspect the reference skills and manifest:

```bash
ls ~/.claude/plugins/cache/claude-plugins-official/superpowers/*/skills/
cat ~/.claude/plugins/cache/claude-plugins-official/superpowers/*/skills/brainstorming/SKILL.md
cat ~/.claude/plugins/cache/claude-plugins-official/superpowers/*/skills/test-driven-development/SKILL.md
cat ~/.claude/plugins/cache/claude-plugins-official/superpowers/*/.claude-plugin/plugin.json
```

### Personal plugin structure

```text
my-standards/
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json
└── skills/
    └── api-endpoints/
        └── SKILL.md
```

```bash
git clone https://github.com/your-username/my-standards
cd my-standards
mkdir -p .claude-plugin skills/api-endpoints
touch .claude-plugin/plugin.json .claude-plugin/marketplace.json
touch skills/api-endpoints/SKILL.md
find . -not -path './.git/*' | sort
```

### Manifest prompts

> "Write the .claude-plugin/plugin.json file. The content should be a JSON object with: name set to my-standards, description set to a one-sentence description of my personal coding standards and conventions, version set to 1.0.0, and author as a nested object with name set to my name."

> "Write the .claude-plugin/marketplace.json file. The content should be a JSON object with: name set to my-standards-marketplace, owner as a nested object with name set to my name, and plugins as an array containing one object with name set to my-standards, source set to a single dot meaning the current directory, and description matching the plugin description."

### Skill prompt

> "Write the skills/api-endpoints/SKILL.md file. Start with YAML frontmatter — three dashes, then description on its own line set to: Guides the creation of new API endpoints. Use when adding routes, handlers, or controllers to the project, then three dashes to close the frontmatter. Then write the skill body describing the conventions for adding endpoints — where route files live, naming conventions, the error handling pattern, the response shape, and what must be present before committing."

```bash
git add .
git commit -m "init: plugin manifest, marketplace manifest, api-endpoints skill"
git push
```

### Register and install globally

```text
/plugin marketplace add your-username/my-standards
/plugin marketplace list
/plugin install my-standards@my-standards-marketplace
/plugin list
/reload-plugins
/my-standards:api-endpoints
```

### Update flow

```text
/plugin marketplace update my-standards-marketplace
/reload-plugins
```

### Project reference prompt

> "Add a section called Active Plugins to CLAUDE.md. Note that the my-standards plugin is active and its conventions apply to all code in this project. The api-endpoints skill should be used when adding new routes."

---

## Vercel (CLI vs. MCP)

### Install and inspect

```bash
npm install -g vercel
vercel --version
vercel ls
vercel env ls
```

If using the Vercel MCP:

```bash
claude mcp add --transport http vercel https://mcp.vercel.com
```

Then authenticate in a Claude Code session with `/mcp`.

### Compare the two paths

MCP request:

> "What's the status of my most recent Vercel deployment?"

CLI request:

> "What's the status of my most recent Vercel deployment? Use the Vercel CLI."

### Three-tier hierarchy

1. CLI for a known operation
2. Thin CLI wrapper over an API when no CLI exists
3. MCP for discovery or actions where structured integration earns its context cost

### External-tools skill prompt

> "Create a skill file at ~/.claude/plugins/my-standards/skills/external-tools.md. The content should document the CLIs available for agent use in my environment and when to use them instead of their MCP equivalents. Include: `vercel` — use the Vercel CLI for deployment status, project management, and environment variable configuration. `vercel ls` for deployment status, `vercel env ls` for environment variables. Use this instead of the Vercel MCP in any session where you already know what operation you need. I'll add more CLIs to this file as I install them."

Reinstall or refresh the plugin after editing it, then confirm the skill is available in a fresh session.

---

## Terminal Module Safety Checklist

- Start broad changes from a clean, committed branch.
- Use Plan Mode before high-impact or ambiguous work.
- Keep secrets in ignored local environment files.
- Review diffs before accepting or committing.
- Run `/verify-app` before every meaningful commit.
- Require concrete evidence for routines and `/goal` completion.
- Prefer direct CLI calls for known external-tool operations.
