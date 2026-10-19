---
created: 2026-08-30
status: draft
tags:
  - algorithm
---

# Sliding Window Log

Sliding window log is the exact [[rate-limiting]] algorithm. Instead of counting requests into buckets, it stores a timestamp for every request and answers the question directly: how many requests has this key made in the last `W` seconds?

On each request, drop every stored timestamp older than `now - W`, count what remains, and allow the request if the count is below the limit — appending its timestamp if so. In Redis this is a sorted set scored by timestamp, trimmed with `ZREMRANGEBYSCORE` and counted with `ZCARD`.

## Why it is worth knowing

It is the only algorithm here with no approximation error. The window truly slides: there is no boundary artefact like [[fixed-window-counter]]'s doubling, no weighted estimate like [[sliding-window-counter]]'s, and no burst allowance like [[token-bucket]]'s. If the policy says 100 requests per minute, then at every instant the trailing minute contains at most 100 requests.

That exactness earns its cost in a narrow band of cases: low limits where each rejection has real consequences, and where you may have to justify the decision afterwards. Login attempts before lockout, password reset requests, financial transaction submissions, anything where "the system said you did this 6 times in an hour" needs to be literally true. The log is also its own audit trail — you can see exactly when the requests came in, which no counter-based algorithm can tell you.

## What you give up

Memory, and not by a small factor. Every algorithm here stores one or two numbers per key; this one stores one entry per *request* per key. A limit of 5,000 requests per hour means up to 5,000 timestamps held for that key, and multiplying that by a large key space is how this algorithm becomes the outage. Fixed window would have used a single integer for the same policy.

The cost also scales with the limit rather than with traffic, which inverts the usual intuition: raising a limit from 100 to 100,000 is free under token bucket and a thousand-fold memory increase here.

There is a second, subtler cost. The trim-count-append sequence is a read-modify-write over a structure that must not be interleaved with another instance's, so it needs to be atomic — a Lua script or a transaction, not three round trips. The naive implementation looks correct in testing and silently over-admits under concurrency, which is the failure mode discussed in [[Design a Rate Limiter]].

And the trim itself is unbounded work. A key that has been idle for hours accumulates nothing, but a key at its limit pays to remove expired entries on every single request, so the expensive path is the hot path.

The practical consequence is that sliding window log is a specialist tool. Reach for it when exactness is a requirement you can name, and reach for [[sliding-window-counter]] when it is merely a preference.
