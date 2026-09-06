# Java Multithreading and Concurrency — My Notes

Quick-recall bullet notes — read a point, explain it in my own words.

---

## 1. Why Multithreading Exists

- Analogy: restaurant — one waiter takes orders, another serves food, another clears tables — all at once
- Without multithreading: program does one thing at a time, finishes fully before next task
- Real example: music app without multithreading would freeze entire UI while downloading next song
- Thread = smallest unit of work CPU can execute
- Every Java program starts with one thread: the **main thread**
- Threads **share the same heap memory** — powerful (can share data) but dangerous (race conditions)

---

## 2. Creating a Thread — 3 Ways

### Way 1: extends Thread
- Override `run()`, call `.start()` (NOT `.run()` directly — calling `.run()` just runs it like a normal method on current thread, no new thread created)
- Downside: class already extends Thread, can't extend anything else (Java single inheritance)

### Way 2: implements Runnable (preferred)
- Implement `run()`, wrap it: `new Thread(task).start()`
- Task and Thread are separate objects — class can still extend something else
- Same Runnable can be reused across multiple threads

### Way 3: Lambda (shortest, most common)
- `new Thread(() -> { ... }).start()` — since Runnable has only one method, lambda replaces the whole class

**Rule of thumb**: Always prefer Runnable/lambda over extending Thread — more flexible, decouples task from thread.

---

## 3. Thread Lifecycle

```
NEW → RUNNABLE → (RUNNING) → TERMINATED
                     |
                     |→ BLOCKED
                     |→ WAITING
                     |→ TIMED_WAITING
(these three go back to RUNNABLE once condition met)
```

- **NEW**: object created, `start()` not called yet
- **RUNNABLE**: `start()` called, ready for CPU, OS scheduler decides when it actually runs
- **RUNNING**: conceptual only — Java API shows this as RUNNABLE too, no separate state
- **BLOCKED**: waiting to enter a `synchronized` block that another thread currently holds
- **WAITING**: waiting indefinitely for another thread's action — caused by `join()` (no timeout) or `object.wait()`
- **TIMED_WAITING**: same as WAITING but auto-wakes after time limit — caused by `sleep(ms)`, `join(ms)`, `wait(ms)`
- **TERMINATED**: `run()` has finished. Calling `start()` again → `IllegalThreadStateException` (can't restart a thread)

### Key Thread methods
- `setName()`/`getName()` — thread naming
- `setPriority()` (1-10, default 5) — just a **hint** to OS, not guaranteed
- `isAlive()` — started and not yet terminated
- `setDaemon(true)` — **must be called before `start()`**
- `Thread.sleep(ms)` — pause current thread
- `Thread.currentThread()` — reference to currently running thread
- `t.join()` — calling thread waits until `t` finishes (useful when one task depends on another's result)
- `t.interrupt()` — sets interrupted flag; if sleeping/waiting, throws `InterruptedException` immediately

### Daemon threads
- Normal threads: JVM waits for them before shutting down
- Daemon threads: JVM does NOT wait — killed automatically once all normal threads finish
- Use case: background tasks (GC, auto-save, cleanup)

---

## 4. Runnable vs Callable

| | Runnable | Callable\<T\> |
|---|---|---|
| Method | `run()` | `call()` |
| Return | void | T (any type) |
| Checked exceptions | Must handle inside | Caller handles via `ExecutionException` |
| Result access | None | Via `Future<T>` |
| Used with | Thread directly | ExecutorService |

- **Future** = a container holding the result of an async task, available "in the future"
- `future.get()` — **blocks** until task completes
- `future.get(timeout, unit)` — throws `TimeoutException` if not done in time
- `future.isDone()` — non-blocking check
- `future.cancel(true)` — true = interrupt if running
- Callable's exception surfaces via `future.get()` throwing `ExecutionException` — use `e.getCause()` to get the real exception

---

## 5. ExecutorService and Thread Pools

- Creating a new `Thread()` every time is **expensive** (time, memory, OS resources)
- ThreadPool = fixed number of threads created upfront, reused for all tasks
- Analogy: taxi company — instead of buying a new car per customer, reuse a fixed fleet

### Types
- `newFixedThreadPool(n)` — exactly n threads, extra tasks queue up. Best for known parallel workload.
- `newSingleThreadExecutor()` — 1 thread, tasks run sequentially, but still async from caller's view
- `newCachedThreadPool()` — creates threads as needed, reuses idle ones, idle threads die after 60s. Good for many short-lived tasks, but risky — can spawn too many threads under heavy load
- `newScheduledThreadPool(n)` — for delayed/repeated tasks

### Submitting tasks
- `execute(Runnable)` — fire and forget, no way to check completion
- `submit(Runnable)` — returns `Future<?>`, `.get()` returns null but tells you when done
- `submit(Callable)` — returns `Future<T>` with actual result

### Shutdown — important
- `shutdown()` — graceful: stops accepting new tasks, lets submitted tasks finish, non-blocking
- `shutdownNow()` — forceful: sends interrupt to running tasks, returns list of tasks never started
- `awaitTermination(timeout, unit)` — waits for shutdown to actually complete
- **Without calling shutdown, JVM won't exit** — pool threads stay alive

### ScheduledExecutorService
- `schedule(task, delay, unit)` — runs once after delay
- `scheduleAtFixedRate(task, initialDelay, period, unit)` — next run starts `period` after **previous START** (if task overruns, next run starts immediately after)
- `scheduleWithFixedDelay(task, initialDelay, delay, unit)` — next run starts `delay` after **previous END**

---

## 6. synchronized Keyword

### The problem: Race Condition
- `count++` looks atomic but is actually 3 steps: read → add 1 → write back
- Two threads both reading 5, both adding 1, both writing 6 → one increment lost
- This unpredictable-outcome-from-shared-mutable-state issue = **race condition**

### synchronized method
- Only ONE thread can execute a synchronized method (on the same object) at a time — others wait
- Uses every object's built-in **monitor lock / intrinsic lock**
- Thread acquires lock → enters → other threads calling ANY synchronized method on the SAME object are blocked → lock released on exit → next waiting thread gets it
- Synchronized methods on **different objects** can run simultaneously — lock is per-object

### synchronized block
- Locks only the critical section, not the whole method — lets non-critical code run concurrently
- Can use a dedicated lock object instead of `this`

### static synchronized
- Locks on the **Class object**, not an instance — all instances share this one lock

### Deadlock
- Two threads each hold a lock the other wants → both wait forever
- Classic scenario: T1 holds lock1 wants lock2; T2 holds lock2 wants lock1
- **Fix**: always acquire multiple locks in the **same order** across all threads

---

## 7. volatile Keyword

### The problem: Memory Visibility
- CPUs cache variables per-thread for performance
- Thread A updates a variable; Thread B may keep reading its own **stale cached copy**, never seeing the update
- Can cause infinite loops (e.g. a `while(running)` loop that never sees `running = false`)

### volatile fixes visibility
- Forces every read/write to go directly to **main memory**, never a cached copy
- Use when: one thread writes, others only read, and the operation is a **single** read/write (not compound)

### volatile is NOT enough for compound operations
- `count++` is still NOT thread-safe even if `count` is volatile — it's read-modify-write, not a single operation
- Need `synchronized` or `AtomicInteger` for that

### volatile vs synchronized
| volatile | synchronized |
|---|---|
| Visibility only | Visibility + atomicity |
| No locking, fast | Uses lock, slower |
| Single read/write only | Works for compound ops too |
| Good for flags/status vars | Good for shared mutable state |

---

## 8. Atomic Classes

- Give both visibility AND atomicity **without locks** — use CPU-level **CAS (Compare And Swap)** instruction, faster than locking
- Package: `java.util.concurrent.atomic`

### AtomicInteger — key methods
- `incrementAndGet()` / `getAndIncrement()` — pre vs post increment semantics
- `decrementAndGet()` / `getAndDecrement()`
- `addAndGet(n)` / `getAndAdd(n)`
- `compareAndSet(expected, newValue)` — "if current value is X, change to Y" — returns true/false based on whether swap happened

### compareAndSet pattern — classic use case
- Safe "check-then-update" without locks: read current value, try `compareAndSet(current, newValue)`, if it fails (someone else changed it meanwhile) — **loop and retry** with fresh value
- Example: last-item-in-stock scenario with many concurrent buyers

### Other atomic types
- `AtomicLong`, `AtomicBoolean`, `AtomicReference<T>` — same idea, different data types

---

## 9. ReentrantLock and ReadWriteLock

### Why ReentrantLock over synchronized
`synchronized` can't: try-without-waiting, give up after timeout, be interrupted while waiting, or report queue length. `ReentrantLock` can do all of these.

- **Reentrant** = if a thread already holds the lock and requests it again, it succeeds (no self-deadlock) — same as synchronized's behavior

### Basic usage
```java
lock.lock();
try { /* critical section */ }
finally { lock.unlock(); }  // ALWAYS in finally — else exception leaves lock stuck forever
```

- `tryLock()` — returns immediately, true/false, no waiting
- `tryLock(timeout, unit)` — waits up to a limit, then gives up
- **Fair lock**: `new ReentrantLock(true)` — gives lock to longest-waiting thread first, prevents starvation (default is unfair — faster, but a thread could theoretically wait forever if unlucky)
- Extra introspection: `isLocked()`, `isHeldByCurrentThread()`, `getHoldCount()`, `getQueueLength()`

### ReadWriteLock
- Regular locks allow only ONE thread total — but reading is safe to do concurrently, only writing needs exclusivity
- **Read lock**: multiple threads can hold simultaneously (shared)
- **Write lock**: exclusive — only one thread, and only when no readers active
- Best for read-heavy scenarios (e.g. a cache)

### Comparison
| synchronized | ReentrantLock | ReadWriteLock |
|---|---|---|
| Simple, auto-release | Manual unlock in finally | Manual unlock in finally |
| No tryLock/timeout | Has tryLock/timeout | Has tryLock/timeout |
| Unfair only | Fair or unfair | Fair or unfair |
| One thread at a time | One thread at a time | Many readers OR one writer |
| Best default choice | When you need tryLock/timeout | Read-heavy scenarios |

---

## 10. Concurrent Collections

- Regular `ArrayList`/`HashMap` are NOT thread-safe — concurrent modification → `ConcurrentModificationException` or corrupted data
- Avoid `Collections.synchronizedList/Map()` in new code — locks the **entire** collection per operation, slow
- Prefer purpose-built concurrent collections instead

### ConcurrentHashMap
- Divides map into segments, locks only the segment being modified — multiple threads can work on different parts simultaneously
- Atomic helper methods: `putIfAbsent`, `compute`, `computeIfAbsent`, `computeIfPresent`, `merge`
- Safe iteration — no `ConcurrentModificationException`

### CopyOnWriteArrayList
- Every write (add/set/remove) creates a **brand new copy** of the internal array
- Readers always work on a stable snapshot — never blocked, never see partial writes
- Good for: many readers, few writers. Bad for: frequent writes (copying array every time is expensive)

### BlockingQueue
- A queue that **blocks**: `take()` waits if empty, `put()` waits if full
- Perfect for **producer-consumer** pattern
- `offer()`/`poll()` — non-blocking (return false/null immediately) or with a timeout variant
- `peek()` — look at head without removing

---

## 11. CompletableFuture

### Why it exists — Future's problems
1. `future.get()` blocks — defeats the purpose of "async"
2. Can't chain tasks (task B automatically starting after task A)
3. Can't combine results of parallel tasks cleanly
4. No exception-handling callback mechanism

CompletableFuture (Java 8) solves all four — chains, combines, and handles errors without blocking.

### Creating one
- `runAsync(runnable)` — task with no return value
- `supplyAsync(supplier)` — task that returns a value
- Default pool: `ForkJoinPool.commonPool()`, or pass your own executor

### Chaining methods
- `thenApply(fn)` — transform the result (like `map()`), synchronous next step
- `thenAccept(consumer)` — consume result, return nothing (good for a final step)
- `thenRun(runnable)` — run something after, ignoring the result entirely
- `thenCompose(fn)` — chain when the **next step is itself async** (returns another CompletableFuture) — avoids nested `CompletableFuture<CompletableFuture<T>>` (like `flatMap()`)
- `thenCombine(otherCf, biFunction)` — combine results of two **independent, parallel** tasks once both finish; total time = max(both), not sum

### Multiple tasks
- `allOf(cf1, cf2, ...)` — waits for ALL to complete (returns `CompletableFuture<Void>`, get individual results separately after)
- `anyOf(cf1, cf2, ...)` — returns as soon as the FIRST one completes

### Exception handling
- `exceptionally(fn)` — runs ONLY on exception, returns a fallback value
- `handle((result, exception) -> ...)` — runs ALWAYS; both params — check which is non-null to know if it succeeded or failed

---

## 12. Common Problems — Quick Reference

| Problem | What Happens | Fix |
|---|---|---|
| **Race Condition** | Shared mutable data read/written by multiple threads unpredictably | `synchronized`, `AtomicInteger`, `ReentrantLock` |
| **Deadlock** | Two threads each hold a lock the other wants — both wait forever | Always acquire multiple locks in the **same order** everywhere |
| **Memory Visibility** | One thread's update invisible to another due to CPU caching | `volatile` (single read/write) or `synchronized` (also gives visibility) |
| **Starvation** | A thread never gets the lock because others keep winning it | Fair lock: `new ReentrantLock(true)` |
| **Livelock** | Threads keep reacting to each other, changing state, but make no real progress (like two people repeatedly stepping the same way in a hallway) | Add randomness/backoff so threads don't always respond identically |
