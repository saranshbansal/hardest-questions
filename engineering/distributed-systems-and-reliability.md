# Distributed Systems & Reliability

Questions about consistency, failure modes, resilience, observability, capacity, and operating services at scale.

## Atlassian

### 4. Networking / OS Fundamentals (rapid-fire style, elevated)

<details>
<summary><strong>Q001 · Explain how you'd diagnose intermittent latency spikes in a service that spans multiple availability zones.</strong></summary>

**Answer guidance**: I'd start with distributed tracing to see whether the spikes correlate with a specific downstream dependency, AZ, or time window (e.g., GC pauses, cross-AZ network hops, noisy-neighbor contention). Intermittent issues are often either resource contention (check CPU steal, GC, connection pool saturation) or a specific slow dependency that's masked by averages — I'd look at p99/p999 latency broken down by AZ and dependency before guessing.


</details>

<details>
<summary><strong>Q002 · Walk through what happens at the OS and network layer when a service experiences connection pool exhaustion under load.</strong></summary>

**Answer guidance**: New requests either queue waiting for a connection or get rejected/timed out, and if the pool is shared with health checks, those can start failing too, potentially triggering load balancer ejection and cascading load onto remaining healthy instances. At the OS level, if this is TCP connections, you may also see accumulating TIME_WAIT sockets or exhausted ephemeral ports if connections churn rather than reuse. The fix is usually a combination of right-sizing the pool, adding backpressure/queueing limits, and ensuring slow downstream calls have their own timeouts so they don't hold connections indefinitely.


</details>

<details>
<summary><strong>Q003 · How would you design a system's retry and backoff strategy to avoid cascading failures during a partial outage?</strong></summary>

**Answer guidance**: Exponential backoff with jitter is table stakes to avoid synchronized retry storms; beyond that I'd add a circuit breaker so a service stops hammering a clearly-failing dependency and fails fast instead, and bound total retry budget per request so retries don't multiply load during an outage exactly when the system is most fragile. I'd also make sure retries are only applied to idempotent operations.


</details>

<details>
<summary><strong>Q004 · Explain the trade-offs between TCP and UDP for a real-time collaboration feature's transport layer.</strong></summary>

**Answer guidance**: TCP gives ordered, reliable delivery which simplifies application logic but head-of-line blocking can hurt real-time responsiveness when a single lost packet stalls everything behind it. UDP (often via WebRTC data channels) avoids that stall and suits low-latency, loss-tolerant use cases, but pushes reliability and ordering concerns back to the application. For collaborative editing, I'd typically still use a reliable, ordered transport (WebSocket over TCP) since correctness of the edit stream matters more than shaving milliseconds, unless the specific feature (e.g., cursor position) is genuinely loss-tolerant.


</details>

<details>
<summary><strong>Q005 · How would you approach root-causing a memory leak in a long-running service without full production access?</strong></summary>

**Answer guidance**: I'd rely on whatever telemetry is exposed rather than live debugging — heap usage trends over time, GC frequency/pause metrics, and periodic heap snapshots or profiling exports if the runtime supports them (e.g., pprof, JVM heap dumps) triggered on a schedule or threshold. I'd correlate the leak's growth rate against deploy timestamps and traffic patterns to narrow down which code path or recent change is responsible, then reproduce in a staging environment with production-like load where I do have full access.


</details>

<details>
<summary><strong>Q006 · What's your strategy for load-balancing across regions when latency and data residency both matter?</strong></summary>

**Answer guidance**: I'd route by residency constraint first (hard requirement) and only optimize for latency within the set of regions a given tenant is legally allowed to use — residency isn't a tunable, it's a filter applied before any latency-based routing decision. Within allowed regions, geo-DNS or anycast plus health-aware routing handles the latency optimization.


</details>

<details>
<summary><strong>Q007 · How do you decide when to introduce a service mesh vs. simpler client-side load balancing?</strong></summary>

**Answer guidance**: A service mesh earns its complexity when you need consistent cross-cutting concerns (mTLS, retries, observability) across many polyglot services without every team reimplementing them — but it adds real operational overhead (sidecar resource cost, upgrade complexity). For a small number of services or a single-language stack, a good client-side library often gets 80% of the benefit at a fraction of the operational cost. I'd only push for a mesh once the number of services and languages makes per-service consistency untenable otherwise.


</details>

<details>
<summary><strong>Q008 · Explain how DNS resolution failures could cascade in a microservices architecture and how you'd mitigate that.</strong></summary>

**Answer guidance**: If services resolve each other via DNS and a DNS backend degrades, you can see a thundering herd of failed lookups and retries across the whole fleet simultaneously — a single point of failure hiding behind seemingly independent services. Mitigations include client-side DNS caching with sane TTLs, fallback resolvers, and treating DNS as a dependency worth its own SLO and monitoring rather than assuming it "just works."


</details>

<details>
<summary><strong>Q009 · How would you set organization-wide standards for observability (tracing, metrics, logging) across polyglot services?</strong></summary>

**Answer guidance**: I'd standardize on an open standard (e.g., OpenTelemetry) so instrumentation isn't tied to a specific vendor or language, provide shared libraries/wrappers per language that make "doing it right" the path of least resistance, and bake trace-context propagation into shared HTTP/RPC client libraries so individual teams don't have to remember to wire it up. Standards that require manual discipline from every team tend to decay; standards embedded in shared tooling tend to stick.


</details>

<details>
<summary><strong>Q010 · As a technical leader, how do you decide the right level of infrastructure abstraction for teams with varying operational maturity?</strong></summary>

**Answer guidance**: I'd offer a paved-road default (a well-supported, opinionated platform) for teams that want to move fast without deep infra expertise, while allowing an escape hatch for teams with the maturity and genuine need to go lower-level — but I'd make the escape hatch intentionally a bit more effortful, so it's a deliberate choice, not the path of least resistance for everyone. The goal is matching abstraction level to team need, not forcing uniformity.



</details>

## Booking.com

### 2. Reliability & Distributed Systems (idempotency, retries, IPC, observability, scalability)

<details>
<summary><strong>Q011 · How would you design an idempotency key system to safely allow clients to retry booking/payment requests?</strong></summary>

**Answer guidance**: The client generates a unique key per logical operation (not per HTTP attempt), and the server persists the key alongside the operation's result atomically with the operation itself (e.g., in the same database transaction), so a retry with the same key returns the stored result rather than re-executing. Keys need a defined expiry/scope so the table doesn't grow unbounded, and I'd make sure the uniqueness constraint is enforced at the database level, not just checked-then-inserted in application code, to avoid races.


</details>

<details>
<summary><strong>Q012 · Explain your approach to designing retry and circuit-breaker strategies across a chain of dependent microservices.</strong></summary>

**Answer guidance**: Each hop in the chain needs its own bounded retry budget with exponential backoff and jitter, and a circuit breaker that opens on sustained failure to fail fast rather than let retries compound down the chain (a naive retry-at-every-hop design can multiply load by the depth of the chain during an outage). I'd also propagate a deadline/timeout budget from the original caller through the chain so downstream services know how much time is actually left, rather than each hop independently timing out on its own schedule.


</details>

<details>
<summary><strong>Q013 · How would you design inter-process communication between services that must tolerate partial network partitions?</strong></summary>

**Answer guidance**: I'd favor asynchronous, message-based communication (a durable queue/stream) over synchronous RPC wherever the business logic allows it, since a queue can buffer during a partition and deliver once connectivity restores, whereas synchronous calls fail immediately. Where synchronous communication is unavoidable, I'd design the caller to handle partition-induced failures gracefully (timeout, fallback, or degraded response) rather than assuming the network is reliable.


</details>

<details>
<summary><strong>Q014 · What's your strategy for building observability (tracing, metrics, alerting) into a system spanning hundreds of microservices?</strong></summary>

**Answer guidance**: I'd mandate distributed tracing with consistent trace-context propagation baked into shared client libraries (so individual teams can't forget it), standardize on a common metrics taxonomy (so cross-service dashboards are comparable), and set alerting on user-facing SLOs (latency, error rate) rather than internal implementation metrics, since SLO-based alerts correlate better with actual customer impact and reduce alert fatigue from noisy internal signals.


</details>

<details>
<summary><strong>Q015 · How would you design a system to gracefully degrade (rather than fail) when a critical downstream dependency is unavailable?</strong></summary>

**Answer guidance**: I'd identify which parts of the user experience genuinely require the dependency versus which can fall back to cached, default, or simplified behavior (e.g., show cached prices with a "may not be current" notice rather than failing the whole search), and design those fallback paths deliberately rather than treating degradation as an afterthought. This requires explicitly deciding upfront what's essential vs. optional per feature, which is a product conversation as much as an engineering one.


</details>

<details>
<summary><strong>Q016 · Explain your approach to capacity planning and load testing before a major seasonal traffic spike (e.g., holiday booking surge).</strong></summary>

**Answer guidance**: I'd load test against realistic traffic shapes (not just raw RPS but the actual mix of read/write, search/booking ratios seen historically during past spikes) well ahead of the event, identify the actual bottleneck component (often not the service you'd guess), and build in headroom plus autoscaling with pre-warmed capacity, since cold-starting infrastructure during the spike itself is too late. I'd also run a dry-run game day exercising the on-call response, not just the infrastructure.


</details>

<details>
<summary><strong>Q017 · How would you design a reconciliation process to detect and correct data drift between two eventually-consistent systems?</strong></summary>

**Answer guidance**: I'd run periodic reconciliation jobs comparing checksums or record counts between the two systems at a granularity fine enough to localize drift quickly (not just a single global count), and define an explicit resolution policy (which system is authoritative, or a manual review queue for genuine conflicts) rather than silently auto-correcting in a way that might mask a real bug in the sync pipeline.


</details>

<details>
<summary><strong>Q018 · What's your approach to designing multi-region failover for a booking system where data consistency is business-critical?</strong></summary>

**Answer guidance**: For the business-critical consistency-sensitive data (bookings, payments), I'd use a single-primary-per-partition design with synchronous or near-synchronous replication to a standby region, accepting some latency cost for the consistency guarantee, and a well-tested, ideally automated failover procedure with clear criteria for when to trigger it. I'd explicitly avoid multi-primary/active-active for this data given the overbooking/double-charge risk of conflicting writes across regions.


</details>

<details>
<summary><strong>Q019 · As a Principal Engineer, how do you drive adoption of a reliability standard (e.g., SLOs, error budgets) across many independent teams?</strong></summary>

**Answer guidance**: I'd start with a small number of high-visibility services to prove the model works and build a track record before mandating it broadly, provide tooling that makes SLO tracking nearly free for teams to adopt (rather than manual spreadsheet-style tracking), and tie error budgets to a genuine incentive (e.g., a burned error budget pausing new feature launches until reliability recovers) so the standard has teeth rather than being purely aspirational.


</details>

<details>
<summary><strong>Q020 · How would you balance strong consistency requirements for payments against the need for horizontal scalability?</strong></summary>

**Answer guidance**: I'd scale payments horizontally by partitioning (e.g., by account or booking ID) so each partition can maintain strong consistency independently via a single-writer model, rather than trying to achieve global strong consistency across the whole payment system, which doesn't scale. Cross-partition operations (rare, like a refund spanning accounts) would use a saga or two-phase-commit-style pattern scoped narrowly to those specific operations rather than applying that overhead system-wide.


</details>
