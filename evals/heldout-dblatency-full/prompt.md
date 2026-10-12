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

1. Since release 3.8 on Thursday: p50 9 ms -> 9 ms, p95 60 ms -> 700 ms, p99 80 ms -> 1.9 s. Database CPU rose from 35% to 52%.
2. Release notes for 3.8: "Add `fulfilment_notes` column to orders and new index `orders_status_idx`. Reduce DB pool size per pod 20 -> 10 (we were near the server connection limit). Upgrade DB driver psycopg2 -> psycopg 3 (no behaviour change). Minor fixes." The driver is created with default options.
3. A teammate ran `EXPLAIN ANALYZE ... WHERE customer_id = 1 ...` in psql for the largest account: 0.9 ms, plan `Index Scan Backward using orders_created_at_idx`, `Rows Removed by Filter: 84`. The teammate wrote: "The query is fast. It's the pool cut to 10, requests queue for a connection. Revert the pool to 20."
4. The team lead's summary: "Nothing in 3.8 changes this endpoint's query."
5. On Friday the team added per-step timing. For this endpoint: pool acquire p99 4 ms; query execution p99 1.85 s.
6. Data shape: customer 1 has 1.6 million orders, customers 2 and 3 have about 300,000 each. The median customer has 40 orders.
7. 99% of requests slower than 500 ms are for customers 1, 2, and 3. Requests for small customers stay under 20 ms. Before the release, p99 for customers 1-3 was 110 ms.
8. With `auto_explain` enabled for statements over 500 ms, a typical entry is:

```
duration: 1712.4 ms  plan:
Limit (actual rows=50)
  -> Sort (Sort Key: created_at DESC) Sort Method: top-N heapsort
     -> Index Scan using orders_customer_id_idx on orders (actual rows=1603112)
          Index Cond: (customer_id = $1)
```

9. On one app connection, `SELECT name, statement, generic_plans, custom_plans FROM pg_prepared_statements` returns one row: name `_pg3_0`, the orders query, `generic_plans 4213`, `custom_plans 5`.
10. On a fresh connection, the first calls for customer 1 take about 1 ms. After roughly ten executions of the query on that connection (for any customers), calls for customer 1 take 1.4-1.9 s. After each deploy, p99 stays normal for a few minutes and then climbs.
11. The migration ran for 40 seconds and finished without errors. Autovacuum and autoanalyze ran on `orders` after it.

Find the root cause and propose a fix. Answer in at most 350 words: your diagnosis, how sure you are and why, and the fix. You cannot run anything; reason from what is below. The team wants a fix PR today.
