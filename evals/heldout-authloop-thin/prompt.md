---
max_turns: 10
allowed_tools: [Skill]
---

Incident ticket: users logged out and caught in login redirect loops

Setup: the front end is a Next.js app. The back end is a separate JSON API with 4 instances (api-1 to api-4) behind a load balancer. An auth service issues JWT access tokens (15-minute lifetime) and refresh tokens. The front end keeps the tokens in an HTTP-only session cookie. When an API call returns 401, the front end calls `/auth/refresh` one time and retries. If the retry also returns 401, it clears the session and redirects to `/login?next=<page>`. After login, the user goes back to `next`.

Facts collected so far:

1. This week support received 47 tickets like "I get logged out every few minutes" and "the page keeps bouncing between the login page and the dashboard". Most users say it stops by itself after a minute or two.
2. The overall API 401 rate rose from about 0.3% to 2.4% of requests.
3. Views of `/login` rose about 3x. Successful logins per active user per day rose from 1.1 to 3.8.
4. Tickets come from Chrome, Safari, and Firefox users, on desktop and mobile.
5. A front-end release on Tuesday at 10:05 changed two things: the session cookie changed from `SameSite=Lax` to `SameSite=Strict`, and the refresh-and-retry logic moved from a client-side fetch wrapper into Next.js middleware.
6. A teammate reproduced a bounce: they opened an app link from an email in a new tab. The first request arrived without the session cookie because of `SameSite=Strict`, and they landed on `/login`. The teammate wrote: "It's the SameSite change. Revert to Lax and we're done."
7. The team lead's summary in the ticket: "Started after Tuesday's front-end release. No API deploys this week. Auth service unchanged."
8. The ticket system shows tickets by filing day: Monday 3, Tuesday 14, Wednesday 19, Thursday 11. It does not show the time of day.
9. Nobody has looked at API-side logs for the 401s yet. The 401 rate on the dashboard is one number for all instances.

Find the root cause and propose a fix. Answer in at most 350 words: your diagnosis, how sure you are and why, and the fix. You cannot run anything; reason from what is below. The team wants a fix PR today.
