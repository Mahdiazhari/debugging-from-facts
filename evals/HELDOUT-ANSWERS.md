# Held-out cases: answer key

Do not show this file to the agents under test.

## heldout-memleak (thin / full)

**True root cause.** The "webhook retry tuning" commit changed `get_client(tenant_id)` to `get_client(tenant_id, timeout)`. The function keeps its `@lru_cache(maxsize=None)`. The caller passes `max(1.0, deadline - time.monotonic())`, a float that differs on almost every call. So each webhook delivery makes a new cache key, and the unbounded cache keeps a new `PartnerClient` with its own `requests.Session`, connection pool, and open socket. Memory and file descriptors grow by one client per webhook. Only `worker-default` (the queue that runs webhooks) grows.

**Fix.** Cache the client by `tenant_id` only. Pass the timeout per request (`session.get(..., timeout=...)`).

**Distractors.**
- The new pandas report export: large peak memory, a teammate's local repro shows 1.4 GB, and the lead's summary says it is the only change that "touches large data". In truth it runs only on `worker-reports`, which is not killed, and its memory goes back to baseline.
- The Celery 5.3.4 -> 5.3.6 bump: a dependency change in the release; both worker groups run it and only one grows.
- The release notes call the real change "webhook retry tuning", which sounds harmless.
- A rising memory graph that any leak would produce.

**Why the thin variant is undecidable.** It has no per-worker-group data, no routing info, no link between the slope and task types, no object counts or FD counts, and no diff of the webhook commit. Facts 7 and 8 weaken the report-export theory, but nothing points to a specific other cause. The Celery bump, the webhook change, and fragmentation after large pandas peaks all stay possible.

## heldout-authloop (thin / full)

**True root cause.** On Monday 19:12 the autoscaling group replaced api-3. The new VM booted from base image v42, a "minimal" image that dropped the time sync service. The api-3 clock runs about 95 s behind. The auth service (correct clock) sets `iat`/`nbf` to the issue time, so on api-3 a new token looks about 95 s in the future. With 30 s leeway, api-3 rejects every token younger than about 65 s with "not yet valid (iat)". Just after login or refresh, about 1 in 4 requests (those that land on api-3) gets 401. The refresh gives another fresh token, the retry can hit api-3 again, the front end clears the session and redirects to login, and the new login token is fresh again: a loop that ends when the token is older than about 65 s.

**Fix.** Enable time sync (chrony or systemd-timesyncd) in base image v42 and on api-3 (or replace or drain api-3). Add a clock-offset check or alert. A larger leeway is only a stopgap.

**Distractors.**
- The Tuesday front-end release (`SameSite=Lax` -> `Strict`, refresh logic moved into middleware). It looks guilty and the lead's summary dates the incident from it.
- The teammate's email-link repro: a real `SameSite=Strict` behaviour that sends a user to `/login`, but it is a different mechanism (no cookie on a cross-site navigation, no API 401).
- "No API deploys this week": true, but it omits the instance replacement, which is not a deploy.
- The overall 401 rate and the login page views: many causes produce them.

**Why the thin variant is undecidable.** It has no per-instance 401 breakdown, no 401 error reason, no time of the first ticket, no infrastructure event log, and no host clock data. Fact 8 (3 tickets on Monday) hints that the incident predates the release, but Monday tickets could be background noise. A front-end cause, a refresh race, an auth-service problem, and a host problem all fit the facts.

## heldout-dblatency (thin / full)

**True root cause.** The psycopg2 -> psycopg 3 upgrade turned on server-side prepared statements (psycopg 3 prepares a query after it runs 5 times on a connection; psycopg2 never does). With `plan_cache_mode = auto`, PostgreSQL then switches the prepared statement to a generic plan. Its row estimate for `customer_id = $1` is the average, which is small, so the generic plan uses `orders_customer_id_idx` plus a sort. That plan is fine for the median customer (40 orders) but reads and sorts about 1.6 M rows for customer 1 (and about 300 k for customers 2 and 3). Before the upgrade every execution got a custom plan, which for large customers is a backward scan on `orders_created_at_idx` that stops after 50 rows. Slowness appears only on connections that have run the query about 10 times, which is why p99 is normal for a few minutes after each deploy.

**Fix.** Add a composite index `orders (customer_id, created_at DESC)` (one plan is then good for all customers), and/or set `plan_cache_mode = force_custom_plan` for the app role, or set `prepare_threshold=None` in psycopg 3.

**Distractors.**
- The pool reduction 20 -> 10: plausible queueing, the teammate's claim. Per-step timing shows pool acquire p99 is 4 ms.
- The teammate's psql `EXPLAIN ANALYZE` with a literal value: fast, because a literal gets a custom plan. It matches "the query is fine" but not what the app runs.
- The migration (new column, `orders_status_idx`): a schema change on the same table in the same release.
- "Driver upgrade, no behaviour change" in the release notes and "nothing in 3.8 changes this endpoint's query" in the lead's summary: both hide the real change.
- DB CPU rise and "large accounts complain": many causes fit.

**Why the thin variant is undecidable.** It has no per-step timing, no plans of the slow production queries, no data distribution, no per-customer latency, no `pg_prepared_statements` output, and no connection-age pattern. Pool queueing, a plan change after the migration, and a driver-related plan change all fit the facts.
