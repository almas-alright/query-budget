# ADR 0002: Postgres only, plain TCP, local/CI only (v0.1)

- Status: Accepted
- Date: 2026-11-24

## Context

Supporting MySQL, TLS termination and production use would multiply the work and the risk. The
goal is a small, finished v0.1.

## Decision

v0.1 supports PostgreSQL's v3 protocol over plain TCP, bound to loopback by default, for
development and CI use only.

## Consequences

Positive: one protocol to learn and test; no certificate handling; small attack surface.
Trade-offs: users must set `sslmode=disable` (or equivalent) for tests; MySQL users are not served
yet. Revisit after v0.1 feedback.
