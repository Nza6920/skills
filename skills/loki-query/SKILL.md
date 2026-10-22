---
name: loki-query
description: Investigate Grafana Loki logs and range metrics with the read-only repository CLI.
disable-model-invocation: true
---

# Loki query

Use the installed `loki-query` CLI for read-only log or range-metric investigation.
Resolve command syntax and type-specific options from CLI help and the
[CLI README](https://github.com/Nza6920/grafana-loki-query-cli#querying).

1. Establish the boundary and request ledger. Require exactly one profile named
   by the user. Run `loki-query config path`, read that TOML file, and use the
   profile's `default_selector` unless the user supplied another reliable
   selector. Keep the credential in the environment variable named by
   `token_env`. If the profile or selector is missing, ask for it and stop.
   Initialize once for this user request: `invocations=0`, `allowance=60`,
   `charged=0`, `remaining=allowance-charged`. A different request-wide allowance
   requires explicit user direction. Preserve this ledger across follow-ups,
   CLI processes, work directories, scans, and resumes in the same request.
   Complete when the selected profile, actual target, selector, and numeric
   remaining allowance are known without guessing.
2. Authorize and reserve one invocation. Choose the narrowest useful time
   window. Choose a log query unless the user
   explicitly needs a metric aggregation; declare metrics with
   `--query-type metric`. Use JSONL for evidence. Use `trace` for an explicitly
   scoped exact trace search. Select JSON body (default) or explicit labels;
   resolve custom literal fields, JSON Pointers and service naming from CLI help.
   A custom mapping replaces the default fields; match only complete case-sensitive
   strings from the selected source. Use `--scan` only when bounded automatic
   scanning is needed; every trace source shares the generic scan engine.
   For `resume`, inspect the saved report first and retain its
   profile, target, query/trace identity, and frozen absolute time range.
   Trace identity includes ID, source, selector and canonical field mapping;
   changed scope starts a fresh investigation. Display/service fields and budgets
   may change. Resume only unresolved intervals; offline `export` uses saved
   evidence and identity without networking or credentials.
   Cap every range at 24 hours; a range beyond one hour requires explicit
   approval in the current request. Obtain explicit authorization before
   changing profiles. Show the profile, actual target, exact time window, and
   LogQL or trace source/fields before execution. Stop when `invocations=5` or
   `remaining=0`. Otherwise increment `invocations` before starting, reserve
   `passed_cap=remaining`, and pass exactly `--max-http-attempts passed_cap`.
   Count ordinary log/metric queries, trace, scans, and resumes, including failed
   or interrupted commands; help, config inspection, and offline `export` do
   not count. Complete when the displayed boundary is authorized, the count is
   at most five, and the positive CLI cap equals the ledger's remaining amount.
3. Execute and settle before any follow-up. Capture this process's stderr
   separately from stdout. With an explicit cap, the CLI's final accounting
   line is exactly `HTTP attempt accounting: used=N cap=N` (decimal integers).
   Accept usage only if this invocation has exactly one accounting line, it is
   final, its cap equals `passed_cap`, and `0 <= used <= passed_cap`. Add `used`
   to `charged`; otherwise charge the entire reserved `passed_cap` and mark
   that charge conservative. Missing, duplicate, malformed, stale, or interrupted
   accounting never establishes zero usage. Recompute `remaining` on every
   outcome, including success, empty results, partial evidence, authentication
   or query errors, interruption, and output/report failures. Read applicable
   completeness reports only when they belong to the current query; deduct
   current invocation usage, never a resume's cumulative totals. Complete when
   the invocation is settled and its evidence, coverage, and error are recorded.
4. Follow evidence within the settled ledger. Empty successful, resolved
   retrieval means no matches under the selected conditions. Saturation,
   parse loss, unresolved intervals, or errors leave retrieval gaps; stdout
   may still hold valid partial evidence. Trace coverage and record budgets count
   candidates before exact matching, so zero matched rows can still have gaps.
   Persist only with an explicitly selected dedicated work directory. For saved
   traces, retain candidate evidence and report the frozen predicate; validate
   loaded identity before resume or export. Exit `4` does not erase that evidence,
   and ordinary saturation can retain exit `0`. Refine or resume only to answer
   an identified remaining question, state the reason, and return to step 2.
   Exhausted HTTP allowance stops networking; saved evidence remains available
   through offline `export`. Only explicit user direction increases the total
   request-wide `allowance`: raising 60 to 90 after charging 60 leaves 30,
   while the five-invocation cap still applies. Process restarts cannot renew
   either limit. Complete when the question is answered or a precise evidence,
   budget, or authorization gap prevents further work.
5. Report returned log/metric evidence, inferences, and unresolved checks
   separately. Include invocation count, total allowance, charged amount,
   remaining amount, and any conservative charges; distinguish observed HTTP
   consumption from reserved charges. Keep credentials and irrelevant production
   content out of the report. Complete when every conclusion traces to returned
   evidence, every executed invocation is settled, and every retrieval or
   resource gap is explicit. Trace timelines are ascending with original lines,
   labels and real duplicates preserved; equal timestamps do not establish
   causality. No matches means none under the selected conditions and observed
   coverage, not proof of event absence, ingestion completeness, a complete
   request lifecycle or a server snapshot.
