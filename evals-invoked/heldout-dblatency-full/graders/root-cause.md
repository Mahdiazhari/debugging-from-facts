---
type: llm
weight: 1
---

Pass if the answer names the root cause as server-side prepared statements introduced by the psycopg2 -> psycopg 3 upgrade (psycopg 3 prepares a query after it runs a few times on a connection), after which PostgreSQL switches the prepared statement to a generic plan (`$1`, `generic_plans 4213`) that uses `orders_customer_id_idx` plus a sort. That plan is fine for small customers but reads and sorts over a million rows for the few very large customers, who used to get a custom plan (backward scan on `orders_created_at_idx`). Fail if the answer names the pool size reduction, the `fulfilment_notes` migration or `orders_status_idx`, table bloat / stale statistics, or "large customers just have a lot of data" as the cause, or names the driver upgrade without connecting it to prepared statements and the generic plan.
