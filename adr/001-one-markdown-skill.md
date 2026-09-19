# ADR-001: One markdown skill with progressive disclosure

## Status
Accepted

## Date
2026-09-19

## Context
We need an agent skill that makes models write idiomatic Bend. Prime
Agent skills are directories with `SKILL.md`. Go in this environment
ships two skills (`golang-patterns`, `golang-testing`). Bend has
proofs, IO, and parallelism as well as "patterns".

Options:

1. Several skills (`bend-patterns`, `bend-proofs`, `bend-parallel`).
2. One large `SKILL.md` that contains the whole guide.
3. One skill `bend` whose `SKILL.md` is the workflow and principles,
   with `references/` for subsystems.

## Decision
Option 3. Name `bend`. Markdown only (no Python kernel package).

## Alternatives Considered

### Family of skills like Go
- Pros: smaller first load; testing split worked for Go
- Cons: Bend has no separate test runtime; proofs and affine rules
  apply to every file; an agent that loads only "patterns" will skip
  laws
- Rejected: splitting would hide the main engineering loop

### One giant SKILL.md copying `bend guide`
- Pros: self-contained offline
- Cons: stale vs `bend guide`; blows context; skill-creator says keep
  SKILL.md as the decision flow
- Rejected: agents should run `bend guide` and `bend base`

### Python-backed skill wrapping the CLI
- Pros: `await bend.check(...)`
- Cons: the agent already has a shell; wrapping hides flags (`--checkup`,
  `-o`, `--publish`)
- Rejected: markdown + "run `bend`" is the right interface

## Consequences
- Description must route all Bend tasks to this one skill.
- `SKILL.md` must tell the agent which reference to open next.
- Guide text lives in the Bend install, not in this repo.
