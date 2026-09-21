# ADR-006: Project conventions and task scope precede law-workflow defaults

## Status
Accepted

## Date
2026-09-19

## Context
The first skill version made the three-file `main.bend` / `LAWS.bend` /
`PROOF.bend` workflow unconditional for nearly every non-throwaway task. A
critical review found that this could impose new architecture during a small
review, conflict with an existing repository, and require files that do not
exist. It also repeated much of every reference in `SKILL.md`, weakening
progressive disclosure.

Bend's laws remain a central engineering feature. The correction is about
scope and authority, not removing law-driven development.

## Decision
Instruction precedence is:

1. User specification and explicit constraints.
2. Existing repository layout, laws, public APIs, and style.
3. Skill defaults where the project has not decided.

For a new proof-oriented project, the three-file layout remains the default.
For an existing project or a narrow task, inspect first and propose new laws or
layout only when their benefit justifies the change.

The user owns the *meaning* of laws. An agent may draft them from an explicit
specification, but may not weaken accepted claims merely to pass.

Validation is task-aware:

- Run `PROOF.bend` only when the project has or requests it.
- Check libraries without manufacturing `main`.
- Run/build runnable targets.
- Re-run proofs and behavior checks after parallelization.
- Audit any successful verdict that reports reliance on unsafe or foreign code.

`SKILL.md` contains precedence, routing, a short workflow, core requirements,
and the validation contract. Detailed examples and subsystem guidance live in
`references/`.

## Alternatives Considered

### Keep the original unconditional law workflow
- Pro: strongly reinforces Bend's differentiator
- Con: creates unrelated files and churn; fails on libraries and existing
  conventions
- Rejected

### Make laws opt-in only
- Pro: minimal intervention
- Con: agents would again treat Bend as merely a functional language and skip
  its primary correctness mechanism
- Rejected: propose laws for new critical pure behavior, but obtain alignment
  with the task and repository

## Consequences
- ADR-003 is superseded.
- Reviews distinguish checker requirements, assurance gates, performance
  guidance, and optional style.
- The main skill is shorter and no longer requires a full guide/Base dump on
  every activation.
