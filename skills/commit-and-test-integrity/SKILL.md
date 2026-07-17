---
name: commit-and-test-integrity
description: Use when writing a commit message, when about to delete or change anything not committed to git, whenever a test fails after a code change, and when a deliverable such as a brainstorming design spec is ready for the user to review — covers Conventional Commits format, the tests-define-the-spec rule (a failing test means suspect the implementation first, never silently edit the test), and publishing a deliverable (push + direct link, not just a local commit).
---

# Commit and Test Integrity

## Publishing Deliverables for Review

A deliverable — a design spec from `superpowers:brainstorming`, or
anything else you're about to tell the user is "ready for review" — is
not done when it exists in your local working copy. It's done when the
user can actually reach it without asking you where it is.

**The rule, independent of which VCS is in play:** commit it, publish
it to wherever the user actually looks, and hand them a direct
reference to the exact thing to look at. A commit sitting only in your
local checkout is invisible to anyone outside that checkout — including
the user, who is not necessarily working from your working copy.

**With Git + GitHub, concretely:** `git commit` → `git push` (creating
the remote branch if it doesn't exist yet) → state the direct file URL:
`https://github.com/<owner>/<repo>/blob/<branch>/<path>`. Saying "it's
committed, please review the file" without pushing, or without giving
that URL, leaves the user unable to act on what you just said — they'd
have to ask where it is, which is the exact round-trip this rule exists
to avoid.

### Red Flags

- "I've committed it, that's done." — Committed-but-unpushed is invisible outside your own checkout; the deliverable isn't reachable yet.
- "I'll just tell them the file path." — A local path doesn't help someone who wants to open it in a browser or share it with someone else. Give the actual remote URL.
- "They can find it, it's in the repo." — Don't make the user go looking for something you just said was ready. Hand them the link.

**All of these mean: push, then state the exact URL, before calling the work done.**

## Commit Conventions

- Follow Conventional Commits format: `feat:`, `fix:`, `chore:`, `docs:`,
  `refactor:`, `test:`. A consistently-typed history is scannable —
  `git log --grep '^feat'` or a changelog generator can answer "what
  shipped" without anyone reading every diff.
- Write clear, descriptive commit messages focused on *why*, not *what*.
  The diff already shows *what* changed; a message that just re-describes
  it adds nothing a reader couldn't get faster from `git show`. The *why*
  is the only thing that isn't already in the diff.
- **Always confirm before deleting or changing anything that is not
  committed to git.** Committed work can be recovered from history;
  uncommitted work can't — the cost of asking is one round-trip, the cost
  of guessing wrong is unrecoverable.

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
