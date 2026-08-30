---
created: 2026-08-30
status: stub
tags: []
---

# Redis

Redis is an in-memory data structure store, used here as the shared state behind a distributed limiter because its single-threaded execution model makes a Lua script an atomic read-modify-write.
