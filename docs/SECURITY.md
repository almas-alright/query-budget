# Security and Privacy

querybudget is a **local development and CI tool**. It is not designed for production traffic.

## Defaults

- Proxy and control endpoint bind to `127.0.0.1` only. Binding elsewhere requires an explicit flag and prints a warning.
- No TLS in v0.1: traffic between app and proxy is plain TCP. Use only on localhost / inside a CI job.
- The proxy never needs the database password. Authentication bytes pass through untouched and are never logged.
- Bound parameter values (`Bind`) are never stored or logged.
- Literal values in SQL are removed by normalization **before** anything is stored, logged at info level, or written to the baseline/report.
- Raw SQL (with literals) may appear only in `trace`-level logs, which are off by default. Document this.
- The control endpoint has no authentication; it is loopback-only and only changes the active scope name.

## Untrusted input

- All bytes from both sides are untrusted. Length fields are validated; a maximum message size (configurable, e.g. 64 MiB) is enforced for buffering. Oversized/invalid messages → stop decoding that connection, keep forwarding.
- The decoder must never panic on any input (later: fuzzing).
- The baseline file is untrusted input when read (it comes from a PR): validate `format_version`, reject unknown versions with a clear error, limit file size.

## What goes into the repository

- `querybudget.baseline.json` contains scope names (test names), normalized SQL and counts. Normalized SQL reveals table/column names but not data. Mention this in the README.
- Never commit database dumps, `.env` files, or traffic captures.

## Checklist before v0.1 release

- [ ] No password, parameter value, or literal reaches info/debug logs (tested).
- [ ] Baseline contains no literal values (tested with sample queries containing emails/numbers).
- [ ] Default binds are loopback (tested).
- [ ] Decoder handles truncated/garbage input without panic (tested).
- [ ] Unknown baseline version is rejected (tested).
- [ ] README states "local/CI only, no TLS".
