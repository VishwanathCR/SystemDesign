# HLD & LLD Roadmap for a Senior Java Developer (10+ YOE)
### Taught through one running example: an Enterprise E-Commerce Order Management Platform (Amazon/Flipkart-style)

This roadmap assumes strong Java fundamentals already. Instead of disconnected practice problems, **every week builds one piece of the same system** — a full e-commerce platform: catalog, cart, inventory, order, payment, shipping, notifications. By the end, you'll have designed the whole platform end-to-end, at both the class level (LLD) and the architecture level (HLD).

Why this example: it's the single most common enterprise Java system (Spring Boot + microservices shops all resemble this), it naturally requires every major pattern and distributed-systems concept, and it's realistic enough that the tradeoffs are grounded, not academic.

**Duration:** 10–12 weeks, ~6–8 hrs/week. Track A (LLD) and Track B (HLD) run in parallel each week — same subsystem, two altitudes.

---

## The System We're Building

**Enterprise E-Commerce Platform** — core modules:
1. Product Catalog & Search
2. Cart & Pricing
3. Inventory Management
4. Order Management (the spine — state machine, orchestration)
5. Payment Processing
6. Shipping & Fulfillment
7. Notifications
8. Reviews & Ratings

We'll touch nearly all of these, but **Order Management** is the backbone we return to every week, because it's where concurrency, consistency, and distributed transactions all collide.

---

## Track A: LLD — Class-Level Design of Each Module

### Week 1 — Domain Modeling: Catalog & Cart
Apply SOLID and composition-over-inheritance to real entities, not toy examples.
- Model `Product`, `ProductVariant` (size/color), `Category`, `Cart`, `CartItem`
- Where does pricing logic live? (Strategy pattern for discounts/coupons — percentage off, flat off, BOGO)
- Immutable `Money` value object (never use `double` for currency — discuss why)
- Builder pattern for `Product` (many optional attributes: dimensions, warranty, specs)

**Deliverable:** Class diagram for Catalog + Cart, with a pluggable discount engine (Strategy) that can add new discount types without modifying `Cart`.

**Self-check:** How would you add "buy 2 get 1 free" without touching existing discount classes?

---

### Week 2 — Order Management Core: State Machines & Patterns
This is the heart of the system.
- Model `Order` lifecycle: `CREATED → PAYMENT_PENDING → CONFIRMED → SHIPPED → DELIVERED → CANCELLED/RETURNED`
- **State pattern** for order status transitions (illegal transitions should be impossible by design, not by `if` checks scattered everywhere)
- **Observer pattern**: when order state changes, notify Inventory, Notification, Shipping modules
- **Command pattern**: represent operations like `CancelOrderCommand`, `RefundCommand` — enables undo/audit logging
- Exception hierarchy: `OrderException` → `InvalidTransitionException`, `InsufficientInventoryException`, etc.

**Deliverable:** `Order` class with a State pattern implementation; show the exact transition table and what's illegal.

**Self-check:** Can `SHIPPED → CREATED` ever happen in your design? If your code doesn't prevent it, your design has a hole.

---

### Week 3 — Concurrency: Inventory & Payment Idempotency
Where senior engineers separate from mid-level.
- Design a thread-safe `InventoryService.reserveStock()` — this is the classic "two people buy the last item" race condition
- Optimistic locking (version column) vs pessimistic locking (`SELECT FOR UPDATE`) — implement both approaches for stock decrement, compare
- `java.util.concurrent`: `CompletableFuture` for parallel calls (e.g., checkout calling Inventory + Pricing + Fraud-check concurrently)
- Idempotency: design `PaymentService.charge(idempotencyKey)` so retried requests never double-charge
- Virtual threads (Java 21+): would they change how you write the checkout orchestration code?

**Deliverable:** Thread-safe stock reservation with a chosen locking strategy, justified in writing.

**Self-check:** Under what load does optimistic locking start failing too often (high contention on hot SKUs, e.g., flash sale)? What do you switch to?

---

### Week 4 — Payment & Shipping Module Design
- Adapter pattern: integrate multiple payment gateways (Razorpay, Stripe, PayPal) behind one `PaymentGateway` interface
- Facade pattern: `CheckoutFacade` hides the orchestration of cart validation + pricing + payment + inventory from the controller layer
- Factory pattern: `ShippingProviderFactory` picks a courier partner based on region/weight
- Retry + circuit breaker at the code level (Resilience4j annotations) — what does the *class design* look like underneath, not just the annotation

**Deliverable:** `PaymentGateway` interface + two adapter implementations; a `CheckoutFacade` that composes everything cleanly.

**Self-check:** If Razorpay's API changes its response schema, how many classes do you touch?

---

### Week 5 — API & Contract Design + Full LLD Walkthrough
- REST API design for Order Management: idempotent `POST /orders`, versioning (`/v2/orders`), pagination for order history
- DTO vs domain model separation — why you never expose JPA entities directly
- Full walkthrough: given a "Place Order" request, trace it through every class you've built Weeks 1–4, end to end
- Concurrency edge case: two devices, same user, double-tap "Place Order" — where exactly does your design prevent a duplicate order?

**Deliverable:** End-to-end sequence diagram: Cart → Checkout → Payment → Inventory → Order Confirmation, referencing every pattern used so far.

**Self-check:** Present this as if in a design review — could a new hire extend it to add "scheduled delivery" without a rewrite?

---

## Track B: HLD — System-Level Design of the Same Platform

### Week 1 — Scale the Catalog & Search
- Estimate scale: 50M products, 10M DAU, 5000 QPS on search, 500 QPS on checkout — practice the math
- Read-heavy catalog: caching strategy (Redis cache-aside), CDN for product images
- Search: why catalog search typically goes to Elasticsearch, not the primary DB — denormalization tradeoffs
- Load balancing across catalog service instances, stateless service design

**Deliverable:** Capacity plan + component diagram for Catalog & Search at the estimated scale.

---

### Week 2 — Order Management at Scale: Database & Consistency
- SQL vs NoSQL decision for `Orders` table (hint: strong consistency + transactions needed → usually relational, sharded)
- Sharding strategy for Orders (shard by `user_id` vs `order_id` — tradeoffs for "get my order history" vs "get order by ID")
- Replication: read replicas for order history queries, primary for writes
- CAP/PACELC applied concretely: during a network partition, do you prioritize accepting the order (availability) or guaranteeing no overselling (consistency)? Justify per use case.

**Deliverable:** Data architecture diagram for Orders — sharding key, replication topology, and your CAP tradeoff decision with reasoning.

---

### Week 3 — Caching & Consistency Across Services
- Cache-aside for product/price data; write-through for inventory counts (staleness is dangerous here)
- Cache invalidation when price or stock changes — event-driven invalidation vs TTL
- Distributed cache coherency: multiple app instances, one Redis cluster — what can still go wrong?
- Session/cart storage: Redis-backed cart vs DB-backed cart — tradeoffs for an enterprise system with guest checkout

**Deliverable:** Design the Cart storage layer for 10M concurrent shopping sessions — where does it live, how does it survive a Redis node failure?

---

### Week 4 — Messaging: Decoupling Order → Inventory → Shipping → Notification
- Kafka topic design: `order-events`, `payment-events`, `shipment-events`
- At-least-once delivery + your Week 3 (Track A) idempotency keys — connect the dots between LLD and HLD here
- Event-driven architecture: Order service publishes `OrderConfirmed`; Inventory, Shipping, Notification services consume independently
- Dead-letter queues: what happens when the Notification service is down for 10 minutes — do orders stall?

**Deliverable:** Event flow diagram for the full order lifecycle across services, with failure handling at each hop.

---

### Week 5 — Distributed Transactions: The Checkout Problem
This is the hardest and most enterprise-realistic problem in the whole system.
- Checkout touches Cart, Inventory, Payment, Order, Shipping — all separate services/databases. No single ACID transaction possible.
- **Saga pattern**: orchestration (a central Order Orchestrator service) vs choreography (services react to each other's events) — implement both conceptually, pick one and justify
- Compensating transactions: if Payment succeeds but Inventory reservation fails, how do you roll back the charge?
- Distributed locking (Redis Redlock) for high-demand flash-sale items where DB-level optimistic locking isn't enough at scale

**Deliverable:** Full Saga design for Checkout — happy path + at least two failure/compensation paths (payment fails after inventory reserved; inventory fails after payment succeeds).

---

### Week 6 — Resilience, Observability & Multi-Region
- Circuit breakers between Order service and Payment/Inventory (what happens to checkout UX when Payment is degraded?)
- Distributed tracing across the 6+ services involved in one checkout (trace ID propagation)
- Rate limiting at the API gateway (protect checkout from bot/scalper traffic during flash sales)
- Multi-region considerations: where does the source-of-truth Order DB live if you have US and EU customers? Data residency implications (GDPR)

**Deliverable:** Resilience + observability overlay on your Week 5 Saga diagram — annotate every hop with timeout, retry, and fallback behavior.

---

## Weeks 7–9: Full System Mock Rounds — Extend the Platform

Now extend the same e-commerce platform with new requirements, combining HLD + LLD in single 45–60 min sessions:

1. **Flash Sale Feature** — 1M users hitting "Buy Now" on a limited-stock item in 60 seconds. HLD: queueing at the gateway, virtual waiting room. LLD: atomic stock decrement, fairness in reservation.
2. **Real-Time Order Tracking** — customer sees live shipment location. HLD: websockets vs polling vs push notifications at scale. LLD: `ShipmentEvent` class design, state updates.
3. **Return & Refund Workflow** — HLD: reverse logistics event flow, refund reconciliation with payment provider. LLD: `ReturnOrder` state machine, partial refund calculation.
4. **Recommendation Engine Integration** — HLD: how a near-real-time recommendation service plugs into Catalog without becoming a bottleneck. LLD: Strategy pattern for swappable recommendation algorithms.
5. **Multi-Warehouse Inventory** — HLD: routing an order to the nearest warehouse with stock, split shipments. LLD: `WarehouseAllocationStrategy` design.
6. **Fraud Detection at Checkout** — HLD: synchronous vs async fraud scoring, latency budget. LLD: rule-engine design (Chain of Responsibility pattern) for fraud checks.

Write each answer down — verbalizing and writing separately catches gaps in both.

---

## Resources (curated, not exhaustive)

**LLD**
- *Effective Java* by Joshua Bloch — re-read the concurrency and generics chapters at your level, not as a beginner
- Refactoring.Guru — fast pattern reference when you forget a name mid-design

**HLD**
- *Designing Data-Intensive Applications* by Martin Kleppmann — non-negotiable for this level
- Engineering blogs: Amazon, Flipkart, Shopify, Uber — read their actual checkout/order-system writeups; more valuable than generic courses
- *System Design Interview* Vol 1 & 2 by Alex Xu — good for structure, less for depth

---

## How to Know You're Ready

For any extension to this platform, you should be able to:
1. Ask clarifying functional + non-functional requirements before designing
2. Produce a class diagram (LLD) or component diagram (HLD) in under 10 minutes
3. Justify every tradeoff with "it depends on X, and here I chose Y because Z"
4. Identify at least one failure mode (race condition, partition, service outage) and show how your design survives it
5. Explain how the design changes at 10x and 100x scale

---

*Tip: since the whole platform is one continuous example, don't treat each week as isolated — by Week 6 you should be able to redraw the entire system (Catalog → Cart → Checkout → Order → Payment → Shipping → Notification) from memory, at both class and architecture level.*
