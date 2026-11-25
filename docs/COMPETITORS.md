# Competitors and Differentiation

Research date: 2026-11-24. Re-check before the public release — this space moves.

## Closest tool

**sql-tap** — https://github.com/mickamy/sql-tap (MIT)
- Transparent proxy for PostgreSQL, MySQL and TiDB using their wire protocols.
- Terminal UI and web UI, EXPLAIN, N+1 detection, slow-query detection.
- CI mode: collects events, prints a report, exits 1 on N+1 or slow queries.
- **Not found in its README (2026-11-24):** baseline files, comparing counts across commits/branches, grouping per test or per HTTP request (it groups by transaction).

## Other related tools

| Tool | What it does | Why querybudget is different |
|---|---|---|
| CodSpeed database instrument (https://docs.codspeed.io/instruments/databases) | Proxy collects queries/plans in CI, results processed by CodSpeed | Commercial hosted service |
| Prosopite (https://github.com/charkost/prosopite) | N+1 detection for Rails | Ruby/Rails only |
| QueryGuard (https://oniani1.github.io/queryguard) | N+1 detection for TypeScript at driver level | Node/TS only |
| queryspy (https://pypi.org/project/queryspy/) | N+1 and query budgets for SQLAlchemy | Python/SQLAlchemy only |
| QueryAudit (https://haroya01.github.io/query-audit) | SQL anti-pattern rules in JUnit tests | Java only |
| RegreSQL (https://github.com/boringSQL/regresql) | Regression tests for SQL files (results, plans) | Tests `.sql` files, not app traffic |
| pg_plan_guard (https://pgxn.org/dist/pg_plan_guard/) | Postgres extension detecting plan drift vs approved baselines | Plans only, in-database, no app traffic counts |
| squawk / eugene (https://github.com/kaaveland/eugene) | Migration safety linting/lock tracing | Different problem (migrations) |

Context: GitLab's design notes describe the same pain — authors manually providing plans and raw SQL
in merge requests (https://docs.gitlab.com/ee/architecture/blueprints/database/automated_query_analysis/).

## Our differentiation (specific, and honestly narrow)

1. **Baseline file in the repo** — query counts as a reviewable snapshot, updated like snapshot tests.
2. **Per-test scopes** — counts attributed to the test that caused them.
3. **PR-ready diff** — "OrdersTest::test_index 3 → 53 queries" with a Markdown report and exit code.
4. **Language-agnostic** — works at the wire level, like sql-tap, unlike per-ORM libraries.

Detection itself (N+1, proxying) is **not** new. If sql-tap adds baselines, our product edge
shrinks; the portfolio and learning value remains. Keep the focus on the CI workflow.
