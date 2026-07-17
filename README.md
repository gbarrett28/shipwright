# shipwright

Reusable Claude Code engineering-methodology skills — quality gates,
commit/issue/doc hygiene, tool preferences, and language guidelines — as a
versioned, installable plugin.

Most guidance repositories end up either too generic ("best practices") or
too tied to one project's specifics. Shipwright tries to thread that
needle: every rule here was extracted from a real, already-enforced
project convention, states the *invariant* rather than the concrete
command, and explains the failure mode it exists to prevent rather than
just asserting the rule. A representative example, from
`commit-and-test-integrity`: *"Confidence that your implementation is
right is not the same as having checked."* That's the style of reasoning
every skill in this repo aims for. See [`docs/design.md`](docs/design.md)
for the full rationale.

## Install

```
/plugin marketplace add https://github.com/gbarrett28/shipwright
/plugin install shipwright@shipwright
```

(Or, for local development, `/plugin marketplace add <path-to-local-clone>`.)

## What's inside

See `skills/` — one directory per skill, each a `SKILL.md`. Start with
`skills/using-shipwright/SKILL.md`, which explains when each of the other
skills applies.

## Design

See [`docs/design.md`](docs/design.md) for why this exists, what each
skill covers, the testing methodology, and what's deliberately out of
scope.

## Contributing

See `CLAUDE.md` for the skill-authoring process (this repo is written and
maintained via Claude Code using the `superpowers` plugin). Significant
additions are discussed in an issue before a PR.
