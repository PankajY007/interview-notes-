# 03 — Multithreading & Concurrency (Interview Ready)

> **Level:** 3-4 Years Experience | **Priority:** 🔴 Critical  
> Covers: Threads, Synchronization, Locks, ExecutorService, CompletableFuture, Atomic, ThreadLocal

---

## 📌 Table of Contents

1. [Thread Basics](#1-thread-basics)
2. [Synchronization](#2-synchronization)
3. [Inter-Thread Communication](#3-inter-thread-communication)
4. [Locks & ReentrantLock](#4-locks--reentrantlock) — ReadWriteLock, StampedLock
5. [ExecutorService & Thread Pool](#5-executorservice--thread-pool)
6. [Callable, Future & CompletableFuture](#6-callable-future--completablefuture)
7. [Atomic Classes](#7-atomic-classes)
8. [ThreadLocal](#8-threadlocal)
9. [Synchronization Utilities](#9-synchronization-utilities) — Semaphore, CountDownLatch, CyclicBarrier
10. [Common Concurrency Problems](#10-common-concurrency-problems) — Deadlock, Race Condition, Thread Dump
11. [Virtual Threads (Java 21)](#11-virtual-threads-java-21)
12. [Interview Questions & Answers](#12-interview-questions--answers) — 15 Q&As

---

## 1. Thread Basics

### Creating Threads (3 Ways)

```java
// Way 1: Extend Thread
class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Thread: " + Thread.currentThread().getName());
    }
}
new MyThread().start();

// Way 2: Implement Runnable (PREFERRED — allows extending other class)
class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("Runnable: " + Thread.currentThread().getName());
    }
}
new Thread(new MyRunnable()).start();

// Way 3: Lambda (Java 8+)
new Thread(() -> System.out.println("Lambda thread")).start();
```

### Thread Lifecycle

```
NEW → (start()) → RUNNABLE → (scheduler) → RUNNING
  ↓                                           ↓
  ↓                                    ┌──────┴──────┐
  ↓                                    ↓             ↓
  ↓                               BLOCKED      WAITING/TIMED_WAITING
  ↓                              (lock wait)   (wait/sleep/join)
  ↓                                    ↓             ↓
  ↓                                    └──────┬──────┘
  ↓                                           ↓
  └──────────────────────────────────→  TERMINATED (DEAD)
```

### Thread Priority & Daemon Threads
```java
Thread t = new Thread(() -> {});
t.setPriority(Thread.MAX_PRIORITY); // 10 (hint to scheduler, not guaranteed)

// Daemon thread — dies when all user threads finish
Thread daemon = new Thread(() -> {
    while (true) { /* background work */ }
});
daemon.setDaemon(true); // MUST set before start()
daemon.start();
// JVM exits when only daemon threads remain
// GC is a daemon thread
```

### Key Methods
```java
Thread.sleep(1000);    // pause current thread for 1s (doesn't release lock)
thread.join();         // wait for thread to finish
thread.join(5000);     // wait max 5 seconds
Thread.yield();        // hint to scheduler to give other threads a chance
thread.interrupt();    // sets interrupt flag
Thread.currentThread().isInterrupted(); // check flag
```

**⚠️ Pitfall: `sleep()` vs `wait()`**
| Feature | sleep() | wait() |
|---------|---------|--------|
| Class | Thread | Object |
| Lock | Does NOT release | Releases lock |
| Wake up | After time expires | notify()/notifyAll() |
| Usage | Pause execution | Inter-thread communication |

---

## 2. Synchronization

### synchronized keyword
```java
public class Counter {
    private int count = 0;

    // Method-level sync — locks on 'this'
    public synchronized void increment() {
        count++;
    }

    // Block-level sync — more granular
    public void incrementBlock() {
        synchronized (this) {
            count++;
        }
    }

    // Static sync — locks on Class object
    public static synchronized void staticMethod() {
        // locks on Counter.class
    }
}
```

### Why `count++` is NOT thread-safe
```
count++ is actually 3 operations:
1. READ count from memory
2. INCREMENT count
3. WRITE count back to memory

Thread A reads count=5
Thread B reads count=5 (before A writes)
Thread A writes count=6
Thread B writes count=6 ← LOST UPDATE! Should be 7
```

### volatile Keyword
```java
// Ensures visibility — reads/writes go directly to main memory
// Does NOT guarantee atomicity
private volatile boolean running = true;

// Thread 1
public void stop() {
    running = false; // visible to all threads immediately
}

// Thread 2
public void run() {
    while (running) { // reads from main memory, not cache
        doWork();
    }
}

// ⚠️ volatile is NOT enough for compound operations like count++
// Use AtomicInteger for that
```

---

## 3. Inter-Thread Communication

### Producer-Consumer Pattern

```java
public class SharedBuffer {
    private final Queue<Integer> queue = new LinkedList<>();
    private final int capacity;

    public SharedBuffer(int capacity) {
        this.capacity = capacity;
    }

    public synchronized void produce(int item) throws InterruptedException {
        while (queue.size() == capacity) {
            wait(); // release lock, wait for consumer
        }
        queue.offer(item);
        System.out.println("Produced: " + item);
        notifyAll(); // wake up consumers
    }

    public synchronized int consume() throws InterruptedException {
        while (queue.isEmpty()) {
            wait(); // release lock, wait for producer
        }
        int item = queue.poll();
        System.out.println("Consumed: " + item);
        notifyAll(); // wake up producers
        return item;
    }
}
```

**⚠️ Always use `while` instead of `if` with `wait()`** — spurious wakeups can occur!

### Modern Alternative: BlockingQueue
```java
BlockingQueue<Integer> queue = new ArrayBlockingQueue<>(10);

// Producer
queue.put(item);  // blocks if full

// Consumer
int item = queue.take(); // blocks if empty

// No manual wait/notify needed!
```

---

## 4. Locks & ReentrantLock

```java
public class BankAccount {
    private final ReentrantLock lock = new ReentrantLock();
    private double balance;

    public void transfer(BankAccount target, double amount) {
        lock.lock(); // acquire lock
        try {
            if (balance >= amount) {
                balance -= amount;
                target.deposit(amount);
            }
        } finally {
            lock.unlock(); // ALWAYS in finally!
        }
    }

    // tryLock — non-blocking, avoids deadlock
    public boolean tryTransfer(BankAccount target, double amount) {
        if (lock.tryLock()) { // returns immediately if can't acquire
            try {
                balance -= amount;
                target.deposit(amount);
                return true;
            } finally {
                lock.unlock();
            }
        }
        return false; // couldn't acquire lock
    }
}
```

### ReentrantLock vs synchronized

| Feature | synchronized | ReentrantLock |
|---------|-------------|---------------|
| Lock type | Implicit | Explicit |
| Try lock | ❌ | ✅ `tryLock()` |
| Timed lock | ❌ | ✅ `tryLock(timeout)` |
| Interruptible | ❌ | ✅ `lockInterruptibly()` |
| Fair lock | ❌ | ✅ `new ReentrantLock(true)` |
| Condition | Single (wait/notify) | Multiple `Condition` objects |
| Scope | Method/Block | Any scope (can span methods) |

### ReadWriteLock
```java
ReadWriteLock rwLock = new ReentrantReadWriteLock();

// Multiple readers allowed simultaneously
public String read() {
    rwLock.readLock().lock();
    try {
        return data;
    } finally {
        rwLock.readLock().unlock();
    }
}

// Only one writer at a time, blocks readers too
public void write(String newData) {
    rwLock.writeLock().lock();
    try {
        this.data = newData;
    } finally {
        rwLock.writeLock().unlock();
    }
}
```

### StampedLock (Java 8+) — Optimistic Reads

```java
// Better than ReadWriteLock when reads vastly outnumber writes
StampedLock lock = new StampedLock();
private double x, y;

// Optimistic Read — NO locking! Just validates after reading
public double distanceFromOrigin() {
    long stamp = lock.tryOptimisticRead(); // non-blocking!
    double currentX = x, currentY = y;    // read shared state
    if (!lock.validate(stamp)) {           // was data modified during read?
        stamp = lock.readLock();           // fall back to full read lock
        try {
            currentX = x;
            currentY = y;
        } finally {
            lock.unlockRead(stamp);
        }
    }
    return Math.sqrt(currentX * currentX + currentY * currentY);
}

// Write Lock
public void move(double deltaX, double deltaY) {
    long stamp = lock.writeLock();
    try {
        x += deltaX;
        y += deltaY;
    } finally {
        lock.unlockWrite(stamp);
    }
}
```

| Lock | Read-Read | Read-Write | Optimistic Read | Reentrant? |
|------|-----------|------------|-----------------|------------|
| `synchronized` | Blocked | Blocked | ❌ | ✅ |
| `ReentrantReadWriteLock` | ✅ Concurrent | Blocked | ❌ | ✅ |
| `StampedLock` | ✅ Concurrent | Blocked | ✅ (no locking!) | ❌ |
```

---

## 5. ExecutorService & Thread Pool

### Thread Pool Types

```java
// Fixed — fixed number of threads
ExecutorService fixed = Executors.newFixedThreadPool(10);
// Use for: CPU-bound tasks, known load

// Cached — creates threads as needed, reuses idle ones
ExecutorService cached = Executors.newCachedThreadPool();
// Use for: many short-lived tasks

// Single — one thread (tasks execute sequentially)
ExecutorService single = Executors.newSingleThreadExecutor();
// Use for: sequential task execution, logging

// Scheduled — schedule tasks with delay or periodic execution
ScheduledExecutorService scheduled = Executors.newScheduledThreadPool(5);
scheduled.scheduleAtFixedRate(() -> cleanup(), 0, 1, TimeUnit.HOURS);
// Use for: cron-like jobs, periodic tasks
```

### Production Best Practice — Custom ThreadPoolExecutor

```java
// ⚠️ NEVER use Executors factory in production (Alibaba Java Guidelines)
// Executors.newFixedThreadPool → uses unbounded LinkedBlockingQueue → OOM risk
// Executors.newCachedThreadPool → unlimited threads → OOM risk

// ✅ Create custom pool with bounded queue
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    5,                      // corePoolSize
    10,                     // maxPoolSize
    60L,                    // keepAliveTime for idle threads
    TimeUnit.SECONDS,
    new ArrayBlockingQueue<>(100),  // bounded queue!
    new ThreadPoolExecutor.CallerRunsPolicy() // rejection policy
);
```

### Rejection Policies
| Policy | Behavior |
|--------|----------|
| `AbortPolicy` | Throws `RejectedExecutionException` (default) |
| `CallerRunsPolicy` | Caller thread executes the task (backpressure) |
| `DiscardPolicy` | Silently drops the task |
| `DiscardOldestPolicy` | Drops oldest task, adds new one |

### Proper Shutdown
```java
executor.shutdown(); // no new tasks, existing continue
try {
    if (!executor.awaitTermination(60, TimeUnit.SECONDS)) {
        executor.shutdownNow(); // interrupt all tasks
    }
} catch (InterruptedException e) {
    executor.shutdownNow();
    Thread.currentThread().interrupt();
}
```

---

## 6. Callable, Future & CompletableFuture

### Callable vs Runnable
```java
// Runnable — no return value, no checked exceptions
Runnable runnable = () -> System.out.println("No return");

// Callable — returns value, can throw exceptions
Callable<Integer> callable = () -> {
    Thread.sleep(1000);
    return 42;
};

Future<Integer> future = executor.submit(callable);
Integer result = future.get();           // blocks until done
Integer result = future.get(5, TimeUnit.SECONDS); // timeout
future.cancel(true);                     // cancel task
future.isDone();                         // check if completed
```

### CompletableFuture (Java 8+) — MOST ASKED

```java
// Async execution
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    return fetchUserFromDB(userId);
});

// Chaining — thenApply (transform result)
CompletableFuture<String> greeting = future
    .thenApply(user -> "Hello, " + user.getName());

// Chaining — thenCompose (flatMap — returns another CompletableFuture)
CompletableFuture<Order> orderFuture = getUserFuture
    .thenCompose(user -> getOrderFuture(user.getId()));

// Chaining — thenAccept (consume result, no return)
future.thenAccept(user -> log.info("User: {}", user));

// Combining two futures
CompletableFuture<String> userFuture = CompletableFuture.supplyAsync(() -> getUser());
CompletableFuture<String> orderFuture = CompletableFuture.supplyAsync(() -> getOrder());

CompletableFuture<String> combined = userFuture.thenCombine(orderFuture,
    (user, order) -> user + " ordered " + order);

// Wait for ALL futures
CompletableFuture<Void> all = CompletableFuture.allOf(future1, future2, future3);

// Wait for ANY (first to complete)
CompletableFuture<Object> any = CompletableFuture.anyOf(future1, future2);

// Exception handling
CompletableFuture<String> safe = future
    .exceptionally(ex -> "Default value")           // handle exception
    .handle((result, ex) -> {                        // handle both
        if (ex != null) return "Error: " + ex.getMessage();
        return result;
    });
```

### Real-World Example: Parallel API Calls
```java
public OrderSummary getOrderSummary(Long userId) {
    CompletableFuture<User> userFuture = CompletableFuture
        .supplyAsync(() -> userService.getUser(userId));
    
    CompletableFuture<List<Order>> ordersFuture = CompletableFuture
        .supplyAsync(() -> orderService.getOrders(userId));
    
    CompletableFuture<Double> walletFuture = CompletableFuture
        .supplyAsync(() -> walletService.getBalance(userId));

    // All 3 API calls run in parallel!
    return CompletableFuture.allOf(userFuture, ordersFuture, walletFuture)
        .thenApply(v -> new OrderSummary(
            userFuture.join(),
            ordersFuture.join(),
            walletFuture.join()
        )).join(); // blocking call at the end
}
```

---

## 7. Atomic Classes

```java
// Thread-safe WITHOUT locks — uses CAS (Compare-And-Swap)
AtomicInteger counter = new AtomicInteger(0);
counter.incrementAndGet();    // atomic i++
counter.decrementAndGet();    // atomic i--
counter.addAndGet(5);         // atomic i += 5
counter.compareAndSet(5, 10); // if current == 5, set to 10

AtomicReference<String> ref = new AtomicReference<>("initial");
ref.compareAndSet("initial", "updated");

// LongAdder (Java 8+) — better than AtomicLong for high contention
LongAdder adder = new LongAdder();
adder.increment(); // each thread updates its own cell
adder.sum();       // aggregates all cells
```

---

## 8. ThreadLocal

```java
// Each thread gets its own copy of the variable
public class UserContext {
    private static final ThreadLocal<String> currentUser = new ThreadLocal<>();

    public static void setUser(String user) {
        currentUser.set(user);
    }

    public static String getUser() {
        return currentUser.get();
    }

    public static void clear() {
        currentUser.remove(); // ⚠️ MUST clear in thread pools!
    }
}

// Real-world usage: Spring Security's SecurityContextHolder uses ThreadLocal
// SimpleDateFormat is NOT thread-safe — use ThreadLocal to make it safe
private static final ThreadLocal<SimpleDateFormat> dateFormat =
    ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd"));
```

**⚠️ Memory Leak Warning:** Always call `remove()` when using ThreadLocal with thread pools. Threads are reused, so old values persist!

---

## 9. Synchronization Utilities

### Semaphore — Control Concurrent Access

```java
// Limits the number of threads accessing a resource simultaneously
// Use case: connection pool, rate limiting, bounded resource access

Semaphore semaphore = new Semaphore(3); // max 3 concurrent threads

public void accessResource() {
    try {
        semaphore.acquire();   // blocks if 3 threads already in
        // access shared resource (DB connection, API, etc.)
        useResource();
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    } finally {
        semaphore.release();   // ALWAYS release in finally!
    }
}

// tryAcquire — non-blocking
if (semaphore.tryAcquire(5, TimeUnit.SECONDS)) {
    try {
        useResource();
    } finally {
        semaphore.release();
    }
} else {
    System.out.println("Resource busy, try later");
}

// Binary Semaphore (permits = 1) ≈ Mutex
// But unlike ReentrantLock, any thread can release (not just the owner)
```

### CountDownLatch — Wait for N Tasks

```java
// One-time use: count goes to 0 and stays there (not reusable)
// Use case: wait for all services to initialize before starting app

CountDownLatch latch = new CountDownLatch(3); // wait for 3 tasks

// Worker threads
for (int i = 0; i < 3; i++) {
    new Thread(() -> {
        try {
            initializeService();    // do some work
        } finally {
            latch.countDown();      // decrement count
        }
    }).start();
}

// Main thread waits
latch.await();                          // blocks until count reaches 0
// OR with timeout:
latch.await(30, TimeUnit.SECONDS);

System.out.println("All services initialized!");
```

**Real-world example:** Microservice health checks — main thread waits until all downstream services respond.

### CyclicBarrier — Threads Wait for Each Other

```java
// Reusable: all threads wait at a barrier point, then proceed together
// Use case: parallel computation where phases must sync

CyclicBarrier barrier = new CyclicBarrier(3, () -> {
    System.out.println("All 3 threads reached barrier — merging results!");
});

for (int i = 0; i < 3; i++) {
    new Thread(() -> {
        try {
            computePartialResult();  // phase 1
            barrier.await();         // wait for all 3 threads

            // All proceed together after barrier
            processNextPhase();      // phase 2
            barrier.await();         // reusable! wait again
        } catch (Exception e) {
            Thread.currentThread().interrupt();
        }
    }).start();
}
```

### CountDownLatch vs CyclicBarrier

| Feature | CountDownLatch | CyclicBarrier |
|---------|---------------|---------------|
| Reusable? | ❌ One-time | ✅ Reusable (auto-resets) |
| Who waits? | One/few threads wait for N events | All N threads wait for each other |
| Count | `countDown()` by any thread | `await()` by each participant |
| Callback | ❌ | ✅ Optional barrier action |
| Use case | "Start when all services ready" | "Sync parallel phases" |

---

## 10. Common Concurrency Problems

### Deadlock
```java
// Thread 1: locks A → waits for B
// Thread 2: locks B → waits for A
// Both stuck forever!

Object lockA = new Object();
Object lockB = new Object();

// Thread 1
synchronized(lockA) {
    synchronized(lockB) { /* work */ }
}

// Thread 2
synchronized(lockB) {    // ← deadlock!
    synchronized(lockA) { /* work */ }
}

// ✅ Fix: Always acquire locks in the SAME ORDER
// Thread 1 & 2 both: lockA → lockB
```

### Deadlock Prevention Strategies
1. **Lock ordering** — always acquire locks in consistent order
2. **tryLock with timeout** — don't wait forever
3. **Lock-free algorithms** — use Atomic classes, CAS
4. **Single lock** — reduce number of locks

### Race Condition
```java
// Check-then-act pattern — NOT atomic
if (!map.containsKey(key)) {  // Thread A checks
    map.put(key, value);       // Thread B might put between check and act
}

// ✅ Fix: Use atomic operations
map.putIfAbsent(key, value);
map.computeIfAbsent(key, k -> createValue());
```

### Starvation
A thread never gets CPU time because higher-priority threads dominate. Fix: use fair locks `new ReentrantLock(true)`.

### Livelock
Threads keep changing state in response to each other but make no progress (like two people trying to pass each other in a corridor).

### Thread Dump Analysis (Production Debugging)

```bash
# Generate thread dump
jstack <PID>                 # basic
jstack -l <PID>              # with lock info
kill -3 <PID>                # send SIGQUIT (output to stdout)

# Using JMX / VisualVM / JConsole for GUI-based analysis
```

**Reading a thread dump:**
```
"http-nio-8080-exec-1" #25 daemon prio=5 tid=0x00007f... nid=0x1a03
   java.lang.Thread.State: BLOCKED (on object monitor)
        at com.example.Service.process(Service.java:45)
        - waiting to lock <0x000000076ab30> (a java.lang.Object)
        - locked <0x000000076ab40> (a java.lang.Object)
```

**What to look for:**
1. **BLOCKED** threads — waiting for a lock held by another thread
2. **WAITING / TIMED_WAITING** — `wait()`, `sleep()`, `park()`
3. **Circular lock dependencies** — Thread A holds Lock1 waiting for Lock2, Thread B holds Lock2 waiting for Lock1 = **deadlock**
4. **Many threads in same state** — possible bottleneck (e.g., DB connection pool exhausted)

---

## 11. Virtual Threads (Java 21)

```java
// Traditional threads = OS threads (heavyweight, ~1MB stack each)
// Virtual threads = lightweight threads managed by JVM (Project Loom)

// Creating Virtual Threads
Thread vThread = Thread.ofVirtual().start(() -> {
    System.out.println("I'm a virtual thread!");
});

// Using ExecutorService (PREFERRED)
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    // Can create MILLIONS of virtual threads!
    for (int i = 0; i < 100_000; i++) {
        executor.submit(() -> {
            // I/O bound work (API calls, DB queries)
            callExternalAPI();
        });
    }
}
```

### Platform Threads vs Virtual Threads

| Feature | Platform (OS) Thread | Virtual Thread |
|---------|---------------------|----------------|
| Managed by | OS | JVM |
| Memory | ~1MB stack each | ~few KB |
| Max count | Thousands (OS limit) | Millions |
| Best for | CPU-bound work | I/O-bound work (REST calls, DB) |
| Blocking cost | Expensive (blocks OS thread) | Cheap (JVM unmounts from carrier) |
| Thread pool needed? | ✅ Yes (to limit threads) | ❌ No (create per task) |

**⚠️ When NOT to use Virtual Threads:**
- CPU-intensive computation (use platform threads / ForkJoinPool)
- When using `synchronized` blocks with I/O inside (pins the carrier thread)
- Use `ReentrantLock` instead of `synchronized` for virtual thread compatibility

```java
// Spring Boot 3.2+ — enable virtual threads
// application.properties:
// spring.threads.virtual.enabled=true
// That's it! All request-handling threads become virtual.
```

---

## 12. Interview Questions & Answers

### Q1: What is the difference between `Thread.start()` and `Thread.run()`?
**A:** `start()` creates a **new thread** and executes `run()` in that thread. Calling `run()` directly executes it in the **current thread** — no new thread is created. Always use `start()`.

---

### Q2: What is a deadlock? How do you detect and prevent it?
**A:** Deadlock = two+ threads waiting for each other's locks, forming a circular wait. **Conditions** (all 4 must hold): Mutual exclusion, Hold-and-wait, No preemption, Circular wait. **Detection:** Thread dumps (`jstack`), JMX, VisualVM. **Prevention:** Lock ordering, tryLock with timeout, reducing lock scope.

---

### Q3: Explain the Java Memory Model (happens-before).
**A:** JMM defines rules for when writes by one thread are visible to reads by another:
1. Program order within a thread
2. `synchronized` block release → subsequent acquire
3. `volatile` write → subsequent read
4. `Thread.start()` → thread's `run()`
5. Thread completion → `join()` return

Without these guarantees, JVM/CPU can reorder instructions for optimization.

---

### Q4: When to use `CompletableFuture` vs `ExecutorService.submit()`?
**A:** 
- `Future` — basic, can only `get()` (blocking) or `cancel()`. No chaining.
- `CompletableFuture` — non-blocking chaining (`thenApply`, `thenCompose`), combining futures (`allOf`, `anyOf`), exception handling (`exceptionally`, `handle`). Use CompletableFuture for modern async programming.

---

### Q5: What is the fork/join framework?
**A:** A framework for parallelizing divide-and-conquer algorithms. Uses work-stealing: idle threads steal tasks from busy threads' queues. `ForkJoinPool` is the default pool for parallel streams. Suitable for recursive tasks that can be split into subtasks.

---

### Q6: `synchronized` method vs `synchronized` block — which is better?
**A:**
- **`synchronized` method:** Locks the entire `this` object (or `Class` object for static). Simple but coarse-grained.
- **`synchronized` block:** Locks only a specific object, can protect specific code sections. More granular, better performance.

```java
// Method — locks 'this' for entire method
public synchronized void update() {
    // everything locked
}

// Block — only critical section is locked
public void update() {
    // non-critical work (no lock)
    synchronized (this) {
        // only critical section locked
    }
    // more non-critical work
}

// Best practice — use a private lock object
private final Object lock = new Object();
public void update() {
    synchronized (lock) { /* ... */ } // external code can't interfere
}
```

---

### Q7: `wait()` vs `sleep()` in depth?
**A:**
| | `wait()` | `sleep()` |
|---|---|---|
| Belongs to | `Object` class | `Thread` class |
| Lock | ✅ **Releases** the lock | ❌ Does NOT release |
| Wake up by | `notify()` / `notifyAll()` | After duration expires |
| Must be in | `synchronized` block | Anywhere |
| Purpose | Inter-thread communication | Pause execution |
| Spurious wakeup | ✅ Possible (use `while` loop) | ❌ No |

**Key gotcha:** Calling `wait()` outside a `synchronized` block throws `IllegalMonitorStateException`.

---

### Q8: What is thread starvation and how to prevent it?
**A:** Starvation occurs when a thread never gets CPU time because other threads monopolize the resource (e.g., higher-priority threads always run first, or a thread never acquires a lock).

**Prevention:**
1. **Fair locks:** `new ReentrantLock(true)` — FIFO ordering for lock acquisition
2. **Avoid priority abuse:** Don't rely on thread priorities (platform-dependent)
3. **Use timeouts:** `tryLock(timeout)` instead of indefinite waiting
4. **Use `Semaphore(permits, true)`** — fair semaphore

---

### Q9: Difference between `Semaphore` and `ReentrantLock`?
**A:**
| | `Semaphore` | `ReentrantLock` |
|---|---|---|
| Permits | Multiple (N concurrent) | Single (mutual exclusion) |
| Owner | No ownership (any thread can release) | Only owner can unlock |
| Reentrant | ❌ No | ✅ Yes (same thread can lock again) |
| Use case | Rate limiting, connection pool | Mutual exclusion, critical section |

A `Semaphore(1)` (binary semaphore) is similar to a lock but has no ownership — any thread can release it.

---

### Q10: `CountDownLatch` vs `CyclicBarrier`?
**A:**
- **CountDownLatch:** Main thread waits for N worker threads to finish. One-time use. Workers call `countDown()`, waiter calls `await()`.
- **CyclicBarrier:** All N threads wait for each other at a sync point. Reusable. Each thread calls `await()`, and they all proceed together.

**Analogy:** CountDownLatch = "Starting gun" (fire when all runners ready). CyclicBarrier = "Checkpoint" (all runners must reach it before any can continue).

---

### Q11: Can we start a thread twice?
**A:** No. Calling `start()` on an already-started thread throws `IllegalThreadStateException`. A thread goes from `NEW` to `RUNNABLE` once, and eventually to `TERMINATED`. It cannot go back. Create a new `Thread` object if you need to run the task again.

---

### Q12: What are Virtual Threads (Java 21)?
**A:** Lightweight threads managed by the JVM (not OS). Key benefits:
- Create millions of them (vs thousands of platform threads)
- Each uses only a few KB (vs ~1MB for platform threads)
- Perfect for I/O-bound workloads (HTTP calls, DB queries)
- When a virtual thread blocks on I/O, the JVM unmounts it from the carrier OS thread and reuses that thread
- In Spring Boot 3.2+: just set `spring.threads.virtual.enabled=true`

**Not suitable for:** CPU-intensive work (no advantage over platform threads).

---

### Q13: How to detect a deadlock in production?
**A:**
1. **Thread dump:** `jstack <PID>` — look for threads in `BLOCKED` state with circular lock dependencies
2. **JMX:** `ThreadMXBean.findDeadlockedThreads()` — programmatic detection
3. **VisualVM / JConsole:** GUI tools that highlight deadlocked threads
4. **Logging:** Monitor for threads stuck beyond expected time
5. **Watchdog:** Implement a health-check that detects if a thread hasn't progressed

---

### Q14: `shutdown()` vs `shutdownNow()`?
**A:**
| | `shutdown()` | `shutdownNow()` |
|---|---|---|
| New tasks | Rejected | Rejected |
| Running tasks | Continue until done | Interrupted (best effort) |
| Queued tasks | Execute remaining | Returned as list (not executed) |
| Return | void | `List<Runnable>` (unexecuted tasks) |

**Best practice:** Call `shutdown()` first, then `awaitTermination()`, then `shutdownNow()` as fallback.

---

### Q15: Why can `ThreadLocal` cause memory leaks in thread pools?
**A:** Thread pool threads are **reused**, not destroyed. If a ThreadLocal value is set during request processing but never `remove()`d:
1. The thread returns to the pool with the value still attached
2. Next request on that thread sees stale data
3. The value (and objects it references) can't be GC'd because the thread's `ThreadLocalMap` holds a strong reference

**Fix:** Always call `threadLocal.remove()` in a `finally` block or use a Servlet `Filter` / Spring `HandlerInterceptor` to clean up after each request.

---

> **Next:** [04_Java8_Plus_Features.md](04_Java8_Plus_Features.md) →
