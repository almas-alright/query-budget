# How It Works — a guided tour

This document follows **one query** from your test to the final report. Each step names the file
that handles it. Claude Code updates this file at the end of each phase so it always matches the
real code.

**Status legend:** ⬜ planned · 🟨 in progress · ✅ implemented (with real file paths)

---

## Step 1 — Your test sends a query ⬜

Laravel runs `Order::with('items')->get()`. PDO opens a TCP connection to `127.0.0.1:6433`
(the proxy) instead of `5432` (Postgres), because `DB_PORT=6433` in the test environment.

Nothing in the app knows the proxy exists.

## Step 2 — The proxy accepts the connection ⬜ `src/proxy/mod.rs`

The listener accepts the socket, connects to the real Postgres, and starts a **session** with two
tasks: one copies client → server, the other server → client.

*Rust idea:* each task is a `tokio::spawn`ed future. It must own everything it uses (`'static`)
and be safe to move between threads (`Send`). That is why sockets are split into owned halves.

## Step 3 — Startup and "no TLS, please" ⬜ `src/wire/startup.rs`

The first message has no type byte. If it is an `SSLRequest`, the proxy itself answers `N`
("no TLS"), and the client continues in plain TCP. The real `StartupMessage` (user, database) is
forwarded unchanged. Authentication (password/SCRAM) passes straight through — the proxy never
sees or needs the password.

## Step 4 — Bytes are forwarded, then a copy is decoded ⬜ `src/proxy/session.rs`, `src/wire/buffer.rs`

The session reads a chunk of bytes, **writes it to the other side first**, then gives the same
bytes to the decoder. Network chunks don't line up with messages, so the decoder keeps a buffer
and only returns complete messages: `[type: 1 byte][length: 4 bytes][payload]`.

*Rust idea:* `BytesMut` grows as data arrives; `split_to(n)` hands out a complete message without
copying.

## Step 5 — Messages become typed values ⬜ `src/wire/frontend.rs`, `src/wire/backend.rs`

`Parse { name, sql }`, `Bind { portal, statement }`, `Execute { portal }`, `Query { sql }` from
the client; `CommandComplete { tag }`, `ErrorResponse { code }`, `ReadyForQuery { status }` from
the server. Everything else becomes `Other(type_byte)` and is ignored.

*Rust idea:* the same letter means different things per direction (`'E'` = Execute from client,
ErrorResponse from server), so there are **two enums**, and `match` forces us to handle every case.

## Step 6 — Matching a query with its result ⬜ `src/proxy/correlate.rs`

Laravel's query arrives as `Parse` (SQL with `$1`), `Bind` (values), `Execute`. The correlator
remembers which SQL belongs to which prepared statement, and when it sees `Execute` it pushes an
"expected result" into a FIFO queue. When the server later sends `CommandComplete`, the backend
task pops the oldest entry → we know which SQL finished, whether it succeeded, and how long it took.
This becomes a `QueryEvent`, sent through a bounded channel to the collector.

## Step 7 — Normalizing the SQL ⬜ `src/normalize.rs`

`SELECT * FROM users WHERE id = 42` and `... id = 43` must count as the *same* query.
The normalizer replaces literals with placeholders, collapses `IN (1,2,3)` to `IN (...)`, and
normalizes whitespace. A hash of the result is the **fingerprint**.

## Step 8 — Counting inside a scope ⬜ `src/collector.rs`, `src/control.rs`

The collector adds the event to the active scope (e.g. `OrdersTest::test_index`, set by the test
calling `POST /scope`), or to `(run)` when scopes are not used.

## Step 9 — Finding N+1 ⬜ `src/analyze.rs`

If one fingerprint ran 50 times in one scope (threshold default 5), that's an N+1 finding.

## Step 10 — Comparing with the baseline ⬜ `src/baseline.rs`

`record` saves the counts to `querybudget.baseline.json`. `check` loads it and compares scope by
scope: more queries than allowed, or a new N+1 → regression.

## Step 11 — Report and exit code ⬜ `src/report/`, `src/main.rs`

A terminal report (and optional Markdown for the PR comment) is printed. Exit code `3` fails the
CI job.

---

## Where to start reading the code

(Claude Code fills this list in as files are created — the 5 files to read first, in order.)
