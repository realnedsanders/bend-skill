# Style and hygiene

Preserve the repository's established style first. For new code without a
local convention, match the installed Base and official demos. The language
requirements below are stronger than the optional style guidance.

## Naming

- For Base-like/type-grouped APIs, prefer `Type.verb` (`Nat.add`,
  `List.reverse.go`, `Game.Pong.press`). Unqualified domain defs such as
  `sort` or `replay` are also idiomatic in official demos.
- Dots are part of the name, not modules.
- File alias: `import ./game.bend as Game` then `Game.tick`.
- Helpers: `foo.fin`, `foo.step`, `foo.go` for the recursive worker
  (accumulator last, decreasing arg first).
- Laws: `snake_case` claim names (`sort_perm`, `you_cant_win`).
- Types/constructors: `PascalCase` (`Sorted`, `Inl`, `MNode`).
- Do not suffix `Type`, `Utils`, `Helper` on modules.

## Files

```
main.bend      implementation
LAWS.bend      user-authorized claims
PROOF.bend     fills; imports LAWS.bend
foo.bend       extra module; import as alias
```

This is a new-project convention, not a migration instruction. Keep an existing
layout unless the user requests a change. Split a large file only when it
creates a clear module boundary. Imports introduce names; they do not run a
module initializer.

## Comments

Why, not what:

```python
# inserting x into a list bounded by lo keeps the bound when lo <= x
def sorted_ins(...):
```

Do not narrate `match xs:`. Do not leave commented-out code. Do not
TODO what you should write now.

Document:

- Why a binder is `+` or `-`
- Why `@unsafe` (and what still holds)
- Rewrite motives that are not obvious
- GPU/host split and bang placement

## Implementation guidance

**Language/checker constraints:**

- Split `.fin` when a computed result must be matched.
- For a new recursive API, put the structurally decreasing argument before
  changing arguments. Preserve an existing public signature when a private
  worker can satisfy termination instead.

**Proof and performance guidance (apply when relevant):**

- Carry evidence in the data when the program already decides a comparison
  (`le_case`).
- Prefer tail recursion with an accumulator in hot folds (`List.reverse.go`);
  non-tail recursion is valid when the algorithm or proof needs it.
- Use Base. Reimplement `List.append` only for a different quantity or a
  proof-specific view, and document why.
- Avoid speculative helpers unrelated to the requested change.

## Equality and Bool

- Properties: `{a == b : T}` in laws.
- Runtime checks: `T.is_eq`, `T.is_lt`, ... returning `Bool`.
- Do not compare with `==` as if it were Python.

## Formatting

Official LSP: `tools/bend-fmt-lsp` in the Bend repo — **format only**,
no diagnostics. 2-space indent, spaces around operators, preserve
comments and line breaks.

If the formatter is not installed, match the neighboring file.

## Review order

1. Laws still mean the user's rules.
2. Checker (`bend PROOF.bend`).
3. Invented syntax / termination / extra `+`.
4. Base naming and unused defs **you** introduced.
5. Parallelism only if performance is in scope.

Do not reformat or rename unrelated defs (Karpathy: surgical diffs).
