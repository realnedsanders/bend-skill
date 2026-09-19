# ADR-007: Separate Bend constraints from optional style

## Status
Accepted

## Date
2026-09-19

## Context
ADR-005 generalized two useful patterns too far. `Type.verb` is Base's naming
scheme, but official demos also expose unqualified domain defs such as `sort`
and `replay`. The termination checker does not require the decreasing argument
to be literally first; earlier recursive arguments may appear when passed
unchanged. A critical review also found generic taste rules mixed with checker
requirements.

## Decision
- Preserve a repository's naming and public API.
- Use `Type.verb` for Base-like or type-grouped APIs; unqualified domain names
  are idiomatic too.
- For a new recursive API, put the decreasing argument as early as practical.
  Earlier arguments are valid when recursive calls pass them unchanged.
- Prefer a private worker over changing an established public signature solely
  for the termination checker.
- Label guidance as language/checker requirement, proof/assurance rule,
  measured performance advice, or optional style.
- Keep evidence-carrying comparisons and closed `~` templates as strong Bend
  patterns when the task needs them; do not impose them on unrelated code.

## Alternatives Considered

### Keep `Type.verb` and decreasing-first as universal review rules
- Pro: simple and uniform
- Con: contradicts official demos and encourages unrelated API churn
- Rejected

## Consequences
- ADR-005 is superseded.
- Review findings must distinguish invalid Bend from merely nonpreferred style.
