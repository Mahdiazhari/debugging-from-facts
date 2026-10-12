---
max_turns: 10
allowed_tools: [Skill]
---

Incident ticket: background worker memory growth after release 2.14

You are on call. Our Python background workers (Celery 5, Redis broker) run in containers with a 2 GiB memory limit. Release 2.14 went out on Tuesday at 14:00. Since then the orchestrator OOM-kills worker containers about every 5 to 7 hours. Tasks that were running at kill time are retried, so no data is lost yet, but some webhook deliveries to partners arrive hours late.

Facts collected so far:

1. Before 2.14, worker container memory stayed flat at 300-350 MB for weeks.
2. After 2.14, the memory graph of the killed containers rises in a near-straight line from about 320 MB to 2 GiB. Then the container is killed and restarts at about 320 MB.
3. Task throughput and task error rate are the same as before the release.
4. Release notes for 2.14, written by the release manager: "New monthly usage report export (pandas). Celery 5.3.4 -> 5.3.6. Webhook retry tuning. Minor fixes."
5. The new task `export_monthly_usage` loads all usage rows for one tenant into a pandas DataFrame and writes a CSV. A teammate ran it locally for the largest tenant: memory went from 300 MB to 1.4 GB, then back to about 330 MB after the task finished. The teammate wrote: "Found it, the report export is the leak. Let's chunk the DataFrame and ship today."
6. There are two worker groups, same image, both on Celery 5.3.6: `worker-default` (6 containers, queues `default` and `webhooks`) and `worker-reports` (1 container, queue `reports`). Each container runs one worker process (concurrency 1). The routing config sends `export_monthly_usage` only to `reports`.
7. Only `worker-default` containers are OOM-killed. `worker-reports` jumps to about 1.5 GB during each export and returns to about 400 MB within minutes. It has not been killed since the release.
8. On `worker-default`, the memory slope follows webhook volume: about 0.45 MB per delivered webhook. At night, when webhook volume drops, the slope drops by the same ratio.
9. An engineer counted live objects in one `worker-default` process with `gc.get_objects()`, one hour apart. `PartnerClient`: 14 -> 618. `requests.Session`: 15 -> 619. In the same hour that process delivered 604 webhooks.
10. In the same hour, `ls /proc/<pid>/fd | wc -l` went from 43 to 649. Almost all new descriptors are TCP sockets to the partner API host.
11. The "webhook retry tuning" commit, with message "respect task deadline in partner calls", contains this diff:

```
 # partners/clients.py
 @lru_cache(maxsize=None)
-def get_client(tenant_id):
-    return PartnerClient(tenant_id, timeout=10)
+def get_client(tenant_id, timeout):
+    return PartnerClient(tenant_id, timeout=timeout)

 # partners/tasks.py
-    client = get_client(t.tenant_id)
+    remaining = deadline - time.monotonic()
+    client = get_client(t.tenant_id, timeout=max(1.0, remaining))
```

12. `PartnerClient.__init__` creates a `requests.Session` and mounts an `HTTPAdapter` with a connection pool. There are 112 active tenants.
13. The team lead's summary in the ticket: "Only the report export touches large data. Everything else in 2.14 is small config changes."

Find the root cause and propose a fix. Answer in at most 350 words: your diagnosis, how sure you are and why, and the fix. You cannot run anything; reason from what is below. The team wants a fix PR today.
