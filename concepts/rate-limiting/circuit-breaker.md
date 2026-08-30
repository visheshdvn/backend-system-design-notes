---
created: 2026-08-30
status: stub
tags: []
---

# Circuit Breaker

A circuit breaker stops calling a dependency that has been failing, returning an immediate error for a cooling-off period so that a slow or broken downstream cannot exhaust the caller's threads and connections.
