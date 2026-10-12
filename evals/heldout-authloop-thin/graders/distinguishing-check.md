---
type: llm
weight: 1
---

Pass if the answer names at least one concrete next check that would separate a server-side, host-specific cause (the truth is clock skew on one API instance) from the front-end `SameSite`/middleware change. Qualifying checks include: break down the 401 rate by API instance; read the error reason in the 401 response bodies or API logs (missing token vs. expired vs. not yet valid vs. bad signature); check whether the rejected requests carried a session cookie/token; find the exact time of the first tickets or the first 401 rise and compare it with the Tuesday 10:05 deploy and with any infrastructure events; check clock sync / time offset on the API hosts. Fail if the only next steps are repeating the email-link repro, reverting the release and waiting, or generic "check the logs" without saying what to look for.
