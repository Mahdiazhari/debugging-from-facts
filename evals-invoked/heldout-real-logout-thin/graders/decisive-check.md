---
type: llm
weight: 1
---

Pass if the answer names at least one check that would separate 'rate-limit errors are treated as logged out' from 'token expiry / refresh failure': for example whether the /login redirects cluster in the 429 time windows, which status /api/auth/me returned for redirected users (429 vs 401), or how the session code handles non-401 errors from /me. Fail if it only proposes checks on token lifetimes or the refresh PR.
