# Bounded read-only queries

Use this command set. Every command still passes the target/approval step in
`SKILL.md`; metadata probes and follow-up cursor commands are queries too. Keep
keys, fields, members, patterns, and indexes as separately quoted arguments.

| Question | Command arguments after the connection options | Default bound |
| --- | --- | --- |
| Connectivity | `PING` | One reply |
| Key presence/type | `EXISTS 'key'`, `TYPE 'key'` | One exact key per command |
| Expiry | `TTL 'key'`, `PTTL 'key'` | One exact key |
| String size | `STRLEN 'key'` | Size only |
| String preview | `GETRANGE 'key' 0 4095` | Up to 4 KiB of value bytes |
| Hash metadata | `HLEN 'key'`, `HEXISTS 'key' 'field'`, `HSTRLEN 'key' 'field'` | Exact key/field |
| Hash fields | `HGET 'key' 'field'`, `HMGET 'key' 'field1' 'field2'` | At most 100 explicit fields; check field sizes when unknown and ask before retrieving a value over 4 KiB |
| List size/item/page | `LLEN 'key'`, `LINDEX 'key' 0`, `LRANGE 'key' 0 99` | At most 100 indexes; both range ends are inclusive |
| Set metadata/member | `SCARD 'key'`, `SISMEMBER 'key' 'member'` | One exact key/member |
| Sorted Set metadata | `ZCARD 'key'`, `ZSCORE 'key' 'member'` | One exact key/member |
| Sorted Set page | `ZRANGE 'key' 0 99 WITHSCORES` | At most 100 members, counting each member/score pair as one item |
| Key discovery | `SCAN 0 MATCH 'prefix:*' COUNT 100` | Ten scan calls total; show at most 100 unique keys |
| Hash discovery | `HSCAN 'key' 0 MATCH 'field-prefix:*' COUNT 100` | Same shared scan budget; one field/value pair is one item |
| Set discovery | `SSCAN 'key' 0 MATCH 'member-prefix:*' COUNT 100` | Same shared scan budget |
| Sorted Set discovery | `ZSCAN 'key' 0 MATCH 'member-prefix:*' COUNT 100` | Same shared scan budget; one member/score pair is one item |

These are templates, not a batch to execute. Replace example keys and prefixes
with the user's actual target before presenting the complete command. Select
only the probes needed for the question; do not collect unrelated values.
For another data type, such as Streams, report the capability gap and offer
type/TTL evidence rather than inventing an unreviewed command path.

## Scan procedure

1. Select an exact node/endpoint and, for `SCAN`, the selected DB. Start at
   cursor `0` with the agreed pattern and `COUNT 100`. For collection scans,
   select one exact key. Complete this step when the scope and first command
   have passed authorization.
2. Keep the returned cursor as an opaque decimal string. Deduplicate keys or
   collection elements for display. Count each executed scan against the shared
   ten-call budget, including failures. Complete this step when the cursor,
   visible items, and remaining budget are recorded.
3. Stop at cursor `0`, 100 displayed unique items, the scan-call limit, or an
   error. Otherwise propose the next exact cursor command and return to the
   authorization step. An empty batch with a nonzero cursor does not end the
   scan. Complete this step when traversal has ended or the report records why
   it stopped and the next cursor.

`COUNT` only suggests work per call. `MATCH` filters after scanning; a narrow
prefix does not guarantee cheap traversal. One reply can exceed 100 items;
small encoded collections may return everything at once. Stop after that
reply, show only the first 100 unique items, and mark omitted output. If items
from a reply were omitted, the returned cursor resumes after that whole reply;
it cannot recover those omitted items. Do not claim a hard server response or
byte limit. Prefer exact fields/indexes when large values or compact collections
make discovery costly, and obtain approval before an expensive value read.

Scan results are observations across time, not an atomic snapshot. Concurrent
changes can affect what is seen. In Cluster mode, `SCAN` covers only the node
serving it; `-c` does not fan out across primaries. Broader coverage requires an
explicitly agreed list of endpoints and authorization of every target/command
pair. Preserve the same invocation-wide scan budget across those endpoints.

## Interpretation and boundaries

- `TTL`/`PTTL` value `-2` means the key did not exist at that command; `-1` means
  it existed without an expiry. An empty String/range or zero length alone does
  not establish key absence. Use the minimum necessary `TYPE`/`EXISTS` evidence.
- Redis can return an error reply even when process status alone is insufficient
  to classify failure. Treat `NOAUTH`, `WRONGPASS`, `NOPERM`, `WRONGTYPE`,
  `MOVED`, `ASK`, TLS errors, and timeouts as errors. Redact credential values
  from any error before displaying it.
- String preview offsets count bytes and may split UTF-8 characters or binary
  values. Label previews and omitted collection items explicitly. Preview at
  most 4 KiB per displayed collection value too; their count does not bound
  their byte size or server work. A prior size probe is an observation, not a
  guarantee about a later reply. Prefer a finite `GETRANGE` to `GET` even for
  Strings that were small at the time of a probe.
- Keep full traversals and unbounded collection reads outside the default
  workflow. `KEYS`, `MONITOR`, subscriptions, blocking reads, scripts/functions,
  writes and administrative changes are outside this skill. A read-looking
  command can still mutate state: examples include `GETEX`, `SORT ... STORE`,
  and `XREADGROUP`. Use the table rather than inferring safety from a name.

Sources: [SCAN family semantics](https://redis.io/docs/latest/commands/scan/),
[TTL](https://redis.io/docs/latest/commands/ttl/),
[GETRANGE](https://redis.io/docs/latest/commands/getrange/),
[GETEX](https://redis.io/docs/latest/commands/getex/),
[SORT](https://redis.io/docs/latest/commands/sort/),
[XREADGROUP](https://redis.io/docs/latest/commands/xreadgroup/),
[READONLY scope](https://redis.io/docs/latest/commands/readonly/).
