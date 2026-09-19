# Gotchas — syntax Bend does not have

Read `bend guide` for the positive grammar. This file is the negative
one. If the checker is angry, look here before changing the spec.

## Control flow

There is no `if`, `else`, `elif`, `cond ? a : b`, `and`/`or` as
short-circuit statements, `for`, `while`, or `return` outside `do`.

```python
# Good
match U32.is_lt(a, b):
  case True{}:
    a
  case False{}:
    b

# Bad
if a < b:
  return a
```

`&&` / `||` exist on `Bool` as operators (with spaces). They are not
Python `and` / `or`.

## Match

- Scrutinee: a parameter or a variable bound by a pattern. Not a call.
- No `let` of a computed value, then `match` on a parameter, in a way
  that breaks binder order. When in doubt, split a helper.
- Several scrutinees: `match a b:` and `case K{x} 1n+p:`.
- `_` in a pattern is discard.
- Empty `match e:` with no cases is allowed when `e : Empty`.
- There is no `with` / `match ... as`.

```python
# Good — computed result becomes a parameter
def insert.fin(x: Nat, h: Nat, t: List<&2, Nat>, r: List<&2, Nat>,
  c: Or(LE(x, h), LE(h, x))) -> List<&2, Nat>:
  match c:
    case Inl{e}:
      x <> h <> t
    case Inr{e}:
      h <> r

def insert(+x: Nat, xs: List<&2, Nat>) -> List<&2, Nat>:
  match xs:
    case Nil{}:
      [x]
    case h <> t:
      insert.fin(x, h, t, insert(x, t), le_case(x, h))
```

Name these helpers `foo.fin` or `foo.step` (dots are characters).

## Recursion

- Must terminate by structural descent. Left-to-right: prefix of
  arguments unchanged, then one constructor field of a parameter.
- Not a descent: `n/2`, `n-1` on `U32`, `Nat.divmod`, an index into
  an array, a hash.
- No mutual recursion. One def plus a tag:

```python
type Which is Data:
  GoA{}
  GoB{}

def both(w: Which, n: Nat) -> U32:
  match w n:
    case GoA{} 0n:
      0
    case GoA{} 1n+p:
      both(GoB{}, p)
    case GoB{} m:
      both(GoA{}, m)
```

- `@unsafe def` skips the check. Comment why. Checker still exits 0.

Fuel (from `tests/run/fuel_loops.bend`): first argument `fuel: Nat`,
match `0n` / `1n+f`, recurse on `f`.

## Types and terms agents invent

| Invented | Bend |
|----------|------|
| `int`, `i32`, `u64`, `f64` | `Nat`, `U32`, `F32` only |
| `Option` | `Maybe` (`None{}` / `Some{x}`) |
| `Result<T,E>` order | `Result<a,b,E,A>` with `Fail` / `Done` |
| `enum` / `struct` | `type T is Data:` constructors |
| `impl` / typeclasses / traits | Extra arguments, or templates `~` |
| `fn` / `lambda x: ` | `x => e`, `+x => e` |
| `list.append` method | `List.append(a, A, xs, ys)` |
| `xs.map(f)` method | `List.map(~A, ~B, ~f, xs)` |
| `x == y` as Bool | `T.is_eq(a, b)` |
| `{a == b : T}` as Bool | equality **type** (a proof) |
| `sorry` / `admit` | missing def, or `?TODO` (open) |
| `have`, `show`, `by` | extra `def` lemmas |
| `namespace` / `open` | `import ./f.bend as M`, then `M.x` |
| unicode `→`, `∀`, `ℕ` | `->`, `for`, `Nat` |
| `mut` / `&` borrows | affine binders; `Array` one owner |

## Operators and literals

- Spaces around operators. `(a + b : T)` → `T.add`.
- Without `: T`, `+ - * / %` are `Nat`.
- Bits: `.&.` `.|.` `.^.` `<<` `>>` (shift amount is `Nat`).
- `++` is `String` concat anywhere.
- `3` is `U32`. `3n` is `Nat`. `'c'` is `Char`. `"s"` is `String`.
- `3n` past `256n` is `U32.to_nat(n)` underneath, up to `4294967295n`.
- Lists: `[a, b]`, `h <> t`. Tuples: `(a, b)`.
- Constructors: `K{field, field}`. Unit: `Unit{}`. Bool: `True{}` /
  `False{}`.
- Arrays: `[v : T*n]` length power of two, or `[v : T^d]` depth.
  `a[i]` and `a[i] <- v` (U32 elements). Indexes wrap.

## Quantities on binders

```
-x   erased
 x   affine (default)
+x   reusable, requires Data
~f   template (syntax, compile-time, closed argument)
```

`type T is Data` allows `+`. `is Type` does not. `Kind(q)` is the
general form: `Type = Kind(&1)`, `Data = Kind(&2)`.

## Functions and templates

```python
def adder(k: U32) -> U32 -> U32:
  x => (x + k : U32)
```

That closure is one-shot. Partial application is a closure too.

```python
def twice(~f: U32 -> U32, x: U32) -> U32:
  f(f(x))

# call: twice(~(x => (x + 1 : U32)), 40)
```

Template arguments first. `~` args must be closed (no caller locals).
A template may call only templates declared **above** it.

## Equality and proofs (syntax only)

```
{a == b : T}     equality type
{a != b : T}     not equal (equals → Empty)
{==}             refl, when both sides compute equal
%e : P           rewrite; `_` in P is the right side of e
%e@E : P         named rewrite
?name            print goal
?TODO            leave open
exs z: T         witness; proof returns (z, p)
for x: A         law parameter
for y: B where P(y)   y comes with a proof of P(y)
```

## Modules

```python
import Base
import ./math.bend as M
import 0x<hash>/main.bend as P
```

The alias is local. There is no `from` / `open` / re-export.

## Comments and layout

`#` comments. Two-space indent in official files. Operators in
`( ... : T )` need spaces. A `#|` line is test expected output in the
Bend repo's harness — do not put `#|` in app code unless you mean that.

## When the checker and the runtime disagree

WONTFIX: a pure `main` is normalized lazily; compiled lanes are
strict. Unused lambda arguments may be skipped by the checker. Do not
rely on side-effect-like work in an unused position (there are no
live side effects there anyway).
