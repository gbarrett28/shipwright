---
name: quality-gates
description: Use before every commit (bronze gate) and before every merge to master/main (silver gate) — covers the two-tier gate contract, keeping newly-written TDD tests inside the bronze run without gaming the fast-subset tag, and the bronze/silver split of doc-hygiene checks.
---

# Quality Gates

## Overview

A two-tier gate protects `master`/`main`: bronze (fast, every feature-branch
commit) and silver (thorough, every merge). Each project supplies its own
gate scripts implementing the checks for its stack; this skill is the
contract both scripts must satisfy, not the concrete commands.

## The Contract

**Bronze gate** — runs before every commit on a feature branch.
1. A gate script runs fast checks: typecheck/lint plus a fast test subset
   (see below).
2. On success it writes a one-time token; a pre-commit hook consumes it and
   blocks the commit if no token is present.
3. Never bypass the hook with `--no-verify` or equivalent.

**Silver gate** — runs before merging to `master`/`main`.
1. A gate script runs the full check suite: typecheck/lint, the full test
   suite, any slow/integration/e2e tests.
2. It performs the full doc-hygiene sweep (below).
3. On success it writes a token consumed by the hook for the merge commit.

## New tests belong in bronze — don't game the fast-subset tag

Tests written via `superpowers:test-driven-development` must be added to
the bronze gate's test run. Bronze's total runtime must also stay under 5
minutes. Reconcile these by keeping tests fast by default (no real I/O, no
sleeps, no slow fixtures) — a suite of a few hundred fast unit tests
comfortably fits the budget.

Only tag a test as slow/excluded from bronze if it is genuinely slow (real
integration I/O, browser automation, etc.) and defer it to silver. Tagging
a fast test as slow "just to be safe" or "to keep bronze snappy" satisfies
the letter of "bronze stays fast" while violating the rule that new tests
belong in bronze — the fast-subset tag is for tests that are actually slow,
not a blanket default. If bronze's runtime approaches the 5-minute ceiling
as the suite grows, that's a signal to tag genuinely slow tests or optimize
fixtures — never to preemptively exclude a fast test.

## Branch Workflow

All work happens on a feature branch (`feature/short-description`); never
commit directly to `master`/`main`. Bronze gates every feature-branch
commit; silver gates every commit to `master`/`main`, including merges.

## Doc Hygiene: Bronze vs Silver

**Bronze** (manual, per-commit, not scripted, doesn't count against the
5-minute budget): any plan file touched by this commit's work must have
accurate checkboxes — completed steps ticked, incomplete steps not. Only
plans/specs relevant to the current commit, not a repo-wide sweep. Don't
commit if a touched plan's checkboxes are inaccurate.

**Silver** (manual, pre-merge, thorough): sweep every spec/plan location in
the repo. Finished specs: incorporate their design into the relevant live
doc with concrete descriptions of what was built, then delete the spec
file. Finished plans (every step ticked): delete. Don't merge if any spec
or plan file with unchecked steps remains.

## Doc Conventions (default locations, overridable per project)

| Kind | Default location | Lifecycle |
|---|---|---|
| Spec | `docs/specs/` or `docs/superpowers/specs/` | Deleted once incorporated into a live doc |
| Plan | `docs/plans/` or `docs/superpowers/plans/` | Deleted once every step is ticked |
| Live doc | project-specific | Permanent; always reflects the current codebase |
