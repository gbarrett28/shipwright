---
name: engineering-principles
description: Use before writing or reviewing code in any language, or when deciding whether a new language-specific guidelines skill is justified — states the language-agnostic principles that shipwright's language skills (python-guidelines, typescript-guidelines, and any future ones) each instantiate for their own language.
---

# Engineering Principles

## Overview

These principles are language-agnostic; `shipwright:python-guidelines`,
`shipwright:typescript-guidelines`, and any future language skill each
state how their language implements them. When adding or reviewing a
language skill, check each principle against it explicitly — a principle
with no stated instantiation in a given language is a gap, not evidence
the principle doesn't apply there. That's how this skill itself got
written: comparing `python-guidelines` and `typescript-guidelines` against
each other found two principles Python had instantiated and TypeScript
had simply never stated, not because they didn't apply to TypeScript but
because nobody had checked.

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
  not a fait accompli. This applies as much to `eslint --fix` or a
  TypeScript-service refactor as to `ruff --fix` — the risk isn't
  language-specific, only the tool name is.
- **Use the project's configured analysis tools rather than bypassing
  them.** If a tool's default is wrong for a specific case, fix the tool's
  config (a scoped override), don't route around the tool.
- **Prefer iteration over indexing.** Index-based loops — manual position
  tracking, indexing two collections by one shared counter — are exactly
  where off-by-one errors, index-out-of-bounds errors, and shape-mismatch
  bugs (assuming two collections share a length and indexing both by the
  same counter) come from. Prefer iterating a sequence directly, or
  composing iterator/iterable primitives (Python's `itertools`,
  TypeScript's `for...of` and array methods, ...), over manual position
  arithmetic. Reach for indexing only when the algorithm is genuinely
  random-access (e.g. binary search).

## Checking a language skill against these principles

For each principle, the language skill should state either how the
language's tooling implements it, or explicitly that it doesn't apply and
why. "Not mentioned" isn't a valid third state — it just means nobody
checked yet.

## Why this exists

`python-guidelines` and `typescript-guidelines` were found to independently
encode most of these same ideas in language-specific vocabulary — and, on
closer comparison, two of them (review automated transformations, use
configured tools) existed only in `python-guidelines`, with no TypeScript
equivalent stated at all. That's the concrete failure this skill exists to
prevent: not just duplicated wording, but a principle quietly present in
one language skill and absent from another with nobody having noticed.
Stating the principles once here means a third language skill only needs
to show how *that* language's tools implement them — and reviewing a
language skill against this list surfaces gaps like the TypeScript one
before they sit unnoticed. See `docs/design.md` for the full history.
