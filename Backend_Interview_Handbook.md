# Backend Engineering Interview Handbook
### Spring Boot · Microservices · Docker · Kubernetes

A question-and-answer style prep guide for mid-to-senior backend interviews. Each section covers concepts, common interview questions, and crisp answers you can expand on verbally.

---

## Table of Contents

**Part 1 — Spring Boot**
1. Startup Lifecycle
2. Bean Scopes and Lifecycle
3. Autowiring Internals
4. Transaction Propagation
5. Spring Data JPA Internals
6. Hibernate Caching
7. Lazy vs Eager Loading
8. Optimistic vs Pessimistic Locking

**Part 2 — Microservices**
9. API Gateway Design
10. Service Discovery
11. Distributed Transactions (Saga)
12. Event-Driven Architecture
13. Idempotency
14. Retry Strategies
15. Circuit Breaker Patterns
16. Observability and Monitoring

**Part 3 — Docker**
17. Multi-stage Builds
18. Image Optimization
19. Container Networking
20. Volumes and Bind Mounts
21. Docker Compose for Local Development
22. Security Best Practices

**Part 4 — Kubernetes**
23. Pod Lifecycle
24. Scheduling and Taints/Tolerations
25. Health Probes
26. Scaling
27. Rolling Updates and Rollbacks
28. Secrets Management
29. Networking (Services, Ingress)

---

# PART 1: SPRING BOOT

## 1. Spring Boot Startup Lifecycle

**Q1: Walk through what happens when a Spring Boot application starts (`SpringApplication.run()`).**

A: At a high level:
1. **`SpringApplication` instance is created** — it detects the application type (Servlet, Reactive, or None) by inspecting the classpath, and figures out primary sources (the main class).
2. **`ApplicationContextInitializers` and `ApplicationListeners`** registered in `spring.factories` (or `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` in Boot 2.7+/3.x) are loaded.
3. **Environment is prepared** — property sources (application.properties/yml, env vars, command-line args) are resolved and merged, and an `ApplicationEnvironmentPreparedEvent` fires.
4. **Banner is printed.**
5. **ApplicationContext is created** — `AnnotationConfigServletWebServerApplicationContext` for a typical web app.
6. **Context is "prepared"** — the context is loaded with the environment, bean definition readers are set up, and the primary source is registered.
7. **Context is "refreshed"** (`AbstractApplicationContext.refresh()`) — this is the core Spring container bootstrap:
 - `BeanFactoryPostProcessors` run (including `ConfigurationClassPostProcessor`, which triggers `@ComponentScan`, `@Configuration`, and **auto-configuration** classes).
 - `BeanPostProcessors` are registered.
 - Singleton beans are instantiated eagerly (unless lazy).
 - The embedded web server (Tomcat/Netty/Jetty) is created and started as a bean during this phase.
8. **`CommandLineRunner`/`ApplicationRunner` beans execute** after the context is fully refreshed.
9. **`ApplicationReadyEvent`** fires — app is ready to serve traffic.

**Q2: What's the role of auto-configuration and how does Spring Boot decide what to configure?**

A: Auto-configuration classes are conditionally applied using `@Conditional` annotations (`@ConditionalOnClass`, `@ConditionalOnMissingBean`, `@ConditionalOnProperty`, etc.). Spring Boot scans `AutoConfiguration.imports` for candidate classes, then each one self-evaluates whether it should activate based on what's on the classpath and what beans already exist. This is why adding `spring-boot-starter-data-jpa` "just works" — `DataSourceAutoConfiguration`, `HibernateJpaAutoConfiguration`, etc. detect the JPA/JDBC classes and configure sensible defaults, but back off if you've defined your own beans (`@ConditionalOnMissingBean`).

**Q3: What's the difference between `ApplicationContext` refresh phases like `postProcessBeanFactory`, `invokeBeanFactoryPostProcessors`, and `finishBeanFactoryInitialization`?**

A:
- `postProcessBeanFactory` — hook for subclasses to add BFPPs before user-defined ones run.
- `invokeBeanFactoryPostProcessors` — runs all `BeanFactoryPostProcessor`s, which can modify bean *definitions* (metadata) before any bean is instantiated. This is where `@Configuration` classes get parsed and `@Bean` methods registered.
- `finishBeanFactoryInitialization` — actually instantiates all remaining non-lazy singleton beans, running the full bean lifecycle (constructor → property injection → `Aware` callbacks → `@PostConstruct`/`InitializingBean` → ready).

---

## 2. Bean Scopes and Lifecycle

**Q1: What are the standard Spring bean scopes?**

A:
- **singleton** (default) — one instance per Spring container.
- **prototype** — a new instance every time the bean is requested.
- **request** — one instance per HTTP request (web-aware contexts only).
- **session** — one instance per HTTP session.
- **application** — one instance per `ServletContext`.
- **websocket** — one instance per WebSocket session.

**Q2: Why can injecting a prototype bean into a singleton be a problem, and how do you fix it?**

A: A singleton is instantiated once, so if you inject a prototype bean via normal field/constructor injection, Spring resolves it *once* at singleton creation time — you effectively get a "frozen" prototype, defeating the purpose. Fixes:
- Use `ObjectProvider<T>` / `ObjectFactory<T>` and call `.getObject()` each time you need a fresh instance.
- Use a **scoped proxy** (`@Scope(value="prototype", proxyMode=ScopedProxyMode.TARGET_CLASS)`) so each method call on the injected reference is delegated to a freshly fetched instance.
- Use `ApplicationContext.getBean()` directly (less idiomatic, tighter coupling).

**Q3: Describe the full bean lifecycle from instantiation to destruction.**

A:
1. Instantiate (constructor call).
2. Populate properties (dependency injection).
3. `Aware` interfaces invoked (`BeanNameAware`, `BeanFactoryAware`, `ApplicationContextAware`, etc.).
4. `BeanPostProcessor.postProcessBeforeInitialization()`.
5. `@PostConstruct` method, then `InitializingBean.afterPropertiesSet()`, then custom `init-method`.
6. `BeanPostProcessor.postProcessAfterInitialization()` — this is where proxies (AOP, transactions) are typically wrapped in.
7. Bean is ready for use.
8. On context shutdown: `@PreDestroy`, then `DisposableBean.destroy()`, then custom `destroy-method` (singleton beans only — prototypes are **not** destroyed by the container).

**Q4: What's the difference between `BeanFactoryPostProcessor` and `BeanPostProcessor`?**

A: `BeanFactoryPostProcessor` operates on bean **definitions** (metadata) before any beans are instantiated — e.g., `PropertySourcesPlaceholderConfigurer` resolving `${}` placeholders. `BeanPostProcessor` operates on bean **instances** after instantiation, wrapping or modifying the actual object — e.g., `AutowiredAnnotationBeanPostProcessor`, or AOP proxy creation via `AnnotationAwareAspectJAutoProxyCreator`.

---

## 3. Autowiring Internals

**Q1: How does `@Autowired` resolve which bean to inject?**

A: Resolution order:
1. **By type** — if exactly one bean of the required type exists, inject it.
2. If multiple candidates exist, Spring tries to narrow by **`@Qualifier`** name, or by matching the **field/parameter name** to a bean name.
3. If still ambiguous, checks for a bean marked **`@Primary`**.
4. If none of the above resolves it uniquely, throws `NoUniqueBeanDefinitionException`.

This resolution is done by `AutowiredAnnotationBeanPostProcessor`, which post-processes bean instantiation and injects dependencies via `DefaultListableBeanFactory.resolveDependency()`.

**Q2: Constructor vs field vs setter injection — which is recommended and why?**

A: **Constructor injection** is the Spring team's recommended approach:
- Enables **immutability** (fields can be `final`).
- Makes **required dependencies explicit** — the object can't be constructed in an invalid state.
- Easier to **unit test** without a Spring container (just call the constructor).
- Avoids circular dependency issues going unnoticed (constructor injection fails fast on circular deps; field/setter injection can mask them via early-reference proxying).

Field injection is discouraged because it hides dependencies, requires reflection, and can't produce immutable/final fields.

**Q3: How does Spring resolve circular dependencies, and when does it fail?**

A: For **singleton beans with setter/field injection**, Spring uses a three-tier cache (`singletonObjects`, `earlySingletonObjects`, `singletonFactories`) to expose an early, not-fully-initialized reference of a bean while it's still being constructed, breaking the cycle. For **constructor injection**, this early-reference trick isn't possible (the object doesn't exist yet), so circular constructor dependencies throw `BeanCurrentlyInCreationException`. As of Spring Boot 2.6+, circular references are **disallowed by default** even for setter injection (`spring.main.allow-circular-references=false` by default) — you must opt back in or, better, refactor to remove the cycle (e.g., extract a shared dependency, use `@Lazy`).

**Q4: What does `@Qualifier` do differently from `@Primary`?**

A: `@Primary` sets a **default** candidate when multiple beans match a type — used at the bean definition site. `@Qualifier` is used at the **injection point** to explicitly pick a specific bean by name/qualifier value, and always overrides `@Primary` when both exist.

---

## 4. Transaction Propagation

**Q1: List Spring's transaction propagation types and when you'd use them.**

A:
- **REQUIRED** (default) — join existing transaction, or create a new one if none exists. Most common choice.
- **REQUIRES_NEW** — always suspends any existing transaction and starts a new, independent one. Useful for audit logging that must commit even if the outer transaction rolls back.
- **NESTED** — runs within a nested transaction (savepoint) inside the existing one, if present; rollback of the nested part doesn't necessarily roll back the outer transaction (rollback to savepoint). Requires JDBC savepoint support.
- **SUPPORTS** — joins a transaction if one exists, otherwise runs non-transactionally.
- **NOT_SUPPORTED** — suspends any existing transaction and runs without one.
- **MANDATORY** — must run within an existing transaction; throws an exception if none exists.
- **NEVER** — must **not** run within a transaction; throws if one exists.

**Q2: Why does calling a `@Transactional` method from another method in the same class not start a new transaction?**

A: Spring's declarative transactions are implemented via **AOP proxies** (JDK dynamic proxy or CGLIB). When method A calls method B on `this` (self-invocation), it bypasses the proxy entirely — you're calling the method directly on the target object, so the proxy's transaction interceptor never gets a chance to run. Fixes: inject a self-reference (`@Autowired` the class into itself), split the method into a separate bean, or use `AopContext.currentProxy()` with `exposeProxy=true`.

**Q3: What causes a transaction to roll back, and how do you customize it?**

A: By default, Spring rolls back on **unchecked exceptions** (`RuntimeException`, `Error`) and **not** on checked exceptions. You customize with `@Transactional(rollbackFor = SomeCheckedException.class)` or `noRollbackFor = ...`. A common gotcha: catching an exception inside the method and swallowing it prevents rollback entirely, since Spring only sees what propagates out of the proxied method.

**Q4: What's the isolation level default, and what problem does each isolation level solve?**

A: Default is usually the underlying **database's default** (e.g., `READ_COMMITTED` for PostgreSQL/SQL Server, `REPEATABLE_READ` for MySQL/InnoDB).
- **READ_UNCOMMITTED** — allows dirty reads.
- **READ_COMMITTED** — prevents dirty reads; non-repeatable reads still possible.
- **REPEATABLE_READ** — prevents dirty + non-repeatable reads; phantom reads still possible (though InnoDB largely prevents these too via gap locks).
- **SERIALIZABLE** — fully isolated, prevents phantom reads, but at the cost of concurrency (effectively serializes transactions).

---

## 5. Spring Data JPA Internals

**Q1: How does Spring Data JPA implement a repository interface at runtime with no code written?**

A: Spring Data creates a **dynamic proxy** for each repository interface via `RepositoryFactoryBeanSupport` / `JpaRepositoryFactory`. Method calls are routed to:
- `SimpleJpaRepository` for standard CRUD methods (`findById`, `save`, `deleteAll`, etc.).
- A **query-derivation mechanism** for methods matching naming conventions (`findByLastNameAndAgeGreaterThan`) — parsed by `PartTree`, which tokenizes the method name into predicates and builds a JPQL/Criteria query.
- Custom `@Query`-annotated methods, executed via `JpaQueryFactory`.
- Custom implementation classes if you provide a `RepositoryImpl` fragment.

**Q2: What is the N+1 select problem and how do you detect/fix it in Spring Data JPA?**

A: It happens when fetching a list of N parent entities triggers 1 query for the parents, then N additional queries — one per parent — to lazily fetch an associated collection/entity. Detect via SQL logging (`spring.jpa.show-sql=true` + `logging.level.org.hibernate.SQL=DEBUG`) or tools like p6spy/Hibernate statistics. Fix with:
- `JOIN FETCH` in JPQL.
- `@EntityGraph` on the repository method.
- Batch fetching (`hibernate.default_batch_fetch_size`).
- `@BatchSize` on the association.

**Q3: What's the difference between `save()` and `saveAndFlush()`, and how does the persistence context relate to it?**

A: `save()` persists/merges the entity into the **persistence context** (first-level cache) but the actual SQL `INSERT`/`UPDATE` may be deferred until flush time (end of transaction, or next query that needs consistent state). `saveAndFlush()` forces an immediate flush to the database. Understanding flush timing matters for things like getting a generated ID immediately, or ensuring visibility to a subsequent native query within the same transaction.

**Q4: How do derived delete/count queries and `@Modifying` queries differ from regular finder methods?**

A: `@Modifying` is required for `@Query` methods that perform `UPDATE`/`DELETE` (bulk operations), since these bypass the persistence context and go straight to the database — meaning entities already loaded in the first-level cache can become stale unless you also set `clearAutomatically = true` (or manually clear the `EntityManager`).

---

## 6. Hibernate Caching

**Q1: Explain Hibernate's cache levels.**

A:
- **First-level cache (L1)** — scoped to the `Session`/`EntityManager` (i.e., per transaction/persistence context). Always on, can't be disabled. Ensures the same entity fetched twice in one session returns the identical object.
- **Second-level cache (L2)** — scoped to the `SessionFactory`, shared across sessions/transactions. Off by default; needs a provider like **Ehcache**, **Caffeine**, or **Infinispan**, plus `@Cacheable` on entities and `hibernate.cache.use_second_level_cache=true`.
- **Query cache** — caches the *result set* (IDs) of a query, separate from L2 entity cache; requires `hibernate.cache.use_query_cache=true` and explicit `.setCacheable(true)` on the query. Needs L2 cache enabled to be useful, since it only stores identifiers and relies on L2 for the actual entity data.

**Q2: What cache concurrency strategies exist for L2 cache, and when do you use each?**

A:
- **READ_ONLY** — for data that never changes (e.g., lookup/reference tables). Fastest, simplest.
- **READ_WRITE** — uses "soft" locks to maintain consistency on update; suitable for data that changes occasionally and needs decent consistency guarantees.
- **NONSTRICT_READ_WRITE** — no locking, small window of staleness possible after updates; use when occasional stale reads are acceptable for performance.
- **TRANSACTIONAL** — full JTA-based transactional consistency (typically with JBoss/Infinispan); strongest guarantee, most overhead.

**Q3: What are the risks of enabling second-level cache carelessly?**

A: Stale data in multi-instance deployments unless using a distributed cache (Ehcache with clustering, Redis-backed, Hazelcast) — a plain local in-memory L2 cache means each service instance has its own cache, so writes on one node aren't visible to another node's cache. Also increases memory footprint and adds complexity in invalidation logic, especially with bulk/native updates that bypass Hibernate and don't trigger cache eviction.

---

## 7. Lazy vs Eager Loading

**Q1: What's the default fetch type for `@OneToMany`/`@ManyToMany` vs `@ManyToOne`/`@OneToOne`?**

A: Collections (`@OneToMany`, `@ManyToMany`) default to **LAZY**. Single-valued associations (`@ManyToOne`, `@OneToOne`) default to **EAGER** — a commonly cited JPA "gotcha" since eager loading on these can silently cause deep object graphs to load and cascading N+1 problems.

**Q2: What is `LazyInitializationException` and when does it occur?**

A: It's thrown when code tries to access a lazily-loaded association/proxy **after** the owning `Session`/`EntityManager` has closed — commonly happens when an entity is fetched inside a `@Transactional` service method, returned to a controller, and then the view or a serializer (e.g., Jackson) tries to touch a lazy collection outside the transaction boundary. Fixes:
- Fetch what you need eagerly within the transaction (`JOIN FETCH`, `@EntityGraph`, DTO projections).
- Use DTOs instead of returning entities directly to the presentation layer.
- (Anti-pattern, avoid in production) `Open Session in View` (`spring.jpa.open-in-view=true`, the Boot default) keeps the session open through the view-rendering phase — convenient but can hide N+1 problems and holds DB connections longer than necessary.

**Q3: How would you decide between eager and lazy fetching for a given association?**

A: Default to **lazy everywhere** and fetch eagerly only per-query as needed (via `JOIN FETCH`/`@EntityGraph`), rather than baking eager fetching into the entity mapping. This gives you control per use case instead of a single fixed strategy that either over-fetches (eager) or triggers repeated round trips (lazy without explicit joins).

---

## 8. Optimistic vs Pessimistic Locking

**Q1: How does optimistic locking work in JPA/Hibernate?**

A: Uses a **`@Version`** column (int/long/timestamp) on the entity. On update, Hibernate includes `WHERE version = :loadedVersion` in the SQL and checks the affected row count. If another transaction updated the row in between (bumping the version), the row count is 0, and Hibernate throws `OptimisticLockException` (wrapped as `ObjectOptimisticLockingFailureException` in Spring). No DB locks are held during the transaction — conflicts are only detected at commit/flush time.

**Q2: How does pessimistic locking work, and what are the lock modes?**

A: Pessimistic locking acquires an actual **database row lock** at read time, blocking other transactions from acquiring conflicting locks until the holder commits/rolls back.
- `PESSIMISTIC_READ` — shared lock (`SELECT ... FOR SHARE`), blocks writers, allows other readers.
- `PESSIMISTIC_WRITE` — exclusive lock (`SELECT ... FOR UPDATE`), blocks both readers-with-lock and writers.
- `PESSIMISTIC_FORCE_INCREMENT` — exclusive lock plus forces a version bump, even without a data change.

**Q3: When would you choose optimistic vs pessimistic locking?**

A:
- **Optimistic** — better for high-read, low-contention scenarios (most web apps); avoids holding DB locks and scales better, but requires handling retry/conflict logic on the failure path (catch the exception, reload, retry, or surface a conflict to the user).
- **Pessimistic** — better for high-contention, short critical sections where conflicts are frequent and retry logic is expensive or unsafe (e.g., decrementing limited inventory, financial ledger updates). Costs concurrency/throughput and risks deadlocks if lock ordering isn't consistent.

**Q4: What's a common pitfall with `@Version` and detached entities?**

A: If you send an entity's version number to a client and back (e.g., in a REST payload) and the client doesn't round-trip it correctly, you can accidentally bypass the optimistic check or trigger spurious conflicts. Also, manually incrementing/setting the version field yourself defeats the mechanism — let Hibernate manage it.

---

# PART 2: MICROSERVICES

## 9. API Gateway Design

**Q1: What responsibilities does an API Gateway typically own?**

A: Request routing to backend services, authentication/authorization (often via JWT validation or delegating to an auth service), rate limiting/throttling, request/response transformation, TLS termination, load balancing, aggregation (composing responses from multiple services for BFF-style APIs), and cross-cutting observability (correlation IDs, logging, metrics).

**Q2: What are the tradeoffs of putting business logic in the gateway?**

A: Gateways should stay thin — routing and cross-cutting concerns only. Putting business/domain logic there creates a hidden coupling point, makes the gateway a bottleneck for releases (every domain change requires a gateway change), and blurs ownership boundaries between teams. It's fine to do generic transformation/aggregation (BFF pattern) but domain rules belong in services.

**Q3: How do you avoid the gateway becoming a single point of failure or a scaling bottleneck?**

A: Run multiple gateway instances behind a load balancer (stateless design so any instance can serve any request), apply circuit breakers/timeouts on downstream calls so one slow service doesn't exhaust gateway threads, cache aggressively where safe, and keep the gateway logic lightweight (avoid synchronous fan-out chains that multiply latency).

**Q4: Gateway vs Service Mesh — how do responsibilities differ?**

A: An API Gateway sits at the **edge**, handling north-south traffic (external clients → services). A service mesh (e.g., Istio, Linkerd) handles **east-west** traffic (service-to-service), via sidecar proxies providing mTLS, retries, traffic shaping, and observability *between* internal services. They're complementary, not competing — many architectures use both.

---

## 10. Service Discovery

**Q1: Client-side vs server-side service discovery — explain both.**

A:
- **Client-side discovery** (e.g., Netflix Eureka + Ribbon) — the client queries a registry directly to get a list of healthy instances and picks one itself (often with client-side load balancing). Pro: fewer network hops. Con: discovery logic is coupled into every client/language.
- **Server-side discovery** (e.g., Kubernetes Services, AWS ALB) — the client calls a fixed endpoint/load balancer, which looks up the registry and routes the request. Pro: simpler clients, language-agnostic. Con: an extra network hop through the LB.

**Q2: How does Kubernetes handle service discovery differently from a tool like Eureka?**

A: Kubernetes uses **DNS-based discovery** — each `Service` gets a stable DNS name (`my-service.namespace.svc.cluster.local`) resolved by `kube-dns`/CoreDNS to a stable virtual IP (ClusterIP), and `kube-proxy` load-balances traffic to healthy pod endpoints via iptables/IPVS rules. There's no explicit client-side registry lookup — it's transparent at the network layer, unlike Eureka where the client actively queries a registry and often does client-side load balancing.

**Q3: What's the role of health checks in service discovery?**

A: Registries (or Kubernetes endpoint controllers) rely on health checks to keep the pool of "discoverable" instances accurate — a service must actively pass liveness/readiness checks to remain registered/routable. Without this, discovery would keep sending traffic to dead or overloaded instances, defeating the purpose.

---

## 11. Distributed Transactions (Saga Pattern)

**Q1: Why can't you just use a 2PC/XA distributed transaction across microservices?**

A: Two-phase commit requires all participants to hold locks and stay available until the coordinator finishes, which doesn't scale in a microservices world with independent deployability, different databases/technologies, and network partitions. It also creates tight temporal coupling — a slow or down service blocks everyone else's transaction. Sagas trade strict ACID atomicity for **eventual consistency** with explicit compensation.

**Q2: Explain the two Saga implementation styles: choreography vs orchestration.**

A:
- **Choreography** — each service publishes events after completing its local transaction; other services subscribe and react, publishing their own follow-up events. No central coordinator. Pro: fully decoupled, no single point of failure. Con: hard to trace/debug the overall flow as complexity grows ("where does this saga actually live?").
- **Orchestration** — a central orchestrator (e.g., a saga coordinator service, or a workflow engine like Camunda/Temporal) explicitly tells each service what to do next and handles compensation logic centrally. Pro: clear, testable, visible flow. Con: the orchestrator becomes a critical component and a potential bottleneck/coupling point.

**Q3: What is a compensating transaction, and what are its limits?**

A: A compensating transaction is a business-level "undo" operation that semantically reverses a previously committed local transaction (e.g., "cancel reservation" to compensate "reserve inventory") — it's not a true rollback since the original transaction already committed. Limits: compensations must be idempotent, must handle the case where the original action already had side effects visible to other systems (e.g., a payment already reached a third-party processor), and some actions genuinely can't be undone (e.g., an email already sent) — those require careful design (e.g., issuing a follow-up correction communication instead).

**Q4: How do you handle partial failure mid-saga?**

A: Track saga state explicitly (a state machine/table per saga instance) so you know exactly which steps completed. On failure, trigger compensations for already-completed steps in reverse order. Use durable messaging (outbox pattern, at-least-once delivery) so saga steps aren't lost if a service crashes mid-flow, and make every step and compensation idempotent since retries are inevitable.

---

## 12. Event-Driven Architecture

**Q1: What problem does event-driven architecture solve versus synchronous request/response between services?**

A: It decouples producers from consumers in **time** (consumer doesn't need to be up when the event is published) and in **knowledge** (producer doesn't need to know who consumes the event). This improves resilience (a downstream outage doesn't cascade back to the producer) and scalability (consumers can process at their own pace), at the cost of eventual consistency and more complex debugging/tracing.

**Q2: What is the Transactional Outbox pattern and why is it needed?**

A: It solves the "dual write" problem — updating your database *and* publishing a message atomically is not naturally guaranteed (if the DB commit succeeds but the message publish fails, or vice versa, you get inconsistency). The outbox pattern writes the event to an **outbox table in the same local DB transaction** as the business change, then a separate process (polling or CDC via tools like Debezium) reads the outbox and reliably publishes the event to the broker, marking it as sent. This guarantees at-least-once delivery consistent with the DB state.

**Q3: Event notification vs event-carried state transfer vs event sourcing — what's the difference?**

A:
- **Event notification** — a lightweight "something happened" signal (e.g., `OrderCreated{orderId}`); consumers call back to fetch details if needed. Minimal payload, but creates a runtime dependency back on the producer.
- **Event-carried state transfer** — the event carries the full relevant state (e.g., `OrderCreated{orderId, items, total, customer}`) so consumers can build local read models without calling back. Reduces coupling but increases payload size and duplication.
- **Event sourcing** — the event stream **is** the system of record; entity state is derived by replaying events, rather than storing current state directly. Enables full audit history and temporal queries but adds significant complexity (snapshotting, schema evolution of events, replay tooling).

**Q4: How do you handle event schema evolution without breaking consumers?**

A: Use a schema registry (e.g., Confluent Schema Registry) with compatibility rules (backward/forward compatible changes only — add optional fields, don't remove/rename required ones), version events explicitly when breaking changes are unavoidable, and favor additive changes. Consumers should tolerate unknown fields gracefully.

---

## 13. Idempotency

**Q1: Why is idempotency critical in distributed/microservices systems?**

A: Networks are unreliable — clients/brokers retry on timeouts even when the original request actually succeeded server-side. Without idempotency, retries can cause duplicate side effects (double-charging a payment, creating duplicate orders). Idempotency guarantees that processing the same logical request multiple times produces the same result as processing it once.

**Q2: How do you implement idempotency for a REST API?**

A: Require clients to send an **idempotency key** (a unique ID, often a UUID, in a header like `Idempotency-Key`) with mutating requests. The server stores a record of `(key → result)` — on receiving a request with a previously-seen key, it returns the stored result instead of reprocessing. Storage needs a reasonable TTL, and the check-and-store needs to be atomic (e.g., a unique constraint in the DB, or a distributed lock) to avoid race conditions on concurrent retries.

**Q3: How does idempotency apply to message consumers?**

A: Since most brokers offer **at-least-once** delivery, consumers must be able to handle duplicate messages safely. Techniques: track processed message IDs (dedup table with unique constraint), design handlers to be naturally idempotent (e.g., `UPSERT` instead of `INSERT`, `SET status = 'X'` instead of `increment`), or use idempotent database operations combined with the outbox's message ID as a natural dedup key.

**Q4: Is "idempotent" the same as "safe" (no side effects)? Give an example that clarifies the distinction.**

A: No — idempotent means repeating the operation produces the **same end state**, not that it has no side effects. `DELETE /orders/123` is idempotent (calling it once or five times leaves the order deleted) but not side-effect-free (it did delete something). `POST /orders` is neither idempotent nor safe by default. `GET /orders/123` is both safe and idempotent.

---

## 14. Retry Strategies

**Q1: What's wrong with naive immediate retries, and what's the alternative?**

A: Immediate, unbounded retries can cause a **retry storm** — amplifying load on an already-struggling downstream service and making the outage worse (or causing cascading failures upstream). Better approach: **exponential backoff with jitter** — increase the delay between attempts exponentially (e.g., 1s, 2s, 4s, 8s...) and add randomized jitter so many clients retrying simultaneously don't all hammer the service at the same instants ("thundering herd").

**Q2: What kinds of failures should you retry, and which should you not?**

A: Retry **transient** failures — timeouts, connection resets, 502/503/504, throttling responses (429). Don't retry **deterministic** failures — 400 (bad request), 401/403 (auth), 404, or business validation errors — since retrying an inherently-invalid request will just fail again and waste resources. Also cap total retry attempts/time and combine with a circuit breaker so retries stop entirely once a downstream is clearly down.

**Q3: How do retries interact with idempotency?**

A: Retries are only safe to perform automatically if the operation is idempotent (or made idempotent via an idempotency key) — otherwise a retry after a false-timeout (where the original request actually succeeded) can cause duplicate effects. This is why retry logic and idempotency design go hand-in-hand in resilient system design.

---

## 15. Circuit Breaker Patterns

**Q1: Explain the three states of a circuit breaker.**

A:
- **Closed** — requests flow normally; failures are counted/tracked (e.g., a rolling error-rate window).
- **Open** — once failures exceed a threshold, the circuit "trips" and further calls **fail fast** immediately without hitting the downstream service, for a configured wait duration. Protects the failing service from more load and protects the caller from wasting resources/threads on doomed calls.
- **Half-Open** — after the wait duration, a limited number of trial requests are let through. If they succeed, the circuit closes again; if they still fail, it re-opens.

**Q2: How does a circuit breaker differ from a timeout, and why do you need both?**

A: A timeout bounds how long you wait for *one* call. A circuit breaker tracks the **aggregate health** of a dependency across many calls and stops sending traffic altogether once it's clearly unhealthy. Timeouts alone don't prevent repeatedly retrying/hammering a downed service — you need the circuit breaker layered on top to actually stop the calls and let the downstream recover.

**Q3: What metrics/thresholds typically configure a circuit breaker (e.g., Resilience4j)?**

A: Failure rate threshold (e.g., open circuit if >50% of the last N calls failed), sliding window size/type (count-based or time-based), minimum number of calls before the threshold is evaluated (to avoid tripping on a tiny sample), wait duration in open state, and permitted calls in half-open state.

**Q4: How does a circuit breaker interact with fallback logic?**

A: When the circuit is open (or a call fails), you typically define a **fallback** — a degraded-but-safe response (cached data, a default value, a "service temporarily unavailable" response) instead of propagating the failure raw. This is what actually delivers graceful degradation to the end user rather than just failing fast internally.

---

## 16. Observability and Monitoring

**Q1: What are the three pillars of observability, and what does each tell you?**

A:
- **Metrics** — numeric, aggregatable time series (latency percentiles, error rate, throughput, resource usage). Good for dashboards, alerting, and trend detection, but low on per-request detail.
- **Logs** — discrete, timestamped, often unstructured or semi-structured events. Good for detailed forensic debugging of a specific incident.
- **Traces** — track a single request's journey across service boundaries, showing where time was spent and where failures occurred (distributed tracing, e.g., via OpenTelemetry, Jaeger, Zipkin). Essential for diagnosing latency/failure in a multi-hop microservices call chain.

**Q2: How does distributed tracing actually correlate a request across services?**

A: A **trace ID** is generated at the entry point (e.g., the gateway) and propagated via headers (commonly W3C Trace Context `traceparent` header, or older B3 headers) to every downstream call. Each service creates **spans** (a unit of work) tagged with that trace ID and a parent span ID, forming a tree/DAG that a tracing backend (Jaeger/Zipkin/Tempo) can reconstruct into a full waterfall view of the request.

**Q3: What's the difference between monitoring and observability?**

A: Monitoring is about watching **known** failure modes via predefined dashboards/alerts ("is CPU > 80%?"). Observability is about being able to answer **novel, unanticipated** questions about system behavior from the data you've collected, without having to ship new code to add instrumentation for that specific question — achieved through rich, high-cardinality structured data (logs/traces/metrics with good labels) that supports ad hoc querying.

**Q4: In a Spring Boot app, what's the typical toolchain for observability?**

A: **Micrometer** as the metrics facade (vendor-neutral instrumentation API) exporting to **Prometheus** (pull-based scraping) visualized in **Grafana**; **Spring Boot Actuator** for health/info/metrics endpoints; **OpenTelemetry** (or Sleuth historically) for distributed tracing exported to **Jaeger/Zipkin/Tempo**; structured JSON logging shipped to an aggregator like the **ELK stack** or **Loki**.

---

# PART 3: DOCKER

## 17. Multi-stage Builds

**Q1: What problem do multi-stage builds solve?**

A: Without them, your final image often carries build-time tooling (JDK, compilers, package managers, source code) that bloats the image and increases attack surface. Multi-stage builds let you use one stage with full build tooling to compile/build the artifact, then **copy only the final artifact** into a clean, minimal runtime stage — discarding everything else.

**Q2: Give an example multi-stage Dockerfile for a Spring Boot app.**

A:
```dockerfile
# Stage 1: build
FROM maven:3.9-eclipse-temurin-21 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn package -DskipTests

# Stage 2: runtime
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```
The final image only contains a JRE and the built jar — no Maven, no source code.

**Q3: How can you further optimize layer caching in a multi-stage build?**

A: Order instructions from least-to-most frequently changing: copy `pom.xml`/`package.json` and run dependency resolution **before** copying the full source tree. This way, Docker's layer cache reuses the (often slow) dependency-download layer as long as dependencies haven't changed, even if application code has.

---

## 18. Image Optimization

**Q1: What are the main levers for reducing Docker image size?**

A:
- Use **minimal base images** (alpine, distroless, `slim` variants) instead of full OS images.
- **Multi-stage builds** to exclude build tooling from the final image.
- Combine `RUN` commands with `&&` and clean up package manager caches in the same layer (`apt-get clean && rm -rf /var/lib/apt/lists/*`) — because each `RUN` creates a layer, and deleting files in a *later* layer doesn't shrink earlier layers.
- Use `.dockerignore` to avoid copying unnecessary files (`.git`, `node_modules`, build artifacts, tests) into the build context/image.
- For JVM apps, consider **jlink**/custom minimal JRE, or GraalVM native-image for drastically smaller/faster-starting images.

**Q2: What's the tradeoff of using `distroless` or `scratch` base images?**

A: They minimize attack surface and size (no shell, no package manager, minimal OS libraries), which is great for security and speed. The tradeoff is **debuggability** — you can't `docker exec` into a shell to poke around, and you lose common utilities. You typically pair this with good external observability (logs/metrics/tracing) rather than relying on live shell debugging.

**Q3: Why does layer ordering matter for both size and build speed?**

A: Docker caches each layer; a layer is invalidated (and everything after it rebuilt) if its inputs change. Placing rarely-changing instructions (installing OS deps, resolving dependencies) early and frequently-changing instructions (copying app source) late maximizes cache hits, speeding up rebuilds — it doesn't reduce final image size directly, but reduces wasted build time/CI cost.

---

## 19. Container Networking

**Q1: What are Docker's built-in network drivers, and when do you use each?**

A:
- **bridge** (default) — creates an isolated private network on the host; containers on the same bridge can reach each other by container name (with a user-defined bridge; the default `bridge` network requires `--link` or IP for name resolution — user-defined bridges get automatic DNS).
- **host** — container shares the host's network namespace directly, no isolation/port mapping needed, but only one container can bind a given port; used for max performance / when you need the container to appear as the host on the network.
- **overlay** — spans multiple Docker hosts (Swarm/multi-host setups), enabling containers on different machines to communicate as if on the same network.
- **none** — no networking at all, fully isolated.
- **macvlan** — assigns a container its own MAC/IP on the physical network, making it appear as a distinct physical device.

**Q2: How does container-to-container name resolution work on a user-defined bridge network?**

A: Docker runs an embedded DNS server (at 127.0.0.11 inside containers) for user-defined bridge networks, automatically resolving other containers' names (and any network aliases) to their internal IPs — this is why `docker-compose` services can reach each other simply via `http://service-name:port` without manual IP management.

**Q3: How does port publishing (`-p 8080:80`) actually work under the hood?**

A: Docker sets up **iptables NAT rules** (on Linux) that DNAT traffic arriving on the host's port 8080 to the container's internal IP on port 80. This is why a published port is reachable from outside the host, whereas an unpublished container port is only reachable from other containers on the same Docker network.

---

## 20. Volumes and Bind Mounts

**Q1: What's the difference between a named volume, a bind mount, and tmpfs?**

A:
- **Named volume** — managed entirely by Docker, stored under Docker's storage area (`/var/lib/docker/volumes/...` on Linux), portable across environments, and the recommended way to persist container data (databases, uploaded files). Docker handles the lifecycle and you interact with it via `docker volume` commands.
- **Bind mount** — maps a specific **host filesystem path** into the container. Useful for local development (mounting source code for hot-reload) but ties you to the host's filesystem layout, and is less portable/manageable than named volumes.
- **tmpfs mount** — stored in host memory only, never persisted to disk; useful for sensitive, short-lived data (avoids leaving secrets on disk) but lost when the container stops.

**Q2: Why do database containers typically use named volumes rather than storing data in the container's writable layer?**

A: The container's writable layer is ephemeral and tied to that specific container instance — if the container is removed (not just stopped), that data is lost. A named volume exists independently of any specific container lifecycle, so you can recreate/upgrade the container (e.g., pull a new image version) while the data persists and gets reattached.

**Q3: What's a common pitfall with bind mounts and file permissions?**

A: The container process often runs as a different UID than your host user, so files created inside the container via a bind mount can end up owned by a UID that's awkward to access/edit on the host (or vice versa) — commonly solved by matching UIDs (`--user $(id -u):$(id -g)`), or configuring the image to run as a specific non-root UID that aligns with expectations.

---

## 21. Docker Compose for Local Development

**Q1: What core benefits does Compose provide over a set of manual `docker run` commands?**

A: Declarative, version-controllable definition of multi-container topology (services, networks, volumes, env vars, dependencies) in a single YAML file; one-command bring-up/tear-down (`docker compose up`/`down`); automatic creation of a shared network with DNS-based service discovery by service name; easy environment-specific overrides via multiple compose files (`docker-compose.override.yml`).

**Q2: How do you handle startup ordering when one service depends on another (e.g., app depends on DB being ready)?**

A: `depends_on` only controls **container start order**, not "readiness" (the DB container starting doesn't mean Postgres is actually accepting connections yet). Use `depends_on` with a `condition: service_healthy`, paired with a `healthcheck` defined on the dependency, so Compose waits for the DB to report healthy before starting the dependent service. Alternatively, build retry/backoff logic into the app's own DB connection startup.

**Q3: Give a minimal example combining an app and a Postgres DB with a healthcheck.**

A:
```yaml
services:
  app:
    build: .
    ports: ["8080:8080"]
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/appdb
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: appdb
      POSTGRES_PASSWORD: secret
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
```

---

## 22. Security Best Practices

**Q1: What are the key Docker security best practices for production images?**

A:
- **Run as non-root** — set a `USER` directive in the Dockerfile instead of defaulting to root, minimizing blast radius if the container is compromised.
- **Use minimal, trusted base images** — official/verified images, pinned by digest (not just `latest`) for reproducibility, and minimal (alpine/distroless) to reduce attack surface and CVE count.
- **Scan images** for known vulnerabilities (Trivy, Grype, Docker Scout) as part of CI.
- **Don't bake secrets into images** (they persist in image layers/history even if "deleted" in a later layer) — inject secrets at runtime via env vars, mounted secret files, or a secrets manager.
- **Set resource limits** (`--memory`, `--cpus`) to prevent a single container from starving the host.
- **Use read-only root filesystems** (`--read-only`) where possible, with explicit writable volumes only where needed.
- **Drop unnecessary Linux capabilities** (`--cap-drop=ALL`, add back only what's needed) rather than running fully privileged.

**Q2: Why is `docker run --privileged` dangerous, and when (rarely) is it justified?**

A: `--privileged` gives the container nearly all capabilities of the host kernel, effectively breaking container isolation — a compromised privileged container can often escape to the host. It's occasionally needed for things like Docker-in-Docker or direct hardware/device access, but should be avoided in production workloads; prefer granting specific capabilities via `--cap-add` instead of the blanket flag.

**Q3: How do you avoid leaking secrets through Docker build layers?**

A: Avoid `COPY`ing secret files or passing secrets via `ARG`/`ENV` in a way that lands in the image history (even in an intermediate stage, since layers can be inspected). Use **BuildKit's `--secret` mount** (`RUN --mount=type=secret,id=mysecret ...`), which makes the secret available only during that specific `RUN` step and never persists it in a layer.

---

# PART 4: KUBERNETES

## 23. Pod Lifecycle

**Q1: Walk through the phases a Pod goes through.**

A:
- **Pending** — Pod accepted by the API server but not yet scheduled, or scheduled but images still being pulled/containers starting.
- **Running** — at least one container is running (or starting/restarting), and the Pod has been bound to a node.
- **Succeeded** — all containers terminated successfully (exit code 0), and won't be restarted (typical for Jobs).
- **Failed** — all containers terminated, and at least one exited with failure.
- **Unknown** — state can't be determined, usually due to a node communication failure.

**Q2: What's the role of init containers in the Pod lifecycle?**

A: Init containers run sequentially **before** any app containers start, and each must complete successfully before the next starts (and before app containers start). Used for setup tasks — waiting for a dependency to be ready, running DB migrations, fetching config/secrets, setting up file permissions on a shared volume — keeping that logic separate from the main app container's image/runtime.

**Q3: What happens during Pod termination (the "graceful shutdown" sequence)?**

A:
1. Pod is marked "Terminating"; it's immediately removed from Service endpoints (so no new traffic is routed to it).
2. `preStop` hook (if defined) is executed.
3. `SIGTERM` is sent to the container's main process.
4. The container has until `terminationGracePeriodSeconds` (default 30s) to shut down cleanly.
5. If it hasn't exited by then, Kubernetes sends `SIGKILL`.

A common pitfall: if your app doesn't handle `SIGTERM` to drain in-flight requests (and/or you don't have a `preStop` hook delaying just long enough for the endpoint removal to propagate), you get dropped connections during rolling deploys.

**Q4: What's the difference between a container restart and a Pod eviction/rescheduling?**

A: A **restart** (governed by `restartPolicy`) happens in place on the same node — same Pod, same IP, kubelet just restarts the failed container process. **Eviction/rescheduling** happens at the Pod level — the whole Pod is deleted (e.g., due to node pressure, node failure, or a controller like the Deployment reconciling desired state) and a **new** Pod (new name, likely new IP) is created, potentially on a different node.

---

## 24. Scheduling and Taints/Tolerations

**Q1: How does the default Kubernetes scheduler decide where to place a Pod?**

A: Two phases:
1. **Filtering** — eliminate nodes that don't meet hard constraints (insufficient CPU/memory, node selectors/affinity rules not matching, taints without matching tolerations, port conflicts, volume zone constraints).
2. **Scoring** — rank remaining feasible nodes using priority functions (e.g., spreading pods to balance resource usage, preferring nodes with the requested image already cached, honoring inter-pod affinity/anti-affinity preferences) and pick the highest-scoring node.

**Q2: Explain taints and tolerations, and how they differ from node affinity.**

A: A **taint** on a node (`kubectl taint nodes node1 key=value:NoSchedule`) repels Pods from being scheduled there **unless** the Pod has a matching **toleration**. This is an *opt-in* mechanism from the Pod's side to be allowed onto otherwise-restricted nodes (e.g., dedicated GPU nodes, nodes reserved for a specific team). **Node affinity**, by contrast, is the Pod expressing a *preference or requirement* to be scheduled on nodes with certain labels — it's the inverse direction of control: taints let nodes reject pods by default; affinity lets pods attract themselves to nodes. They're often combined (taint the special nodes, then use both a toleration and an affinity rule on the pods meant to land there, so only intended pods go there and unrelated pods don't get randomly excluded).

**Q3: What are the three taint effects?**

A:
- `NoSchedule` — new Pods without a matching toleration won't be scheduled; existing Pods already running are unaffected.
- `PreferNoSchedule` — scheduler tries to avoid placing Pods there but isn't guaranteed to.
- `NoExecute` — new Pods won't be scheduled **and** existing Pods without a matching toleration are evicted. Often used for node problem conditions (e.g., `node.kubernetes.io/not-ready`).

**Q4: What's the difference between `requests` and `limits` in Pod resource specs, and how does the scheduler use them?**

A: The scheduler uses **requests** to decide node placement (it only considers nodes with enough allocatable capacity to satisfy the sum of requests of all pods, including the new one). **Limits** are enforced at runtime by the kubelet/container runtime as a hard cap (CPU gets throttled beyond its limit; memory beyond its limit gets the container OOMKilled) — they don't factor into scheduling decisions directly, only requests do.

---

## 25. Health Probes

**Q1: Explain the three probe types and what each is actually for.**

A:
- **Liveness probe** — "is this container alive/healthy, or should it be restarted?" If it fails repeatedly, kubelet kills and restarts the container. Use for detecting deadlocks/hangs where the process is technically running but stuck.
- **Readiness probe** — "is this container ready to accept traffic right now?" If it fails, the Pod is removed from the Service's endpoint list (no new traffic routed) but the container is **not** restarted — useful for temporary unavailability (warming up a cache, waiting on a downstream dependency) where killing the process would be wrong.
- **Startup probe** — used for slow-starting containers; disables liveness/readiness checks until the startup probe succeeds, preventing kubelet from prematurely killing a container that just needs more time to boot (avoids having to set an overly generous liveness `initialDelaySeconds` for all cases).

**Q2: What's a common misconfiguration mistake with liveness probes?**

A: Making the liveness probe check downstream dependencies (e.g., DB connectivity) — if the database goes down, this causes **every** app Pod's liveness probe to fail simultaneously, triggering mass restarts of pods that are otherwise perfectly healthy, which doesn't fix the actual problem and can create restart storms. Downstream dependency checks belong in the **readiness** probe (so the pod is taken out of rotation) — liveness should only check "is my own process fundamentally stuck/broken."

**Q3: What probe mechanisms are available?**

A: `httpGet` (expects 2xx/3xx response), `tcpSocket` (checks if a port accepts connections), `exec` (runs a command inside the container, success = exit code 0), and `grpc` (checks a gRPC health-checking-protocol endpoint).

---

## 26. Scaling

**Q1: How does the Horizontal Pod Autoscaler (HPA) work?**

A: HPA periodically (default every 15s) queries metrics (via the metrics-server for CPU/memory, or custom/external metrics via adapters for things like queue depth or request rate) and compares current utilization to a target (e.g., target 50% CPU). It computes desired replica count roughly as `desiredReplicas = ceil(currentReplicas * (currentMetricValue / desiredMetricValue))`, then updates the Deployment/ReplicaSet's replica count accordingly, subject to min/max bounds and stabilization windows to avoid flapping.

**Q2: Horizontal vs Vertical Pod Autoscaling — when do you use each, and can you combine them?**

A: **HPA** adds/removes Pod replicas — good for stateless workloads that can scale out. **VPA** adjusts a Pod's CPU/memory **requests/limits** over time based on observed usage — good for right-sizing workloads that can't easily scale horizontally, or for setting sane initial resource requests. They can conflict if both target CPU on the same workload (VPA changing requests can confuse HPA's utilization percentage), so combining them on the same metric needs care — typically VPA on memory + HPA on CPU, or VPA in "recommendation only" mode.

**Q3: What's Cluster Autoscaler, and how does it relate to HPA?**

A: HPA/VPA scale workloads assuming there's node capacity available. **Cluster Autoscaler** scales the **nodes themselves** — adding nodes when Pods are unschedulable due to insufficient capacity, and removing underutilized nodes (after trying to safely evict/reschedule their Pods elsewhere) to save cost. They work together: HPA increases replica count → if no node has room → Cluster Autoscaler provisions a new node → scheduler places the pending Pods.

**Q4: What are common pitfalls that make HPA ineffective?**

A: Not setting resource **requests** on containers (HPA's CPU/memory percentage calculations are relative to requests — without them, HPA can't compute utilization meaningfully). Overly aggressive scale-down causing flapping (mitigated via `stabilizationWindowSeconds` in the scaling behavior policy). Scaling on a metric that doesn't actually reflect load (e.g., CPU for an I/O-bound service that's actually bottlenecked on queue depth or DB connections — better to scale on a custom metric there).

---

## 27. Rolling Updates and Rollbacks

**Q1: How does a Deployment perform a rolling update by default?**

A: The Deployment controller creates a **new ReplicaSet** for the new Pod template and gradually scales it up while scaling the old ReplicaSet down, governed by `maxSurge` (how many extra Pods above desired count can be created during the rollout) and `maxUnavailable` (how many Pods can be unavailable during the rollout). This achieves zero-downtime deploys as long as readiness probes correctly gate when new Pods start receiving traffic.

**Q2: How do you roll back a bad deployment?**

A: `kubectl rollout undo deployment/my-app` reverts to the previous ReplicaSet revision (Kubernetes keeps a bounded history via `revisionHistoryLimit`). You can target a specific revision with `--to-revision=N`, and inspect history with `kubectl rollout history deployment/my-app`. This works because old ReplicaSets (scaled to 0, not deleted) retain the previous Pod template, so rollback just reverses the scale-up/scale-down direction.

**Q3: What's the difference between `RollingUpdate` and `Recreate` deployment strategies?**

A: `RollingUpdate` (default) replaces Pods incrementally for zero/minimal downtime, but means old and new versions run **simultaneously** for a period — this requires your change to be backward/forward compatible (e.g., DB schema changes, API contracts). `Recreate` kills **all** old Pods before creating new ones — guarantees no version overlap (useful when old and new versions genuinely can't coexist, e.g., certain stateful/singleton workloads) but causes downtime during the switch.

**Q4: How do readiness probes and rollout status interact to prevent a bad rollout from fully replacing good Pods?**

A: `kubectl rollout status` watches whether new Pods actually become **ready** (per their readiness probe) before the rollout is considered progressing successfully. If new Pods keep crash-looping or failing readiness, the rollout stalls (respecting `maxUnavailable`, it won't tear down more old Pods than allowed), and `progressDeadlineSeconds` will eventually mark the rollout as failed if it doesn't make progress — giving you a signal to intervene/rollback before the whole fleet is replaced by a broken version.

---

## 28. Secrets Management

**Q1: How does Kubernetes store Secrets by default, and why is that not fully secure on its own?**

A: Secrets are stored in **etcd**, and by default etcd data is only **base64-encoded**, not encrypted — base64 is an encoding, not encryption, so anyone with etcd access (or an etcd backup) can trivially decode secret values. Mitigations: enable **encryption at rest** for etcd (`EncryptionConfiguration` with a KMS provider), restrict RBAC access to Secret objects tightly, and use a proper external secrets manager for sensitive production data.

**Q2: What are common ways to inject Secrets into Pods, and what are the tradeoffs?**

A:
- **Env vars** (`envFrom`/`valueFrom.secretKeyRef`) — simple, but env vars can leak via process listings (`/proc/[pid]/environ`), crash dumps, or accidental logging.
- **Mounted volumes** — Secret data appears as files in a tmpfs-backed volume; generally preferred since file permissions are more controllable and it avoids some of the env-var leak vectors, and updates to the Secret can be picked up without a Pod restart (with some propagation delay).
- **External secrets managers** (HashiCorp Vault, AWS Secrets Manager, GCP Secret Manager) via an operator/CSI driver (e.g., External Secrets Operator, Secrets Store CSI Driver) — Secrets are fetched dynamically at runtime rather than stored as native K8s Secret objects at all, giving better auditing, rotation, and centralized access control.

**Q3: What RBAC/access-control practices matter for Secrets specifically?**

A: Scope `get`/`list`/`watch` on `secrets` narrowly via RBAC — `list`/`watch` on secrets is especially sensitive since it can expose *all* secret values in a namespace, not just ones a service account specifically needs. Use separate namespaces to isolate blast radius, and avoid mounting the default service account token into Pods that don't need API server access (`automountServiceAccountToken: false`).

**Q4: How do you handle Secret rotation without downtime?**

A: Mounted-volume Secrets update automatically (kubelet syncs periodically), but the **application** needs to actually re-read the file rather than caching the value in memory at startup — either via a file-watcher/hot-reload, or by relying on a rolling restart triggered by your deployment pipeline when the Secret changes (e.g., checksumming the Secret content into a Pod annotation so a change triggers a new rollout, a common pattern with Helm).

---

## 29. Networking (Services, Ingress)

**Q1: What are the main Service types, and when do you use each?**

A:
- **ClusterIP** (default) — internal-only virtual IP, reachable within the cluster. Used for internal service-to-service communication.
- **NodePort** — exposes the service on a static port on every node's IP, reachable from outside the cluster via `<NodeIP>:<NodePort>`. Mostly used for dev/testing or when there's no cloud load balancer integration.
- **LoadBalancer** — provisions an external cloud load balancer (on supported cloud providers) that routes to the service; the standard way to expose a service externally in cloud environments.
- **ExternalName** — maps a Service to an external DNS name (CNAME-style), useful for referring to external systems via internal service names.

**Q2: How does a Kubernetes Service actually route traffic to the right Pods?**

A: A Service selects Pods via a **label selector**; the Endpoints/EndpointSlice controller keeps an up-to-date list of matching, ready Pod IPs. `kube-proxy` (running on every node) programs the routing rules — either **iptables** (random selection among matching endpoints, via chained probabilistic DNAT rules) or **IPVS** (a proper in-kernel load balancer supporting round-robin, least-connection, etc., more efficient at large scale) — that intercept traffic destined for the Service's ClusterIP and redirect it to a chosen Pod IP.

**Q3: What does Ingress add on top of Services, and why can't you just use a LoadBalancer Service for everything?**

A: A `LoadBalancer` Service typically provisions **one external load balancer per Service**, and only does L4 routing (no path/host-based rules). **Ingress** is an L7 (HTTP/HTTPS) routing layer — a single entry point (often a single external LB) that routes to many backend Services based on host name and URL path, plus handles TLS termination, and can integrate features like rewrites/redirects/rate limiting depending on the Ingress controller (NGINX Ingress, Traefik, AWS ALB Ingress Controller, etc.). Using one LoadBalancer per Service for dozens of services would be expensive and unwieldy — Ingress consolidates this.

**Q4: What's the difference between an Ingress resource and an Ingress Controller?**

A: The **Ingress resource** is just a declarative Kubernetes API object describing desired routing rules (host/path → service mappings, TLS config) — it does nothing by itself. The **Ingress Controller** is the actual running component (a Pod/Deployment, e.g., ingress-nginx) that watches Ingress resources and configures a real proxy/load balancer to implement those rules. You must have a controller installed for Ingress resources to have any effect.

**Q5: What's the emerging alternative to Ingress, and why was it introduced?**

A: The **Gateway API** — a newer, more expressive and role-oriented successor to Ingress, designed to address Ingress's limitations (heavy reliance on controller-specific annotations for anything beyond basic routing, no clean way to model shared infrastructure across teams). Gateway API separates concerns into distinct resource types (`GatewayClass`, `Gateway`, `HTTPRoute`, etc.) mapped to different personas (infra admins vs application developers), and supports richer native L4/L7 routing semantics without vendor-specific annotation hacks.

---

## Quick-Reference Cheat Sheet

| Topic | One-line takeaway |
|---|---|
| Spring Boot startup | Auto-config + `refresh()` drives conditional bean registration and embedded server bootstrap |
| Bean scopes | Singleton default; use `ObjectProvider`/scoped proxy to safely mix prototype into singleton |
| Autowiring | Type → Qualifier/name → Primary; prefer constructor injection |
| Tx propagation | REQUIRED for normal joins, REQUIRES_NEW for independent side effects, NESTED for savepoints |
| JPA internals | Dynamic proxies + `PartTree` query derivation; watch for N+1 |
| Hibernate caching | L1 always on (session-scoped); L2 opt-in and needs a distributed backend in multi-instance deployments |
| Lazy vs eager | Default lazy for collections, eager for `@ManyToOne`/`@OneToOne` — fetch deliberately per query instead |
| Locking | Optimistic for low contention (version column); pessimistic for high contention (row locks) |
| API Gateway | Edge concerns only — routing, auth, rate limiting; not business logic |
| Service discovery | Client-side (Eureka) vs server-side (K8s DNS + kube-proxy) |
| Saga | Choreography (decoupled, hard to trace) vs orchestration (central, easier to reason about) |
| Event-driven | Outbox pattern solves the dual-write problem |
| Idempotency | Idempotency keys + dedup storage; UPSERT over INSERT |
| Retries | Exponential backoff + jitter; only retry transient failures |
| Circuit breaker | Closed → Open (fail fast) → Half-Open (probe) |
| Observability | Metrics (trends), logs (forensics), traces (cross-service journey) |
| Multi-stage builds | Build in one stage, ship only the artifact in a minimal runtime stage |
| Image optimization | Minimal base + `.dockerignore` + combined `RUN` layers |
| Container networking | User-defined bridge gives free DNS-based service discovery |
| Volumes | Named volumes for persistence, bind mounts for dev, tmpfs for secrets-in-memory |
| Compose | `depends_on` + `condition: service_healthy` for real readiness ordering |
| Docker security | Non-root user, minimal base, scan images, never bake secrets into layers |
| Pod lifecycle | SIGTERM → grace period → SIGKILL; readiness removes from Service before shutdown |
| Scheduling | Filter then score; taints repel, affinity attracts |
| Probes | Liveness = restart if stuck; readiness = pull from traffic if not ready; never check downstream deps in liveness |
| Scaling | HPA scales pods, VPA resizes pods, Cluster Autoscaler scales nodes |
| Rolling updates | maxSurge/maxUnavailable control the transition; rollback reuses the old ReplicaSet |
| Secrets | Base64 ≠ encryption; enable etcd encryption at rest; prefer external secrets managers |
| Networking | ClusterIP internal, LoadBalancer external L4, Ingress external L7 with host/path routing |

---

*Tip for interviews: for every answer, be ready to follow up with a real example from a project you've worked on — interviewers weight "I've actually debugged this" answers much higher than textbook definitions.*
