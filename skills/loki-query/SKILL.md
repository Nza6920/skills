---
name: loki-query
description: Investigate Grafana Loki logs and range metrics with the read-only repository CLI.
disable-model-invocation: true
---

# Loki query

Use the installed `loki-query` CLI for read-only log or range-metric investigation.

1. Establish the query boundary. Require exactly one profile named by the user.
   Run `loki-query config path`, read that TOML file, and use the selected
   profile's `default_selector` unless the user supplied another reliable
   selector. Use only the environment variable name in `token_env`; leave its
   credential value in the environment. If the profile or selector is missing,
   ask for it and stop. Complete this step when both are known without guessing.
2. Authorize one query. Choose `log` unless the user explicitly needs a metric
   aggregation. Build complete LogQL and choose the narrowest useful time
   window. Use the default log contract for log evidence and explicitly select
   the metric query type for range aggregations. Metric JSONL records contain a
   string value instead of a log line. Resolve current type-specific options,
   defaults, and record fields from the repository README and CLI help rather
   than copying syntax into this skill.
   Cap every query at 24 hours. For a window beyond one hour, proceed only with
   explicit approval in the current user request. Show the profile, exact time
   window, and LogQL in a commentary update before execution. Complete this step
   when the visible query boundary is within policy and authorized.
3. Execute the query. Treat an empty result as a successful query with no
   matches. Exit status `4` plus an incomplete-results warning means stdout may
   contain valid partial evidence; report it as incomplete, never as complete
   success. Complete this step when the CLI returns records, an empty result,
   partial evidence, or a handled error.
4. Follow the evidence. Trace identifiers and refine LogQL only when another
   query can answer the user's question. Before each follow-up, state its reason
   and return to step 2. Obtain new approval when switching profiles or proposing
   a window beyond one hour that the current request has not approved. Stop
   after five CLI queries in one user request, including the first. Complete
   this step when the evidence answers the question or the remaining gap cannot
   be closed within the authorized boundary and query cap.
5. Report matching log or metric evidence, resulting inferences, and unresolved
   checks as separate sections. Include only relevant production content and
   keep credentials out of the report. Complete this step when every conclusion
   is traceable to returned JSONL records.

Resolve current syntax and options from `loki-query --help` and
`loki-query query --help`.
