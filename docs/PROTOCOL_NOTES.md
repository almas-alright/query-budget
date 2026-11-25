# Postgres Wire Protocol — the parts querybudget needs

Official reference (read alongside this file):
- Message flow: https://www.postgresql.org/docs/current/protocol-flow.html
- Message formats: https://www.postgresql.org/docs/current/protocol-message-formats.html

Always check details against the official docs; this file is a learning summary, not a spec.

## 1. Two phases

**Startup phase** — messages have **no type byte**:

```text
[length: i32 big-endian, includes itself][code or version: i32][payload…]
```

| Code | Message | querybudget action |
|---|---|---|
| `80877103` | SSLRequest | Reply single byte `N` ourselves; do not forward |
| `80877104` | GSSENCRequest | Reply `N` ourselves; do not forward |
| `80877102` | CancelRequest | Forward to upstream unchanged (new connection, then closes) |
| `196608` (3.0) | StartupMessage (user, database, …) | Forward unchanged; switch to normal phase |

Clients configured for "direct" TLS (newer libpq option) start with a TLS handshake instead of a
startup message. A first byte of `0x16` is a sign of that: close with a clear log message
explaining `sslmode=disable`.

**Normal phase** — every message:

```text
[type: 1 byte][length: i32 big-endian, includes itself but NOT the type byte][payload]
```

Authentication messages (`R` from server, `p` from client) are forwarded and ignored.

## 2. Same letter, different meaning

| Byte | Client → server (frontend) | Server → client (backend) |
|---|---|---|
| `Q` | Query (simple) | — |
| `P` | Parse | — |
| `B` | Bind | — |
| `E` | **Execute** | **ErrorResponse** |
| `C` | **Close** | **CommandComplete** |
| `D` | **Describe** | **DataRow** |
| `S` | **Sync** | **ParameterStatus** |
| `H` | Flush | CopyOutResponse |
| `X` | Terminate | — |
| `Z` | — | ReadyForQuery |
| `1` / `2` / `3` | — | ParseComplete / BindComplete / CloseComplete |
| `T` | — | RowDescription |
| `K` | — | BackendKeyData |

→ That is why `wire` has a `FrontendMessage` enum and a separate `BackendMessage` enum.

## 3. Messages we decode (v0.1)

Frontend:
- `Query`: one null-terminated string (may contain several statements — v0.1 counts it as one entry).
- `Parse`: statement name (string), query (string), param count + type OIDs (ignored).
- `Bind`: portal name, statement name, then formats/values (values ignored — never logged).
- `Execute`: portal name, max rows.
- `Close`: `S` (statement) or `P` (portal) + name → remove from map.
- `Sync`, `Terminate`.

Backend:
- `CommandComplete`: tag string, e.g. `SELECT 5`, `INSERT 0 1`, `UPDATE 3`.
- `ErrorResponse`: fields; we keep only the SQLSTATE code (`C` field).
- `ReadyForQuery`: transaction status `I` idle / `T` in transaction / `E` failed transaction.

Everything else → `Other(type_byte)`.

## 4. Simple vs extended query

**Simple:**
```text
client: Q "SELECT 1"
server: T (RowDescription) … D (DataRow)… C "SELECT 1"  Z 'I'
```

**Extended** (what most drivers/ORMs use with parameters):
```text
client: P ""  "SELECT * FROM users WHERE id = $1"
        B ""  ""  [42]
        D P ""
        E ""  0
        S
server: 1 (ParseComplete) 2 (BindComplete) T … D … C "SELECT 1"  Z 'I'
```

Drivers may **pipeline**: send several Parse/Bind/Execute groups before one Sync. Results still
come back **in order**, which is why a FIFO of pending executions works.

## 5. Correlation rules (v0.1)

- On `Parse`: `statements[name] = sql` (unnamed `""` is overwritten).
- On `Bind`: `portals[portal] = statement`.
- On `Execute`: look up portal → statement → sql; push `Pending { sql, kind: Extended, started_at }`.
- On `Query`: push `Pending { sql, kind: Simple, started_at }`.
- Backend `CommandComplete` → pop front, emit `Ok` event.
- Backend `ErrorResponse` → pop front, emit `Error` event. After an error in extended mode, the
  server skips messages until `Sync`; discard remaining pending executions sent before that `Sync`
  (track a "sync generation" number). Simple `Query` with several statements may produce several
  `CommandComplete`s — v0.1 emits one event per `Query` (first result) and ignores extras until
  `ReadyForQuery`.
- `PortalSuspended` (`s`, Execute stopped at max rows) and `EmptyQueryResponse` (`I`, empty query)
  also end a pending execution — pop the front.
- On `ReadyForQuery`: sanity-check the FIFO; if out of sync, log once and mark the connection unobserved.

These rules must be covered by unit tests with byte fixtures before they are used on real traffic.
