# IO and effects

Effects live in `IO`. Pure code is the default. Proofs, termination,
and the GPU never run host code.

## do-notation

Works for any monad with `M.bind` and `M.pure` (`IO`, `Maybe`,
`Result`, yours).

```python
def greet(name: String) -> IO(String):
  do IO<String>:
    IO.sleep(1000)
    return "Hello, " ++ name

def main() -> IO(Unit):
  do IO<Unit>:
    name : String <- IO.try(String, IO.get_env("USER"))
    chan : Chan(String) <- IO.fork(String, greet(name))
    IO.print("Waiting...")
    text : String <- IO.join(String, chan)
    IO.print(text)
```

Rules:

- Annotate every bind: `x : T <- m`.
- `x : T = v` is a pure let and annotates `v`; only a name may be typed.
  Destructure a typed value in the following body instead of typing a pattern.
- A statement `m` is a `Unit` step.
- `return e` is `M.pure`.
- Leading type args of the monad (`Maybe<&2, U32>`) are passed to
  `bind` / `pure`.

`IO.try` unwraps `Result` or exits. `IO.die` exits with your error.
`IO.args` is argv without the runtime's flags (`--` ends them).
Bend 2.0.22 also provides `IO.random_u32`, `File.read_at`, `File.size`,
`File.write_bytes`, and `TCP.poll`; query their exact Base signatures.

## Handles

Opaque handles such as `File` and `Socket` are affine. Operations that keep
a live handle normally return it beside the result. Creation returns a new
handle inside `Result`; `close` consumes the handle and returns `IO(Unit)`.
Follow the exact installed Base signature rather than assuming one uniform
shape.

```python
def http_talk(s: Socket) -> IO(Unit):
  do IO<Unit>:
    read : Socket & Result<&1, &1, U32 & String, String> <- TCP.recv(s, 8192)
    http_reply(read)
```

Close on both `Done` and `Fail`. Do not forge handles. Do not define
a new handle `type` (WONTFIX: only Base may leave handle laws open).

## Concurrency

One event loop. Each computation runs pure (possibly parallel) until
the next effect. Waiting on socket/sleep/channel yields.

- `IO.fork` / `IO.join` — result channel
- `IO.spawn`, `Chan.new`, `Chan.send`, `Chan.recv`, `Chan.close`

The process ends when every computation is done, or reports deadlock
if they all wait.

Servers that loop forever: `@unsafe` + comment, or fuel. Prove the
per-connection **pure** function (`http_response`), not the accept loop. See
release-matched
[`demos/io_http_server`](https://github.com/bendlang/bend/tree/v2.0.22/demos/io_http_server)
or the corresponding demo in a matching source checkout. `TCP.poll` adds a
receive deadline for dropping idle connections.

## Graphics apps

`App.run(~S, ~App{+s => (s, image), tick}, title, w, h, init)`.

- `view` is pure: state in, `(state, Image)` out (state is affine).
- `tick(events, state) -> IO(Maybe<S>)`; `None` quits.
- `Image` is a quadtree: `Pix{color}`, `Qua{tl,tr,bl,br}`.
- Events: `Key`, `Mouse`, `Move`, `Close`.
- Underneath: `Window.open` / `frame` / `close`; `Audio.*`.

Graphics performance: [parallelism.md](parallelism.md) and
`bend guide shaders`.

## Foreign effects

```python
def Clock.now() -> IO(U32):
  import "./clock.c"
  import "./clock.js"
```

- Host symbol: def name, lowercased, dots → underscores (`clock_now`).
- `.c` for native; `.js` for JS and `bend file.bend`.
- The C backend defines the run function and registers it with
  `io_eff(CID_..., run, need)` from a constructor. A blocking JS effect also
  supplies `<name>_need` and parks/resumes through the runtime API. Copy the
  matching upstream effect; these runtime names are version-coupled.
- Need 0 (run now), `IO_READ`, `IO_TIME`, etc. Read `bend guide effects`,
  but treat a matching checkout's `bend2/effs/` and compiler as the ABI source
  of truth. The Bend 2.0.22 guide still shows an obsolete extra `hot` argument
  to `io_node`; current sealing derives from constructor identity.
- **No ABI promise.** Rebuild and retest effects on every Bend upgrade.
- Output binary name must not collide with the effect files.

Foreign effects are part of the trusted computing base. Bend does not prove
the imported C or JS. Prefer Base effects. When a custom effect is necessary:

- Audit both backends for equivalent observable behavior.
- Return affine handles on success and failure as required by the Bend type.
- Use `Result` for recoverable host errors; do not silently turn them into values.
- Follow upstream ownership and blocking patterns rather than retaining runtime
  pointers or blocking the event loop ad hoc.
- Test native and JS paths separately.

JS calling Bend: `import Game from "./game.bend"` with `bend2/main.ts`
preloaded. Constructors `{ $: "Name", field: value }`, `Nat` as
`BigInt`. An `Array` argument **is** the caller's array (in-place).
Copy first if you keep it. Only non-IO defs.

## Laws about IO

You can equate IO terms definitionally (`Hello.main() ==
IO.print("...")`). You cannot usefully quantify over the scheduler.
Pin pure helpers that IO calls.
