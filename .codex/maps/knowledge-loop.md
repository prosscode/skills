# knowledge-loop Map

> Static understanding snapshot, not a decision history.
> See `.codex/decisions/knowledge-loop.md` for the paired decision history; it has not been distilled yet.
> Verified: 2026-09-10 (12 cross-checked claims: 9 confirmed, 3 partial)

## Responsibilities

The Knowledge Loop coordinates three repository-local skills around one stable module registry. `loop-commit` records implementation context and decisions in Git, `loop-map` maintains verified current-architecture snapshots, and `loop-distill` derives current decision consensus from immutable commit history.

## Key types

- `.codex/MODULES.md` — Registry of exact legal module IDs shared by all three skills.
- Git `## Decision` block — Immutable decision event produced by `loop-commit`.
- `.codex/maps/<module-id>.md` — Source-verified current-architecture snapshot produced by `loop-map`.
- `.codex/decisions/<module-id>.md` — Derived current-consensus snapshot produced by `loop-distill`.
- Knowledge Loop Conventions — Root instructions that load relevant module knowledge before editing and maintain stale maps afterward.
- `single`, `all`, and `next` — Module-selection modes supported by `loop-distill`.
- Full mapping and targeted refresh — New-map and bounded-maintenance modes supported by `loop-map`.

## Public entry points

- `$loop-commit` — Commit the explicit staged boundary with relevant conversation context and material Decisions.
- `$loop-map` or `/loop-map` — Create, fully verify, or selectively refresh a module architecture map.
- `$loop-distill` — Consolidate one registered module, all modules, or the next due module from Git Decision history.

## Data flow / lifecycle

1. A durable module ID is registered in `.codex/MODULES.md` by `loop-commit` during normal development or by `loop-map` after verified legacy research.
2. `loop-commit` binds the final staged diff, visible conversation, and material choices into a Git commit containing independent Decision blocks.
3. `loop-distill` locates exact `MODULE: <id>` tags in commits reachable from `HEAD`, resolves supersession and clustering, and writes an approved current-consensus snapshot.
4. `loop-map` researches current source, independently verifies important claims, and writes an approved current-architecture snapshot under the same module ID.
5. Root Knowledge Loop Conventions direct later tasks to resolve affected modules, read their maps and decisions, inspect relevant Git history, and refresh maps only when implementation changes invalidate documented facts.
6. Outputs from map and distill return to `loop-commit`, closing the loop without either skill creating Git commits.

## Dependencies (inbound / outbound)

- Inbound to `loop-commit`: explicit staged scope, current conversation, registered module IDs, recent commit style, and user approval for registry or instruction changes.
- Outbound from `loop-commit`: immutable Git history, structured Decision blocks, module registry updates, and approved Knowledge Loop instructions.
- Inbound to `loop-distill`: `.codex/MODULES.md`, Decision-bearing commits reachable from `HEAD`, and an optional existing snapshot as the stable-ID baseline.
- Outbound from `loop-distill`: `.codex/decisions/<module-id>.md`, left unstaged for review and later commit.
- Inbound to `loop-map`: current source at `HEAD`, an optional existing map and paired decision snapshot, relevant diffs, and supplied dependency source.
- Outbound from `loop-map`: `.codex/maps/<module-id>.md` and verified legacy registry entries, followed by a handoff to `loop-commit`.

## Invariants and gotchas

- Existing staged changes are the `loop-commit` boundary; an empty index requires an explicit all-changes or named-path selection and never implies `git add -A`.
- Only exact IDs from `.codex/MODULES.md` are legal `MODULE` values.
- A Decision represents one material choice; one commit may contain several Decisions using the same module ID.
- Git history is authoritative for decisions; decision snapshots are derived and never rewrite history.
- Maps describe current behavior, not design proposals or decision history.
- A registered module may temporarily have no map or decision snapshot.
- `/` in a module ID maps to subdirectories under both `.codex/maps/` and `.codex/decisions/`.
- Incremental distillation retains the existing snapshot and stable `D<n>` IDs rather than rebuilding from only new commits.
- Targeted map refresh leaves the map unchanged when its documented claims remain accurate and escalates when impact cannot be bounded.
- Distillation currently orders decisions by author date; rebases, cherry-picks, or clock differences may not reflect commit ancestry.

## Confirmed bugs / technical debt

- `loop-distill` and `loop-map` retain generic UI labels instead of the shared `Knowledge Loop · Distill` and `Knowledge Loop · Map` convention used by `loop-commit`.
- `loop-map` forbids commit and push globally, but only its legacy-bootstrap path explicitly forbids staging; the general map workflow should state the same staging boundary.

## Open questions

- Should supersession resolution follow Git ancestry or topological order, using author date only for presentation?
- Should `loop-distill all` define a separate lightweight count-only scan before full Decision parsing?
- Should the general `loop-map` guardrails explicitly prohibit staging in every mode?

## To verify

- Confirm that a fresh Codex session discovers and invokes the three repository-local skill copies as expected.
- Exercise targeted refresh against a later implementation change to an existing verified map.
- Exercise snapshot compression after a module exceeds the full-detail decision threshold.
