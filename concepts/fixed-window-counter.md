---
created: 2026-08-30
status: draft
tags: []
---

# Fixed Window Counter

Fixed window counter is the simplest [[rate-limiting]] algorithm. Divide time into fixed intervals — say each calendar minute — keep one counter per `(key, window)` pair, increment it on each request, and reject once it exceeds the limit. When the clock rolls over into the next minute, counting starts again from zero.

In Redis this is two commands and one key that expires on its own:

```
INCR   ratelimit:user42:2026-08-30T14:03
EXPIRE ratelimit:user42:2026-08-30T14:03 60
```

## Why anyone uses it

It is the cheapest correct-ish thing available. One integer per active key, no timestamps to store, no arithmetic beyond an increment, and expiry handled by the store rather than by a cleanup job. The window boundary is embedded in the key name, so old windows evict themselves and there is no state to reclaim.

It is also the easiest to explain to the people who set the limits. "10,000 requests per hour" maps exactly onto what the algorithm does, with none of the burst-parameter confusion that [[token-bucket]] introduces.

## The boundary problem

The flaw is structural and it is the reason this algorithm gets replaced.

Because the counter resets instantaneously at the window edge, a caller can spend its full allowance at the very end of one window and its full allowance again at the very start of the next. With a limit of 100 per minute, 100 requests at 14:03:59 and 100 more at 14:04:00 are both permitted — 200 requests inside a two-second span, against a policy that claimed to allow 100 per minute.

The peak burst is exactly twice the intended limit, and it arrives in an arbitrarily short interval. Worse, it is not a rare accident: any client that polls on a round schedule, or that retries after a rejection by waiting for the window to reset, is *drawn toward* the boundary. Rate limiters that reject with a `Retry-After` pointing at the window reset actively synchronise their clients into this pattern, converting a theoretical edge case into a scheduled [[thundering-herd]].

[[sliding-window-log]] eliminates this exactly, and [[sliding-window-counter]] eliminates most of it for almost the same cost as fixed window.

## What you give up

You give up the guarantee your limit appears to make. Provisioning against a fixed window limit means provisioning for double it, and the doubling arrives concentrated rather than spread out — which is the harder shape to absorb. Whether that matters depends entirely on the window length. At a one-minute window, 2× over two seconds is a genuine spike. At a monthly quota window, the same structural flaw is irrelevant, because nobody is capacity-planning around the first second of the month.

That is the real rule for this algorithm: fixed window is fine for long windows where the limit is about cost or quota, and a poor fit for short windows where the limit is about capacity.

You also give up any notion of fairness within the window. The first caller to arrive after a reset takes the whole allowance, so a fast, tightly-looping client systematically beats a polite one to the counter.
