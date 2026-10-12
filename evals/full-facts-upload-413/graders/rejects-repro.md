---
type: llm
weight: 1
---

Pass if the response rejects the teammate's local repro as a different failure because its 413 body is Django JSON while production returns an nginx HTML body (or an equivalent signal mismatch). Fail if it treats the repro as evidence for the cause.
