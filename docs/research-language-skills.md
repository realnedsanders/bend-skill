# Research: what makes a great programming-language skill

Compared artifacts:

- `/home/user/.agents/skills/golang-patterns/SKILL.md` (ECC origin)
- `/home/user/.agents/skills/golang-testing/SKILL.md` (ECC origin)
- `/home/user/.agents/skills/karpathy-guidelines/SKILL.md`
- Prime Agent `skill-creator` (frontmatter, progressive disclosure)
- Official Bend `README.md` / `AGENTS.md` agent snippet
- Lean 4 / mathlib conventions (no installable Lean skill was present
  on this machine; Serper web search was not configured, so Lean is
  reconstructed from common agent-facing Lean 4 practice and from
  Bend's own contrast with Lean in `bend2/bend.lean` and the guide)

## Go skills: what works

Both Go skills are **behavior contracts**, not language tutorials.

Shared shape:

1. **Routing description.** Name the tasks: "Use when writing or
   reviewing Go code and idiomatic structure or conventions are in
   question." The description is the only text in the system prompt.
2. **When to activate.** A short list, not a novel.
3. **Core principles first.** Three to seven rules that decide 80% of
   diffs (simplicity, useful zero value, accept interfaces return
   structs).
4. **Good vs bad code.** Every rule has a pair. Agents copy the good
   shape.
5. **Anti-patterns table.** Explicit "never do this".
6. **Tooling as the loop.** `gofmt`, `go test -race`, `staticcheck`.
   The skill assumes the compiler/linter is the source of truth.
7. **Quick-reference table.** Idioms in one screen.
8. **Single-file, self-contained.** Neither Go skill uses
   `references/`. That works because Go's idioms are stable and the
   files stay ~14–17 KB.

What the Go skills deliberately skip:

- A full language grammar.
- A copy of Effective Go.
- Framework fashion (no "use this web router").

`golang-testing` is a second skill because Go has a real test runtime
(`testing`, tables, fuzz, benches). Bend does **not**. Splitting a
`bend-testing` skill would be fake: tests are `#|` golden output plus
`bend PROOF.bend`.

## Lean 4: what a great skill would include (and Bend must invert)

Lean 4 is the closest *cousin* and the worst *template* to copy
blindly. Bend's README and guide position Bend against Lean: faster
checker, no tactics, no inference, affine, parallel.

A typical Lean 4 agent skill (mathlib / Batteries / lake) would stress:

| Lean 4 hygiene | Why agents fail without it |
|----------------|----------------------------|
| Search mathlib before defining | Duplicate lemmas, wrong simp normal form |
| `rw` / `simp` / `induction` | Proofs are tactic scripts, not terms |
| Typeclasses (`Decidable`, `Fintype`) | Inference fills instances |
| Universes, positivity | `Type u`, inductive restrictions |
| `sorry` is a hole, CI bans it | Models leave sorry |
| `namespace` / `section` / `open` | Name management |
| `structure` vs `inductive` | Projections vs recursion |
| `lake build`, `#lint` | Project loop |

Bend inverts almost every row:

| Bend analogue | Skill must say |
|---------------|----------------|
| Base is small; write the helper | Do not wait for mathlib |
| Proof = `def` of the law's type | No tactics. `{==}`, `%e : P`, match, IH |
| No typeclasses, no inference | Annotate every bind and operator |
| One universe, no positivity; live/dead | Do not inhabit Empty in live code |
| `?TODO` / missing `def` is an open law | `bend PROOF.bend` must print "All terms check." |
| Dots are characters; `import as` | No `open` / namespaces |
| `type ... is Data` / `is Type` | Kind is a quantity cap, not a universe |
| `bend file.bend` / `bend PROOF.bend` | That is the whole loop |

Useful Lean habits that **do** transfer:

- State the theorem before the proof (Bend: `LAWS.bend` first).
- Keep lemmas small and named after the claim (`add_zero`, `add_succ`).
- Induction on the data the program already matches.
- Computationally relevant proofs: return evidence (`Or(LE, LE)`),
  then match it in the proof. Insertion-sort demo is the Bend version
  of "make the program produce the proof's case split".
- Do not prove floating point (Bend: F32 is axiomatic; Lean: `Float`
  is similarly not mathlib's `ℝ`).

A Bend skill that reads like a Lean tactic guide would make agents
invent `simp`, `have`, and `if`, then fail the checker.

## Karpathy guidelines (cross-cutting)

Relevant to Bend because models over-invent:

- Do not add features beyond the ask.
- Surface assumptions (Bend: quantities, termination, GPU).
- Success criteria that can be looped: here, `bend PROOF.bend`
  exit 0 and "All terms check."
- Surgical diffs: do not "improve" `LAWS.bend` claims.

## skill-creator constraints

- YAML frontmatter: `name` matches directory; `description` non-empty
  and routing-heavy (max 1024 chars).
- Progressive disclosure: keep `SKILL.md` as the decision flow;
  park long syntax, Base maps, shader cost model in `references/`.
- Markdown skill, not Python: the agent should run `bend`, not a
  wrapped Python API.
- Location: this repo is the product. Skill directory
  `skills/bend/` (name `bend`). Docs and ADRs stay outside the skill
  so they do not inflate agent context.

## What "great" means for Bend specifically

Bend is young (limitations list is long). A great skill therefore:

1. **Enforces the language's actual rules** so agents stop inventing
   Python/Lean/Rust.
2. **Makes laws the engineering process**, not a garnish. Go's
   equivalent is `error` handling; Lean's is theorems; Bend's is
   `LAWS.bend`.
3. **Treats the checker as the test suite.** There is no `go test`.
   Proofs and `#|` goldens replace it.
4. **Teaches resource and parallel cost**, because the runtime has no
   GC and no work-stealing. Wrong `+` or an unbalanced fork is a
   correctness-adjacent performance bug.
5. **Points at living oracles** (`bend guide`, `bend base`, demos)
   instead of freezing a stale copy of GUIDE.md.
6. **Has a review checklist** an agent can run before it claims done.

## Decision: one skill, not a family

Go splits patterns vs testing because the testing stack is large.
Bend's "testing" is laws + checker + golden `#|`. Parallelism and
proofs are not optional add-ons; they are how you write Bend at all.

One skill `bend` with `references/` for proofs, ownership,
parallelism, IO, style, tooling, Base, and gotchas.

See `docs/skill-design.md` and `adr/`.
