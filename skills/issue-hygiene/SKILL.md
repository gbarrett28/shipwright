---
name: issue-hygiene
description: Use before filing a new issue in an issue tracker, and whenever closing or updating an existing one — covers search-before-create, linking commits/PRs to issues, using the real closure reason, and keeping descriptions current. Tracker-agnostic (GitHub Issues, Linear, Jira, GitLab Issues).
---

# Issue Hygiene

## Overview

Generic policy for working with an issue tracker, deliberately tracker-
agnostic — GitHub Issues, Linear, Jira, GitLab Issues all qualify. Each
project's own CLAUDE.md states which concrete tracker is in use and how to
reach it; this skill states the policy.

## The Rules

- **Search before creating.** Check for an existing open (or recently
  closed) issue covering the same problem before filing a new one, to avoid
  duplicates accumulating over time.
- **Link work to issues.** When a commit or PR resolves an issue, reference
  it using the tracker's linking convention (e.g. GitHub's `Closes #123` /
  `Fixes #123` keywords) so closure is automatic and traceable, rather than
  closing manually with no link back to the change that fixed it.
- **Set the real closure reason**, not just "closed," when the tracker
  supports it (resolved vs. won't-fix vs. duplicate vs. not-planned) — a
  pile of issues closed with no reason recorded is no more useful than an
  open pile.
- **Keep descriptions current.** If investigation reveals the actual
  scope/root-cause differs from the original issue text, update the issue
  description to reflect current understanding rather than leaving a stale
  title/body that misleads the next reader.
- **Don't file an issue for something being fixed in the same session.**
  Issues track work that isn't actionable *right now*; filing one for a bug
  you're about to fix in the current change is process theater, not
  hygiene.
