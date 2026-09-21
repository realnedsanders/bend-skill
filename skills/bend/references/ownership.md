# Ownership, quantities, kinds

Bend is affine. This is how it is fast (no GC; match frees the node)
and how arrays mutate in place. Fighting it with extra `+` is how
frames get 2x slower (see `bend guide shaders`).

## Quantities

| Mark | Meaning | Runtime |
|------|---------|---------|
| `-A` | Erased | Deleted; types and proofs only |
| (none) | Affine | At most one use; it may be discarded |
| `+x` | Reusable | Requires `Data`; permits more than one use |

```python
def replicate(-A: Data, n: Nat, +x: A) -> List<A>:
  match n:
    case 0n:
      Nil{}
    case 1n+p:
      x <> replicate(A, p, x)
```

Matching a `+` value hands out `+` fields. On a plain value, write
`+r` in the pattern to make a field reusable.

Erased parameters still appear in calls at the source (`replicate(U32,
3n, 7)`) but not at runtime.

The compiler decides borrow versus ownership; `+` alone does not guarantee a
refcount operation. Boxed sharing can add counts, while a reusable parameter
that is only borrowed may compile to a peek. Conversely, discarding a large
owned structure is permitted but its runtime destruction/sink is not
necessarily zero-cost. Inspect emitted C and measure hot paths.

## Kinds

Every type has a kind, which caps use:

- `Type` = `Kind(&1)` — at most once (functions, arrays, IO handles)
- `Data` = `Kind(&2)` — copyable
- `Kind(a)` parameter accepts both

```python
type List<a, -A: Kind(a)> is Kind(a):
  Nil{}
  Con{head: A, tail: List<a, A>}
```

A list is as reusable as its elements:

- `List<U32>` = `List<&1, U32>`
- `+List<U32>` = `List<&2, U32>`
- Closures: `List<&1, U32 -> U32>`

Two-element types combine with `a <&> b` (the min).

`type Shape is Data:` when values should be copyable. `is Type` when
they must not (unique resources).

## Closures vs defs vs templates

- Top-level `def`: callable any times.
- `x => e` / partial app: **one** call, even if captures are `Data`.
- `~f` template: checked once, instantiated/inlined at closed arguments,
  multi-shot, and may call only templates declared above it.

`List.map` is a template for this reason. Do not write:

```python
# Bad — f is used now and passed to the recursive call
def bad_map(-A: Type, -B: Type, f: A -> B, xs: List<A>) -> List<B>:
  match xs:
    case Nil{}: Nil{}
    case h <> t: f(h) <> bad_map(A, B, f, t)
```

## Arrays

`Array<T>` is `Type`: one checked owner. Bend 2.0.22 adds an explicit
`@unsafe` escape hatch for shared arrays; ordinary code remains single-owner.

```python
a = [0 : U32*8n]   # length power of two
a[5] <- 42         # in-place; rebinds a if not last statement
a[5]               # read: yields array & element
```

- Last statement `a[i] <- v` is the written array.
- `a[i]` sugar is `Array<U32>`. Other `T`: `Array.get` for `Data`
  elements, otherwise `Array.swap`, plus `Array.set`. `Array.clone`
  can duplicate an array only when its element type is `Data`.
- Indexes wrap.
- For safe code, do not hand an `Array` down a fork tree. Build cons lists for
  tiles (`bend guide shaders`) or move disjoint arrays into branches.
- `Array.fork(T, a)` creates two handles to one block in O(1), and
  `Array.join(T, a, b)` merges them. Both are `@unsafe`: document the aliasing
  protocol, join every fork, and prevent unsynchronized writes to one cell.
- Shared `U32` cells provide `Array.atomic.add/min/max/and/or/xor/exch/cas`;
  shared `F32` provides `Array.atomic.fadd`. Each returns the handle with the
  old value. A match on a shared handle copies its part as `Array.clone` does.
- Unbalanced `ALeaf`/`ANode` trees trap at runtime (WONTFIX).
  Construct arrays with the literals / Base ops.

## Handles

`File`, `Socket`, `Listener`, `Window`, and `Audio` are **laws of Base**
(opaque `Type`). User handle types are WONTFIX. Reuse Base's. Operations that
keep one of these handles live normally return it beside the result; creation
returns a new handle, while `close` consumes it. Check the installed signature.

`Chan(A)` is different: it is `Data`, so linked computations may each hold a
copy. Do not generalize channel behavior to the opaque affine handles.

## Sharing and hot types

`+` on a boxed value is a count. SHADERS.md:

- Borrow (checker needs `+` for a second **source** use; compiler may
  still `term_peek`) vs own (`term_keep` / `ctr_take`).
- A def that **returns** its argument shares it. Match and rebuild,
  or pass fields.
- Two consumers of one list on device: disaster (counts + Metal
  compile time).
- `_ = x` drops on the spot.
- Verify in `bend x.bend -o x.c`: `term_peek` = borrow; `term_keep`,
  `ctr_take`, `rfc_seal` = counts. New keeps in a device def are
  regressions.

CPU code: still avoid `+` you do not need. Sharing a `Data` tree
across a parallel let is supported (`tests/rfc/share_forked.bend`)
but you pay counts.

## Practical rules

1. Start affine. Add `+` only for actual reuse or when a callee's
   reusable contract requires it, and only on `Data`.
2. Put reusable element types in `List<&2, A>` / `+List<A>` when the
   algorithm copies heads (`insert`, `sort`).
3. Do not mark functions, arrays, or opaque `Type` handles `+` in safe code.
   Shared-array `+` belongs only inside a documented `@unsafe`
   `Array.fork`/`Array.join` protocol. `Chan(A)` is a separate copyable `Data`
   abstraction.
4. Prefer tail recursion that **moves** the structure (reverse onto
   an accumulator) over copying.
5. Drop unused forks' values; do not "return the scene" from a
   function that also returns it to a second consumer without a plan.
