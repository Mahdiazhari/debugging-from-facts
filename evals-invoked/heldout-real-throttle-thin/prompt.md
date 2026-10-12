---
max_turns: 10
allowed_tools: [Skill]
---

Use the debugging-from-facts skill if it is available to you.

# Debugging task

Find the root cause and propose a fix. Answer in at most 350 words: your diagnosis, how sure you are and why, and the fix. You cannot run anything; reason from what is below. The team wants a fix PR today.

## Report

A document portal has a Django REST Framework (DRF) backend. Uploads are
rate-limited by a DRF scoped throttle: `upload` = 20/hour per user. An
operator ran a bulk upload for one user account on Monday.

## Facts

1. The bulk run got 77 responses `201 Created`, then `429 Request was
   throttled. Expected available in 2510 seconds.` and later waits of up to
   3305 s.
2. A backend deploy at 14:20 UTC restarted the backend. Between 14:43 and
   14:48 the same user got 76 more `201`s, then a `429`.
3. During the 429 windows, an alert fired: 19 responses `503` from the
   ingress in front of the backend. No 503 outside those windows.
4. Last week a PR changed `DocumentListCreateView.get_throttles()` so that
   only POST requests use the `upload` scope (GET uses the default `user`
   scope). The PR summary says "throttle rates unchanged".
5. A teammate says: "The limit is per IP, and the bulk script ran from two
   machines, so it got double the quota."
6. The bulk script used one machine and one account (from its own log).
