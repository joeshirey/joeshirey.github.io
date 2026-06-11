---
layout: post
title: "Part 6: The Autonomy Ladder"
date: 2026-06-02 12:00:00 -0700
excerpt: "Every external action runs through an autonomy gate with three settings -- gated, push-draft, and full -- so the coordinator does as much as I trust it to and no more."
tags: ["hermes-agent", "ai-agents", "automation"]
---

![The Autonomy Ladder](/assets/images/the-autonomy-ladder-hero.jpeg)

The previous posts in this series describe a system that does a lot of work. The coordinator plans tasks, dispatches them to coding engines, discovers and vets external tools, runs code inside sandboxes, and, as of the [last post](/2026/06/02/using-github-issues-as-backlog.html), manages a project's backlog as GitHub Issues. That's a useful agent. But it raises a question I've been sidestepping: how much of that work is it allowed to do without asking me first?

An always-on agent that stops and waits for permission before every action isn't useful. It's a chatbot with extra steps. An always-on agent that pushes code, opens PRs, closes issues, and modifies shared systems without asking isn't safe. It's a liability. The interesting system is somewhere in between, and the hard part is drawing the line.

## Three levels

Every external action in the system (push, open a PR, create an issue, close a stale ticket) runs through an autonomy gate with three settings:

**Gated** is the default. The coordinator prepares the action, previews exactly what it would do, and stops. It returns `awaiting_confirmation` with a `command_preview`, the literal commands it would run on the remote. Nothing happens until I review the preview and re-invoke with `--confirm`. The coordinator does the work; I make the call.

**Push-draft** lets the coordinator push code and open draft PRs without confirmation. Draft PRs are visible but explicitly not ready for review. They signal "work in progress" without triggering notifications or review requests. The coordinator can stage work while I'm away, but it can't mark anything as ready for someone to act on.

**Full** removes the confirmation gate. The coordinator pushes, opens PRs, and can mark them ready for review. This is the mode for projects where I've decided the coordinator has earned enough trust to act independently, where the review process and CI checks downstream are strong enough to catch what the coordinator might miss.

Each level is a strictly wider scope than the one below it. Gated is a subset of push-draft, which is a subset of full. Moving up the ladder means granting more autonomy; moving down means pulling it back.

## What's always allowed

Some actions are local-only and never need permission:

- **Commits.** A commit is a local operation. It modifies the working tree and the git history on my machine, but it doesn't touch any remote. The coordinator can commit freely because there's no blast radius beyond my local checkout.
- **Reading.** Files, git history, test output, CI status checks. Anything read-only is unrestricted.
- **Planning.** The coordinator can plan all day without a gate. Plans don't change shared state.
- **Discovery.** Searching the skill indexes is read-only. The coordinator can discover and rank candidates without permission. The gate comes later, when it tries to vault one.
- **Running tests.** Local test execution is safe and informative.

These are the operations that make the agent useful even in gated mode. It can do all the thinking, all the preparation, and all the local work without stopping. The gate only fires when the action would be visible outside my machine.

## What's never allowed

Some actions are hard-blocked regardless of autonomy level:

- **Merging a PR.** The coordinator never merges. When CI is green and a PR is ready, it alerts me. I click the button. This is the one action I never want automated, because a merge is irreversible in the ways that matter: it changes main, it triggers deploys, it affects other people's work. The coordinator can do everything up to the merge and everything after it, but the merge itself is mine.
- **Overriding a FAIL audit.** If the security auditor says FAIL, the source is not vaulted, not injected, not executed. There's no `--force` flag. The answer is no.
- **Running untrusted code on the host.** Tier 2 and 3 tools go through the sandbox or they don't run. If no sandbox is available, they're blocked. There's no fallback to unsandboxed execution.
- **Fabricating trust.** The coordinator can't write a SKILL.md, stash it in `/tmp`, run it through the ingestion pipeline, and pretend it came from a trusted source. Vaulted skills must come from real remote sources with verifiable provenance.

These aren't autonomy levels. They're walls. They exist because some actions have consequences that no amount of trust in the agent should override. A merge affects other people. A FAIL override defeats the security pipeline. Unsandboxed execution of untrusted code is the thing the sandbox was built to prevent.

## Per-project configuration

Different projects deserve different levels of trust. A personal side project where I'm the only contributor can safely run at `push-draft` or `full`. A shared repo with downstream consumers should probably stay at `gated` until I've watched the coordinator handle a few cycles cleanly.

The autonomy level is set per project, with a clear precedence chain:

1. **Command-line flag** (`--autonomy`) — overrides everything, for one-off adjustments
2. **Project config file** (`.hermes-github.yaml` or `.hermes-backlog.yaml` in the repo root) — per-repo default
3. **Global config** (`config.yaml` under `github.autonomy` or `github_backlog.autonomy`) — system-wide default
4. **Hard default** — `gated`

The precedence means I can set a global default of `gated`, override it to `push-draft` for a specific repo, and still pass `--autonomy full` on one command if I'm watching the terminal and want to skip the confirmation for a push I've already reviewed.

Both the GitHub lifecycle tool (commits, pushes, PRs, CI) and the backlog tool (issues, labels, triage, grooming) respect the same precedence chain. They share the pattern because they share the problem: both make changes visible to other people.

## The confirmation flow

In gated mode, a mutating action plays out in two rounds:

**Round one:** I tell the coordinator to push the branch. It runs the push command internally, sees that autonomy is `gated`, and stops before executing. It returns:
- `status: awaiting_confirmation`
- `command_preview: git push origin feature/add-auth`
- A human-readable explanation of what would happen

**Round two:** I read the preview, decide it's fine, and re-invoke with `--confirm`. The coordinator runs the actual command.

The gap between the two rounds is where the human judgment lives. It's where I notice that the branch name is wrong, or the target repo isn't the one I meant, or the diff includes a file I didn't intend to push. The preview makes the action legible before it's irreversible.

This is deliberate friction. It's not a bug in the workflow. It's the feature that makes `gated` mode safe. The coordinator does the work of preparing the action; I do the work of deciding whether to take it. Both jobs are real work, and splitting them is the point.

## Graduated trust in the backlog

The backlog tool has a subtlety that's worth calling out. Not all mutations are equal, and the tool treats them differently:

**Create, enrich, and triage** only add things. They create issues, add labels, write comments, and rewrite issue bodies with structured metadata. They never close or merge anything. Even at full autonomy, these operations can only make the backlog bigger and more organized. They can't shrink it or remove work.

**Groom** is the exception. The weekly grooming sweep can close issues — but only two kinds: stale issues that have been idle past a grace period, and confirmed duplicates. And even then, it only closes behind the autonomy gate. In default `gated` mode, grooming produces a digest of what it *would* do and writes nothing. With `--confirm` or at higher autonomy, it applies the changes. And `--no-close` is always available as a safety valve: it lets the groomer run its analysis and apply elevations and warnings, but suppresses every close.

This is the autonomy ladder at its most granular. "What can the agent do?" isn't one question. It's a question per operation, per context, per project. The backlog tool answers it differently for creation (always safe to automate) versus deletion (safe only behind a gate, and only for specific cases).

## The cron question

Scheduled tasks add a dimension. A nightly triage sweep or a weekly grooming run is useful precisely because it happens while I'm not watching. But an agent acting on a schedule in `full` autonomy mode is an agent making changes to shared systems at 2am without oversight.

My approach: scheduled tasks are **documented, not auto-registered**. The config files have cron expressions for nightly triage (`13 2 * * *`) and weekly grooming (`7 9 * * 1`), and the `approvals.cron_mode` is set to `deny`. The system knows *when* these jobs should run, but it won't run them unless I explicitly wire them up. If I want a nightly triage on a specific repo, I register it myself with the autonomy level I'm comfortable with.

In `gated` mode, a scheduled run produces a digest and stops. I review it next morning and apply with `--confirm` if it looks right. In `push-draft` or `full`, the scheduled run applies changes autonomously. The autonomy level I choose for the cron job is a statement about how much I trust the automation on that particular repo.

This is conservative by design. An automated task that silently closes stale issues or re-pulls skill indexes while I'm asleep should be an explicit choice, not a default.

## What I've learned

After running this system for a while, the biggest lesson is that `gated` mode is a better default than I expected. The confirmation step is fast. I read a one-line preview and type `--confirm`, and the peace of mind is worth the two seconds. I've caught wrong branches, unintended pushes, and grooming actions I didn't agree with, all in the preview step.

The second lesson is that `push-draft` is the sweet spot for most active projects. The coordinator stages work as draft PRs while I'm focused on something else, and I review them in batch when I'm ready. It's autonomous enough to be useful and constrained enough to be safe. Draft PRs are the right primitive here. They communicate "in progress" without demanding anyone's attention.

I've only used `full` on repos where I'm the sole contributor and the CI pipeline is comprehensive enough to catch regressions. Even there, the merge wall holds. The coordinator opens the PR, CI runs, and I merge when it's green. Full autonomy means "do everything except the last step," which is about as much trust as I'm willing to extend to an agent acting on my behalf in a shared system.

## Where this leaves things

This is the last piece of the core system. The previous posts covered the coordinator/implementer split, swappable coding engines, dynamic skill discovery, the security pipeline, and the GitHub Issues backlog. The autonomy ladder ties them together by answering the question that runs underneath all of them: how much should the agent do on its own?

The answer, it turns out, is less about the technology and more about the relationship. Gated mode isn't a limitation. It's a way to build trust incrementally. You start with previews and confirmations, you watch the agent make good decisions, and you gradually widen the scope. The ladder is there so you can climb it at whatever pace makes sense for each project.

The system I have now plans, codes, reviews, discovers tools, vets them, manages a backlog, and delivers work to GitHub — all coordinated by an agent that never writes code itself. It's not finished. But it's useful, and it's safe, and those two things turned out to be harder to get simultaneously than I expected.

---

### The Hermes Agent series

1. [Part 1: I Built an Always-On AI Coding Agent That Plans, Codes, and Reviews Its Own Work](/2026/05/24/always-on-ai-coding-agent.html)
2. [Part 2: One Coordinator, Swappable Coding Engines](/2026/06/01/swappable-coding-engines.html)
3. [Part 3: Dynamic Tool Discovery and Injection](/2026/06/02/dynamic-tool-discovery.html)
4. [Part 4: Running Untrusted Tools Safely](/2026/06/02/running-untrusted-tools-safely.html)
5. [Part 5: GitHub Issues as the Agent's Backlog](/2026/06/02/using-github-issues-as-backlog.html)
6. **Part 6: The Autonomy Ladder** (this post)
7. [Part 7: How the Agent Learns From Its Mistakes](/2026/06/10/how-the-agent-learns-from-its-mistakes.html)

[Implementation details and source](https://github.com/joeshirey/HermesCoderAgent)
