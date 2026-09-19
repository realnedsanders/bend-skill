# ADR-002: Skill lives in `skills/bend`; research lives in `docs/` and `adr/`

## Status
Accepted

## Date
2026-09-19

## Context
This repository was created for the skill. The user already added
`adr/` and `docs/` and asked to document findings along the way.
`skill-creator` default project path is `.prime/agent/skills/<name>/`.

## Decision
- Authoritative skill: `skills/bend/` (directory name = skill `name`).
- Research and ADRs: `docs/` and `adr/` at repo root, **outside** the
  skill, so agents loading the skill do not ingest the research.
- Do not also copy into `.prime/agent/skills/` unless the user asks
  to install it in a Bend project.

Filename convention for ADRs: `adr/NNN-kebab-title.md` with Status,
Date, Context, Decision, Alternatives Considered, Consequences.

## Alternatives Considered

### Skill is the repo root (`./SKILL.md`)
- Pros: matches many standalone skill repos
- Cons: ADRs and research would sit next to SKILL.md and might be
  treated as skill files
- Rejected: user already created `docs/` and `adr/` as siblings

### Install only under `.prime/agent/skills/bend`
- Pros: auto-discovered in this repo
- Cons: this repo is the skill product, not a Bend application
- Rejected for the canonical copy; README will say how to install

## Consequences
- Consumers copy or symlink `skills/bend` into a project's
  `.prime/agent/skills/bend` or `~/.prime/agent/skills/bend`.
- Future ADRs continue this numbering in `adr/`.
