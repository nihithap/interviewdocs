# Java Interview Handbook
### Version History (Java 1.0 → Java 25) + Most-Asked Coding Questions

---

## Table of Contents

**Part 1 — Java Version-by-Version Feature Guide**
1. The Early Era: Java 1.0 – 1.4
2. Java 5 (Tiger) — the biggest early overhaul
3. Java 6 – 7
4. Java 8 (LTS) — the modern-Java turning point
5. Java 9 – 10
6. Java 11 (LTS)
7. Java 12 – 16
8. Java 17 (LTS)
9. Java 18 – 20
10. Java 21 (LTS)
11. Java 22 – 24
12. Java 25 (LTS)
13. Version Cheat Sheet (LTS timeline + "which version introduced X")

**Part 2 — Most-Asked Coding Interview Questions**
14. Strings
15. Arrays
16. Linked Lists
17. Sorting & Searching
18. Recursion & DP
19. Collections & HashMap Internals
20. Multithreading & Concurrency
21. Streams & Functional Java
22. OOP / Design Questions

---

# PART 1: JAVA VERSION-BY-VERSION FEATURE GUIDE

## 1. The Early Era: Java 1.0 – 1.4

**Q1: What did Java 1.0 (1996) and 1.1 (1997) introduce?**

A: **1.0** shipped the core language, the AWT GUI toolkit, applets, and the original object model. **1.1** added the **inner classes**, **JDBC** (database connectivity), **RMI** (Remote Method Invocation), **JavaBeans**, a completely redesigned **AWT event model** (delegation-based, replacing the clunky 1.0 event-inheritance model), and internationalization support.

**Q2: What was significant about Java 1.2 (1998, marketed as "Java 2")?**

A: Java 1.2 introduced the **Collections Framework** (`List`, `Set`, `Map`, `ArrayList`, `HashMap`, etc. — before this, developers used `Vector`/`Hashtable`/arrays with no unified API), **Swing** (a richer, pure-Java GUI toolkit replacing heavy reliance on AWT), the **JIT compiler** becoming standard for performance, and the `strictfp` keyword. This release was big enough that Sun rebranded the platform "J2SE."

**Q3: What did Java 1.3 and 1.4 add?**

A: **1.3** (2000) added **HotSpot JVM** as default, `java.math` (BigInteger/BigDecimal improvements), and JNDI. **1.4** (2002) was a major release: added **assertions** (`assert` keyword), **NIO** (New I/O — non-blocking I/O, buffers, channels), **regular expressions** (`java.util.regex`), **exception chaining** (`Throwable.getCause()`), the **logging API** (`java.util.logging`), and `StringBuffer` improvements. 1.4 was the **last version without generics or autoboxing** — a common interview trivia point.

---

## 2. Java 5 (Tiger, 2004) — the biggest early overhaul

**Q1: What are the headline features of Java 5, and why is it considered a landmark release?**

A: Java 5 fundamentally changed how Java code is written:
- **Generics** — compile-time type safety for collections (`List<String>` instead of raw `List` with casts).
- **Enhanced for-loop** (`for (String s : list)`).
- **Autoboxing/unboxing** — automatic conversion between primitives and wrapper classes (`Integer i = 5;`).
- **Enums** (`enum`) — type-safe, full-featured (can have fields/methods), replacing the old "public static final int" constant pattern.
- **Varargs** (`void foo(String... args)`).
- **Annotations** (`@Override`, `@Deprecated`, and the ability to define custom annotations) — foundation for later frameworks (Spring, Hibernate) to rely heavily on.
- **Static imports** (`import static`).
- **`java.util.concurrent`** — the entire modern concurrency package: `ExecutorService`, `ConcurrentHashMap`, `CountDownLatch`, `Semaphore`, locks (`ReentrantLock`), atomic variables (`AtomicInteger`), thread-safe queues.
- **`StringBuilder`** — a non-synchronized, faster alternative to `StringBuffer`.

**Q2: Why does interviewers often ask "what's the difference between `Integer` autoboxing caching and `==`" in the context of Java 5?**

A: Autoboxing (Java 5+) caches `Integer` objects for values **-128 to 127** (`Integer.valueOf()` uses an internal cache pool). So `Integer a = 100; Integer b = 100; a == b` is `true` (same cached object), but `Integer a = 200; Integer b = 200; a == b` is `false` (different objects, outside the cache range) — a classic gotcha testing whether candidates understand autoboxing is syntactic sugar over `Integer.valueOf()`, and that `==` compares references, not values, for boxed types. Always use `.equals()` for value comparison.

---

## 3. Java 6 (2006) – Java 7 (2011)

**Q1: What did Java 6 add?**

A: Mostly performance and tooling improvements rather than language changes: scripting engine support (`javax.script`, e.g., running JavaScript via Rhino), improvements to `java.util.concurrent`, JDBC 4.0, JAX-WS for web services, and general JVM performance/monitoring tooling (`jconsole`, compiler/GC improvements). No major new language syntax.

**Q2: What are Java 7's key features (Project Coin)?**

A: Java 7 bundled a set of small syntax improvements known as "Project Coin," plus one big infrastructure change:
- **Diamond operator** (`Map<String, List<Integer>> m = new HashMap<>();` — no need to repeat generic types on the right side).
- **Try-with-resources** — automatic closing of `AutoCloseable` resources (`try (FileReader fr = new FileReader(f)) {...}`), removing manual `finally { close(); }` boilerplate.
- **Multi-catch** (`catch (IOException | SQLException e)`).
- **Strings in switch statements**.
- **Underscore in numeric literals** (`1_000_000` for readability).
- **Binary literals** (`0b1010`).
- **NIO.2** (`java.nio.file` — `Path`, `Files`, `WatchService` for filesystem event monitoring).
- **Fork/Join framework** (`ForkJoinPool`) — foundation for parallel streams later in Java 8.

---

## 4. Java 8 (LTS, 2014) — the modern-Java turning point

**Q1: Why is Java 8 considered the most important release after Java 5, and what are its headline features?**

A: Java 8 introduced **functional programming** concepts into Java:
- **Lambda expressions** — `(a, b) -> a + b`, enabling concise functional-style code.
- **Functional interfaces** (`java.util.function`) — `Function`, `Predicate`, `Supplier`, `Consumer`, `BiFunction`, etc., single-abstract-method interfaces lambdas can implement.
- **Stream API** (`java.util.stream`) — declarative, pipeline-based processing of collections (`filter`, `map`, `reduce`, `collect`), with a **parallel** variant (`parallelStream()`) built on Fork/Join.
- **Default and static methods in interfaces** — interfaces can now provide method bodies, enabling API evolution without breaking existing implementers (this is how `Stream`/`Collection` methods like `forEach` were retrofitted without breaking backward compatibility).
- **`Optional<T>`** — an explicit container for "value or absence," reducing `NullPointerException`s from ambiguous return values.
- **New Date/Time API** (`java.time` — `LocalDate`, `LocalDateTime`, `ZonedDateTime`, `Duration`, `Period`) — replacing the notoriously flawed, mutable, non-thread-safe `java.util.Date`/`Calendar`.
- **Method references** (`String::toUpperCase`, `System.out::println`).
- **`CompletableFuture`** — composable async programming.
- **Removal of PermGen** — replaced by **Metaspace** (native memory instead of a fixed-size heap region), fixing a common `OutOfMemoryError: PermGen space` class of problems.
- **Nashorn** JavaScript engine (since removed in Java 15).

**Q2: What's the difference between `map()` and `flatMap()` in the Stream API?**

A: `map()` transforms each element **1-to-1** (`Stream<T> → Stream<R>`). `flatMap()` transforms each element into a **stream of elements**, then flattens all those inner streams into a single stream — used when each input maps to **zero or more** outputs (e.g., a `Stream<List<String>>` of word-lists flattened into a single `Stream<String>` of all words):
```java
List<List<String>> nested = List.of(List.of("a","b"), List.of("c"));
List<String> flat = nested.stream()
    .flatMap(List::stream)
    .collect(Collectors.toList()); // [a, b, c]
```

**Q3: What's the difference between intermediate and terminal stream operations, and why does laziness matter?**

A: **Intermediate operations** (`filter`, `map`, `sorted`, `distinct`) are **lazy** — they just build up a pipeline description and don't actually execute until a terminal operation is invoked. **Terminal operations** (`collect`, `forEach`, `reduce`, `count`) trigger the actual traversal/execution. Laziness matters because it enables **short-circuiting** (e.g., `findFirst()` can stop early without processing the whole stream) and lets the JVM optimize the whole pipeline as a single pass rather than materializing intermediate collections at every step. A stream can also only be **consumed once** — reusing a stream after a terminal op throws `IllegalStateException`.

**Q4: Why was `Optional` added, and what's a common misuse of it?**

A: `Optional<T>` makes "this might not have a value" **explicit in the type system**, forcing callers to consciously handle absence (`orElse`, `orElseGet`, `orElseThrow`, `ifPresent`) rather than accidentally dereferencing a `null`. Common misuses interviewers probe for: using `Optional` as a **field type** or **method parameter** (it's designed for **return types** only — Optional fields add serialization/boilerplate overhead for no benefit), and calling `.get()` without checking `.isPresent()` first (defeats the entire purpose — just as unsafe as an unchecked null dereference).

---

## 5. Java 9 (2017) – Java 10 (2018)

**Q1: What is the Java Platform Module System (JPMS), introduced in Java 9, and what problem does it solve?**

A: JPMS ("Project Jigsaw") introduced **modules** (`module-info.java`) — a way to explicitly declare a module's dependencies (`requires`) and what packages it exposes (`exports`), enforced at compile and runtime. It solves the historical **"JAR hell"/classpath problem**: prior to modules, all JARs on the classpath were fully visible to each other regardless of intent, internal APIs (like `sun.misc.*`) could be accessed by anyone, and there was no reliable way to know a library's true dependencies. Modules also enabled `jlink` — building a **custom, minimal runtime image** containing only the modules an application actually needs, instead of shipping a full JRE.

**Q2: What other notable Java 9 features come up in interviews?**

A: **`try`-with-resources improvement** (can use effectively-final variables declared outside the try without re-declaring them inside). **Collection factory methods** (`List.of()`, `Set.of()`, `Map.of()` — create **immutable** collections concisely). **Private methods in interfaces** (to share code between default methods without exposing it). **JShell** — Java's first official **REPL** (Read-Eval-Print Loop) for quick experimentation. **Stream API additions**: `takeWhile`, `dropWhile`, `iterate` with a predicate.

**Q3: What did Java 10 introduce?**

A: **`var`** — local variable type inference (`var list = new ArrayList<String>();` — the compiler infers the type at compile time; `var` is **not** dynamic typing, the variable is still statically typed, just without an explicit annotation). Also **Application Class-Data Sharing (AppCDS)** for faster startup, and a switch to a **6-month release cadence** starting from this version (a major process change — before Java 9, releases had no fixed schedule and often took years).

---

## 6. Java 11 (LTS, 2018)

**Q1: What are Java 11's key features?**

A: Java 11 was the first LTS after Java 8, and consolidated several changes:
- **New `String` methods**: `isBlank()`, `strip()`/`stripLeading()`/`stripTrailing()` (Unicode-aware trim), `lines()` (splits into a `Stream<String>` by line), `repeat(n)`.
- **`var` in lambda parameters** (`(var x, var y) -> x + y`) — mainly useful for adding annotations to inferred lambda params.
- **New `HttpClient`** (`java.net.http`) — standardized, promoted from Java 9's incubator; supports HTTP/2 and both sync and async (`CompletableFuture`-based) requests, replacing the old clunky `HttpURLConnection`.
- **Removal of Java EE and CORBA modules** from the JDK (developers needing them had to add them as external dependencies).
- **Single-file source-code launching** — `java HelloWorld.java` runs a `.java` file directly without a separate explicit `javac` compile step (useful for scripts/quick tests).
- **Flight Recorder (JFR)** open-sourced and made available in OpenJDK (was previously a commercial-only feature).

**Q2: Why did Java 11 matter so much for enterprise adoption, and what was the "Java 8 vs 11" migration pain point commonly asked about?**

A: Java 11 was the first LTS release under Oracle's new release model, and importantly, **Oracle JDK became a paid/subscription product for production use starting with 11** (Java 8's public updates being free for longer created a huge lag in enterprise migration) — many companies switched to **OpenJDK distributions** (Adoptium/Temurin, Amazon Corretto, Azul Zulu) instead of Oracle JDK to stay on free updates. The other pain point: the **removal of Java EE modules** (`javax.xml.bind`/JAXB, `javax.activation`, etc.) broke applications that implicitly relied on them being bundled, requiring explicit new dependencies to be added.

---

## 7. Java 12 – 16 (2019 – 2021, all non-LTS except where noted)

**Q1: What did Java 12 and 13 add?**

A: **Java 12**: **Switch expressions (preview)** — `switch` as an expression returning a value, with the arrow syntax (`case X -> ...`); Shenandoah low-pause GC (experimental). **Java 13**: **Text blocks (preview)** — multi-line string literals using `"""` delimiters, avoiding manual `\n` concatenation; refined switch expressions with `yield`.

**Q2: What did Java 14 finalize/preview?**

A: **Switch expressions finalized** (no longer preview). **Records (preview)** — a compact way to declare immutable data-carrier classes (`record Point(int x, int y) {}`) auto-generating constructor, accessors, `equals()`, `hashCode()`, `toString()`. **Pattern matching for `instanceof` (preview)** — `if (obj instanceof String s)` binds `s` directly, removing the redundant explicit cast. **Helpful NullPointerExceptions** — JVM now reports precisely which variable was null in a chained call (e.g., `a.b.c` failing tells you specifically `b` was null), a big debugging quality-of-life improvement.

**Q3: What did Java 15 and 16 finalize?**

A: **Java 15**: **Text blocks finalized**; **Sealed classes (preview)** — restrict which classes can extend/implement a class/interface (`sealed class Shape permits Circle, Square {}`); Nashorn JS engine removed. **Java 16**: **Records finalized**; **Pattern matching for `instanceof` finalized**; **`jpackage`** tool for packaging self-contained Java applications as native installers; strong encapsulation of JDK internals by default (`--illegal-access` no longer available to bypass module boundaries as easily) — this tightened JPMS enforcement significantly.

---

## 8. Java 17 (LTS, 2021)

**Q1: What are Java 17's headline, finalized features (a very common "what's new since 8/11" interview question)?**

A: Java 17 finalized several previously-preview features from 14–16, making it the next big LTS milestone:
- **Sealed classes** (finalized) — `permits` clause restricts the type hierarchy, enabling exhaustive pattern matching (the compiler knows all possible subtypes) and more expressive domain modeling.
- **Records** (finalized, from 16) — immutable data carriers with compact syntax.
- **Pattern matching for `instanceof`** (finalized, from 16).
- **Enhanced pseudo-random number generators** — new `RandomGenerator` interface with more algorithm implementations.
- **Strong encapsulation of JDK internals** — reflective access to internal JDK APIs is **disabled by default** now (not just discouraged), pushing everyone fully onto supported public APIs.
- **Removal of the Applet API**, deprecation of the Security Manager for future removal.

**Q2: What's a sealed class and why does it matter for pattern matching?**

A:
```java
public sealed interface Shape permits Circle, Square, Triangle {}
public record Circle(double radius) implements Shape {}
public record Square(double side) implements Shape {}
public record Triangle(double base, double height) implements Shape {}

double area(Shape s) {
    return switch (s) {
        case Circle c -> Math.PI * c.radius() * c.radius();
        case Square sq -> sq.side() * sq.side();
        case Triangle t -> 0.5 * t.base() * t.height();
        // no `default` needed — compiler knows this is exhaustive!
    };
}
```
Because `permits` closes the hierarchy to a known, fixed set of subtypes, the compiler can verify a `switch` over that type is **exhaustive** without needing a `default` branch — catching a missed case at compile time instead of at runtime, which is a major reliability win for domain modeling (very similar to sum types/algebraic data types in functional languages).

**Q3: How do records differ from a regular class with manually written getters/equals/hashCode?**

A: `record Point(int x, int y) {}` automatically generates: a canonical constructor, `public` accessor methods matching the component names (`x()`, `y()` — note: **not** `getX()`), `equals()`/`hashCode()` based on all components, and a readable `toString()`. Records are implicitly **final** (can't be extended) and all fields are implicitly **final** (immutable) — they're meant purely for modeling immutable data, not general-purpose classes with mutable state or inheritance.

---

## 9. Java 18 – 20 (2022 – 2023, all non-LTS)

**Q1: What did Java 18 and 19 add?**

A: **Java 18**: **UTF-8 by default** across the JDK's APIs (removing platform-dependent default charset inconsistencies that caused subtle cross-platform bugs); a simple built-in web server (`jwebserver`) for quick static file serving/testing. **Java 19**: **Virtual Threads (first preview, Project Loom)** — lightweight, JVM-managed threads that make massively concurrent blocking-style code cheap (millions of threads instead of thousands); **Structured Concurrency (incubator)**; **Pattern matching for switch (third preview)**; **Record patterns (preview)**.

**Q2: What did Java 20 finalize/preview?**

A: Java 20 continued refining **Virtual Threads (second preview)**, **Structured Concurrency (second incubator)**, **Record patterns (second preview)**, and **Scoped Values (first incubator)** — none finalized yet in 20, but this release matters for interview context since it shows the run-up to Java 21 finalizing virtual threads.

---

## 10. Java 21 (LTS, 2023)

**Q1: What are Java 21's headline features (the most commonly asked "recent Java LTS" question)?**

A: Java 21 is a landmark LTS release:
- **Virtual Threads (finalized)** — the biggest concurrency change since Java 5's `java.util.concurrent`. Virtual threads are managed by the JVM (not the OS) and are extremely lightweight (you can create **millions**), allowing simple **thread-per-request/blocking-style code** to scale like async/reactive code without the complexity of reactive programming models.
- **Sequenced Collections** — new interfaces (`SequencedCollection`, `SequencedSet`, `SequencedMap`) providing a uniform way to access the **first/last** element and get a **reversed view** of ordered collections, filling a long-standing API gap (previously `List`, `Deque`, and `LinkedHashSet` each had inconsistent ways to do this, if at all).
- **Record patterns (finalized)** — deconstruct records directly in `instanceof`/`switch`: `if (obj instanceof Point(int x, int y))`.
- **Pattern matching for switch (finalized)** — `switch` can match on type patterns and record patterns with guards (`case Integer i when i > 0 -> ...`).
- **Generational ZGC** — the low-pause-time garbage collector gets generational support, significantly improving throughput for typical allocation patterns.
- **Key Encapsulation Mechanism API**, **String Templates (preview)**, **Structured Concurrency & Scoped Values (preview)**.

**Q2: What are Virtual Threads and how do they differ from Platform Threads?**

A: **Platform threads** are thin wrappers directly around OS threads — expensive to create (each consumes significant stack memory, MBs by default) and context-switch, so applications historically had to use **thread pools** and reactive/async programming to handle high concurrency (e.g., thousands of concurrent connections) without exhausting OS resources. **Virtual threads** are lightweight threads scheduled by the **JVM** on top of a small pool of platform "carrier" threads — when a virtual thread performs a blocking operation (I/O, `Thread.sleep`), the JVM **unmounts** it from its carrier thread (instead of blocking an OS thread), freeing that carrier to run other virtual threads, then remounts it when the operation completes. This lets you write straightforward, blocking-style, thread-per-request code that scales to handle huge numbers of concurrent tasks, without rewriting business logic in a reactive style:
```java
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    IntStream.range(0, 100_000).forEach(i ->
        executor.submit(() -> {
            Thread.sleep(Duration.ofSeconds(1)); // cheap to block on a virtual thread
            return i;
        })
    );
}
```

**Q3: What is Structured Concurrency, and what problem does it solve?**

A: It treats a group of related concurrent subtasks as a **single unit of work** with a clear parent-child lifecycle — if the parent task is cancelled or fails, all its child tasks are automatically cancelled too, and the parent can't complete until all children have completed/been cancelled. This prevents common concurrency bugs like **thread leaks** (a spawned task outliving the operation that started it, because nothing tracked/cancelled it) and makes error handling and cancellation propagate predictably, rather than needing manual bookkeeping across independently-managed threads/futures.

**Q4: What's a record pattern and how does it enable nested deconstruction?**

A:
```java
record Point(int x, int y) {}
record Line(Point start, Point end) {}

if (shape instanceof Line(Point(var x1, var y1), Point(var x2, var y2))) {
    // x1, y1, x2, y2 are directly bound — no manual .start().x() chains needed
}
```
This lets you deconstruct nested record structures directly in a pattern, extracting deeply nested components in one match rather than a chain of accessor calls — a big readability win for data-oriented code.

---

## 11. Java 22 – 24 (2024 – 2025, all non-LTS)

**Q1: What did Java 22 add?**

A: **Statements before `super()` (preview)** — allows some validation/setup code to run before the mandatory `super()` call in a constructor, previously disallowed. **Unnamed variables and patterns** (`var _ = ...`, `case Point(var x, var _) ->`) for cases where a value is required syntactically but never used. **Foreign Function & Memory API (finalized)** — a safe, supported, pure-Java replacement for JNI to call native code and manage off-heap memory. **Stream gatherers (preview)** — custom intermediate stream operations beyond the built-in set.

**Q2: What did Java 23 add?**

A: **Primitive types in patterns, `instanceof`, and `switch` (preview)** — pattern matching extended to work with primitives, not just reference types. **Structured Concurrency (third preview)**, **Stream gatherers (finalized)** — enabling custom reusable intermediate operations (e.g., windowing/batching elements) that weren't expressible with the built-in stream operations alone. **Class-File API (second preview)** — a standard API for parsing/generating JVM class files, reducing reliance on third-party bytecode libraries like ASM within the JDK's own tooling.

**Q3: What did Java 24 add?**

A: **Stream Gatherers finalized** further refined; continued preview iterations of **Structured Concurrency**, **Scoped Values**, and **Primitive types in patterns**. **Quantum-resistant cryptography algorithms** added (ML-KEM, ML-DSA) in anticipation of post-quantum security needs. Ahead-of-time class loading/linking improvements for faster startup. Removal of the 32-bit x86 port.

---

## 12. Java 25 (LTS, September 2025)

**Q1: What are Java 25's key finalized features (most relevant for "latest Java" interview questions right now)?**

A: Java 25 shipped **18 JEPs**, with several long-running preview features finally reaching maturity:
- **Compact Source Files and Instance Main Methods (JEP 512, finalized)** — lets beginners (and quick scripts) write a minimal program without `public class`, `static`, or `String[] args` boilerplate:
```java
void main() {
    System.out.println("Hello, World!");
}
```
This had been previewing since Java 21 (as JEP 445) through Java 24, and is now permanent — a deliberate simplification aimed at lowering Java's learning-curve barrier, while remaining fully compatible with traditional class-based Java.
- **PEM Encodings of Cryptographic Objects (JEP 470)** — standard API for reading/writing PEM-format keys/certificates without hand-rolled parsing.
- **Stable Values (JEP 502, preview)** — a way to declare deferred, at-most-once-initialized immutable values, useful for lazy initialization patterns with better JIT-optimization potential than manual double-checked-locking.
- **Structured Concurrency (JEP 505, fifth preview)** and **Scoped Values (JEP 506)** — continuing to mature from Java 21's initial preview, aimed particularly at simplifying multi-task concurrent code (explicitly called out as beneficial for AI/agentic workloads doing parallel model calls).
- **Primitive Types in Patterns, `instanceof`, and `switch` (JEP 507, third preview)** — removing long-standing restrictions so pattern matching works uniformly across primitive and reference types.
- **Vector API (JEP 508, 10th incubator)** — SIMD-style vector computation API for data-parallel numeric workloads, still incubating after many releases due to needing further JVM/hardware-intrinsic work.
- **JFR CPU-Time Profiling (JEP 509, experimental)** — more precise CPU-time-based profiling via Java Flight Recorder.
- **Post-Quantum cryptography continues** to expand (building on Java 24's ML-KEM/ML-DSA).

**Q2: Why does Java 25 matter for enterprises specifically, and what's its support timeline?**

A: As an **LTS release** (the successor to Java 21), Java 25 is the version most enterprises will plan multi-year migrations around <cite index="7-1">Oracle plans to provide long-term support for Java 25 for at least eight years</cite>. This gives it the same "stability target" role that Java 8, 11, 17, and 21 played previously — interviewers may ask you to compare it against Java 21 or Java 17 as the "current" vs "previous" LTS.

**Q3: How would you summarize the overall direction Java has taken from 8 → 21 → 25 in one interview-ready sentence?**

A: Java has moved from a purely object-oriented, verbose, imperative style (pre-8) → incorporating **functional programming** (streams/lambdas, Java 8) → **immutable, data-oriented modeling** (records, sealed classes, pattern matching, Java 14–21) → **massively scalable, simple concurrency** (virtual threads, structured concurrency, Java 21–25), while consistently working to **lower the barrier to entry** for newcomers (compact source files, better NPEs, `var`) without sacrificing the strong static typing and backward compatibility Java is known for.

---

## 13. Version Cheat Sheet

### LTS Releases (the ones companies actually standardize on)
| Version | Year | Support Model Note |
|---|---|---|
| Java 8 | 2014 | Still widely used in legacy systems; extended paid support |
| Java 11 | 2018 | First LTS under the new 6-month cadence model |
| Java 17 | 2021 | Sealed classes, records finalized — common current baseline |
| Java 21 | 2023 | Virtual threads finalized — the current major concurrency shift |
| Java 25 | 2025 | Latest LTS; instance main methods finalized, structured concurrency maturing |

### "Which version introduced X" — rapid-fire answers
| Feature | Version |
|---|---|
| Generics, enums, autoboxing, annotations, enhanced for-loop | Java 5 |
| Diamond operator, try-with-resources, strings in switch | Java 7 |
| Lambdas, Streams, `Optional`, default methods, new Date/Time API | Java 8 |
| Module system (JPMS), `List.of()`/immutable factories, JShell | Java 9 |
| `var` (local type inference) | Java 10 |
| New `HttpClient`, single-file source launching | Java 11 |
| Switch expressions | Java 12 (preview) → Java 14 (final) |
| Text blocks | Java 13 (preview) → Java 15 (final) |
| Records | Java 14 (preview) → Java 16 (final) |
| Pattern matching for `instanceof` | Java 14 (preview) → Java 16 (final) |
| Sealed classes | Java 15 (preview) → Java 17 (final) |
| Helpful NullPointerExceptions | Java 14 |
| Pattern matching for `switch`, Record patterns | Java 19–20 (preview) → Java 21 (final) |
| Virtual Threads | Java 19 (preview) → Java 21 (final) |
| Sequenced Collections | Java 21 |
| Foreign Function & Memory API | Java 22 (final) |
| Stream Gatherers | Java 22 (preview) → Java 24 (final) |
| Compact source files / instance `main()` | Java 21 (preview) → Java 25 (final) |
| Structured Concurrency | Java 21 → still preview through Java 25 |

---

# PART 2: MOST-ASKED CODING INTERVIEW QUESTIONS

## 14. Strings

**Q1: Reverse a string without using built-in reverse methods.**
```java
public static String reverse(String s) {
    char[] chars = s.toCharArray();
    int left = 0, right = chars.length - 1;
    while (left < right) {
        char temp = chars[left];
        chars[left++] = chars[right];
        chars[right--] = temp;
    }
    return new String(chars);
}
```
Time: O(n), Space: O(n) for the char array.

**Q2: Check if a string is a palindrome.**
```java
public static boolean isPalindrome(String s) {
    int left = 0, right = s.length() - 1;
    while (left < right) {
        if (s.charAt(left++) != s.charAt(right--)) return false;
    }
    return true;
}
```

**Q3: Check if two strings are anagrams of each other.**
```java
public static boolean isAnagram(String a, String b) {
    if (a.length() != b.length()) return false;
    int[] counts = new int[26];
    for (char c : a.toCharArray()) counts[c - 'a']++;
    for (char c : b.toCharArray()) counts[c - 'a']--;
    for (int count : counts) if (count != 0) return false;
    return true;
}
```
Time: O(n), Space: O(1) (fixed 26-size array).

**Q4: Find the first non-repeating character in a string.**
```java
public static char firstNonRepeating(String s) {
    Map<Character, Integer> freq = new LinkedHashMap<>();
    for (char c : s.toCharArray()) freq.merge(c, 1, Integer::sum);
    for (var entry : freq.entrySet()) {
        if (entry.getValue() == 1) return entry.getKey();
    }
    return '\0'; // not found
}
```
`LinkedHashMap` preserves insertion order, so the first entry with count 1 is truly the first non-repeating character.

**Q5: Why is `String` immutable in Java, and what's the practical benefit?**

A: `String` internals (`char[]`/`byte[]`) are `final` and never exposed for mutation after construction. Benefits: enables the **String pool** (literal strings can be safely shared/interned since they can never change underneath another reference), thread-safety without synchronization, and safety as a `HashMap` key (a mutable key changing after insertion would break the hash bucket it lives in). This is a very common conceptual interview question paired with the coding ones.

---

## 15. Arrays

**Q1: Find the two numbers in an array that sum to a target (Two Sum).**
```java
public static int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> seen = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (seen.containsKey(complement)) {
            return new int[]{seen.get(complement), i};
        }
        seen.put(nums[i], i);
    }
    throw new IllegalArgumentException("No solution");
}
```
Time: O(n), Space: O(n) — the classic follow-up is "can you do it in O(1) space if the array is sorted?" (two-pointer approach instead).

**Q2: Find the missing number in an array of 1..N.**
```java
public static int findMissing(int[] nums, int n) {
    int expectedSum = n * (n + 1) / 2;
    int actualSum = 0;
    for (int num : nums) actualSum += num;
    return expectedSum - actualSum;
}
```
Alternative (avoids overflow risk on large n): XOR all numbers 1..n with all array elements — the missing number is what remains after all pairs cancel out.

**Q3: Find the maximum subarray sum (Kadane's Algorithm).**
```java
public static int maxSubArray(int[] nums) {
    int maxSoFar = nums[0], maxEndingHere = nums[0];
    for (int i = 1; i < nums.length; i++) {
        maxEndingHere = Math.max(nums[i], maxEndingHere + nums[i]);
        maxSoFar = Math.max(maxSoFar, maxEndingHere);
    }
    return maxSoFar;
}
```
Time: O(n), Space: O(1). One of the most frequently asked DP-style array questions.

**Q4: Remove duplicates from an array while preserving order.**
```java
public static int[] removeDuplicates(int[] nums) {
    return Arrays.stream(nums).distinct().toArray();
}
// Manual version without streams, in-place for a sorted array:
public static int removeDupsSorted(int[] nums) {
    if (nums.length == 0) return 0;
    int writeIndex = 1;
    for (int i = 1; i < nums.length; i++) {
        if (nums[i] != nums[writeIndex - 1]) {
            nums[writeIndex++] = nums[i];
        }
    }
    return writeIndex; // new logical length
}
```

**Q5: Rotate an array to the right by k steps, in place.**
```java
public static void rotate(int[] nums, int k) {
    k %= nums.length;
    reverse(nums, 0, nums.length - 1);
    reverse(nums, 0, k - 1);
    reverse(nums, k, nums.length - 1);
}
private static void reverse(int[] nums, int start, int end) {
    while (start < end) {
        int temp = nums[start];
        nums[start++] = nums[end];
        nums[end--] = temp;
    }
}
```
The "reverse three times" trick achieves O(n) time, O(1) space rotation — a favorite because the naive solution (shifting one at a time) is O(n·k).

---

## 16. Linked Lists

**Q1: Reverse a singly linked list.**
```java
static class Node { int val; Node next; Node(int val) { this.val = val; } }

public static Node reverse(Node head) {
    Node prev = null;
    while (head != null) {
        Node next = head.next;
        head.next = prev;
        prev = head;
        head = next;
    }
    return prev;
}
```
Time: O(n), Space: O(1) — expect an immediate recursive-version follow-up too.

**Q2: Detect a cycle in a linked list (Floyd's Tortoise and Hare).**
```java
public static boolean hasCycle(Node head) {
    Node slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) return true;
    }
    return false;
}
```
The fast pointer moves 2x speed — if there's a cycle, they're guaranteed to meet; if not, `fast` reaches `null` first.

**Q3: Find the middle node of a linked list in one pass.**
```java
public static Node findMiddle(Node head) {
    Node slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    return slow; // slow is at the middle when fast reaches the end
}
```

**Q4: Merge two sorted linked lists.**
```java
public static Node mergeSorted(Node l1, Node l2) {
    Node dummy = new Node(0);
    Node curr = dummy;
    while (l1 != null && l2 != null) {
        if (l1.val <= l2.val) { curr.next = l1; l1 = l1.next; }
        else { curr.next = l2; l2 = l2.next; }
        curr = curr.next;
    }
    curr.next = (l1 != null) ? l1 : l2;
    return dummy.next;
}
```

---

## 17. Sorting & Searching

**Q1: Implement Binary Search.**
```java
public static int binarySearch(int[] arr, int target) {
    int low = 0, high = arr.length - 1;
    while (low <= high) {
        int mid = low + (high - low) / 2; // avoids overflow vs (low+high)/2
        if (arr[mid] == target) return mid;
        else if (arr[mid] < target) low = mid + 1;
        else high = mid - 1;
    }
    return -1;
}
```
Time: O(log n) — interviewers often ask why `low + (high - low) / 2` is preferred over `(low + high) / 2` (overflow safety for very large indices).

**Q2: Implement Quicksort.**
```java
public static void quickSort(int[] arr, int low, int high) {
    if (low < high) {
        int pivotIndex = partition(arr, low, high);
        quickSort(arr, low, pivotIndex - 1);
        quickSort(arr, pivotIndex + 1, high);
    }
}
private static int partition(int[] arr, int low, int high) {
    int pivot = arr[high];
    int i = low - 1;
    for (int j = low; j < high; j++) {
        if (arr[j] < pivot) {
            i++;
            int temp = arr[i]; arr[i] = arr[j]; arr[j] = temp;
        }
    }
    int temp = arr[i + 1]; arr[i + 1] = arr[high]; arr[high] = temp;
    return i + 1;
}
```
Average O(n log n), worst case O(n²) (already-sorted input with a naive pivot choice) — a common follow-up is "how would you avoid the worst case?" (randomized pivot selection).

**Q3: Implement Merge Sort.**
```java
public static void mergeSort(int[] arr, int left, int right) {
    if (left >= right) return;
    int mid = left + (right - left) / 2;
    mergeSort(arr, left, mid);
    mergeSort(arr, mid + 1, right);
    merge(arr, left, mid, right);
}
private static void merge(int[] arr, int left, int mid, int right) {
    int[] temp = new int[right - left + 1];
    int i = left, j = mid + 1, k = 0;
    while (i <= mid && j <= right) temp[k++] = (arr[i] <= arr[j]) ? arr[i++] : arr[j++];
    while (i <= mid) temp[k++] = arr[i++];
    while (j <= right) temp[k++] = arr[j++];
    System.arraycopy(temp, 0, arr, left, temp.length);
}
```
Always O(n log n), stable, but O(n) extra space — a common comparison question is "quicksort vs merge sort," testing whether you know the space/stability/worst-case tradeoffs.

---

## 18. Recursion & Dynamic Programming

**Q1: Fibonacci — recursive, memoized, and iterative (a classic "optimize this" progression question).**
```java
// Naive recursive — O(2^n), exponential, asked first to set up the optimization discussion
public static int fibNaive(int n) {
    if (n <= 1) return n;
    return fibNaive(n - 1) + fibNaive(n - 2);
}

// Memoized (top-down DP) — O(n) time, O(n) space
public static int fibMemo(int n, Map<Integer, Integer> memo) {
    if (n <= 1) return n;
    if (memo.containsKey(n)) return memo.get(n);
    int result = fibMemo(n - 1, memo) + fibMemo(n - 2, memo);
    memo.put(n, result);
    return result;
}

// Iterative (bottom-up) — O(n) time, O(1) space — usually the "best" expected final answer
public static int fibIterative(int n) {
    if (n <= 1) return n;
    int prev = 0, curr = 1;
    for (int i = 2; i <= n; i++) {
        int next = prev + curr;
        prev = curr;
        curr = next;
    }
    return curr;
}
```

**Q2: Factorial (recursive) and why deep recursion risks `StackOverflowError`.**
```java
public static long factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}
```
Each recursive call adds a stack frame; without tail-call optimization (which the JVM does **not** perform, unlike some functional languages), a sufficiently large `n` exhausts the call stack. This is a common follow-up to test understanding of the call stack, not just recursion syntax.

**Q3: Coin Change — minimum coins to make a target amount (classic DP question).**
```java
public static int coinChange(int[] coins, int amount) {
    int[] dp = new int[amount + 1];
    Arrays.fill(dp, amount + 1); // "infinity" sentinel
    dp[0] = 0;
    for (int i = 1; i <= amount; i++) {
        for (int coin : coins) {
            if (coin <= i) {
                dp[i] = Math.min(dp[i], dp[i - coin] + 1);
            }
        }
    }
    return dp[amount] > amount ? -1 : dp[amount];
}
```
Time: O(amount × coins.length), Space: O(amount).

**Q4: Check if a number is prime, and generate primes up to N (Sieve of Eratosthenes).**
```java
public static boolean isPrime(int n) {
    if (n < 2) return false;
    for (int i = 2; (long) i * i <= n; i++) {
        if (n % i == 0) return false;
    }
    return true;
}

public static List<Integer> sieve(int n) {
    boolean[] isComposite = new boolean[n + 1];
    List<Integer> primes = new ArrayList<>();
    for (int i = 2; i <= n; i++) {
        if (!isComposite[i]) {
            primes.add(i);
            for (int j = i * 2; j <= n; j += i) isComposite[j] = true;
        }
    }
    return primes;
}
```
The sieve runs in O(n log log n) — much faster than checking each number individually when you need **all** primes up to N.

---

## 19. Collections & HashMap Internals

**Q1: How does `HashMap` work internally? (Extremely common conceptual question.)**

A: A `HashMap` stores entries in an internal **array of buckets**. For each key, `hashCode()` is computed and passed through an internal **hash spreading function** (to reduce collisions from poor `hashCode()` implementations), then reduced modulo the array length to pick a bucket index. Each bucket holds a **linked list** of entries that hashed to the same index (collision chaining) — since **Java 8**, if a bucket's chain grows beyond a threshold (8 entries, and the table has ≥64 buckets), that bucket is converted to a **balanced red-black tree** instead of a linked list, improving worst-case lookup from O(n) to O(log n) for heavily-collided buckets. The default **load factor** is 0.75 — once the map's size exceeds `capacity × loadFactor`, it **resizes** (doubles capacity) and rehashes all entries.

**Q2: Why must you override both `equals()` and `hashCode()` together, and what breaks if you don't?**

A: `HashMap`/`HashSet` use `hashCode()` to locate the correct bucket, then `equals()` to check for an actual match within that bucket's entries. If you override `equals()` but not `hashCode()` (or vice versa), you can end up with two "equal" objects that hash to **different buckets** — meaning a `HashMap` will fail to find an existing key that should logically match, silently creating duplicate entries or failing lookups. The contract: **equal objects must have equal hash codes** (but not necessarily vice versa — unequal objects can share a hash code, that's just a collision).

**Q3: `ArrayList` vs `LinkedList` — when would you choose each?**

A: `ArrayList` is backed by a resizable array — O(1) random access (`get(i)`), but O(n) insertion/removal in the middle (requires shifting elements), and amortized O(1) append (occasional O(n) resize/copy). `LinkedList` is a doubly-linked list — O(1) insertion/removal **once you have a reference to the node** (e.g., at the head/tail, or via an iterator), but O(n) random access (must traverse from an end). In practice, `ArrayList` is the default choice for most use cases (better cache locality, lower memory overhead per element) — `LinkedList` is rarely the right choice unless you specifically need frequent head/tail insertion/removal (and even then, `ArrayDeque` often outperforms it).

**Q4: `HashMap` vs `TreeMap` vs `LinkedHashMap` — how do they differ in ordering and complexity?**

A: `HashMap` — no ordering guarantee, O(1) average get/put. `LinkedHashMap` — maintains **insertion order** (or optionally access order, useful for building an LRU cache), O(1) average get/put with a small overhead for the internal linked list. `TreeMap` — maintains **sorted key order** (natural ordering or a custom `Comparator`), backed by a red-black tree, O(log n) get/put — chosen when you need range queries (`headMap`, `tailMap`, `firstKey`) or sorted iteration.

**Q5: Design a simple LRU cache using `LinkedHashMap`.**
```java
public class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int capacity;

    public LRUCache(int capacity) {
        super(capacity, 0.75f, true); // accessOrder = true
        this.capacity = capacity;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > capacity;
    }
}
```
`LinkedHashMap`'s `accessOrder=true` constructor reorders entries on every access (moving recently-accessed entries to the end), and overriding `removeEldestEntry` lets the map automatically evict the least-recently-used entry once capacity is exceeded — a favorite "design something using standard library internals" question.

---

## 20. Multithreading & Concurrency

**Q1: What's the difference between `synchronized`, `ReentrantLock`, and `volatile`?**

A: **`synchronized`** — intrinsic monitor lock; simple, JVM-managed, automatically released even on exception (via the implicit unlock at block/method exit), but inflexible (can't try-lock, can't interrupt a waiting thread, can't have separate read/write conditions). **`ReentrantLock`** — explicit lock offering more control: `tryLock()` (non-blocking attempt), `lockInterruptibly()`, fairness policies, and multiple `Condition` objects per lock — but requires manual `unlock()` in a `finally` block, or you risk deadlocking other threads if you forget. **`volatile`** — not a lock at all; guarantees **visibility** (a write by one thread is immediately visible to other threads, preventing CPU-cache/reordering staleness) and prevents instruction reordering around it, but does **not** provide atomicity for compound operations (e.g., `volatile int count; count++;` is still a race condition, since increment is read-modify-write, not a single atomic op).

**Q2: Implement a simple Producer-Consumer using `wait()`/`notify()`.**
```java
class SharedBuffer {
    private final Queue<Integer> queue = new LinkedList<>();
    private final int capacity;

    public SharedBuffer(int capacity) { this.capacity = capacity; }

    public synchronized void produce(int value) throws InterruptedException {
        while (queue.size() == capacity) wait();
        queue.add(value);
        notifyAll();
    }

    public synchronized int consume() throws InterruptedException {
        while (queue.isEmpty()) wait();
        int value = queue.poll();
        notifyAll();
        return value;
    }
}
```
`wait()` must always be in a `while` loop (not `if`) to guard against **spurious wakeups** and to re-check the condition after being notified (since another thread might've changed state again before this thread actually resumes). `notifyAll()` is generally safer than `notify()` (which wakes only one arbitrary waiting thread and can cause missed-signal deadlocks if the wrong thread wakes up for the wrong condition).

**Q3: What's the modern (Java 5+) alternative to raw `wait()`/`notify()` for producer-consumer, and why prefer it?**

A:
```java
BlockingQueue<Integer> queue = new LinkedBlockingQueue<>(capacity);
// producer:
queue.put(value); // blocks if full
// consumer:
int value = queue.take(); // blocks if empty
```
`BlockingQueue` implementations (`LinkedBlockingQueue`, `ArrayBlockingQueue`) handle all the wait/notify/locking logic internally, are far less error-prone (no risk of forgetting the `while` loop, wrong lock object, or missed notifications), and are the idiomatic, expected answer in modern Java interviews over hand-rolled `wait()`/`notify()`.

**Q4: What's a deadlock, and how do you prevent it?**

A: A deadlock occurs when two or more threads are each waiting on a lock the other holds, with neither able to proceed (classic example: Thread A holds Lock 1 and wants Lock 2, while Thread B holds Lock 2 and wants Lock 1). Prevention strategies: always **acquire locks in a consistent global order** across all threads, use **lock timeouts** (`tryLock(timeout)`) instead of indefinite blocking, minimize the scope/number of locks held simultaneously, and prefer higher-level concurrency utilities (`java.util.concurrent` classes, or in modern Java, structured concurrency) that manage lock ordering internally rather than hand-coding multi-lock acquisition.

**Q5: `Runnable` vs `Callable` — what's the difference?**

A: `Runnable.run()` returns `void` and can't throw checked exceptions. `Callable<V>.call()` **returns a value** and **can throw checked exceptions**, and is designed to be submitted to an `ExecutorService` returning a `Future<V>` you can later `.get()` (blocking) to retrieve the result or unwrap any exception the task threw.

---

## 21. Streams & Functional Java

**Q1: Group a list of employees by department using streams.**
```java
record Employee(String name, String department, double salary) {}

Map<String, List<Employee>> byDept = employees.stream()
    .collect(Collectors.groupingBy(Employee::department));

// with a downstream collector — average salary per department
Map<String, Double> avgSalaryByDept = employees.stream()
    .collect(Collectors.groupingBy(Employee::department,
              Collectors.averagingDouble(Employee::salary)));
```

**Q2: Find the top 3 highest-paid employees using streams.**
```java
List<Employee> top3 = employees.stream()
    .sorted(Comparator.comparingDouble(Employee::salary).reversed())
    .limit(3)
    .collect(Collectors.toList());
```

**Q3: Count word frequency in a sentence using streams.**
```java
Map<String, Long> wordCounts = Arrays.stream(sentence.toLowerCase().split("\\s+"))
    .collect(Collectors.groupingBy(w -> w, Collectors.counting()));
```

**Q4: What's the difference between `Collectors.toList()` and `Collectors.toUnmodifiableList()` / `Stream.toList()`?**

A: `Collectors.toList()` doesn't guarantee mutability/immutability by contract (in practice it typically returns a mutable `ArrayList`, but that's not a documented guarantee). `Collectors.toUnmodifiableList()` and `Stream.toList()` (Java 16+, a convenience shorthand) explicitly return an **immutable** list — attempting to modify it throws `UnsupportedOperationException`. Using the immutable versions by default is generally better practice unless you specifically need a mutable result.

**Q5: Explain `reduce()` with an example, and when you'd need the three-argument overload.**
```java
int sum = numbers.stream().reduce(0, Integer::sum);

// three-arg overload — needed for PARALLEL streams with a type-changing accumulator
int totalLength = words.stream().reduce(
    0,
    (partialResult, word) -> partialResult + word.length(),  // accumulator
    Integer::sum                                              // combiner, merges partial results from parallel chunks
);
```
The combiner is required whenever the accumulator's result type differs from the stream's element type — it tells the Stream API how to merge partial results computed independently in different threads when running as a `parallelStream()`.

---

## 22. OOP / Design Questions

**Q1: Implement a thread-safe Singleton (a perennial favorite).**
```java
public class Singleton {
    private static volatile Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) { // double-checked locking
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```
The `volatile` keyword is essential here — without it, another thread could observe a **partially constructed** object due to instruction reordering (the reference could be assigned before the constructor fully finishes). The simplest, safest modern alternative interviewers also accept:
```java
public enum Singleton {
    INSTANCE;
    public void doSomething() { /* ... */ }
}
```
Enum-based singletons are inherently thread-safe, serialization-safe, and reflection-attack-safe (a common follow-up: "why is enum singleton considered the best approach?").

**Q2: Design an immutable class.**
```java
public final class ImmutablePoint { // final class: prevent subclassing from adding mutability
    private final int x;
    private final int y;
    private final List<String> tags; // mutable field type example

    public ImmutablePoint(int x, int y, List<String> tags) {
        this.x = x;
        this.y = y;
        this.tags = List.copyOf(tags); // defensive copy, and immutable itself
    }

    public int getX() { return x; }
    public int getY() { return y; }
    public List<String> getTags() { return tags; } // safe to return directly — already immutable
}
```
Key rules: mark the class `final`, all fields `private final`, no setters, and **defensively copy** any mutable object passed in (or returned) so external code can't reach in and mutate internal state after construction.

**Q3: What's the difference between method overloading and overriding, and how does the JVM resolve each?**

A: **Overloading** — same method name, different parameter list, within the same class; resolved at **compile time** based on the static/declared type of the arguments (static/early binding). **Overriding** — a subclass redefines a superclass method with the **same signature**; resolved at **runtime** based on the actual object's type (dynamic/late binding via **virtual method dispatch**) — this is what enables polymorphism (`Animal a = new Dog(); a.makeSound();` calls `Dog`'s override, even though `a` is statically typed as `Animal`).

**Q4: Explain the difference between abstract classes and interfaces, especially post-Java 8.**

A: Before Java 8, the distinction was clean: abstract classes could have state (fields) and constructors, interfaces could only have abstract method signatures and constants. Since Java 8's **default methods** (and Java 9's private interface methods), interfaces can carry behavior too, blurring the line — but key differences remain: a class can implement **multiple** interfaces but extend only **one** abstract class (single inheritance of state); abstract classes can have **instance fields/constructors** (real state), interfaces still cannot have instance state (only `static final` constants); use an abstract class when subclasses share significant common state/implementation, use an interface to define a **contract/capability** that unrelated classes might implement.

**Q5: What is the SOLID principle, briefly, and can you give a Java example of violating/fixing the Single Responsibility Principle?**

A: SOLID = **S**ingle Responsibility, **O**pen/Closed, **L**iskov Substitution, **I**nterface Segregation, **D**ependency Inversion — five object-oriented design guidelines for maintainable code.
```java
// Violates SRP — this class both computes AND persists AND formats
class Report {
    void calculate() { /* ... */ }
    void saveToDatabase() { /* ... */ }
    void printToConsole() { /* ... */ }
}

// Fixed — each class has one reason to change
class ReportCalculator { void calculate() { /* ... */ } }
class ReportRepository { void save(Report r) { /* ... */ } }
class ReportPrinter { void print(Report r) { /* ... */ } }
```
This is a very common "explain a design principle with code" style question, especially for mid/senior-level interviews.

---

## Quick-Reference Cheat Sheet

| Category | Interview essentials to remember |
|---|---|
| Java 5 | Generics, enums, autoboxing, `java.util.concurrent`, annotations |
| Java 8 | Lambdas, Streams, `Optional`, default methods, new Date/Time API — the single most-tested version |
| Java 9 | Modules (JPMS), `List.of()` immutable factories |
| Java 11 | New `HttpClient`, string methods (`isBlank`, `strip`), first post-8 LTS |
| Java 17 | Sealed classes, records, pattern matching for `instanceof` — all finalized |
| Java 21 | Virtual threads, record patterns, pattern matching for switch, sequenced collections |
| Java 25 | Compact source files/instance `main()` finalized, structured concurrency maturing, latest LTS |
| Strings | Immutable for pooling/thread-safety/hash-key safety; two-pointer technique for reverse/palindrome |
| Arrays | HashMap for O(n) lookups (Two Sum); Kadane's for max subarray; reverse-thrice trick for rotation |
| Linked Lists | Fast/slow pointers for cycle detection and finding the middle |
| Sorting | Quicksort avg O(n log n)/worst O(n²) in-place; Merge sort always O(n log n) but O(n) space, stable |
| Recursion/DP | Memoization turns exponential into polynomial; watch for `StackOverflowError` on deep recursion |
| Collections | `HashMap` = array of buckets + linked list/tree collision handling since Java 8; always pair `equals`+`hashCode` |
| Concurrency | `volatile` = visibility only, not atomicity; prefer `BlockingQueue`/`java.util.concurrent` over raw `wait/notify` |
| Streams | Intermediate ops are lazy; a stream can only be consumed once |
| OOP | Overloading = compile-time; overriding = runtime (virtual dispatch); enum singleton is safest |

---

*Tip: for the version-history questions, interviewers usually care less about exact JEP numbers and more about whether you can explain **why** a feature mattered (what problem it solved) — lead with that, then mention the version if you're confident. For coding questions, always state the time/space complexity out loud even if not asked; it signals you're thinking beyond "does it compile."*
