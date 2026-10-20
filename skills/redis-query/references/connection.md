# Connection and onboarding

## Select a file

Accept environment names matching `^[a-z0-9][a-z0-9_-]*$` and select exactly
`$HOME/.redis-env.<environment>`. Never source every matching file, guess a
default environment, or fall back to the original single-file `~/.redis-env`.
If that older file exists, guide the user to copy its settings into a named
environment file locally and add the environment type; leave the original
untouched.

## Missing or incomplete configuration

Give the user this template, replacing `uat` in the filename with the selected
environment. The user creates and fills it in locally; never ask them to send
their password in the conversation. The subshell's noclobber setting protects
an existing file from replacement.

```bash
(
  umask 077
  set -C
  cat > "$HOME/.redis-env.uat" <<'EOF'
# Fill in real values locally before running any query.
export REDISCLI_AUTH='REPLACE_LOCALLY'
export REDIS_HOST='redis-uat.example.com'
export REDIS_PORT='6379'
export REDIS_ENV_TYPE='non-production'

# Optional; defaults shown.
export REDIS_DB='0'
export REDIS_TLS='0'
export REDIS_CLUSTER='0'

# Optional TLS files when REDIS_TLS=1.
# export REDIS_CACERT='/absolute/path/to/ca.pem'
# export REDIS_CERT='/absolute/path/to/client.crt'
# export REDIS_KEY='/absolute/path/to/client.key'
EOF
)
```

For a production environment, the user sets `REDIS_ENV_TYPE='production'`.
After creating or editing the file, they run:

```bash
chmod 600 "$HOME/.redis-env.uat"
"${EDITOR:-vi}" "$HOME/.redis-env.uat"
```

For an existing incomplete file, guide the user to edit only missing/invalid
fields; do not overwrite it or replace its password. Stop before connecting
while configuration is missing, still contains template values, or is invalid.
Resume only after the loader validates the selected file.

## Isolated loading

Use a non-interactive Bash process with tracing disabled. These files are
trusted, user-maintained shell input; source only the selected file. Do not
`cat` it, print the environment, enable tracing, use password-bearing URIs, or
pass the password through `-a`/`--pass`/`AUTH` arguments.

The preparation example below makes no Redis connection. Set `redis_env` to
the explicit user selection. It prints only target metadata. To execute after
authorization, append the reviewed `redis-cli "${redis_args[@]}" ...` call
inside the same Bash process. Across tool calls, assume shell state is lost:
reload and assert the non-secret target still equals the approved values before
appending the command. Always check file access metadata and guide the user to
set mode `600` when needed; never print the file to inspect it.

```bash
bash --noprofile --norc <<'BASH'
set +x
set -euo pipefail
redis_env='uat'
fail() { printf '%s\n' "$1" >&2; exit 1; }
[[ "$redis_env" =~ ^[a-z0-9][a-z0-9_-]*$ ]] || fail 'Invalid environment name.'
redis_env_file="$HOME/.redis-env.$redis_env"
[[ -f "$redis_env_file" && -r "$redis_env_file" ]] || fail 'Selected configuration is missing or unreadable.'

unset REDISCLI_AUTH REDIS_HOST REDIS_PORT REDIS_ENV_TYPE
unset REDIS_DB REDIS_TLS REDIS_CLUSTER REDIS_CACERT REDIS_CERT REDIS_KEY
source "$redis_env_file" >/dev/null 2>&1 || fail 'Could not load selected configuration.'
set +x
set -euo pipefail
[[ -n "${REDISCLI_AUTH:-}" && "$REDISCLI_AUTH" != 'REPLACE_LOCALLY' ]] || fail 'Set REDISCLI_AUTH locally.'
[[ -n "${REDIS_HOST:-}" && "$REDIS_HOST" != 'redis-uat.example.com' ]] || fail 'Set REDIS_HOST locally.'
[[ "${REDIS_PORT:-}" =~ ^[0-9]{1,5}$ ]] || fail 'Invalid REDIS_PORT.'
(( 10#$REDIS_PORT >= 1 && 10#$REDIS_PORT <= 65535 )) || fail 'Invalid REDIS_PORT.'
case "${REDIS_ENV_TYPE:-}" in
  production|non-production) ;;
  *) fail 'Set REDIS_ENV_TYPE to production or non-production.' ;;
esac
REDIS_DB="${REDIS_DB:-0}"
REDIS_TLS="${REDIS_TLS:-0}"
REDIS_CLUSTER="${REDIS_CLUSTER:-0}"
[[ "$REDIS_DB" =~ ^(0|[1-9][0-9]*)$ ]] || fail 'Invalid REDIS_DB.'
[[ "$REDIS_TLS" =~ ^[01]$ && "$REDIS_CLUSTER" =~ ^[01]$ ]] || fail 'TLS and Cluster flags must be 0 or 1.'
[[ "$REDIS_CLUSTER" == 0 || "$REDIS_DB" == 0 ]] || fail 'Redis Cluster requires DB 0.'
[[ -z "${REDIS_CERT:-}" && -z "${REDIS_KEY:-}" || -n "${REDIS_CERT:-}" && -n "${REDIS_KEY:-}" ]] || fail 'Supply both REDIS_CERT and REDIS_KEY.'
if [[ "$REDIS_TLS" == 0 ]]; then
  [[ -z "${REDIS_CACERT:-}${REDIS_CERT:-}${REDIS_KEY:-}" ]] || fail 'TLS file options require REDIS_TLS=1.'
fi

export REDISCLI_AUTH
redis_args=(-h "$REDIS_HOST" -p "$REDIS_PORT" -n "$REDIS_DB" --no-raw)
[[ "$REDIS_CLUSTER" == 0 ]] || redis_args+=(-c)
if [[ "$REDIS_TLS" == 1 ]]; then
  redis_args+=(--tls)
  if [[ -n "${REDIS_CACERT:-}" ]]; then
    [[ -r "$REDIS_CACERT" ]] || fail 'CA file is unreadable.'
    redis_args+=(--cacert "$REDIS_CACERT")
  fi
  if [[ -n "${REDIS_CERT:-}" ]]; then
    [[ -r "$REDIS_CERT" && -r "$REDIS_KEY" ]] || fail 'Client certificate or key file is unreadable.'
    redis_args+=(--cert "$REDIS_CERT" --key "$REDIS_KEY")
  fi
fi
printf 'Environment=%s Type=%s Host=%s Port=%s DB=%s TLS=%s Cluster=%s\n' \
  "$redis_env" "$REDIS_ENV_TYPE" "$REDIS_HOST" "$REDIS_PORT" \
  "$REDIS_DB" "$REDIS_TLS" "$REDIS_CLUSTER"
printf 'Connection command:'
printf ' %q' redis-cli "${redis_args[@]}"
printf '\n'
BASH
```

## Connection options

`REDISCLI_AUTH` is recognized by `redis-cli` as the complete password. For
example, `export REDISCLI_AUTH='xxxxxx@xxxxx'` preserves `@` as part of that
password. Keep the value unchanged in the environment, without splitting it or
URL-encoding it. Use password-only authentication and omit the username option.
The other connection variables above must be mapped to arguments. Quote
arguments individually and retain them in an array. Use `--no-raw` to
distinguish nil/error replies and escape control bytes.

TLS is opt-in through `REDIS_TLS=1`. Use a readable custom CA when required and
provide both client certificate and key for mutual TLS. Keep certificate
verification enabled. If the installed CLI lacks a requested TLS option, stop
and report the capability gap rather than downgrading the connection.

`REDIS_CLUSTER=1` adds `-c` to follow Cluster `MOVED`/`ASK` redirects. Include that
behavior in the visible target; it does not enumerate or scan all nodes. Cluster
supports only DB 0. A non-Cluster DB defaults to 0 and is always shown explicitly.

Use a process-level timeout available in the execution environment; do not
assume a connection timeout option from the latest manual exists in the locally
installed CLI. A timeout leaves the query unsuccessful or incomplete, not an
empty successful result.

Sources: [Redis CLI](https://redis.io/docs/latest/develop/tools/cli/),
[SELECT and Cluster DB restrictions](https://redis.io/docs/latest/commands/select/).
