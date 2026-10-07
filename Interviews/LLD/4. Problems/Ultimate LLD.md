
---

## Progress Tracker

> Tables below don't support interactive checkboxes in Obsidian, so tracking lives here. Check items off as you complete them.

### Tier 1 — Foundations (1–20)

- [ ] 1. Parking Lot
- [ ] 2. Vending Machine
- [ ] 3. ATM
- [x] 4. Tic-Tac-Toe
- [ ] 5. Snake & Ladder
- [ ] 6. Deck of Cards + a card game
- [ ] 7. Elevator System
- [ ] 8. Logging Framework
- [ ] 9. Rate Limiter (single node)
- [ ] 10. LRU + LFU Cache
- [ ] 11. Splitwise
- [ ] 12. Library Management
- [ ] 13. Coffee Machine / Beverage Builder
- [ ] 14. Notification Service
- [ ] 15. Chess
- [ ] 16. Traffic Signal Controller
- [ ] 17. Online Auction
- [ ] 18. Inventory & Order Management
- [ ] 19. In-Memory File System
- [ ] 20. In-Memory Pub/Sub Bus

### Tier 2 — Product-scale (21–40)

- [ ] 21. Jira / Kanban Board
- [ ] 22. Google Calendar
- [ ] 23. BookMyShow / Ticketmaster
- [ ] 24. Uber — Ride Matching & Dispatch
- [ ] 25. Food Delivery (Swiggy/Zomato)
- [ ] 26. Amazon — Cart, Checkout, Orders
- [ ] 27. Payment Gateway
- [ ] 28. Digital Wallet / UPI
- [ ] 29. WhatsApp Messaging
- [ ] 30. Slack Workspace
- [ ] 31. Twitter/X Feed
- [ ] 32. Google Docs — Collaborative Editing
- [ ] 33. Stack Overflow
- [ ] 34. Yelp / Nearby Search
- [ ] 35. Netflix — Catalog & Playback
- [ ] 36. Airline Reservation
- [ ] 37. Airbnb / Hotel Booking
- [ ] 38. Stock Exchange — Order Matching Engine
- [ ] 39. URL Shortener with Analytics
- [ ] 40. Git — Core Object Model

### Tier 3 — Platform & Infra (41–50)

- [ ] 41. Distributed Job Scheduler (Airflow-lite)
- [ ] 42. CI/CD Pipeline Engine
- [ ] 43. Feature Flag & Experimentation Service
- [ ] 44. ML Feature Store (Feast-like)
- [ ] 45. Model Serving Router / Inference Gateway (KServe-like)
- [ ] 46. Experiment Tracking + Model Registry (MLflow-like)
- [ ] 47. API Gateway with Distributed Rate Limiting
- [ ] 48. Message Queue with Consumer Groups (Kafka-lite)
- [ ] 49. Object Store Metadata Service (S3-like)
- [ ] 50. Container Scheduler (Kubernetes-lite)

### Tier 1

| # | Problem | Concepts under test | What you must get right | Status |
|---|---------|--------------------|--------------------------|--------|
| 1 | Parking Lot | Inheritance, Strategy, Factory | Spot-allocation strategy and pricing strategy must be swappable without touching `ParkingLot` |
| 2 | Vending Machine | State pattern | Illegal transitions (dispense before payment) impossible by construction, not by `if` checks |
| 3 | ATM | State + Chain of Responsibility | Cash dispense as a chain of denomination handlers; rollback on partial failure |
| 4 | Tic-Tac-Toe | Clean abstraction, generalization | Board should generalize to N×N without rewrites; win-check as a strategy |
| 5 | Snake & Ladder | Composition over inheritance | `Jump` as one abstraction, not separate Snake/Ladder classes |
| 6 | Deck of Cards + a card game | Enums, generics, layering | Separate deck/shuffle mechanics from game rules cleanly |
| 7 | Elevator System | State + scheduling strategy, threads | SCAN vs FCFS dispatch pluggable; request queue thread-safe |
| 8 | Logging Framework | Chain of Responsibility, Singleton | Levels, multiple appenders (console/file), async writes without losing ordering |
| 9 | Rate Limiter (single node) | Strategy, thread safety | Token bucket + sliding window behind one interface; correctness under concurrent hits |
| 10 | LRU + LFU Cache | Data structure design, generics | O(1) get/put; eviction policy injectable so LRU→LFU is a one-line swap |
| 11 | Splitwise | Strategy, graph settlement | Equal/exact/percent splits; balance simplification to minimum transactions |
| 12 | Library Management | Entity modelling, business rules | Borrow limits, due dates, fines, reservations — rules outside the entity classes |
| 13 | Coffee Machine / Beverage Builder | Builder + Decorator | Add-ons compose price and description dynamically, no combinatorial subclasses |
| 14 | Notification Service | Observer + Strategy + Template | Channel (email/SMS/push) and template both swappable; retry policy per channel |
| 15 | Chess | Polymorphism, Command/Memento | Per-piece move validation, plus undo/redo of moves |
| 16 | Traffic Signal Controller | State machine, timers | Intersection-level coordination; emergency preemption without breaking the FSM |
| 17 | Online Auction | Observer, optimistic concurrency | Concurrent bids on one item; no lost updates, watchers notified |
| 18 | Inventory & Order Management | Repository, transactions | Stock reservation vs deduction; oversell prevention |
| 19 | In-Memory File System | Composite pattern | `find` / `ls -R` / size-rollup traverse files and dirs uniformly |
| 20 | In-Memory Pub/Sub Bus | Observer + producer-consumer | Multiple subscribers, per-subscriber offsets, backpressure when a consumer is slow |

## Tier 2 — Product-scale (21–40)

Now the interesting part. Each of these has a **design tension** — a place where two requirements pull against each other. That tension is what's actually being interviewed.

**21. Jira / Kanban Board**
- Scope: projects, boards, issues, sprints, workflows, comments, activity log.
- Entities: `Project`, `Board`, `Column`, `Issue` (Story/Bug/Epic/Subtask), `Sprint`, `WorkflowState`, `Transition`, `User`.
- Tension: workflows are **user-configurable state machines**, so you can't hardcode `TODO → IN_PROGRESS → DONE`. Model transitions as data with guard conditions, and make issue-type hierarchy (Epic → Story → Subtask) work without a class explosion. Follow-up: audit trail + "who changed what when".

**22. Google Calendar**
- Entities: `Event`, `RecurrenceRule`, `Attendee`, `Calendar`, `Reminder`.
- Tension: **recurring events with exceptions**. Do you materialize occurrences or compute them lazily? Editing "this event only" vs "all future events" is where most designs collapse. Add timezone + DST correctness.

**23. BookMyShow / Ticketmaster**
- Entities: `Show`, `Screen`, `Seat`, `SeatLock`, `Booking`, `PaymentIntent`.
- Tension: **seat locking with TTL**. Two users click the same seat simultaneously; one must lose cleanly. Lock expiry, idempotent booking confirmation, and what happens if payment succeeds after the lock expired.

**24. Uber — Ride Matching & Dispatch**
- Entities: `Rider`, `Driver`, `Trip`, `Location`, `MatchingService`, `PricingStrategy`.
- Tension: **driver-to-rider matching** with a spatial index (geohash/quadtree) plus a driver-state machine (offline → available → assigned → on-trip). One driver must never be matched to two riders. Surge pricing as a strategy.

**25. Food Delivery (Swiggy/Zomato)**
- Entities: `Restaurant`, `MenuItem`, `Cart`, `Order`, `DeliveryPartner`, `OrderStateMachine`.
- Tension: three-party order lifecycle (customer, restaurant, delivery partner) where each can fail or cancel at different stages. Menu availability vs cart contents drifting apart.

**26. Amazon — Cart, Checkout, Orders**
- Entities: `Product`, `Cart`, `Order`, `Inventory`, `Coupon`, `Address`, `Payment`.
- Tension: **price and stock at add-to-cart time vs checkout time**. Coupon/discount stacking rules that don't turn into nested `if`s (rules engine or chain).

**27. Payment Gateway**
- Entities: `PaymentIntent`, `Transaction`, `PSPAdapter`, `Refund`, `Ledger`, `WebhookHandler`.
- Tension: **idempotency and exactly-once semantics**. Client retries the same charge; the network drops the PSP response. Design idempotency keys, a state machine that can't double-charge, and a double-entry ledger that always balances.

**28. Digital Wallet / UPI**
- Entities: `Account`, `Ledger`, `TransferService`, `Transaction`.
- Tension: money transfer between two accounts under concurrency — no negative balances, no lost money. Double-entry bookkeeping, deadlock avoidance on multi-account locks (ordered locking).

**29. WhatsApp Messaging**
- Entities: `User`, `Chat` (1:1/group), `Message`, `DeliveryReceipt`, `Device`.
- Tension: **per-chat message ordering** and delivery states (sent → delivered → read) across multiple devices, plus offline queueing and dedup on retry.

**30. Slack Workspace**
- Entities: `Workspace`, `Channel`, `Thread`, `Message`, `Membership`, `Mention`, `Reaction`.
- Tension: threading model (do replies live in the channel timeline?), unread-count computation per user per channel at scale, and permission checks (private channels, guests) that don't get re-implemented in every call site.

**31. Twitter/X Feed**
- Entities: `User`, `Tweet`, `Follow`, `Timeline`, `FeedGenerator`.
- Tension: **fanout-on-write vs fanout-on-read**, and the hybrid for celebrity accounts. Design the interface so the strategy is swappable per user tier.

**32. Google Docs — Collaborative Editing**
- Entities: `Document`, `Operation`, `Version`, `Session`, `Cursor`.
- Tension: two users editing the same offset simultaneously. Implement operational transform for insert/delete on a single line, or model a CRDT sequence. Also: presence and cursor rebasing.

**33. Stack Overflow**
- Entities: `Question`, `Answer`, `Comment`, `Vote`, `Tag`, `Badge`, `ReputationEngine`.
- Tension: `Post` as a shared abstraction for questions/answers/comments without over-abstracting; reputation as event-driven rules rather than scattered increments.

**34. Yelp / Nearby Search**
- Entities: `Place`, `Review`, `Rating`, `GeoIndex`, `SearchQuery`.
- Tension: proximity search with filters (open now, rating > 4, cuisine) — the index design vs filter design, and keeping aggregate ratings consistent as reviews stream in.

**35. Netflix — Catalog & Playback**
- Entities: `Title`, `Season`/`Episode`, `Profile`, `WatchHistory`, `Playlist`, `Recommendation`.
- Tension: resume-playback position per profile per device, and content availability that varies by region and time window.

**36. Airline Reservation**
- Entities: `Flight`, `SeatMap`, `FareClass`, `Booking`, `Passenger`, `Itinerary`.
- Tension: multi-leg itineraries booked atomically, fare-class inventory buckets, and cancellation/refund rules that differ per fare. Overbooking policy as a strategy.

**37. Airbnb / Hotel Booking**
- Entities: `Property`, `AvailabilityCalendar`, `Booking`, `PricingRule`, `Review`.
- Tension: **overlapping date-range bookings**. Interval representation and conflict detection, plus dynamic pricing per night (weekend/seasonal/min-stay rules).

**38. Stock Exchange — Order Matching Engine**
- Entities: `Order` (limit/market/stop), `OrderBook`, `Trade`, `MatchingEngine`, `PriceLevel`.
- Tension: price-time priority matching, partial fills, and cancel/modify racing with a match. The data structure choice (heaps vs sorted map of price levels with FIFO queues) *is* the answer.

**39. URL Shortener with Analytics**
- Entities: `ShortLink`, `KeyGenerator`, `ClickEvent`, `AnalyticsAggregator`.
- Tension: unique key generation without collisions under concurrency, custom aliases, expiry, and click analytics aggregation that doesn't slow down the redirect path.

**40. Git — Core Object Model**
- Entities: `Blob`, `Tree`, `Commit`, `Ref`/`Branch`, `Index`, `MergeStrategy`.
- Tension: content-addressed storage and the commit DAG. Implement `log`, `diff`, and three-way merge with conflict detection. Great test of whether you can model a graph cleanly.

## Tier 3 Platform & Infra (41–50)

These map onto the systems you already work with day to day, so you can push much further on realism than an interviewer expects. Strong differentiator for platform/infra roles.

**41. Distributed Job Scheduler (Airflow-lite)**
- Entities: `Job`, `DAG`, `Task`, `TaskInstance`, `Trigger`, `Executor`, `Scheduler`.
- Tension: DAG dependency resolution + topological execution, retries with backoff, catchup/backfill semantics, and ensuring **one task instance runs exactly once** across multiple scheduler replicas (leader election / locking).

**42. CI/CD Pipeline Engine**
- Entities: `Pipeline`, `Stage`, `Step`, `Runner`, `Artifact`, `Trigger`, `CacheKey`.
- Tension: parallel stages with fan-in/fan-out, artifact passing between steps, cancellation propagation to running steps, and secrets scoping.

**43. Feature Flag & Experimentation Service**
- Entities: `Flag`, `Rule`, `Segment`, `Variant`, `Experiment`, `Evaluator`.
- Tension: **deterministic bucketing** (same user always gets the same variant) with percentage rollouts, targeting rules, and flag dependencies. Client-side SDK cache + invalidation.

**44. ML Feature Store (Feast-like)**
- Entities: `Entity`, `FeatureView`, `FeatureService`, `OnlineStore`, `OfflineStore`, `Materialization`.
- Tension: **point-in-time correctness** — training reads must not leak future feature values (the classic as-of join), while serving reads must be single-digit-ms from the online store. Model the same feature definition serving both paths without duplicating logic.

**45. Model Serving Router / Inference Gateway (KServe-like)**
- Entities: `Model`, `ModelVersion`, `Endpoint`, `Router`, `Predictor`, `Transformer`, `TrafficSplit`.
- Tension: canary/shadow traffic splitting, dynamic request batching (latency vs throughput knob), cold-start on scale-from-zero, and per-model resource isolation.

**46. Experiment Tracking + Model Registry (MLflow-like)**
- Entities: `Experiment`, `Run`, `Metric`, `Param`, `Artifact`, `RegisteredModel`, `Stage`.
- Tension: time-series metrics logged at high frequency vs cheap querying; model lineage (which run + which data produced this artifact) and stage-transition governance.

**47. API Gateway with Distributed Rate Limiting**
- Entities: `Route`, `Filter`/`Middleware` chain, `AuthProvider`, `RateLimiter`, `CircuitBreaker`.
- Tension: rate limit shared across N gateway nodes (local counters + sync vs central store), plus a middleware chain where order matters and any filter can short-circuit.

**48. Message Queue with Consumer Groups (Kafka-lite)**
- Entities: `Topic`, `Partition`, `Producer`, `Consumer`, `ConsumerGroup`, `OffsetStore`, `Rebalancer`.
- Tension: partition assignment and **rebalancing when a consumer joins or dies**, at-least-once vs at-most-once via offset commit timing, and per-partition ordering guarantees.

**49. Object Store Metadata Service (S3-like)**
- Entities: `Bucket`, `Object`, `ObjectVersion`, `MultipartUpload`, `Part`, `Policy`, `LifecycleRule`.
- Tension: multipart upload assembly (parts arriving out of order, abandoned uploads), versioning with delete markers, and lifecycle transitions — all as metadata operations separate from byte storage.

**50. Container Scheduler (Kubernetes-lite)**
- Entities: `Pod`, `Node`, `Scheduler`, `ResourceRequest`, `Predicate`, `PriorityFunction`, `Controller`.
- Tension: two-phase scheduling — filter (predicates: does it fit, does it tolerate taints) then score (priorities: spread, least-loaded) — plus a reconciliation loop that drives actual state toward desired state, and preemption of lower-priority pods.
