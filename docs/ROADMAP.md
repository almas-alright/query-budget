# Roadmap (short scope → v0.1)

Do not start a phase before the previous phase's exit criteria are met.
Time estimates assume part-time work (10–15 h/week) and are rough guesses.

| Phase | Name | Main result | Rough time |
|---|---|---|---|
| 0 | Setup and foundation | Cargo project, CLI skeleton, config, errors, CI | ~3–4 days |
| 1 | Passthrough proxy (M1) | Traffic flows through unchanged; test command runner | ~1 week |
| 2 | Reading the protocol (M2) | Every executed query becomes a `QueryEvent` | ~1–1.5 weeks |
| 3 | Normalize, count, N+1 (M3) | Per-scope counts + N+1 + terminal report | ~1 week |
| 4 | Baseline, CI, release (M4 = v0.1) | `record`/`check`, Markdown report, Action, Laravel demo, release | ~1 week |

**Total:** about 4–5 weeks part-time.

## Phase 0 — Setup and foundation
- **Goal:** a small, tested Rust skeleton.
- **Exit:** `cargo run -- --help` shows `record` and `check`; invalid config gives a clear error; CI green.
- **Rust focus:** Cargo, modules, structs, `Result`/`?`, derive macros.

## Phase 1 — Passthrough proxy (Milestone M1)
- **Goal:** the proxy is invisible.
- **Exit:** `psql` and `tokio-postgres` work through the proxy; the proxy runs a child command and exits with its code; clean shutdown.
- **Rust focus:** async/await, tasks, ownership moves, sockets, processes, `select!`.

## Phase 2 — Reading the protocol (Milestone M2)
- **Goal:** see every query.
- **Exit:** for a scripted session, the proxy emits exactly the expected `QueryEvent`s (simple, prepared, pipelined, errors); decoder unit tests cover partial messages; traffic is still unchanged.
- **Rust focus:** enums + match, slices, `Bytes`, `Option`, collections, channels, unit tests.

## Phase 3 — Normalize, count, N+1 (Milestone M3)
- **Goal:** useful numbers.
- **Exit:** per-scope counts and N+1 findings in a terminal report; scopes work via the control endpoint; same run twice → same report.
- **Rust focus:** iterators, strings, hashing, `BTreeMap`, traits, shared state.

## Phase 4 — Baseline, CI, release (Milestone M4 = v0.1)
- **Goal:** fails a PR on regression.
- **Exit:** `record`/`check` with correct exit codes; Markdown report; GitHub Action; Laravel demo with an N+1 branch; README with demo GIF; tagged v0.1 release with a Linux binary.
- **Rust focus:** serde, files, snapshot tests, integration tests, release builds.

## After v0.1 (not committed — choose based on feedback)

- Query plan (EXPLAIN) changes, e.g. "Index Scan → Seq Scan".
- Optional AI explanations (Claude, OpenAI-compatible, Ollama) — advisory only (ADR 0005).
- SQL comment tags for scopes (works with parallel tests).
- Splitting `wire` into its own crate.
- Fuzzing and published overhead benchmark.
- TLS support; MySQL.
