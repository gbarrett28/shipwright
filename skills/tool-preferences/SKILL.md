---
name: tool-preferences
description: Use before choosing which tool to reach for — code navigation/analysis, PR review, or library/API documentation — when a specialized plugin or MCP tool might already be installed for that role.
---

# Tool Preferences

## Overview

Generic, availability-conditional tool preferences. Each project's own
CLAUDE.md states *which* concrete tool fills each role below; this skill
states the preference order and fallback behavior.

## The Rules

- **Prefer semantic/LSP-aware code-navigation tools** (e.g. serena, a
  language server) over raw filesystem tools (`Read`, `Glob`, `Grep`) for
  code analysis and modification, when such a tool is installed and
  running. Text search finds text matches, not references — it misses
  renamed/shadowed usages and false-positives on comments or strings. The
  result looks like a normal answer either way, so an incomplete grep-based
  result doesn't announce itself as wrong; the cost lands later, as a bug
  from a reference nobody found.
- **Prefer specialized review-agent plugins** (e.g. `pr-review-toolkit`,
  `coderabbit`) over ad hoc manual review when installed.
- **Prefer up-to-date library-documentation tools** (e.g. `context7`) over
  training-data knowledge for external library/API usage, when installed.
- **Prefer project-specific automation over manually reproducing its
  behavior** — existing build scripts, project generators, migration
  tools, codegen — rather than hand-writing what one of those would do for
  you. Consistent with `shipwright:using-shipwright`'s "Check Before You
  Build": the automation already encodes the correct behavior; reproducing
  it by hand risks drifting from it.

## Fallback behavior

If a project's designated tool for a role is unavailable (not installed,
not responding), don't silently fall back to a generic tool and continue —
ask the user to check the plugin/restart it first. Silent fallback hides a
broken tool setup behind degraded results that look normal.

## Mechanical Enforcement (recommended)

Self-discipline decays over a long session: the preference above holds at
the start of a task and drifts as investigative momentum builds — "let me
just grep for this real quick" wins over re-deriving a project's tool
mandate from its CLAUDE.md on every single tool call. A model that
correctly used the mandated tool for the first ten lookups of a session
can still lapse on the eleventh; nothing about having complied before
prevents lapsing later, because the check is re-derived from context each
time, not remembered as a standing constraint.

Where a project treats its semantic-navigation tool as *mandatory* (not
merely preferred), don't rely on re-deriving that from prose alone —
enforce it mechanically with a `PreToolUse` hook that intercepts
`Read`/`Grep`/`Glob` calls against the mandated tool's file types and
blocks or redirects them (see the harness's own hook-configuration
tooling, e.g. a `settings.json`-based hook skill, for the mechanism). A
hook makes the mandate a technical guarantee instead of a memory exercise
— it can't lapse under investigative momentum the way a self-check can,
because it isn't a self-check at all.

This is a recommendation for projects that want the mandate to hold
without relying on self-correction, not a requirement that every project
add such a hook. Either way, state the mandate's actual strictness
(advisory vs. mechanically enforced) explicitly in the project's own
CLAUDE.md, so an agent reading it knows whether a lapse will be caught
automatically or depends entirely on it noticing its own drift.
