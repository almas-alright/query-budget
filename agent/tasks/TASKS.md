# querybudget Task Tracker (v0.1)

Update this file as work progresses.

**Status:** `[ ]` not started · `[/]` in progress · `[x]` complete · `[!]` blocked
**Mode:** 🧑‍💻 YOU = owner writes the code, Claude guides/reviews · 🤖 CLAUDE = Claude writes and explains
**Learn:** the Rust concept the task teaches (see `agent/learning/RUST_LEARNING_MAP.md`)

## Current status

- Current phase: Phase 0 — Setup and foundation
- Current milestone: M0
- Overall status: Planning done, implementation not started
- Last reviewed: 2026-11-24

---

## Phase 0 — Setup and foundation

- [ ] T0.1 🧑‍💻 Install Rust, run `cargo new`, build and run hello world — *Learn: Cargo basics*
- [ ] T0.2 🤖 Add `docker-compose.yml` with Postgres 17 for local development — *Learn: how tests will reach Postgres*
- [ ] T0.3 🤖 Create module layout from `docs/ARCHITECTURE.md` (empty modules + `lib.rs`) — *Learn: modules, `pub`*
- [ ] T0.4 🧑‍💻 CLI with clap: `record` and `check` subcommands, `--listen`, `--upstream`, trailing test command — *Learn: structs, derive macros*
- [ ] T0.5 🤖 Config: `querybudget.toml` + flags override, validation (ports, threshold ≥ 2) — *Learn: serde, `Result`*
- [ ] T0.6 🧑‍💻 Error types with `thiserror`; map to exit codes in `main.rs` (2 = usage/config, 4 = internal) — *Learn: enums, `?`, `From`*
- [ ] T0.7 🤖 Logging with `tracing` (`QB_LOG` env var for level) — *Learn: crates, macros*
- [ ] T0.8 🤖 CI workflow: fmt, clippy, test — *Learn: toolchain checks*
- [ ] T0.9 🧑‍💻 Unit tests for config validation — *Learn: `#[cfg(test)]`, assertions*

**Acceptance criteria**
- [ ] `cargo run -- --help` lists `record` and `check`
- [ ] Invalid config → clear message and exit code 2
- [ ] CI is green

---

## Phase 1 — Passthrough proxy (M1: "invisible proxy")

- [ ] T1.1 🤖 Listener on `--listen` (default `127.0.0.1:6433`); connect to upstream per client — *Learn: async, `TcpListener`*
- [ ] T1.2 🧑‍💻 Session: split both sockets, two tasks copying bytes each way (no decoding yet) — *Learn: ownership moves, `tokio::spawn`, `Send`/`'static`*
- [ ] T1.3 🤖 Handle startup: reply `N` to SSLRequest/GSSENCRequest, forward everything else; detect direct-TLS first byte `0x16` and log a clear error — *Learn: reading fixed-size headers*
- [ ] T1.4 🤖 Runner: spawn the trailing test command, wait, capture its exit code — *Learn: `tokio::process`*
- [ ] T1.5 🧑‍💻 Shutdown: when the child exits, stop accepting, drain sessions (short timeout), return — *Learn: `select!`, cancellation*
- [ ] T1.6 🤖 Integration test: `tokio-postgres` simple + prepared queries through the proxy against Docker Postgres — *Learn: integration tests*
- [ ] T1.7 🤖 Integration test: proxy exits with the child's exit code — *Learn: process testing*
- [ ] T1.8 🤖 Update `docs/HOW_IT_WORKS.md` steps 1–3 — *Learn: explain the flow*

**Acceptance criteria**
- [ ] `psql "host=127.0.0.1 port=6433 sslmode=disable …"` works
- [ ] Integration tests pass through the proxy
- [ ] Child exit code is returned; no hanging after the child exits

---

## Phase 2 — Reading the protocol (M2: "see every query")

Read `docs/PROTOCOL_NOTES.md` first.

- [ ] T2.1 🧑‍💻 `wire/buffer.rs`: incremental framing — feed bytes, get complete `(type, payload)` messages; handle splits at any byte — *Learn: `BytesMut`, `Option`, big-endian ints*
- [ ] T2.2 🧑‍💻 Tests for T2.1: split every fixture at every position; oversized length rejected — *Learn: test design*
- [ ] T2.3 🧑‍💻 `wire/frontend.rs`: `FrontendMessage` enum (Query, Parse, Bind, Execute, Close, Sync, Terminate, Other) + parser — *Learn: enums with data, `match`, slices*
- [ ] T2.4 🧑‍💻 `wire/backend.rs`: `BackendMessage` enum (CommandComplete, ErrorResponse(sqlstate), ReadyForQuery, PortalSuspended, EmptyQueryResponse, Other) + parser — *Learn: same, other direction*
- [ ] T2.5 🤖 Byte fixtures for all decoded messages with comments — *Learn: reading the protocol docs*
- [ ] T2.6 🧑‍💻 `proxy/correlate.rs`: statement/portal maps + pending FIFO + `QueryEvent` (rules in PROTOCOL_NOTES §5) — *Learn: `HashMap`, `VecDeque`, state machines*
- [ ] T2.7 🧑‍💻 Correlator unit tests: simple, extended, pipelined, error-before-Sync, Close, multi-statement Query — *Learn: testing state machines*
- [ ] T2.8 🤖 Wire decoder + correlator into sessions: forward first, then decode copy; on decode error → warn once, stop decoding that connection — *Learn: channels between tasks*
- [ ] T2.9 🤖 Bounded `mpsc` channel from sessions to a collector stub that prints events (debug) — *Learn: `mpsc`, backpressure*
- [ ] T2.10 🤖 Integration test: scripted session → exact expected event list — *Learn: end-to-end assertions*
- [ ] T2.11 🤖 Update `docs/HOW_IT_WORKS.md` steps 4–6

**Acceptance criteria**
- [ ] Every executed statement in the scripted session produces exactly one event with correct SQL and outcome
- [ ] Traffic is still byte-for-byte unchanged (Phase 1 tests still pass)
- [ ] Decoder never panics on truncated or garbage input (tests)

---

## Phase 3 — Normalize, count, N+1 (M3: "useful numbers")

- [ ] T3.1 🧑‍💻 `normalize.rs`: tokenizer that replaces string/number literals with `?`, keeps `$1`, collapses `IN (…)`, normalizes whitespace and keyword case; handles quoted identifiers and `--`/`/* */` comments — *Learn: iterators, `peekable`, `String` vs `&str`*
- [ ] T3.2 🧑‍💻 Normalizer tests with real ORM SQL (Eloquent, Prisma, Knex, SQLAlchemy samples) — *Learn: table-driven tests*
- [ ] T3.3 🤖 Fingerprint = stable hash of normalized text (document the hash choice; must be stable across runs and platforms) — *Learn: hashing*
- [ ] T3.4 🤖 Ignore list (default BEGIN/COMMIT/ROLLBACK/SAVEPOINT/RELEASE/SET/SHOW), configurable — *Learn: config + matching*
- [ ] T3.5 🧑‍💻 `collector.rs`: per-scope counts in `BTreeMap`s; `(run)` default and `(unscoped)` bucket — *Learn: `BTreeMap`, entry API*
- [ ] T3.6 🤖 `control.rs`: loopback `POST /scope` and `POST /scope/end` with axum; share active scope with collector — *Learn: shared state (`Arc<Mutex>`) or messages*
- [ ] T3.7 🧑‍💻 `analyze.rs`: N+1 findings (threshold default 5) — *Learn: iterators, filtering*
- [ ] T3.8 🤖 Terminal report (`Display`), snapshot-tested with insta — *Learn: traits, snapshot tests*
- [ ] T3.9 🤖 Determinism test: same scripted run twice → identical report — *Learn: why ordering matters*
- [ ] T3.10 🤖 Update `docs/HOW_IT_WORKS.md` steps 7–9

**Acceptance criteria**
- [ ] `WHERE id = 42` and `WHERE id = 43` → same fingerprint; different tables → different fingerprints
- [ ] Scopes work through the control endpoint
- [ ] N+1 is reported for a loop of 50 identical queries
- [ ] Report output is deterministic

---

## Phase 4 — Baseline, CI, release (M4 = v0.1)

- [ ] T4.1 🧑‍💻 `baseline.rs`: types + serde, `format_version: 1`, sorted output, size limit, reject unknown versions — *Learn: serde, file I/O*
- [ ] T4.2 🧑‍💻 Compare rules: count increase beyond tolerance or new N+1 → regression; new/removed scopes and decreases → info — *Learn: enums for findings*
- [ ] T4.3 🤖 `record` writes baseline; `check` compares; exit codes per PRODUCT_BRIEF — *Learn: `ExitCode`*
- [ ] T4.4 🤖 Markdown report (`--report-md`), snapshot-tested — *Learn: string building*
- [ ] T4.5 🤖 End-to-end tests: record → check (no change → 0) → modify script (N+1) → check → 3 — *Learn: full integration tests*
- [ ] T4.6 🤖 `examples/laravel-demo`: minimal Laravel app (orders/customers), tests, tiny scope helper trait, branch/flag that introduces an N+1 — *Learn: real-world validation*
- [ ] T4.7 🤖 `action.yml` composite GitHub Action + example workflow that comments the Markdown report on the PR — *Learn: CI distribution*
- [ ] T4.8 🤖 Release workflow: Linux x86_64 binary on tag (macOS optional) — *Learn: release builds*
- [ ] T4.9 🤖 README: quickstart, demo GIF of the failing PR, limitations, driver TLS notes (`sslmode=disable`) — *Learn: documentation*
- [ ] T4.10 🧑‍💻 Run the security checklist in `docs/SECURITY.md`; fix gaps
- [ ] T4.11 🧑‍💻 Self-check questions in `agent/learning/RUST_LEARNING_MAP.md` — answer them in `agent/learning/LEARNING.md`
- [ ] T4.12 🤖 Choose license (LICENSE-DECISION.md), tag v0.1.0

**Acceptance criteria**
- [ ] Laravel demo: clean branch passes, N+1 branch fails with a readable report
- [ ] README quickstart works on a clean machine
- [ ] v0.1.0 release with binary is published
- [ ] Security checklist passes
- [ ] You can explain the whole flow in `docs/HOW_IT_WORKS.md` without notes

---

## Decision log

| Date | Decision | Reason | Revisit when |
|---|---|---|---|
| 2026-11-24 | Short v0.1 scope (no plans, no AI) | Ship fast for portfolio; validate core workflow | After v0.1 feedback |
| 2026-11-24 | Mixed working mode (🧑‍💻 / 🤖) | Learn Rust on the core parts | If pace is too slow |
| 2026-11-24 | Observe-only proxy | Never break user traffic | Never |
| 2026-11-24 | Own normalizer instead of `pg_query` | No C dependency; good learning task | If fingerprint accuracy issues appear |
| 2026-11-24 | Postgres only, plain TCP | Smallest useful scope | After v0.1 |

## Work log

| Date | Task | Result | Next step |
|---|---|---|---|
| 2026-11-24 | Planning pack created | Scope, architecture, tasks drafted | Start T0.1 |
