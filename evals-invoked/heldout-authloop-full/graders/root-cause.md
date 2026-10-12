---
type: llm
weight: 1
---

Pass if the answer names the root cause as the clock on api-3 running about 95 seconds behind (no time sync on the replacement VM built from the minimal base image v42), so api-3 rejects freshly issued tokens as "not yet valid" for roughly the first 65 seconds after issue (95 s skew minus 30 s leeway); requests and retries that land on api-3 get 401, the front end then clears the session and redirects to login, and the new token is again fresh, which makes the loop. Fail if the answer names the `SameSite=Strict` change, the middleware move, token expiry, or the auth service as the root cause, or only says "api-3 is bad" without naming the clock skew.
