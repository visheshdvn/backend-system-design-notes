---
created: 2026-08-30
status: draft
tags: [interview-relevant]
---

# Rate Limiting

Rate limiting is capping how many operations a given caller may perform in a given period of time, and rejecting or delaying the ones beyond that cap. It is the mechanism that turns "this service has finite capacity" from an implicit hope into an enforced contract.

## What problem it solves

Four distinct problems wear the same clothes, and conflating them is the most common design mistake.

**Capacity protection.** A service can serve some number of requests per second before latency degrades. Without a limiter, one caller's bad deploy — a retry loop with no backoff, say — consumes that capacity and everyone else's requests queue behind it. The limiter converts a system-wide brownout into a targeted rejection of the caller causing it.

**Fairness between tenants.** On a shared multi-tenant system, capacity is a commons, and one heavy tenant will happily take all of it. A per-tenant limit makes each tenant's consumption independent of every other tenant's behaviour, which is the property people actually mean by "isolation" here.

**Cost control.** When a request costs real money downstream — an LLM token budget, an SMS gateway, a metered third-party API — the limit is a spending cap, not a capacity cap. The system could physically serve more; you don't want to pay for it.

**Abuse prevention.** Credential stuffing, scraping, and enumeration all look like a normal request repeated at abnormal volume. A limit on login attempts per account is a security control that happens to share an implementation with the other three.

These pull in different directions. Capacity protection wants a global limit across all callers. Fairness wants per-tenant limits that deliberately sum to more than capacity, because tenants are rarely all busy at once. Cost control wants a long window — a monthly quota — where capacity protection wants a window of seconds. A real system usually runs several limiters at different layers rather than one limiter trying to satisfy all four.

## Throttling versus shedding

Two different things can happen to a request that exceeds a limit -

**Shedding** rejects the request immediately — HTTP 429, and the caller decides whether to retry. The request costs almost nothing to refuse, and the caller learns about the limit right away.

**Throttling** delays the request until it fits within the limit, holding it in a queue. The caller sees latency instead of an error. This is gentler, but it converts a load problem into a memory problem: queued requests occupy connections and buffers, and a queue that grows faster than it drains is just a slower way to fail. See [[backpressure]] for how that signal should propagate back to the caller.

Shedding is the right default for public APIs. Throttling makes sense for internal batch traffic, where the caller has nowhere better to be and the queue depth is bounded.

## The four questions

Any rate limiter design answers these, whether or not it says so out loud.

**What is the limit key?** The dimension you count along — API key, user ID, IP address, tenant, endpoint, or a tuple of several. This choice matters more than the algorithm and gets the least attention. Limiting by IP breaks for users behind a corporate NAT, where thousands of people share one address, and is trivially evaded by an attacker with a proxy pool. Limiting by account is meaningless for an unauthenticated signup endpoint, which is exactly where you need it most.

**Which algorithm?** [[token-bucket]] and [[leaky-bucket]] think in terms of a flow rate; [[fixed-window-counter]], [[sliding-window-log]], and [[sliding-window-counter]] think in terms of counting events in a period. They differ mainly in how they treat bursts and how much state they cost.

**Where does the state live?** In-process memory is fast and wrong the moment you run more than one instance — N instances each enforcing a limit of L together enforce a real limit of N×L. Shared state in [[Redis]] is correct, but it puts a network round trip in every request path and makes the limiter a dependency that can take down the thing it protects.

**What happens on rejection?** The status code, whether you tell the caller when to come back, and — critically — what the limiter does when *it* fails.

## What you give up

Every rate limiter is a lie about capacity. The limit is a number someone chose, usually from a load test run under conditions that no longer hold, and it does not track the system's actual health. A limit set when queries were cheap becomes far too generous after a schema change makes them expensive, and far too strict after you add a cache. The limiter will happily admit traffic that melts the service, and reject traffic the service could easily have served.

You also accept false rejections. A legitimate user who happens to sit behind the same NAT as a scraper, or whose client library retries aggressively on a flaky connection, gets a 429 they did nothing to earn. This is a real cost paid by real users and it is invisible in the metrics — rejections look like the system working correctly.

And the limiter becomes a dependency. A Redis-backed limiter in front of every request means Redis latency is now your latency, and Redis being unavailable is a decision you must have made in advance: fail open and lose all protection at the worst possible moment, or fail closed and manufacture a total outage from a partial one. Neither answer is comfortable, which is why the choice belongs in a design review rather than in a catch block.

## Where it shows up

The end-to-end walkthrough lives in [[Design a Rate Limiter]]. On the Spring side, [[spring-cloud-gateway-rate-limiter]] gives you a Redis-backed token bucket by configuration rather than by writing one.
