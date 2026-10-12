---
type: llm
weight: 1
---

Pass if the answer names at least one concrete next check that would separate pool queueing (the main distractor) from a slow plan inside the database (the truth: a generic plan from server-side prepared statements after the psycopg 3 upgrade). Qualifying checks include: split request latency into pool-acquire time and query-execution time; capture the actual plans of slow executions in production (`auto_explain`, `log_min_duration_statement`, `pg_stat_statements` max/mean times); run the query the way the app does (PREPARE/EXECUTE with a parameter, more than five times, or with `plan_cache_mode = force_generic_plan`) instead of with a literal; check `pg_prepared_statements` on an app connection; compare latency by customer size; check whether slowness depends on how long a connection has been in use. Fail if the only next steps are rerunning EXPLAIN ANALYZE with a literal value, reverting the pool and watching, or generic "check the database metrics".
