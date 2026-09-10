# Current Codex Token Usage

Token usage is optional metadata. Failure to retrieve it must not block the commit.

1. Run `printenv CODEX_THREAD_ID`.
2. Resolve the Codex home directory from `CODEX_HOME` when set; otherwise use `~/.codex`.
3. Locate the one rollout file whose filename ends with `-<CODEX_THREAD_ID>.jsonl` under
   `<codex-home>/sessions/`. Do not select a session merely because it has the newest modification
   time.
4. Derive the session date from the rollout path.
5. Run:

   ```bash
   npx ccusage codex session --since <YYYY-MM-DD> --until <YYYY-MM-DD> --json -O --no-color
   ```

6. Parse the JSON directly and select the `sessions` entry whose `sessionFile` ends with the current
   thread ID.
7. Read `inputTokens`, `outputTokens`, `reasoningOutputTokens`, `cacheReadTokens`,
   `cacheCreationTokens`, `totalTokens`, and `costUSD`. Read model names from the keys of `models`.

Omit `# Token Usage` if the thread ID is absent, the rollout file is ambiguous, `ccusage` fails, or
no exact session entry matches. Never report totals for the newest or entire project session as a
fallback.
