---
type: llm
weight: 1
---

Pass if the fix configures a cache shared by all workers and replicas (for example Django DatabaseCache, Redis, or Memcached) for the throttle history. Fail if the fix changes the throttle rates, reverts the get_throttles PR, or changes the throttle key to work around the problem.
