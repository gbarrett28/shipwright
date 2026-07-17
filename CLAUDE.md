# Shipwright

## What This Project Is

`shipwright` is a Claude Code plugin: a versioned, collaboratively-maintained
library of reusable engineering-methodology skills (quality gates,
commit/issue/doc hygiene, tool preferences, language guidelines). Projects
install it via the Claude Code plugin marketplace mechanism and reference its
skills the same way they'd reference `superpowers`. This repo has almost no
application code — its product is the skill files themselves plus the
plugin/marketplace manifests that make them installable. It does not run its
own bronze/silver gate script; skill correctness is verified via the
RED-GREEN-REFACTOR testing described below, not automated CI.

## Required Superpowers

Working in this repo is itself Claude-Code-agent work, so the usual
superpowers discipline applies:

| Skill | Invoke when |
|---|---|
| `superpowers:brainstorming` | Before adding a new skill, removing one, or restructuring the plugin. |
| `superpowers:writing-skills` | For **any** creation or edit of a file under `skills/` — see "Skill Authoring" below, this is the core process for this repo. |
| `superpowers:verification-before-completion` | Before claiming a skill change is done, before creating a commit or PR. |
| `superpowers:requesting-code-review` | Before merging any PR. |
| `superpowers:systematic-debugging` | Before fixing any bug (e.g. a manifest that fails to load, a skill that doesn't trigger). |

## Skill Authoring

Every skill in `skills/` is written and maintained via
`superpowers:writing-skills` — its Iron Law applies here without exception:
**no skill without a failing test first.** Concretely, each skill is
classified into one of three testing tiers during brainstorming, before
writing begins, since the tier determines the required rigor:

- **Discipline-enforcing** (a rule the agent must follow under pressure —
  currently `using-shipwright`, `quality-gates`, `commit-and-test-integrity`):
  any change requires a RED-GREEN-REFACTOR cycle — a baseline pressure-
  scenario run via a fresh subagent *without* the change, the change written
  to address the observed failure, then a verification run *with* the
  change. Record the scenario and outcome in the PR description.
- **Pattern** (`issue-hygiene`, `tool-preferences`): lighter application-
  scenario testing — confirm a fresh subagent applies the guidance correctly
  to a realistic situation.
- **Reference** (`python-guidelines`, `typescript-guidelines`):
  retrieval/application testing — confirm a fresh subagent finds and
  correctly applies the relevant rule when asked.

## Repo Structure

| Path | Purpose |
|---|---|
| `.claude-plugin/plugin.json` | Plugin manifest (name, version, description, author) |
| `.claude-plugin/marketplace.json` | Marketplace manifest listing this repo's one plugin |
| `skills/<name>/SKILL.md` | One skill per directory; frontmatter `name` + `description` — see `superpowers:writing-skills` for conventions |
| `README.md` | Install instructions and contribution guide for humans landing on the repo page |

## Versioning

Bump `version` in **both** `.claude-plugin/plugin.json` and the matching
entry in `.claude-plugin/marketplace.json` whenever a skill's behavior
changes — patch for wording/clarity fixes, minor for a new skill, major for
a breaking rename or removal. Consuming projects pin to a marketplace, not a
version, so an un-bumped version on a real behavior change is silently
invisible to them.

## Commits & Contributions

This repo is public specifically to take outside contributions. Significant
additions (a new skill, a new discipline tier) should be discussed in an
issue before a PR, per `shipwright:issue-hygiene`. Commits follow
`shipwright:commit-and-test-integrity`'s Conventional Commits convention.

**AI attribution policy for this repo:** `shipwright:commit-and-test-integrity`
leaves the `Co-Authored-By` trailer as a project-by-project policy decision
rather than a blanket default. This repo's policy: include it. All commits
to date have, and that's the expectation going forward.
