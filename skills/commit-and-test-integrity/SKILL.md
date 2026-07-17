---
name: commit-and-test-integrity
description: Use when writing a commit message, when about to delete or change anything not committed to git, and whenever a test fails after a code change — covers Conventional Commits format and the tests-define-the-spec rule (a failing test means suspect the implementation first, never silently edit the test).
---

# Commit and Test Integrity

## Commit Conventions

- Follow Conventional Commits format: `feat:`, `fix:`, `chore:`, `docs:`,
  `refactor:`, `test:`.
- Write clear, descriptive commit messages focused on *why*, not *what*.
- **Always confirm before deleting or changing anything that is not
  committed to git.**

### AI attribution

Whether AI-assisted commits carry an attribution trailer (e.g.
`Co-Authored-By: <model name> <noreply@anthropic.com>`) is a project/org
policy decision, not a Shipwright mandate — some organizations require it,
some prohibit it, most don't have a policy at all. Follow the project's own
CLAUDE.md if it states one; if it's silent, ask rather than assuming either
default.

## Test Specification Integrity

Tests define the specification for each module.

When a test fails after a code change:
1. **Assume the implementation is wrong, not the test.**
2. If you believe the test itself is wrong — even when you're confident
   the new behavior is correct — you MUST:
   - Document the spec change in detail (what changed and why).
   - Get explicit user approval for the spec change.
   - Update the test with clear comments explaining the change.
   - **Never** silently modify a test to make it pass.

Confidence that your implementation is right is not the same as having
checked. "I'm sure this is correct" and "the test is now wrong" are two
separate claims — the second one needs the approval step above regardless
of how justified the first one is. Deadline pressure doesn't change this:
documenting a spec change and getting a one-line approval costs seconds,
not minutes.

### Red Flags

- "I'm confident my change is correct, the test must just be outdated."
- "I'll update the assertion to match the new behavior and move on."
- "There's no time to write up a spec-change note right now."

**All of these mean: stop, document the proposed spec change, and get
approval before touching the test.**
