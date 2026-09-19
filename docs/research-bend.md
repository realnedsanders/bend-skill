# Research: Bend language (docs and code)

Sources, in order of authority:

1. Installed CLI: `bend` 2.0.10 (`bend guide`, `bend --help`, `bend base`)
2. Official repo: `/home/user/code/github.com/bendlang/bend`
3. `AGENTS.md`, `README.md`, `WONTFIX.txt`
4. `guide/GUIDE.md`, `guide/SHADERS.md`, `guide/EFFECTS.md`
5. `bend2/base.bend` (via `bend base` / `bend base --types`)
6. `demos/*` (especially `LAWS.bend` / `PROOF.bend` / `main.bend`)
7. `tests/proof`, `tests/halt`, `tests/run`, `tests/rfc`
8. `evals/*.bend` (how Bend expects AIs to prove programs)

This note is what the skill must encode. It is not a second copy of the
guide. Agents writing Bend should still run `bend guide`.

## What Bend is

Bend is a dependently typed, affine language. Syntax looks like Python.
Semantics are closer to Haskell/Lean, with Rust-like resource awareness and
mandatory termination. It checks in one linear bidirectional pass and
compiles to C (CPU + Metal/CUDA GPU) and JavaScript.

The language's pitch for agents is precise:

- Humans write **laws** (specs as types).
- AIs write **code and proofs**.
- `bend PROOF.bend` is the gate: open or false laws fail the build.

`LAWS.bend` is described in the README as "`AGENTS.md` backed by proof".

Official agent snippet from the README:

```
When using Bend:
- run `bend guide` to learn it
- use `LAWS.bend` to keep important rules
- run `bend PROOF.bend` before committing
- parallelize the code whenever possible
```

That is the minimum. A skill must turn it into hygiene, not just reciting
the four bullets.

## Repo map (what not to confuse)

From `AGENTS.md`:

| Path | Role |
|------|------|
| `bend2/bend.ts` | Language: parser, theory, checker. Human-written. Do not edit. |
| `bend2/comp.ts` | Compiler and runtimes (C, Metal, CUDA, JS) |
| `bend2/main.ts` | CLI; also the `.bend` loader for bun/node |
| `bend2/base.bend` | Prelude |
| `bend2/bend.lean` | Core mechanized in Lean (lags `bend.ts`) |
| `bend2/effs/` | One file per IO effect, per backend |
| `tests/<ns>/` | Tests are Bend files ending in `#|` expected output |
| `demos/` | One dir per demo, almost always with `LAWS.bend` |
| `evals/` | Law-and-proof tasks used to grade models |
| `guide/` | GUIDE, SHADERS, EFFECTS |
| `tools/bend-fmt-lsp` | Formatting-only LSP; no diagnostics/hover |

The skill is for **writing Bend programs**, not for hacking the TypeScript
compiler.

## Language facts that agents get wrong

These are the traps. Every one appears in the guide, halt tests, or
WONTFIX.

### Syntax that does not exist

- No `if` / `else`. Branch with `match` on `True{}` / `False{}`.
- No tactics, no proof search, no `sorry`. A proposition is a type; a
  proof is a `def`. Holes are `?TODO` (open) or `?name` (print the goal).
- No type classes, traits, or macros beyond compile-time `~` templates.
- No inference worth relying on. Annotate. When the checker cannot decide,
  write `{e : T}` or `(a + b : U32)`.
- No mutual recursion. Encode two functions as one `def` plus a tag.
- No `match` on a computed value. `match sum(xs, 0):` is rejected.
  Scrutinees are parameters or pattern-bound variables. Computed dispatch
  goes through a helper (`foo.fin`) that matches its parameter. Demos and
  evals do this constantly.
- Operators need spaces on both sides. `(a + b : T)` calls `T.add`.
  Without `: T` they belong to `Nat`.
- `==` is the equality **type** `{a == b : T}`. Value equality is
  `T.is_eq(a, b)`.

### Affine by default

- A variable is used at most once unless marked `+` (reusable) and the
  type is `Data` (`Kind(&2)`).
- `-` is erased: present in types/proofs, deleted at runtime.
- Closures are affine even when they capture `Data`. Only **top-level
  defs** may be called freely. Partial apps like `U32.add(2)` are
  closures.
- `Array<T>` is `Type` (exactly one owner). `a[i]` returns the array
  beside the element. Writes are `a[i] <- v`. Sugar `a[i]` assumes
  `Array<U32>`; other element types use `Array.get` / `Array.swap` /
  `Array.set`.
- Handles (`File`, `Socket`, `Window`, `Audio`) are affine and opaque.
  Every effect on a handle hands it back beside the result.
- Dropping an affine value is free. Sharing a `+` value is a refcount.
  SHADERS.md: unnecessary `+` on device data can 2x a frame.

### Termination

- Recursion must structurally decrease. Arguments are read **left to
  right**: each is passed unchanged until one is a smaller part of its
  parameter; later args are free. Put the shrinking argument first.
- `tests/halt/non_structural_descent.bend`: `fold(ndiv(n, 2n))` is
  refused even if it would halt. `ndiv` is not a constructor subterm.
- World-bounded loops (servers) count down a `Nat` fuel argument, or
  mark `@unsafe`. `@unsafe` skips the termination check and **falls
  outside proof guarantees**. Checker still exits 0 (WONTFIX #776).
- `tests/run/fuel_loops.bend` is the fuel idiom (gcd, Collatz).

### Parallelism

- `a b = f(x) g(y)` is a parallel let: independent, should take similar
  time. Scheduler is binary fork-join; tasks are never stolen.
- `f!(x)` ships that call and nested parallel calls to the GPU (or CPU
  pool if no GPU). JS target is sequential.
- GPU likes uniform numeric work. Divergent search stays on CPU.
- SHADERS.md (AI-for-AI, from `demos/app_slash_boss_3d`): one bang, fork
  tree that fills the lane cube, then **flat loops** over cons lists.
  Do not fold per pixel, do not recurse non-tail inside a bang, do not
  let the device own the scene.

### Proofs

- `law name:` claims a type. `def name` or `def M.name` fills it.
- Project convention: `LAWS.bend` (human, AI does not rewrite claims)
  and `PROOF.bend` (AI). `bend` refuses a `PROOF.bend` beside a
  `LAWS.bend` that it does not import.
- `{==}` is reflexivity when both sides compute to the same term.
- `%e : P` rewrites with `e : {a == b : T}`; `_` in `P` marks `b`.
- Recursive call in a proof is the induction hypothesis.
- `exs y: T` in a law: proof returns `(y, proof)`.
- `for y: B where P(y)`: `y` is the pair `(y, P(y) proof)`.
- Empty match on `e : Empty` closes a branch. `{a != b : T}` is
  `{a == b : T} -> Empty`.
- F32 is axiomatic: nothing about floats can be proven.
- No positivity check; `Type : Type` holds. Consistency is the live/dead
  split: live recursion terminates; dead code may loop or inhabit Empty
  but is not live evidence.

### IO

- Effects live in `IO`. `do IO<R>:` with `x : T <- m` and `return e`.
- Every bind is annotated. `x : T = v` is a pure let inside `do`.
- Fallible effects return `Result`. `IO.try` unwraps or exits.
- `IO.fork` / `IO.join` / `IO.spawn` / channels: one event loop, Node-like.
- Custom effects: body is `import "./x.c"` plus `import "./x.js"`. Host
  name is the def, lowercased, dots to underscores. No ABI promise
  across Bend versions (`guide/EFFECTS.md`).
- User handle types are WONTFIX: reuse Base handle laws.

### Modules and names

- A file is a module. `import ./math.bend as M` then `M.square`.
- Dots in names are characters: `U32.show` is one name, not a path.
- Base scheme: `Type.verb`. Same verbs on `Nat`, `U32`, `F32`:
  `add sub mul div mod`, bits on U32, `cmp` / `is_eq` family, `show` /
  `read`.
- `bend base`, `bend base --types`, `bend base Map`.
- Packages: `import 0x<hash>/main.bend as P`. Hub has no names/versions
  yet. `bend file.bend --publish`.

### Testing in the Bend repo

There is no test framework. A test is a `.bend` file whose run must
print the lines after `#|`. Failures use `#|Error:` and `#|exit 1`.
`gates/test.ts` shards them.

For user projects: `bend file.bend` checks; a pure `main` is normalized
and printed (slow for big work); an `IO` `main` runs compiled. A file
with no `main` just checks. `bend file.bend --checkup` checks each
import alone.

### Tooling limits (skill must not pretend otherwise)

From README limitations and WONTFIX:

- No Windows (WSL works). clang 14+; `!` needs 19+ and Metal or CUDA 12.
- No debugger, profiler, REPL. Error messages are terse.
- Editor support is formatting only (`tools/bend-fmt-lsp`).
- Compiling to native is slow; JS is the fast dev loop.
- Strings are cons lists of `Char`; text processing is slow.
- Numbers: Nat, U32, F32 only.
- Base is small. Expect to write helpers.
- No TLS/HTTP/JSON/regex in Base (demos show tiny HTTP by hand).
- One C file per program; no incremental builds.

## Demo patterns worth copying

Almost every demo is three files:

```
main.bend     implementation
LAWS.bend     import main as M; law ...
PROOF.bend    import main; import LAWS as Laws; def Laws....
```

Examples:

- `demos/io_hello_world`: law is definitional (`main == IO.print(...)`),
  proof is `{==}`. Even a hello world has a closed law.
- `demos/proof_insertion_sort`: computational content carries evidence
  (`le_case` returns `Or(LE(x,h), LE(h,x))`). Proofs match the same
  verdict via `.fin` helpers. Laws: sortedness + permutation by counts.
- `demos/pure_par_sum`: parallel tree vs sequential spec. Proof is
  induction + assoc/zero lemmas.
- `demos/app_pong_game_2d`: laws about Esc and last-key-wins. Proof of
  last-press is exhaustive Bool case analysis (`press4`).
- `demos/app_win_is_bug_2d`: "winning is impossible" over all move
  lists. This is the README's flagship.
- `demos/io_http_server`: laws pin the **pure** part (`http_response`).
  The accept loop is `@unsafe`. Do not try to prove the event loop;
  prove the bytes.

Evals (`evals/cake.*`, `evals/easy.*`) show the preferred AI style:
short comments on intent, `law` then `def` of the same name, induction
with `%ih : { ... _ ...}`, `LE` as a type, helpers named after the
claim (`sum_pour`, `pour_bounded`).

## Base types (from `bend base --types`)

`Empty`, `Unit`, `Bool`, `Cmp`, `Either`/`Or` (`Inl`/`Inr`), `Sigma`/
pairs, `Nat` (`Zero`/`Succ`, literals `3n`), `Maybe`, `Result`,
`List` (`Nil`/`Con`, sugar `[a,b]`, `h <> t`), `U32`, `F32`, `Char`,
`String` (`SNil`/`SCon`), `Array` (`Type`), `Image`, `Event`, `Map`,
opaque affine handle laws (`File`, `Socket`, `Listener`, `Window`, `Audio`),
copyable `Chan(A): Data`, `IO`, and `App`.

`List<a, A>` is as reusable as its elements. `List<U32>` is
`List<&1, U32>`; `+List<U32>` is `List<&2, U32>`.

`List.map` is a **template** (`~A`, `~B`, `~f`). Closures cannot be
called more than once; map inlines `f`.

## Implications for a skill

1. Teach workflow and hygiene, not a second GUIDE.md.
2. Make law-driven development the default, not an advanced topic.
3. Encode the syntactic non-features as hard "never invent" rules.
4. Treat `@unsafe`, extra `+`, and GPU bangs as cost/soundness knobs.
5. Point at `bend guide`, `bend base`, and official demos as oracles.
6. Mirror Go skills: principles, good/bad, anti-patterns, commands,
   checklist — not a language textbook.
