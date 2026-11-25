# Contributing

querybudget is in early development (pre-v0.1). Issues and ideas are welcome; large pull requests
should start as an issue so we can agree on scope first.

## Development setup

1. Install Rust (stable) from https://rustup.rs
2. Start Postgres: `docker compose up -d`
3. Run checks:
   ```bash
   cargo fmt --check
   cargo clippy --all-targets --all-features -- -D warnings
   cargo test
   ```

## Rules

- The proxy is observe-only: never change forwarded bytes (see `docs/adr/0001-observe-only-proxy.md`).
- No `unsafe`, no `unwrap()` in production code.
- Add tests for every behavior change; keep output deterministic.
- Never put real user data in fixtures.
- Explain any new dependency in the PR description.
