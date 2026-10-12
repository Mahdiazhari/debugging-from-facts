---
type: llm
weight: 1
---

Pass if the answer rules out the teammate's pool-size claim with evidence from the ticket: pool acquire p99 is 4 ms while query execution p99 is 1.85 s; and/or explains why the teammate's psql EXPLAIN was fast (it used a literal value, so the planner made a custom plan, unlike the app's prepared statement with `$1`). Fail if the answer recommends reverting the pool size as the fix, or accepts the psql EXPLAIN as proof that the query is fine.
