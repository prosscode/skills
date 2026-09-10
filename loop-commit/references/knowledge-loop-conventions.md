# Knowledge Loop Conventions

Use the canonical block below when `SKILL.md` requires knowledge-loop reconciliation. Translate its
prose to match an existing instruction file when appropriate, but keep literal identifiers and
all-uppercase field names unchanged.

```markdown
## Knowledge Loop Conventions

### Before editing code

1. Resolve every module the change touches through `.codex/MODULES.md`; map `/` in an ID to subdirectories.
2. Read `.codex/maps/<module>.md` and `.codex/decisions/<module>.md` when present for every affected module. Do not load unrelated module files.
3. Run `git log --oneline -10 -- <path>` for files about to change.
4. When recent commits contain `MODULE: <current module>`, inspect those commit bodies with `git show`.

### After finishing a task

- If implementation changes make an existing module map inaccurate about responsibilities, public entry points, lifecycle or data flow, dependencies, invariants, or known limitations, use `$loop-map` in targeted-refresh mode when available; otherwise report the map as stale before finishing. Use a full refresh when the impact cannot be bounded. Leave the map unchanged when its documented claims remain true.
- When using `$loop-commit`, fill `MODULE`, `WHY`, `ALTERNATIVES`, `CHOSEN`, `TRADEOFFS`, and `RISKS` in every Decision.
- When a new Decision replaces one in `.codex/decisions/`, add `SUPERSEDES`.
```
