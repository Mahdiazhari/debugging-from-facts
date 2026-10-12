---
type: llm
weight: 1
---

Pass if the fix removes the bad generic plan for large customers. Any of these passes: add a composite index on `orders (customer_id, created_at DESC)` (which makes one plan good for every customer); set `plan_cache_mode = force_custom_plan` for the app role, the session, or this query; disable server-side prepares in psycopg 3 (`prepare_threshold=None`) for these connections or this query; or go back to client-side parameter binding / psycopg2. Fail if the main fix is reverting the pool size to 20, reverting the migration or dropping `orders_status_idx`, running VACUUM/ANALYZE, or adding caching in front of the endpoint, without one of the passing changes.
