# ADR 0005: No AI in v0.1; any future AI is optional and advisory

- Status: Accepted
- Date: 2026-11-24

## Context

LLM integration (Claude, OpenAI-compatible APIs, Ollama) could explain why counts changed or
suggest fixes. It is not needed to detect regressions and would add privacy and reliability concerns.

## Decision

v0.1 contains no AI features. A later `explain` command may be added behind a Cargo feature and an
explicit opt-in. It would receive only normalized SQL and computed findings, could never change
pass/fail or exit codes, and its output would be labeled as advice. Local models (Ollama) should be
a first-class option.

## Consequences

Positive: v0.1 stays small, deterministic and offline.
Trade-offs: no "smart" explanations at launch.
