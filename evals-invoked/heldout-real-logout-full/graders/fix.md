---
type: llm
weight: 1
---

Pass if the fix makes only a 401 mean no session and lets 429/5xx/network errors surface as an error (error page, retry message) without redirecting to /login. Raising or exempting the rate limit for /api/auth/me may be mentioned as an extra, but fail if that or a token-lifetime / refresh change is the only fix.
