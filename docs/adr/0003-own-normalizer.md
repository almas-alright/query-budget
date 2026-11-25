# ADR 0003: Own lightweight SQL normalizer instead of pg_query (v0.1)

- Status: Accepted
- Date: 2026-11-24

## Context

Queries must be grouped by shape (literals removed). `pg_query` (bindings to libpg_query, Postgres's
real parser) produces excellent fingerprints but needs a C toolchain to build and is reported to
work only on Linux and macOS. Most ORM queries already use `$1` placeholders, so the remaining work
is mostly literals in simple queries and `IN (…)` lists.

## Decision

v0.1 uses a small hand-written tokenizer (strings, numbers, identifiers, comments, placeholders)
to normalize SQL. It is a 🧑‍💻 learning task and is covered by many tests with real ORM output.

## Consequences

Positive: pure Rust, builds everywhere, teaches iterators and string handling.
Trade-offs: edge cases (dollar-quoted strings, unusual syntax) may normalize imperfectly. If tests
or users show fingerprint errors, revisit `pg_query` behind a Cargo feature.
