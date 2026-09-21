# Skill design: `bend`

This is the design the implementation follows. ADRs record the
decisions; this file is the map.

## Goals

The user asked for a skill that:

- Enforces good programming practices
- Maximizes software-engineering style and hygiene
- Leverages what Bend actually provides
- Is informed by Bend docs/code and by great language skills (Go, Lean)

Non-goals:

- Replacing `bend guide`
- Teaching the TypeScript compiler
- Inventing a test framework Bend does not have
- A Python-callable wrapper around `bend`

## Audience

An agent about to write or review `.bend` files, including
`LAWS.bend` and `PROOF.bend`. The agent may know Python, Lean, Rust,
or Haskell and will try to write those.

## Shape

```
skills/bend/
  SKILL.md                 # load first: workflow, principles, checklist
  references/
    gotchas.md             # syntax that does not exist
    laws-and-proofs.md     # law-driven development
    ownership.md           # quantities, kinds, arrays, handles
    parallelism.md         # fork-join, GPU, shader rules
    io-and-effects.md      # IO, do, foreign C/JS
    style.md               # naming, modules, comments, layout
    tooling.md             # CLI, #| tests, fmt, gates
    base.md                # Base cheat sheet
```

`SKILL.md` must stay short enough to load on every Bend task.
References load when the task hits that subsystem.

## Description (routing)

Must fire on concrete Bend signals: writing/reviewing `.bend`, laws and
proofs, Base, Bend CLI/checker errors, IO/effects, and CPU/GPU parallelism in
Bend. Negative-language guidance belongs in the loaded body, not routing.

## Workflow encoded in SKILL.md

```
inspect project → clarify behavior/target → choose assurance gate → code →
validate final code → measure when performance is in scope
```

1. User requirements and existing repository conventions come first.
2. Check `bend version` when unknown. Use targeted Base lookups; consult
   `bend guide` on syntax/version uncertainty.
3. Preserve existing laws. For new critical pure behavior, propose laws and use
   the three-file layout when accepted.
4. User ownership of laws is semantic: agents may draft claims, but may not
   weaken accepted meaning merely to pass.
5. Implement the smallest change with structural recursion, explicit quantities,
   and helpers for computed matches.
6. Validate only the files/gates the task uses. Re-run proofs and behavior after
   parallelization; measure before claiming a speedup or GPU execution.

## Principles (the Go-skill equivalent)

1. Preserve the user's specification and the repository's accepted laws.
2. Affine default. `+` permits reuse but does not itself dictate runtime cost.
3. Structural termination: earlier recursive arguments pass unchanged until a
   constructor subterm decreases. Fuel or justified `@unsafe` at world loops.
4. Bend does almost no inference. Annotate each operator's own expression.
   Never invent `if`, tactics, typeclasses, or match-on-call syntax.
5. Query Base first; use `Type.verb` for Base-like/type-grouped APIs rather than
   as a universal naming mandate.
6. Proofs are programs. Use small lemmas and evidence-carrying decisions.
7. Parallel lets need independent, balanced work; `!` is a request, not proof of
   GPU use or speedup.
8. IO is explicit. Prove pure kernels; audit foreign/unsafe boundaries.
9. Validation is task-aware. Open holes and unexplained unsafe remain failures.
10. Keep diffs surgical; do not rewrite laws to fit wrong code.

## Success criteria for an agent using the skill

- Existing proof gates pass; every def named as relying on unsafe or foreign
  code is explicitly audited.
- The edited library checks, or the requested runnable/backend is built and run.
- No invented syntax.
- Recursive calls satisfy the actual left-to-right structural rule.
- Every `+` is required by reuse or a reusable callee contract on `Data`.
- Laws still mean what the user asked.
- Performance claims record representative inputs, hardware, flags, and repeated
  measurements.

## Installation

Repo path: `skills/bend/`.

To surface in Prime Agent, copy or symlink to
`.prime/agent/skills/bend/` (project) or `~/.prime/agent/skills/bend/`
(user). This repo does not assume a global install.
