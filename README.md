# shipwright

Reusable Claude Code engineering-methodology skills — quality gates,
commit/issue/doc hygiene, tool preferences, and language guidelines — as a
versioned, installable plugin.

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

## Contributing

See `CLAUDE.md` for the skill-authoring process (this repo is written and
maintained via Claude Code using the `superpowers` plugin). Significant
additions are discussed in an issue before a PR.
