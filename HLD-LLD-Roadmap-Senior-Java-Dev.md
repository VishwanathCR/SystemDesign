# HLD & LLD Roadmap for a Senior Java Developer (10+ YOE)

This roadmap assumes you already know Java well and have built production systems. It's not a "learn to code" plan — it's a **design maturity** plan: turning implicit intuition into explicit, interview-ready and architecture-review-ready reasoning.

Total duration: **10–12 weeks**, ~6–8 hrs/week. Adjust pace freely — you can compress this to 6 weeks if grinding daily.

---

## How This Roadmap Is Structured

- **Track A: LLD (Low-Level Design)** — object-oriented design, design patterns, concurrency, clean API design, class diagrams
- **Track B: HLD (High-Level Design)** — distributed systems, scalability, databases, infra tradeoffs
- Run them **in parallel**, not sequentially — LLD in first half of each week, HLD in second half. They reinforce each other (e.g., a rate limiter is both an LLD class-design exercise and an HLD building block).

Each week has: **Concepts → Practice Problem → Self-Check Questions**.

---

## Track A: LLD Roadmap

### Week 1 — OOP Foundations You Actually Need to Defend
Not "what is inheritance" — but *when does it hurt you in production*.
- SOLID principles, with real counter-examples (over-applying DIP, premature abstraction)
- Composition vs inheritance — decision framework
- Interfaces vs abstract classes vs sealed classes (Java 17+)
- Immutability, defensive copying, builder pattern for complex objects
- Encapsulation at the *module* level (Java modules / package-private discipline), not just field-level

**Practice:** Design a `Library Management System` — books, members, holds, fines. Focus on where you'd change behavior later without touching existing classes.

**Self-check:** Can you explain a time SOLID *slowed you down* and how you course-corrected?

---

### Week 2 — Design Patterns, by Problem Not by Name
Memorizing 23 GoF patterns is useless. Learn them as answers to recurring problems:

| Problem | Pattern(s) |
|---|---|
| Too many constructor params / optional config | Builder |
| Need interchangeable algorithms at runtime | Strategy |
| Object creation logic getting messy | Factory / Abstract Factory |
| Need to react to state changes | Observer |
| Wrapping external APIs without polluting your domain | Adapter, Facade |
| Adding behavior without subclass explosion | Decorator |
| Expensive object creation, reuse instances | Flyweight, Object Pool |
| Undo/redo, transactional operations | Command, Memento |
| State-dependent behavior (order lifecycle, etc.) | State |
| Traversing collections uniformly | Iterator |

**Practice:** Design a `Notification System` (Email/SMS/Push) using Strategy + Factory + Observer. Then design a `Payment Gateway Integration Layer` using Adapter + Facade.

**Self-check:** For each pattern, name a real system where using it would've been *wrong* — patterns aren't free.

---

### Week 3 — Concurrency & Thread-Safety in Design
This is where senior candidates separate from mid-level ones.
- `java.util.concurrent` deep dive: `ExecutorService`, `CompletableFuture`, `ConcurrentHashMap`, `CountDownLatch`, `Semaphore`
- Designing thread-safe singletons, caches, connection pools
- Lock-free structures, CAS, `AtomicInteger`/`AtomicReference`
- Producer-consumer, thread pools sizing (CPU-bound vs I/O-bound)
- Virtual threads (Java 21+) — when they change your concurrency design

**Practice:** Design an in-memory `Rate Limiter` (token bucket / sliding window) that's thread-safe under high concurrency. Then a thread-safe `LRU Cache` from scratch (no `LinkedHashMap` shortcut — implement with `HashMap` + doubly linked list).

**Self-check:** Explain why you chose `synchronized` vs `ReentrantLock` vs lock-free for a specific case.

---

### Week 4 — Classic LLD Interview/Design Problems (Round 1)
Do these **end-to-end**: requirements clarification → class diagram → core interfaces → key methods → edge cases.
- Parking Lot System
- Elevator System
- Splitwise / Expense Sharing App
- Chess Game

For each: identify entities, define relationships (composition/aggregation), think about extensibility (new floor type, new payment mode) without rewrites.

**Self-check:** Could a new engineer extend your design without reading your mind?

---

### Week 5 — Classic LLD Problems (Round 2) + API Design
- Design a `Cab Booking System` (Uber-lite)
- Design a `Movie Ticket Booking System` (BookMyShow-lite) — handle seat locking/concurrency
- Design a `Logging Framework` (like a mini Log4j)
- Design a `Rate Limiter as a library` (pluggable strategies)

Also cover:
- Idempotent API design, versioning strategies
- Exception hierarchy design (checked vs unchecked, domain exceptions)
- Fluent/chainable API design principles

**Self-check:** In the ticket booking system, how do you prevent two users from double-booking the same seat? (This bridges into HLD — distributed locking.)

---

## Track B: HLD Roadmap

### Week 1 — Scalability Fundamentals
- Vertical vs horizontal scaling, stateless vs stateful services
- Load balancing algorithms (round robin, least connections, consistent hashing) and where each breaks down
- Back-of-the-envelope estimation: QPS, storage, bandwidth math — practice doing this fast
- Latency numbers every engineer should know (memory vs disk vs network)

**Practice:** Estimate capacity for a URL shortener handling 100M URLs/month, 10:1 read:write ratio.

---

### Week 2 — Databases at Scale
- SQL vs NoSQL — real decision criteria, not dogma
- Indexing strategies, query optimization mental models
- Sharding strategies (range, hash, geo) and resharding pain
- Replication: leader-follower, multi-leader, quorum-based
- CAP theorem and PACELC — apply it to real systems (not just recite it)
- Database connection pooling at scale (relates to your Java `HikariCP` experience)

**Practice:** Design the data layer for a `Ride-sharing App` — decide SQL vs NoSQL per data type (user profiles vs live location vs trip history).

---

### Week 3 — Caching, CDNs, and Consistency
- Cache-aside, write-through, write-behind strategies
- Cache invalidation strategies (the "two hard things in CS" joke, but concretely)
- Redis/Memcached use cases, eviction policies
- CDN basics, edge caching
- Consistency models: strong, eventual, causal — pick based on business requirement, not preference

**Practice:** Design a `News Feed System` (Twitter-lite) — fan-out on write vs fan-out on read tradeoff, caching hot users.

---

### Week 4 — Messaging, Queues & Async Architecture
- Message queues (Kafka, RabbitMQ, SQS) — when to use pub-sub vs point-to-point
- At-least-once vs exactly-once vs at-most-once delivery semantics
- Event-driven architecture, event sourcing, CQRS — when they're worth the complexity
- Idempotency and deduplication in distributed consumers
- Dead-letter queues, retry strategies, backoff

**Practice:** Design a `Notification Delivery System` at scale (millions of push/email/SMS) using queues, with retry and dedup logic.

---

### Week 5 — Microservices, APIs & Resilience
- Monolith vs microservices — honest tradeoffs (you likely have war stories; formalize them)
- Service discovery, API gateways, BFF pattern
- Circuit breakers, bulkheads, timeouts, retries (Resilience4j — you likely know this from Spring)
- Distributed tracing, observability (metrics, logs, traces — the three pillars)
- Saga pattern for distributed transactions (orchestration vs choreography)

**Practice:** Design the checkout flow for an `E-commerce Order System` — inventory service, payment service, order service — handle partial failures with Saga.

---

### Week 6 — Distributed Systems Deep Cuts
- Distributed locking (Redis Redlock, ZooKeeper) — ties back to your seat-booking LLD problem
- Consensus basics: Paxos/Raft (conceptual level — you don't need to implement it, but must reason about it)
- Rate limiting at the system level (API gateway layer) vs library level (Week 3 LLD)
- Leader election
- Idempotency keys in distributed writes

**Practice:** Revisit the Movie Ticket Booking LLD from Track A Week 5 — now solve seat-locking at HLD scale using distributed locks + TTL-based holds.

---

## Weeks 7–10: Full System Design Mock Rounds

Combine HLD + LLD in single end-to-end sessions (45–60 min each, simulate interview or architecture review conditions):

1. **Design WhatsApp/Messaging System** — HLD: message delivery, presence, group chat fan-out. LLD: message queue per user, delivery acknowledgment state machine.
2. **Design a Distributed Rate Limiter as a Service** — HLD: sliding window across nodes using Redis. LLD: token bucket class design, thread safety.
3. **Design an Online Code Judge (LeetCode-style)** — HLD: sandboxed execution, queueing, scaling judges. LLD: submission state machine, plugin architecture for languages.
4. **Design a Ride-Sharing Dispatch System** — HLD: geospatial indexing (quad-tree/geohash), matching algorithm at scale. LLD: driver-rider matching class design.
5. **Design a Distributed Job Scheduler** (like Quartz but distributed) — HLD: leader election, partition ownership. LLD: job/trigger class hierarchy, misfire handling.
6. **Design a Payment Processing System** — HLD: idempotency, reconciliation, ledger consistency. LLD: transaction state machine, double-entry bookkeeping model.

For each, write your answer down (even bullet form) — verbalizing separately from writing catches gaps.

---

## Resources (curated, not exhaustive)

**LLD**
- *Head First Design Patterns* (refresher, fast read given your experience)
- *Effective Java* by Joshua Bloch — re-read chapters on generics, concurrency, enums as a senior dev, not a beginner
- Refactoring.Guru (pattern reference, free)

**HLD**
- *Designing Data-Intensive Applications* by Martin Kleppmann — the single best book for this level; don't skip it
- *System Design Interview Vol 1 & 2* by Alex Xu — good for pattern recognition, less good for depth
- ByteByteGo / High Scalability blog for real-world case studies (Uber, Netflix, Discord engineering blogs are gold — more valuable than generic courses)

---

## How to Know You're Ready (Self-Assessment)

You're at the target level when you can, for any of the above problems:
1. Ask clarifying requirement questions before designing (functional + non-functional)
2. Produce a class diagram or component diagram in under 10 minutes
3. Justify every major tradeoff with "it depends on X, and here I chose Y because Z"
4. Identify at least one failure mode and how your design handles it
5. Explain how you'd evolve the design if scale grew 100x

---

*Tip: Since you have 10+ years of hands-on Java experience, don't over-invest in Weeks 1–2 fundamentals — skim for gaps, then spend the saved time on Weeks 6–10 where the real differentiation happens.*
