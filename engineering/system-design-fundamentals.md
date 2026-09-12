# Full System Design Question Bank

Thirty-eight additional system-design prompts covering reusable foundations and canonical product patterns. Each card is deliberately scoped so it does not repeat the existing architecture, distributed-systems, or data-and-storage collections.

## Requirements and architecture

<details>
<summary><strong>SD-F001 · How do you turn an ambiguous product request into a system-design problem?</strong></summary>

**Answer guidance**: Clarify actors, core workflows, scale, latency, durability, privacy, availability, and success metrics before drawing boxes. State assumptions and separate must-have requirements from nice-to-haves. Use the answers to define an MVP boundary and an evolution path; a design that cannot name its constraints is not yet a design.
</details>

<details>
<summary><strong>SD-F002 · How would you choose service boundaries for a new product?</strong></summary>

**Answer guidance**: Group capabilities around business invariants, ownership, data locality, and change cadence rather than around nouns or organizational silos. Start with a modular monolith when boundaries are uncertain; extract services when independent scaling, deployment, or isolation has a measurable benefit. Define APIs and failure ownership before splitting.
</details>

<details>
<summary><strong>SD-F003 · When is a modular monolith a better choice than microservices?</strong></summary>

**Answer guidance**: Prefer a modular monolith when the domain and traffic shape are evolving, the team is small, or transactions cross many modules. It keeps local calls, one deployment, and simpler debugging while preserving explicit module interfaces. Move to services only when independent scale, fault isolation, compliance, or team autonomy outweighs network and operational complexity.
</details>

<details>
<summary><strong>SD-F004 · How would you design an API that can evolve without breaking clients?</strong></summary>

**Answer guidance**: Prefer additive, backward-compatible changes: stable resource identifiers, optional fields, tolerant readers, pagination, idempotency where writes can be retried, and explicit error contracts. Version only when semantics must change, measure client usage, and publish a deprecation window. Contract tests and schema validation prevent accidental breaking changes.
</details>

<details>
<summary><strong>SD-F005 · How do you select REST, GraphQL, or gRPC for an interface?</strong></summary>

**Answer guidance**: Choose REST for broadly interoperable resource APIs, GraphQL for client-specific read composition with strong governance, and gRPC for typed internal low-latency calls and streaming. Evaluate caching, observability, authorization, compatibility, tooling, and failure behavior—not just payload size. Avoid exposing a flexible query surface without depth, cost, and N+1 controls.
</details>

<details>
<summary><strong>SD-F006 · How would you design an API gateway for many backend services?</strong></summary>

**Answer guidance**: Keep the gateway focused on edge concerns—authentication, routing, rate limits, request shaping, and telemetry—while keeping domain decisions in services. Use timeouts, bounded retries, circuit breaking, and response-size limits. Avoid turning it into a distributed monolith; deploy it redundantly and provide a clear bypass or degraded path for failures.
</details>

<details>
<summary><strong>SD-F007 · How should a design handle configuration and feature flags safely?</strong></summary>

**Answer guidance**: Store versioned configuration in a strongly controlled service, validate it before publication, and distribute immutable snapshots with local caching. Separate flags from secrets, scope rollouts by tenant or cohort, and record who changed what. Every flag needs an owner, expiry date, kill switch, and a tested default for control-plane unavailability.
</details>

## Core platform patterns

<details>
<summary><strong>SD-F008 · Design a URL-shortening service that supports redirects, custom aliases, and analytics.</strong></summary>

**Answer guidance**: Generate collision-resistant IDs and map aliases to immutable destinations in a durable store; serve redirects from a cache with a defined invalidation policy. Keep click analytics asynchronous so redirect latency is independent of aggregation. Protect aliases with uniqueness checks, rate-limit creation, and decide whether edits, expiry, abuse scanning, and privacy-preserving analytics are in scope.
</details>

<details>
<summary><strong>SD-F009 · Design a globally scalable rate limiter for APIs.</strong></summary>

**Answer guidance**: Pick a policy—token bucket for bursts, leaky bucket for smoothing, or a sliding window for precise quotas—and define the identity and limits. Use an atomic counter store near the enforcement point; partition keys and tolerate bounded approximation under replication lag. Decide fail-open versus fail-closed by endpoint risk, return retry metadata, and monitor rejected traffic.
</details>

<details>
<summary><strong>SD-F010 · Design a distributed job scheduler for delayed and recurring work.</strong></summary>

**Answer guidance**: Persist jobs with due times, lease them atomically to workers, and make handlers idempotent because crashes can produce duplicate execution. Use sharded time buckets or a priority queue for due-job scans, visibility timeouts for recovery, and backoff/dead-letter handling for poison jobs. Define clock-skew, misfire, cancellation, and fairness semantics explicitly.
</details>

<details>
<summary><strong>SD-F011 · How would you build a reliable task queue for asynchronous work?</strong></summary>

**Answer guidance**: Choose at-least-once delivery as the practical default, with durable enqueue, consumer acknowledgements, visibility timeouts, bounded retries, and a dead-letter queue. Make consumers idempotent using a task key or transactional outbox. Partition for throughput, preserve ordering only where required, and expose queue age and poison-message rates as operational signals.
</details>

<details>
<summary><strong>SD-F012 · Design a distributed lock or lease service and explain its limits.</strong></summary>

**Answer guidance**: Use a consensus-backed store for short leases with fencing tokens, monotonic expiry, and renewal; a holder must present the token to the protected resource so a paused process cannot write after its lease expires. Never treat a lock as proof of liveness. Keep critical sections short, define recovery, and prefer idempotent workflows or a single-writer design when possible.
</details>

<details>
<summary><strong>SD-F013 · How would you design a service-discovery mechanism for dynamic instances?</strong></summary>

**Answer guidance**: Instances register leases and health state; clients or a discovery layer resolve healthy endpoints with cached snapshots and zone-aware balancing. Use TTLs, graceful draining, and a last-known-good fallback during control-plane outages. Avoid making every request depend on discovery, and secure registration so an untrusted instance cannot impersonate a service.
</details>

<details>
<summary><strong>SD-F014 · Design a configuration/secret distribution service for production workloads.</strong></summary>

**Answer guidance**: Encrypt secrets with envelope keys, restrict access by workload identity, audit reads and rotations, and deliver short-lived credentials where possible. Distribute signed, versioned snapshots so applications can start with a known-good cache while the control plane is unavailable. Separate secret rotation from application deploys and test partial rollout and revocation paths.
</details>

<details>
<summary><strong>SD-F015 · How would you design a safe distributed cache?</strong></summary>

**Answer guidance**: Define whether the cache is an optimization or a source of truth, then choose cache-aside, read-through, or write-through semantics accordingly. Prevent stampedes with request coalescing and jittered expiry, address hot keys with replication, and use bounded values and eviction policies. Plan invalidation, stale reads, serialization compatibility, and behavior when the cache is unavailable.
</details>

<details>
<summary><strong>SD-F016 · Design a content-delivery system for large immutable and mutable assets.</strong></summary>

**Answer guidance**: Put versioned immutable assets behind an edge cache and object storage; use content hashes for safe, long-lived caching. Mutable assets need cache-control, purge, or short TTL semantics and an origin fallback. Add range requests, upload verification, malware scanning, signed URLs, quotas, and regional replication based on asset size, privacy, and recovery requirements.
</details>

## Data flows and event-driven design

<details>
<summary><strong>SD-F017 · How would you use an event-driven architecture without losing business correctness?</strong></summary>

**Answer guidance**: Make the transactional state change and event publication atomic with an outbox or CDC, then use idempotent consumers and explicit schemas. Events should describe durable facts, not transient commands, and include IDs, versions, and causation metadata. Define ordering per aggregate, replay policy, poison-event handling, and what users see while projections lag.
</details>

<details>
<summary><strong>SD-F018 · Design a schema registry and event-compatibility process.</strong></summary>

**Answer guidance**: Register versioned schemas with owners and compatibility rules; prefer adding optional fields and tolerant consumers over renaming or changing meaning. Validate producers in CI and at the broker boundary, retain examples, and support a deprecation window with usage telemetry. Treat sensitive-field classification and retention as part of the contract.
</details>

<details>
<summary><strong>SD-F019 · How would you design a reliable data-ingestion pipeline from external partners?</strong></summary>

**Answer guidance**: Authenticate and validate at the edge, land the raw immutable payload, then normalize asynchronously into a curated model. Use partner-specific checkpoints, deduplication keys, replayable transformations, quarantine for malformed records, and backpressure. Track freshness, completeness, and reconciliation totals; never silently drop a record because a downstream schema changed.
</details>

<details>
<summary><strong>SD-F020 · Design a change-data-capture pipeline that downstream consumers can trust.</strong></summary>

**Answer guidance**: Capture commits with transaction ordering and a durable offset, publish enough before/after metadata for consumers, and define snapshot-plus-stream bootstrap. Consumers checkpoint independently and tolerate duplicates; compaction or tombstones must preserve deletion semantics. Monitor source lag, schema drift, failed records, and the gap between source and projection.
</details>

<details>
<summary><strong>SD-F021 · How would you design a workflow engine for long-running business processes?</strong></summary>

**Answer guidance**: Persist workflow state and immutable transitions, model timers and external callbacks as durable events, and make every activity retryable and idempotent. Use leases for workers, compensating actions for non-transactional side effects, and versioned definitions so in-flight workflows remain interpretable. Provide operator pause, resume, replay, and audit controls.
</details>

<details>
<summary><strong>SD-F022 · Design a fan-out system that delivers one event to millions of recipients.</strong></summary>

**Answer guidance**: Separate event acceptance from delivery, partition recipients, and choose push, pull, or hybrid fan-out based on recipient activity and freshness needs. Precompute for hot events only when storage cost is justified; otherwise let clients consume a cursor-based feed. Apply quotas, backpressure, deduplication, and per-recipient retry so one slow consumer cannot block the broadcast.
</details>

<details>
<summary><strong>SD-F023 · How would you design a transactional outbox for reliable side effects?</strong></summary>

**Answer guidance**: Commit the domain mutation and an outbox row in one local transaction, then publish rows asynchronously with a checkpoint and idempotent relay. Consumers still need deduplication because publication is at-least-once. Partition and monitor the outbox, define retention and ordering, and reconcile rows stuck between the database and broker; use a saga or compensation for effects that cannot be atomic.
</details>

<details>
<summary><strong>SD-F024 · Design a replay and backfill mechanism for derived data.</strong></summary>

**Answer guidance**: Retain immutable source events or snapshots, version transformations, and replay into an isolated projection before an atomic cutover. Bound replay load so it cannot starve live traffic, and preserve event-time semantics for late arrivals. Compare counts and checksums, support pause/resume, and retain the old projection for rollback until correctness is proven.
</details>

## Resilience, security, and operations

<details>
<summary><strong>SD-F025 · How would you allocate dependency budgets across a request path?</strong></summary>

**Answer guidance**: Start with the end-to-end latency SLO and allocate an explicit time, concurrency, and error budget to each downstream call, including serialization and queueing overhead. Enforce deadlines with a propagated request context, reserve capacity for critical paths, and reject optional work early. Revisit budgets from traces and load tests; a timeout without a budget merely moves the queue elsewhere.
</details>

<details>
<summary><strong>SD-F026 · Design a disaster-recovery strategy for a stateful service.</strong></summary>

**Answer guidance**: Set RTO and RPO from business impact, then select backups, point-in-time recovery, replicas, or a warm/hot secondary accordingly. Protect backups from the same failure domain, encrypt and regularly restore-test them, and document DNS, credentials, dependencies, and data-integrity checks. A failover runbook and game days matter more than a nominal replication diagram.
</details>

<details>
<summary><strong>SD-F027 · How would you secure service-to-service communication in a zero-trust environment?</strong></summary>

**Answer guidance**: Authenticate workload identity with short-lived certificates or tokens, authorize each request using least-privilege policy, and encrypt in transit with managed rotation. Propagate identity and trace context safely, isolate networks as defense in depth, and log decisions without leaking payloads. Design for credential expiry and control-plane outages rather than assuming a permanently healthy mesh.
</details>

<details>
<summary><strong>SD-F028 · Design tenant isolation for a multi-tenant service.</strong></summary>

**Answer guidance**: Choose logical, schema, or physical isolation per data sensitivity and noisy-neighbor risk, then enforce tenant context at every API, query, cache key, queue, and log boundary. Add quotas and per-tenant admission control, test cross-tenant access explicitly, and isolate encryption keys where required. Plan tenant migration and deletion without weakening the isolation invariant.
</details>

<details>
<summary><strong>SD-F029 · How would you build an abuse-prevention architecture for a public API?</strong></summary>

**Answer guidance**: Combine identity and reputation signals with layered quotas, burst limits, payload validation, anomaly detection, and progressive challenges. Keep enforcement cheap at the edge, provide appeal and allow-list paths, and avoid blocking legitimate shared networks solely by IP. Record decisions and tune thresholds against false-positive and abuse-loss metrics.
</details>

<details>
<summary><strong>SD-F030 · What should a production readiness review cover for a new service?</strong></summary>

**Answer guidance**: Verify SLOs, capacity headroom, dependency timeouts, failure behavior, backups, migrations, security controls, on-call ownership, dashboards, alerts, runbooks, and rollback. Require load and fault tests that represent the stated risks. The review should produce explicit launch gates and follow-up owners, not just an architecture approval.
</details>

<details>
<summary><strong>SD-F031 · How would you instrument a system so operators can explain a bad request?</strong></summary>

**Answer guidance**: Use correlation IDs and distributed traces across boundaries, structured logs with stable fields, and RED/USE metrics tied to user-facing SLOs. Sample intelligently while preserving errors and representative slow traces; redact secrets and personal data. Make deploy versions, feature flags, dependency calls, and queue time visible so a trace supports diagnosis rather than merely showing topology.
</details>

<details>
<summary><strong>SD-F032 · Design a safe zero-downtime deployment strategy for a stateful system.</strong></summary>

**Answer guidance**: Separate backward-compatible schema changes from code rollout: expand, deploy readers/writers, backfill, then contract after verification. Use canaries or blue-green traffic shifting with health and business metrics, and keep rollback possible at each step. Coordinate long-running jobs, caches, message schemas, and connection draining; a database migration is part of the release, not an afterthought.
</details>

## Product-scale designs

<details>
<summary><strong>SD-F033 · Design an autocomplete/typeahead service with useful results under 100 ms.</strong></summary>

**Answer guidance**: Precompute compact prefix or n-gram structures from an authoritative catalog and serve them from a memory-friendly, replicated read store near users. Rank by prefix quality, popularity, personalization, and freshness while applying authorization filters before returning suggestions. Bound query length and fan-out, cache hot prefixes, publish versioned snapshots, and define behavior when the latest catalog is unavailable.
</details>

<details>
<summary><strong>SD-F034 · How would you design a recommendation system with both cold start and feedback loops?</strong></summary>

**Answer guidance**: Combine content and popularity priors for new users/items with collaborative or learned signals as interactions accumulate. Separate offline training from online feature serving, enforce freshness and privacy constraints, and explore deliberately so the model does not reinforce a narrow feedback loop. Measure business value, diversity, latency, and harmful outcomes, not only click-through rate.
</details>

<details>
<summary><strong>SD-F035 · Design a collaborative presence and live-status service.</strong></summary>

**Answer guidance**: Use short-lived heartbeats over WebSockets or a similar channel, aggregate presence in an expiring store, and publish coarse updates rather than every heartbeat. Define what “online” means under partitions and mobile sleep, and make absence eventually consistent. Isolate presence from durable user data; it is an ephemeral signal and should fail without blocking core workflows.
</details>

<details>
<summary><strong>SD-F036 · Design a privacy-preserving user data deletion workflow.</strong></summary>

**Answer guidance**: Inventory every authoritative and derived copy, record a deletion request with authorization and retention exceptions, then fan it out through durable, auditable work. Make retries idempotent, verify completion with reconciliation, and handle backups, caches, search indexes, analytics, and vendor processors explicitly. Return status without exposing deleted data and retain only the minimum compliance evidence.
</details>

<details>
<summary><strong>SD-F037 · How would you design a quota and billing-metering platform?</strong></summary>

**Answer guidance**: Emit immutable usage facts with idempotency keys, aggregate them into versioned meter periods, and separate fast enforcement counters from authoritative invoice calculations. Define late events, corrections, currency/time-zone rules, and dispute reconciliation. Protect the path with tenant quotas and backpressure, and make every invoice amount traceable to source facts.
</details>

<details>
<summary><strong>SD-F038 · Design a multi-region active-active service and explain when not to use it.</strong></summary>

**Answer guidance**: Start with a conflict model: partition data by home region, use commutative operations or deterministic conflict resolution, and route reads and writes with clear failover semantics. Replicate asynchronously with bounded lag and reconcile after partitions. Active-active is justified only when its latency and availability benefits exceed conflict, cost, operational, and compliance complexity; otherwise use a simpler single-writer or warm-secondary design.
</details>
