# Critical review — 2026-09-19

## Scope and evidence

Reviewed the skill as both Bend guidance and an Agent Skills artifact.

- Installed compiler: `bend 2.0.10`
- Source comparison: `bendlang/bend` commit
  `15ae0c86f3193b8f645b4bedbc438655b648d0da`
- Primary sources: installed `bend guide`, targeted `bend base <Name>`,
  `guide/GUIDE.md`, `guide/SHADERS.md`, `guide/EFFECTS.md`, `bend2/base.bend`,
  demos, tests, and `WONTFIX.txt`
- Skill validation: YAML/frontmatter, local Markdown links, Agent Skills
  validator, and selected snippets/commands with the installed checker
- Independent passes: Bend factual accuracy, skill design/progressive
  disclosure, and a usability scenario (verified U32 sort + GPU request)

The installed CLI and the source checkout are not the same release. The skill
now records that distinction and tells agents to trust the installed CLI for
the project being compiled.

`bend guide` exists and remains the language source of truth. Named guide pages
are versioned separately: on the reviewed 2.0.10 install, `bend guide shaders`
works but `bend guide effects` does not; the later source checkout documents
that newer command.

## Critical/high findings resolved

1. **Invalid “Good” computed-match example.** It referenced undefined `Ord`,
   `Lt`, `Eq`, `Gt`, and `cmp`. Removed from the main skill; detailed checked
   syntax lives in the gotchas reference.
2. **Unconditional proof-project workflow.** The skill could create
   `LAWS.bend`/`PROOF.bend` during narrow work and conflict with existing
   repositories. Added explicit precedence and task-aware assurance gates.
3. **Exact clean verdict conflicted with intentional unsafe loops.** Clean
   safe-code output and unsafe-count output are now distinguished; every unsafe
   occurrence/count must be audited.
4. **Progressive disclosure was nominal.** `SKILL.md` repeated most references,
   the CLI, examples, anti-patterns, and checklist. It was reduced from 255 to
   about 125 lines and now routes to targeted references.
5. **Full guide/Base dumps on every activation.** Replaced with targeted
   `bend base <Name>` and guide use on version/syntax uncertainty.
6. **Unavailable source paths.** Added official-repo discovery and release
   matching. Source-tree paths are no longer assumed to exist beside a user
   project.
7. **GPU sort ambiguity.** Distinguished CPU fork-join merge sort from uniform
   GPU algorithms. The skill now requires input/container/padding clarification,
   final equivalence laws, proof reruns, and honest reporting of CPU fallback.
8. **Foreign effects under-specified.** Marked C/JS as trusted code, added
   registration/need guidance, backend parity and resource checks, and the
   version-specific named-guide fallback.

## Factual corrections resolved

- `Chan(A)` is `Data`, not an affine opaque `Type` handle.
- `Either` has minimum kind; `Or` is an affine proposition specialization.
- `List.for_each` and `List.show` are templates.
- `Array.clone` requires `Data` elements.
- `+` permits reuse but does not itself decide borrow/refcount behavior.
- Discarding is permitted but destruction is not universally runtime-free.
- Live-handle operations usually return the handle; creation and close differ.
- Bun, not generic Node, is the safe JS default for Base foreign effects.
- Both `?name` and `?TODO` are open terms that fail checking.
- Primitive F32 algebra is axiomatic, but reflexivity/congruence remain valid.
- `Type.verb` is a Base/type-grouped convention, not a universal public-name
  requirement.
- Structural recursion permits earlier unchanged arguments; the decreasing
  argument need not literally be first.
- A native `-o` command builds; validation executes the resulting binary too.
- A bang requests GPU execution but can fall back to CPU; JS ignores it.
- Runtime review notes now include JS NaN payload behavior and Nat/array limits.

## Design corrections

- User specification and existing project conventions outrank skill defaults.
- User ownership of laws means semantic authority; agents may draft claims from
  an explicit spec.
- Validation is file/task-aware rather than requiring `main.bend` and
  `PROOF.bend` everywhere.
- Style, checker constraints, proof guidance, and measured performance advice
  are labeled separately.
- Installation commands create parent directories, offer copy/symlink choices,
  and include `/reload` + `/skill:bend` verification.

## Open decision

The repository has no `LICENSE`. The skill invites copying/installing, so reuse
terms should be chosen before publication. This review did not invent a license
on the owner's behalf.
