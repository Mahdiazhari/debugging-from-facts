---
type: llm
weight: 1
---

Pass if the response treats "v1.8 changed three things" as a summary rather than a complete fact, and asks for the full v1.7 to v1.8 difference including configuration, manifests, ConfigMaps, ingress, or the live nginx config on each replica. Asking only for the upload code diff is not enough. Fail otherwise.
