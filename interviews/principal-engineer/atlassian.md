# Atlassian Principal Engineer Interview Questions

This file contains the Atlassian loop, grouped by interview round. Each question includes model answer guidance.

## ATLASSIAN

### 1. Coding & System Fundamentals (streaming, RBAC, hierarchical data, API/crawler design)

**1. How would you design a distributed rate limiter that must stay consistent across multiple regions with sub-100ms latency requirements?**
Model answer: I'd start by clarifying whether "consistent" means exact global counts or an acceptable approximation, since that decision drives the whole architecture. For sub-100ms, a strict global counter (e.g., a single Redis cluster with cross-region calls) is a non-starter — cross-region round trips alone can exceed budget. I'd use a local-first token bucket per region with periodic async reconciliation (gossip or a lightweight aggregator), accepting a bounded overshoot in exchange for latency. For tenants who need hard guarantees (e.g., billing-sensitive quotas), I'd carve out a stricter path with a regional leader and tighter sync, explicitly trading availability for correctness only where it's worth the cost. I'd call out this consistency/latency trade-off explicitly to the interviewer and ask what the business tolerance for overshoot actually is.

**2. Walk through the trade-offs between a sliding-window vs. token-bucket algorithm for a multi-tenant API gateway.**
Model answer: Token bucket is simple, memory-cheap, and naturally supports bursts up to the bucket size, but can allow two bursts to land back-to-back at a window boundary, effectively doubling the momentary rate. Sliding window (log or counter-based) gives smoother, more accurate enforcement at the cost of more memory (log) or approximation error (counter-based sliding window). For a multi-tenant gateway, I'd default to a sliding-window counter approximation — good accuracy, O(1) memory — and reserve exact sliding-window logs for premium tenants with contractual SLAs, since the memory cost scales with active tenant count.

**3. How would you model a hierarchical, tenant-aware RBAC system that supports millions of resources and inherited permissions?**
Model answer: I'd model permissions as a directed graph of principals, roles, and resources, where resources sit in a hierarchy (e.g., org → project → issue) and permission checks walk up the tree unless explicitly overridden. To make "can user X access resource Y" fast at scale, I'd precompute and cache an effective-permissions materialized view per (user, resource-subtree) rather than recomputing the graph walk on every request, invalidating that cache on role or hierarchy changes. Tenant isolation would be enforced at the storage layer (partitioned by tenant) so a bug in one tenant's permission graph can't leak into another's. The hard part is invalidation correctness — I'd bias toward slightly stale reads with short TTLs over slightly wrong reads with no TTL.

**4. What's your approach to designing a real-time event stream processor that must guarantee exactly-once delivery at scale?**
Model answer: True exactly-once end-to-end is expensive and rarely worth it; I'd first push to define exactly-once *effect* via idempotent consumers (dedupe keys, upserts) layered on at-least-once delivery, which is far cheaper and more resilient than trying to guarantee exactly-once transport. Where the processing includes side effects (e.g., writing to multiple systems), I'd use a transactional outbox or a stream processing framework's exactly-once semantics (e.g., Kafka Streams transactions) scoped to the parts of the pipeline that truly need it, not the whole system. I'd also plan for replay: consumers must tolerate reprocessing the same event without corrupting state.

**5. How would you design a tagging/labeling system that supports fast top-N and faceted search queries across billions of records?**
Model answer: I'd separate the write path (tag assignment, stored in a normalized table for correctness) from the read path (a search index like Elasticsearch/OpenSearch with tags as filterable, faceted fields), updated via async CDC rather than dual writes. For top-N queries I'd precompute popular aggregations (e.g., top tags per project) incrementally rather than scanning on read, since billions of records make ad hoc aggregation too slow. Faceted search benefits from inverted indexes with cardinality-aware sharding — high-cardinality tenants get their own shards to avoid noisy-neighbor query costs.

**6. Describe how you'd evolve a monolithic permission model into a scalable, hierarchical resource-access system without downtime.**
Model answer: I'd run this as a strangler-fig migration: introduce the new hierarchical model alongside the old flat model, dual-write both, and validate them against each other on read (shadow reads, logging mismatches) before ever trusting the new model for enforcement. Once mismatch rates are near zero for a sustained period, flip reads to the new model behind a feature flag, keep the old model as a fallback, and only decommission it after a full deprecation window. The key discipline is never having a single migration step that both changes behavior and removes the rollback path at the same time.

**7. How would you architect a web crawler service that respects per-tenant rate limits and deduplicates content efficiently?**
Model answer: I'd separate URL discovery/scheduling from fetching: a scheduler maintains per-tenant (or per-domain) rate-limited queues, and a pool of fetcher workers pulls from whichever queue currently has budget, so one tenant's aggressive crawl can't starve others. For dedup, I'd hash normalized content (not raw bytes, to avoid false negatives from whitespace/ad noise) and check against a probabilistic structure like a Bloom filter for a fast first pass, falling back to an exact store for confirmed matches — this keeps the hot path cheap while bounding false positives to an acceptable rate.

**8. What data structures would you choose for a moving-average/streaming-aggregate service processing millions of events per second, and why?**
Model answer: For a fixed-window moving average, a circular buffer with a running sum gives O(1) updates without recomputing the full window each time. For approximate aggregates at very high cardinality (e.g., per-user moving averages across millions of users), I'd reach for sketch structures — count-min sketch or t-digest — that trade exactness for bounded memory. The choice hinges on whether downstream consumers (billing, alerting, dashboards) need exact numbers or can tolerate a known error bound; I'd push to clarify that early since it changes the whole design.

**9. How do you approach backward compatibility when redesigning a core data model (e.g., RBAC or resource hierarchy) used by hundreds of internal teams?**
Model answer: I treat the old model as a contract, not an implementation detail, even if internally I know it's flawed. I'd version the API/schema explicitly, provide an adapter layer that translates old-model calls onto the new model under the hood, and give consuming teams a long, well-communicated deprecation timeline with usage dashboards so they can self-serve migration status. For anything that can't be perfectly translated (semantic differences, not just structural ones), I'd flag those cases explicitly rather than silently approximating, because silent semantic drift is what causes production incidents six months later.

**10. As a Principal Engineer, how would you evaluate whether to build a new event-processing framework in-house vs. adopt an existing one (e.g., Kafka Streams, Flink)?**
Model answer: I'd start from the assumption that build is the wrong answer unless proven otherwise — frameworks like Flink represent years of edge-case hardening that's expensive to replicate. I'd evaluate against concrete criteria: does an existing tool meet our latency/throughput/exactly-once requirements, what's the operational cost of running it at our scale, and is there a genuine gap (not just an annoyance) that justifies the multi-year cost of owning custom infrastructure. I'd also weigh org-level cost: even if in-house is technically superior on paper, it creates a bus-factor and hiring liability that a well-adopted open-source tool doesn't.

### 2. System Design (real-time collaboration, issue tracking, notification pipelines, plugin architecture)

**1. Design a real-time collaborative document editing system (like Confluence) — how do you handle conflict resolution and offline sync?**
Model answer: For conflict resolution I'd use either Operational Transformation or CRDTs; I lean CRDTs for simplicity of reasoning about convergence and easier offline support, accepting the trade-off of larger metadata overhead per edit. Offline clients would buffer local operations and replay them against the CRDT structure on reconnect, merging deterministically without a central arbiter deciding "who wins." For the collaboration transport, I'd use WebSockets with a fallback to long-polling, and a presence service so users see who else is editing. The interesting scaling problem is document sharding — very large or very hot documents (e.g., a company-wide wiki page) need special-cased handling so one document doesn't bottleneck a shared editing cluster.

**2. Design an issue-tracking system (like Jira) that scales to millions of tickets across thousands of organizations — what are the sharding and indexing strategies?**
Model answer: I'd shard primarily by tenant (organization), since most queries are tenant-scoped and this keeps noisy neighbors isolated; within very large tenants, I'd further shard by project. For search/filtering, I'd maintain a separate indexed read store (search engine) fed by CDC from the source of truth, since Jira's flexible custom-field querying doesn't map well to a single relational index strategy. Cross-tenant admin queries (rare but real) would go through a separate analytics pipeline rather than the live transactional path, so they can't degrade normal ticket operations.

**3. How would you design a notification pipeline that must support millions of users, multiple channels (email, Slack, in-app), and per-user preference rules?**
Model answer: I'd separate "event happened" from "notification delivered": events are published to a stream, a rules/preferences engine evaluates per-user routing and batching logic, and channel-specific delivery workers handle the actual send with channel-appropriate retry/backoff. Preference evaluation should be cacheable and versioned so a preference change takes effect quickly without re-querying a database on every event. I'd also build in digest/batching logic at the routing layer (not per-channel) so a user who wants a daily email digest doesn't get 50 individual emails from 50 individual events.

**4. Design a plugin/marketplace architecture that allows third-party extensions without compromising platform security or performance.**
Model answer: I'd run third-party code in an isolated execution environment (sandboxed process, WASM runtime, or serverless function) with a narrow, capability-based API rather than direct access to internal services — plugins call a well-defined gateway, not our database. Resource limits (CPU, memory, request timeouts) need to be enforced per-plugin so a misbehaving extension can't degrade the host app; I'd also require static review or automated scanning of manifests declaring exactly what data/permissions a plugin requests, shown transparently to installing admins.

**5. How would you design multi-region data residency for a SaaS platform serving enterprise customers with strict compliance requirements?**
Model answer: I'd pin a tenant's primary data to a specific region at provisioning time based on their residency requirement, and make that binding explicit and immutable without an active migration process — trying to make it silently "flexible" invites compliance bugs. Shared/global services (auth, billing) need careful design to either replicate per-region or prove they don't touch regulated data. The hardest part is usually not the data plane but the control plane — logs, backups, and support tooling that engineers use day-to-day also need to respect residency, which is often where compliance violations actually happen.

**6. What architectural decisions would you make to support both strong consistency for billing/permissions and eventual consistency for activity feeds?**
Model answer: I'd treat these as genuinely different systems with different guarantees rather than forcing one data store to serve both. Billing and permissions go through a strongly consistent, likely single-writer-per-partition store (e.g., a relational database with proper transactions), because correctness failures there have direct financial/security consequences. Activity feeds are read-heavy and tolerant of staleness, so I'd let them run off an async event stream into a denormalized read store optimized for feed queries, explicitly accepting lag in exchange for scalability.

**7. How would you design a search indexing pipeline that stays near-real-time as issues/documents are created and updated at high volume?**
Model answer: I'd use CDC (change data capture) off the primary datastore into a stream, with index writers consuming that stream and applying updates incrementally rather than batch reindexing. To handle bursty write volume, I'd decouple ingestion rate from indexing rate with a buffer, and monitor indexing lag as a first-class SLO, alerting when it exceeds a threshold users would notice. For rare full-corpus schema changes, I'd support blue-green reindexing into a parallel index and cut over atomically rather than mutating the live index in place.

**8. Design a system for audit logging across a multi-product suite (Jira, Confluence, Bitbucket) with a unified query interface.**
Model answer: I'd define a common audit event schema (actor, action, resource, timestamp, product, metadata) that every product emits to, rather than trying to normalize disparate per-product logs after the fact. Events would land in an append-only, tamper-evident store (important for audit integrity) and be indexed for the unified query layer. Given audit logs are often compliance-critical and rarely deleted, I'd plan storage tiering — hot recent data queryable fast, older data in cheaper cold storage still queryable but with higher latency.

**9. As Principal Engineer, how would you drive a cross-team architectural decision (e.g., migrating from REST to GraphQL) across multiple product lines?**
Model answer: I'd start by writing a concise architecture doc that states the problem being solved (not "GraphQL is better" but the specific pain — e.g., over-fetching, N+1 client calls) and the migration cost, then socialize it with the affected teams' senior engineers before it's a fait accompli, since buy-in from people who'll do the work matters more than a mandate from above. I'd propose a pilot on one bounded product surface, measure the actual impact, and use that evidence to build the case org-wide rather than asserting it up front. Where teams disagree, I'd rather narrow the scope of the migration than force adoption against strong technical objections.

**10. How do you evaluate and justify a build-vs-buy decision for critical infrastructure (e.g., a workflow engine or search platform) to leadership?**
Model answer: I frame it in terms of total cost of ownership, not just initial build cost: engineering time to build, ongoing operational burden, opportunity cost of not shipping product features, and the risk of accumulating a bespoke system that becomes a hiring and onboarding liability. I'd present a small number of concrete options with honest trade-offs rather than a single recommendation dressed as inevitable, and tie the decision to business risk in language leadership can act on — e.g., "buy gets us to market in one quarter with vendor lock-in risk X; build gets us full control in three quarters with Y engineer-years of ongoing cost."

### 3. Front-End & Modern JavaScript Practices

**1. How do you evaluate whether a feature should be built as a micro-frontend vs. part of a monolithic front-end app?**
Model answer: I look at team topology first, not technology — micro-frontends solve an organizational scaling problem (independent deploy cadence for separate teams) more than a technical one, and they add real complexity (shared design system drift, bundle duplication, cross-app navigation). If a single team owns the feature and it doesn't need independent deployment, I'd keep it in the monolith. I'd only reach for micro-frontends when the coordination cost of shared deploys is demonstrably higher than the runtime/complexity cost of splitting.

**2. What's your strategy for enforcing performance budgets across dozens of teams contributing to a shared front-end platform?**
Model answer: Budgets need to be automated and enforced in CI, not aspirational documentation — bundle size and key metrics (LCP, TTI) checked on every PR with hard failure thresholds, not just dashboards nobody looks at. I'd also make the cost visible at the point of decision (e.g., a bundle-size diff comment on the PR) rather than surfaced only after merge. For legitimate exceptions, I'd have a lightweight override process with an owner and expiry date, so exceptions don't silently become permanent.

**3. How would you design a component library/design system that scales across multiple product teams while allowing local customization?**
Model answer: I'd separate core primitives (strict, versioned, rarely broken) from composable patterns (looser, teams can extend) so teams aren't forced to choose between full conformance and full escape-hatch. Theming/customization should go through defined design tokens rather than arbitrary CSS overrides, which preserves visual consistency while allowing brand or product-specific variation. Governance matters as much as code: a clear contribution process and a small core team reviewing breaking changes prevents the library from fragmenting into per-team forks.

**4. Explain trade-offs between server-side rendering, static generation, and client-side rendering for a B2B SaaS dashboard.**
Model answer: A B2B dashboard is typically behind auth, personalized, and not SEO-sensitive, which removes the strongest argument for SSR/SSG (crawlability, fast first paint for anonymous users). I'd lean client-side rendering with aggressive code-splitting and a fast API, since the data is highly dynamic and per-user anyway — pre-rendering personalized dashboard content offers little benefit. I'd reconsider SSR only if initial load time on data-heavy views becomes a measured pain point for users on poor connections.

**5. How do you approach state management in a large, long-lived single-page application with many contributing teams?**
Model answer: I'd draw a clear line between server state (data fetched from APIs — best handled by a dedicated caching/fetching library) and client/UI state (local to a component or a small feature), since conflating them into one global store is a common source of complexity in large apps. For genuinely shared client state, I'd scope it narrowly to the features that need it rather than a single global store everyone touches, since that shared mutable surface becomes a coordination bottleneck across teams.

**6. What's your approach to migrating legacy front-end code (e.g., jQuery to a modern framework) incrementally in production?**
Model answer: I'd migrate page-by-page or feature-by-feature behind the existing routing, never attempt a big-bang rewrite of the whole app at once — that's a classic way to stall for a year and ship nothing. A strangler pattern where the new framework mounts into isolated DOM regions alongside legacy code lets both coexist during the transition, with automated visual/functional regression tests protecting the surfaces not yet migrated.

**7. How would you architect a front-end plugin system so third-party UI extensions can't degrade core app performance?**
Model answer: I'd load third-party UI in isolated contexts (iframes, or web components with strict resource budgets) rather than sharing the host app's JS execution context directly, so a slow or buggy extension can't block the main thread. I'd enforce a resource/timeout budget per extension and fail gracefully (hide or placeholder the extension) rather than let it hang the whole page.

**8. Describe your approach to setting front-end engineering standards (linting, testing, accessibility) across a large organization.**
Model answer: Standards need to be enforced by tooling, not convention — shared lint configs, CI gates, and accessibility checks (automated axe-core scans plus manual review for complex flows) baked into the shared build pipeline every team inherits by default. I'd version these standards like a product, with a changelog and migration guides when rules tighten, so teams aren't blindsided by a sudden wave of CI failures.

**9. How do you balance technical debt reduction against feature delivery pressure as a technical leader?**
Model answer: I treat debt paydown as an ongoing cost of doing business, not a special project that competes for a separate budget — I push for a standing allocation (e.g., a percentage of each sprint) rather than periodic "debt sprints" that get deprioritized under pressure. I also prioritize debt by actual cost (velocity drag, incident frequency) rather than aesthetic discomfort, and I'm explicit with stakeholders about which debt is genuinely blocking future work versus which is safe to defer.

**10. How would you mentor a front-end team through adopting a new rendering architecture with minimal disruption?**
Model answer: I'd start with a small, low-risk pilot surface, pair closely with a couple of engineers to build internal expertise before broad rollout, and document the concrete patterns (not just theory) that emerged from the pilot. I'd resist the urge to mandate the new architecture everywhere immediately — mentorship works better as "here's how we did it and what we learned" than "everyone must do this now," which builds genuine buy-in rather than compliance.

### 4. Networking / OS Fundamentals (rapid-fire style, elevated)

**1. Explain how you'd diagnose intermittent latency spikes in a service that spans multiple availability zones.**
Model answer: I'd start with distributed tracing to see whether the spikes correlate with a specific downstream dependency, AZ, or time window (e.g., GC pauses, cross-AZ network hops, noisy-neighbor contention). Intermittent issues are often either resource contention (check CPU steal, GC, connection pool saturation) or a specific slow dependency that's masked by averages — I'd look at p99/p999 latency broken down by AZ and dependency before guessing.

**2. Walk through what happens at the OS and network layer when a service experiences connection pool exhaustion under load.**
Model answer: New requests either queue waiting for a connection or get rejected/timed out, and if the pool is shared with health checks, those can start failing too, potentially triggering load balancer ejection and cascading load onto remaining healthy instances. At the OS level, if this is TCP connections, you may also see accumulating TIME_WAIT sockets or exhausted ephemeral ports if connections churn rather than reuse. The fix is usually a combination of right-sizing the pool, adding backpressure/queueing limits, and ensuring slow downstream calls have their own timeouts so they don't hold connections indefinitely.

**3. How would you design a system's retry and backoff strategy to avoid cascading failures during a partial outage?**
Model answer: Exponential backoff with jitter is table stakes to avoid synchronized retry storms; beyond that I'd add a circuit breaker so a service stops hammering a clearly-failing dependency and fails fast instead, and bound total retry budget per request so retries don't multiply load during an outage exactly when the system is most fragile. I'd also make sure retries are only applied to idempotent operations.

**4. Explain the trade-offs between TCP and UDP for a real-time collaboration feature's transport layer.**
Model answer: TCP gives ordered, reliable delivery which simplifies application logic but head-of-line blocking can hurt real-time responsiveness when a single lost packet stalls everything behind it. UDP (often via WebRTC data channels) avoids that stall and suits low-latency, loss-tolerant use cases, but pushes reliability and ordering concerns back to the application. For collaborative editing, I'd typically still use a reliable, ordered transport (WebSocket over TCP) since correctness of the edit stream matters more than shaving milliseconds, unless the specific feature (e.g., cursor position) is genuinely loss-tolerant.

**5. How would you approach root-causing a memory leak in a long-running service without full production access?**
Model answer: I'd rely on whatever telemetry is exposed rather than live debugging — heap usage trends over time, GC frequency/pause metrics, and periodic heap snapshots or profiling exports if the runtime supports them (e.g., pprof, JVM heap dumps) triggered on a schedule or threshold. I'd correlate the leak's growth rate against deploy timestamps and traffic patterns to narrow down which code path or recent change is responsible, then reproduce in a staging environment with production-like load where I do have full access.

**6. What's your strategy for load-balancing across regions when latency and data residency both matter?**
Model answer: I'd route by residency constraint first (hard requirement) and only optimize for latency within the set of regions a given tenant is legally allowed to use — residency isn't a tunable, it's a filter applied before any latency-based routing decision. Within allowed regions, geo-DNS or anycast plus health-aware routing handles the latency optimization.

**7. How do you decide when to introduce a service mesh vs. simpler client-side load balancing?**
Model answer: A service mesh earns its complexity when you need consistent cross-cutting concerns (mTLS, retries, observability) across many polyglot services without every team reimplementing them — but it adds real operational overhead (sidecar resource cost, upgrade complexity). For a small number of services or a single-language stack, a good client-side library often gets 80% of the benefit at a fraction of the operational cost. I'd only push for a mesh once the number of services and languages makes per-service consistency untenable otherwise.

**8. Explain how DNS resolution failures could cascade in a microservices architecture and how you'd mitigate that.**
Model answer: If services resolve each other via DNS and a DNS backend degrades, you can see a thundering herd of failed lookups and retries across the whole fleet simultaneously — a single point of failure hiding behind seemingly independent services. Mitigations include client-side DNS caching with sane TTLs, fallback resolvers, and treating DNS as a dependency worth its own SLO and monitoring rather than assuming it "just works."

**9. How would you set organization-wide standards for observability (tracing, metrics, logging) across polyglot services?**
Model answer: I'd standardize on an open standard (e.g., OpenTelemetry) so instrumentation isn't tied to a specific vendor or language, provide shared libraries/wrappers per language that make "doing it right" the path of least resistance, and bake trace-context propagation into shared HTTP/RPC client libraries so individual teams don't have to remember to wire it up. Standards that require manual discipline from every team tend to decay; standards embedded in shared tooling tend to stick.

**10. As a technical leader, how do you decide the right level of infrastructure abstraction for teams with varying operational maturity?**
Model answer: I'd offer a paved-road default (a well-supported, opinionated platform) for teams that want to move fast without deep infra expertise, while allowing an escape hatch for teams with the maturity and genuine need to go lower-level — but I'd make the escape hatch intentionally a bit more effortful, so it's a deliberate choice, not the path of least resistance for everyone. The goal is matching abstraction level to team need, not forcing uniformity.

### 5. SQL / Data Manipulation

**1. Write a query to identify all products that have had at least two price changes within a rolling 90-day window.**
Model answer:
```sql
SELECT product_id
FROM (
  SELECT product_id, changed_at,
         COUNT(*) OVER (
           PARTITION BY product_id
           ORDER BY changed_at
           RANGE BETWEEN INTERVAL '90 days' PRECEDING AND CURRENT ROW
         ) AS changes_in_window
  FROM price_changes
) t
WHERE changes_in_window >= 2
GROUP BY product_id;
```
I'd flag that "rolling" needs a precise definition (any 90-day window ending anywhere, vs. the last 90 days from today) since that changes the query significantly.

**2. How would you design a schema to efficiently support both transactional writes and analytical rollups at scale?**
Model answer: I'd keep the transactional (OLTP) schema normalized and optimized for write correctness and point lookups, and stream changes (via CDC) into a separate analytical store (columnar warehouse) optimized for scans and aggregation. Trying to serve both patterns from one schema usually compromises both — wide denormalized tables hurt transactional write performance, and normalized tables hurt analytical scan performance.

**3. Explain your approach to indexing strategy for a table with high write throughput and frequent range queries.**
Model answer: Every index adds write cost, so I'd index deliberately for the specific range queries that matter, likely a composite index ordered to match the most common filter + range pattern, rather than indexing every column defensively. For very high write throughput, I'd also consider whether some indexes can be maintained asynchronously (e.g., in a secondary read-optimized store) rather than synchronously on every write.

**4. How would you detect and prevent data duplication in a system fed by multiple upstream event sources?**
Model answer: Prevention is better than detection — I'd require idempotency keys or natural dedup keys on ingestion, and use upsert semantics keyed on that identifier rather than blind inserts. For detection after the fact, I'd run periodic reconciliation jobs comparing counts/hashes against expected source-of-truth totals, since silent duplication often isn't caught until a downstream report looks wrong.

**5. Write a query approach to compute a sliding-window moving average of active users over the last 7 days.**
Model answer:
```sql
SELECT day,
       AVG(daily_active_users) OVER (
         ORDER BY day
         ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
       ) AS moving_avg_7d
FROM daily_active_user_counts
ORDER BY day;
```
At real scale, I'd precompute daily active user counts incrementally rather than recomputing distinct-user counts from raw event data on every query.

**6. How do you decide between denormalization and normalization when designing a schema for a high-scale reporting system?**
Model answer: I lean denormalized for reporting/analytical workloads where read performance and query simplicity matter more than storage cost or update anomalies, since reporting data is typically append-only or batch-updated rather than subject to frequent in-place edits. I keep the source-of-truth transactional data normalized and treat the denormalized reporting schema as a derived, rebuildable artifact.

**7. What's your strategy for schema migrations on a table with billions of rows and zero downtime tolerance?**
Model answer: I'd avoid any migration that locks the table or rewrites it in place; instead, add new columns as nullable, backfill asynchronously in batches with rate limiting to avoid replication lag or lock contention, and only enforce constraints (NOT NULL, etc.) once backfill is verified complete. For structural changes, I'd use a shadow-table/dual-write approach and cut over once validated, similar to the general strangler-migration pattern.

**8. How would you design a data model to support hierarchical permissions queries efficiently (e.g., "can user X access resource Y")?**
Model answer: I'd use a closure table or materialized path pattern to avoid recursive graph traversal on every permission check, so "is Y a descendant of any resource X has access to" becomes a simple indexed lookup rather than a recursive query. I'd pair this with the effective-permissions cache described earlier for the hottest paths.

**9. Explain how you'd approach data partitioning strategy for a multi-tenant system with wildly uneven tenant sizes.**
Model answer: Naive hash-based tenant partitioning breaks down when one tenant is 1000x larger than another — that tenant's shard becomes a hotspot. I'd use a hybrid: most tenants share pooled partitions via hash/range partitioning, but oversized tenants get dedicated partitions (or their own database), decided by a monitored size/traffic threshold rather than statically at onboarding.

**10. As a Principal Engineer, how do you evaluate whether a data problem needs a new datastore vs. better modeling in the existing one?**
Model answer: I push teams to articulate the specific failure mode of the current datastore (a query pattern it can't serve efficiently, a scale ceiling being approached, a consistency model mismatch) before considering a new one — "this would be easier in X" isn't sufficient justification given the operational cost of running another datastore. If better indexing, partitioning, or a read replica solves it, that's almost always cheaper than introducing new infrastructure with its own on-call burden and expertise requirement.

---
