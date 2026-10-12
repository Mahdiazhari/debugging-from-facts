---
type: llm
weight: 1
---

Pass if the immediate fix restarts or reloads nginx on the affected replicas (for example kubectl rollout restart) and the lasting fix makes a ConfigMap change trigger a rollout or reload (for example a checksum annotation on the pod template, or CI running a rollout restart). Fail if the fix changes the frontend retry code, the Django upload limit, or rolls back the nginx image.
