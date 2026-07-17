# Extracting shared engineering principles from the language skills

**Status: proposal, not implemented.** This is a design decision (new
skill + edits to two existing skills) worth a real choice rather than a
unilateral call — see "Recommendation" for my pick, but all four options
below are genuinely live.

## Problem statement

An external review of the repo (a ChatGPT session given the full source
tree) flagged that `python-guidelines` and `typescript-guidelines`
independently encode the same handful of ideas in language-specific
vocabulary. Checking that claim against the actual current text (not the
review's paraphrase):

| Shared idea | `python-guidelines` says | `typescript-guidelines` says |
|---|---|---|
| Prefer structural fixes over local suppressions | "No `#noqa`... fix the violation or add a `per-file-ignores` entry" | "Never use `any` unless the object truly can be anything at runtime" (implicit in Type Safety) |
| Prefer static guarantees over runtime checks | "mypy — run in strict mode, so the type checker itself enforces... rather than relying on reviewers" | "Core principle: prefer language features... that make errors impossible rather than just unlikely" |
| Make exceptions explicit and centrally documented | "Per-File Ignores... add a `[tool.ruff.lint.per-file-ignores]` entry... Never suppress project-wide" | The namespace-merging pattern's own "when a plain discriminated union is still fine" section (documented exceptions, not scattered ones) |
| Review automated transformations before committing | "Unsafe fixes — may change semantics; ALWAYS review the diff before staging" | Not currently stated |
| Use the project's configured tools rather than bypassing them | "Run the project's configured linter and type checker" (added in the 0.5.0 pass) | Not currently stated explicitly, though implied |

Four of five ideas really are present in both, just phrased per-language.
That's a real finding, not just the reviewer's inference — I verified it
against the current text above, not the review's summary of an earlier
version.

**How urgent is this?** Not very, by volume: the actual overlapping
content is maybe 10-15 lines total across both files, out of ~155 combined.
The review itself hedges ("not yet... just something I'd watch as the
collection grows"). The real question isn't "is there duplication today"
(yes, a little) but "is this the right moment to architect for more of
it" — see Approach E below.

## Approaches

### Approach A — new `engineering-principles` skill, cross-referenced (recommended)

A short (~25-line), standalone skill stating the five principles above in
language-agnostic form. `python-guidelines` and `typescript-guidelines`
each get a one-line `**REQUIRED BACKGROUND:** shipwright:engineering-principles`
marker (the exact convention `superpowers:writing-skills` itself uses for
`superpowers:test-driven-development`) and, where a rule is a direct
instance of a principle, a short note saying so — but keep their own
language-specific content, examples, and vocabulary intact. Nothing gets
deleted from either language skill; they gain a citation, not a rewrite.

**Draft content**, ready to use if this is the chosen approach:

```markdown
---
name: engineering-principles
description: Use before writing or reviewing code in any language, or when deciding whether a new language-specific guidelines skill is justified — states the language-agnostic principles that shipwright's language skills (python-guidelines, typescript-guidelines, and any future ones) each instantiate for their own language.
---

# Engineering Principles

## Overview

These principles are language-agnostic; `shipwright:python-guidelines`,
`shipwright:typescript-guidelines`, and any future language skill each
state how their language implements them. When adding a new language
skill, check whether a rule you're about to write is actually one of these
in disguise — if so, it belongs here once, referenced by the language
skill, not restated.

## The Principles

- **Prefer structural fixes over local suppressions.** If a rule is
  inconvenient, first ask whether the code or types should change.
  Suppressions (`# noqa`, `any`, `#[allow(...)]`, `NOLINT`, ...) are the
  last resort and must remain visible and auditable — never scattered
  silently.
- **Prefer static guarantees over runtime checks.** Make invalid states
  unrepresentable at compile/check time rather than catching them at
  runtime or relying on convention.
- **Make exceptions explicit and centrally documented**, not scattered
  inline. A reader should be able to find every exception to a rule in one
  place.
- **Review every automated transformation before committing it.**
  Autofixers, codemods, IDE refactors, and language-service "organize
  imports" can change semantics; treat their output as a diff to review,
  not a fait accompli.
- **Use the project's configured analysis tools rather than bypassing
  them.** If a tool's default is wrong for a specific case, fix the tool's
  config (a scoped override), don't route around the tool.

## Why this exists

Extracted after `python-guidelines` and `typescript-guidelines` were found
to independently encode these same five ideas in language-specific
vocabulary — see `docs/design.md`. Stating them once here means a third
language skill only needs to show how *that* language's tools implement
these principles, not re-derive the principles from scratch.
```

**Testing tier:** Reference — same tier as the language skills it
supports. Verification would be a retrieval/application scenario: give a
subagent this skill plus a made-up third language's tooling (to avoid
leaking real python/typescript-guidelines content) and confirm it can
correctly identify which principle a given rule instantiates.

**Cross-reference edits to existing skills** (illustrative, not full
diffs): `python-guidelines` gains `**REQUIRED BACKGROUND:**
shipwright:engineering-principles` under its `## Philosophy` heading, and
its "No `#noqa`" bullet gains a trailing clause like "(this project's
instance of *prefer structural fixes over local suppressions*)".
Same shape for `typescript-guidelines`'s Type Safety section. Both stay
otherwise unchanged.

**Pros:**
- Matches shipwright's own established architecture — small, focused,
  cross-referenced skills (`using-shipwright` → everything else), the
  exact "modules + dispatcher" shape the review praised elsewhere.
- Scales cleanly to a 3rd/4th/5th language skill: each new one adds one
  cross-reference line, not a rewrite of a shared file.
- Neither language skill's SDO trigger description gets muddier — each
  still says precisely "Python" or "TypeScript," not "Python or
  TypeScript or whatever else lands here."
- Nothing is deleted from either existing skill, so no information loss
  and no re-verification of already-passing content is strictly required.

**Cons:**
- An 8th skill, and a new kind of skill (nothing currently in the repo is
  "infrastructure other skills cite" rather than "something `using-shipwright`
  triggers directly") — adds a category future contributors need to
  understand.
- Real but small **invocation-multiplicity cost**: if `using-shipwright`'s
  table were naively extended to also trigger `engineering-principles`
  directly on every code-writing task, that's a second skill load per
  Python/TypeScript task. The design above avoids this by making it a
  `REQUIRED BACKGROUND` of the language skill (loaded when *it* loads, not
  independently) — but this only works if that discipline is maintained
  as more language skills get added.
- For a project that only ever uses one language, this is a permanent
  extra ~25-line skill in the plugin for content that (for that project)
  never varies — mild, not really an issue given plugin skills are lazy.

### Approach B — merge into one `language-guidelines` skill (the review's literal proposal, reread carefully)

On a literal reread, the review's own proposal is closer to Approach A
than it first appears (it says the new skill should be "very short, maybe
20 lines" — too short to contain full Python/TypeScript implementation
detail, so it can't be proposing folding both files' full content in).
But a genuine Approach B exists as its own option: merge
`python-guidelines` + `typescript-guidelines` into a single
`language-guidelines` skill, with a shared "Philosophy" section up top
followed by `## Python` and `## TypeScript` subsections.

**Pros:** one file, one version-bump story, no cross-reference to
maintain, easiest for a human to skim start-to-finish.

**Cons:** the file would run ~150+ lines combining both languages' full
content — the same "catalogue" shape the review explicitly warned
`tool-preferences` against becoming ("if it grows into a long list...
it becomes a catalogue rather than guidance"), just applied to languages
instead of tool roles. A single trigger description covering two
unrelated languages is inherently less precise (SDO explicitly wants
triggers scoped to one condition, not "Python or TypeScript"). Worst of
all, this doesn't scale: a 3rd language makes the file bigger and the
trigger description muddier, exactly backwards from what you'd want as
the collection grows — this is the shape the review's own architectural
instincts (stated elsewhere in the same review) argue against.

### Approach C — lateral cross-references, no new skill

Don't add a principles layer at all. Just have `python-guidelines` and
`typescript-guidelines` note where they parallel each other — e.g.
python-guidelines's `#noqa` rule gets a footnote "(see
`typescript-guidelines`'s `any` rule for the same principle in
TypeScript)." Zero new skills, minimal edit.

**Pros:** cheapest possible change, zero new architecture to learn,
addresses "are they consistent" directly by pointing at each other.

**Cons:** doesn't deliver the benefit the review specifically flagged as
valuable — a common foundation future language skills can cite without
re-deriving. Cross-references between N peer skills grow O(n²) (every
skill needs to know about every other one) instead of O(n) (every skill
cites one shared skill), so this gets worse, not better, exactly as more
languages are added. Weakest option for the stated goal of "does this new
rule instantiate a Shipwright principle, or is it just a personal
preference" — there's no single place to check that question against.

### Approach E — do nothing yet, revisit at the 3rd language skill

Explicitly decline to act now. The actual duplication today is small
(10-15 lines), the review itself says "not yet," and extraction always
costs at least one round of edits + a version bump + verification,
whether done now or later.

**Pros:** avoids speculative architecture for a problem that's currently
minor; genuinely defensible given the review's own hedge.

**Cons:** retrofitting later, after a 3rd/4th language skill already
exists with its own independently-phrased version of these principles,
means editing *more* files at once than doing it now (2 skills today vs.
3-4 later) — "cheaper now than later" is a real, not hypothetical, cost
difference, since Approach A's edits to existing skills are additive
(citations) rather than rewrites either way.

## Recommendation

**Approach A**, but the honest case for it is "cheap and clean, not
urgent" rather than "solves a pressing problem." The actual trigger for
doing this *now* rather than waiting (Approach E) is that it's strictly
cheaper to extract from 2 language skills than from 3+, and Approach A's
edits to the existing skills are additive (a `REQUIRED BACKGROUND` line +
short annotations), not rewrites — so there's little downside to doing it
early versus a real, if modest, upside (each future language skill costs
one cross-reference line instead of re-deriving the same five ideas).

If the honest answer is "this doesn't feel worth touching right now,"
Approach E is a perfectly defensible choice too — nothing about the
current state is broken, and the review's own framing supports waiting.

## If approved: rollout sketch

1. Brainstorm/confirm naming (`engineering-principles` is my working name;
   alternatives: `language-agnostic-principles`, `static-safety-principles`).
2. Write `skills/engineering-principles/SKILL.md` (draft above), reference tier.
3. Add `REQUIRED BACKGROUND` + short annotation lines to `python-guidelines`
   and `typescript-guidelines` (additive edits, not rewrites).
4. Add `shipwright:engineering-principles` to `using-shipwright`'s
   required-invocation table? — **open question, see below.**
5. Verify with a retrieval/application scenario (reference tier).
6. Update `docs/design.md`'s skill table and skill count (7 → 8).
7. Minor version bump (new skill).

## Open questions for you

1. Approach A, E, or something else?
2. If A: does `using-shipwright`'s table gain a row for
   `engineering-principles`, or does it stay purely a `REQUIRED BACKGROUND`
   of the language skills (my draft assumes the latter, to avoid double
   invocation — but it's worth confirming that's the right call)?
3. If A: is `engineering-principles` the right name, or does one of the
   alternatives read better as a skill someone would search for?
