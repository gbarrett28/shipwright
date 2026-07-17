---
name: using-shipwright
description: Use when starting any conversation in a project that has installed the shipwright plugin, or before starting any non-trivial new work — establishes the required-skill-invocation policy for shipwright's own skills, the check-before-you-build discipline (search skills/plugins/prior work before building something from scratch), token-efficiency and plan-sprint-size guidance, and the git-worktree caveat. Read this before any other shipwright skill.
---

# Using Shipwright

## Overview

`shipwright` extends the same invocation discipline `superpowers:using-superpowers`
establishes for superpowers skills to shipwright's own skills. Violating the
letter of the rules is violating the spirit of the rules — the same
principle applies here. If there is even a 1% chance a shipwright skill
applies to what you're doing, invoke it. This is not negotiable.

## Required Skill Invocations

These moments require invoking a specific `superpowers` skill — no
exceptions:

| Skill | Invoke when |
|---|---|
| `superpowers:brainstorming` | Before any new feature, significant code change, or design decision. No code is written until a design is presented and approved. |
| `superpowers:verification-before-completion` | Before claiming any task is complete, before stating tests pass, and before creating a commit or PR. |
| `superpowers:systematic-debugging` | Before attempting to fix any bug, test failure, or unexpected behaviour. Root cause first, fix second. |
| `superpowers:test-driven-development` | Before writing any implementation code for a feature or bugfix — write the failing test first. |
| `superpowers:finishing-a-development-branch` | After all implementation tasks are complete and verified, before merging/pushing — structures the merge/PR/cleanup decision. |
| `superpowers:requesting-code-review` | Before merging any feature branch — dispatches a fresh subagent reviewer with no session history for unbiased review. |
| `superpowers:receiving-code-review` | When receiving code review feedback — verify technically before implementing; never agree blindly. |

And these moments require invoking a `shipwright` skill:

| Skill | Invoke when |
|---|---|
| `shipwright:quality-gates` | Before every commit (bronze gate) and before every merge to `master`/`main` (silver gate). |
| `shipwright:commit-and-test-integrity` | When writing a commit message, when about to delete or change anything not committed to git, and whenever a test fails after a code change. |
| `shipwright:issue-hygiene` | Before filing a new issue in an issue tracker, and whenever closing or updating one. |
| `shipwright:tool-preferences` | Before choosing which tool to use for code navigation, review, or library documentation. |
| `shipwright:python-guidelines` | Before writing or reviewing Python code. |
| `shipwright:typescript-guidelines` | Before writing or reviewing TypeScript code. |

## Check Before You Build

Before starting non-trivial work — a new feature, a new file, a new
process, a new skill — actively check three things, don't rely on what you
already remember:

1. **Skills** — search installed skills (`ToolSearch`/skill listing) for one
   that already covers this, even if you're confident you know how to do it
   unaided.
2. **Plugins/tools** — is there an installed plugin or tool purpose-built
   for this?
3. **Previous work** — memory, git history, existing docs/specs/plans in
   the repo: has this already been solved, decided, or attempted?

**"I already know how to do this" is exactly the moment to check, not skip
checking.** Confidence in your own knowledge doesn't tell you whether a
better, pre-existing, already-tested answer is sitting one search away.

Concrete example this rule exists because of: while building this very
plugin, a session went straight into hand-drafting seven `SKILL.md` files
from scratch — despite `superpowers:writing-skills` (and a separate
`skill-creator` plugin) being installed and directly on-point for exactly
that task. It was caught only because the user asked "does Claude have a
skill for creating skills?" — not because the agent checked proactively
first.

## Red Flags — you are about to skip a required skill

- "This commit is trivial, I don't need to run the gate."
- "We can clean up the process stuff later."
- "I already know what quality-gates/commit-and-test-integrity says, I don't need to re-check it."
- "I already know how to do this" (see "Check Before You Build" above).

**All of these mean: invoke the skill before proceeding.**

## Rationalization Table

| Excuse | Reality |
|---|---|
| "It's just a one-line fix" | The gate script runs in seconds; skipping it to save seconds is a bad trade against a broken `master`. |
| "Nobody's reviewing this right now" | The rules exist independent of whether anyone's watching. |

## Token Efficiency

Prefer the approach that achieves the final result with the fewest total
tokens: avoid redundant reads, unnecessary exploration, and verbose output
where concise output suffices. When choosing a plan execution mode, prefer
inline execution (`superpowers:executing-plans`) over subagent-driven
execution — it uses fewer total tokens. Never offer the visual-companion
feature during brainstorming; if a visual is genuinely needed, use whatever
direct tool the project provides for that purpose instead.

## Plan Sprint Size

When writing an implementation plan, strongly prefer breaking it into
sprints of at most ~3 hours of inline execution each. Sprints are separate
plan files, each producing working, independently-testable software.

Two reasons, not one: running out of context mid-task forces an expensive
recovery (re-reading state to figure out where things stood before
continuing — tokens spent reconstructing, not building), and a ~3-hour
checkpoint gives a human partner something concrete to look at and react
to. The context-exhaustion risk may be less severe on current tooling than
it once was, but the checkpoint-visibility benefit holds regardless — ~3
hours has empirically been a size where progress stays visible without
constant check-ins.

- If a spec covers multiple independent subsystems, each subsystem is its
  own sprint.
- If a single subsystem exceeds ~3 hours, break it at a natural integration
  point.
- Only combine into a single sprint when splitting would produce work that
  cannot be meaningfully tested on its own.

This is a strong preference, not an absolute rule: if a feature is
genuinely simpler to implement atomically and the token cost is low, a
single sprint is fine.

## Git Worktrees

Do not use git worktrees for interactive feature work — not all tools work
correctly inside them. Use a feature branch in the main working directory
instead. Concretely, not hypothetically: a fresh worktree doesn't get
gitignored/untracked content the main checkout has (installed dependencies
like `node_modules` or a venv, and any real local fixture data), so tools
that assume either one fail inside a worktree in ways that look like a
code bug but are actually a missing-file problem. (This doesn't apply to
isolation mechanisms your harness manages itself, e.g. background-job
worktree isolation — that's a runtime concern, not a project workflow
choice.)
