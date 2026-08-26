# RxJava: The Reactive Extensions Library for the JVM

A comprehensive guide to ReactiveX for Java — covering core concepts, operators, schedulers, subjects, testing, version history, and ecosystem comparisons for architects and senior developers building event-driven, asynchronous systems.

---

## 1. What Is RxJava?

RxJava is a Java implementation of **ReactiveX** (Reactive Extensions), a library for composing asynchronous and event-based programs using **observable sequences**. Originally developed by **Netflix** for its API orchestration layer, it is now maintained by the ReactiveX community.

RxJava extends the **Observer pattern** with functional programming operators (`map`, `filter`, `flatMap`, `reduce`, and hundreds more) to enable **declarative composition of asynchronous operations**. Instead of polling or blocking, you define **data pipelines** that react to events as they happen.

| Problem | RxJava Solution |
|---------|----------------|
| Nested callbacks ("callback hell") | Declarative operator chains |
| Manual thread management | Schedulers abstract threading |
| Hard-to-reason state | Observable sequences + pure transformations |
| Error handling scattered across code | Unified `onError` channel |
| Backpressure absent in most async APIs | `Flowable` with configurable strategies |

### Quick First Impression

```java
// RxJava 3.x with Java 8 lambdas
Observable.just(1, 2, 3, 4, 5)
    .map(i -> i * 2)
    .filter(i -> i > 2)
    .subscribe(System.out::println);  // prints 4, 6, 8, 10
```

This single chain expresses: create a stream of items → double each → keep only those > 2 → print results. No loops, no mutable state, no thread management.

---

## 2. Core Concepts

**Observable** — a source that emits data items over time. Where `Iterable<T>` pulls, `Observable<T>` pushes.

**Observer** — consumer with three callbacks: `onNext(T)` per item, `onError(Throwable)` on failure, `onComplete()` on stream finish. Only one terminal event is ever delivered.

**Disposable** — returned by `subscribe()`; calling `dispose()` cancels the subscription.

**Scheduler** — controls which thread(s) work runs on (see §7).

**Operators** — 500+ functions that transform, filter, combine observable sequences.

**Subject** — both Observable and Observer; bridges imperative→reactive code and multicasts.

**Backpressure** — slow consumer signals fast producer to slow down. `Flowable` supports it; `Observable` does not.

**Lifecycle:** `subscribe() → onSubscribe(Disposable) → onNext(T)* → onComplete()|onError() → disposed`

---

## 3. Observable Types

RxJava provides five core reactive types:

- **Observable\<T\>** — 0..N items, no backpressure. UI events, WebSocket messages.
- **Flowable\<T\>** — 0..N items, backpressure-aware. DB queries, high-frequency data.
- **Single\<T\>** — 1 item or error. HTTP response, DB insert.
- **Maybe\<T\>** — 0 or 1 item or error. Cache lookup.
- **Completable** — no items, only completion/error. DB update, file write.

```java
Observable.just("a", "b");
Flowable.interval(1, MS).onBackpressureDrop();
Single.fromCallable(() -> httpClient.get("/user/42"));
Maybe.fromCallable(() -> cache.get("user:42"));
Completable.fromCallable(() -> { db.update(sql); return null; });
```

| Scenario | Type | Rationale |
|----------|------|-----------|
| Mouse clicks, UI gestures | `Observable` | Bounded rate, no backpressure |
| High-frequency ticks | `Flowable` | Backpressure essential |
| HTTP response | `Single` | Exactly one result |
| Cache lookup | `Maybe` | May or may not be present |
| DB update, no return | `Completable` | Only completion signal |

---

## 4. Creating Observables

```java
Observable.just("a", "b", "c");                             // Wrap items
Observable.fromIterable(Arrays.asList(1, 2, 3));             // From Iterable
Observable.range(10, 5);                                     // 10, 11, 12, 13, 14
Observable.fromCallable(() -> api.fetchData());              // Executed on subscribe
Observable.defer(() -> Observable.just(System.currentTimeMillis()));  // Per-subscriber
Observable.interval(1, SECONDS);                             // 0, 1, 2... every second
Observable.timer(5, SECONDS);                                // 0L after 5 seconds
Observable.empty();                                          // Complete immediately
Observable.error(new IOException("failure"));                // Error immediately
Observable.never();                                          // Never emits

Single.just("result");   Single.fromCallable(() -> compute());
Maybe.just("present");   Maybe.empty();
Completable.fromCallable(() -> { sideEffect(); return null; });
```

Full custom control with `create()`:

```java
Observable<String> custom = Observable.create(emitter -> {
    try {
        emitter.onNext("start");
        for (String line : readFile()) emitter.onNext(line);
        emitter.onComplete();
    } catch (Exception e) { emitter.onError(e); }
});
```

**Best practice:** Prefer `fromCallable`, `fromIterable`, `just`, or `defer` over `create()` unless wrapping callback-based APIs.

---

## 5. Cold vs Hot Observables

**Cold Observable** — starts emitting only on subscribe; each subscriber gets the full sequence independently. *Examples:* `just`, `fromIterable`, HTTP requests.

```java
Observable.just("a", "b").subscribe(s -> log("Sub1: "+s));
Observable.just("a", "b").subscribe(s -> log("Sub2: "+s));  // Both get "a", "b"
```

**Hot Observable** — emits regardless of subscribers; late subscribers miss earlier emissions. *Examples:* Subjects, mouse events, stock ticks, Kafka.

**Cold → Hot conversion:** `publish().connect()` for manual control; `share()` (`publish().refCount()`) for auto-connect on first subscriber.

```java
ConnectableObservable<Long> co = Observable.interval(1, SECONDS).publish();
co.connect();                                              // manual start
Observable<Long> hot = Observable.interval(1, SECONDS).share();  // auto
```

**Multicasting** — `share()` shares one upstream subscription across all downstream consumers:

```java
Observable<Data> shared = api.fetchData().share();
d1 = shared.subscribe(data -> ui1.display(data));
d2 = shared.subscribe(data -> ui2.display(data));  // same network call
```

---

## 6. Operators

### 6.1 Creating Operators

`just`, `fromIterable`, `fromCallable`, `create`, `defer`, `interval`, `range`, `empty`, `error`, `never`.

### 6.2 Transforming Operators

#### `map` — Transform each item

```java
Observable.just("hello", "world").map(s -> s.toUpperCase());
// HELLO / WORLD
```

#### `flatMap` — Map to Observable and flatten (interleaved)

```java
ordersObservable.flatMap(order -> lineItemsObservable(order));
```

#### `concatMap` — flatMap maintaining order

```java
ordersObservable.concatMap(order -> lineItemsObservable(order));
// Each order's items complete before next order's start
```

#### `switchMap` — Switch to latest Observable, cancel previous

```java
searchInput.switchMap(query -> apiSearch(query));
// Cancels previous in-flight API call when new query arrives
```

#### `scan` — Accumulate emissions

```java
Observable.range(1, 5).scan(0, (acc, value) -> acc + value);
// 1, 3, 6, 10, 15
```

#### `buffer` and `window`

```java
Observable.range(1, 10).buffer(3);          // [1,2,3] [4,5,6] [7,8,9] [10]
Observable.interval(100, MS).buffer(1, SECONDS);  // ~10 items/second batches
```

#### `groupBy` — Group by key

```java
Observable.fromIterable(transactions)
    .groupBy(Transaction::getCurrency)
    .flatMapSingle(group -> group
        .reduce(0.0, (sum, tx) -> sum + tx.getAmount())
        .map(total -> group.getKey() + ": " + total));
```

### 6.3 Filtering Operators

```java
events.filter(e -> e.type() == CLICK);          // emit only matching
Observable.range(1, 100).take(3);                // 1, 2, 3
Observable.range(1, 100).takeLast(3);            // 98, 99, 100
Observable.range(1, 5).skip(2);                  // 3, 4, 5
Observable.range(1, 5).skipLast(2);              // 1, 2, 3
Observable.range(1, 5).elementAt(2);             // 3
Observable.range(1, 5).first(0);                 // Single with 1
Observable.range(1, 5).last(0);                  // Single with 5
Observable.just(1,1,2,2,3,1).distinct();         // 1, 2, 3
Observable.just(1,1,2,2,3,1).distinctUntilChanged(); // 1, 2, 3, 1
Observable.range(1, 100).ignoreElements();       // Completable
```

#### Throttling / Rate Limiting

```java
clicks.throttleFirst(500, MS);                   // Click, then ignore 500ms
sensorReadings.sample(1, SECONDS);               // Latest reading each second
searchInput.debounce(300, MS);                   // Wait for 300ms pause in typing
```

### 6.4 Combining Operators

```java
Observable.merge(obs1, obs2);                    // Interleaved
Observable.concat(obs1, obs2);                   // Sequential (first completes)
Observable.zip(names, ages, (n, a) -> n+" is "+a);  // Pairwise by index
Observable.combineLatest(f1, f2, (v1, v2) -> v1+" "+v2);  // Any source emits → combine
mainEvts.withLatestFrom(latestState, (e, s) -> process(e, s));
Observable.range(3, 3).startWith(1, 2);          // 1, 2, 3, 4, 5
```

### 6.5 Conditional Operators

```java
Observable.amb(Arrays.asList(obs1, obs2));           // First to emit wins
Observable.empty().defaultIfEmpty("fallback");      // "fallback"
Observable.just(2,4,6).all(i -> i%2==0);            // true
Observable.just(1,2,3).any(i -> i>2);               // true
Observable.sequenceEqual(obs1, obs2);
ticks.takeUntil(stopSignal);
```

### 6.6 Error Handling Operators

```java
Observable.error(new RuntimeException("fail"))
    .onErrorReturn(e -> -1);                        // -1 on error

api.fetchData()
    .onErrorResumeNext(cache.getCachedData());      // Backup Observable

api.call().retry(3);                                // Retry up to 3 times
api.call().retry(3, e -> e instanceof IOException); // Selective retry

// Exponential backoff retry
api.call().retryWhen(errors -> errors
    .zipWith(Observable.range(1,5), (e, i) -> i)
    .flatMap(a -> Observable.timer((long)Math.pow(2,a), SECONDS)));
// 2s, 4s, 8s, 16s, 32s
```

### 6.7 Utility Operators

```java
Observable.just("delayed").delay(2, SECONDS);
api.fetchData().timeout(5, SECONDS);

Observable.just("a", "b")
    .doOnSubscribe(d -> log("Subscribed"))
    .doOnNext(s -> log("Emitting: " + s))
    .doAfterNext(s -> log("Emitted: " + s))
    .doOnError(e -> log("Error: " + e))
    .doOnComplete(() -> log("Complete"))
    .doFinally(() -> log("Disposed"))
    .subscribe();
```

### 6.8 Complex Pipeline Example

```java
Observable.fromIterable(trades)
    .filter(trade -> trade.getAmount() > 10_000)
    .map(trade -> enrichWithMarketData(trade))
    .flatMap(trade -> validateTrade(trade))
    .subscribeOn(Schedulers.io())
    .observeOn(Schedulers.computation())
    .subscribe(
        trade -> processTrade(trade),
        error -> logError(error),
        () -> logComplete()
    );
```

This pipeline: reads trades → filters high-value → enriches with market data → validates → moves to IO thread for upstream, computation thread for downstream → processes results or errors.

---

## 7. Schedulers and Concurrency

### 7.1 Built-in Schedulers

| Scheduler | Thread Pool | Use Case |
|-----------|-------------|----------|
| `Schedulers.computation()` | Fixed pool = CPU cores | CPU-bound — number crunching |
| `Schedulers.io()` | Elastic (cached, unbounded) | I/O-bound — network, file, DB |
| `Schedulers.newThread()` | New thread per task | Rare one-off operations |
| `Schedulers.single()` | Single thread, sequential | Shared mutable state |
| `Schedulers.trampoline()` | Current thread | Testing |
| `Schedulers.from(Executor)` | Custom `Executor` | Integrate existing pools |
| `AndroidSchedulers.mainThread()` | Android UI thread | Android (via RxAndroid) |

### 7.2 `subscribeOn` vs `observeOn`

- **`subscribeOn`** — controls the thread where upstream (source + operators above it) executes. Only the **first** `subscribeOn` takes effect.
- **`observeOn`** — controls the thread where **all downstream** operators execute from that point. Each `observeOn` creates a thread boundary.

```java
Observable.fromCallable(() -> api.fetchData())
    .subscribeOn(Schedulers.io())           // fetch on IO thread
    .map(data -> process(data))              // still on IO thread
    .observeOn(Schedulers.computation())     // switch to computation
    .filter(processed -> processed.isValid())// on computation thread
    .observeOn(AndroidSchedulers.mainThread()) // switch to main
    .subscribe(result -> updateUI(result));  // on UI thread
```

### 7.3 Threading Patterns

```java
// Background → UI (Android)
Observable.fromCallable(() -> repo.fetchUser(id))
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(user -> textView.setText(user.getName()), this::handleError);

// Parallel computation
Observable.range(1, 100)
    .flatMap(i -> Observable.just(i)
        .subscribeOn(Schedulers.computation())
        .map(HeavyComputation::calculate))
    .subscribe(result -> collect(result));
```

---

## 8. Subjects

A **Subject** is both Observable and Observer — you push items in and downstream consumers observe.

### 8.1 Subject Types

**PublishSubject** — emits to currently subscribed observers. Late subscribers miss earlier emissions.

```java
PublishSubject<String> s = PublishSubject.create();
s.onNext("A");                      // dropped — no subscriber
s.subscribe(v -> log("Sub1: " + v));
s.onNext("B");                      // Sub1 receives B
```
*Use case: Event bus, one-shot notifications.*

**BehaviorSubject** — emits most recent item (or default) to new subscribers.

```java
BehaviorSubject<String> s = BehaviorSubject.createDefault("initial");
s.subscribe(v -> log("Sub1: " + v));  // receives "initial"
s.onNext("A");
s.subscribe(v -> log("Sub2: " + v));  // receives "A" (most recent)
```
*Use case: State management — always provides current state.*

**ReplaySubject** — emits **all** items to new subscribers, regardless of when they subscribe.

```java
ReplaySubject<String> s = ReplaySubject.create();
s.onNext("A"); s.onNext("B");
s.subscribe(v -> log("Sub: " + v));  // receives A, B
```
*Use case: Caching. **Caution:** Can cause memory leaks on long-lived streams.*

**AsyncSubject** — emits only the **last** item after `onComplete()`.

```java
AsyncSubject<String> s = AsyncSubject.create();
s.onNext("A"); s.onNext("B"); s.onNext("C"); s.onComplete();
// Output: C
```

**UnicastSubject** — buffers until subscribed; single subscriber only.

```java
UnicastSubject<String> s = UnicastSubject.create();
s.onNext("buffered1");
s.subscribe(v -> log(v));  // receives buffered1, then live
```

### 8.2 Usage Patterns

```java
// Event bus with PublishSubject
public class EventBus {
    private static final PublishSubject<Object> bus = PublishSubject.create();
    public static void post(Object e) { bus.onNext(e); }
    public static <T> Observable<T> on(Class<T> cls) { return bus.ofType(cls); }
}
EventBus.on(LoginEvent.class).subscribe(e -> navigateToDashboard());

// State management with BehaviorSubject
public class UserState {
    private final BehaviorSubject<User> state = BehaviorSubject.createDefault(ANONYMOUS);
    public Observable<User> observe() { return state.hide(); }
    public void login(User user) { state.onNext(user); }
}
```

**Note:** Subjects are often discouraged in production code in favor of `Observable.create` or `fromEmitter`. They remain useful for bridging legacy callback APIs, but misuse (leaking, no backpressure) is a real risk. When you just need cold→hot conversion, prefer `ConnectableObservable` or `share()`.

---

## 9. Backpressure

### 9.1 The Problem

When an Observable emits faster than the Observer consumes, items pile up in an unbounded buffer → **OutOfMemoryError** or thread starvation.

### 9.2 Flowable — Backpressure-Aware

`Flowable` implements the reactive-streams `Publisher` protocol with backpressure.

```java
// Dangerous — unbounded buffer
Observable.interval(1, MS).subscribe(i -> heavyProcessing(i));
// Safe — backpressure-aware
Flowable.interval(1, MS).onBackpressureDrop().subscribe(i -> heavyProcessing(i));
```

### 9.3 Backpressure Strategies

| Strategy | Behavior | Use Case |
|----------|----------|----------|
| `onBackpressureBuffer()` | Buffer all items (bounded or unbounded) | Consumer occasionally slower |
| `onBackpressureBuffer(cap, onOverflow)` | Bounded buffer + overflow callback | Predictable memory bound |
| `onBackpressureDrop()` | Drop items when downstream busy | Sensor readings, ticks (lossy OK) |
| `onBackpressureLatest()` | Keep only the latest | Real-time dashboards |
| `onBackpressureError()` | Throw `MissingBackpressureException` | Fail-fast |

```java
Flowable.range(1, 1_000_000).onBackpressureBuffer();                     // unbounded
Flowable.range(1, 1_000_000).onBackpressureBuffer(1000, () -> log.warn("overflow"));  // bounded

Flowable.interval(1, MS).onBackpressureDrop()
    .observeOn(Schedulers.io())
    .subscribe(i -> { Thread.sleep(10); System.out.println(i); });       // ~100/sec, rest dropped

Flowable.interval(1, MS).onBackpressureLatest()
    .observeOn(Schedulers.io())
    .subscribe(i -> { Thread.sleep(50); System.out.println(i); });       // always most recent
```

### 9.4 Observable vs Flowable

| Aspect | Observable | Flowable |
|--------|------------|----------|
| Backpressure | No | Yes |
| Use case | Cold/bounded, UI events, small data | Hot/unbounded, DB queries, high-freq data |
| Overhead | Lower | Slightly higher (backpressure protocol) |
| Reactive Streams | No | Yes (implements `Publisher`) |

**Rule of thumb:** Use `Flowable` when the source may exceed ~10,000 items or the emission rate is unbounded. Use `Observable` for UI events and small, bounded sources.

---

## 10. Testing RxJava

### 10.1 TestScheduler — Virtual Time

`TestScheduler` lets you control time deterministically without real wall-clock delays.

```java
@Test
void testDebounce() {
    TestScheduler s = new TestScheduler();
    PublishSubject<String> src = PublishSubject.create();
    TestObserver<String> o = src.debounce(300, MS, s).test();

    src.onNext("A"); src.onNext("B");
    s.advanceTimeBy(200, MS);
    src.onNext("C");
    s.advanceTimeBy(100, MS);
    src.onNext("D");
    s.advanceTimeBy(400, MS);

    o.assertValues("D");  // Only D after 300ms quiet period
}
```

### 10.2 TestObserver / TestSubscriber Assertions

```java
@Test
void testOperatorChain() {
    Observable.just(1, 2, 3, 4, 5)
        .map(i -> i * 2).filter(i -> i > 5)
        .test()
        .assertValues(6, 8, 10)
        .assertValueCount(3)
        .assertComplete()
        .assertNoErrors();
}
```

| Method | Purpose |
|--------|---------|
| `assertValue(T)` / `assertValues(T...)` | Check emitted values |
| `assertValueCount(int)` | Check number of emissions |
| `assertComplete()` / `assertNotComplete()` | Verify completion |
| `assertError(Class)` / `assertErrorMessage(String)` | Verify error |
| `assertNoErrors()` | Verify no errors |
| `assertOf(Consumer<TestObserver>)` | Custom assertion |
| `awaitTerminalEvent()` | Block until terminal event |

### 10.3 Testing Time-Based Operators

```java
@Test void testInterval() {
    TestScheduler s = new TestScheduler();
    TestObserver<Long> o = Observable.interval(1, SECONDS, s).take(3).test();
    s.advanceTimeBy(3, SECONDS);
    o.assertValues(0L, 1L, 2L).assertComplete();
}

@Test void testTimeout() {
    TestScheduler s = new TestScheduler();
    TestObserver<String> o = Observable.<String>never()
        .timeout(5, SECONDS, s).test();
    s.advanceTimeBy(6, SECONDS);
    o.assertError(TimeoutException.class);
}

@Test void testExponentialBackoff() {
    TestScheduler s = new TestScheduler();
    TestObserver<String> o = Observable.error(new IOException())
        .retryWhen(errors -> errors
            .zipWith(Observable.range(1,3), (e,i)->i)
            .flatMap(a -> Observable.timer((long)Math.pow(2,a), SECONDS, s)))
        .test();
    s.advanceTimeBy(14, SECONDS);  // 2+4+8 seconds
    o.assertError(IOException.class);
}
```

### 10.4 Testing Error Handling

```java
@Test void testOnErrorReturn() {
    Observable.error(new RuntimeException("fail"))
        .onErrorReturn(e -> -1)
        .test().assertValue(-1).assertComplete();
}

@Test void testRetry() {
    AtomicInteger c = new AtomicInteger(0);
    Observable<Integer> flaky = Observable.fromCallable(() -> {
        if (c.incrementAndGet() < 3) throw new IOException("attempt "+c);
        return c.get();
    });
    flaky.retry(3).test().assertValue(3).assertComplete();
}
```

---

## 11. RxJava 3.x vs 2.x vs 1.x

### 11.1 Version Timeline

| Version | Released | Status | Package |
|---------|----------|--------|---------|
| RxJava 1.x | 2014 | **Legacy** (EOL) | `rx` |
| RxJava 2.x | 2016 | Maintenance | `io.reactivex` |
| **RxJava 3.x** | **2020** | **Active** | **`io.reactivex.rxjava3`** |

### 11.2 Key Differences: 3.x vs 2.x

| Aspect | RxJava 2.x | RxJava 3.x |
|--------|------------|------------|
| Package | `io.reactivex` | `io.reactivex.rxjava3` |
| Functional interfaces | Custom | Java 8 `io.reactivex.rxjava3.functions` |
| Null safety | Partial | Stricter (NPE on null emissions) |
| Reactive Streams | Compliant | Improved compliance |
| Deprecated APIs | `Func1`, `Action1` | Removed (Java 8 equivalents) |
| `Single.zip` | N/A | Added |
| `TestObserver.assertValue(Predicate)` | Present | Changed to `assertValueAt(0, Predicate)` |

### 11.3 Migration from 2.x to 3.x

Primarily a **package rename**:

```java
// RxJava 2.x                                // RxJava 3.x
import io.reactivex.Observable;              import io.reactivex.rxjava3.core.Observable;
import io.reactivex.Single;                  import io.reactivex.rxjava3.core.Single;
import io.reactivex.schedulers.Schedulers;    import io.reactivex.rxjava3.schedulers.Schedulers;
import io.reactivex.observers.TestObserver;   import io.reactivex.rxjava3.observers.TestObserver;
```

**Breaking changes:** Stricter null rejection; `TestObserver.assertValue(Predicate)` → `assertValueAt(0, Predicate)`; some deprecated operators removed.

### 11.4 Library Compatibility

Check these for RxJava 3.x support before migrating:
- **Retrofit:** `com.squareup.retrofit2:adapter-rxjava3`
- **RxAndroid:** `io.reactivex.rxjava3:rxandroid`
- **Vert.x:** `io.vertx:vertx-rx-java3`

**Recommendation:** Start new projects on RxJava 3.x. Migrate existing 2.x when ecosystem supports it.

---

## 12. RxJava in Practice

### 12.1 Android

RxJava is the dominant reactive library on Android. Key integrations:

| Library | Purpose | RxJava Type |
|---------|---------|-------------|
| **Retrofit** | HTTP client | `Single`, `Observable` |
| **Room** | Local DB | `Flowable`, `Maybe`, `Single` |
| **RxBinding** | UI events | `Observable<ViewClick>` |
| **RxLifecycle** / **AutoDispose** | Lifecycle | Auto-dispose on destroy |

```java
// Retrofit + RxJava Android
disposable = apiService.getUser(userId)
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(user -> textView.setText(user.getName()), this::showError);

// Auto-dispose on Activity destroy
api.getUser(id).compose(bindToLifecycle()).subscribe(...);
```

### 12.2 Server-Side

RxJava integrates with Vert.x, Apache Camel, RxNetty, Ratpack, and Retrofit for service calls.

```java
// Vert.x + RxJava
router.get("/api/trades/:id").handler(ctx -> {
    tradeService.getTrade(ctx.request().getParam("id"))
        .subscribeOn(Schedulers.io())
        .subscribe(
            trade -> ctx.response().end(Json.encode(trade)),
            error -> ctx.fail(500, error));
});
```

### 12.3 Banking / Trading Use Cases

For a banking context, RxJava excels at:

- **Trade processing pipelines** — filter, enrich, validate, persist
- **Market data feeds** — high-frequency price streams with backpressure
- **Event-driven risk calculations** — react to position changes
- **API orchestration** — compose multiple downstream service calls

```java
Observable.fromIterable(blotter.getNewTrades())
    .filter(trade -> !trade.isCancelled())
    .flatMap(trade -> Observable.zip(
        enrichWithMarketData(trade),
        checkCreditLimit(trade),
        validateCompliance(trade),
        (enriched, creditOk, complianceOk) -> {
            if (!creditOk || !complianceOk) return TradeResult.rejected(trade);
            return TradeResult.accepted(enriched);
        }
    ))
    .subscribeOn(Schedulers.io())
    .observeOn(Schedulers.computation())
    .subscribe(this::publishResult, this::escalateToOps, () -> log("Batch complete"));
```

---

## 13. RxJava vs Other Reactive Frameworks

### 13.1 Reactive Streams Standard

**Reactive Streams** (reactive-streams.org) standardizes async stream processing with non-blocking backpressure via four interfaces: `Publisher`, `Subscriber`, `Subscription`, `Processor`. Java 9+ includes `java.util.concurrent.Flow`. RxJava 3.x `Flowable` implements `Flow.Publisher`.

```java
// RxJava ↔ Java Flow interop
Flow.Publisher<Integer> javaP = Flowable.range(1, 10).toFlowPublisher();
Flowable<Integer> rxF = Flowable.fromPublisher(javaFlowPublisher);
```

### 13.2 RxJava vs Project Reactor

| Aspect | RxJava 3.x | Project Reactor 3.x |
|--------|-----------|---------------------|
| Created by | Netflix → ReactiveX | Pivotal/VMware (Spring) |
| Reactive types | `Observable`, `Flowable`, `Single`, `Maybe`, `Completable` | `Flux<T>` (≈Flowable), `Mono<T>` (≈Single/Maybe) |
| Operators | 500+ | ~350+ |
| Backpressure | Flowable (explicit strategy) | Built into Flux |
| Schedulers | `io`, `computation`, `newThread`, `single`, `trampoline` | `boundedElastic`, `parallel`, `single`, `immediate` |
| Context propagation | Manual | Built-in `Context` API (tracing) |
| Spring integration | Works alongside | **Native** WebFlux, RSocket |
| Android | Dominant | Limited |
| Testing | `TestScheduler`, `TestObserver` | `StepVerifier`, `TestPublisher` |

**FlatMap variants:**

| Semantics | RxJava | Reactor |
|-----------|--------|---------|
| Interleaved | `flatMap` | `flatMap` |
| Ordered | `concatMap` | `concatMap` |
| Latest (cancel previous) | `switchMap` | `switchMap` |
| Sequential, buffer concurrent | — | `flatMapSequential` |

**Choose RxJava for:** Android apps, existing RxJava codebase, non-Spring servers (Vert.x, Ratpack), largest operator library.
**Choose Reactor for:** New Spring Boot/WebFlux projects, Spring Data Reactive, distributed tracing with context propagation, Micrometer metrics.

### 13.3 RxJava vs Kotlin Coroutines / Flow

| Aspect | RxJava | Kotlin Coroutines + Flow |
|--------|--------|-------------------------|
| Language | Any JVM language | Kotlin only |
| Asynchrony | Observable chains + Schedulers | `suspend` + `Flow` |
| API surface | 500+ operators | Smaller, simpler |
| Learning curve | Steep | Moderate |
| Backpressure | Flowable + explicit strategies | `Flow.buffer/conflate/collectLatest` |
| Structured concurrency | Manual (Disposables) | Built-in via `coroutineScope` |
| Cancellation | `dispose()` | Coroutine scope cancellation |
| Testing | `TestScheduler` | `kotlinx-coroutines-test` |
| Android | Historically dominant | Preferred for new Kotlin projects |

```kotlin
// RxJava
api.fetchData().subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe({ data -> updateUI(data) }, { error -> handle(error) })

// Kotlin coroutines
viewModelScope.launch {
    val data = withContext(Dispatchers.IO) { api.fetchData() }
    updateUI(data)
}
```

**Use Kotlin Flow when:** New Kotlin-only projects, already using coroutines, simpler model preferred, Jetpack Compose + StateFlow.
**Use RxJava when:** Android legacy with existing RxJava codebase, teams familiar with Rx operators, Java-only projects.

### 13.4 RxJava vs Java 9 Flow API

| Aspect | RxJava | Java Flow API |
|--------|--------|---------------|
| Standard | Library | JDK (Java 9+) |
| Types | 5 reactive types | 4 interfaces (bare) |
| Operators | 500+ | None |
| Schedulers | Multiple built-in | None (use `ExecutorService`) |
| Use case | Full reactive programming | Interop compatibility layer |

**Conclusion:** Java Flow is a compatibility standard, not a programming model. Use RxJava or Reactor for real work; use Java Flow adapters for interop.

---

## 14. When to Use RxJava

### 14.1 Good Fits

- **Event-driven systems** — UI events, market data feeds, sensor data
- **API orchestration** — compose multiple async services
- **Data pipelines** — transform, filter, enrich data streams
- **Caching layers** — `Maybe` for cache-then-network patterns
- **High-frequency data** — `Flowable` with backpressure for ticks, logs, metrics
- **Complex UI interactions** — debounced search, throttled clicks, drag-and-drop

### 14.2 Poor Fits

- Simple CRUD applications (RxJava overhead > benefit)
- Small codebase with one async op (use `CompletableFuture`)
- New Kotlin projects (prefer coroutines + Flow)
- Spring WebFlux projects (Reactor is native)
- Teams unfamiliar with functional programming (learning curve is steep)

### 14.3 Migration Strategy

1. **Start small** — Wrap a blocking service call in `Single.fromCallable` + `subscribeOn(Schedulers.io())`
2. **Compose gradually** — Introduce `zip` and `flatMap` for multiple calls
3. **Add backpressure** — Switch to `Flowable` on unbounded streams
4. **Test thoroughly** — Use `TestScheduler` and `TestObserver` for every pipeline
5. **Monitor** — Instrument with `doOnSubscribe`/`doFinally` for latency tracking

---

## 15. Further Reading and Resources

### Official Documentation

- [ReactiveX Documentation](http://reactivex.io/)
- [RxJava 3.x GitHub](https://github.com/ReactiveX/RxJava)
- [RxJava 3.x Javadoc](http://reactivex.io/RxJava/3.x/javadoc/)

### Reactive Streams

- [Reactive Streams Specification](https://www.reactive-streams.org/)
- [Java 9 Flow API](https://docs.oracle.com/javase/9/docs/api/java/util/concurrent/Flow.html)

### Books

- *Learning RxJava* by Thomas Nield (O'Reilly)
- *Reactive Programming with RxJava* by Tomasz Nurkiewicz and Ben Christensen (O'Reilly)
- *RxJava for Android App Development* by K. Venkatesh

### Key Libraries

| Library | Artifact |
|---------|----------|
| **RxJava 3** | `io.reactivex.rxjava3:rxjava` |
| **RxAndroid** | `io.reactivex.rxjava3:rxandroid` |
| **RxKotlin** | `io.reactivex.rxjava3:rxkotlin` |
| **Retrofit RxJava** | `com.squareup.retrofit2:adapter-rxjava3` |
| **AutoDispose** | `com.uber.autodispose2:autodispose` |
| **Vert.x RxJava** | `io.vertx:vertx-rx-java3` |

---

*RxJava brings the Reactive Extensions programming model to the JVM with over 500 operators, backpressure support, flexible threading, and a mature ecosystem spanning Android and server-side Java. While Kotlin coroutines and Project Reactor have gained ground, RxJava remains the most comprehensive reactive library for Java, and understanding its concepts transfers directly to other ReactiveX implementations and the broader reactive programming paradigm.*
