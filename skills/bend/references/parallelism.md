# Parallelism and GPU

Bend's parallel primitive is a **parallel let**, not threads.

```python
a b = pow2(p) pow2(p)
```

You promise:

1. Independent — guaranteed by safe purity + affinity; justify races yourself
   if you opt into unsafe shared arrays.
2. Similar runtime — you must keep the tree balanced.

The scheduler is contention-free binary fork-join. A task is handed
to a core once and never stolen. Unbalanced work = idle cores.

## CPU vs GPU vs JS

```python
pow2(20n)    # parallel CPU in a native binary
pow2!(20n)   # GPU (`!` after the name); CPU pool if no GPU
```

- Native `!` also builds `file.gpu` beside the binary. Keep them together.
  GPU use is on by default; `--gpu 4GB` caps its heap and `--gpu off` sends
  bangs to the CPU pool.
- Unified heap. Apple unified memory: CPU↔GPU is cheap. CUDA: first
  touch of a page faults over PCIe — keep a frame on one side.
- JS target: sequential; ignore `!`.
- GPU: uniform numeric work (mandelbrot, nbody, raster tiles).
- CPU: divergent search (n-queens, per-ray tree walk).

## Graphics case study — not universal rules

`bend guide shaders` records measurements from
`demos/app_slash_boss_3d`. Apply these only to similar image/tree workloads,
and profile on the target hardware:

1. That renderer uses **one bang per frame**, builds candidate lists on the
   host, forks to tiles, then runs flat tail loops.
2. Its ~16384-leaf / 4^7 frontier matches the documented machine's lane cube;
   it is a measured tuning point, not a language constant.
3. A def is *flat* when it has no parallel let, bang, closure apply, non-flat
   call, or non-tail self-call. Flat code becomes a native tail loop.
4. Its leaves walk short lists rather than performing a divergent tree walk
   per pixel.
5. Wide records and shared device-owned scene data were measured as costly.
6. Bang placement can change ownership: a bang in a `do` continuation may
   own a captured tree, while a pure wrapper can return `(image, scene)`.

For similar renderers, avoid forking per pixel, repeated bangs per frame,
non-tail recursion in device leaves, and shared counted lists. These are
performance findings, not checker requirements. Re-measure before copying
machine-specific widths or ownership tricks.

## Algorithmic parallelism (non-graphics)

Divide-and-conquer with equal subproblems:

```python
def sum(d: Nat, +i: Nat) -> Nat:
  match d:
    case 0n:
      i
    case 1n++p:
      a b = sum(p, i) sum(p, Nat.add(pow2(p), i))
      Nat.add(a, b)
```

Law the final parallel function against a sequential specification, then
rerun the proof after every fork or bang change. Release-matched examples are
available upstream in
[`demos/pure_par_sum`](https://github.com/bendlang/bend/tree/v2.0.25/demos/pure_par_sum)
and
[`demos/pure_par_sort`](https://github.com/bendlang/bend/tree/v2.0.25/demos/pure_par_sort).
Use a source checkout matching the installed release when possible.

Bend 2.0.22 also has experimental shared arrays. Use `Array.fork`/`Array.join`
and `Array.atomic.*` only behind a documented `@unsafe` aliasing protocol; see
[ownership.md](ownership.md). Prefer safe fork trees when they fit.

### Sorting decision

Do not equate "parallel sort" with "GPU-efficient sort":

- A recursive merge sort can expose balanced CPU fork-join work, but its merge
  is data-dependent and is not automatically a good GPU workload.
- A GPU request needs an algorithm with uniform work (for example a fixed-shape
  sorting network or a carefully designed radix approach), plus explicit laws
  for padding, bounds, order, and permutation.
- Clarify `List<U32>` versus `Array<U32>`, arbitrary versus power-of-two length,
  ascending unsigned order, padding policy, and whether GPU execution or merely
  parallel native execution is required.
- Base `List.sort` is useful executable code, but Base does not expose a ready
  sortedness/permutation proof contract. For a verified sort, use or adapt an
  evidence-carrying implementation whose decisions the proof can inspect.

A bang may silently use the CPU pool when no GPU is available. Report that
limitation instead of claiming device execution from source syntax alone.

## IO vs parallel pure

The event loop is one-threaded (Node-like). Pure work between
effects can still fork. `IO.fork` / `IO.spawn` start **computations**
that each run pure code until the next effect — that is concurrency,
not data-parallel `!`.

A bang that follows host forks in the **same** evaluation may run on
the CPU pool. `IO.now()` between host build and bang, as in the
slash-boss turn loop, is the documented way to split them.

## Older-toolchain diagnostics

Before restructuring valid code around a GPU memory fault, reproduce it on the
current compiler. Bend 2.0.23 fixed device fork spines that continued past 128
levels, and 2.0.24 fixed fork-free leaves under `!` that could exhaust lane
memory during deep monadic loops.
