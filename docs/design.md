# Design

## Why this exists

`shipwright` was extracted from a single project's `CLAUDE.md`
(`killer_sudoku`), which had accumulated two kinds of content: process rules
that had nothing to do with that project's domain (which skill to invoke
and when, the two-tier quality-gate pattern, branch/doc/commit conventions,
even language coding guidelines), and genuinely project-specific
configuration (what the codebase is, domain conventions, concrete tool
wiring). When a second, unrelated project started, the choice was between
copy-pasting the methodology prose into a new `CLAUDE.md` and letting the
two drift, or extracting it once into something both projects — and any
future one — install and reference. `shipwright` is that extraction,
packaged as a standalone Claude Code plugin rather than a shared file,
specifically so it can be versioned, installed via the normal plugin
mechanism, and opened to outside collaborators.

## Structure

A single-plugin marketplace repo, matching the shape of `superpowers` and
`pr-review-toolkit`:

```
shipwright/
  .claude-plugin/
    marketplace.json   # lists this repo as a marketplace with one plugin: shipwright
    plugin.json         # plugin manifest (name, version, description)
  .claude/
    settings.json        # self-referential install config (see "Dogfooding" below)
  skills/
    using-shipwright/SKILL.md
    quality-gates/SKILL.md
    commit-and-test-integrity/SKILL.md
    issue-hygiene/SKILL.md
    tool-preferences/SKILL.md
    python-guidelines/SKILL.md
    typescript-guidelines/SKILL.md
  CLAUDE.md             # skill-authoring process for contributors
  README.md
```

Consuming projects install it via the standard plugin flow
(`/plugin marketplace add` + `/plugin install shipwright@shipwright`), or —
for portability across clones — by committing an `extraKnownMarketplaces` +
`enabledPlugins` block in their own `.claude/settings.json` pointing at the
GitHub source, the same way this repo does for itself.

## The eight skills

Each skill was assigned a testing tier during brainstorming, before being
written, because the tier determines how much pressure-testing rigor it
needs (see "Testing methodology" below). The tiers describe different
*kinds* of guidance, not just different rigor levels:

- **Discipline-enforcing** skills govern behavior — a rule the agent must
  keep following even under time pressure, sunk cost, or a plausible-
  sounding rationalization. These are the ones tested adversarially,
  because that's the failure mode they exist to resist.
- **Pattern** skills encode a reusable engineering approach — a way of
  thinking about a class of problem, applied per situation rather than
  followed as a strict rule under pressure.
- **Reference** skills capture mature, already-proven project conventions —
  what to do, not a discipline to defend. Extracted from real enforced
  practice, not invented preferences (see the language-skill inclusion
  criterion below).

| Skill | Tier | Covers |
|---|---|---|
| `using-shipwright` | Discipline-enforcing | Required-skill-invocation table for the other six skills; the "Check Before You Build" rule (search skills/plugins/prior work before building from scratch — see below); token-efficiency and plan-sprint-size guidance; the git-worktree caveat |
| `quality-gates` | Discipline-enforcing | The bronze/silver two-tier gate contract; keeping newly-written TDD tests inside the bronze run without gaming the fast-subset tag; the bronze/silver split of doc-hygiene checks |
| `commit-and-test-integrity` | Discipline-enforcing | Conventional Commits; confirm-before-deleting-uncommitted-work; tests-define-the-spec (a failing test means suspect the implementation first, never silently edit the test) |
| `issue-hygiene` | Pattern | Tracker-agnostic issue hygiene: search-before-create, link commits/PRs to issues, real closure reasons, keep descriptions current |
| `tool-preferences` | Pattern | Prefer semantic code-navigation, specialized review agents, and up-to-date library docs over generic tools/training data, when installed — with an explicit non-silent fallback rule |
| `engineering-principles` | Reference | The five language-agnostic principles both language skills instantiate: structural fixes over suppressions, static over runtime guarantees, centralized/explicit exceptions, review automated transformations, use configured tools rather than bypassing them |
| `python-guidelines` | Reference | Strict ruff/mypy rules, no inline `noqa`/`type: ignore`, the safe-then-unsafe auto-fix workflow |
| `typescript-guidelines` | Reference | Safety-by-construction philosophy, the namespace-merging pattern for discriminated unions, type safety, code hygiene, error handling, tooling (added alongside `engineering-principles` — see below) |

`python-guidelines` and `typescript-guidelines` are extracted near-verbatim
from `killer_sudoku`'s enforced conventions (the one adaptation:
TypeScript's "before removing code" rule was generalized from a
serena-specific instruction to "a reference-finding tool (e.g. serena's
`find_referencing_symbols`, or your language server's find-references)",
since this skill has to work for projects without serena installed).
`issue-hygiene` is net-new — `killer_sudoku` had no issue-tracker
conventions to extract from.

**`engineering-principles` exists because comparing the two language
skills against each other found a real gap, not just duplication.**
An external review flagged that `python-guidelines` and
`typescript-guidelines` encode overlapping ideas in language-specific
vocabulary. Checking that against the actual text found something sharper
than redundancy: two principles — "review every automated transformation
before committing" and "use the project's configured tools rather than
bypassing them" — existed in `python-guidelines` and were simply absent
from `typescript-guidelines`, along with no TypeScript equivalent of "no
`#noqa`" for `@ts-ignore`/`eslint-disable`. Not because those ideas don't
apply to TypeScript — ESLint autofixes and IDE refactors carry the same
risk `ruff --fix` does — but because nobody had checked the two skills
against a shared list before. `engineering-principles` states the five
principles once; `python-guidelines` and `typescript-guidelines` each
cite it via `REQUIRED BACKGROUND` and note which of their rules
instantiate which principle. Verified with a retrieval/application
scenario deliberately run against Rust — a language neither skill covers —
to confirm the principles generalize past the two languages they were
extracted from, not just describe them after the fact.

**Language-skill inclusion criterion, stated explicitly:** a language gets
a `*-guidelines` skill because a real project's already-enforced
conventions were extracted into it — not because the language is popular
or because someone had opinions about it. Python and TypeScript are the
only two languages covered because those are the only two this repository
has actually extracted from so far. The absence of a `rust-guidelines` or
`go-guidelines` is not a judgment about those languages — it just means
nobody has extracted one yet. Adding one requires the same bar: a real,
already-enforced project convention to extract, not an invented style
guide (see the deferred `html-css-guidelines` skill below for a case where
this bar wasn't met and the skill was skipped as a result).

## Testing methodology

Skill-authoring here follows `superpowers:writing-skills`'s TDD-for-docs
model: a skill isn't written until a baseline (RED) test shows what an
agent does *without* it, then the skill is written to address that gap
(GREEN), then re-verified. Rigor scales by tier:

- **Discipline-enforcing** skills got full RED-GREEN pressure-scenario
  testing via the `Agent` tool — a realistic scenario combining time
  pressure, sunk cost, and a plausible-sounding rationalization, run once
  without the skill and once with it.
- **Pattern/reference** skills got lighter application/retrieval-scenario
  testing — a single realistic question per skill, run together in one
  subagent call, checking the answer applies the rule correctly.

Two things fell out of that process worth recording as design decisions,
not just history:

1. **The RED tests for `quality-gates` and `commit-and-test-integrity`
   found no baseline failures.** Subagents dispatched from within the
   `killer_sudoku` session inherit its `CLAUDE.md` as ambient context
   regardless of prompt framing, and — since the equivalent rules hadn't
   been stripped out of `killer_sudoku`'s `CLAUDE.md` yet — those subagents
   already had the real rule available and applied it correctly. Rather
   than invent rationalization-table entries for failures that didn't
   occur (which `writing-skills` explicitly warns against), those skills
   ship as accurate reference content plus whatever narrower claims *did*
   fail testing.
2. **`using-shipwright`'s "Check Before You Build" section exists because
   of a real failure, not a hypothetical one.** While building this very
   plugin, a session went straight into hand-drafting all seven `SKILL.md`
   files from scratch despite `superpowers:writing-skills` (and a separate
   `skill-creator` plugin) being installed and directly on-point — caught
   only because the user asked whether such a skill existed, not because
   the agent checked proactively. That's documented verbatim in the skill
   itself as the concrete example the rule exists to prevent.

## Versioning

`version` in `.claude-plugin/plugin.json` and the matching entry in
`.claude-plugin/marketplace.json` moves together: `0.1.0` at scaffold,
`0.2.0` after `using-shipwright` + `quality-gates`, `0.3.0` after
`commit-and-test-integrity`, `0.4.0` after the remaining four skills
shipped as one batch, `0.4.1` for a content-completeness fix (the
superpowers required-invocation table and a missing doc-convention row),
`0.5.0` for a round of external-review-driven refinements (new rules in
`issue-hygiene`/`tool-preferences`, the AI-attribution policy softened
from mandatory to project-configurable, wording clarifications across
`quality-gates`/`python-guidelines`), `0.6.0` for the new
`engineering-principles` skill plus the TypeScript gap it found (tooling
and suppression-discipline rules added to `typescript-guidelines`).
Consuming projects pin to the
marketplace, not a
specific version, so a real behavior change without a version bump is
invisible to them — every skill addition gets at least a minor bump.

## Dogfooding

This repo's own `.claude/settings.json` registers and enables `shipwright`
for itself via the GitHub source (not the local directory path used during
initial development, which isn't portable to a fresh clone). Without this,
contributors working on `shipwright` would be following its own rules from
memory rather than through genuine skill invocation — which is exactly what
happened during initial development, before this was set up.

## Deliberately out of scope

- **No bronze/silver gate script or CI for this repo.** `shipwright` is
  markdown and JSON, not application code with an automated test suite;
  skill correctness is verified through the RED-GREEN-REFACTOR process
  above, not a script. `CLAUDE.md` states this explicitly so it doesn't
  read as an oversight.
- **No branch/PR workflow yet.** All commits through `v0.4.0` went directly
  to `main` — this is solo bootstrapping, not the intended long-term
  pattern. `CLAUDE.md` already asks that significant additions be
  discussed in an issue before a PR once there are outside contributors.
- **An `html-css-guidelines` skill** was considered and deferred: unlike
  the Python/TypeScript guidelines, there was no existing enforced
  HTML/CSS convention to extract from, and inventing one with no proven
  usage risks baking untested preferences into a shared plugin. Add it if
  a project surfaces real, already-enforced conventions worth
  generalizing.
