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

Requires Bend 2 (`bend version`). Detailed language claims were checked
against the official Bend 2.0.25 Linux release binary. Source research used
`bendlang/bend` tag `v2.0.25` at commit
`c65bcb788dbfb298bb434c1d858b47c193841dc0`;
when they differ, the installed CLI governs the project being compiled.

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
- Decreasing argument first; `@unsafe` only at documented trusted boundaries
- `+` only for real reuse of `Data`
- `Type.verb` names; look up Base instead of reinventing it

## Validate the skill

```bash
uvx --from skills-ref agentskills validate ./skills/bend
```

Then check the Markdown links and run selected Bend examples against the target
`bend version`; the language is young enough that schema validation alone is
not sufficient.

CI runs the skill validator and checks Markdown links on every pull request and
push to `main`.

## Dependency updates

[Renovate](https://github.com/apps/renovate) watches Bend releases and the
GitHub Actions used by CI. Its Dependency Dashboard issue lists new Bend
releases, and Renovate opens a grouped pull request immediately. Bend pull
requests never automerge: check the skill against the new compiler before
merging a changed compatibility claim.

## Releases

GitHub releases use `v<bend-version>-skill.<revision>`, such as
`v2.0.25-skill.0`. The first skill release for a Bend version is `.0`;
skill-only fixes increment the revision, and a new Bend version starts again at
`.0`. Bare Bend-version tags are not published.

After validation and link checks pass on `main`, CI creates a release when
`skills/bend/` changed. A commit already carrying a release tag is a no-op, and
repository-only changes do not increment the revision. The CI workflow's manual
dispatch can recover a missed release or explicitly force the next revision.

## Sources

- Official `bend` 2.0.25 release binary (`bend guide`, `bend base`)
- Official repo `bendlang/bend` (demos, tests, evals, AGENTS.md)
- Go skills `golang-patterns` / `golang-testing` (shape)
- Lean 4 practice by contrast (no tactics skill existed locally)

See [docs/skill-design.md](docs/skill-design.md) and the ADRs.
