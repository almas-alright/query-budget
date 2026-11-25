# querybudget

**Snapshot tests for your database. Catch N+1 queries and query-count regressions in every pull request — for any language, with zero code changes.**

> **Project status:** Planning / starter pack
> **Implementation status:** Not started
> **Primary language:** Rust
> **Target release:** v0.1 (short scope — see "v0.1 boundary" below)
> **Name:** `querybudget` is a working name. Check GitHub and crates.io availability before the first public release.

## What it does, in one example

Your `/orders` page loads 50 orders. Today it runs **3 queries**. A teammate adds
`$order->customer->name` inside a loop and forgets eager loading. The code still works,
the tests still pass, nobody notices. Now the page runs **53 queries**.

querybudget runs your existing test suite through a small proxy that sits between your app
and Postgres. It counts what the database actually did, compares it with a baseline file
committed in your repo, and fails the PR:

```text
querybudget: 1 regression found

  OrdersTest::test_index        3 → 53 queries   (+50)
    N+1: executed 50× in one scope
      SELECT * FROM "customers" WHERE "id" = $1 LIMIT $2

  Exit code 3 (query regression)
```

It works the same for Laravel, Node, Python, Go — anything that talks to Postgres —
because it watches the database connection, not your code.

## How it works (short)

```text
your tests ──▶ querybudget proxy ──▶ Postgres
                     │ (forwards every byte unchanged, only observes)
                     ▼
         count queries per test → compare with baseline → report + exit code
```

Full walkthrough: [docs/HOW_IT_WORKS.md](docs/HOW_IT_WORKS.md).

## Proposed CLI experience

```bash
# 1. Record a baseline (run once, commit the file)
querybudget record --upstream 127.0.0.1:5432 -- php artisan test
#   → writes querybudget.baseline.json

# 2. In CI / before merging: check against the baseline
querybudget check --upstream 127.0.0.1:5432 --report-md report.md -- php artisan test
#   → exit 0 if nothing got worse, exit 3 on a query regression
```

Your tests connect to the proxy (default `127.0.0.1:6433`) instead of Postgres directly.

These commands describe the intended UX, not implemented commands.

## Product principles

- **Observe only.** The proxy forwards bytes unchanged. A bug in querybudget may produce a wrong report, but must never break your traffic.
- **Deterministic.** Same test run → same counts → same report. No AI decides pass/fail.
- **Zero code changes** for the basic mode. Per-test scopes need only a tiny optional helper.
- **Local and CI only.** Not for production traffic.
- **Small and boring dependencies.** One binary, no server, no database of its own.

## v0.1 boundary

**In scope**
- Transparent Postgres proxy (plain TCP, local/CI use).
- Reads simple-query and extended-query (prepared statement) traffic.
- Query normalization (literals removed) and fingerprinting.
- Counting per scope (whole run, or per test via a small control endpoint).
- N+1 detection (same query repeated in one scope).
- Baseline file (`querybudget.baseline.json`), `record` and `check` commands.
- Terminal report and Markdown report (for PR comments).
- GitHub Action + Laravel demo + README demo GIF.

**Out of scope for v0.1** (possible later)
- MySQL and other databases.
- TLS between your app and the proxy.
- Query plan (EXPLAIN) comparison.
- AI explanations.
- Web UI, hosted service, production use.
- Parallel test runs with per-test scopes.

## Quick links

- [Start here](START_HERE.md)
- [Product brief](docs/PRODUCT_BRIEF.md)
- [How it works (guided tour)](docs/HOW_IT_WORKS.md)
- [Architecture](docs/ARCHITECTURE.md)
- [Postgres protocol notes](docs/PROTOCOL_NOTES.md)
- [Configuration](docs/CONFIG.md)
- [Rust learning map](docs/RUST_LEARNING_MAP.md)
- [Testing](docs/TESTING.md)
- [Security](docs/SECURITY.md)
- [Roadmap](docs/ROADMAP.md)
- [Competitors and differentiation](docs/COMPETITORS.md)
- [Task tracker](tasks/TASKS.md)
- [Learning journal](LEARNING.md)
- [Claude Code instructions](CLAUDE.md) / [Agent agreement](AGENT.md)
- [Expected output samples](demos/EXPECTED_OUTPUT.md)

## Development status

See `tasks/TASKS.md`. Complete one phase and its acceptance criteria before moving to the next.
