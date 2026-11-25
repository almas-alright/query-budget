# Rust Learning Map

Learn each concept **when the project needs it**. Main references:
- The Rust Book: https://doc.rust-lang.org/book/
- Rust by Example: https://doc.rust-lang.org/rust-by-example/
- Tokio tutorial: https://tokio.rs/tokio/tutorial
- Rustlings exercises (optional practice): https://github.com/rust-lang/rustlings

Each row: concept → where it appears in querybudget → PHP/Node comparison → reading.

## Phase 0 — Setup and foundation

| Concept | In this project | Compared with PHP/Node | Read |
|---|---|---|---|
| Cargo, crates, `Cargo.toml` | Project setup, dependencies | Like Composer / npm + `composer.json` | Book ch. 1, 14 |
| Modules, `pub`, `mod` | `src/` layout | Like namespaces/`use`, but privacy is the default | Book ch. 7 |
| Structs, `impl` | `Config` | Like a class with only data + methods, no inheritance | Book ch. 5 |
| `Result`, `?` | Config loading | Instead of exceptions: errors are return values you must handle | Book ch. 9 |
| `thiserror` / `anyhow` | `error.rs`, `main.rs` | Typed exceptions inside, a simple "catch-all" at the edge | crate docs |
| Derive macros | `#[derive(Debug, Parser, Deserialize)]` | Like annotations/decorators generating code | Book ch. 5, 19 |

## Phase 1 — Passthrough proxy

| Concept | In this project | Compared with PHP/Node | Read |
|---|---|---|---|
| `async`/`.await`, futures | Every socket operation | Like JS Promises, but lazy: nothing runs until awaited/spawned | Tokio tutorial "Hello Tokio" |
| `tokio::spawn`, `'static`, `Send` | One task per connection | Like spawning a Node "worker" — it must own its data | Tokio "Spawning" |
| Ownership and moves | Moving sockets into tasks | PHP copies/refcounts silently; Rust makes ownership explicit | Book ch. 4 |
| `TcpListener`, `TcpStream`, `into_split` | `proxy/mod.rs`, `session.rs` | Like Node `net.createServer` | Tokio "I/O" |
| `tokio::process::Command` | `runner.rs` runs the test command | Like `proc_open` / `child_process.spawn` | tokio docs |
| `tokio::select!`, shutdown | Stop when child exits | Like `Promise.race` | Tokio "Select" |

## Phase 2 — Reading the protocol (🧑‍💻 core)

| Concept | In this project | Compared with PHP/Node | Read |
|---|---|---|---|
| Enums with data + `match` | `FrontendMessage`, `BackendMessage` | Like a tagged union; much stronger than PHP enums | Book ch. 6 |
| Slices `&[u8]`, `&str` | Parsing without copying | Like `substr` that doesn't copy | Book ch. 4.3 |
| `Bytes` / `BytesMut` | `wire/buffer.rs` framing | Like a Node `Buffer` you can split cheaply | `bytes` docs |
| Big-endian integers | `i32::from_be_bytes` | Like `unpack('N', …)` / `buf.readInt32BE()` | std docs |
| `Option` | "not enough bytes yet" | Instead of `null` — the compiler forces you to check | Book ch. 6 |
| Unit tests, `#[cfg(test)]` | Byte fixture tests | Like PHPUnit/Jest but in the same file | Book ch. 11 |
| `HashMap`, `VecDeque` | Statement map, pending FIFO | Like associative arrays / a queue | Book ch. 8 |
| Channels (`tokio::sync::mpsc`) | Events from sessions to collector | Like an in-process message queue | Tokio "Channels" |

## Phase 3 — Normalize, count, N+1 (🧑‍💻 core)

| Concept | In this project | Compared with PHP/Node | Read |
|---|---|---|---|
| Iterators, `chars()`, `peekable()` | Hand-written SQL tokenizer | Like looping over a string with lookahead | Book ch. 13 |
| `String` vs `&str` | Building normalized SQL | Owned vs borrowed text | Book ch. 8.2 |
| Hashing (`std::hash`) | Fingerprints | Like `crc32`/`md5` of a string | std docs |
| `BTreeMap` | Deterministic output | Like a sorted array by key | std docs |
| Traits (`Display`, `From`) | Pretty printing, error conversion | Like interfaces / `__toString` | Book ch. 10 |
| `Arc<Mutex<_>>` *or* message passing | Active scope shared with control endpoint | Shared state across async tasks | Tokio "Shared state" |

## Phase 4 — Baseline, reports, release

| Concept | In this project | Compared with PHP/Node | Read |
|---|---|---|---|
| `serde` derive, JSON | Baseline file | Like `json_encode` with typed classes | serde.rs |
| File I/O, paths | Read/write baseline | Like `file_put_contents` | std docs |
| Snapshot testing (`insta`) | Report tests | Like Jest snapshots | insta docs |
| Process exit codes | `std::process::ExitCode` | Like `exit(3)` | std docs |
| Integration tests (`tests/`) | Real Postgres through the proxy | Like feature tests | Book ch. 11.3 |
| Release builds, CI | GitHub Actions, binaries | Like building a Docker image for release | cargo docs |

## Self-check before calling v0.1 done

You should be able to explain, without notes:
1. Why sockets are split into two owned halves before spawning two tasks.
2. Why `wire` has two message enums.
3. What happens when a network chunk ends in the middle of a message.
4. Why the baseline uses `BTreeMap`.
5. What `?` does and when it converts an error type.
6. Why the proxy forwards bytes *before* decoding them.
