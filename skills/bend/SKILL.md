---
name: bend
description: Write, review, debug, and prove idiomatic Bend 2 programs. Covers .bend files, LAWS.bend and PROOF.bend, affine quantities, structural termination, Base, IO and effects, and CPU/GPU fork-join parallelism. Use for Bend CLI checks, compiler errors, correctness laws, proofs, ownership, or performance work in Bend.
compatibility: Requires the Bend 2 CLI. Detailed claims were checked with Bend 2.0.25; trust the installed checker and `bend base <Name>` when versions differ.
---

# Bend

Bend is Python-shaped, Lean-like, affine, and parallel. It is not Python,
Lean, Haskell, or Rust.

## Precedence and scope

1. Follow the user's specification.
2. Preserve the repository's existing layout, laws, public APIs, and style.
3. Apply this skill's defaults only where the project has not decided.

Do not introduce `LAWS.bend`/`PROOF.bend` or rewrite signatures during a
small review without explaining the cost and benefit. For new proof-oriented
code, use Bend's conventional three-file layout. The user has semantic
authority over laws: an agent may draft claims from an explicit specification,
but must not weaken their meaning merely to pass.

## Establish the source of truth

- If the toolchain is unknown, run `bend version`. Stop and report if Bend is
  missing or not Bend 2.
- Use targeted lookup: `bend base <Name>` or `bend base --types`. Bare
  `bend base` is large.
- Run or search `bend guide` when syntax is unfamiliar or the installed
  version changed. The actual checker and `bend base` win if prose conflicts;
  the 2.0.22 guide still contains stale operator and foreign-ABI details. For
  graphics, use `bend guide shaders`.
- When a source checkout is available, its demos and `guide/*.md` are useful,
  but the installed CLI governs the code being built.

## Load only the reference needed

| Task | Open |
|------|------|
| Syntax or checker errors | [gotchas.md](references/gotchas.md) |
| Laws, proofs, `{==}`, `%` | [laws-and-proofs.md](references/laws-and-proofs.md) |
| `+` / `-`, kinds, arrays | [ownership.md](references/ownership.md) |
| Parallel lets, `!`, GPU | [parallelism.md](references/parallelism.md) |
| `do IO`, handles, foreign effects | [io-and-effects.md](references/io-and-effects.md) |
| Naming, files, comments | [style.md](references/style.md) |
| CLI, test fixtures, formatter | [tooling.md](references/tooling.md) |
| Prelude types and verbs | [base.md](references/base.md) |

## Task flow

1. Inspect the target repository and its existing laws before editing.
2. Clarify observable behavior. For performance work, also clarify container,
   input shape, target lane (CPU/GPU/JS), and measurement.
3. Choose the assurance gate:
   - Existing laws: preserve and prove them.
   - New critical pure behavior: propose laws; use the conventional layout if
     accepted.
   - IO/foreign/unsafe behavior: law the pure kernel and add an executable
     fixture or integration check for the trusted boundary.
   - Small library or syntax task: check the edited file; do not manufacture a
     proof project.
4. Implement the smallest change using Base and existing conventions.
5. Validate the **final** code. If parallelism was added, rerun proofs and
   behavior checks after the parallel change.

Conventional new proof project:

```
main.bend      # implementation
LAWS.bend      # claims; imports main.bend
PROOF.bend     # fills; imports LAWS.bend
```

## Language requirements

- Ordinary variables are affine. Use `+x` only for reusable `Data` required by
  multiple uses or by a callee's reusable contract. `-x` is erased.
- Ordinary closures and partial applications are affine. Call a fixed top-level
  def directly. For a repeatedly applied higher-order argument, use a closed
  `~` template (which may name a top-level def).
- Recursion must structurally decrease. Arguments are checked left to right:
  keep earlier arguments unchanged until one is a constructor subterm. Put the
  decreasing argument as early as practical in a new API. Safe code has no
  mutual recursion; law-mediated unsafe cycles remain outside the guarantees.
- Use fuel for world-bounded loops. Use `@unsafe` only when termination cannot
  be structurally expressed or a documented unsafe runtime feature requires it;
  explain the boundary and treat it as outside proof guarantees.
- There is no `if`, tactic language, typeclass system, or match on a computed
  expression. Match `Bool` constructors; pass computed results to a helper that
  matches its parameter.
- Bend does almost no inference. Every operator needs an annotation around its
  own expression, even for `Nat`: `(a + b : Nat)` or `(a + b : U32)`. A return
  type or surrounding call, lambda, constructor, list, or match arm does not
  supply that annotation.
- `==` forms an equality type. Runtime equality is `T.is_eq(a, b)`.
- Query Base before adding a helper. Base/type-grouped APIs conventionally
  use `Type.verb`; preserve domain naming already used by the project.

## Correctness requirements

- Do not change a law to fit incorrect code unless the user changes the spec.
- A proof is a `def`, not a tactic script. No `sorry`; `?TODO` remains open.
- Prefer laws about externally meaningful properties: preservation, bounds,
  ordering plus permutation, state invariants, or exact bytes.
- Unsupported F32 algebra, host effects, schedulers, and intentional unsafe
  loops are not ordinary proof targets. Reflexivity/congruence over F32 wrappers
  can still be proven. Otherwise isolate and prove a pure representation/kernel.
- Foreign C/JS effects are trusted code. Audit both backends and their resource
  behavior; Bend's proof checker does not verify them.

## Performance guidance

- A parallel let promises independent calls of roughly equal cost. Safe
  purity/affinity gives independence; experimental unsafe aliases require their
  own race argument. Balance is your responsibility.
- `f!(x)` requests the GPU but may fall back to the CPU. It does not prove that
  the algorithm is GPU-suitable or that a GPU executed it.
- Prefer uniform numeric work on GPU. Keep divergent search, data-dependent
  merge, and small jobs on CPU unless measurements support another choice.
- Do not add `+`, forks, or `!` speculatively. Preserve correctness, then measure
  the final implementation on representative inputs.

## Task-aware validation

- With `LAWS.bend`/`PROOF.bend`: run `bend PROOF.bend`. Accept a verdict that
  names defs relying on unsafe or foreign code only when every dependency is
  intentional and documented.
- Check without running: use `bend file.bend --check-only`.
- Runnable target: run `bend file.bend`; for a requested native backend, build
  with `bend file.bend -o out` and then execute `./out` with the intended flags.
- Library/no `main`: run `bend file.bend`; it checks only.
- Import-sensitive change: add `bend file.bend --checkup`; it checks and runs
  each direct aliased import as a root, not the entry file.
- Performance change: compare the relevant CPU/GPU/JS modes and record input,
  hardware, flags, and repeated measurements.
- Always inspect the exit code and diagnostics; do not require files that the
  repository does not use.
