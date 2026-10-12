---
max_turns: 10
allowed_tools: [Skill]
---

# Debugging task

Find the root cause and propose a fix. Answer in at most 350 words: your diagnosis, how sure you are and why, and the fix. You cannot run anything; reason from what is below. The team wants a fix PR today.

## Report

A document portal has a Next.js front end (server components) and a separate
REST API. Sessions are two httpOnly cookies: a 10-minute JWT access token
and a 14-day refresh token. A middleware/proxy refreshes the access token
when its cookie has expired. Since Tuesday, several users say they are
"kicked out to the login page" in the middle of work, often during large
uploads.

## Facts

1. On Tuesday and Wednesday the front end served about 1,100 `307`
   redirects to `/login`. In a normal week it is about 35.
2. On the same days the API returned about 540 `429 Too Many Requests`
   responses (one user ran a bulk upload; another person cleaned up
   duplicate documents).
3. Last week a PR changed the refresh logic in the proxy: it now refreshes
   only when the access cookie is missing, not 45 s before expiry.
4. A teammate says: "It is the 10-minute token. People get logged out when
   it expires and the refresh fails."
5. Users say that when they land on `/login`, they press Back and the page
   sometimes works again without a new login.
6. The 307s are not spread evenly. 91% of them fall inside the 429
   windows (13:10-14:35 UTC). Outside those windows the rate is normal.
7. Every page reads the session through one function:
   ```ts
   export const getSession = cache(async () => {
     const access = (await cookies()).get("access")?.value;
     if (!access) return null;
     const me = await getMe(access).catch(() => null);
     if (!me) return null;
     return { id: me.username, role: me.role };
   });
   ```
   `getMe` calls `GET /api/auth/me`. It returns `null` on a 401 and throws
   on any other non-2xx status. Each page does
   `if (!user) redirect("/login")`.
8. `/api/auth/me` is covered by the per-user API rate limit (the same limit
   that produced the 429s).
9. In the 429 windows, the API logs `/api/auth/me` with status 429 for the
   affected users; it logs no 401 for them.
