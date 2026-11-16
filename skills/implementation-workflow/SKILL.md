---
name: implementation-workflow
description: Use when a user asks to implement or plan a change, or explicitly asks to clean up a task's temporary implementation documents.
---

# Implementation Workflow

Follow the applicable `AGENTS.md` instructions. For read-only explanations, reviews, and estimates without an implementation plan, answer directly. For a cleanup-only request, go to [Temporary documents](#temporary-documents).

## Review and classify the request

Check the request and available facts for conflicts, missing requirements, edge cases, and side effects. Explain any material problem or risk. Ask the user to clarify missing information and confirm each assumption before relying on it.

Classify the complete change by its impact, uncertainty, and verification
needs. Include affected callers, tests, fixtures, configuration, and
documentation. File count alone does not determine the classification.

A change is small only when all of these conditions hold:

- It has one bounded objective, and the affected behavior and dependencies
  are understood.
- It follows an established project pattern and requires no unresolved
  design decision.
- It preserves existing contracts used by callers and requires no migration
  of stored data.
- It has no material risk to access control, confidential data, financial
  correctness, or stored data.
- Reverting it requires no data restoration or compensation for external
  effects.
- One review checkpoint and focused verification can adequately cover the
  change and its affected behavior.

A bug fix may change behavior while remaining small when it restores the
existing contract and meets every condition above.

Classify the change as larger if any condition is not met. Use initial
read-only investigation to resolve uncertainty before classifying.
If material uncertainty remains, state it and use the larger-change workflow.

State the classification and its main reason before editing files.

For example, correcting typos in five documentation files can be small.
Changing access rules in one file is larger because it affects access control.

If a change stops meeting the small-change conditions during implementation,
stop further edits. Report the changes already made, the reason for
reclassification, and the scope that needs approval. Do not automatically
revert existing work. Follow the larger-change workflow before resuming.

For every larger change, including one that arises mid-chat, state the scope and what the change will deliver. Combine this with any findings or questions from the review. Use scope approval already given for the same change. Otherwise, wait for approval before writing the design and plan. Reconfirm the scope if it changes.

## Check the worktree

Before creating or editing files, including documents, check whether the directory is a Git repository. In a repository, run `git worktree list` and use the worktree the user named in this chat. If the user named none and the list has one worktree, use it. If it has multiple worktrees, list their paths and branches. Ask the user to choose and wait for the answer.

## Small changes

If the user only requests planning, give a short approach in chat and stop there. When authorized to implement, make one complete change in a single pass. Skip design and plan files and intermediate checkpoints. Report the approach, files changed, verification, and any accepted risks.

## Larger changes

Create two short English documents in the existing project documentation folder, or in `docs/` if there is none.

- Use the [design template](assets/design-template.md) for `<task>-design.md`. Record the scope the user approved.
- Use the [plan template](assets/plan-template.md) for `<task>-plan.md`. Follow the approved scope. Give each checkpoint a goal, file list, and verification steps with expected results. Make each checkpoint reviewable on its own.

Ask the user to approve both documents before implementation. For a planning-only request, stop at the requested planning deliverable and wait for authorization to implement.

Work within the approved design and plan. If you find a scope gap, a material error in the documents, a new risk, or a decision or assumption they do not cover, explain the issue and wait. After the user approves a change, update both documents as applicable before resuming.

At each checkpoint and at the end, report what changed and why, how it maps to the plan, and how to verify the result. List departures from the plan or state that there were none. Wait for permission to start the next checkpoint. If the user explicitly requests all checkpoints in one run, proceed between checkpoints and continue to honor scope and risk approvals.

## Temporary documents

Keep the design and plan as temporary working files. Never stage or commit them, mention them or their names in persistent files, or link to them from another file.

The user handles cleanup by default. When the user explicitly asks the agent to clean up, identify the current task's temporary design and plan and delete only those files. If their identity is unclear, ask the user which files to remove.
