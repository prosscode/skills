# Snapshot Compression

Use this only when a decision snapshot has roughly 30 full-detail active entries or is otherwise too
long to serve as a practical pre-edit briefing.

- Insert `## Constraint index (compressed decisions)` before `## Active`. Move settled decisions
  there as one-line bullets shaped like
  `- **D<n>** — <surviving constraint, trap, or contract> (<source-sha>)`.
- Keep IDs unchanged. Indexed decisions remain current decisions and continue to participate in ID
  reuse and `SUPERSEDES` resolution; `Active` contains recent, in-flight, or nuanced entries that
  still need full detail.
- Do not compress an entry when an actionable qualification or manual annotation cannot be preserved
  safely in one sentence.
- Keep each compressed batch recoverable from Git. Record a batch-specific reference in the index
  preamble, such as `D1–D20: git show <full-sha>:.codex/decisions/<id>.md`, and retain earlier
  references on later rounds.
- Before proposing compression, verify that the referenced reachable commit contains the full text
  of every entry in that batch. If it does not, leave those entries uncompressed and warn the user.
- Do not create a sibling archive file. The pinned snapshot and per-entry source commits provide the
  detailed history without duplicating stale text in the working tree.

When re-distilling an already split snapshot, append new decisions to `Active` and move them to the
index only after they settle. Keep only the latest `Last distilled` line, and propose another
compression round if the full-detail section becomes oversized again.
