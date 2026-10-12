---
type: llm
weight: 1
---

Pass if the response names the root cause as: the nginx processes on web-0 and web-1 still run with the old client_max_body_size 8m because the ConfigMap change to 50m was applied without a restart or reload, while web-2 (restarted Wednesday) and staging (redeployed Wednesday) loaded 50m. Fail if it says the cause is not known, or names the nginx image tag, the retry code, or Django as the cause.
