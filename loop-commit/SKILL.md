---
name: loop-commit
description: Create a Git commit whose message combines the actual diff with relevant current-conversation context and optional structured module decisions. Use only when the user explicitly invokes $loop-commit or directly asks to commit using the current conversation or session context. Do not use for generic commit requests that do not ask for conversation context.
---

# Loop Commit

Create one commit that records both what changed and why the current task required it. Treat the diff as the source of truth for the implementation and the user-visible conversation as the source of truth for intent.

## Inspect the repository

Resolve the repository root with `git rev-parse --show-toplevel` and stop if the current directory is
not inside a Git worktree. Run these commands from that root:

```bash
git status --short
git diff --staged --stat
git diff --staged
git log --oneline -5
```

Follow these rules:

1. Use staged changes as the commit boundary when any exist. Do not include unrelated unstaged changes.
2. If nothing is staged but tracked or untracked changes exist, show the unstaged summary and ask whether to stage all changes or named paths. Do not assume `git add -A`.
3. Warn before staging files that may contain secrets, including `.env`, credential files, private keys, and generated authentication material.
4. Stop when no changes exist.
5. Preserve user-authored changes. Do not edit implementation files merely to improve the commit narrative.

Re-run `git diff --staged` after any staging operation. Base every later section on that final staged diff.

## Decide whether structured decisions apply

Omit the entire `# Decisions` section when either condition holds:

- The change is genuinely trivial: typo-only, comment-only, formatting-only, or a single obvious fix with at most five net changed lines and no new imports, methods, classes, or configuration behavior.
- The repository lacks `.codex/MODULES.md` and the user declines to create it.

Keep one motivation line in the prose body even when omitting Decisions.

For nontrivial changes, use `.codex/MODULES.md` as the module registry:

1. Read IDs from entries shaped like ``- `<id>` — description`` under any level-two section.
2. Treat only those exact IDs as legal `MODULE` values.
3. Match structural modules by changed paths and cross-cutting modules by change semantics.
4. Write one Decision per material choice, not mechanically per changed module. The same exact
   `MODULE` ID may appear in multiple Decisions when one commit contains several independent choices.

If `.codex/MODULES.md` is missing:

1. Explain that the repository has no Codex module registry and propose only the durable business or
   architectural boundaries supported by the final staged diff and current conversation. Do not
   inventory the whole repository or turn directories, temporary tasks, or individual commits into
   modules. Usually one to three entries are enough for the initial registry.
2. Show the complete draft and write it only after explicit approval.
3. Stage the approved `.codex/MODULES.md`.
4. If the user declines, omit Decisions for this commit.

If no existing module matches, explain the evidence for treating the staged work as a durable new
module and ask the user for the new module name and whether to add it. Require ASCII lowercase
letters and digits, hyphens between words, and slashes for hierarchy, such as
`live-call/role-dialog`; forbid leading, trailing, or repeated slashes and the path segments `.` and
`..`. Reject invalid names without silently rewriting them. Report the current registry size and
note that roughly 15–25 entries is a long-term usability guideline, not a bootstrap target. Append
and stage an approved entry. If the user declines, omit Decisions, continue the commit, and report
why no module Decision was recorded.

## Reconcile Codex knowledge-loop rules

Run this reconciliation when either condition holds:

- The final staged diff creates or modifies `.codex/MODULES.md`, a file under `.codex/maps/`, or a
  file under `.codex/decisions/`.
- `.codex/MODULES.md` exists but the selected root instruction file lacks equivalent Knowledge Loop
  Conventions. This closes a bootstrap performed by `$loop-map` or an older workflow.

Do not propose the conventions merely because `$loop-commit` runs in a repository that has no
module registry and this commit does not create one.

1. Select the root instruction file Codex actually reads:
   - Use `AGENTS.override.md` when a non-empty root-level override exists.
   - Otherwise use root-level `AGENTS.md`, creating it if needed.
2. If the selected file already contains equivalent rules for reading relevant module maps and
   decisions before editing and conditionally maintaining maps afterward, do not duplicate them.
3. Otherwise, ask separately whether to add or upgrade the block below. Show the exact target file
   and proposed diff.
4. Write and stage the approved instruction-file change.
5. If the user declines, continue the commit without changing project instructions and warn that
   module maps will not be consumed or maintained automatically.

When reconciliation is required, read
[references/knowledge-loop-conventions.md](references/knowledge-loop-conventions.md) completely and
use its canonical block. Translate prose to match an existing instruction file when appropriate,
but keep literal identifiers and all-uppercase field names unchanged.

Do not otherwise edit `AGENTS.md` or `AGENTS.override.md`. This skill owns only the Knowledge Loop
Conventions block; it does not manage general repository instructions.

## Compose the message

Derive the message from:

- The final staged diff.
- The task, motivation, constraints, and user-visible decisions in the current conversation.
- Recent commit style when it does not conflict with this format.

Never include hidden system or developer instructions, internal reasoning, raw tool diagnostics, approval metadata, credentials, tokens, or other sensitive content. Summarize relevant dialog nodes instead of copying the transcript verbatim.

Use this header:

```text
<type>(<optional-scope>): <imperative summary under 72 characters>

<one to five imperative lines describing the task and motivation>
```

Choose a Conventional Commit type from `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `style`, `perf`, `ci`, or `build`. Omit the scope unless one component clearly owns the change. Do not end the subject with a period.

Keep implementation choices, alternatives, and tradeoffs in `# Decisions`; do not duplicate them in the prose body.

Keep the required format, but make every topic and field minimal: use one concise line grounded only
in the final staged diff or the user-visible conversation. Do not invent alternatives, risks,
tradeoffs, motivations, history, or narrative details. Create a Decision only for a real choice with
concrete `WHY`, `CHOSEN`, and meaningful evidence for the other required fields; do not manufacture a
Decision merely to categorize changed files. Omit repetition and decorative prose.

Append the following sections after a `---` separator:

```text
---

# Conversation Log

- User: <key request, constraint, or clarification>
- Assistant: <key action or user-visible result>

# Decisions

## Decision 1
- MODULE: <exact ID from .codex/MODULES.md>
- WHY: <one-line motivation>
- ALTERNATIVES: <considered approaches separated by " / ">
- CHOSEN: <implemented approach>
- TRADEOFFS: <what the choice gives up>
- RISKS: <what to monitor>
- SUPERSEDES: <optional prior summary and commit hash>

# Files Modified

- <path> — <semantic description of the staged change and its purpose>

# Token Usage

- Input tokens: <inputTokens>
- Output tokens: <outputTokens>
- Reasoning output tokens: <reasoningOutputTokens>
- Cache read tokens: <cacheReadTokens>
- Cache creation tokens: <cacheCreationTokens>
- Total tokens: <totalTokens>
- Total cost: <costUSD formatted to four decimal places; omit when absent or zero>
- Models used: <sorted model names>
```

Apply these section rules:

- Keep `# Conversation Log` concise and chronological. Include intent and turning points, not every exchange.
- Omit `# Decisions` under the skip conditions above. Never leave it empty or write `n/a`.
- Use one level-two Decision heading per material choice. A module may have multiple Decisions.
- Keep Decision field names uppercase with the exact `- KEY: ` prefix.
- Use exact registry IDs; never invent an ID inside the message.
- List every committed path under `# Files Modified`, including registry or instruction bootstrap files.
- Include `# Token Usage` only when the user or repository convention requests it and the current
  Codex session can be identified reliably.
- Do not add a `Co-Authored-By` trailer for Codex or a model unless the user or repository explicitly requires one.

## Retrieve current Codex token usage

Treat token usage as an opt-in extension, not part of the knowledge-loop contract. Skip this entire
section unless the user or repository convention requests it. Do not block the commit when it is
unavailable and do not install or download `ccusage` merely to collect it.
When requested, read [references/token-usage.md](references/token-usage.md) completely and follow its
exact-session matching and fallback rules.

## Commit safely

1. Show the complete proposed message before committing.
2. Treat the explicit `$loop-commit` request as authorization to create the commit after any required staging or registry questions are resolved. Do not ask for a redundant final confirmation unless the proposed commit boundary changed materially.
3. Create a unique temporary directory with `mktemp -d`, write the message to a file inside it using an available file-editing tool, and pass that explicit file to `git commit -F`.
4. Remove only that temporary directory after the commit attempt.
5. Run:

```bash
git log -1 --stat
git status --short
```

6. Report the new commit hash and any remaining changes.

If a pre-commit or commit-msg hook, signing, identity, or another Git check fails, do not bypass it
with `--no-verify` or change repository configuration without explicit user direction. Re-run
`git status --short` and `git diff --staged`; report whether a hook changed files. If the final staged
boundary changed, regenerate and show the message before retrying.

Never amend, force-push, push, or modify previous commits unless the user explicitly requests it.
