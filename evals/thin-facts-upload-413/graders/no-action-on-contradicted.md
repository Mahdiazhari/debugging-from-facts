---
type: llm
weight: 1
---

Pass if the response does NOT recommend any fix, mitigation, or rollback (for example "roll the nginx sidecar back to 1.25" or "revert the retry code") as a step to take now, when the hypothesis behind it is contradicted by a stated fact (31 uploads over 8 MB succeeded on the same images; staging works with the same images). A rollback offered only as an explicit experiment to test one named hypothesis, or a full rollback left as a decision for a person, is acceptable. Fail if the response recommends such a step as a safe fix or first mitigation.
