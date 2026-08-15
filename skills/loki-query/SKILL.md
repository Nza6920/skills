---
name: loki-query
description: Investigate Grafana Loki logs with the read-only repository CLI.
disable-model-invocation: true
---

# Loki query

Use the installed `loki-query` CLI for read-only log investigation.

1. Establish the query boundary. Require exactly one profile named by the user.
   Run `loki-query config path`, read that TOML file, and use the selected
   profile's `default_selector` unless the user supplied another reliable
   selector. Use only the environment variable name in `token_env`; leave its
   credential value in the environment. If the profile or selector is missing,
   ask for it and stop. Complete this step when both are known without guessing.
2. Authorize one query. Build complete LogQL and choose the narrowest useful
   time window. Default to `--since 15m`, `--limit 100`, and `--output jsonl`.
   Cap every query at 24 hours. For a window beyond one hour, proceed only with
   explicit approval in the current user request. Show the profile, exact time
   window, and LogQL in a commentary update before execution. Complete this step
   when the visible query boundary is within policy and authorized.
3. Execute the query. Treat an empty result as a successful query with no
   matches. Complete this step when the CLI returns entries, an empty result, or
   a handled error.
4. Follow the evidence. Trace identifiers and refine LogQL only when another
   query can answer the user's question. Before each follow-up, state its reason
   and return to step 2. Obtain new approval when switching profiles or proposing
   a window beyond one hour that the current request has not approved. Stop
   after five CLI queries in one user request, including the first. Complete
   this step when the evidence answers the question or the remaining gap cannot
   be closed within the authorized boundary and query cap.
5. Report matching log evidence, resulting inferences, and unresolved checks as
   separate sections. Include only relevant production content and keep
   credentials out of the report. Complete this step when every conclusion is
   traceable to returned JSONL entries.

Resolve current syntax and options from `loki-query --help` and
`loki-query query --help`.
