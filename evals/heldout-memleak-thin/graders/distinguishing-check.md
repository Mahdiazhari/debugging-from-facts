---
type: llm
weight: 1
---

Pass if the answer names at least one concrete next check that would separate a leak in per-task code that runs often (the true cause is in the webhook path) from the report export. Qualifying checks include: find out which workers/queues run the export and whether those same workers are the ones that grow; correlate memory slope with counts of each task type (webhooks vs. exports); take a production memory profile or object-count diff (tracemalloc, memray, `gc.get_objects()` type counts, heap snapshot) on a growing worker; count open file descriptors / sockets over time on a growing worker; read the diff of the "webhook retry tuning" and other 2.14 commits. Fail if the only next steps are re-running the export locally, adding memory, restarting workers, or generic "add monitoring" with no specific measurement.
