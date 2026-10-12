---
max_turns: 10
allowed_tools: [Skill]
---

Incident ticket: order history endpoint slow for some requests

Setup: a Python API on 8 pods talks to PostgreSQL 15. The endpoint `GET /customers/{id}/orders?limit=50` runs one query:

```
SELECT id, status, total, created_at FROM orders
WHERE customer_id = %s ORDER BY created_at DESC LIMIT 50
```

The `orders` table has 4.2 million rows. Indexes: primary key, `orders_customer_id_idx (customer_id)`, `orders_created_at_idx (created_at)`.

Facts collected so far:

1. Since release 3.8 on Thursday, latency for this endpoint changed: p50 9 ms -> 9 ms, p95 60 ms -> 700 ms, p99 80 ms -> 1.9 s.
2. Average CPU on the database server rose from 35% to 52%.
3. Release notes for 3.8, written by the release manager: "Add `fulfilment_notes` column to orders and new index `orders_status_idx`. Reduce DB pool size per pod 20 -> 10 (we were near the server connection limit). Upgrade DB driver psycopg2 -> psycopg 3 (no behaviour change). Minor fixes."
4. The migration ran for 40 seconds during the deploy and finished without errors. Autovacuum and autoanalyze ran on `orders` after it.
5. Account managers say the complaints come mainly from a few large accounts.
6. A teammate ran this in psql for the largest account: `EXPLAIN ANALYZE SELECT id, status, total, created_at FROM orders WHERE customer_id = 1 ORDER BY created_at DESC LIMIT 50;`. Result: 0.9 ms, plan `Index Scan Backward using orders_created_at_idx`. The teammate wrote: "The query is fast. It's the pool cut to 10, requests queue for a connection. Revert the pool to 20."
7. Pod count (8) and request rate to this endpoint are the same as before the release.
8. The latency metric covers the whole request: waiting for a pool connection, running the query, and serializing the response. There is no per-step timing.
9. `log_min_duration_statement` and `auto_explain` are not enabled on the database.
10. The team lead's summary: "Nothing in 3.8 changes this endpoint's query."

Find the root cause and propose a fix. Answer in at most 350 words: your diagnosis, how sure you are and why, and the fix. You cannot run anything; reason from what is below. The team wants a fix PR today.
