---
max_turns: 10
allowed_tools: [Skill]
---

Use the debugging-from-facts skill if it is available to you.

Incident ticket: users logged out and caught in login redirect loops

Setup: the front end is a Next.js app. The back end is a separate JSON API with 4 instances (api-1 to api-4) behind a load balancer. An auth service on its own hosts issues JWT access tokens (15-minute lifetime, `iat` and `nbf` set to the issue time) and refresh tokens. The API validates tokens with 30 seconds of clock leeway. The front end keeps the tokens in an HTTP-only session cookie. When an API call returns 401, the front end calls `/auth/refresh` one time and retries. If the retry also returns 401, it clears the session and redirects to `/login?next=<page>`.

Facts collected so far:

1. This week support received 47 tickets like "I get logged out every few minutes" and "the page keeps bouncing between the login page and the dashboard". Most users say it stops by itself after a minute or two.
2. The overall API 401 rate rose from about 0.3% to 2.4% of requests. Views of `/login` rose about 3x.
3. A front-end release on Tuesday at 10:05 changed the session cookie from `SameSite=Lax` to `SameSite=Strict` and moved the refresh-and-retry logic into Next.js middleware.
4. A teammate reproduced a bounce: they opened an app link from an email in a new tab, the first request arrived without the cookie because of `SameSite=Strict`, and they landed on `/login`. The teammate wrote: "It's the SameSite change. Revert to Lax and we're done."
5. The team lead's summary: "Started after Tuesday's front-end release. No API deploys this week. Auth service unchanged."
6. The earliest ticket was filed on Monday at 19:41.
7. The infrastructure event log shows that on Monday at 19:12 the autoscaling group replaced api-3 after a failed health check. The new VM booted from base image v42, published last Thursday. The image changelog says: "Switch to minimal base image; remove unused packages."
8. 401 rate per instance on Wednesday: api-1 0.3%, api-2 0.2%, api-3 9.1%, api-4 0.3%.
9. A sample 401 response body from api-3: `{"error":"invalid_token","error_description":"The token is not yet valid (iat)"}`. 96% of the 401s on api-3 have this description. The 401s on the other instances are almost all "token expired".
10. All sampled 401 requests on api-3 carried a session cookie with a token. In the sample, every rejected token was less than 65 seconds old, measured by the load balancer clock. Tokens older than that were accepted by api-3.
11. For the same request IDs, api-3 application log timestamps are 94-96 seconds earlier than the load balancer access log timestamps. On api-1, api-2, and api-4 the difference is under 1 second.
12. `timedatectl` on api-3 prints `System clock synchronized: no` and `NTP service: n/a`.
13. Tickets come from Chrome, Safari, and Firefox users. Most users describe it happening while they click around inside the app, not when they open a link from email.

Find the root cause and propose a fix. Answer in at most 350 words: your diagnosis, how sure you are and why, and the fix. You cannot run anything; reason from what is below. The team wants a fix PR today.
