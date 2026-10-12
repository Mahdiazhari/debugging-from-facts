---
type: llm
weight: 1
---

Pass if the fix restores correct time on the API hosts: enable a time sync service (chrony, systemd-timesyncd, or similar) in base image v42 and on api-3, and/or correct the clock or take api-3 out of rotation / replace it with a host that syncs time. Adding a clock-offset alert or health check is a plus but is not required. Raising the JWT leeway is acceptable only as a temporary measure next to the time sync fix. Fail if the main fix is reverting `SameSite` to `Lax`, reverting the middleware change, changing token lifetime, or changing the front-end redirect logic, without fixing the clock.
