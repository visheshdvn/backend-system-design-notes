---
created: 2026-08-30
status: draft
tags: []
---

# Spring Cloud Gateway Rate Limiter

Spring Cloud Gateway ships a `RequestRateLimiter` filter whose default implementation, `RedisRateLimiter`, is a [[token-bucket]] backed by [[Redis]] and executed as an atomic Lua script. It is the framework's answer to most of what [[Design a Rate Limiter]] builds by hand.

## Configuration

Three parameters, mapping onto the token bucket directly:

```yaml
spring.cloud.gateway.routes[0].filters[0].name: RequestRateLimiter
spring.cloud.gateway.routes[0].filters[0].args:
  redis-rate-limiter.replenishRate: 100
  redis-rate-limiter.burstCapacity: 200
  redis-rate-limiter.requestedTokens: 1
```

`replenishRate` is the sustained rate `R`, `burstCapacity` is the bucket size `B`, and `requestedTokens` is the cost of one request — the knob that lets an expensive endpoint consume more than one token, turning a request limiter into a rough cost limiter.

Setting `burstCapacity` equal to `replenishRate` disables bursting. Setting it to zero blocks all traffic, which is an occasionally useful way to disable a route.

## The KeyResolver

The filter will not work until you supply a `KeyResolver` bean deciding what to count along — the limit-key question that [[rate-limiting]] treats as the first and most consequential decision. There is no default, deliberately, because no default is safe.

```java
@Bean
KeyResolver apiKeyResolver() {
    return exchange -> Mono.just(exchange.getRequest().getHeaders().getFirst("X-API-Key"));
}
```

The reactive signature matters: this runs on the Netty event loop, so the resolver must not block. Resolving a tenant by hitting a database here will stall the loop under load and degrade every route on the gateway, not just this one. Look the tenant up from an already-cached claim in the token instead.

A resolver returning `Mono.empty()` denies the request outright. That is the correct behaviour for a missing API key, but it surprises people who expected an unkeyed request to fall through unlimited.

## Behaviour worth knowing before you rely on it

**It fails open.** If Redis is unreachable, the filter logs and allows the request. That is the sane default for capacity protection and the wrong one for an authentication endpoint, per the failure discussion in [[Design a Rate Limiter]]. If you need fail-closed anywhere, you are writing your own `RateLimiter` implementation.

**Headers are on by default.** The filter emits `X-RateLimit-Remaining`, `X-RateLimit-Burst-Capacity`, and `X-RateLimit-Replenish-Rate`. Disable them with `redis-rate-limiter.includeHeaders: false` if you would rather not advertise the policy.

**Rejection is a bare 429.** No `Retry-After`, so clients have nothing to pace against and will retry blindly. Adding that header — with jitter — takes a small custom filter, and skipping it is how you end up with the synchronised retry storm described in [[Design a Rate Limiter]].

**It needs the reactive Redis starter**, `spring-boot-starter-data-redis-reactive`. The blocking starter will not satisfy it, and the resulting failure names a missing bean rather than the actual problem.

## What you give up

You get one algorithm. Token bucket is a good default, but there is no path to [[sliding-window-counter]] or [[sliding-window-log]] short of implementing the `RateLimiter` interface yourself — so the login-endpoint case, where exactness is a security property, is not served by this filter.

You also inherit Spring Cloud Gateway itself, which is a reactive, Netty-based component. If the rest of the estate is servlet-stack Spring MVC, adopting the gateway for rate limiting alone means taking on a second concurrency model, and its debugging characteristics under load are not the ones the team already knows. For a single service, [[bucket4j]] in a filter is a considerably smaller commitment.
