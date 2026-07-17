---
name: python-guidelines
description: Use before writing or reviewing Python code — covers strict ruff/mypy linting rules, the no-inline-noqa/type-ignore policy (use per-file-ignores or stubs instead), and the safe-then-unsafe auto-fix workflow.
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

## Per-File Ignores

When a rule conflicts with a legitimate domain convention (e.g. N803/N806 for
ML matrix naming `X`, T20 for CLI scripts, PLW0603 for module-level singletons),
add a `[tool.ruff.lint.per-file-ignores]` entry in `pyproject.toml` with a
comment explaining the reason. Never suppress project-wide.
