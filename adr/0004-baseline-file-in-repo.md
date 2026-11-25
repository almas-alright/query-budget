# ADR 0004: Baseline is a committed, sorted JSON file

- Status: Accepted
- Date: 2026-11-24

## Context

Regressions need something to compare against. Options: a database, a hosted service, CI
artifacts, or a file in the repository.

## Decision

`querybudget.baseline.json`, committed to the repo, with `format_version`, sorted keys
(`BTreeMap`), normalized SQL only, and counts per scope. Updating it is an explicit
`querybudget record` + commit, visible in PR review — like snapshot tests.

## Consequences

Positive: no infrastructure; reviewers see query-count changes in the diff; works offline.
Trade-offs: merge conflicts on the baseline in busy repos; test renames show up as removed+new
scopes. The format is versioned from day one so it can evolve.
