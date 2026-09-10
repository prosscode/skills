# Map Module Report Contracts

Use these contracts in explorer tasks. Keep evidence in task reports; remove all `path:line`
citations from the final map.

## Contents

- Full research workflow
- Adversarial verification workflow
- Verdict application
- Failure handling
- Reader task
- Verifier task
- Final map template

## Full research workflow

Split by concern rather than file and name specific classes, methods, states, and flows:

- small subsystem (up to about 5 files): 3 research concerns;
- medium subsystem: 4 concerns;
- large subsystem or god-files: 5–6 concerns.

Dispatch one read-only explorer per concern in bounded waves. Respect the available concurrency
limit, keep one slot for the root agent, and reuse an explorer only for a related concern. Require
first-hand source inspection and repository-relative `path:line` evidence.

Collect every report. If one fails, cover the gap locally or mark the concern for extra independent
verification. Synthesize the baseline with the final map template, merging overlaps, distinguishing
public names from implementation names, and placing uncertainty only under `Open questions` or
`To verify`. Write or update `.codex/maps/<module-id>.md` with `apply_patch`.

## Adversarial verification workflow

Select checkable claims about state transitions, lifecycle edges, public contracts, entry points,
routing, error codes, defaults, feature gates, implementation differences, invariants, suspected
bugs, and every inferred or unverified assertion.

Use 8–10 claims for a small subsystem, 12–15 for a medium subsystem, and at least 15 for a large
subsystem. Weight failed-reader and state-machine concerns more heavily. Group claims into independent
verifier assignments and require fresh source reads plus one verdict per claim. A verifier must not
verify its own research concern; prefer a fresh explorer or cross-assign unrelated claims.

## Verdict application

- `confirmed`: promote important facts and remove matching `To verify` items.
- `refuted` or `partial`: correct the body with `correctedStatement`.
- `external_unverifiable`: retain a concise residual under `To verify`.
- `isBug: true`: add one impact-oriented item under `Confirmed bugs / technical debt`.

Keep genuinely open design or intent questions under `Open questions`. Do not claim full verification
when assignments fail; state the residual explicitly.

## Failure handling

- If all readers fail, stop before writing a map and report the research blocker.
- If one reader fails, recover through local inspection or extra verifier coverage and disclose any
  remaining gap.
- If dependency source cannot be found, record what is missing; do not guess or mark it confirmed.
- If the requested root exists only in history, identify the likely successor and ask whether to map
  the historical snapshot or current subsystem.
- If code and an existing map disagree, trust current source, update the map, and retain historical
  rationale only in the decision file.

## Reader task

Provide the explorer with:

- module id and repository-relative root;
- one concern key and a focused list of classes, methods, states, or flows;
- shared context and related maps;
- an explicit instruction to inspect source read-only and not edit files.

Require this report:

```markdown
## Concern
<key>

## Responsibility
<one concise paragraph>

## Key types
- `<type>` — <role>

## Entry points
- <entry point>

## Data flow and lifecycle
<ordered flow or state transitions>

## Dependencies
- In: <caller or upstream>
- Out: <callee, service, SDK, or external dependency>

## Invariants and gotchas
- <claim>

## Open questions
- <question whose intent cannot be answered from source>

## Unverified
- <claim that still lacks source>

## Evidence
- `<repo-relative-path>:<line>` — <what this proves>
```

Ask the explorer to ground every factual claim in evidence, distinguish direct facts from
inference, search only relevant slices of god-files, and return `none` for empty sections.

## Verifier task

Provide numbered claims with a verification hint. Tell the explorer to reopen source, default to
skepticism, inspect any external dependency source supplied in the shared context, and return one
item per claim:

```markdown
### <claim-id>
- verdict: confirmed | refuted | partial | external_unverifiable
- finding: <what source shows, with repo-relative path:line evidence>
- correctedStatement: <ready-to-paste sentence without line numbers>
- isBug: true | false
- confidence: high | medium | low
- missingSource: <required only for external_unverifiable>
```

Interpret verdicts strictly:

- `confirmed`: source proves the claim as written.
- `refuted`: a material part is wrong.
- `partial`: the core is right but scope, condition, or transition is incomplete.
- `external_unverifiable`: required source is genuinely unavailable from the project and supplied
  external context.

## Final map template

```markdown
# <module-id> Map
> Static understanding snapshot, not a decision history.
> See `.codex/decisions/<module-id>.md` for the paired decision history (note when it does not yet
> exist).
> Verified: YYYY-MM-DD (verification summary)

## Responsibilities

## Key types

## Public entry points

## Data flow / lifecycle

## Dependencies (inbound / outbound)

## Invariants and gotchas

## Confirmed bugs / technical debt

## Open questions

## To verify
```

Omit `Confirmed bugs / technical debt` when none are confirmed. Omit `To verify` when no external
or failed-check residual remains. Preserve the other headings so maps stay comparable.
