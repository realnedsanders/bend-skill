# Laws and proofs

Bend has no tactics. A proposition is a type. A proof is a `def`.

## Files

Project default (official demos, README):

| File | Owner | Content |
|------|--------|---------|
| `LAWS.bend` | User-authorized specification | `law` claims; import the code |
| `PROOF.bend` | Implementation work | `def Laws.name`; import `LAWS.bend` |
| `main.bend` | Implementation work | Program code |

"User-authorized" is semantic ownership, not a ban on agent drafting. An
agent may propose initial laws from an explicit specification. Once accepted,
it must not change their meaning merely to make a proof pass.

The compiler **refuses** a `PROOF.bend` beside a `LAWS.bend` that it
does not import.

Same-file `law` + `def` is fine for a lemma module or an eval.

A law with no def is an open claim. `bend PROOF.bend` fails until
every law holds.

Do not rewrite a law to match a buggy program. Change code, or ask
the user to change the spec.

## What to put in a law

Good laws are properties an editor could break:

- Sort: output is sorted **and** a permutation (counts).
- Game: no move list wins; player never stands on `'F'`.
- HTTP: body after `\r\n\r\n` is the page; response injective in the
  page.
- Parallel sum: tree equals a sequential spec.
- Pong: Esc quits; last key press wins.

Bad laws:

- Restate the code (`{f(x) == <copy of body>}`) unless the point is
  definitional pinning (hello world).
- Assert unsupported algebraic identities for primitive `F32` operations.
  Reflexivity and congruence for defined wrappers remain provable.
- Quantify over IO schedules or `@unsafe` loops. Pin the pure kernel
  instead.

## Law syntax

```python
law add_zero:
  for x: Nat
  {Nat.add(x, 0n) == x : Nat}

law sort_perm:
  for +x  : Nat
  for +xs : List<&2, Nat>
  {Sort.count(x, Sort.sort(xs)) == Sort.count(x, xs) : Nat}

law page_after_blank:
  for +page: String
  exs head: String
  {head ++ ("\r\n\r\n" ++ page) == Srv.http_response(page) : String}
```

- `for +x` / `for -x` set quantity on the parameter.
- `for y: B where P(y)`: the binder is the pair `(y, proof of P(y))`.
- `exs z: T`: the proof returns `(z, p)`.
- `for y: B where P(y)` packages a value with evidence; inside the claim,
  `y` is that pair, not transparently the original value. Destructure it when
  you need both components. Use separate `exs` clauses when the later claim
  must refer directly to a witness, as in `(witness, evidence, proof)`.
- The claim is any type. `{a == b : T}` is the common one. It can
  also be a `Type` you defined (`Sorted(xs)`, `Bounded(m, xs)`).

Filling:

```python
def add_zero(x):
  ...

# or, from PROOF.bend
def Laws.add_zero(x):
  ...
```

The fill may omit argument types; the law has them.

## Proof moves

### Computation / refl

If both sides of `==` reduce to the same term, `{==}`.

Hello world's `main` unfolds to `IO.print(...)`. Esc-quits is
definitional. Use this first.

### Match / induction

Match the same inductive value the code matches. The recursive call
is the induction hypothesis.

```python
def add_zero(x):
  match x:
    case 0n:
      {==}
    case 1n+p:
      %add_zero(p) : {1n+Nat.add(p, 0n) == 1n+_ : Nat}
      {==}
```

Motive: after `%e : P`, `_` stands for the **right** side of `e`'s
equality. `P` is the goal with that hole. Then the goal becomes `P`
with the left side there. `%x@e : P` is the explicit-motive form: the motive
can also refer to the equality proof as `x`.

Stack rewrites; finish with `{==}`.

### Helpers for computed matches

If the program called `le_case(x, h)` and branched, the proof cannot
`match le_case(x, h)`. Pass the verdict into `.fin` and match that.

See the release-matched
[`demos/proof_insertion_sort`](https://github.com/bendlang/bend/tree/v2.0.25/demos/proof_insertion_sort)
example, or the corresponding directory in a source checkout matching the
installed compiler.

### Evidence in the program

A runtime `Bool` such as `U32.is_le(a, b)` is a decision result, not by itself a
proof of a proposition. For a verified U32 ordering algorithm, define a
proof-level relation over U32/Word and a comparison that returns branch
evidence, or use a proved conversion to a proof-friendly representation. Do
not assume Base supplies a theorem connecting each U32 predicate to a custom
`LE` type.

Prefer returning proofs from the implementation:

```python
def LE(a: Nat, b: Nat) -> Data:
  match a b:
    case 0n b0:
      Unit
    case 1n+a1 0n:
      Empty
    case 1n+a1 1n+b1:
      LE(a1, b1)

def le_case(x: Nat, h: Nat) -> Or(LE(x, h), LE(h, x)):
  ...
```

Then insert/sort and the proof share one case tree. This is the Bend
form of "make illegal states unrepresentable" plus "don't resplit
decidable equality in the proof".

### Template theorems

A law may take closed `~` parameters. Its proof is checked once against those
opaque parameters, then instantiated like any other template; hypotheses may
be reused without spending an affine closure. Prefer this over duplicating a
proof for each closed operation. See upstream `tests/proof/template_law.bend`.

### Empty and inequality

- `match e:` with no cases, when `e : Empty`.
- `{a != b : T}` is `{a == b : T} -> Empty`.
- To refute `e : {1n == 0n : Nat}`, rewrite through a discriminator
  `disc` (`0n → Empty`, `1n+p → Unit`) and return `Unit{}`.
- `Equal.sym`, `Equal.trans`, `Equal.cong` are in Base.

### Witnesses

```python
def Laws.page_after_blank(page):
  ("HTTP/1.1 200 OK\r\n...", {==})
```

### Dependent types as defs

```python
def Sorted.from(lo: Nat, xs: List<&2, Nat>) -> Type:
  match xs:
    case Nil{}:
      Unit
    case h <> t:
      (LE(lo, h) & Sorted.from(h, t))
```

A proof of `Sorted.from(lo, xs)` is a nested pair of `LE` proofs.
This is normal and preferred over encoding everything as `== Bool`.

## Style of lemmas

- Name after the claim: `add_zero`, `add_succ`, `add_comm`,
  `count_ins`, `sum_pour`.
- Keep them small. One rewrite idea each.
- Prove `Nat` facts you need; Base is not mathlib. Do not paste a
  giant arithmetic library "for later".
- Quantities: if the IH needs `b` twice, the law takes `+b` and `b`'s
  type must be `Data`.

## Holes

- `?name` prints a named goal but remains an open term — it fails checking.
- `?TODO` leaves an open term with the dedicated TODO report — it also fails.
- A law with no def fails the gate.

There is no `sorry` that checks.

## What not to prove

- Algebraic identities that require facts about primitive `F32` operations.
- Host effects (clock, TCP). Prove functions that build the bytes or
  the next pure state.
- Termination of `@unsafe` servers. Use fuel in the proven kernel.

## Live and dead terms

Bend separates live runtime code from dead terms used for types, erased
arguments, and equations. Dead terms may loop or inhabit `Empty`, but they
cannot cross into live evidence. Live recursive code must terminate. This wall
is part of the theory; do not treat a dead inhabitant as an escape hatch for a
runtime proof.

## Gate

```bash
bend PROOF.bend
# All terms check.
```

`All terms check.` is the clean safe-code verdict. Bend 2.0.22 instead names
transitive dependencies on trusted code:

```text
All terms check, but N defs rely on unsafe or foreign code:
- name
```

The exit code is still 0 (WONTFIX), but the named defs are not fully covered by
Bend's checker. Importing an unused unsafe def does not taint a file; calling it
or naming it in a type does. Audit every listed def and treat an unexplained or
changed list as a failed review.

The undocumented `bend file.bend --checkup` diagnostic checks and runs each
direct aliased import as its own root, not the entry file. Use it when a cycle
or a missing fill is unclear; do not treat it as a stable public interface.
