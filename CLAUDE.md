# Claude Code Instructions

Read and follow [`agent/AGENT.md`](agent/AGENT.md) as the primary project working agreement.
**The learning rules in agent/AGENT.md are as important as the engineering rules.** The owner is
learning Rust through this project.

All paths in these documents are relative to the repository root.
Agent and planning files live in `agent/` (see `agent/README.md`). Public project docs live in
`docs/`. Rust source code lives in `src/`, `tests/` and `Cargo.toml` (created in Phase 0).
Keep this separation: never put Rust code or build files in `agent/`, and never put agent-only
notes in `docs/`.

Before implementation, also read:
- `README.md`
- `agent/planning/PRODUCT_BRIEF.md`
- `docs/ARCHITECTURE.md`
- `docs/HOW_IT_WORKS.md`
- `docs/PROTOCOL_NOTES.md` (from Phase 2)
- `agent/learning/RUST_LEARNING_MAP.md`
- `docs/CONFIG.md`
- `docs/SECURITY.md`
- `agent/tasks/TASKS.md`

Work only on the active phase unless the user explicitly changes scope.
For tasks marked 🧑‍💻 YOU, do not write the solution — guide, hint, review.
Update `agent/tasks/TASKS.md` as tasks progress. Do not claim unrun checks passed.
v0.1 has no AI features; do not add any.
