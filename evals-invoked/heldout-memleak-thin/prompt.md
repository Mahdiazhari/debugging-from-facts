---
max_turns: 10
allowed_tools: [Skill]
---

Use the debugging-from-facts skill if it is available to you.

Incident ticket: background worker memory growth after release 2.14

You are on call. Our Python background workers (Celery 5, Redis broker) run in containers with a 2 GiB memory limit. Release 2.14 went out on Tuesday at 14:00. Since then the orchestrator OOM-kills worker containers about every 5 to 7 hours. Tasks that were running at kill time are retried, so no data is lost yet, but some webhook deliveries to partners arrive hours late.

Facts collected so far:

1. Before 2.14, worker container memory stayed flat at 300-350 MB for weeks.
2. After 2.14, the container memory graph rises in a near-straight line from about 320 MB to 2 GiB. Then the container is killed and restarts at about 320 MB.
3. Task throughput and task error rate are the same as before the release. Queue depth is normal, except for short spikes after each kill.
4. Release notes for 2.14, written by the release manager: "New monthly usage report export (pandas). Celery 5.3.4 -> 5.3.6. Webhook retry tuning. Minor fixes."
5. The new task `export_monthly_usage` loads all usage rows for one tenant into a pandas DataFrame, pivots it, and writes a CSV to object storage. For the largest tenant this is about 1.8 million rows.
6. A teammate ran `export_monthly_usage` for the largest tenant on a laptop. Process memory went from 300 MB to 1.4 GB during the task. The teammate wrote in the channel: "Found it, the report export is the leak. Let's chunk the DataFrame and ship today."
7. In the same local run, process memory went back to about 330 MB a few seconds after the task finished.
8. The report task runs on the first day of each month and on demand from the admin page. The admin audit log shows 9 on-demand exports since Tuesday.
9. `worker_max_tasks_per_child` is not set. Worker processes live until the container stops.
10. The team lead's summary in the ticket: "Only the report export touches large data. Everything else in 2.14 is small config changes."
11. Nobody has profiled memory in production yet. The memory graph is per container; it has no breakdown by task.

Find the root cause and propose a fix. Answer in at most 350 words: your diagnosis, how sure you are and why, and the fix. You cannot run anything; reason from what is below. The team wants a fix PR today.
