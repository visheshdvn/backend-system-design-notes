---
created: 2026-08-30
status: draft
tags: []
---

# Rate Limiting MOC

Rate limiting looks like one topic and is really three: a policy question about what you are protecting and from whom, an algorithm question about how you count, and a systems question about where the state lives once you have more than one server. Most confused discussions are people answering different ones of those.

## Start here

[[rate-limiting]] is the entry point and worth reading before any of the algorithms. It separates the four reasons anyone rate limits — capacity, fairness, cost, abuse — which pull in genuinely different directions, and it frames the four questions every design has to answer. The algorithms below are the answer to only one of those four questions, which is the proportion of attention they usually fail to get.

## The algorithms

Read these as two pairs rather than four independent options.

**Rate-shaped.** [[token-bucket]] and [[leaky-bucket]] think about flow. Token bucket allows bursts in proportion to recent idleness, which is why it is the common default — real client traffic is bursty and mostly innocent. Leaky bucket is its mirror: it smooths the *output* instead of the input, and it earns its place only when the downstream genuinely cannot absorb a spike. Read them together; the contrast is where the understanding is.

**Count-shaped.** [[fixed-window-counter]] is the cheap, obvious one, and its boundary flaw — 2× the limit across a window edge — is the specific problem the other two exist to solve. [[sliding-window-log]] solves it exactly and pays in memory proportional to the limit. [[sliding-window-counter]] solves most of it for two integers, which is why it usually wins. That progression is the actual argument, so read them in that order rather than alphabetically.

The short version of the trade: token bucket when bursts are legitimate, sliding window counter when they are not, sliding window log when "approximately" is not an acceptable answer, fixed window when the window is long enough that its flaw stops mattering.

## Putting it together

[[design-a-rate-limiter]] is where the concepts stop being separable. The decisions that only surface at assembly time are the atomicity trap — read-modify-write across instances is a lost-update race that testing will not catch — and the failure policy, where fail-open and fail-closed are both bad and the real answer is a degraded in-process fallback. It also covers why `Retry-After` without jitter builds a [[thundering-herd]] on purpose.

[[spring-cloud-gateway-rate-limiter]] is the same thing when the framework has already done it: a Redis-backed token bucket by configuration, with a fail-open default and a bare 429 that you will probably want to improve. [[bucket4j]] is the smaller-commitment alternative for a single service.

## Where this connects outward

[[backpressure]] is the neighbouring idea — rate limiting rejects at the door, backpressure propagates the signal upstream so the load is never generated. [[load-shedding]] is the honest critique of everything here: static limits are numbers someone guessed, while shedding reacts to live health signals. Everything in this cluster is the version you ship first, knowing that.

[[circuit-breaker]] and [[api-gateway]] show up in the assembly note as supporting machinery rather than as part of the topic proper.

## Gaps

Nothing here covers **distributed coordination cost** — the question of whether limiter state can be approximate and gossiped between nodes rather than centralised in [[redis]], which is how limiters work at CDN scale. That is the largest missing piece.

Also thin: **client-side rate limiting** and the retry-with-backoff behaviour that decides whether a limiter helps or amplifies, and **quota versus rate**, where the long-window billing case behaves differently enough to deserve its own note.
