---
tags:
  - locking
---

#locking #CAS

In Java 8+, `ConcurrentHashMap` avoids global synchronization by combining hardware-level atomics (**CAS**) with fine-grained mutual exclusion (**Bucket-Level `synchronized`**).

---

## 1. Overview of the Strategy

| Step | Bucket State | Mechanism Used | Concurrency Impact |
| :--- | :--- | :--- | :--- |
| **First Entry** | Empty (`null`) | **CAS** (`compareAndSet`) | **Lock-free.** Zero lock overhead; no threads put to sleep. |
| **Collision** | Occupied (List/Tree) | **Bucket Lock** (`synchronized(head)`) | **Localized.** Blocks only threads accessing the exact same bucket. |
| **Read (`get`)** | Any | **Volatile read** | **100% Lock-free.** Reads never acquire monitors. |

---

## 2. CAS (Compare-And-Swap) for Empty Buckets

When a key hashes to a bucket that has no entries yet (`table[i] == null`):

1. The map performs a hardware-level atomic CAS instruction:
   $$\text{CAS}(\text{address} = \text{table}[i], \text{expected} = \text{null}, \text{update} = \text{NewNode})$$
2. **If uncontended:** The write succeeds immediately without entering a monitor or context switching.
3. **If contested:** If another thread wrote to `table[i]` a nanosecond earlier, CAS fails. The thread does not crash—it simply retries in a spin loop, finds the bucket occupied, and transitions to the bucket-lock path.

---

## 3. Bucket-Level Lock for Collisions

When the target bucket is already occupied (`table[i] != null`), updating pointers in a linked list or red-black tree requires multi-step consistency.

Instead of locking the entire table or allocating separate lock objects:

```java
// Simplified internal logic:
Node<K, V> head = table[i];

synchronized (head) {
    // Only threads targeting bucket [i] block here.
    // Buckets 0, 1, 2 ... i-1, i+1 remain completely free.
    
    // Traverse the chain:
    // 1. Update value if key exists, OR
    // 2. Append new node at tail, OR
    // 3. Insert into Red-Black Tree (if bin is treeified)
}
```

### Why this design is effective:
* **Zero per-bucket lock objects:** It synchronizes directly on the existing `Node` instance (the head of the bucket), saving substantial memory overhead.
* **Independent partitions:** 1,000 threads writing to 1,000 distinct buckets execute in parallel without blocking each other.
* **Minimal hold time:** Locks are held strictly for the microsecond it takes to traverse that bucket's short chain (chains exceeding 8 elements convert into Red-Black trees to cap search time at $\mathcal{O}(\log n)$).

---

## 4. Execution Flow on `put(K, V)`

```
                  put(key, value)
                         │
                 Hash & find bucket [i]
                         │
               Is table[i] == null?
               ┌─────────┴─────────┐
             YES                  NO
               │                   │
         Attempt CAS         Lock bucket head
     (null -> NewNode)     via synchronized(head)
               │                   │
         CAS successful?      Traverse chain/tree:
         ┌─────┴─────┐        • Overwrite existing key
       YES          NO        • Or append new node
        │            │             │
      Done!       (Retry)────► Release lock
                                   │
                             Update counterCells 
                             (LongAdder mechanism)
```

---

## 5. Quick Reference Takeaways

* **CAS** provides lock-free, zero-overhead inserts into empty slots.
* **Bucket Lock** confines write contention to a single hash index by locking only the head node.
* **Volatile Reads** ensure readers never block on writer locks.