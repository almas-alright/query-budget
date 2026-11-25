# ADR 0001: The proxy is observe-only

- Status: Accepted
- Date: 2026-11-24

## Context

querybudget sits in the path of every database query a test suite runs. If it changes, delays on
purpose, or breaks traffic, users will remove it immediately. Decoding the Postgres protocol fully
(COPY, notifications, every message type) is large and error-prone.

## Decision

Bytes are forwarded exactly as received, in both directions, before any decoding. The decoder works
on a copy, understands only the messages needed for counting, and treats everything else as
`Other`. A decoding problem disables observation for that connection, never forwarding.

Exceptions (handled before forwarding starts): SSLRequest and GSSENCRequest are answered with `N`
by the proxy itself so the session continues in plain TCP.

## Consequences

Positive: a bug degrades the report, not the user's tests; less protocol code to write.
Trade-offs: no TLS in v0.1; we cannot rewrite or annotate queries; correlation relies on the
in-order response rule.
