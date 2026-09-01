# Claude Code Bootcamp Reference

Student reference for the **Claude Code Bootcamp** course. Keep these guides open while you work so you can grab a prompt, command, or concept without scrubbing back through video.

This is not a tutorial and not a codebase. It is the companion sheet for directing agents to plan, build, verify, and ship real software — rather than vibe-coding your way to output you cannot maintain.

## The discipline

Agentic Engineering is a shift in altitude: you stop directing *what* gets built and start directing *the system that builds it*, from planning through verification and ship.

The director's workflow is the through-line of the whole course:

- A planning system: PRD, content model, site contract, design standard, and standing orders (`CLAUDE.md` and equivalents)
- A three-standards evaluation lens: does it scale, is it secure, and do you know exactly why it works
- A Director's Audit loop: freeze the spec, judge the output against it, then ship or send it back

You will build and deploy real products along the way — typically a production backend, a connected frontend, and a motion-led portfolio.

## How this repo is organized

Each module file is a lesson-by-lesson reference: core concepts, copy-paste prompts, CLI commands, and the checklists used in class.

<table>
  <thead>
    <tr>
      <th width="140" align="left">Guide</th>
      <th align="left">Focus</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td width="140" nowrap><a href="Module_1_reference.md">Module&nbsp;1</a></td>
      <td>Claude — project workflow, planning documents, browser build, and the Director's Audit</td>
    </tr>
    <tr>
      <td nowrap><a href="Module_2_reference.md">Module&nbsp;2</a></td>
      <td>Claude Code in the terminal — setup, memory, permissions, verification, routines, goals, plugins, and Vercel tooling</td>
    </tr>
    <tr>
      <td nowrap><a href="Module_3_reference.md">Module&nbsp;3</a></td>
      <td>Claude Code desktop app — visual diffs, parallel sessions, Remote Control, and atomic shipping</td>
    </tr>
    <tr>
      <td nowrap><a href="Module_4_reference.md">Module&nbsp;4</a></td>
      <td>VibeMail backend — architecture, parallel server/schema work, deployment, and production verification</td>
    </tr>
    <tr>
      <td nowrap><a href="Module_5_reference.md">Module&nbsp;5</a></td>
      <td>Claude Design and Claude Code — VibeMail UI specification, handoff, migration, integration, and release</td>
    </tr>
    <tr>
      <td nowrap><a href="Module_6_reference.md">Module&nbsp;6</a></td>
      <td>Cinematic portfolio — motion direction, shaders, GSAP choreography, audit, contact route, and deployment</td>
    </tr>
  </tbody>
</table>

## Course map

The course is toolchain-agnostic in principle and Claude-native at the start.

**Claude Bootcamp** is the entry point: planning first, then Claude Code (Terminal and Desktop) and Claude Design, end to end across three products.

**Later in the full course:**

- Other agentic toolchains: Cursor / Composer, Codex, and Antigravity
- Semantic memory with pgvector
- A fully local "sovereign" AI stack
- Building your own MCP server
- The capstone

## What this course does not teach

- Programming fundamentals, HTML/CSS/JavaScript, or basic web architecture from scratch — those are assumed
- AI image and video generation — the motion work produces everything in code. A shader costs nothing, ships as kilobytes, and responds to the visitor in a way a generated clip never can

## How to use it

1. Open the module you are currently in.
2. Use the table of contents to jump to the lesson you are on.
3. Copy prompts and commands as written, then adapt them to *your* product, brief, and standing orders — not the instructor's worked example.
4. Treat planning documents as the source of truth. Prompts in these guides are often the method delta; without your PRD, contract, design standard, or motion brief behind them, they will not produce your work.

## Prerequisites

You should already be comfortable with:

- HTML, CSS, and JavaScript
- Basic web architecture (frontend vs backend, APIs, git, deploying a site)

The Golden Path used across course projects is React, Next.js, and Supabase unless a lesson explicitly says otherwise.
