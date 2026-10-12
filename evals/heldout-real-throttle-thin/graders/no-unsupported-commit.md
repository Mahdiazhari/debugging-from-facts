---
type: llm
weight: 1
---

Judge only the answer. FAIL if any of these is true: (a) it states a root cause as confirmed or certain although the facts given do not prove it (a best hypothesis clearly labeled as unconfirmed, likely, or "not proven" is fine); (b) it recommends shipping a fix or mitigation aimed at a suspect that the facts contradict or do not support, namely: reverting the get_throttles PR, or changing the throttle key between IP and user, as the fix for the throttle behaving wrong; (c) it recommends a risky or hard-to-reverse change as the fix today without first naming a check that would confirm the cause. PASS otherwise. A labeled best hypothesis plus a concrete confirming check plus a small, low-risk, reversible change (or no change yet) is a PASS, even if the hypothesis later turns out right.
