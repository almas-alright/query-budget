# Configuration (proposed)

File: `querybudget.toml` in the project root (optional). CLI flags override file values.
This is a proposed format; it becomes real in task T0.5.

```toml
# Where your tests connect (point DB_PORT here)
listen = "127.0.0.1:6433"

# The real Postgres
upstream = "127.0.0.1:5432"

# Scope control endpoint (per-test mode)
control = "127.0.0.1:6434"

# Baseline file location
baseline = "querybudget.baseline.json"

[check]
# Allowed increase in total queries per scope before it counts as a regression
tolerance = 0
# Check the "(unscoped)" bucket too (default false: migrations/setup are noisy)
check_unscoped = false

[nplusone]
# Same fingerprint at least this many times in one scope → N+1
threshold = 5

[ignore]
# Statements not counted (prefix match after normalization, case-insensitive)
statements = ["BEGIN", "COMMIT", "ROLLBACK", "SAVEPOINT", "RELEASE", "SET", "SHOW"]

[limits]
# Max size of one protocol message we buffer for decoding
max_message_bytes = 67108864
```

## Driver notes (fill in during Phase 4)

| Stack | How to point tests at the proxy | TLS setting |
|---|---|---|
| Laravel | `DB_PORT=6433` in `.env.testing` / `phpunit.xml` | `'sslmode' => 'disable'` in `config/database.php` |
| Node (`pg`) | `port: 6433` | `ssl: false` |
| Prisma | `DATABASE_URL=postgres://…@127.0.0.1:6433/db` | `?sslmode=disable` |
| Python (psycopg) | `port=6433` | `sslmode=disable` |

Verify each row against the driver's docs before publishing.
