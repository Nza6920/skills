---
name: db-query
description: Run a guarded read-only MySQL query through a configured db-query profile.
disable-model-invocation: true
---

# DB Query

Treat every invocation as access to a specific database environment.

1. Verify that both `db-query` and `usql` are on `PATH`. If either is absent,
   report the missing dependency and installation command, then stop. Completion:
   both commands resolve without installing anything implicitly.
2. Resolve the target with `db-query profiles`. If the user did not name exactly
   one profile, ask them to choose. Completion: one explicit profile is selected.
3. Draft one read-only statement. Use fully qualified `database.table` names.
   Bound detail reads with an outer literal `LIMIT` of at most 1000; use `SHOW`,
   `DESCRIBE`, or `EXPLAIN` when schema or plan evidence is needed. Completion:
   the statement passes `db-query` safety rules and answers one concrete question.
4. Show the selected profile and exact SQL before execution. For a production
   profile, wait for explicit approval of that exact pair; any SQL change resets
   approval. Apply the same rule to schema-discovery queries. Completion: the
   production approval is current, or the profile is non-production.
5. Feed SQL through stdin to `db-query query --profile <profile> --stdin`. Add
   `--confirm-profile <profile>` only after the production approval in step 4.
   Keep JSON output for analysis. Completion: the CLI returns a structured result
   or a structured error without exposing credentials.
6. Report the evidence and conclusion. Name the profile and distinguish returned
   rows from inference. Keep query results in the conversation unless the user
   explicitly requests an artifact. Completion: no result or credential is
   persisted unintentionally.

Use a new approved query for each follow-up. When the requested operation would
write data, provide a read-only verification query and explain that this skill's
boundary ends before mutation.
