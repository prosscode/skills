# Personal Codex Skills

An evolving personal collection of reusable Codex skills. The repository can contain skills for
different workflows and domains; each top-level skill directory is intended to be independently
discoverable and usable.

This repository is itself managed by the three `loop-*` skills described below. They maintain the
repository's module registry, architecture map, and decision history; other skills in the
collection may remain independent of that workflow.

## Current skills

The repository currently includes a coordinated Knowledge Loop skill group:

| Skill | Purpose |
| --- | --- |
| `$loop-map` | Research and maintain a source-verified architecture map for a registered module. |
| `$loop-commit` | Create a Git commit that records the staged change together with relevant conversation context and material decisions. |
| `$loop-distill` | Derive a current decision snapshot from structured `MODULE` blocks in Git history. |

Other independent skills may be added to this repository without participating in the Knowledge
Loop or sharing its `.codex/` artifacts.

## Knowledge Loop skill group

The three `loop-*` skills share the module registry in `.codex/MODULES.md` and keep different kinds
of knowledge in separate artifacts:

1. Use `$loop-map` to document how a module works today under `.codex/maps/`.
2. Use `$loop-commit` to capture why a staged implementation change was made in Git history.
3. Use `$loop-distill` to consolidate those immutable decision records into
   `.codex/decisions/`.
4. Continue using the map and decision snapshot as context for later work; refresh a map when an
   implementation change makes its documented facts inaccurate.

Architecture maps describe current behavior, while decision snapshots describe current consensus.
Neither replaces source code, tests, CI, or the Git history that produced it.

## Repository layout

```text
.
├── <skill-name>/      # An independently usable Codex skill
│   ├── SKILL.md       # Skill entrypoint
│   ├── agents/        # Optional host interface metadata
│   └── references/    # Optional workflow-specific guidance
├── loop-commit/       # Current Knowledge Loop skill
├── loop-map/          # Current Knowledge Loop skill
├── loop-distill/      # Current Knowledge Loop skill
├── .codex/
│   ├── MODULES.md     # Knowledge Loop module registry
│   ├── maps/          # Knowledge Loop architecture snapshots
│   └── decisions/      # Knowledge Loop decision snapshots
├── AGENTS.md          # Repository guidance and workflow conventions
└── LICENSE            # MIT License
```

Each skill should contain a `SKILL.md` entrypoint. An `agents/openai.yaml` interface definition and
`references/` for workflow-specific guidance are optional, depending on the skill.

## Design principles

- Keep the module ID stable across maps, decisions, and commits.
- Treat the final staged diff as the commit boundary; unrelated working-tree changes stay out.
- Keep source-verified facts separate from decision history and design proposals.
- Prefer bounded targeted refreshes when a change affects only a small set of mapped claims.
- Preserve Git history as the authoritative record of implementation decisions.

## License

Licensed under the [MIT License](LICENSE).
