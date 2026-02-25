---
layout: post
title: "The Code is the Documentation"
date: 2026-02-18
excerpt: "Documentation rot slows down every team. Discover how to eliminate this friction by leveraging AI workflows to automate the descriptive overhead of technical documentation, keeping your docs as current as your code."
---

![Hero Image](/assets/images/code-documentation-hero.jpg)

Let’s be honest: "The code is the documentation" is usually just a polite way of saying, "I didn’t write a README, and I’m hoping you’re smart enough to figure out my nested ternary operators."

We’ve all been there. You jump into a repo and the `/docs` folder is a ghost town of `.md` files that describe a version of the system that hasn't existed since it was first written. This is documentation rot. It’s the friction that slows down every PR, every onboarding session, and every cross-team sync.

Even if you’re skeptical about agentic coding - the idea of an AI autonomously making logic changes to your codebase - there is a massive middle ground: using AI to kill the friction of documentation. You aren't giving an AI free reign over your code; you're just making it handle the descriptive overhead that humans usually skip.

---

## THE HUMAN SIDE OF TECHNICAL DOCS

We don't just write docs to satisfy a linter. Accurate, living documentation is the bridge between the black box of the code and the humans surrounding it:

*   **Onboarding**: It’s the difference between a new engineer shipping on day two or spending a week just poking around.
*   **Non-Technical Stakeholders**: Clear docs allow PMs, designers, and Ops to understand the what and where without needing to shoulder-tap you every twenty minutes.
*   **Code Validation**: Sometimes, seeing the AI describe your code back to you is the best way to realize, "Wait, that’s not actually what I intended that function to do." It’s a review session that writes itself.

---

## THE SETUP: GEMINI CLI & ANTIGRAVITY

I’ve been leaning on two tools for my development these days: [Gemini CLI](https://github.com/google/gemini-cli) for terminal work and [Antigravity](https://antigravity.google/) for staying in the flow of the IDE. Both allow for custom workflows, essentially recipes that tell the AI: "Stop guessing and follow this specific process."

### 1. THE TERMINAL APPROACH: GEMINI CLI

I wrote a [custom slash command](https://geminicli.com/docs/cli/custom-commands/) called `/update-docs`. Instead of me staring at a blank file, I run this, and the CLI scans my recent changes and updates the documentation based on the actual logic in the files.

You can install this globally (by dropping the config into `~/.gemini/commands/`) or keep it project-centric (checking it into your project root at `./gemini/commands/`). Check out my [`update-docs.toml`](https://github.com/joeshirey/AgenticCodingHelperTemplates/blob/main/gemini-cli/commands/update-docs.toml) as an example you can fork and modify for your own setup.

### 2. THE IDE APPROACH: ANTIGRAVITY

When I’m in the flow, I use Antigravity’s [workflows](https://antigravity.google/docs/rules-workflows) - markdown files that act as a flight plan for the AI agent. Just like the CLI, you have options:

*   **Global Workflows**: Stored in `~/.gemini/antigravity/global_workflows/`, these are available across every project you open.
*   **Project Workflows**: Stored in your workspace at `.agent/workflows/`, these allow you to define rules specific to that repo's tech stack or documentation style.

I use this [`/update-docs`](https://github.com/joeshirey/AgenticCodingHelperTemplates/blob/main/antigravity/global_workflows/update-docs.md) workflow as an example to keep my docs reconciled with my code.

---

## TAILORING TO YOUR STANDARDS

The best part of this approach is its flexibility. While I have my own preferences for what a project needs, you can tailor these tools to any documentation standard. For example, my workflows often handle:

*   **PRD (Product Requirements Document)**: This defines what is being built and why. It’s the source of truth for features and user needs.
*   **TDD (Technical Design Document)**: This is the blueprint for how it will be built. It outlines architecture, data models, and component interactions.

Because these templates are customizable, you can also include things like `DEPLOYMENT.md` or even Mermaid diagrams. Letting the AI read the code and draw the diagram keeps your architecture visuals from becoming obsolete.

---

## SET IT AND FORGET IT: AUTOMATED WORKFLOWS

If you want to reach "Doc Nirvana," don't even make running the command a manual step. You can integrate these CLI commands directly into your automation pipelines.

Whether you trigger the update as part of a local commit script or as a step in your CI/CD pipeline, the goal is the same: your documentation should be updated automatically every time the code changes. This ensures that the documentation is never an afterthought, it’s a byproduct of the commit itself.

---

## ELIMINATING THE TOIL

If you find this approach useful, you may find other workflows that can automate your specific technical toil. Letting an AI write your docs isn't about giving up control; it’s about automating the part of your job that creates the most friction.

By defining these workflows -whether it's through a `.toml` file in the CLI or a `.md` workflow in Antigravity - you’re building a "Doc-as-Code" pipeline that actually stays current. You’re not trusting the AI to invent logic; you’re just asking it to describe the logic you already wrote.

At the end of the day, the documentation rot is gone, and you get to go back to writing the code you actually care about.
