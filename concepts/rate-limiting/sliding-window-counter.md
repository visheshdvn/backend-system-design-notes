---
created: 2026-08-30
status: draft
tags: [interview-relevant]
---
# Sliding Window Counter

Sliding window counter is a [[rate-limiting]] algorithm that approximates a true sliding window using two fixed-window counters. It keeps [[fixed-window-counter]]'s cost and removes most of its boundary flaw, which makes it the usual production compromise.

## How it works

Keep the counter for the current window and the counter for the previous one. To estimate the request count over the trailing window, take all of the current window's count and a *fraction* of the previous window's, weighted by how far into the current window you are.

If the window is a minute and the clock reads 14:04:15 — a quarter of the way in — then 45 seconds of the trailing minute fall inside the previous window:

```
estimate = current + previous * (1 - 0.25)
```

With 30 requests counted in the current minute and 80 in the previous, the estimate is `30 + 80 × 0.75 = 90`. Against a limit of 100, the request is allowed.

The weight moves smoothly from 1 to 0 as the current window fills, so the previous window's influence decays continuously rather than vanishing at the boundary. That single change is what kills the doubling: a caller that spent its full allowance just before the boundary carries almost all of that spend forward across the edge, and cannot immediately spend again.

## Why this is usually the answer

It costs two integers per key — one more than fixed window, and dramatically less than [[sliding-window-log]]'s one entry per request. Both counters are plain `INCR`-able values with natural expiry. It has no burst parameter to misconfigure, unlike [[token-bucket]]. And the error is small and bounded in a direction you can reason about.

Cloudflare published measurements on this years ago: against 400 million requests, the approximation produced well under 1% incorrect decisions. For capacity protection that is comfortably good enough, because the limit itself was an estimate to begin with.

## What you give up

Accuracy, in a specific and knowable way. The weighting assumes requests were spread *uniformly* across the previous window, and that assumption is wrong precisely when it matters. If a caller crammed all 80 of its previous-window requests into the final second before the boundary, the algorithm still treats them as evenly spread and discounts 25% of them — so it under-counts a genuine burst and admits slightly more than the policy allows. The converse also happens: a caller whose traffic landed at the *start* of the previous window is over-counted and gets rejected slightly early.

So this algorithm can both over-admit and under-admit, by a margin that depends on traffic shape rather than on anything you control. If you need a hard guarantee — a legal or financial limit where "approximately 100" is not an acceptable answer — this is the wrong tool and [[sliding-window-log]] is the right one.

You also give up a little explicability. When a customer asks why they were rejected at 96 requests against a limit of 100, the honest answer involves a weighted estimate over two windows, and that is a harder conversation than either "you made 100 requests" or "your bucket was empty".
