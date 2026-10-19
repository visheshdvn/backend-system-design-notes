---
tags:
  - concurrency
  - jvm
  - "#java"
  - "#java-internals"
---

**Definition:** The JMM is a specification that defines how the Java Virtual Machine (JVM) manages memory. It dictates how threads interact with memory and how the JVM controls the visibility and ordering of variable changes across multiple threads.

---

## 1. JVM Memory Architecture: Heap vs. Stack
The JVM divides memory into two primary areas that determine how data is shared or isolated.

* **The Heap (Shared Memory)**
	* **What it stores:** All objects (class instances), arrays, and their instance variables.
	* **Concurrency impact:** Shared across all threads. Any thread with a reference to an object here can modify it, making this the primary source of concurrency issues.
* **Thread Stacks (Private Memory)**
	* **What it stores:** Method calls (stack frames), local variables, and primitive types.
	* **Concurrency impact:** Strictly private to the thread that created it. Local variables are inherently thread-safe.

## 2. The Concurrency Problem (Hardware vs. Software)
Modern CPUs have multiple cores, each with its own high-speed cache (L1, L2, L3). When a Java thread updates an object in the shared Heap, the CPU often writes it to its *local cache* first for performance, rather than immediately flushing it to main memory (RAM).

This hardware optimization creates three critical problems:
1. **Visibility:** Thread A updates a variable in its local cache. Thread B (on another core) reads the old "stale" value from main memory or its own cache.
2. **Atomicity (Race Conditions):** Operations like `count++` are not atomic (they require Read $\rightarrow$ Increment $\rightarrow$ Write). Without synchronization, simultaneous threads will overwrite each other's work.
3. **Instruction Reordering:** The compiler and CPU can reorder instructions to optimize performance. This is safe for a single thread, but can completely break the logic of a multi-threaded program.

## 3. The Solution: "Happens-Before" & JMM Rules
The JMM guarantees memory consistency using the **"Happens-Before"** relationship. 
* *Rule:* If Action A *happens-before* Action B, the JMM guarantees that the results of A are fully visible to B, and A is ordered before B.

The JMM enforces this using specific keywords:

### `volatile` Keyword
* **What it does:** Forces the CPU to immediately flush writes to main memory (bypassing the cache) and forces reads to fetch directly from main memory.
* **Solves:** The **Visibility** problem and prevents instruction **Reordering** around that variable.
* **Limitation:** Does *not* solve Atomicity (e.g., `volatile count++` is still not thread-safe).

### `synchronized` Blocks / Locks
* **What it does:** 
	* *Entering* the block: Clears the thread's local cache (forces reads from main memory).
	* *Exiting* the block: Flushes all modified variables back to main memory.
* **Solves:** Both **Visibility** and **Atomicity** (ensures mutual exclusion).

### Thread Lifecycle Rules
* **`Thread.start()`:** Calling start on a thread *happens-before* any code executing inside that new thread. (The child thread sees everything the parent did).
* **`Thread.join()`:** Returning from a join *happens-before* the next instruction in the calling thread. (The parent thread sees everything the child did before it terminated).