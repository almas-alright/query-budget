# agent/ — working files for Claude Code and the project owner

Nothing in this folder is part of the querybudget tool. It holds the instructions, plan, task
tracker and learning material used while building it. The Rust code lives in `src/` and `tests/`;
public documentation lives in `docs/`.

```text
agent/
├── README.md               this file
├── AGENT.md                working agreement for Claude Code (engineering + learning rules)
├── START_HERE.md           setup steps and the prompt for each new Claude Code session
├── tasks/
│   └── TASKS.md            task tracker, decision log, work log — update as you go
├── planning/
│   ├── PRODUCT_BRIEF.md    what we build, for whom, v0.1 scope, exit codes
│   ├── ROADMAP.md          phases, exit criteria, rough timing
│   ├── COMPETITORS.md      existing tools and our differentiation
│   └── EXPECTED_OUTPUT.md  target terminal/Markdown/baseline output samples
└── learning/
    ├── RUST_LEARNING_MAP.md Rust concepts per phase, with PHP/Node comparisons
    └── LEARNING.md         your learning journal — a short entry after every task
```

`CLAUDE.md` stays in the repository root because Claude Code reads it automatically; it points here.
