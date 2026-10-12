---
type: llm
weight: 1
---

Pass if the answer says the session check treats every getMe error as no session: getMe throws on the 429 from /api/auth/me, the .catch(() => null) turns that into null, and the page redirects a still-signed-in user to /login. Fail if it names token expiry, the proxy refresh change, or the bulk upload itself as the root cause.
