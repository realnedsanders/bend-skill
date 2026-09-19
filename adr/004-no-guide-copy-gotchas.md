# ADR-004: Do not duplicate `bend guide`; ban invented syntax

## Status
Accepted

## Date
2026-09-19

## Context
Bend 2 is young. The guide, Base, and limitations list change with
the binary. A skill that pastes GUIDE.md will rot and will crowd out
hygiene rules.

Agents still invent `if`, typeclasses, tactics, inference, mutual
recursion, and `match f(x)`.

## Decision
- `SKILL.md` says: run `bend guide` (and `bend guide shaders` for
  graphics) in the environment that will compile the code.
- `references/gotchas.md` is a **negative grammar**: constructs that
  do not exist, with the Bend replacement.
- `references/base.md` is a cheat sheet of naming and how to query
  Base (`bend base`, `bend base --types`, `bend base Map`), not a
  dump of `base.bend`.

## Alternatives Considered

### Vendor GUIDE.md into the skill
- Pros: works without `bend` installed
- Cons: stale; the README already tells agents to run `bend guide`
- Rejected

### Only say "read the guide" with no gotchas
- Pros: tiny skill
- Cons: models still emit `if` after reading a guide once
- Rejected: gotchas earn their tokens

## Consequences
- If Bend adds `if` later, update `gotchas.md` and this ADR's
  successor.
- Skill verification includes: no copied GUIDE.md.
