# Requirements
- It must store and retrieve Key-Value Pairs
- It should manage limited memory using cache eviction policies
- Supports 2 core operations
	- accessData(get)
	- updateData(write)
- It should be Thread safe and handle concurrent operations
- It should be extensible to support -
	- Different eviction policies (- How & what entry to evict from cache, e.g. LRU, LFU etc.)
	- Custom write policies (- How cache interacts with DB, Write-through, write-back, write-around etc.)
- Support read your own writes consistency - i.e. read operation should always get the latest write information

# Ideas
- We will use Generics for Key value pairs
- To ensure RYOW consistenct - all read & writes to the same key can be routed to the same thread. Since operations in a single thread runs sequentially it ensures request ordering & consistency.
	- (there can be other approaches as well)
- We can go with right-through write policy.
- For eviction policy we can use LRU.

# Out of Scope
- Request Collapsing
	- means: If there are multiple requests for the same key and cache doesn't have it, we should send only one request to db to get the data into the cache rather than a db call for every request
- Hot loading
	- Prefill cache with some values at startup from DB.
- 