---
name: db-query
description: Run guarded read-only MySQL investigations through an explicit db-query profile.
disable-model-invocation: true
---

# DB query

Treat every invocation as access to one specific database environment. Keep
mutations outside this skill: for a write request, provide read-only verification
SQL and identify the separate approval path required for execution.

1. Verify the tooling. Resolve `db-query` on `PATH`. If it is missing, read the
   [installation instructions](https://github.com/Nza6920/db-cli#install),
   report the current installation command, then stop. Complete this step when
   `db-query` resolves without implicit installation.
2. Establish the target. Run `db-query profiles` and require exactly one profile
   named by the user. Ask the user to choose when it is missing or ambiguous.
   Complete this step when the selected profile exists and is explicit.
3. Draft one read-only statement for one concrete question. Use fully qualified
   `database.table` names. Bound a detail read with an outer literal `LIMIT` no
   greater than the selected profile's `max_rows` (default 1000). Read that
   profile's configuration to resolve the cap. For UNION or nested queries, the
   LIMIT must bound the whole result; grouped and window queries also need it.
   Use `SHOW`, `DESCRIBE`, or `EXPLAIN` for schema or plan evidence. For complex joins, subqueries, or broad scans, state the performance
   risk and use `EXPLAIN` when plan evidence is needed. If the cost cannot be
   bounded confidently, split the investigation into multiple bounded statements
   executed one at a time. Complete this step when the exact SQL satisfies
   `db-query` safety rules, has an explicit performance bound or split plan, and
   answers only the stated question.
4. Authorize the query pair: the selected profile plus the exact SQL. Show both
   before execution. For a production profile, proceed only after the user
   explicitly approves that unchanged pair in the current conversation. Apply
   the same gate to schema-discovery SQL. Complete this step when the pair is
   visible and either approved for production or confirmed as non-production.
5. Execute through stdin with
   `db-query query --profile <profile> --stdin`. Add
   `--confirm-profile <profile>` only for the approved production pair. Keep JSON
   output for analysis. Treat `CONNECTION_FAILED`, `QUERY_FAILED`,
   `QUERY_TIMEOUT`, and `RESULT_ENCODING_FAILED` as structured database evidence,
   and propose uniquely aliased columns after a duplicate-column error. Complete
   this step when the CLI returns a structured result or error while credential
   values remain in the environment.
6. Report the database evidence and resulting conclusion separately. Name the
   profile, distinguish returned rows from inference, and keep results in the
   conversation unless the user requested an artifact. Complete this step when
   every conclusion is traceable to returned rows and no credential or result is
   persisted unintentionally.

Treat every follow-up or split SQL statement as a new query pair and return to
step 3. Any change to the production profile or SQL requires new explicit
approval.

Resolve current syntax and options from `db-query --help` and
`db-query query --help`.
