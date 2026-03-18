# Problem Bank: Memory Leak Debugging

## Problem 1: Unbounded Cache Without Eviction

### Problem Statement
A Java order-processing service has its memory grow by 1GB per hour. The GC runs frequently but reclaims less each cycle. A new "event deduplication" feature was deployed 2 weeks ago. The service processes 500 events/second. Diagnose and fix the memory leak.

### Solution Walkthrough

**Initial Clues:**
- Memory growth: linear, 1GB/hour
- GC log: Full GC every 15 minutes, reclaiming only 200MB (was reclaiming 1.5GB before the new feature)
- New feature: event deduplication cache using `HashMap<String, EventCacheEntry>`

**Root Cause:**
The deduplication cache stores every event ID to detect duplicates. It uses a plain `HashMap` with no size limit and no eviction policy. At 500 events/second, the cache grows by 1.8M entries/hour. Each entry is ~500 bytes (key + value + HashMap overhead), so growth is ~900MB/hour -- matching the observed 1GB/hour.

**Diagnosis Steps:**
1. Take heap dump: `jmap -dump:live,format=b,file=heap.hprof <pid>`
2. Open in Eclipse MAT, run "Leak Suspects" report
3. Top suspect: `HashMap` instance with 7.2M entries, retaining 3.6GB
4. Entries are `EventCacheEntry` objects with `eventId` keys
5. Check code: `deduplicationCache.put(event.getId(), new EventCacheEntry(...))` -- no `remove()` anywhere
6. Check cache size over time:
   ```
   T=0h: 0 entries
   T=1h: 1,800,000 entries
   T=4h: 7,200,000 entries
   T=8h: 14,400,000 entries -> OOM
   ```

**Fix:**
```java
// Before (broken):
private final Map<String, EventCacheEntry> cache = new HashMap<>();

// After (fixed):
private final Cache<String, EventCacheEntry> cache = Caffeine.newBuilder()
    .maximumSize(100_000)
    .expireAfterWrite(Duration.ofMinutes(5))
    .build();
```

**Prevention:**
- Code review rule: every `new HashMap()` used as a cache must justify why it has no size limit
- Add memory usage metric: export cache size via Prometheus, alert if > expected max
- Load testing must run for at least 2 hours to catch linear leaks
- Use `WeakHashMap` or `SoftReference` for caches that should yield to GC pressure

### Follow-up Questions
- "What's the difference between `maximumSize` and `expireAfterWrite`? When would you use each?"
- "What happens to the deduplicated events when they're evicted? Could you process a duplicate?"
- "How would you size the cache? What's the tradeoff between size and deduplication effectiveness?"

---

## Problem 2: Event Listener Not Unregistered

### Problem Statement
A service that processes orders registers an event listener for each order to track status updates. Memory grows by 500MB/hour. The heap dump shows 200,000+ `OrderEventListener` objects, each holding a reference to a 50KB `OrderContext`. The order processing rate is 100 orders/second. Diagnose and fix.

### Solution Walkthrough

**Initial Clues:**
- 200,000 listeners in memory (should be ~100 active orders at any time)
- Each listener retains a 50KB `OrderContext`: 200,000 * 50KB = 10GB potential retention
- Listeners are added in `processOrder()` method
- Expected lifecycle: register when order starts, unregister when order completes

**Root Cause:**
The listener is registered at the start of order processing and unregistered in the `onSuccess()` callback. But the `onError()` and `onTimeout()` paths don't unregister the listener. 15% of orders fail or time out, meaning 15 listeners/second accumulate without cleanup. Over 8 hours: 15 * 3600 * 8 = 432,000 leaked listeners.

**Diagnosis Steps:**
1. Compare heap dumps: T=0 has 50 listeners, T=1h has 54,100 listeners
2. Growth rate: ~54,000/hour = 15/second -- matches the failure rate (15% of 100/sec)
3. Check code:
   ```java
   void processOrder(Order order) {
       OrderEventListener listener = new OrderEventListener(order.getContext());
       eventBus.register(listener);  // Registered here

       orderService.submit(order)
           .thenAccept(result -> {
               eventBus.unregister(listener);  // Unregistered on success
               // ...
           });
       // No unregister on failure or timeout!
   }
   ```
4. Check failure rate: 15% of orders fail -> 15 leaked listeners/second

**Fix:**
```java
void processOrder(Order order) {
    OrderEventListener listener = new OrderEventListener(order.getContext());
    eventBus.register(listener);

    orderService.submit(order)
        .whenComplete((result, error) -> {
            eventBus.unregister(listener);  // Always unregister
            if (error != null) {
                handleError(order, error);
            } else {
                handleSuccess(order, result);
            }
        })
        .orTimeout(30, TimeUnit.SECONDS);
}
```

**Prevention:**
- Use `try-with-resources` or `AutoCloseable` pattern for listener lifecycle management
- Unit test: verify listener count before and after processing (including failure paths)
- Add metric: `eventBus.getListenerCount()` exported to Prometheus, alert if > 1000
- Code review checklist: "If you register X, where is X unregistered on ALL paths?"

### Follow-up Questions
- "What if you can't modify the eventBus to support `getListenerCount()`?"
- "Would `WeakReference` solve this? What are the tradeoffs?"
- "How would you write a unit test that catches this leak?"

---

## Problem 3: Closure-Retained HTTP Response Bodies

### Problem Statement
An async HTTP client service sends requests and processes responses via callbacks. Memory grows by 2GB/hour. The heap dump shows thousands of `byte[]` arrays (1-5MB each) retained by lambda objects. The service makes 50 HTTP requests/second, and 5% of them time out. Diagnose and fix.

### Solution Walkthrough

**Initial Clues:**
- Heap dump: 5,000 `byte[]` arrays averaging 3MB each, retained by `lambda` instances
- The `byte[]` arrays contain raw HTTP response bodies
- Retained size: 5,000 * 3MB = 15GB (but only 2GB visible -- many are pending GC)
- Timeout rate: 5% of 50 req/s = 2.5 timeouts/second

**Root Cause:**
The async HTTP callback captures the entire `HttpResponse` object (including the body `byte[]`) in its closure. When a request times out, the `CompletableFuture` is abandoned but never explicitly cancelled. The lambda and its captured response reference remain in memory because the future's reference chain prevents garbage collection.

**Diagnosis Steps:**
1. Eclipse MAT: "Path to GC Roots" for a retained `byte[]`
   ```
   byte[] <- HttpResponse.body <- lambda$processResponse$1
          <- CompletableFuture.stack <- ForkJoinPool.workQueue
   ```
2. The `CompletableFuture` is reachable from the thread pool's work queue
3. Check code:
   ```java
   httpClient.sendAsync(request)
       .thenApply(response -> {
           // Lambda captures 'response' which includes the body byte[]
           processResponse(response.body());
           return response.statusCode();
       });
   // No timeout, no cancellation on timeout
   ```
4. Timed-out futures pile up in the ForkJoinPool with their captured response bodies

**Fix:**
```java
httpClient.sendAsync(request)
    .orTimeout(5, TimeUnit.SECONDS)
    .thenApply(response -> {
        byte[] body = response.body();
        int statusCode = response.statusCode();
        // Process immediately, don't retain the full response
        processResponse(body);
        return statusCode;
    })
    .exceptionally(error -> {
        if (error instanceof TimeoutException) {
            log.warn("Request timed out: {}", request.uri());
        }
        return -1;
    });
```

**Prevention:**
- Always use `.orTimeout()` on `CompletableFuture` chains
- Extract needed data early in the callback chain; don't pass large objects through
- Add memory pressure tests: run service under load for 2 hours, verify RSS stays bounded
- Monitor `CompletableFuture` completion rates: alert if incomplete futures > threshold
- In Python: use `asyncio.wait_for()` with timeout, avoid capturing large objects in coroutines

### Follow-up Questions
- "What's the difference between `.orTimeout()` and `.completeOnTimeout()`?"
- "How would you diagnose this in Python with `asyncio`?"
- "Could you use `WeakReference` in the lambda? What would the tradeoff be?"

---

## Sample Session Flow

### Opening
**Interviewer**: "Hey. I've been staring at memory graphs all morning, and this service is driving me crazy. It just keeps growing. I've profiled a lot of services, and I want to see how you approach this. Ready?"

### Phase 1: Symptom (5 minutes)
**Interviewer**: "This order-processor service eats 1GB of RAM per hour. Every 8 hours, the container gets OOM-killed and restarts. The team just increases the memory limit, but that's not a fix -- it just delays the crash. Where do you start?"

### Phase 2: Profiling (15 minutes)
*[Candidate asks for heap dump]*
**Interviewer**: "Good. Here's the heap histogram from two different timestamps..."
*[Show comparison table]*
**Interviewer**: "What jumps out at you? What would you investigate next?"

### Phase 3: Root Cause (15 minutes)
*[Candidate identifies the suspect class]*
**Interviewer**: "You think it's the EventCacheEntry. Walk me through the proof. I don't want 'I think' -- I want 'here's the evidence.'"

### Phase 4: Fix (10 minutes)
**Interviewer**: "Fix the leak. And how do we make sure nobody introduces this kind of leak again?"

*[Generate scorecard]*
