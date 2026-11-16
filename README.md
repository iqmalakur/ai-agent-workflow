# AI Agent Workflow

This repository contains my workflow for using AI agents to plan, implement, and review changes.

## Skills

| Skill | When to use it |
| --- | --- |
| [`implementation-workflow`](skills/implementation-workflow/SKILL.md) | Plan or implement a change, or clean up a task's temporary design and plan. Use a single pass for small changes and approved design, plan, and checkpoints for larger changes. |
| [`coding-guideline`](skills/coding-guideline/SKILL.md) | Write or review production code, tests, fixtures, or embedded code. Apply the shared rules and the relevant language or platform references. |

## Usage

1. Make each skill directory available to your coding agent. Keep its `SKILL.md` and bundled `assets/` or `references/` together.
2. Ask the agent to use the relevant skill. For implementation work that changes code, use both skills.

Applicable `AGENTS.md` instructions can override skill guidance.
