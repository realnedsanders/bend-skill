# Base cheat sheet

Do not vendor `base.bend`. Prefer targeted queries; bare `bend base` is
large enough to crowd out the task:

```bash
bend base --types      # compact type inventory
bend base List         # one name and its subnames
bend base Map
bend base IO
bend base Equal
bend base              # everything; use only when a full dump is intentional
```

If this file and the installed `bend base <Name>` disagree, trust the install.

## Types

| Type | Kind | Constructors / notes |
|------|------|----------------------|
| `Empty` | Data | no constructors |
| `Unit` | Data | `Unit{}` |
| `Bool` | Data | `False{}` `True{}` |
| `Cmp` | Data | `LT{}` `EQ{}` `GT{}` |
| `Either<a,b,A,B>` | `Kind(a <&> b)` | `Inl{value}` `Inr{value}` |
| `Or(A,B)` | Type | affine proposition specialization of `Either` |
| pairs `A & B` | | `(a, b)` / `Tuple` |
| `Nat` | Data | `Zero{}` `Succ{pred}` ; literals `3n` |
| `Maybe<a,A>` | kind of A | `None{}` `Some{value}` |
| `Result<a,b,E,A>` | min | `Fail{error}` `Done{value}` |
| `List<a,A>` | kind of A | `Nil{}` `Con{head,tail}` ; `[a,b]` `h <> t` |
| `U32` | Data | word of 32 bits; literals `3` |
| `F32` | Data | primitive algebra is axiomatic; refl/congruence still work |
| `Char` | Data | `Chr{code: U32}` ; `'c'` |
| `String` | Data | `SNil{}` `SCon{head,tail}` ; `"..."` **cons list** |
| `Array<T>` | Type | `ALeaf` `ANode` ; prefer literals |
| `Image` | Data | `Pix{color}` `Qua{tl,tr,bl,br}` |
| `Event` | Data | `Key` `Mouse` `Move` `Close` |
| `Map<a,V>` | kind of V | string keys; `get`/`has` return the map too |
| `Set()` | Data | string set backed by `Map<&2, Unit>` |
| `IO(A)` | Type | law; `do IO<A>:` |
| opaque handles | Type | `File` `Socket` `Listener` `Window` `Audio` |
| `Chan(A)` | Data | copyable channel handle used by linked computations |
| `App<S>` | Type | `App{view, tick}` |

`List<U32>` is affine in the list spine unless you write
`List<&2, U32>` / `+List<U32>`.

## Verb scheme

`Type.verb`. Same verbs on `Nat`, `U32`, `F32`:

- arith: `add sub mul div mod`; `Nat.min`/`Nat.max` are structural, rebuild
  their result, and have `Nat.ge_refl`, `Nat.max_ge_l`, and `Nat.max_ge_r` laws
- bits (`U32`): `and or xor not shl shr`
- cmp: `cmp` → `Cmp` (not on `F32`); `is_eq is_ne is_lt is_le is_gt is_ge` → `Bool`
- `show` → `String`; `read` → `Maybe`
- conversions: `T.to_x`, `T.from_x` (`U32.to_nat`)
- `U32.log2` returns the floor base-2 logarithm of a positive value as `Nat`

Operators with `( ... : T )` desugar to these. `<`-style comparisons
too. Value equality is `T.is_eq`, never `==`.

## List (common)

Templates (`~`) for the function argument:

- `List.map(~A, ~B, ~f, xs)`
- `List.filter(~A, ~f, xs)` — `A` is `Data`
- `List.foldl` / `List.foldr`
- `List.any` / `List.all` / `List.find` / `List.contains`
- `List.sort(~A, ~le, xs)` — merge sort with fuel
- `List.for_each(~a, ~A, ~f, xs)`
- `List.show(~a, ~A, ~f, xs)`

Non-templates: `length append concat reverse is_empty head tail last
get set take drop zip range replicate`.

`List.map` cannot take an affine closure you plan to call on every element.
Pass a closed template, such as `~Top.show` or `~(x => ...)`.

## Array

`Array.map(~T, ~U, ~f, a)` walks the block and writes a fresh array. Both `T`
and `U` must be `Data`; its speed comes with this stricter contract. To map
an array of affine elements, consume its `ALeaf`/`ANode` tree explicitly with
a project-local helper.

Other common operations: `new size get set swap clone to_list`. Query
`bend base Array` before choosing one because element-kind requirements differ.
See [ownership.md](ownership.md) before using `fork`, `join`, or atomics.

## Map

Common operations: `new set get has del pop keys size to_list from_list`.
`get` takes a default. `get` and `has` return the map beside the answer
(affine spine). Values must be `Data` for `get` because it returns a value
while preserving the map.

`Set` provides `new add has del size to_list from_list` for string keys.

## Equal

`Equal.sym`, `Equal.trans`, `Equal.cong` — use in proofs instead of
rolling your own once, unless you are proving those.

## IO (names only)

Print/env/time/sleep/random, spawn/channels, files, TCP/UDP, window/audio.
Bend 2.0.22 adds `IO.random_u32`, `File.read_at`, `File.size`,
`File.write_bytes`, and deadline-based `TCP.poll`. No TLS, HTTP, JSON, or regex
in Base. Demos implement tiny HTTP on TCP. Add foreigns if you must; see
[io-and-effects.md](io-and-effects.md).

## Shared arrays (unsafe)

`Array.fork`/`Array.join` alias one block. `Array.atomic.*` provides U32
atomics and F32 `fadd`. These operations are experimental trusted boundaries,
not a relaxation of ordinary affine ownership; see
[ownership.md](ownership.md).

## When Base is missing something

Write a small local def in the project's naming style. Use `Type.verb` when
building a Base-like or type-grouped API. Prove it if a law needs it. Do not
import an imaginary stdlib or copy a speculative theorem library.
