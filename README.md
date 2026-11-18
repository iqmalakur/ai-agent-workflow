# AI Agent Workflow

This repository contains my workflow for using AI agents to plan, implement,
and review changes. It includes shared agent instructions and two skills.

## Setup

1. Place [`AGENTS.md`](AGENTS.md) where your coding agent loads its instructions.
2. If you use Claude Code, place [`CLAUDE.md`](CLAUDE.md) next to `AGENTS.md`.
   Claude Code uses the `@AGENTS.md` import to load the shared instructions.
3. Install each skill where your agent looks for skills. Keep its `SKILL.md`
   with any bundled `assets/` and `references/` directories.

## Skills

| Skill | When to use it |
| --- | --- |
| [`implementation-workflow`](skills/implementation-workflow/SKILL.md) | Plan or implement a change, or clean up its temporary design and plan when asked. |
| [`coding-guideline`](skills/coding-guideline/SKILL.md) | Write or review code and tests. Apply the shared rules and the relevant language or platform references. |

## Usage

Use both skills when an implementation changes code. Check the worktree before
editing. Follow applicable `AGENTS.md` instructions when they override skill
guidance.

- For a small change, implement and verify it in one pass.
- For a larger change, confirm the scope and get approval for the design and
  plan before implementation. Get approval before the next checkpoint unless
  the user asked to run all checkpoints in one pass.
