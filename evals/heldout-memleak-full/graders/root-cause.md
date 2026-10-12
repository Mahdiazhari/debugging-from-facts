---
type: llm
weight: 1
---

Pass if the answer names the root cause as the `lru_cache` on `get_client` whose key now includes the `timeout` argument: the timeout is a float computed from the remaining deadline, so almost every call is a new cache key, and the unbounded cache keeps a new `PartnerClient` (with its `requests.Session`, connection pool, and open socket) for every webhook delivery. Fail if the answer names the report export / pandas, the Celery 5.3.4 -> 5.3.6 bump, or generic "memory fragmentation" as the cause, or if it only says "the webhook change" without saying that the cache key grows without bound because of the per-call timeout.
