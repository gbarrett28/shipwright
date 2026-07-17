---
name: python-guidelines
description: Use before writing or reviewing Python code — covers strict ruff/mypy linting rules, the no-inline-noqa/type-ignore policy (use per-file-ignores or stubs instead), the safe-then-unsafe auto-fix workflow, and preferring iteration/itertools over index-based array access.
---

# Python Coding Guidelines

**REQUIRED BACKGROUND:** `shipwright:engineering-principles` — this skill
is Python's instantiation of those language-agnostic principles.

## Philosophy

Prefer designs that let Ruff and mypy prove correctness rather than
requiring human discipline to remember the rule every time. Everything
below follows from that: tooling catches what memory won't.

## Linting and Type Checking

Run the project's configured linter and type checker at every bronze gate
(this project's instance of *use the project's configured analysis tools
rather than bypassing them*) — for example:

```bash
python -m ruff check .          # enforced in CI and pre-commit
python -m mypy . --ignore-missing-imports
```

(The exact commands are this project's config; the requirement is that
*some* strict linter and type checker run every bronze gate, not these
specific invocations.)

**Rules:**

- **ruff** — strict extended rule-set (see `[tool.ruff]` in `pyproject.toml`). Zero tolerance.
- **mypy** — run in strict mode, so the type checker itself enforces full annotation coverage on public functions rather than relying on reviewers to notice a missing one.
- **No `#noqa`** — never add inline `# noqa: …` comments. Fix the violation or add a `per-file-ignores` entry with a comment explaining why (*prefer structural fixes over local suppressions*; *make exceptions explicit and centrally documented*).
- **No inline `# type: ignore`** — use `cast()`, fix the stub, or add a `per-file-ignores` entry instead.
- **Stubs for missing third-party types** — create `stubs/<pkg>/` with `.pyi` files rather than adding `# type: ignore[import-untyped]` per-import.

## Auto-fix Workflow

Before manually fixing violations, always try auto-fix first — for example:

```bash
# 1. Safe fixes only (always run first)
python -m ruff check --fix .

# 2. Unsafe fixes — may change semantics; ALWAYS review the diff before staging
python -m ruff check --unsafe-fixes --fix .
git diff                         # review every change before git add
```

(*Review every automated transformation before committing it.*)

The `--unsafe-fixes` flag enables transforms that could alter runtime behaviour
(e.g. `try/except/pass` → `contextlib.suppress`, nested `if` → single `and`).
They are often correct but must be verified in context.

## Iteration Over Indexing

Prefer iterating directly over a sequence, or composing iterators via
`itertools`, rather than indexing into arrays/lists by position.
Index-based loops (`for i in range(len(x)): x[i]`) are exactly where
off-by-one errors, index-out-of-bounds errors, and shape-mismatch bugs
(assuming two sequences share a length and indexing both with one shared
index) come from — an iterator can't be indexed out of bounds because it
never exposes a position to get wrong.

Concretely, prefer:

- `for item in seq:` over `for i in range(len(seq)): seq[i]`.
- `for a, b in zip(xs, ys):` over `for i in range(len(xs)): xs[i], ys[i]`
  — `zip` (or `itertools.zip_longest` when lengths may genuinely differ
  on purpose) makes a length mismatch impossible to get wrong instead of
  asserting the lengths match and then indexing both by hand.
- `itertools.pairwise(seq)` over `for i in range(len(seq) - 1): seq[i], seq[i + 1]`.
- `enumerate(seq)` when the position is genuinely needed alongside the
  value — never `range(len(seq))` plus a separate `seq[i]` lookup.
- `itertools.combinations(seq, 2)` over nested `range(len(...))` loops
  with an `i == j` skip when comparing every pair of elements.

Reach for direct indexing only when the algorithm is genuinely
random-access (e.g. binary search) — not as a default habit for "loop
over this and also that."

## Per-File Ignores

When a rule conflicts with a legitimate domain convention (e.g. N803/N806 for
ML matrix naming `X`, T20 for CLI scripts, PLW0603 for module-level singletons),
add a `[tool.ruff.lint.per-file-ignores]` entry in `pyproject.toml` with a
comment explaining the reason. Never suppress project-wide.
