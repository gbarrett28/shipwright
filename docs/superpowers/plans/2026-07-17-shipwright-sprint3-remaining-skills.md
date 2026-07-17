# Shipwright Sprint 3: Remaining Skills Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Ship the four remaining shipwright skills — `issue-hygiene`, `tool-preferences` (pattern tier), `python-guidelines`, `typescript-guidelines` (reference tier) — completing the full skill set from `docs/superpowers/specs/2026-07-17-shipwright-plugin-design.md`.

**Architecture:** Four independent `SKILL.md` files. Pattern/reference tier per `CLAUDE.md`'s Skill Authoring rules: lighter application/retrieval-scenario testing, no RED baseline required (unlike Sprint 1/2's discipline-enforcing skills) — one verification scenario per skill, run together in a single subagent call for efficiency since none of them are adversarial/pressure scenarios.

**Tech Stack:** Markdown (skills), JSON (version bump), git/GitHub.

## Global Constraints

- Repo: `C:\Users\geoff\PycharmProjects\shipwright`, current version `0.3.0`.
- `python-guidelines` and `typescript-guidelines` are extracted verbatim from `killer_sudoku`'s current `CLAUDE.md` (`Python Coding Guidelines` / `TypeScript Coding Guidelines` sections, lines 196–331 as of this sprint), with one generalization: the TypeScript guideline's reference to "serena's `find_referencing_symbols`" becomes a generic "a reference-finding tool (e.g. serena's `find_referencing_symbols`, or your language server's find-references)" — since this skill must work for projects without serena installed. No other content changes.
- `issue-hygiene` and `tool-preferences` are drawn from the design spec's already-approved bullet lists (net-new content for `issue-hygiene`; extracted/generalized from killer_sudoku's "Agent Protocol: Tool Use" / "PR Review Tools" / "Library Documentation" / "TypeScript Language Server" sections for `tool-preferences`).
- One combined version bump at the end: `0.3.0` → `0.4.0` (minor, four new skills shipped as one batch).

---

### Task 1: Write all four skill files

**Files:**
- Create: `skills/issue-hygiene/SKILL.md`
- Create: `skills/tool-preferences/SKILL.md`
- Create: `skills/python-guidelines/SKILL.md`
- Create: `skills/typescript-guidelines/SKILL.md`

- [x] **Step 1: Write `skills/issue-hygiene/SKILL.md`**

```markdown
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
```

- [x] **Step 2: Write `skills/tool-preferences/SKILL.md`**

```markdown
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
  running.
- **Prefer specialized review-agent plugins** (e.g. `pr-review-toolkit`,
  `coderabbit`) over ad hoc manual review when installed.
- **Prefer up-to-date library-documentation tools** (e.g. `context7`) over
  training-data knowledge for external library/API usage, when installed.

## Fallback behavior

If a project's designated tool for a role is unavailable (not installed,
not responding), don't silently fall back to a generic tool and continue —
ask the user to check the plugin/restart it first. Silent fallback hides a
broken tool setup behind degraded results that look normal.
```

- [x] **Step 3: Write `skills/python-guidelines/SKILL.md`**

```markdown
---
name: python-guidelines
description: Use before writing or reviewing Python code — covers strict ruff/mypy linting rules, the no-inline-noqa/type-ignore policy (use per-file-ignores or stubs instead), and the safe-then-unsafe auto-fix workflow.
---

# Python Coding Guidelines

## Linting and Type Checking

Python code is checked at every bronze gate with:

```bash
python -m ruff check .          # enforced in CI and pre-commit
python -m mypy . --ignore-missing-imports
```

**Rules:**

- **ruff** — strict extended rule-set (see `[tool.ruff]` in `pyproject.toml`). Zero tolerance.
- **mypy** — `strict = true`. All public functions require full type annotations.
- **No `#noqa`** — never add inline `# noqa: …` comments. Fix the violation or add a `per-file-ignores` entry with a comment explaining why.
- **No inline `# type: ignore`** — use `cast()`, fix the stub, or add a `per-file-ignores` entry instead.
- **Stubs for missing third-party types** — create `stubs/<pkg>/` with `.pyi` files rather than adding `# type: ignore[import-untyped]` per-import.

## Auto-fix Workflow

Before manually fixing violations, always try auto-fix first:

```bash
# 1. Safe fixes only (always run first)
python -m ruff check --fix .

# 2. Unsafe fixes — may change semantics; ALWAYS review the diff before staging
python -m ruff check --unsafe-fixes --fix .
git diff                         # review every change before git add
```

The `--unsafe-fixes` flag enables transforms that could alter runtime behaviour
(e.g. `try/except/pass` → `contextlib.suppress`, nested `if` → single `and`).
They are often correct but must be verified in context.

## Per-File Ignores

When a rule conflicts with a legitimate domain convention (e.g. N803/N806 for
ML matrix naming `X`, T20 for CLI scripts, PLW0603 for module-level singletons),
add a `[tool.ruff.lint.per-file-ignores]` entry in `pyproject.toml` with a
comment explaining the reason. Never suppress project-wide.
```

- [x] **Step 4: Write `skills/typescript-guidelines/SKILL.md`**

```markdown
---
name: typescript-guidelines
description: Use before writing or reviewing TypeScript code — covers safety-by-construction philosophy, the namespace-merging pattern for discriminated unions (vs bare unions), type-safety rules, code hygiene, and error-handling rules.
---

# TypeScript Coding Guidelines

## Design Philosophy: Safety By Construction

**Core principle:** Prefer language features and structures that make errors **impossible** rather than just **unlikely**.

- **Type system:** Make invalid states unrepresentable through strong typing; prefer `readonly` arrays and tuples
- **Iteration:** Use `for...of` and destructuring to couple related variables; avoid raw index loops unless necessary
- **Configuration:** Single source of truth — no magic numbers scattered through code
- **Error handling:** Surface errors to the user unless there is a clear automatic resolution

## Self-Documenting Code

- Keep JSDoc comments up to date; tiered: short summary first, then detail
- Inline comments should explain WHY or WHAT, not HOW (mechanics are visible in the code)

## OO Over Discriminated Unions

**Strong preference: use the namespace-merging pattern instead of bare discriminated unions** whenever a type has per-variant behaviour.

### Why

A bare discriminated union (`type Foo = A | B | C`) scatters behaviour into switch statements elsewhere in the codebase. The compiler cannot enforce that every variant is handled in every dispatch site. The result is:

- Silent fallthrough (`default: return state`) — new variants do nothing, no compile error
- External metadata that drifts (`Set<string>`, `string[]` tracking which variants have a property)
- Behaviour spread across multiple files, far from the type definition

### The namespace-merging pattern

Keep the type as plain data (serialisable, no class instances) and put per-variant static methods in a same-name namespace:

```typescript
// Each variant is a named interface + namespace
export interface PlaceDigitAction {
  readonly type: 'placeDigit';
  readonly row: number; readonly col: number; readonly digit: number;
}
export namespace PlaceDigitAction {
  export function apply(a: PlaceDigitAction, state: PuzzleState): PuzzleState { ... }
}

// The union gets its own namespace with an exhaustiveness guard
export type UserAction = PlaceDigitAction | RemoveDigitAction | ...;
export namespace UserAction {
  export function apply(action: UserAction, state: PuzzleState): PuzzleState {
    switch (action.type) {
      case 'placeDigit': return PlaceDigitAction.apply(action, state);
      // ...
      default: assertNeverAction(action);  // compile error on missing case
    }
  }
}
```

The compiler now **enforces** that every variant defines all required methods. Adding a new variant without updating every dispatch function is a type error.

### When a property belongs on the type itself

If code elsewhere maintains a `Set<string>` or `string[]` to track which variants of a type have a given property — that is a red flag. The property belongs on the type. Example: `CLASSIC_EXCLUDED_RULES: Set<string>` should be `rule.killerOnly: boolean` on `SolverRule`.

### When a plain discriminated union is still fine

- Very small, stable unions (2–3 variants) with no per-variant behaviour
- Pure structural narrowing with no dispatch (e.g. `type GitHubAction = CommentAction | IssueAction`)
- When all dispatch is in a single, focused location and the union will never grow

### Warning signs to refactor

- Any `default: return x` (silent fallthrough) in a switch over a union discriminant
- An external `Set<string>` / `string[]` tracking which variants of a type have a property
- More than one `switch` / `if-else` chain dispatching on the same discriminant across the codebase
- Behaviour relevant to a type variant living in a different file from the type definition

## Type Safety

- Always use the strongest possible return type annotation
- Always use the weakest possible parameter type annotation
- Never use `any` unless the object truly can be anything at runtime
- Prefer `unknown` over `any` for external data; narrow explicitly

## Code Hygiene

- All `import` statements at the top of the file — no dynamic/inline imports
- No `* as` star imports — name every symbol explicitly
- Before removing code, verify it is unused via a reference-finding tool (e.g. serena's `find_referencing_symbols`, or your language server's find-references)

## Error Handling

- Surface exceptions unless there is a clear way to resolve them automatically
- Catch only for graceful degradation; always log or rethrow otherwise
```

---

### Task 2: Verify all four skills with application/retrieval scenarios

**Files:** none created (verification step).

- [x] **Step 1: Dispatch one combined verification subagent**

Use the `Agent` tool (`subagent_type: general-purpose`, foreground) with
all four skills' content pasted in, and four scenario questions:

```
You have these four house-style references available (treat them as
binding project policy, not suggestions):

--- shipwright:issue-hygiene ---
<paste skills/issue-hygiene/SKILL.md>

--- shipwright:tool-preferences ---
<paste skills/tool-preferences/SKILL.md>

--- shipwright:python-guidelines ---
<paste skills/python-guidelines/SKILL.md>

--- shipwright:typescript-guidelines ---
<paste skills/typescript-guidelines/SKILL.md>

Answer all four scenarios concretely and separately:

1. You're about to close issue #482 after merging its fix. Walk through
   what you actually do.

2. You need to find every place a function called `calculateDiscount` is
   called before removing it. A semantic code-navigation MCP tool (serena)
   is installed and running. What tool do you use, and why?

3. You wrote a Python function using ML matrix-naming conventions
   (variables named `X` and `y`); ruff flags N803 (argument name should be
   lowercase). What do you do?

4. You're adding a new variant to a TypeScript discriminated union
   `type UserAction = PlaceDigitAction | RemoveDigitAction`, where each
   variant has different apply-to-state behavior. Bare union with a switch
   statement, or the namespace-merging pattern? Why?
```

- [x] **Step 2: Record and evaluate**

Append the response under a `## Verification result` heading in this plan
file. Confirm: (1) mentions the real closure reason + linking convention,
not just "close it"; (2) names serena/semantic tool over grep/Read; (3)
adds a `per-file-ignores` entry with a comment, not an inline `# noqa`;
(4) chooses namespace-merging with reasoning about per-variant behavior
and exhaustiveness, not a bare union. If any answer is wrong, revise the
corresponding skill and re-run this step once. If all four are correct,
proceed to Task 3.

## Verification result

All four passed:

1. **issue-hygiene**: verified auto-close via linking keyword rather than
   assuming, used `gh issue close 482 --reason completed` with a real
   closure reason, cited updating the description if root cause differed,
   and explicitly noted not filing new issues for same-session fixes.
2. **tool-preferences**: named `mcp__plugin_serena_serena__find_referencing_symbols`
   specifically, explicitly rejected falling back to `Grep` due to false-
   positive/negative risk on text matching.
3. **python-guidelines**: added a `per-file-ignores` entry scoped to the
   specific file with an explanatory comment, explicitly rejected inline
   `# noqa`, kept the exemption narrow rather than project-wide.
4. **typescript-guidelines**: chose namespace-merging, correctly identified
   per-variant behavior as the trigger condition, cited silent-fallthrough
   risk, and produced a correct code example with exhaustiveness guard.

**All four pass. No revisions needed — proceeding to Task 3.**

---

### Task 3: Validate frontmatter, version bump, finalize

**Files:**
- Modify: `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json` (version bump to `0.4.0`)

- [x] **Step 1: Validate all four skills' frontmatter**

```bash
node -e "
const fs = require('fs');
for (const f of [
  'skills/issue-hygiene/SKILL.md',
  'skills/tool-preferences/SKILL.md',
  'skills/python-guidelines/SKILL.md',
  'skills/typescript-guidelines/SKILL.md',
]) {
  const text = fs.readFileSync(f, 'utf8');
  const m = text.match(/^---\n([\s\S]*?)\n---/);
  if (!m) throw new Error(f + ': no frontmatter block found');
  if (!/^name: /m.test(m[1])) throw new Error(f + ': missing name field');
  if (!/^description: /m.test(m[1])) throw new Error(f + ': missing description field');
  if (m[0].length > 1024) throw new Error(f + ': frontmatter exceeds 1024 chars');
  console.log(f + ': OK');
}
"
```

- [x] **Step 2: Bump version to 0.4.0 in both manifest files, validate JSON**

```bash
node -e "JSON.parse(require('fs').readFileSync('.claude-plugin/plugin.json','utf8')); JSON.parse(require('fs').readFileSync('.claude-plugin/marketplace.json','utf8')); console.log('OK')"
```

- [x] **Step 3: Commit and push**

```bash
git add skills/issue-hygiene/SKILL.md skills/tool-preferences/SKILL.md skills/python-guidelines/SKILL.md skills/typescript-guidelines/SKILL.md .claude-plugin/plugin.json .claude-plugin/marketplace.json docs/superpowers/plans/2026-07-17-shipwright-sprint3-remaining-skills.md
git commit -m "feat: add issue-hygiene, tool-preferences, python-guidelines, typescript-guidelines; bump to 0.4.0

Completes the full shipwright skill set from the design spec. Pattern/
reference-tier skills, verified with application/retrieval scenarios per
CLAUDE.md's testing-tier rules (lighter than Sprint 1/2's discipline-
enforcing RED-GREEN-REFACTOR cycle)."
git push
```

---

## Sprint 3 exit note

This completes shipwright's full skill set (7 skills total). The only
remaining work from the original design spec is Sprint 3 there /
"Sprint 4" here in practice: restructuring `killer_sudoku`'s own CLAUDE.md
to install and reference this plugin — a `killer_sudoku`-side plan, to be
written there.
