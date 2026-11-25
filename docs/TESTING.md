# Testing and Quality

## Layers

1. **Unit tests** (in the same file, `#[cfg(test)]`)
   - `wire`: hand-built byte fixtures for every decoded message; partial messages (split at every byte position); unknown types; invalid lengths.
   - `correlate`: sequences of messages → expected `QueryEvent`s (simple, extended, pipelined, error before Sync, Close).
   - `normalize`: many real SQL samples from Laravel/Eloquent, Prisma, Knex, SQLAlchemy, Django output. Equal queries → equal fingerprint; different queries → different fingerprint.
   - `analyze`, `baseline`: comparison rules, tolerance, new/removed scopes.
2. **Snapshot tests** (`insta`) for terminal and Markdown reports.
3. **Integration tests** (`tests/`) against a real Postgres (`docker-compose.yml` locally, service container in CI):
   - `tokio-postgres` through the proxy: simple query, prepared statements, transactions, errors.
   - `psql` scripted session through the proxy.
   - End-to-end: `querybudget record -- <script>` then `check` → expected exit codes.
4. **Real-world smoke test** (Phase 4): the Laravel demo test suite passes through the proxy unchanged.

## Rules

- A test that needs Postgres is clearly named/marked; `cargo test` without Postgres should still run all unit tests.
- Byte fixtures live in `tests/fixtures/` or as constants in test modules, with a comment explaining each byte group.
- Never use real user data in fixtures.
- Same input → same output: run the end-to-end test twice and compare reports byte-for-byte.

## CI (GitHub Actions)

- `cargo fmt --check`
- `cargo clippy --all-targets --all-features -- -D warnings`
- `cargo test` (unit)
- integration tests with `postgres:17` service container
- Later (not v0.1): fuzzing the `wire` decoder with `cargo-fuzz`, property tests with `proptest`.

## Performance

No performance claims in v0.1. If the README mentions overhead, it must come from a reproducible
script (e.g. `pgbench` direct vs through the proxy) with the hardware and versions stated.
