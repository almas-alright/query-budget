# Expected Output (target UX, sample data)

These are design targets, not real output. Use them to agree on wording before implementation
and later as starting points for snapshot tests.

## Terminal — `check` with no changes

```text
querybudget check
  proxy 127.0.0.1:6433 → 127.0.0.1:5432
  running: php artisan test
  ...test output...

  12 scopes checked · 0 regressions · 0 new N+1
  ✔ no query regressions
```

## Terminal — `check` with a regression

```text
querybudget: 1 regression found

  OrdersTest::test_index        3 → 53 queries   (+50)
    N+1: executed 50× in one scope
      SELECT * FROM "customers" WHERE "id" = $1 LIMIT $2

  Info:
    + new scope   InvoicesTest::test_download (4 queries)
    ↓ decreased   UsersTest::test_list 7 → 5

  Exit code 3 (query regression)
  If this change is intended, run `querybudget record` and commit querybudget.baseline.json
```

## Markdown — PR comment (`--report-md`)

```markdown
### 🔴 querybudget: 1 query regression

| Test | Before | After | Change |
|---|---:|---:|---:|
| `OrdersTest::test_index` | 3 | 53 | **+50** |

**N+1 in `OrdersTest::test_index`** — executed 50×:
`SELECT * FROM "customers" WHERE "id" = $1 LIMIT $2`

<details><summary>Info (1 new scope, 1 decrease)</summary>

- New: `InvoicesTest::test_download` (4 queries)
- Decreased: `UsersTest::test_list` 7 → 5

</details>

Intended? Run `querybudget record` and commit the baseline.
```

## Baseline file (excerpt)

```json
{
  "format_version": 1,
  "tool_version": "0.1.0",
  "scopes": {
    "OrdersTest::test_index": {
      "total": 3,
      "queries": {
        "a1f3…": { "sql": "SELECT * FROM \"orders\" LIMIT $1", "count": 1 },
        "b72c…": { "sql": "SELECT * FROM \"order_items\" WHERE \"order_id\" IN (...)", "count": 1 },
        "c09e…": { "sql": "SELECT * FROM \"customers\" WHERE \"id\" IN (...)", "count": 1 }
      }
    }
  }
}
```

## README demo GIF (Phase 4)

1. Show the Laravel test passing with `check` (✔).
2. Switch to the N+1 branch.
3. Run `check` → red report, exit code 3.
4. Show the PR comment on GitHub.
