# ADR-005: Style is Base-first, Type.verb, evidence-carrying code

## Status
Superseded by ADR-007

## Date
2026-09-19

## Context
Base is small and consistent: `Type.verb`, `T.to_x` / `T.from_x`,
operators desugar to those verbs. Demos and evals share a proof style
(small lemmas, `.fin` for computed matches, `LE` as a type, `%ih`
rewrites). Go skills succeed by showing that style, not by listing
every stdlib function.

## Decision
The skill mandates:

- Name defs `Type.verb` or `Module.Type.verb`. Dots are characters.
- Prefer Base. Do not invent a second `List.map` if a template exists.
- Use `~` templates for a function that must run many times (map,
  fold with a closed `f`). Do not use affine closures as multi-shot
  callbacks.
- Put the decreasing argument first.
- When a program branches on a comparison, return evidence
  (`Or(LE(a,b), LE(b,a))`) so the proof can match the same branch.
- Comments explain why (quantities, termination, rewrite motive),
  never restate the next line.
- Format: the official formatter is `tools/bend-fmt-lsp` (indent and
  spacing only). Match demo indentation (2 spaces) if no formatter
  is wired.

## Alternatives Considered

### Python-style names (`sort_list`, `snake_case` modules)
- Pros: familiar
- Cons: fights Base and `import as M` (`M.square`, `U32.show`)
- Rejected

### Lean-style `namespace` / unicode / tactic blocks
- Pros: familiar to prover people
- Cons: not in the language
- Rejected

## Consequences
- Review checklist flags non-`Type.verb` public defs and closures
  used as maps.
- Style reference stays short; Base reference tells how to look up
  verbs.
