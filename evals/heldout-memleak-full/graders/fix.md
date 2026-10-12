---
type: llm
weight: 1
---

Pass if the proposed fix removes the per-call timeout from the cache key, for example: cache the client by `tenant_id` only and pass the timeout per request (`session.get(..., timeout=...)`), or bound the cache in a way that keys on tenant only, or revert the `get_client` signature change. Reverting the whole "webhook retry tuning" commit also passes. Fail if the main fix is chunking the pandas export, reverting Celery, raising the memory limit, or setting `worker_max_tasks_per_child` / restarting workers; those may appear only as clearly labelled temporary mitigations next to a correct fix.
