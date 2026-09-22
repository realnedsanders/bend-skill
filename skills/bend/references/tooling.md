# Tooling

Bend is one binary. There is no cargo/npm for Bend projects yet. The
hub stores content hashes, not names or versions.

## Version and official source

Use `bend version` when the environment is unknown or after an upgrade. The
CLI guide and Base output are the authority for the binary that will compile
the project.

Demos, tests, `guide/EFFECTS.md`, and `bend2/effs/` live in the
[official Bend repository](https://github.com/bendlang/bend); they are not
necessarily installed with the CLI. Use a checkout/tag matching the version
from `bend version` when available. Do not assume the current working tree
contains the Bend compiler source. Foreign-effect internals have no
compatibility promise.

## CLI

```bash
bend version
bend --help
bend guide
bend guide shaders
bend base
bend base --types
bend base Map

bend file.bend              # check; run main
bend file.bend --check-only # check target and imports; run nothing
bend file.bend --checkup    # undocumented: check/run direct imports as roots
bend PROOF.bend             # fill-and-prove gate
bend file.bend -o out       # native (clang 14+; 19+ with !)
bend file.bend -o out.c
bend file.bend -o out.js    # JavaScript output
bend file.bend -o out.cjs   # CommonJS output
bend page.html -o dist
bend file.bend --publish    # print import 0x<hash>/main.bend as ...
bend update                 # prints curl | sh first
```

Behavior:

- `main : IO(_)` → compiled run.
- `main` returns a value → **normalize** (can be slow) and print.
- No `main` → check only.
- `PROOF.bend` next to `LAWS.bend` must import it.

Native run flags: `./out --threads N`, `./out --gpu 4GB`, and
`./out --gpu off`. GPU use is on by default when the program has bangs; the
`out.gpu` companion must stay beside `out`.

Linux extras: `libx11-dev` for Window, `libasound2-dev` for Audio.
CUDA 12 at `$CUDA_HOME` or `/usr/local/cuda`. No Windows; WSL works.

## What counts as a test

The language has **no** test framework.

**Projects:** `bend PROOF.bend` is the suite. Optionally a small
`main` that prints a known value.

**Bend's own repo:** a file ends with `#|` lines the run must print:

```
def main() -> IO(Unit):
  IO.print(U32.show(123))

#|123
```

Expected failure:

```
#|Error:
#|- expected : ...
#|exit 1
```

Do not add `#|` to an application unless you are inside that harness.

User-level stand-in: a `demo.bend` whose `main` prints a fixture, plus
laws. For IO, prefer proving the pure function that builds the
string.

## Dev loop

1. Edit.
2. If the project has `PROOF.bend`, run it; otherwise use `--check-only` when
   checking must not execute `main`, or check/run the edited target.
3. For IO iteration, emit JS and run it with **Bun**. Base's foreign JS effects
   can use `bun:ffi`; Node is only suitable when the generated program and its
   effects have no Bun dependency. Use native when testing threads/GPU.
4. Use `bend main.bend --checkup` when each direct aliased import must be
   checked and run as its own root; it does not replace checking `main.bend`.
   Relative imports resolve from the entry file; absolute import paths are
   used as written.

The Bun and Node `.bend` loaders report transitive unsafe or foreign
dependencies on stderr, like the CLI. Treat an unexpected warning as a review
failure even when loading succeeds.

Compiling native is slow (clang/CUDA/Metal). Do not use `-o binary`
as your unit-test loop.

## Editor

`bend2-fmt-lsp`: full-document format, stdio, language ids `bend` /
`bend2`. No hover, completion, or diagnostics. Wire format-on-save;
rely on `bend` for errors.

## Packages

```python
import 0x<hash>/main.bend as P
```

Fetched from the hub, hash-checked. `bend file.bend --publish`
uploads a file and its imports. No search, accounts, or semver yet.
Pin hashes. Do not pretend there is a package name.

## Limitations to remember in reviews

From upstream README / WONTFIX — do not "fix" these in user code:

- `@unsafe` still exits 0.
- No CSE: two `f(p)` calls run twice; bind once.
- Termination is structural. Before the first decreasing recursive argument,
  earlier arguments must pass through unchanged.
- Allocation failure is fatal, not `Fail`.
- JS has no graphics/audio; deep non-tail recursion can overflow the host stack.
- JS does not preserve `F32` NaN payload bits; native C is the bit reference.
- A `Nat` beyond 2^48-1 and an array block class beyond 31 fail-stop at runtime.
- Strings are slow (linked lists).
- One GPU, one event loop, one C file, no incremental compile.
