# Coding & Data Structures

Questions about algorithms, data structures, performance fundamentals, and coding problem solving.

## Atlassian

### 1. Coding & System Fundamentals (streaming, RBAC, hierarchical data, API/crawler design)

<details>
<summary><strong>Q001 · How would you design a distributed rate limiter that must stay consistent across multiple regions with sub-100ms latency requirements?</strong></summary>

**Answer guidance**: I'd start by clarifying whether "consistent" means exact global counts or an acceptable approximation, since that decision drives the whole architecture. For sub-100ms, a strict global counter (e.g., a single Redis cluster with cross-region calls) is a non-starter — cross-region round trips alone can exceed budget. I'd use a local-first token bucket per region with periodic async reconciliation (gossip or a lightweight aggregator), accepting a bounded overshoot in exchange for latency. For tenants who need hard guarantees (e.g., billing-sensitive quotas), I'd carve out a stricter path with a regional leader and tighter sync, explicitly trading availability for correctness only where it's worth the cost. I'd call out this consistency/latency trade-off explicitly to the interviewer and ask what the business tolerance for overshoot actually is.


</details>

<details>
<summary><strong>Q002 · Walk through the trade-offs between a sliding-window vs. token-bucket algorithm for a multi-tenant API gateway.</strong></summary>

**Answer guidance**: Token bucket is simple, memory-cheap, and naturally supports bursts up to the bucket size, but can allow two bursts to land back-to-back at a window boundary, effectively doubling the momentary rate. Sliding window (log or counter-based) gives smoother, more accurate enforcement at the cost of more memory (log) or approximation error (counter-based sliding window). For a multi-tenant gateway, I'd default to a sliding-window counter approximation — good accuracy, O(1) memory — and reserve exact sliding-window logs for premium tenants with contractual SLAs, since the memory cost scales with active tenant count.


</details>

<details>
<summary><strong>Q003 · How would you model a hierarchical, tenant-aware RBAC system that supports millions of resources and inherited permissions?</strong></summary>

**Answer guidance**: I'd model permissions as a directed graph of principals, roles, and resources, where resources sit in a hierarchy (e.g., org → project → issue) and permission checks walk up the tree unless explicitly overridden. To make "can user X access resource Y" fast at scale, I'd precompute and cache an effective-permissions materialized view per (user, resource-subtree) rather than recomputing the graph walk on every request, invalidating that cache on role or hierarchy changes. Tenant isolation would be enforced at the storage layer (partitioned by tenant) so a bug in one tenant's permission graph can't leak into another's. The hard part is invalidation correctness — I'd bias toward slightly stale reads with short TTLs over slightly wrong reads with no TTL.


</details>

<details>
<summary><strong>Q004 · What's your approach to designing a real-time event stream processor that must guarantee exactly-once delivery at scale?</strong></summary>

**Answer guidance**: True exactly-once end-to-end is expensive and rarely worth it; I'd first push to define exactly-once *effect* via idempotent consumers (dedupe keys, upserts) layered on at-least-once delivery, which is far cheaper and more resilient than trying to guarantee exactly-once transport. Where the processing includes side effects (e.g., writing to multiple systems), I'd use a transactional outbox or a stream processing framework's exactly-once semantics (e.g., Kafka Streams transactions) scoped to the parts of the pipeline that truly need it, not the whole system. I'd also plan for replay: consumers must tolerate reprocessing the same event without corrupting state.


</details>

<details>
<summary><strong>Q005 · How would you design a tagging/labeling system that supports fast top-N and faceted search queries across billions of records?</strong></summary>

**Answer guidance**: I'd separate the write path (tag assignment, stored in a normalized table for correctness) from the read path (a search index like Elasticsearch/OpenSearch with tags as filterable, faceted fields), updated via async CDC rather than dual writes. For top-N queries I'd precompute popular aggregations (e.g., top tags per project) incrementally rather than scanning on read, since billions of records make ad hoc aggregation too slow. Faceted search benefits from inverted indexes with cardinality-aware sharding — high-cardinality tenants get their own shards to avoid noisy-neighbor query costs.


</details>

<details>
<summary><strong>Q006 · Describe how you'd evolve a monolithic permission model into a scalable, hierarchical resource-access system without downtime.</strong></summary>

**Answer guidance**: I'd run this as a strangler-fig migration: introduce the new hierarchical model alongside the old flat model, dual-write both, and validate them against each other on read (shadow reads, logging mismatches) before ever trusting the new model for enforcement. Once mismatch rates are near zero for a sustained period, flip reads to the new model behind a feature flag, keep the old model as a fallback, and only decommission it after a full deprecation window. The key discipline is never having a single migration step that both changes behavior and removes the rollback path at the same time.


</details>

<details>
<summary><strong>Q007 · How would you architect a web crawler service that respects per-tenant rate limits and deduplicates content efficiently?</strong></summary>

**Answer guidance**: I'd separate URL discovery/scheduling from fetching: a scheduler maintains per-tenant (or per-domain) rate-limited queues, and a pool of fetcher workers pulls from whichever queue currently has budget, so one tenant's aggressive crawl can't starve others. For dedup, I'd hash normalized content (not raw bytes, to avoid false negatives from whitespace/ad noise) and check against a probabilistic structure like a Bloom filter for a fast first pass, falling back to an exact store for confirmed matches — this keeps the hot path cheap while bounding false positives to an acceptable rate.


</details>

<details>
<summary><strong>Q008 · What data structures would you choose for a moving-average/streaming-aggregate service processing millions of events per second, and why?</strong></summary>

**Answer guidance**: For a fixed-window moving average, a circular buffer with a running sum gives O(1) updates without recomputing the full window each time. For approximate aggregates at very high cardinality (e.g., per-user moving averages across millions of users), I'd reach for sketch structures — count-min sketch or t-digest — that trade exactness for bounded memory. The choice hinges on whether downstream consumers (billing, alerting, dashboards) need exact numbers or can tolerate a known error bound; I'd push to clarify that early since it changes the whole design.


</details>

<details>
<summary><strong>Q009 · How do you approach backward compatibility when redesigning a core data model (e.g., RBAC or resource hierarchy) used by hundreds of internal teams?</strong></summary>

**Answer guidance**: I treat the old model as a contract, not an implementation detail, even if internally I know it's flawed. I'd version the API/schema explicitly, provide an adapter layer that translates old-model calls onto the new model under the hood, and give consuming teams a long, well-communicated deprecation timeline with usage dashboards so they can self-serve migration status. For anything that can't be perfectly translated (semantic differences, not just structural ones), I'd flag those cases explicitly rather than silently approximating, because silent semantic drift is what causes production incidents six months later.


</details>

<details>
<summary><strong>Q010 · As a Principal Engineer, how would you evaluate whether to build a new event-processing framework in-house vs. adopt an existing one (e.g., Kafka Streams, Flink)?</strong></summary>

**Answer guidance**: I'd start from the assumption that build is the wrong answer unless proven otherwise — frameworks like Flink represent years of edge-case hardening that's expensive to replicate. I'd evaluate against concrete criteria: does an existing tool meet our latency/throughput/exactly-once requirements, what's the operational cost of running it at our scale, and is there a genuine gap (not just an annoyance) that justifies the multi-year cost of owning custom infrastructure. I'd also weigh org-level cost: even if in-house is technically superior on paper, it creates a bus-factor and hiring liability that a well-adopted open-source tool doesn't.



</details>

## NVIDIA

### 1. Core Coding & Systems Problems

<details>
<summary><strong>Q011 · Design an efficient system to validate whether a given IP address falls within a dynamic, frequently-updated set of CIDR rules at scale.</strong></summary>

**Answer guidance**: I'd store CIDR ranges in a trie (binary trie over the IP bits, or a Patricia/radix trie for compactness), which gives O(bit-length) lookup regardless of rule count and naturally supports longest-prefix matching if rules can overlap. For frequent updates, I'd build the new trie version off the hot path and swap it atomically (copy-on-write) rather than mutating the live structure under concurrent reads, avoiding locking on the read-heavy validation path.


</details>

<details>
<summary><strong>Q012 · Design a "smart calendar" service that resolves scheduling conflicts across time zones and recurring events efficiently.</strong></summary>

**Answer guidance**: I'd normalize all event storage to UTC with the originating time zone as metadata (never store local time as the source of truth, since DST rules change and local time is ambiguous around transitions). Recurring events would be stored as a rule (RRULE-style) plus exceptions, expanded into concrete instances lazily/on-demand within a queried window rather than materializing years of occurrences upfront. Conflict detection is then an interval-overlap problem per user, best served by an interval tree for efficient range queries when checking availability across many events.


</details>

<details>
<summary><strong>Q013 · How would you optimize a hot-path algorithm that's called millions of times per second in a low-latency service?</strong></summary>

**Answer guidance**: I'd profile before optimizing anything — intuition about hot spots is frequently wrong. Once the actual bottleneck is identified, common wins include reducing allocations (object pooling, avoiding per-call heap allocation), improving cache locality (data layout, avoiding pointer-chasing structures), and batching where possible to amortize fixed overhead. I'd also check whether the algorithmic complexity itself is the issue before micro-optimizing constant factors.


</details>

<details>
<summary><strong>Q014 · Walk through how you'd profile and eliminate a performance bottleneck in a multi-threaded C++ service.</strong></summary>

**Answer guidance**: I'd use a sampling profiler (e.g., perf) to find CPU hotspots first, and separately check for lock contention with tools that show time spent waiting (e.g., perf lock, or thread-sanitizer-adjacent contention profilers), since multi-threaded bottlenecks are as often about synchronization as raw compute. If contention is the issue, I'd look at reducing critical section size, using lock-free structures for hot counters, or sharding shared state to reduce contention rather than reaching for a bigger lock.


</details>

<details>
<summary><strong>Q015 · How would you design a caching layer for a service with strict memory constraints and unpredictable access patterns?</strong></summary>

**Answer guidance**: With unpredictable access, an adaptive eviction policy (e.g., ARC, which balances recency and frequency) tends to outperform plain LRU, which can thrash under scan-heavy or cyclic access patterns. I'd cap memory hard via a fixed-size structure rather than relying on soft limits, and monitor hit rate as a first-class metric to detect when the working set has outgrown the cache and whether resizing (or an entirely different tier) is warranted.


</details>

<details>
<summary><strong>Q016 · Explain your approach to choosing data structures for a system requiring both fast lookups and ordered iteration at scale.</strong></summary>

**Answer guidance**: A hash map alone gives fast lookup but no ordering; a balanced tree (e.g., a skip list or B-tree) gives both O(log n) lookup and ordered iteration in one structure, at some lookup-speed cost versus a pure hash map. If lookups vastly outnumber iteration, I'd consider maintaining both a hash map and a separate sorted structure, updated together, trading memory for speed on the dominant access pattern.


</details>

<details>
<summary><strong>Q017 · How would you design an efficient deduplication system for a high-throughput event pipeline?</strong></summary>

**Answer guidance**: For a fast first-pass filter, a Bloom filter (or counting Bloom filter if deletions are needed) gives constant-time, memory-efficient "definitely not seen" answers with a tunable false-positive rate; confirmed candidates get checked against an exact store (e.g., a keyed cache with TTL matching the dedup window). This two-tier approach keeps the hot path cheap while bounding incorrect dedup to an acceptable rate.


</details>

<details>
<summary><strong>Q018 · What's your approach to designing an API that must remain performant under both bursty and sustained high load?</strong></summary>

**Answer guidance**: I'd decouple ingestion from processing with a buffer/queue so bursts are absorbed rather than directly hitting downstream capacity limits, size that buffer based on measured burst patterns rather than guesswork, and apply backpressure/load shedding at the edge once buffers approach capacity so the system degrades gracefully (rejecting excess load) instead of falling over entirely.


</details>

<details>
<summary><strong>Q019 · As a Principal Engineer, how do you decide when a performance problem warrants an algorithmic fix vs. an infrastructure/hardware fix?</strong></summary>

**Answer guidance**: I look at whether the current approach has fundamentally poor complexity (e.g., O(n²) where O(n log n) exists) — that's an algorithmic problem no amount of hardware fixes economically, versus a constant-factor or throughput ceiling that's genuinely cheaper to solve by scaling infrastructure than by months of engineering effort. I'd quantify both costs (engineer-time to fix algorithmically vs. dollar cost to scale hardware) rather than defaulting to either option on instinct.


</details>

<details>
<summary><strong>Q020 · How would you lead a team through re-architecting a critical low-level service without regressing performance SLAs?</strong></summary>

**Answer guidance**: I'd insist on a benchmark suite reflecting real production traffic patterns before any architectural change begins, so every candidate change is validated against measured SLA impact rather than assumed improvement. I'd roll out changes incrementally (canary a subset of traffic) with automatic rollback triggers tied to the SLA metrics, rather than a single cutover where a regression is discovered only after full exposure.



</details>

## Booking.com

### 1. Core Coding Problems (itinerary reconstruction, caching)

<details>
<summary><strong>Q021 · Design an efficient algorithm to reconstruct a valid trip itinerary from a set of unordered flight/booking segments, handling cycles and invalid data.</strong></summary>

**Answer guidance**: This maps to finding an Eulerian path through a graph where segments are edges — I'd build an adjacency structure and use Hierholzer's algorithm to construct the path in O(E log E), which naturally handles the "use every segment exactly once" constraint. For invalid data (segments that don't form a connected path, or genuine cycles when only a linear itinerary is expected), I'd validate the graph's in/out-degree properties upfront and surface a clear error rather than silently returning a partial or wrong itinerary.


</details>

<details>
<summary><strong>Q022 · Design an insertion-based cache with O(1) operations that also supports eviction policies tuned for booking-search traffic patterns.</strong></summary>

**Answer guidance**: A doubly-linked list plus hash map gives O(1) insert/access/evict for classic LRU. For booking-search patterns specifically, pure recency (LRU) can underperform because popular destinations/dates get searched repeatedly in bursts — I'd consider an LFU-leaning or ARC-style hybrid that also weights frequency, since a plain-LRU cache can evict a highly popular but momentarily-not-most-recent search result in favor of a one-off query.


</details>

<details>
<summary><strong>Q023 · How would you design a caching strategy for search results that must stay fresh as hotel/flight inventory changes in near real-time?</strong></summary>

**Answer guidance**: I'd cache at a short TTL for pure availability/pricing data (since it changes frequently and staleness has direct customer/revenue impact) but cache more aggressively for stable descriptive data (hotel amenities, photos) that changes rarely. For high-value inventory changes (e.g., last room sold), I'd support active invalidation pushed from the inventory system rather than relying solely on TTL expiry, since waiting out a TTL on a sold-out room risks showing unavailable inventory.


</details>

<details>
<summary><strong>Q024 · What's your approach to designing a deduplication system for booking requests to prevent double-charging under retries?</strong></summary>

**Answer guidance**: I'd require clients to generate an idempotency key per booking attempt, and the server stores the outcome of the first request against that key, returning the cached result for any retry with the same key rather than reprocessing the charge. The key implementation detail is making the check-and-store atomic (e.g., a unique constraint in the database) so concurrent retries can't both slip through before either has recorded its result.


</details>

<details>
<summary><strong>Q025 · How would you design an efficient system to rank and return top-N search results under strict latency SLAs at massive scale?</strong></summary>

**Answer guidance**: I'd use a multi-stage retrieval-then-ranking pipeline: a cheap, broad filter/retrieval stage (using indexed structures to quickly narrow billions of candidates to a manageable set) followed by a more expensive, precise ranking model applied only to that smaller candidate set. Trying to run a complex ranking function over the full candidate space directly would blow the latency budget.


</details>

<details>
<summary><strong>Q026 · Explain your approach to designing a distributed lock/reservation system to prevent overbooking of limited inventory.</strong></summary>

**Answer guidance**: I'd use optimistic concurrency (a version/compare-and-swap on inventory count) rather than pessimistic distributed locks where possible, since locks introduce latency and failure-mode complexity at scale; the reservation decrements available inventory conditionally on the read version, and retries on conflict. For genuinely scarce, high-contention inventory (e.g., a single remaining unit), a short-lived pessimistic lock or a single-writer queue per inventory item avoids the retry storm optimistic concurrency would cause under heavy contention.


</details>

<details>
<summary><strong>Q027 · How would you architect a system to merge and reconcile inventory data from thousands of third-party suppliers?</strong></summary>

**Answer guidance**: I'd normalize each supplier's feed into a common internal schema at the ingestion boundary, rather than letting supplier-specific quirks leak into core systems, and track data provenance/freshness per supplier so conflicting data (two suppliers reporting different prices for effectively the same room) can be resolved by a defined precedence or freshness rule rather than ad hoc logic. Reconciliation jobs would flag anomalies (e.g., a supplier suddenly reporting wildly different prices) for review rather than blindly trusting every feed.


</details>

<details>
<summary><strong>Q028 · What data structures would you use to support fast, flexible search filtering (dates, price, location) across billions of listings?</strong></summary>

**Answer guidance**: I'd use a search engine with inverted indexes for categorical/text filters (location, amenities) combined with range-indexed structures (e.g., BKD-trees, as used in Lucene/Elasticsearch) for numeric/date range filters like price and date availability, since combining arbitrary filter combinations efficiently is exactly what these engines are optimized for versus rolling a custom solution.


</details>

<details>
<summary><strong>Q029 · As a Principal Engineer, how do you evaluate whether a caching problem needs a new caching layer vs. smarter invalidation logic?</strong></summary>

**Answer guidance**: I'd first check whether the pain is actually cache capacity/latency (genuinely needs a new layer or resizing) or staleness/correctness (needs better invalidation) — these are different problems people often conflate. Adding a new caching layer to paper over an invalidation bug just adds complexity without fixing the root cause, so I push for a clear diagnosis of which failure mode is actually occurring before proposing infrastructure changes.


</details>

<details>
<summary><strong>Q030 · How would you lead a redesign of a core booking algorithm while maintaining backward compatibility for partner integrations?</strong></summary>

**Answer guidance**: I'd version the external-facing API/contract explicitly and keep the old behavior available under the old version while the new algorithm ships under a new version or feature flag, giving partners a migration window with clear communication and deprecation timelines rather than a breaking change with no notice. Internally, I'd run both algorithms in shadow mode against real traffic to compare outputs before fully cutting partners over.


</details>
