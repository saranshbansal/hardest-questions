# System Design & Architecture

Questions about designing platforms, services, APIs, and cross-team architectural direction.

## Atlassian

### 2. System Design (real-time collaboration, issue tracking, notification pipelines, plugin architecture)

<details>
<summary><strong>Q001 · Design a real-time collaborative document editing system (like Confluence) — how do you handle conflict resolution and offline sync?</strong></summary>

**Answer guidance**: For conflict resolution I'd use either Operational Transformation or CRDTs; I lean CRDTs for simplicity of reasoning about convergence and easier offline support, accepting the trade-off of larger metadata overhead per edit. Offline clients would buffer local operations and replay them against the CRDT structure on reconnect, merging deterministically without a central arbiter deciding "who wins." For the collaboration transport, I'd use WebSockets with a fallback to long-polling, and a presence service so users see who else is editing. The interesting scaling problem is document sharding — very large or very hot documents (e.g., a company-wide wiki page) need special-cased handling so one document doesn't bottleneck a shared editing cluster.


</details>

<details>
<summary><strong>Q002 · Design an issue-tracking system (like Jira) that scales to millions of tickets across thousands of organizations — what are the sharding and indexing strategies?</strong></summary>

**Answer guidance**: I'd shard primarily by tenant (organization), since most queries are tenant-scoped and this keeps noisy neighbors isolated; within very large tenants, I'd further shard by project. For search/filtering, I'd maintain a separate indexed read store (search engine) fed by CDC from the source of truth, since Jira's flexible custom-field querying doesn't map well to a single relational index strategy. Cross-tenant admin queries (rare but real) would go through a separate analytics pipeline rather than the live transactional path, so they can't degrade normal ticket operations.


</details>

<details>
<summary><strong>Q003 · How would you design a notification pipeline that must support millions of users, multiple channels (email, Slack, in-app), and per-user preference rules?</strong></summary>

**Answer guidance**: I'd separate "event happened" from "notification delivered": events are published to a stream, a rules/preferences engine evaluates per-user routing and batching logic, and channel-specific delivery workers handle the actual send with channel-appropriate retry/backoff. Preference evaluation should be cacheable and versioned so a preference change takes effect quickly without re-querying a database on every event. I'd also build in digest/batching logic at the routing layer (not per-channel) so a user who wants a daily email digest doesn't get 50 individual emails from 50 individual events.


</details>

<details>
<summary><strong>Q004 · Design a plugin/marketplace architecture that allows third-party extensions without compromising platform security or performance.</strong></summary>

**Answer guidance**: I'd run third-party code in an isolated execution environment (sandboxed process, WASM runtime, or serverless function) with a narrow, capability-based API rather than direct access to internal services — plugins call a well-defined gateway, not our database. Resource limits (CPU, memory, request timeouts) need to be enforced per-plugin so a misbehaving extension can't degrade the host app; I'd also require static review or automated scanning of manifests declaring exactly what data/permissions a plugin requests, shown transparently to installing admins.


</details>

<details>
<summary><strong>Q005 · How would you design multi-region data residency for a SaaS platform serving enterprise customers with strict compliance requirements?</strong></summary>

**Answer guidance**: I'd pin a tenant's primary data to a specific region at provisioning time based on their residency requirement, and make that binding explicit and immutable without an active migration process — trying to make it silently "flexible" invites compliance bugs. Shared/global services (auth, billing) need careful design to either replicate per-region or prove they don't touch regulated data. The hardest part is usually not the data plane but the control plane — logs, backups, and support tooling that engineers use day-to-day also need to respect residency, which is often where compliance violations actually happen.


</details>

<details>
<summary><strong>Q006 · What architectural decisions would you make to support both strong consistency for billing/permissions and eventual consistency for activity feeds?</strong></summary>

**Answer guidance**: I'd treat these as genuinely different systems with different guarantees rather than forcing one data store to serve both. Billing and permissions go through a strongly consistent, likely single-writer-per-partition store (e.g., a relational database with proper transactions), because correctness failures there have direct financial/security consequences. Activity feeds are read-heavy and tolerant of staleness, so I'd let them run off an async event stream into a denormalized read store optimized for feed queries, explicitly accepting lag in exchange for scalability.


</details>

<details>
<summary><strong>Q007 · How would you design a search indexing pipeline that stays near-real-time as issues/documents are created and updated at high volume?</strong></summary>

**Answer guidance**: I'd use CDC (change data capture) off the primary datastore into a stream, with index writers consuming that stream and applying updates incrementally rather than batch reindexing. To handle bursty write volume, I'd decouple ingestion rate from indexing rate with a buffer, and monitor indexing lag as a first-class SLO, alerting when it exceeds a threshold users would notice. For rare full-corpus schema changes, I'd support blue-green reindexing into a parallel index and cut over atomically rather than mutating the live index in place.


</details>

<details>
<summary><strong>Q008 · Design a system for audit logging across a multi-product suite (Jira, Confluence, Bitbucket) with a unified query interface.</strong></summary>

**Answer guidance**: I'd define a common audit event schema (actor, action, resource, timestamp, product, metadata) that every product emits to, rather than trying to normalize disparate per-product logs after the fact. Events would land in an append-only, tamper-evident store (important for audit integrity) and be indexed for the unified query layer. Given audit logs are often compliance-critical and rarely deleted, I'd plan storage tiering — hot recent data queryable fast, older data in cheaper cold storage still queryable but with higher latency.


</details>

<details>
<summary><strong>Q009 · As Principal Engineer, how would you drive a cross-team architectural decision (e.g., migrating from REST to GraphQL) across multiple product lines?</strong></summary>

**Answer guidance**: I'd start by writing a concise architecture doc that states the problem being solved (not "GraphQL is better" but the specific pain — e.g., over-fetching, N+1 client calls) and the migration cost, then socialize it with the affected teams' senior engineers before it's a fait accompli, since buy-in from people who'll do the work matters more than a mandate from above. I'd propose a pilot on one bounded product surface, measure the actual impact, and use that evidence to build the case org-wide rather than asserting it up front. Where teams disagree, I'd rather narrow the scope of the migration than force adoption against strong technical objections.


</details>

<details>
<summary><strong>Q010 · How do you evaluate and justify a build-vs-buy decision for critical infrastructure (e.g., a workflow engine or search platform) to leadership?</strong></summary>

**Answer guidance**: I frame it in terms of total cost of ownership, not just initial build cost: engineering time to build, ongoing operational burden, opportunity cost of not shipping product features, and the risk of accumulating a bespoke system that becomes a hiring and onboarding liability. I'd present a small number of concrete options with honest trade-offs rather than a single recommendation dressed as inevitable, and tie the decision to business risk in language leadership can act on — e.g., "buy gets us to market in one quarter with vendor lock-in risk X; build gets us full control in three quarters with Y engineer-years of ongoing cost."



</details>

## NVIDIA

### 3. Team-Dependent Loop: Coding, System Design, Domain Knowledge

<details>
<summary><strong>Q011 · Design a high-throughput driver-level interface between application code and GPU hardware — what are the key abstraction boundaries?</strong></summary>

**Answer guidance**: I'd draw the boundary so the driver exposes a stable, minimal set of primitives (memory allocation, kernel launch, synchronization) while keeping hardware-specific details (register layouts, command encoding) fully hidden below that line, so application code and even higher driver layers can evolve independently of specific hardware generations. The interface needs careful design around asynchronous operation and explicit synchronization points, since forcing synchronous semantics at this layer would kill throughput.


</details>

<details>
<summary><strong>Q012 · How would you architect a scalable model-serving platform (e.g., for inference) that must minimize latency and maximize GPU utilization?</strong></summary>

**Answer guidance**: I'd use dynamic batching (accumulating requests within a small latency budget to form efficient batches) to improve GPU utilization without unacceptably hurting per-request latency, and support model instance multiplexing on a GPU (e.g., via MIG or concurrent execution) for models that don't need a full GPU's throughput alone. Autoscaling should be driven by queue depth and latency SLO breach risk, not just raw CPU/GPU utilization, since utilization alone doesn't capture user-facing latency.


</details>

<details>
<summary><strong>Q013 · Walk through designing DGX Cloud-style infrastructure for elastic, multi-tenant AI workloads.</strong></summary>

**Answer guidance**: I'd separate the control plane (tenant provisioning, quota management, billing) from the data plane (actual GPU clusters running workloads), with strong tenant isolation enforced at the scheduling and networking layers so tenants can't see or affect each other's workloads. Elasticity requires fast provisioning of GPU resources, which favors pre-warmed capacity pools over cold-starting hardware, along with a scheduler that can preempt lower-priority workloads for higher-priority elastic demand.


</details>

<details>
<summary><strong>Q014 · How would you design an autonomous vehicle perception pipeline's software architecture to meet strict real-time latency budgets?</strong></summary>

**Answer guidance**: I'd architect the pipeline as a set of bounded-latency stages (sensor fusion, object detection, tracking, prediction) with explicit deadline budgets per stage, and design for graceful degradation — if a stage can't complete within budget, the system should fall back to a safe, simpler output rather than blocking the whole pipeline. This requires careful hardware-software co-design, since perception latency at this level is as much about memory bandwidth and sensor I/O as raw compute.


</details>

<details>
<summary><strong>Q015 · What's your approach to designing observability for GPU cluster health across thousands of nodes?</strong></summary>

**Answer guidance**: I'd instrument GPU-specific metrics (utilization, memory, ECC errors, temperature, throttling events) alongside standard infra metrics, and build automated anomaly detection since manually watching thousands of dashboards doesn't scale — the goal is surfacing degraded nodes (silent data corruption from ECC errors, thermal throttling) before they cause a training job failure hours into a run, since restarting a multi-day job is expensive.


</details>

<details>
<summary><strong>Q016 · How would you evaluate whether a new hardware generation requires software architecture changes vs. drop-in compatibility?</strong></summary>

**Answer guidance**: I'd look at whether the new generation changes fundamental characteristics the software architecture assumes (e.g., memory hierarchy size ratios, interconnect topology) versus just improving throughput within the same model — the former requires architectural rework, the latter often just needs re-tuning parameters. I'd insist on running representative benchmarks on the new hardware early, since assumptions about "just faster" frequently break down at the margins.


</details>

<details>
<summary><strong>Q017 · Describe your approach to cross-functional collaboration between hardware and software teams when performance targets aren't being met.</strong></summary>

**Answer guidance**: I'd push for shared, agreed-upon instrumentation both teams trust, so debates aren't "your numbers vs. my numbers" but a shared dataset both sides interpret together. I'd also make sure the software team understands hardware constraints (and vice versa) well enough to jointly diagnose whether a shortfall is a hardware limit, a software inefficiency, or a mismatched expectation — often it's a bit of all three, and premature blame-assignment derails the actual fix.


</details>

<details>
<summary><strong>Q018 · How do you decide the right software abstraction layer to expose to internal ML teams without over-engineering?</strong></summary>

**Answer guidance**: I'd start from what ML teams actually need to iterate quickly (a small number of well-tested, high-level operations) rather than exposing every hardware knob defensively, and add lower-level escape hatches only when a real use case demonstrates the high-level abstraction is insufficient. Abstractions built speculatively ahead of demonstrated need tend to be wrong and get reworked anyway.


</details>

<details>
<summary><strong>Q019 · As a Principal Engineer, how would you influence a multi-team roadmap when your architectural recommendation conflicts with a shorter-term deadline?</strong></summary>

**Answer guidance**: I'd quantify the cost of the shortcut explicitly (what technical debt it creates, what it'll cost to unwind later) rather than simply asserting the "right" architecture, and present both paths with their trade-offs to the decision-makers rather than unilaterally blocking the deadline. Sometimes the deadline-driven path is genuinely the right call given business context I don't fully see — my job is making the trade-off visible, not always winning the argument.


</details>

<details>
<summary><strong>Q020 · How would you mentor senior engineers on making trade-offs between correctness, performance, and time-to-market in systems programming?</strong></summary>

**Answer guidance**: I'd encourage them to make trade-offs explicit and reversible where possible — e.g., ship the simpler-but-slower correct version first, instrument it, and only invest in performance optimization once data shows it's actually needed — rather than guessing upfront which corners are safe to cut. I'd also model this by narrating my own trade-off reasoning openly in reviews, rather than presenting decisions as already-settled.



</details>

## Booking.com

### 3. System Design: Fraud Detection

<details>
<summary><strong>Q021 · Design a real-time credit card fraud detection system — what are the key components and how do you balance latency vs. accuracy?</strong></summary>

**Answer guidance**: Key components: a real-time feature store (recent transaction velocity, device/IP reputation, historical patterns), a scoring layer combining fast rule-based checks with a ML model for nuanced scoring, and an action layer (approve/decline/step-up-verification) that must decide within a tight latency budget (often under a few hundred milliseconds) since this sits in the checkout path. I'd balance latency vs. accuracy by tiering: cheap rules catch the obvious cases instantly, and only ambiguous cases get routed to the more expensive model or, for borderline cases, a step-up challenge (e.g., 3D Secure) rather than blocking outright.


</details>

<details>
<summary><strong>Q022 · How would you architect a feature pipeline that serves both real-time fraud scoring and offline model training consistently?</strong></summary>

**Answer guidance**: I'd build features through a shared feature-computation logic (ideally the same code or a shared feature store definition) used both online (low-latency, serving recent feature values) and offline (batch, for training), to avoid training/serving skew where the model is trained on features computed differently than how they're served in production — a common, hard-to-detect source of model degradation.


</details>

<details>
<summary><strong>Q023 · What's your approach to designing a rules engine that can be updated by fraud analysts without redeploying code?</strong></summary>

**Answer guidance**: I'd build a declarative rules DSL or configuration format that analysts can author and test in a sandbox against historical transaction data before deployment, with rule changes going through a lightweight approval and audit trail (since these rules directly affect customer transactions and need traceability). Rules would be versioned and hot-reloadable in the scoring service rather than requiring a code deploy for every change.


</details>

<details>
<summary><strong>Q024 · How would you design a system to explain fraud model decisions for compliance and customer dispute resolution?</strong></summary>

**Answer guidance**: For rule-based decisions, explainability is inherent (the triggered rule is the explanation). For ML-based scores, I'd use interpretable techniques (e.g., SHAP values or a simpler interpretable model for the final decision layer) so a specific decline can be traced to contributing factors, and I'd log the full feature vector and model version used for every decision so disputes can be investigated against exactly what the system saw at decision time.


</details>

<details>
<summary><strong>Q025 · Explain your approach to handling adversarial behavior (fraud patterns that evolve to evade detection) at the architecture level.</strong></summary>

**Answer guidance**: I'd design for continuous retraining and rapid rule iteration rather than a static model, with monitoring specifically for score/feature distribution drift that might indicate fraudsters adapting. I'd also avoid over-relying on any single detection signal, since adversaries will optimize against whatever signal is publicly inferable (e.g., through repeated probing), so a layered defense (multiple independent signals) is more robust than one strong but singular detector.


</details>

<details>
<summary><strong>Q026 · How would you design a feedback loop so fraud model performance improves continuously from confirmed fraud/chargeback data?</strong></summary>

**Answer guidance**: I'd pipe confirmed fraud/chargeback outcomes (which often arrive weeks after the original transaction) back into the training data with correct labels, being careful about label latency and selection bias (only outcomes we eventually learn about get labeled, and that population isn't a random sample). I'd retrain on a defined cadence and validate new model versions in shadow mode against live traffic before promoting them to production.


</details>

<details>
<summary><strong>Q027 · What's your strategy for balancing false positives (blocking legitimate customers) against fraud losses at scale?</strong></summary>

**Answer guidance**: I'd frame this explicitly as a cost trade-off — quantify the cost of a false positive (lost revenue, customer trust) against the cost of a false negative (fraud loss), and tune the decision threshold to the point that minimizes total expected cost given those weights, rather than optimizing accuracy in the abstract. I'd also segment thresholds by risk context (e.g., transaction size, customer history) rather than a single global threshold, since the right trade-off differs by segment.


</details>

<details>
<summary><strong>Q028 · How would you architect a fraud detection system to meet strict data privacy and regulatory requirements across regions?</strong></summary>

**Answer guidance**: I'd ensure feature computation and model training respect data residency (features derived from EU customer data stay within EU infrastructure, for example), minimize retention of sensitive raw data beyond what's needed for the fraud use case, and build access controls/audit logging around who can query raw transaction/fraud data, since this is exactly the kind of system regulators scrutinize closely.


</details>

<details>
<summary><strong>Q029 · As a Principal Engineer, how do you evaluate whether to build fraud detection in-house vs. integrate a third-party provider?</strong></summary>

**Answer guidance**: I'd weigh how much fraud patterns are specific to our business (in-house tends to win when domain-specific signals matter a lot) against the multi-year investment in building comparable detection accuracy to a mature third-party provider that's seen fraud patterns across many customers. A common pragmatic path is starting with a third-party provider for baseline coverage while building in-house capability for our most business-specific fraud vectors over time, rather than an all-or-nothing choice.


</details>

<details>
<summary><strong>Q030 · How would you lead cross-functional alignment between engineering, risk, and legal teams on a new fraud detection architecture?</strong></summary>

**Answer guidance**: I'd involve risk and legal early in the design phase, not just at a final review gate, since architectural decisions (data retention, explainability, what signals we can legally use) are genuinely constrained by their requirements, not just implementation details bolted on afterward. I'd translate technical trade-offs into terms each stakeholder cares about (risk in fraud-loss dollars, legal in compliance exposure, engineering in build/maintenance cost) so the group is making one shared decision rather than three separate ones that later conflict.


</details>
