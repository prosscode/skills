---
name: loop-map
description: Create, fully refresh, or incrementally maintain a source-verified structural or cross-cutting architecture map under `.codex/maps/`, including legacy-project registry bootstrap. Use when the user invokes `$loop-map` or `/loop-map`, asks to map, document, understand, or refresh a module, or project instructions require targeted maintenance after implementation changes invalidate mapped facts. Do not use for a single-file question that leaves the existing map accurate. Authorizes bounded, read-only explorer subagents for research and verification.
---

# Loop Map

Produce a “how it works today” snapshot. Keep decision history in the paired
`.codex/decisions/<module-id>.md`; do not turn the map into a design proposal or changelog.

Read `references/report-contracts.md` completely before full research or verification. Use its task
contracts, sizing rules, verdict application, failure handling, and final map template.

## Choose the mode

- Use **full mapping** for a new map, an explicitly requested full refresh, broad architectural
  changes, an unverified existing map, or a change whose impact cannot be bounded confidently.
- Use **targeted refresh** when an existing verified map is being maintained after a bounded code
  change and only a limited set of mapped claims may have changed.
- If a targeted assessment proves that all mapped claims remain accurate, leave the map unchanged
  and report that no refresh was needed.

## Ownership and collaboration

- Keep synthesis, verdict judgment, file edits, module registration, and the final response in the
  root agent.
- Explorers are read-only. Make verification independent from the research that produced each claim,
  follow available concurrency, and expose expected research and verification cost before mapping
  multiple modules or a very large subsystem.

## Phase 0: Scope and mode

1. Resolve `<module-id>` and the subsystem root. Derive them from the request and repository when
   safe; ask only if different interpretations would materially change the map.
   - A **structural module** normally has one primary source root.
   - A **cross-cutting module** may have several explicit roots or entry points connected by one
     durable contract. Record what is inside and outside that boundary; do not use a broad theme as
     a substitute for a verifiable module.
2. Read `.codex/MODULES.md` when present, the paired decision file if present, the current map if
   refreshing, and directly related maps. A registered module may temporarily have no map; mapping
   remains demand-driven rather than a prerequisite for development.
3. Confirm that the root exists at the current `HEAD`. If it is missing, inspect Git history and
   symbol moves to distinguish a deleted historical subsystem from a renamed successor. Do not map
   a historical commit or switch module ids without user confirmation.
4. Inspect the root with `rg --files`, class declarations, constructors or registration points,
   public entry points, core state fields, and file sizes. Skim key files to confirm boundaries.
5. Choose full mapping or targeted refresh using the criteria above. If the current task's change
   boundary cannot be separated from unrelated working-tree changes, do not guess; use full mapping
   or report the unresolved boundary.
6. For full mapping, split work by concern rather than file and follow the sizing rules in the report
   contracts.
7. Build a short shared context containing the module purpose, root, core files, known related
   maps, and any external dependency source supplied through the task or environment.

### Legacy registry bootstrap

When mapping an existing subsystem and `.codex/MODULES.md` does not exist, read
[references/legacy-bootstrap.md](references/legacy-bootstrap.md) completely before research and
follow its provisional-ID, verification, approval, and handoff rules.

## Targeted refresh

Skip the full research and adversarial-verification phases only when the targeted-refresh criteria
are met.

1. Use the current task and its staged and unstaged diff to identify the implementation change
   boundary. Preserve unrelated user changes.
2. Compare that boundary with the current map and list every potentially affected claim about
   responsibilities, key types, public entry points, lifecycle or data flow, inbound or outbound
   dependencies, invariants, bugs, open questions, and verification residuals.
3. Reopen the relevant source and verify every affected claim plus its immediately adjacent edges.
   Inspect dependency source supplied through the task or shared context when required. If required
   external source is unavailable, retain a concise residual under `To verify` instead of guessing.
   Do not infer that a claim is unchanged solely because its named file was untouched.
4. Escalate to full mapping if the affected claims span most sections, reveal an undocumented
   subsystem boundary, depend on an unverified baseline, or the internal impact cannot be checked
   from available project source. Missing external source alone is not a reason to escalate.
5. If no documented claim changed, do not edit the map.
6. Otherwise, patch only the affected statements and sections. Preserve the previous full
   `Verified` line and add or replace this line immediately after it:

   ```markdown
   > Maintained: YYYY-MM-DD (targeted verification: <bounded change summary>)
   ```

7. Keep evidence out of the map, run the validation steps below, and report which mapped claims
   changed. Do not commit or push unless the user explicitly asks.

## Phase 1: Full research

Follow the full-research workflow in `references/report-contracts.md`. Require first-hand source
evidence in reports, keep that evidence out of the final map, and never synthesize a failed concern
as established fact.

## Phase 2: Full adversarial verification

Follow the adversarial-verification workflow and sizing rules in `references/report-contracts.md`.
The root agent judges returned evidence; low-confidence or unsupported verdicts remain unresolved.

## Phase 3: Full patch

Apply verdicts using `references/report-contracts.md`. Add or replace
`> Verified: YYYY-MM-DD (verification summary)` only after full verification, remove a stale
`> Maintained:` line, and keep unresolved residuals explicit.

Use `apply_patch` for edits. Match punctuation exactly when replacing existing text. For large
section rewrites, anchor on unique headings and fail on ambiguous matches.

## Phase 4: Register, connect, and validate

1. If the module is new and the registry exists, add it to the matching section only after the map
   is verified and after showing the proposed registry diff for approval. This registration path is
   for an existing subsystem whose boundary was established by source verification; normal staged
   feature work remains `$loop-commit`'s registration path. Reject duplicate IDs and IDs whose
   existing descriptions conflict with the verified boundary. Keep the registry near its documented
   long-term size and describe the verified boundary.
2. Check whether the root `AGENTS.md` or `AGENTS.override.md` tells Codex to resolve affected module IDs,
   read the matching map and decision files before editing, and conditionally run a targeted map
   refresh when implementation changes invalidate mapped facts. If those rules are missing or only
   mention decisions, do not edit the instruction file. Tell the user that `$loop-commit` will
   propose the canonical Knowledge Loop Conventions upgrade when the map is staged for commit. If
   `$loop-commit` is unavailable, warn that the map will not be consumed automatically.
3. Confirm that the final map:
   - follows the reference template;
   - contains no source `path:line` citations;
   - separates facts, confirmed bugs, open questions, and external residuals;
   - pairs to the same module id under `.codex/decisions/`.
4. Run `git diff --check` and inspect the focused diff for the map and registry.
5. Do not commit or push unless the user explicitly asks. Mention confirmed risks that belong in
   a future Decision block.

Do not leave a newly registered ID without its approved verified map. If writing either the registry
or map fails, stop, report the partial state precisely, and do not stage or commit anything.

Use the failure rules in `references/report-contracts.md`. In all modes, trust current source over an
old map, never guess about missing dependency source, and do not map a historical subsystem or switch
to a successor without user confirmation.
