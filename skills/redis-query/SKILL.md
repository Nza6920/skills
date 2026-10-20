---
name: redis-query
description: Run bounded read-only Redis queries with the official redis-cli and explicitly selected local environment credentials.
disable-model-invocation: true
---

# Redis query

Use the official `redis-cli` for read-only queries. Each invocation selects one
named environment from `~/.redis-env.<environment>`; for example, `uat` selects
`~/.redis-env.uat`. Use password-only authentication through `REDISCLI_AUTH`;
keep its entire value, including any `@` characters, unchanged.

1. Verify the tooling and select the environment. Resolve `redis-cli` on `PATH`
   and inspect its local version/help. If unavailable, link the
   [official CLI installation guidance](https://redis.io/docs/latest/develop/tools/cli/)
   and stop without installing it implicitly. Require the user to name one
   environment, even on follow-ups in a new invocation; never infer it from a
   hostname or reuse the last environment silently. Complete this step when
   both the installed CLI and the explicit environment name are known.
2. Load only the selected local configuration. Read
   [connection and onboarding](references/connection.md) for the file template,
   isolated loading procedure, and optional DB, TLS, and Cluster settings. If
   the file or required fields are missing, guide the user to create or repair
   it locally, then stop until validation succeeds. Never display file contents
   or credential values. If the user's stated environment type conflicts with
   the file, stop and resolve the mismatch. Complete this step when host, port,
   environment type, DB, and connection options are validated without
   falling back to inherited credentials or localhost.
3. Draft the smallest query that answers the question. Read
   [bounded queries](references/queries.md) and use its command set. Prefer an
   exact key and explicit fields or indexes. For discovery, require a concrete
   key prefix or other user-approved pattern. Share a budget of ten scan calls
   across `SCAN`, `HSCAN`, `SSCAN`, and `ZSCAN` per invocation, with `COUNT 100`;
   display at most 100 items and preview String values with at most 4 KiB by
   default. `COUNT` is a work hint, not a response limit. If a larger read is
   necessary, explain the proposed limits and obtain approval before changing
   them. Complete this step when every command and argument is read-only,
   scoped, and within the agreed limits.
4. Authorize the exact target and command. Before each Redis command, show the
   environment name/type, host, port, DB, TLS/Cluster settings and any
   certificate paths, plus the complete `redis-cli` command with literal key,
   pattern, range, and cursor arguments. Keep the password solely in
   `REDISCLI_AUTH`; omit it from commands, URLs, and output. For
   `REDIS_ENV_TYPE=production`, execute only after explicit approval of that
   unchanged target/command pair in the current conversation. For validated
   `non-production`, execute after showing the pair. This applies equally to
   `PING`, type/size/TTL probes, and each subsequent scan cursor. Complete this
   step when the pair is visible and authorized for its environment type.
5. Execute the approved command in the isolated shell using the selected
   configuration and quoted argument arrays. Preserve the approved target; if
   a fresh shell must reload the file, check all non-secret connection fields
   against the approved values before making a connection. A changed target or
   command returns to step 4. Apply a ten-second process deadline using the
   execution runtime or an available timeout utility; yielding a tool call is
   not a deadline. Inspect Redis error replies as well as process status. On
   authentication, ACL, connection, TLS, redirection, wrong-type, or timeout
   errors, report the error and stop that query without automatic retries,
   environment switching, or broader scans. Complete this step when a result or
   handled error is available and no credential value has entered the
   conversation or an artifact.
6. Report the selected environment and DB, relevant returned evidence, and the
   conclusion. State preview/truncation limits, scan calls used, remaining
   cursors and node coverage where applicable. An empty batch with a nonzero
   cursor is incomplete; a Cluster node scan does not establish cluster-wide
   absence. Distinguish missing keys, expired-between-commands observations,
   errors, and incomplete searches. Keep results in the conversation unless an
   artifact was requested. Complete this step when every conclusion is tied to
   returned data and its scope is explicit.

For each follow-up command in the same invocation, return to step 3 and retain
the scan budget. A new invocation requires a new explicit environment choice.
Keep writes, deletion, expiry changes, script/function execution, administrative
changes, blocking reads, subscriptions, and continuous monitoring outside this
skill. Offer a bounded read-only verification command for requests outside its
scope.

Resolve version-specific options from local `redis-cli --help` and the
[official CLI documentation](https://redis.io/docs/latest/develop/tools/cli/).
Do not treat the Redis `READONLY` command as a general write-prevention switch.
