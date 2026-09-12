# IDS-011 · Design an app for waste management

## Contents

1. [Interview framing and scope](#interview-framing-and-scope)
2. [Requirements and assumptions](#requirements-and-assumptions)
3. [Architecture](#architecture)
4. [Data and APIs](#data-and-apis)
5. [Correctness and distributed operation](#correctness-and-distributed-operation)
6. [Reliability, security, and operations](#reliability-security-and-operations)
7. [Scale and evolution](#scale-and-evolution)
8. [Trade-offs and follow-ups](#trade-offs-and-follow-ups)

> **Interview note:** These are interview designs, not claims about any proprietary implementation. State the assumptions aloud and challenge them when the interviewer changes the product, traffic, or regulatory constraints.

## Interview framing and scope

Start by naming the actors, the primary user journey, and the boundary of the first release. Separate the source of truth from derived views, and agree which guarantees users actually need. Keep the first diagram small; add partitions, queues, caches, and regional controls only when a requirement motivates them.

## Requirements and assumptions

| Dimension | Baseline assumption | Design consequence |
| --- | --- | --- |
| Availability | 99.95% for the core user journey | Multi-AZ stateless services and tested failover |
| Latency | p95 under 250 ms for interactive reads; p99 under 1 s | Cache hot paths and bound fan-out |
| Durability | No acknowledged write is lost | Quorum/transactional persistence and replayable events |
| Growth | 3x traffic and 2x data in 24 months | Partition keys, online migration, and backpressure |

Functional requirements, exclusions, and scale numbers below are deliberately explicit. They are starting points, not facts; ask the interviewer to change one and recalculate capacity.

## Architecture

The control plane owns identity, policy, configuration, and lifecycle; the data plane serves the high-volume user path. APIs are stateless behind an edge gateway. Durable writes go to a partitioned primary store, then to an outbox/event stream. Consumers build search, cache, notification, analytics, and audit projections. Every asynchronous boundary has a stable event ID, a retry policy, and a dead-letter path.

```mermaid
flowchart LR
  C[Client] --> E[Edge / API gateway]
  E --> S[Domain services]
  S --> DB[(Partitioned source of truth)]
  S --> O[Transactional outbox]
  O --> Q[Event stream]
  Q --> P[Read models, search, notifications]
  E --> K[(Distributed cache)]
  S --> M[Metrics, logs, traces]
```

### Component responsibilities

- **Edge:** TLS termination, authentication, request IDs, quota enforcement, schema validation, and coarse routing; no business transactions.
- **Domain services:** Validate commands, enforce invariants, own their tables, and expose versioned contracts.
- **Source of truth:** Stores authoritative state with constraints and a recovery point objective compatible with the SLO.
- **Event pipeline:** Publishes committed facts, supports replay, preserves ordering only within a declared key, and isolates slow consumers.
- **Projection workers:** Materialize query-specific views; consumers are idempotent and record the last applied version.
- **Operations plane:** Provides feature flags, key rotation, audit trails, dashboards, and safe administrative repair tools.

## Data and APIs

### Critical contracts

Use opaque IDs, cursor pagination, explicit version fields, and idempotency keys on commands. Return a stable error envelope (`code`, `message`, `retryable`, `request_id`). A mutating request is accepted only after its command is durably recorded; clients can safely retry the same idempotency key.

```http
POST /v1/resources
Idempotency-Key: 01J...
If-Match: "version-7"

{ " ...": "domain fields" }

HTTP/1.1 202 Accepted
{ "resource_id": "r_123", "operation_id": "op_456", "state": "accepted" }
```

### Data model and indexes

Keep immutable identity and mutable state separate where auditability matters. Typical tables are `entities(id, tenant_id, state, version, created_at, updated_at)`, `commands(idempotency_key, actor_id, request_hash, result, created_at)`, and an append-only `events(event_id, aggregate_id, aggregate_version, type, payload, committed_at)`. Index by the access path (`tenant_id, updated_at, id`), enforce uniqueness at the database, and use TTL only for derived or explicitly expirable data.

## Correctness and distributed operation

### Key invariants

1. A committed command has one authoritative outcome, even if the client retries.
2. State transitions are validated against the current version; stale updates fail or merge explicitly.
3. A published event corresponds to a committed source-of-truth change (transactional outbox).
4. Derived views may lag, but expose a version/timestamp so clients can reason about staleness.
5. Reconciliation can detect and repair divergence without silently inventing business state.

### Read/write flows

**Write:** authenticate -> validate policy and idempotency key -> transactionally update the aggregate and outbox -> acknowledge -> asynchronously update projections. **Read:** authorize -> check a versioned cache -> query the appropriate projection/source -> attach freshness metadata -> emit latency and correctness telemetry. Use bounded retries with jitter only for transient failures; never retry a non-idempotent command without its key.

### Partitioning and replication

Partition by the dominant locality key (tenant, user, geography, or aggregate) and add a hot-key strategy for celebrity objects or global counters. Replicate synchronously across availability zones for acknowledged writes and asynchronously to a secondary region. Promote only after fencing the old writer; rebuild projections from the event log after failover. Cross-partition transactions should be avoided or implemented as a saga with compensating actions.

## Reliability, security, and operations

### Failure modes and recovery

- **Dependency timeout:** deadlines, circuit breakers, cached/stale reads, and a clear degraded response.
- **Duplicate delivery:** event IDs and consumer inbox tables make handlers idempotent.
- **Poison event:** exponential backoff, bounded attempts, dead-letter queue, and operator replay after correction.
- **Partition or region loss:** route to the healthy region, preserve a write fence, and reconcile when the partition heals.
- **Corrupt projection:** stop the consumer, snapshot the source, replay from a known offset, and compare checksums.

### Security, privacy, and abuse controls

Use least-privilege service identities, envelope encryption, key rotation, mTLS internally, and immutable audit records for sensitive operations. Classify fields, minimize collection, redact secrets and personal data from logs, and define retention/deletion workflows. Apply per-identity and per-network rate limits, payload limits, abuse scoring, anomaly detection, and admin approval for high-impact operations.

### Observability and SLOs

Track request rate, p50/p95/p99 latency, error budget burn, saturation, queue age, consumer lag, cache hit ratio, replication lag, stale-read age, and reconciliation drift. Propagate a trace ID through synchronous and asynchronous work. Page on user-visible symptoms and budget burn, not on every transient dependency error; attach runbooks and a tested rollback to each alert.

## Scale and evolution

### Capacity sketch

For 10 million daily active users and 20 requests per user per day, average traffic is about 2,300 requests/second. A 10x peak gives 23,000 requests/second. If a write is 2 KB and 5% of requests write, primary ingress is roughly 2.3 MB/s before replication; three replicas and indexes make the real storage budget several times larger. Size queues for peak bursts, not averages, and retain enough history for replay and compliance.

Scale stateless tiers horizontally, split read models by query shape, and isolate hot tenants. Use online repartitioning (dual-write, backfill, verify, cut over), schema compatibility windows, and feature flags. Keep an explicit cost model: cache memory, stream partitions, replicas, egress, and operational toil are part of the design.

## Trade-offs and follow-ups

- Strong synchronous cross-region writes improve RPO but increase tail latency and reduce availability during inter-region faults; single-writer plus asynchronous replication is often a better default.
- A relational source of truth gives constraints and transactions; a key-value store may win on predictable scale but shifts invariants into application code.
- Materialized views make reads fast and flexible but introduce lag and replay complexity.
- Build versus buy should include migration, operability, compliance, and lock-in—not just infrastructure price.

### Interviewer follow-up questions and strong-answer cues

1. **What changes at 100x traffic?** Identify the first bottleneck, shard key, hot-key mitigation, and which guarantees can be relaxed.
2. **What if the event stream is unavailable?** Keep the command path safe with an outbox, bound backlog, and define when to reject writes.
3. **How do you prove no duplicate charge/action?** Name the idempotency key, unique constraint, transaction boundary, and reconciliation job.
4. **How do you migrate without downtime?** Explain compatible schema, backfill checkpoints, dual reads/writes, validation, and rollback.
5. **What would you measure first in production?** Pick one user SLO, one saturation signal, one correctness invariant, and one cost metric.


## System-specific deep dive

### Scope and civic scope

Design a municipal waste-management platform for residents, collectors, fleet operators, and administrators: pickup requests/schedules, route planning, bin/vehicle telemetry, missed-pickup reporting, recycling classification, payments/fees, and public dashboards. Assume 5 million households, 200,000 collection assets, 100,000 GPS pings/minute, 500,000 daily pickup tasks, and intermittent rural connectivity.

### Architecture and flows

Resident and driver apps use an API gateway and offline-first local queues. Scheduling owns service calendars and exceptions; dispatch owns route plans; fleet telemetry ingests device events; a GIS optimizer produces routes subject to capacity, time windows, hazardous-material rules, and driver hours. A task assignment is durable before a driver sees it. Public metrics are aggregated from completed tasks and weighbridge records, not raw device pings.

### Data and APIs

`POST /v1/pickup-requests`, `GET /v1/households/{id}/schedule`, `POST /v1/tasks/{id}/checkpoint`, `POST /v1/missed-pickups`, and `GET /v1/routes/{id}` support idempotent offline replay. Model `households`, `service_calendars`, `pickup_tasks`, `routes`, `vehicles`, `telemetry`, `material_weights`, and `incidents`. Geospatial indexes support nearby assets; time-bucketed partitions retain telemetry cheaply. A task transition includes device sequence, GPS accuracy, and server receipt time.

### Correctness, privacy, and safety

A pickup is counted once despite retries; route assignments are versioned and driver acknowledgements are fenced. Resolve conflicts by server sequence plus operator review, not client clock. Minimize household location data, protect worker identity, and restrict live routes to authorized dispatchers. Apply geofence/device attestation where safety needs it, detect fabricated checkpoints, and retain only coarse public statistics. Hazardous waste requires specialized workflows, permits, and incident escalation.

### Operations and evolution

During network outages, drivers continue from signed route snapshots and sync later; the server rejects stale assignments safely. Monitor missed-pickup rate, route completion, telemetry lag, vehicle capacity violations, app sync backlog, and regional service equity. Discuss citizen incentives, contractor multi-tenancy, dynamic routing, seasonal demand, and whether to integrate IoT bin sensors.
