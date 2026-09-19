# ADR-003: Law-driven development is the default workflow

## Status
Superseded by ADR-006

## Date
2026-09-19

## Context
Bend's README presents `LAWS.bend` as the way humans specify intent
to AIs. Demos almost all ship `LAWS.bend` + `PROOF.bend` + `main.bend`.
Evals grade models on proving laws, not on green unit tests.

Agents trained on Python/Go will write functions and skip specs.
Agents trained on Lean may write tactics or `sorry`.

## Decision
The skill treats this layout and loop as default for any non-throwaway
program:

```
main.bend      implementation (AI)
LAWS.bend      claims (user-owned; AI must not weaken)
PROOF.bend     proofs (AI); imports LAWS.bend
```

Gate: `bend PROOF.bend` before commit. A `PROOF.bend` next to
`LAWS.bend` must import it (compiler rule).

Laws pin **pure** kernels (HTTP bytes, sort permutation, "cannot
win"). `@unsafe` event loops are allowed when the world does not
terminate; they stay out of the proof kernel.

The AI must not edit a law's statement to make a wrong program pass,
unless the user asks to change the spec.

## Alternatives Considered

### Laws only when the user asks for proofs
- Pros: faster hello world
- Cons: contradicts Bend's reason to exist; even `io_hello_world`
  has a law
- Rejected as default; a one-liner scratch file may skip laws if the
  user says so

### Put laws in the same file as code (eval style)
- Pros: matches `evals/*.bend` and many `tests/proof/*.bend`
- Cons: README and demos separate them so humans can own claims
- Rejected for projects; same-file `law`/`def` is fine for a single
  lemma file or an eval

## Consequences
- Skill checklist includes "laws still mean what the user asked".
- Proof reference documents `{==}`, `%`, `.fin` helpers, `exs`.
- `@unsafe` requires a comment naming why termination cannot be
  structural.
