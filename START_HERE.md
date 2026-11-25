# Start Here — querybudget

## What this package is

A planning and project-control starter pack. It is **not** an implemented Rust application.
Market research (Phase "−1") is already done — see `docs/COMPETITORS.md`. You start at **Phase 0: setup**.

## Working mode: mixed (you + Claude Code)

- Tasks marked **🧑‍💻 YOU** — you write the code. Claude Code guides, reviews and explains, but does not write the solution for you.
- Tasks marked **🤖 CLAUDE** — Claude Code writes the code and explains every important line.
- After every task, Claude Code asks you 2–3 short questions to check you understood.
- You add a few lines to `LEARNING.md` after every task.

The 🧑‍💻 tasks are the core of the project (reading Postgres messages, normalizing and counting queries).
They are the parts interviewers will ask you about.

## Before the first session

1. Install Rust: https://rustup.rs (then `rustc --version` and `cargo --version`).
2. Install Docker (for a local Postgres).
3. Create an empty GitHub repository and put this pack in it.
4. Optional but recommended: read chapters 1–4 of *The Rust Programming Language* (https://doc.rust-lang.org/book/).

## First Claude Code session

Open this folder in Claude Code and paste:

```text
Read CLAUDE.md, AGENT.md, README.md, docs/PRODUCT_BRIEF.md,
docs/ARCHITECTURE.md, docs/HOW_IT_WORKS.md, docs/RUST_LEARNING_MAP.md,
docs/SECURITY.md and tasks/TASKS.md.

We are starting Phase 0. I am learning Rust, so follow the
learning rules in AGENT.md exactly.

First: state the active task, its acceptance criteria, whether it is
a 🧑‍💻 YOU task or a 🤖 CLAUDE task, and the Rust concepts it teaches.
Then propose a short plan and wait for my approval before changing files.
```

## Suggested work rhythm

- One task (or a small group of related tasks) per session.
- Claude Code states the task, criteria and concepts first.
- Review every diff before accepting it. Ask "why" about anything you don't understand.
- Run the tests yourself and look at the real output.
- Answer the 2–3 check questions.
- Write your `LEARNING.md` entry.
- Update `tasks/TASKS.md`, then commit.

## Rough time (estimate, not a promise)

- Part-time (10–15 h/week): about 4–5 weeks to v0.1.
- Full-time: about 2 weeks.

## Important

The name `querybudget` is provisional. The architecture is a proposal; change it with a decision
record (`adr/`) when real code teaches you something new.
