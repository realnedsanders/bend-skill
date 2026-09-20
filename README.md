# bend-skill

An agent skill for [Bend](https://github.com/bendlang/bend) 2.

It tells an agent how to write Bend the way the language is actually used:
preserve project conventions, apply laws to critical pure behavior, default to
affine values, use structural recursion and proofs-as-programs, query Base, and
balance fork-join work. It does **not** copy
`bend guide`. Agents still run `bend guide` and `bend base`.

## Install

### From [skills.sh](https://skills.sh)

```bash
npx skills add https://github.com/realnedsanders/bend-skill --skill bend
```

### From a local clone

From this repository root, choose one install scope and either copy or symlink:

```bash
# User install (copy)
mkdir -p ~/.agents/skills
cp -R "$PWD/skills/bend" ~/.agents/skills/bend

# User install (symlink for development)
mkdir -p ~/.agents/skills
ln -s "$PWD/skills/bend" ~/.agents/skills/bend

# Project install
mkdir -p /path/to/project/.agents/skills
cp -R "$PWD/skills/bend" /path/to/project/.agents/skills/bend
```

Requires Bend 2 (`bend --version`). Detailed language claims were checked
against the installed Bend 2.0.10 CLI. Source research also used
`bendlang/bend` commit `15ae0c86f3193b8f645b4bedbc438655b648d0da`;
when they differ, the installed CLI governs the project being compiled.
Bend 1 / HVM programs do not apply.

## Layout

| Path | What |
|------|------|
| [`skills/bend/`](skills/bend/) | The skill (`SKILL.md` + `references/`) |
| [`docs/`](docs/) | Research notes (Bend, Go/Lean skills, design) |
| [`adr/`](adr/) | Architecture decisions |

## What the skill enforces

- Default project: `main.bend` + `LAWS.bend` + `PROOF.bend`
- Gate: `bend PROOF.bend` → `All terms check.`
- No invented `if`, tactics, typeclasses, inference, or `match f(x)`
- Decreasing argument first; `@unsafe` only for the outside world
- `+` only for real reuse of `Data`
- `Type.verb` names; look up Base instead of reinventing it

## Validate the skill

```bash
uvx --from skills-ref agentskills validate ./skills/bend
```

Then check the Markdown links and run selected Bend examples against the target
`bend --version`; the language is young enough that schema validation alone is
not sufficient.

CI runs the skill validator and checks Markdown links on every pull request and
push to `main`.

## Dependency updates

[Renovate](https://github.com/apps/renovate) watches Bend releases and the
GitHub Actions used by CI. Its Dependency Dashboard issue lists new Bend
releases. Bend updates require approval in that issue before Renovate opens a
pull request, because the skill must be checked against each new compiler
release rather than having its compatibility claim changed silently.

To enable it, install the free Renovate GitHub App for this public repository
and complete its onboarding pull request if prompted. Configuration lives in
[`renovate.json`](renovate.json).

## Sources

- Installed `bend` 2.0.10 (`bend guide`, `bend base`)
- Official repo `bendlang/bend` (demos, tests, evals, AGENTS.md)
- Go skills `golang-patterns` / `golang-testing` (shape)
- Lean 4 practice by contrast (no tactics skill existed locally)

See [docs/skill-design.md](docs/skill-design.md) and the ADRs.
