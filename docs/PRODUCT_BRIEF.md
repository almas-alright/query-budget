# Product Brief

## One-sentence description

querybudget is a CI tool that runs your test suite through a transparent Postgres proxy and fails
the pull request when the number of database queries goes up or an N+1 pattern appears.

## Problem

N+1 queries and accidental extra queries are among the most common backend performance bugs.
They pass code review and tests because the code still works — it is only slower. They are
usually found in production, by users.

Existing fixes are mostly tied to one framework or language (Prosopite for Rails, QueryGuard for
Node, queryspy for SQLAlchemy, QueryAudit for JUnit). Teams with several languages, or frameworks
without such a library, have nothing that works for all of them. See `COMPETITORS.md`.

## Target users

- Backend engineers on Postgres who want query regressions caught in PRs.
- Teams with mixed stacks (e.g. Laravel + Node services) who want one tool.
- Tech leads who want "query budget" as a visible, reviewable number per test.

## Core user journey

1. Point the test database connection at the proxy port (one config change, e.g. `DB_PORT=6433`).
2. Run `querybudget record -- <test command>` once. Commit `querybudget.baseline.json`.
3. In CI, run `querybudget check -- <test command>`.
4. If a test now runs more queries (beyond tolerance) or a new N+1 appears, CI fails and posts a Markdown report.
5. If the increase is intended, the developer runs `record` again and commits the updated baseline — the change is visible in the PR diff, like a snapshot test.

## v0.1 features

### Proxy
- Listens on `127.0.0.1:6433` by default, forwards to a configured upstream Postgres.
- Forwards bytes unchanged (observe only).
- Answers TLS/GSS encryption requests with "not supported" so clients continue in plain TCP.
- Clear error when a client insists on TLS.

### Observation
- Reads simple query (`Query`) and extended query (`Parse`/`Bind`/`Execute`) messages.
- Matches each execution with its result (`CommandComplete` / `ErrorResponse`) to get status and duration.
- Ignores noise statements by default (`BEGIN`, `COMMIT`, `ROLLBACK`, `SAVEPOINT`, `RELEASE`, `SET`, `SHOW`), configurable.

### Analysis
- Normalizes SQL: removes literal values (strings, numbers), collapses `IN (…)` lists, normalizes whitespace.
- Fingerprint = hash of normalized SQL.
- Scopes: default one scope for the whole run; optional per-test scopes via a loopback control endpoint.
- N+1: the same fingerprint executed at least N times (default 5) within one scope.

### Baseline and check
- `record` writes `querybudget.baseline.json` (sorted, stable, diff-friendly; contains only normalized SQL).
- `check` compares the current run with the baseline:
  - **Regression (fails):** a scope's total query count increased by more than the tolerance (default `0`), or a new N+1 appeared.
  - **Info (does not fail):** new scopes, removed scopes, decreases.
- Terminal report, Markdown report (`--report-md`), exit codes.

### Distribution
- GitHub Action (composite) that downloads the release binary.
- Laravel demo app with a branch that introduces an N+1.

## Exit codes

| Code | Meaning |
|---|---|
| 0 | Test command succeeded, no query regression |
| 2 | Usage or configuration error |
| 3 | Query regression found |
| 4 | Internal proxy error (e.g. could not bind or reach upstream) |
| other | The test command failed; its exit code is returned and the report is still printed |

## Non-goals (v0.1)

- MySQL or other databases.
- TLS between app and proxy; production traffic.
- Query plan / EXPLAIN analysis.
- AI explanations.
- Web UI or hosted service.
- Per-test scopes with parallel test runners.
- Judging whether a query is "good" — v0.1 only reports *changes* against the baseline, plus N+1.

## Success signals (validation, not promises)

- The Laravel demo PR with an N+1 fails with a readable report.
- A real open-source test suite passes through the proxy unchanged.
- The same test run twice produces identical reports.
- A new user can follow the README quickstart in a few minutes.

## Risks to validate early

1. **Noisy counts** — caches, background jobs, migrations, transaction wrappers. Mitigation: ignore list, tolerance, scopes, "(unscoped)" bucket not checked.
2. **Protocol edge cases** — COPY, pipelining, LISTEN/NOTIFY, cancel requests. Mitigation: observe-only design, unknown messages forwarded untouched.
3. **Normalization mistakes** — two different queries getting the same fingerprint, or one query getting two. Mitigation: many unit tests with real ORM output.
4. **TLS expectations** — some drivers default to requiring TLS. Mitigation: document `sslmode=disable` / `prefer` per driver.
