# Booking.com Principal Engineer Interview Questions

This file contains the Booking.com loop, grouped by interview round. Each question includes model answer guidance.

## BOOKING.COM

### 1. Core Coding Problems (itinerary reconstruction, caching)

**1. Design an efficient algorithm to reconstruct a valid trip itinerary from a set of unordered flight/booking segments, handling cycles and invalid data.**
Model answer: This maps to finding an Eulerian path through a graph where segments are edges — I'd build an adjacency structure and use Hierholzer's algorithm to construct the path in O(E log E), which naturally handles the "use every segment exactly once" constraint. For invalid data (segments that don't form a connected path, or genuine cycles when only a linear itinerary is expected), I'd validate the graph's in/out-degree properties upfront and surface a clear error rather than silently returning a partial or wrong itinerary.

**2. Design an insertion-based cache with O(1) operations that also supports eviction policies tuned for booking-search traffic patterns.**
Model answer: A doubly-linked list plus hash map gives O(1) insert/access/evict for classic LRU. For booking-search patterns specifically, pure recency (LRU) can underperform because popular destinations/dates get searched repeatedly in bursts — I'd consider an LFU-leaning or ARC-style hybrid that also weights frequency, since a plain-LRU cache can evict a highly popular but momentarily-not-most-recent search result in favor of a one-off query.

**3. How would you design a caching strategy for search results that must stay fresh as hotel/flight inventory changes in near real-time?**
Model answer: I'd cache at a short TTL for pure availability/pricing data (since it changes frequently and staleness has direct customer/revenue impact) but cache more aggressively for stable descriptive data (hotel amenities, photos) that changes rarely. For high-value inventory changes (e.g., last room sold), I'd support active invalidation pushed from the inventory system rather than relying solely on TTL expiry, since waiting out a TTL on a sold-out room risks showing unavailable inventory.

**4. What's your approach to designing a deduplication system for booking requests to prevent double-charging under retries?**
Model answer: I'd require clients to generate an idempotency key per booking attempt, and the server stores the outcome of the first request against that key, returning the cached result for any retry with the same key rather than reprocessing the charge. The key implementation detail is making the check-and-store atomic (e.g., a unique constraint in the database) so concurrent retries can't both slip through before either has recorded its result.

**5. How would you design an efficient system to rank and return top-N search results under strict latency SLAs at massive scale?**
Model answer: I'd use a multi-stage retrieval-then-ranking pipeline: a cheap, broad filter/retrieval stage (using indexed structures to quickly narrow billions of candidates to a manageable set) followed by a more expensive, precise ranking model applied only to that smaller candidate set. Trying to run a complex ranking function over the full candidate space directly would blow the latency budget.

**6. Explain your approach to designing a distributed lock/reservation system to prevent overbooking of limited inventory.**
Model answer: I'd use optimistic concurrency (a version/compare-and-swap on inventory count) rather than pessimistic distributed locks where possible, since locks introduce latency and failure-mode complexity at scale; the reservation decrements available inventory conditionally on the read version, and retries on conflict. For genuinely scarce, high-contention inventory (e.g., a single remaining unit), a short-lived pessimistic lock or a single-writer queue per inventory item avoids the retry storm optimistic concurrency would cause under heavy contention.

**7. How would you architect a system to merge and reconcile inventory data from thousands of third-party suppliers?**
Model answer: I'd normalize each supplier's feed into a common internal schema at the ingestion boundary, rather than letting supplier-specific quirks leak into core systems, and track data provenance/freshness per supplier so conflicting data (two suppliers reporting different prices for effectively the same room) can be resolved by a defined precedence or freshness rule rather than ad hoc logic. Reconciliation jobs would flag anomalies (e.g., a supplier suddenly reporting wildly different prices) for review rather than blindly trusting every feed.

**8. What data structures would you use to support fast, flexible search filtering (dates, price, location) across billions of listings?**
Model answer: I'd use a search engine with inverted indexes for categorical/text filters (location, amenities) combined with range-indexed structures (e.g., BKD-trees, as used in Lucene/Elasticsearch) for numeric/date range filters like price and date availability, since combining arbitrary filter combinations efficiently is exactly what these engines are optimized for versus rolling a custom solution.

**9. As a Principal Engineer, how do you evaluate whether a caching problem needs a new caching layer vs. smarter invalidation logic?**
Model answer: I'd first check whether the pain is actually cache capacity/latency (genuinely needs a new layer or resizing) or staleness/correctness (needs better invalidation) — these are different problems people often conflate. Adding a new caching layer to paper over an invalidation bug just adds complexity without fixing the root cause, so I push for a clear diagnosis of which failure mode is actually occurring before proposing infrastructure changes.

**10. How would you lead a redesign of a core booking algorithm while maintaining backward compatibility for partner integrations?**
Model answer: I'd version the external-facing API/contract explicitly and keep the old behavior available under the old version while the new algorithm ships under a new version or feature flag, giving partners a migration window with clear communication and deprecation timelines rather than a breaking change with no notice. Internally, I'd run both algorithms in shadow mode against real traffic to compare outputs before fully cutting partners over.

### 2. Reliability & Distributed Systems (idempotency, retries, IPC, observability, scalability)

**1. How would you design an idempotency key system to safely allow clients to retry booking/payment requests?**
Model answer: The client generates a unique key per logical operation (not per HTTP attempt), and the server persists the key alongside the operation's result atomically with the operation itself (e.g., in the same database transaction), so a retry with the same key returns the stored result rather than re-executing. Keys need a defined expiry/scope so the table doesn't grow unbounded, and I'd make sure the uniqueness constraint is enforced at the database level, not just checked-then-inserted in application code, to avoid races.

**2. Explain your approach to designing retry and circuit-breaker strategies across a chain of dependent microservices.**
Model answer: Each hop in the chain needs its own bounded retry budget with exponential backoff and jitter, and a circuit breaker that opens on sustained failure to fail fast rather than let retries compound down the chain (a naive retry-at-every-hop design can multiply load by the depth of the chain during an outage). I'd also propagate a deadline/timeout budget from the original caller through the chain so downstream services know how much time is actually left, rather than each hop independently timing out on its own schedule.

**3. How would you design inter-process communication between services that must tolerate partial network partitions?**
Model answer: I'd favor asynchronous, message-based communication (a durable queue/stream) over synchronous RPC wherever the business logic allows it, since a queue can buffer during a partition and deliver once connectivity restores, whereas synchronous calls fail immediately. Where synchronous communication is unavoidable, I'd design the caller to handle partition-induced failures gracefully (timeout, fallback, or degraded response) rather than assuming the network is reliable.

**4. What's your strategy for building observability (tracing, metrics, alerting) into a system spanning hundreds of microservices?**
Model answer: I'd mandate distributed tracing with consistent trace-context propagation baked into shared client libraries (so individual teams can't forget it), standardize on a common metrics taxonomy (so cross-service dashboards are comparable), and set alerting on user-facing SLOs (latency, error rate) rather than internal implementation metrics, since SLO-based alerts correlate better with actual customer impact and reduce alert fatigue from noisy internal signals.

**5. How would you design a system to gracefully degrade (rather than fail) when a critical downstream dependency is unavailable?**
Model answer: I'd identify which parts of the user experience genuinely require the dependency versus which can fall back to cached, default, or simplified behavior (e.g., show cached prices with a "may not be current" notice rather than failing the whole search), and design those fallback paths deliberately rather than treating degradation as an afterthought. This requires explicitly deciding upfront what's essential vs. optional per feature, which is a product conversation as much as an engineering one.

**6. Explain your approach to capacity planning and load testing before a major seasonal traffic spike (e.g., holiday booking surge).**
Model answer: I'd load test against realistic traffic shapes (not just raw RPS but the actual mix of read/write, search/booking ratios seen historically during past spikes) well ahead of the event, identify the actual bottleneck component (often not the service you'd guess), and build in headroom plus autoscaling with pre-warmed capacity, since cold-starting infrastructure during the spike itself is too late. I'd also run a dry-run game day exercising the on-call response, not just the infrastructure.

**7. How would you design a reconciliation process to detect and correct data drift between two eventually-consistent systems?**
Model answer: I'd run periodic reconciliation jobs comparing checksums or record counts between the two systems at a granularity fine enough to localize drift quickly (not just a single global count), and define an explicit resolution policy (which system is authoritative, or a manual review queue for genuine conflicts) rather than silently auto-correcting in a way that might mask a real bug in the sync pipeline.

**8. What's your approach to designing multi-region failover for a booking system where data consistency is business-critical?**
Model answer: For the business-critical consistency-sensitive data (bookings, payments), I'd use a single-primary-per-partition design with synchronous or near-synchronous replication to a standby region, accepting some latency cost for the consistency guarantee, and a well-tested, ideally automated failover procedure with clear criteria for when to trigger it. I'd explicitly avoid multi-primary/active-active for this data given the overbooking/double-charge risk of conflicting writes across regions.

**9. As a Principal Engineer, how do you drive adoption of a reliability standard (e.g., SLOs, error budgets) across many independent teams?**
Model answer: I'd start with a small number of high-visibility services to prove the model works and build a track record before mandating it broadly, provide tooling that makes SLO tracking nearly free for teams to adopt (rather than manual spreadsheet-style tracking), and tie error budgets to a genuine incentive (e.g., a burned error budget pausing new feature launches until reliability recovers) so the standard has teeth rather than being purely aspirational.

**10. How would you balance strong consistency requirements for payments against the need for horizontal scalability?**
Model answer: I'd scale payments horizontally by partitioning (e.g., by account or booking ID) so each partition can maintain strong consistency independently via a single-writer model, rather than trying to achieve global strong consistency across the whole payment system, which doesn't scale. Cross-partition operations (rare, like a refund spanning accounts) would use a saga or two-phase-commit-style pattern scoped narrowly to those specific operations rather than applying that overhead system-wide.

### 3. System Design: Fraud Detection

**1. Design a real-time credit card fraud detection system — what are the key components and how do you balance latency vs. accuracy?**
Model answer: Key components: a real-time feature store (recent transaction velocity, device/IP reputation, historical patterns), a scoring layer combining fast rule-based checks with a ML model for nuanced scoring, and an action layer (approve/decline/step-up-verification) that must decide within a tight latency budget (often under a few hundred milliseconds) since this sits in the checkout path. I'd balance latency vs. accuracy by tiering: cheap rules catch the obvious cases instantly, and only ambiguous cases get routed to the more expensive model or, for borderline cases, a step-up challenge (e.g., 3D Secure) rather than blocking outright.

**2. How would you architect a feature pipeline that serves both real-time fraud scoring and offline model training consistently?**
Model answer: I'd build features through a shared feature-computation logic (ideally the same code or a shared feature store definition) used both online (low-latency, serving recent feature values) and offline (batch, for training), to avoid training/serving skew where the model is trained on features computed differently than how they're served in production — a common, hard-to-detect source of model degradation.

**3. What's your approach to designing a rules engine that can be updated by fraud analysts without redeploying code?**
Model answer: I'd build a declarative rules DSL or configuration format that analysts can author and test in a sandbox against historical transaction data before deployment, with rule changes going through a lightweight approval and audit trail (since these rules directly affect customer transactions and need traceability). Rules would be versioned and hot-reloadable in the scoring service rather than requiring a code deploy for every change.

**4. How would you design a system to explain fraud model decisions for compliance and customer dispute resolution?**
Model answer: For rule-based decisions, explainability is inherent (the triggered rule is the explanation). For ML-based scores, I'd use interpretable techniques (e.g., SHAP values or a simpler interpretable model for the final decision layer) so a specific decline can be traced to contributing factors, and I'd log the full feature vector and model version used for every decision so disputes can be investigated against exactly what the system saw at decision time.

**5. Explain your approach to handling adversarial behavior (fraud patterns that evolve to evade detection) at the architecture level.**
Model answer: I'd design for continuous retraining and rapid rule iteration rather than a static model, with monitoring specifically for score/feature distribution drift that might indicate fraudsters adapting. I'd also avoid over-relying on any single detection signal, since adversaries will optimize against whatever signal is publicly inferable (e.g., through repeated probing), so a layered defense (multiple independent signals) is more robust than one strong but singular detector.

**6. How would you design a feedback loop so fraud model performance improves continuously from confirmed fraud/chargeback data?**
Model answer: I'd pipe confirmed fraud/chargeback outcomes (which often arrive weeks after the original transaction) back into the training data with correct labels, being careful about label latency and selection bias (only outcomes we eventually learn about get labeled, and that population isn't a random sample). I'd retrain on a defined cadence and validate new model versions in shadow mode against live traffic before promoting them to production.

**7. What's your strategy for balancing false positives (blocking legitimate customers) against fraud losses at scale?**
Model answer: I'd frame this explicitly as a cost trade-off — quantify the cost of a false positive (lost revenue, customer trust) against the cost of a false negative (fraud loss), and tune the decision threshold to the point that minimizes total expected cost given those weights, rather than optimizing accuracy in the abstract. I'd also segment thresholds by risk context (e.g., transaction size, customer history) rather than a single global threshold, since the right trade-off differs by segment.

**8. How would you architect a fraud detection system to meet strict data privacy and regulatory requirements across regions?**
Model answer: I'd ensure feature computation and model training respect data residency (features derived from EU customer data stay within EU infrastructure, for example), minimize retention of sensitive raw data beyond what's needed for the fraud use case, and build access controls/audit logging around who can query raw transaction/fraud data, since this is exactly the kind of system regulators scrutinize closely.

**9. As a Principal Engineer, how do you evaluate whether to build fraud detection in-house vs. integrate a third-party provider?**
Model answer: I'd weigh how much fraud patterns are specific to our business (in-house tends to win when domain-specific signals matter a lot) against the multi-year investment in building comparable detection accuracy to a mature third-party provider that's seen fraud patterns across many customers. A common pragmatic path is starting with a third-party provider for baseline coverage while building in-house capability for our most business-specific fraud vectors over time, rather than an all-or-nothing choice.

**10. How would you lead cross-functional alignment between engineering, risk, and legal teams on a new fraud detection architecture?**
Model answer: I'd involve risk and legal early in the design phase, not just at a final review gate, since architectural decisions (data retention, explainability, what signals we can legally use) are genuinely constrained by their requirements, not just implementation details bolted on afterward. I'd translate technical trade-offs into terms each stakeholder cares about (risk in fraud-loss dollars, legal in compliance exposure, engineering in build/maintenance cost) so the group is making one shared decision rather than three separate ones that later conflict.

### 4. Culture Fit / Technical Leadership & Behavioral

**1. Tell me about a time you influenced a technical decision across teams that didn't report to you.**
Model answer: I'd structure this as a specific story: the situation, the technical disagreement or gap, how I built the case (data, prototype, or a written proposal) rather than relying on authority I didn't have, how I engaged the other team's engineers as partners rather than dictating, and the outcome — including what I'd do differently. The key theme to convey is earning influence through credibility and clear reasoning, not title.

**2. How do you know your API or system is actually working correctly in production, beyond passing tests?**
Model answer: Tests validate what I anticipated; production validates what I didn't. I'd point to monitoring real user-facing outcomes (success rates, latency distributions, business metrics like conversion) rather than just infrastructure health, canary/shadow deployments that compare new behavior against old on real traffic before full rollout, and a habit of actually looking at production data periodically rather than assuming "no alerts" means "all correct."

**3. Describe how you approach mentoring senior engineers who disagree with your architectural direction.**
Model answer: I'd genuinely engage with the disagreement rather than treating it as something to overcome — senior engineers who push back often see something I've missed. I'd ask them to make their strongest case, look for the kernel of validity even if I still disagree overall, and where we can't align, be explicit about "disagree and commit" versus continuing to litigate the decision after it's made, so the team isn't stuck in permanent debate.

**4. Tell me about a time you had to say no to a stakeholder's technical request and how you handled it.**
Model answer: I'd walk through a concrete example where I understood the underlying need behind the request (not just the literal ask), explained the specific technical or scaling risk of doing it as requested, and proposed an alternative that met the actual need with acceptable trade-offs — framing the "no" as "here's a better way to get what you need" rather than a flat refusal.

**5. How do you stay current technically while spending much of your time on leadership and cross-team alignment?**
Model answer: I'd be honest that this requires deliberate effort — dedicated time for hands-on work (even small), staying close to a few technical deep-dives rather than trying to be expert everywhere, and leaning on strong relationships with engineers doing the day-to-day work as a source of ground truth, since secondhand technical understanding degrades fast if not periodically refreshed with direct exposure.

**6. Describe a time you helped another engineer succeed technically — what was your approach?**
Model answer: I'd describe a specific mentoring situation, focusing on diagnosing what was actually blocking them (skill gap, confidence, unclear expectations, or a systemic issue like unclear ownership) rather than assuming it was purely technical, and how I tailored support accordingly — pairing, targeted feedback, or advocating for them to get a stretch opportunity — with a concrete, positive outcome.

**7. How do you give critical feedback on a design or code review in a way that maintains trust?**
Model answer: I focus feedback on the work, not the person, and ask questions ("what led you to this approach?") before asserting a better one, since that often surfaces context I was missing and shows respect for their reasoning. I also try to be as generous with praise for what's genuinely good as I am direct about what needs to change, so feedback doesn't read as uniformly negative even when the substance includes real critique.

**8. What does being a "T-shaped engineer" mean to you, and how have you demonstrated it?**
Model answer: I'd describe having deep expertise in one or two areas alongside broad enough competence across adjacent domains (front-end, infra, data) to collaborate effectively and make good system-level trade-offs without needing a specialist for every conversation. I'd back it with a concrete example — a project where my breadth let me spot a cross-domain issue (e.g., a database design decision that would hurt a downstream front-end use case) that a narrower specialist might have missed.

**9. Tell me about a technical decision you made that you later reversed — what did you learn?**
Model answer: I'd pick a genuine example, own the original reasoning honestly (why it seemed right at the time, not just "I was wrong"), describe what new information or changed circumstances prompted the reversal, and be specific about the lesson — often something like "I should have prototyped before committing" or "I underweighted operational cost relative to elegance" — rather than a vague platitude.

**10. What motivates you to want to be a Principal Engineer here specifically, versus staying in a narrower technical track?**
Model answer: I'd connect genuine personal motivation (wanting technical decisions I care about to have broader organizational impact, enjoying mentorship and cross-team problem-solving) to something specific about this company's technical challenges or culture, rather than a generic answer about "wanting more scope," since interviewers can tell when this hasn't been thought through specifically for their context.

---

*Tip: At Principal Engineer level, interviewers weigh your reasoning process, trade-off articulation, and cross-org influence more heavily than a single "correct" answer. Use these model answers as a scaffold for your own voice and real examples — reciting them verbatim will read as rehearsed rather than authoritative. Practice narrating your thought process out loud and be ready to defend a position under pushback.*
