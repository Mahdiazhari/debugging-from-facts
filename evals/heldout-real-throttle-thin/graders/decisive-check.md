---
type: llm
weight: 1
---

Pass if the answer names at least one check that would separate per-process throttle storage from the other suspects: for example the Django CACHES setting / which cache backend is in use, the number of gunicorn workers or processes, or a test that the throttle triggers at the 5th login attempt regardless of which worker answers. Fail if it only proposes checks on the get_throttles PR, IP addresses, or the 503 alert.
