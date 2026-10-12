---
type: llm
weight: 1
---

Pass if the answer says the throttle history is kept per process: no CACHES setting means Django uses the default local-memory cache (LocMemCache), each of the 4 gunicorn workers keeps its own counters, so the real limit is about 4x the configured one (about 80/hour for upload, about 16/min for login) and a restart clears the counters. Fail if it names the get_throttles PR, per-IP keys, or the 503s as the cause.
