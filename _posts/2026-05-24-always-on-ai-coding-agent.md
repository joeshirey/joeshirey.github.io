---
layout: post
title: "Part 1: I Built an Always-On AI Coding Agent That Plans, Codes, and Reviews Its Own Work"
date: 2026-05-24
excerpt: "I took a 2019 MacBook Pro that was collecting dust and turned it into an autonomous coding coordinator. It plans before it codes, delegates to a separate AI for implementation, and reviews every line before calling the work done."
tags: ["hermes-agent", "ai-agents", "automation"]
---

![Always-On AI Coding Agent](/assets/images/always-on-ai-coding-agent-hero.jpeg)

*How to turn a spare Mac into an autonomous coding coordinator using Hermes Agent and Claude Code*

Most AI coding tools are reactive. You ask, they answer. You paste code, they suggest edits. But they don't *manage* work. They don't break a feature request into tasks, write the code, review it for security issues, check that it matches the spec, and report back when it's done.

I wanted something that does all of that, autonomously, 24/7, accessible from my phone.

So I took a 2019 MacBook Pro that was collecting dust, installed some open-source tools, and built a coding agent that operates like a senior engineering lead: it plans before it codes, delegates implementation to a separate AI, and reviews every line before calling the work done. I can message it from Telegram while I'm out walking the dog, and come back to find a tested, reviewed pull request waiting for me.

---

## The architecture: separation of concerns

The system has two layers, and the boundary between them is the most important design decision I made.

<div style="display: flex; flex-direction: column; align-items: center; gap: 0; margin: 2rem auto; max-width: 520px; font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;">
  <div style="background: #f0f4f8; border: 2px solid #94a3b8; border-radius: 10px; padding: 14px 28px; text-align: center; width: 100%; box-sizing: border-box;">
    <div style="font-weight: 600; font-size: 1.05rem; color: #1e293b;">You</div>
    <div style="font-size: 0.85rem; color: #64748b; margin-top: 4px;">Telegram / Discord / WhatsApp / CLI / Web UI</div>
  </div>
  <div style="font-size: 1.5rem; color: #94a3b8; line-height: 1.2;">↓</div>
  <div style="background: #e0f2fe; border: 2px solid #38bdf8; border-radius: 10px; padding: 14px 28px; text-align: center; width: 100%; box-sizing: border-box;">
    <div style="font-weight: 600; font-size: 1.05rem; color: #0c4a6e;">Hermes Coordinator</div>
    <div style="font-size: 0.85rem; color: #0369a1; margin-top: 4px;">plans, reviews, communicates</div>
  </div>
  <div style="font-size: 1.5rem; color: #94a3b8; line-height: 1.2;">↓</div>
  <div style="background: #fef3c7; border: 2px solid #f59e0b; border-radius: 10px; padding: 14px 28px; text-align: center; width: 100%; box-sizing: border-box;">
    <div style="font-weight: 600; font-size: 1.05rem; color: #78350f;">Claude Code</div>
    <div style="font-size: 0.85rem; color: #92400e; margin-top: 4px;">writes code, runs tests, commits</div>
  </div>
</div>

**Layer 1: The Coordinator** runs on [Hermes Agent](https://github.com/NousResearch/hermes-agent), an open-source autonomous AI agent framework from Nous Research, similar to [OpenClaw](https://github.com/openclaw/openclaw). It handles planning, task decomposition, review, and communication, and it uses Gemini 3.5 Flash as its model, which is fast, cheap, and good at orchestration. What I like about Hermes is that it's always on and learns over time. The more I use it, the better it gets at adapting to my coding approach.

**Layer 2: The Implementer** is [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview) running in print mode (`claude -p`). It receives a standalone prompt, writes code, runs tests, and returns results. I chose Claude Code because I'm learning it, but you could substitute any coding agent with a non-interactive mode, such as [Gemini CLI](https://github.com/google-gemini/gemini-cli), [Goose](https://github.com/block/goose), or [OpenCode](https://github.com/opencode-ai/opencode).

The coordinator *never writes code*. Claude Code *never plans*. This separation prevents the failure mode I've seen in single-agent setups where the AI gets lost halfway through a complex task because it's trying to do everything at once.

---

## The coordinator: a senior engineering lead in a config file

The coordinator's behavior is defined in a single file called `SOUL.md`, the agent's personality and operating manual combined. The core philosophy:

1. **Plan first:** before any coding begins, break the work into small, independently testable tasks
2. **Delegate all coding:** use Claude Code for every implementation task, never write code directly
3. **Review everything:** after each task, run a two-stage review: spec compliance first, then code quality
4. **Communicate clearly:** keep the user informed of what was planned, what was done, and what passed review

When you send the coordinator a message like *"Add user authentication to the API"*, it doesn't start writing code. It asks clarifying questions if anything is ambiguous, then creates an implementation plan where each task maps to a single Claude Code dispatch. It executes each task via `claude -p` with the exact prompt and allowed tools, checks each result against the spec and then against quality standards, and reports what got done and what needs attention.

One thing worth noting: Claude Code has no memory between dispatches. Each task prompt must be completely self-contained, including file paths, function signatures, project conventions, everything. The coordinator's planning skill enforces this, producing prompts that any Claude Code instance could execute cold.

---

## Seven roles, one agent

Rather than running seven separate AI agents (expensive, slow, hard to coordinate), the coordinator applies seven "role lenses" at different stages of the workflow. Each role is a Hermes skill, a markdown file with a charter, review checklist, and prompt templates.

<div style="overflow-x: auto; -webkit-overflow-scrolling: touch; margin: 1.5rem 0;">
<table style="width: 100%; border-collapse: collapse; min-width: 480px; font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif; font-size: 0.95rem;">
  <thead>
    <tr style="border-bottom: 2px solid #334155;">
      <th style="text-align: left; padding: 10px 12px; color: #1e293b; width: 18%;">Role</th>
      <th style="text-align: left; padding: 10px 12px; color: #1e293b; width: 28%;">When applied</th>
      <th style="text-align: left; padding: 10px 12px; color: #1e293b;">What it checks</th>
    </tr>
  </thead>
  <tbody>
    <tr style="border-bottom: 1px solid #e2e8f0;">
      <td style="padding: 10px 12px; font-weight: 600;">Architect</td>
      <td style="padding: 10px 12px;">During planning</td>
      <td style="padding: 10px 12px;">System design, dependencies, interfaces, scalability</td>
    </tr>
    <tr style="border-bottom: 1px solid #e2e8f0; background: #f8fafc;">
      <td style="padding: 10px 12px; font-weight: 600;">Implementer</td>
      <td style="padding: 10px 12px;">During dispatch</td>
      <td style="padding: 10px 12px;">Task decomposition, prompt construction, tool selection</td>
    </tr>
    <tr style="border-bottom: 1px solid #e2e8f0;">
      <td style="padding: 10px 12px; font-weight: 600;">Quality</td>
      <td style="padding: 10px 12px;">During spec review</td>
      <td style="padding: 10px 12px;">Spec compliance, test coverage, TDD enforcement</td>
    </tr>
    <tr style="border-bottom: 1px solid #e2e8f0; background: #f8fafc;">
      <td style="padding: 10px 12px; font-weight: 600;">Security</td>
      <td style="padding: 10px 12px;">Before merging</td>
      <td style="padding: 10px 12px;">OWASP Top 10, hardcoded secrets, dependency CVEs</td>
    </tr>
    <tr style="border-bottom: 1px solid #e2e8f0;">
      <td style="padding: 10px 12px; font-weight: 600;">Docs</td>
      <td style="padding: 10px 12px;">After implementation</td>
      <td style="padding: 10px 12px;">README updates, API docs, changelogs</td>
    </tr>
    <tr style="border-bottom: 1px solid #e2e8f0; background: #f8fafc;">
      <td style="padding: 10px 12px; font-weight: 600;">DevOps</td>
      <td style="padding: 10px 12px;">For infrastructure changes</td>
      <td style="padding: 10px 12px;">CI/CD, Docker configs, environment management</td>
    </tr>
    <tr>
      <td style="padding: 10px 12px; font-weight: 600;">Reviewer</td>
      <td style="padding: 10px 12px;">During code quality review</td>
      <td style="padding: 10px 12px;">Correctness, readability, maintainability, integration</td>
    </tr>
  </tbody>
</table>
</div>

These seven roles are inspired by [Squad](https://github.com/bradygaster/squad), a framework for scaffolding teams of specialist AI agents. Squad assigns each agent a character identity and a functional role (frontend, backend, tester, etc.). I adapted the concept, consolidating the roles into seven that fit my coordinator's review workflow.

The roles aren't separate agents or separate API calls. They're checklists and perspective shifts that the coordinator applies when reviewing Claude Code's output. The Architect lens during planning. The Quality lens during spec review. The Security lens before anything gets committed.

---

## The workflow engine

Four workflow skills chain together to form the development pipeline:

### 1. Writing plans

Every non-trivial task starts with a plan. The writing-plans skill produces structured implementation plans where each task is small enough for a single Claude Code dispatch (5-15 turns). Each task includes the exact `claude -p` prompt, allowed tools, timeout, and verification command.

A good plan makes implementation obvious. If Claude Code has to guess, the plan is incomplete.

### 2. Claude Code-driven development

This is the execution engine. For each task in the plan, the coordinator dispatches Claude Code with the complete prompt, then runs a spec review (did it implement what was asked, nothing more, nothing less?), then a quality review (is the code clean and tested?). If either review fails, it re-dispatches with specific fix instructions. After all tasks complete, an integration review checks whether everything works together.

The two-stage review matters because spec compliance and code quality are different problems. Code can be beautiful but wrong (doesn't match the spec). Code can be correct but terrible (works but unmaintainable). Checking both, in order, catches both failure modes.

### 3. Test-driven development

Every Claude Code prompt includes TDD instructions: write a failing test first, verify the failure, write minimal code to pass, verify the pass, run the full suite for regressions. The coordinator verifies TDD was actually followed. If Claude Code skipped writing tests first, it gets re-dispatched with explicit enforcement.

### 4. Pre-commit verification

Before anything gets committed, a verification pipeline runs: static security scan on the diff, test suite execution, coordinator self-review, then an independent Claude Code review from a fresh instance with no context about how the code was written. If issues are found, a fix loop runs up to twice before escalating to the user.

No agent should verify its own work. Fresh context finds what familiarity misses.

---

## Access anywhere

The always-on server means the coordinator is reachable from anywhere. Hermes supports multiple messaging platforms out of the box:

* **[Telegram](https://telegram.org/)**: Message from your phone. Ask for a feature, get a progress report, approve a plan. This is what I use when I'm away from my desk.
* **[Discord](https://discord.com/)**: Connect a bot to your server for team access.
* **[WhatsApp](https://www.whatsapp.com/)**: Another option for mobile access.
* **CLI**: Work from the terminal. A `coder` alias sets the right `HERMES_HOME` environment variable.
* **[hermes-webui](https://github.com/nesquena/hermes-webui)**: A three-panel layout with sessions, chat, and agent details.

---

## Keeping it running

Four macOS LaunchAgents auto-start everything on login and restart on crash:

1. Primary Hermes gateway
2. Coding coordinator gateway
3. Primary web UI (port 8787)
4. Coding coordinator web UI (port 8788)

Each uses `KeepAlive` with `SuccessfulExit: false`, so if the process crashes (non-zero exit), macOS restarts it automatically. Clean shutdowns stay down.

A nightly cron job backs up the entire coordinator configuration to a private GitHub repo (skills, SOUL.md, config files, but not secrets or databases). If the machine dies, I can recreate the setup from the backup in under an hour.

---

## What I'm investigating next

**Dynamic skill injection.** Right now the seven roles are static. In practice, reviewing a React frontend requires different expertise than reviewing a Python API. I'm working on having the coordinator detect the tech stack (check `package.json`, `requirements.txt`, and whatever else the project has) and inject technology-specific skills into the Claude Code prompts via `--append-system-prompt-file`. Think curated best-practice guides for React, FastAPI, Go, loaded dynamically based on what the project actually uses.

**Smarter task sizing.** The coordinator sometimes creates tasks that are too small (trivial one-line changes) or too big (Claude Code runs out of turns). I want to add feedback from execution results back into the planning skill: if a task consistently finishes in 2 turns, combine it with the next one. If it hits the turn limit, break it down further.

**CI integration.** Right now the coordinator runs tests locally. Wiring it into GitHub Actions so it can create PRs, wait for CI, and respond to review comments would close the loop between "code written" and "code shipped."

---

## Getting started

If you want to build this yourself, I've written a detailed technical reference with every command, config file, and skill definition you need. It covers:

- Preparing a Mac for always-on operation
- Installing Hermes Agent and Claude Code
- Creating the coordinator instance with all seven role skills and four workflow skills
- Setting up Telegram, auto-start, backup, and web UIs
- Supporting both Gemini and Claude as the coordinator model

These instructions should also work on WSL2, in a Docker container, or on Linux, though I haven't tested those environments. I'm using an old Mac that has nothing important on it beyond what gets backed up nightly to GitHub, so I'm not worried about Hermes or Claude breaking anything I can't fix. I'd be more careful on my main computer.

The full implementation details are in the companion repo: **[HermesCoderAgent](https://github.com/joeshirey/HermesCoderAgent)**. You could follow it step by step, or you could point a tool like Claude Code or Gemini CLI at it and have it build this interactively with you.

---

*This setup uses [Hermes Agent](https://github.com/NousResearch/hermes-agent) by Nous Research, [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview) by Anthropic, and [hermes-webui](https://github.com/nesquena/hermes-webui) by nesquena. The seven-role model is inspired by [Squad](https://github.com/bradygaster/squad). The workflow skills are inspired by [Superpowers](https://github.com/obra/superpowers) by obra. Both Squad and Superpowers are worth checking out.*

---

### The Hermes Agent series

1. **Part 1: I Built an Always-On AI Coding Agent That Plans, Codes, and Reviews Its Own Work** (this post)
2. [Part 2: One Coordinator, Swappable Coding Engines](/2026/06/01/swappable-coding-engines.html)
3. [Part 3: Dynamic Tool Discovery and Injection](/2026/06/02/dynamic-tool-discovery.html)
4. [Part 4: Running Untrusted Tools Safely](/2026/06/02/running-untrusted-tools-safely.html)
5. [Part 5: GitHub Issues as the Agent's Backlog](/2026/06/02/using-github-issues-as-backlog.html)
6. [Part 6: The Autonomy Ladder](/2026/06/02/the-autonomy-ladder.html)
7. [Part 7: How the Agent Learns From Its Mistakes](/2026/06/10/how-the-agent-learns-from-its-mistakes.html)

[Implementation details and source](https://github.com/joeshirey/HermesCoderAgent)
