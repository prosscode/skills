---
name: loop-distill
description: "Consolidate structured MODULE Decision blocks from Git commit bodies into stable snapshots under `.codex/decisions/` for one registered module or all modules at a release boundary. Use only when the user explicitly invokes `$loop-distill` or directly asks to distill, consolidate, or refresh module decision history; do not invoke implicitly."
---

# Loop Distill

Distill immutable Decision records from `git log` into one reviewable snapshot for a registered module. Treat Git history as the source of truth and `.codex/decisions/<id>.md` as a derived view.

Never amend, rebase, delete, or otherwise rewrite commits. Record corrections in new commits with `SUPERSEDES`.

## Identify the module

1. Run from the repository root.
2. Read `.codex/MODULES.md`.
   - Do not silently create the registry when it is missing.
   - For modules introduced by current staged work, stop and tell the user to use `$loop-commit`.
   - For an existing legacy subsystem whose boundary first needs verification, stop and tell the
     user to use `$loop-map` in legacy-bootstrap mode.
3. Choose the mode:
   - Use **single** when the user supplies one exact registered ID.
   - Use **all** when the user asks for all modules, a release distillation, or repository-wide
     consolidation. Enumerate exact IDs in registry order.
   - Use **next** for requests such as "the next due module": calculate pending Decision counts and
     let the user choose from the candidates.
4. Resolve targets:
   - Use an exact registered ID when the user supplied one.
   - If the ID is unknown, list the registered IDs and ask the user to choose.
   - If neither a target nor a mode was supplied, ask whether to process one module or all modules
     instead of guessing.
5. Map each ID to `.codex/decisions/<id>.md`, preserving `/` as a subdirectory separator. For example,
   map `native/jni` to `.codex/decisions/native/jni.md`.
6. Read every selected existing destination before mining history. Preserve its `D<n>` identifiers
   and manual annotations.

In **all** mode, first show a plan with each registered module, matching or pending Decision count,
destination, and expected action (`create`, `update`, or `skip`). Skip modules with no decisions or
no changes since their last valid distillation point.

## Mine Decision blocks

Collect commits whose bodies mention the target tag. Do not pre-filter by changed path because
`MODULE` is the authoritative index. Use commits reachable from `HEAD` by default so abandoned or
unmerged refs do not become current consensus.

```bash
git log HEAD --fixed-strings --grep="MODULE: <id>" \
  --pretty=format:'===%H===%n%aI%n%an%n--BODY--%n%B%n--END--%n'
```

For an existing snapshot whose `Last distilled` commit is reachable from `HEAD`, first scan
`<last-distilled-sha>..HEAD`. Fall back to a full `HEAD` scan when the marker is missing, unreachable,
the snapshot structure is inconsistent, or correct clustering and supersession cannot be established
incrementally. Use `--all` only when the user explicitly asks to include other refs, and distinguish
commits not reachable from `HEAD` from current consensus.

During an incremental scan, treat the existing snapshot as the current baseline: retain unchanged
entries and stable `D<n>` IDs, then apply new decision events, supersessions, and clusters to that
baseline. Never replace the snapshot with a view derived only from the incremental commits.

Use the explicit sentinels rather than `--pretty=fuller`, which indents bodies and can obscure
Markdown headings.

For each commit:

1. Split the record on the sentinels.
2. Find every `## Decision` block. A commit may contain multiple blocks for the same `MODULE`; parse
   each as an independent decision event and do not merge them merely because their module IDs match.
3. Parse these fields:

| Field | Requirement | Purpose |
| --- | --- | --- |
| `MODULE` | Required | Match exactly to the target ID |
| `WHY` | Required | Capture motivation |
| `ALTERNATIVES` | Required | Inform review; omit from the snapshot |
| `CHOSEN` | Required | Capture the selected approach |
| `TRADEOFFS` | Required | Capture accepted costs |
| `RISKS` | Required | Capture monitoring concerns |
| `SUPERSEDES` | Optional | Link to a replaced decision |

Skip well-formed blocks for other modules in the same commit. Retain malformed target blocks as warnings with their commit hashes; never discard them silently.

If no matching decisions remain, report that result and do not write that module. In **single** mode,
stop; in **all** mode, mark the module `skip` and continue with the remaining modules. Suggest:

```bash
git log --all --fixed-strings --grep="MODULE:" --pretty=format:'%h %s'
```

## Resolve supersession

Sort parsed decisions by author date ascending. Walk forward and resolve each `SUPERSEDES` value against:

1. A `D<n>` entry from the existing snapshot.
2. A prior decision's `CHOSEN` text using conservative substring or semantic matching.

Mark a matched predecessor as superseded. Keep only the latest member of a supersession chain active.

Do not guess when multiple targets match or no target matches. Preserve the decision as unresolved and add a warning for user review.

## Cluster decisions and preserve IDs

Group active decisions by topic using these signals in order:

1. Membership in the same supersession chain.
2. Strong overlap in `CHOSEN` and `WHY`.
3. Repeatedly modified files from `git show --name-only <sha>`.

When uncertain, keep decisions separate and report the possible merge as a warning. Deduplicate cherry-picked or repeated blocks by content while retaining every source hash.

Use the latest decision in a cluster for `What`, `Why`, `Tradeoffs`, and `Watch out`. List every contributing commit under `Source`.

Assign stable IDs:

- Reuse matching `D<n>` IDs from an existing snapshot.
- Treat IDs in both the constraint index and `Active` as existing decisions when resolving `SUPERSEDES` and reusing IDs.
- Never renumber an existing ID.
- For a new snapshot, order clusters by their earliest contributing commit and assign `D1`, `D2`, and so on.
- For new clusters in an existing snapshot, continue after the highest assigned ID.

## Draft the snapshot

Use this shape:

```markdown
# <Module display name> Decisions

> Snapshot of current consensus. Evolution: `git log --grep="MODULE: <id>"`
> Last distilled: <YYYY-MM-DD> (HEAD = <short-sha>)

## Active

### D1: <short paraphrased title>

- **What**: <current approach in one sentence>
- **Why**: <motivation in one sentence>
- **Tradeoffs**: <accepted costs in one sentence>
- **Watch out**: <risks in one sentence>
- **Source**: <abbrev-sha-1>, <abbrev-sha-2>

## Superseded

- ~~<old decision>~~ → replaced by **D1** in <abbrev-sha> (<YYYY-MM-DD>)
```

Apply these rules:

- Paraphrase instead of quoting commit bodies.
- Keep active fields to one concise sentence each.
- Omit `ALTERNATIVES`; preserve access to them through source hashes.
- List all contributing hashes.
- Put Active before Superseded.
- Omit the Superseded section when empty.
- Preserve non-conflicting manual annotations.
- For an existing destination, prepare a minimal diff and avoid rewriting unchanged entries.
- Flag conflicts between manual annotations and newly distilled content.

## Compress oversized snapshots

When a snapshot grows past roughly 30 full `Active` entries, or is otherwise too long to serve as a
practical pre-edit briefing, read
[references/snapshot-compression.md](references/snapshot-compression.md) completely and follow its
constraint-index, stable-ID, and recoverability rules.

## Review before writing

Show the full draft for a new file or the proposed diff for an update. Include:

- The exact destination and whether it will be created or updated.
- New active decisions.
- Decisions moving to Superseded.
- Decisions moving to the constraint index and the Git references that recover their full text.
- Malformed blocks and their hashes.
- Ambiguous or unresolved `SUPERSEDES` values.
- Uncertain clusters.

Ask for explicit approval to write the proposed snapshot. Apply requested edits, show the revised draft or diff, and ask again. If the user declines, discard the draft and leave the workspace unchanged.

In **all** mode, show one combined review organized by module and request one explicit approval for
the displayed batch. If a module contains malformed blocks, ambiguous supersession, or an unsafe
incremental boundary, skip writing that module by default while allowing clean modules in the
approved batch to proceed. Report every skipped module and reason.

## Write the approved snapshot

Write only the approved content to `.codex/decisions/<id>.md`, creating parent directories when needed. Do not stage, commit, or push.

After writing:

1. Show the destination path.
2. Summarize what changed and repeat any accepted warnings.
3. Tell the user to review the file and commit it when ready, optionally with `$loop-commit`.
4. If the root `AGENTS.md` or `AGENTS.override.md` does not direct Codex to read relevant module
   maps and decisions, mention that `$loop-commit` will propose the canonical knowledge-loop
   rules when the snapshot is staged for commit. Do not edit instruction files from this skill.

## Guardrails

- Never modify Git history or create corrective fixup commits.
- Never create or mutate module IDs or registries.
- Never use file paths as a substitute for exact `MODULE` tags.
- Never hide malformed input, ambiguous links, or clustering uncertainty.
- Never overwrite a snapshot without showing the proposed content and receiving approval.
- Never stage, commit, or push.
