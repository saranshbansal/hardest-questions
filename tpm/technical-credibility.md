# TPM · Technical Credibility

Technical conversations, architecture awareness, delivery risk, and production readiness.

> **Role track:** Technical Program Manager (TPM) — these questions assess programme leadership, technical credibility, and cross-team execution. They are intentionally separate from the engineering interview question bank.

<details>
<summary><strong>TPM-T001 · Your programme involves multiple teams building services that need to communicate via REST APIs. Walk me through how you would ensure API design consistency across teams.</strong></summary>

**Interview prompts**

1. How would you approach API versioning across multiple teams, and what contract management practices would you put in place?
2. What steps would you take to ensure backward compatibility when APIs need to evolve?
3. How would you detect and resolve integration issues early - before they become programme-level blockers?

**Follow-up**

Idempotency is a practical concern in distributed systems. Can you explain why it matters for APIs in a multi-service programme, and how you would ensure teams account for it in their designs?

**What this tests**

Tests whether the TPM can have credible technical conversations about API design while focusing on programme-level coordination contract management, cross-team consistency, and early detection of integration risk.

**Evaluator guide**

Do they mention API design standards, style guides, or shared schema definitions (e.g., OpenAPI/Swagger)? Do they discuss versioning strategies (URL path versioning, header-based versioning) and when to use each? Do they understand contract testing or consumer-driven contracts as a way to catch breaking changes early? TPM signal: Do they frame this as a cross-team coordination problem, not just a technical one? TPM signal: Do they talk about governance who owns the API standards, how are exceptions handled, who approves breaking changes? TPM signal: Do they mention integration environments, cross-team integration testing cadence, or dependency mapping? Red flag: Gets lost in low-level HTTP semantics without connecting to programme delivery. Red flag: No plan for detecting integration issues before they reach production.

**Example strong answer**

"API consistency across teams is fundamentally a coordination problem, not just a technical one. I'd approach it in three layers: standards, contracts, and early detection. First, standards. I'd work with the engineering leads to establish a shared API style guide naming conventions, error response format, pagination patterns, authentication headers. I'd push for an OpenAPI/Swagger spec as the source of truth for every API. This becomes the shared language between teams. I'd set up a lightweight review process where any new API or breaking change to an existing one goes through a 30-minute cross-team design review before implementation starts. For versioning, I'd advocate for URL path versioning (/v1/, /v2/) because it's the most visible and easiest for teams to manage across environments. The critical rule is: existing versions must remain backward compatible. If you need to remove a field or change behaviour, that's a new version. I'd track API version lifecycles on the programme roadmap so deprecation doesn't surprise anyone. Second, contracts. I'd introduce consumer-driven contract testing each consuming team writes tests that describe what they expect from the provider's API. These tests run in CI on the provider's side. If a provider change breaks a consumer's contract, CI fails before the change is merged. This catches integration issues at development time rather than in a shared test environment. Third, early detection. I'd establish a regular integration test cadence at minimum, a shared integration environment where all services deploy daily and automated integration tests run. I'd track integration test pass rates as a programme health metric. When failures spike, that's an early warning. On idempotency this is critical in distributed systems because network failures, retries, and timeouts mean the same request can be sent multiple times. If a POST to create a payment isn't idempotent, a network retry could create two charges. I'd ensure teams design idempotent endpoints by requiring an idempotency key (a client-generated unique ID) for any state-changing operation. I'd add this to our API standards checklist and verify it during design reviews."

</details>

<details>
<summary><strong>TPM-T002 · Your programme has 4 teams contributing to the same product with bi-weekly releases. Describe how you would structure the branching strategy and environment pipeline.</strong></summary>

**Interview prompts**

1. How would you enable parallel development across teams while minimising merge conflicts and integration risk?
2. What testing stages and environments would you set up between development and production?
3. How would you coordinate release readiness across teams, and what criteria would you use for a go/no-go decision?

**Follow-up**

One team's feature is not ready by the release cut-off, but the other three teams' work is. How do you handle this situation without delaying the entire release?

**What this tests**

Tests the TPM's ability to coordinate multi-team release processes. The follow-up specifically tests whether they've built in feature isolation a hallmark of mature programme management.

**Evaluator guide**

Do they describe a branching model that supports parallel work (trunk-based with feature flags, GitFlow, or similar)? Do they mention environment progression: dev UT, integration/staging, pre-prod, prod? Do they discuss release readiness criteria: tests passing, performance benchmarks, security scans, stakeholder sign-off? TPM signal: Do they talk about feature flags or feature toggles for isolating incomplete work? TPM signal: Do they describe a release coordination cadence release planning meetings, cutoff dates, go/no-go ceremonies? TPM signal: Do they mention rollback plans and release validation? Red flag: No mechanism for isolating features everything ships together or not at all. Red flag: No clear go/no-go criteria release decisions are ad-hoc.

**Example strong answer**

"With 4 teams on bi-weekly releases, I'd structure this around trunk-based development with feature flags, supported by a clear environment pipeline and release coordination cadence. Branching strategy: all teams develop on short-lived feature branches off main. Branches should live no more than 2-3 days before merging back. This minimises merge conflicts and forces continuous integration. Feature flags wrap any incomplete or risky work, so code can be merged to main without being visible to users. This is critical because it decouples deployment (getting code to production) from release (making it available to users). Environment pipeline: I'd set up four stages: Development: each team has their own dev environment for fast iteration. Integration: all teams' code merges here daily. Automated integration tests run. This is where cross-team issues surface. Staging/Pre-prod: mirrors production configuration. Performance testing, security scanning, and UAT happen here. Release candidates are promoted from integration to staging. Production: final deployment with canary or blue-green strategy. Release coordination: I'd establish a release cadence with clear milestones: Day 1 of sprint: release planning what's targeting this release, what are the dependencies? End of sprint week 1: feature freeze for the release. All code targeting this release must be merged to main. Sprint week 2: stabilisation. Bug fixes only. Release candidate deployed to staging. Day before release: go/no-go meeting. Criteria: all automated tests pass, no P1/P2 bugs outstanding, performance within thresholds, security scan clean, rollback plan documented, on-call team briefed. For the follow-up this is exactly why feature flags exist. If one team's feature isn't ready, we disable that flag and release the other three teams' work on schedule. The incomplete feature stays in the codebase but is invisible to users. The team continues working on it and targets the next release. This avoids the painful choice between delaying everyone and shipping a broken feature. If the programme doesn't have feature flags yet, I'd make implementing them a priority they're one of the highest-leverage investments for multiteam release coordination."

</details>

<details>
<summary><strong>TPM-T003 · You're coordinating a programme across 6 microservices owned by different teams. What architectural coupling risks would you watch for?</strong></summary>

**Interview prompts**

1. What are the signs of tight coupling between microservices, and how would you identify them as a TPM?
2. Once you identify coupling risks, what actions would you take - and who would you involve - to address them?
3. How would you structure cross-team dependency tracking to make coupling visible before it causes delivery problems?

**Follow-up**

A team tells you they need to make a synchronous call to another team's service for every request. What concerns would you raise, and how would you facilitate a discussion about alternatives?

**What this tests**

Tests architectural awareness at TPM level not designing the system, but understanding enough to spot delivery risks caused by architectural decisions.

**Evaluator guide**

Do they identify coupling indicators: synchronous dependencies, shared databases, coordinated deployments, shared data models? Do they understand the delivery impact of coupling one team blocks another, changes cascade across teams? Do they mention dependency matrices, service interaction diagrams, or architecture decision records? TPM signal: Do they frame coupling as a delivery risk, not just a technical concern? TPM signal: Do they talk about involving architects or tech leads rather than making the technical decision themselves? TPM signal: Do they mention tracking dependencies in programme planning identifying critical paths through service dependencies? Red flag: Cannot identify what tight coupling looks like in practice. Red flag: Tries to make the architectural decision rather than facilitating the discussion.

**Example strong answer**

"As a TPM, I'm not designing the architecture, but I need to understand coupling risks because they directly translate into delivery risks. When services are tightly coupled, teams lose autonomy they can't deploy independently, they can't change their service without coordinating with others, and one team's delay cascades to everyone. Signs of tight coupling I'd watch for: Coordinated deployments: if service A can't deploy without service B deploying at the same time, they're coupled. I'd track deployment patterns are teams deploying independently or always in lockstep? Shared databases: if two services read from and write to the same database tables, changes to the schema require coordination. This is one of the most common and dangerous forms of coupling. Synchronous chains: if a user request triggers a chain of synchronous calls across 4 services, the system's reliability is the product of all 4 services' reliability. This also means one slow service makes everything slow. Shared models: if teams are sharing data transfer objects or code libraries that change frequently, every change requires all consumers to update. High inter-team communication overhead: if the same two teams are constantly in meetings resolving integration issues, their services are likely coupled. How I'd identify these: I'd maintain a service dependency map a visual diagram showing which services call which, and whether those calls are synchronous or asynchronous. I'd review this with engineering leads quarterly. I'd also track how often teams need to coordinate deployments and how often integration issues arise in cross-team standups. When I identify coupling risks, I wouldn't prescribe the solution I'd facilitate the conversation. I'd bring the relevant tech leads together, present the delivery data (e.g., 'Teams A and B have had 5 blocked stories in the last 3 sprints due to shared database dependencies'), and ask them to propose solutions. Options might include: introducing an event-driven pattern to decouple, splitting the shared database, or creating a dedicated API for the shared data. For dependency tracking, I'd maintain a programme-level dependency board. Every cross-team dependency gets logged: what team A needs from team B, by when, and what happens to the programme if it's late. I'd review this in weekly programme syncs and flag any dependency with less than two weeks of buffer. On the synchronous call follow-up: my concerns would be about reliability and latency impact, plus the delivery dependency it creates. If team B's service is down, team A's service is also down. I'd facilitate a discussion with both teams' tech leads about alternatives could this be an asynchronous call with eventual consistency? Could team A cache the data it needs? Could team A own a read replica? The goal is to find an approach that gives team A what it needs without creating a runtime and delivery dependency on team B."

</details>

<details>
<summary><strong>TPM-T004 · You're leading a programme to build a customer-facing platform within six months. Walk me through how you would approach the technical planning phase.</strong></summary>

**Interview prompts**

1. What questions would you ask engineering leads to understand the technical landscape and identify risks early?
2. How would you structure the architecture review process to ensure quality without creating bottlenecks?
3. How would you translate technical plans into a programme roadmap with meaningful milestones?

**Follow-up**

The engineering leads disagree on a fundamental architecture choice one favours a monolithic approach for speed, the other wants microservices for scalability. How do you facilitate this decision?

**What this tests**

Tests the TPM's ability to drive technical planning without being the technical decisionmaker. The follow-up specifically tests facilitation skills when technical leaders disagree.

**Evaluator guide**

Do they ask the right questions: scale requirements, existing systems, team capabilities, integration points, security/compliance needs? Do they actively clarify MVP scope—such as required features or number of products—or do they jump straight into designing full-scale architecture without considering an MVP approach? Do they describe a structured review process with clear decision points? Do they create milestones that are verifiable (not just "Phase 1 complete" but "API contracts defined and validated by consumer teams")? TPM signal: Do they separate technical decisions from programme planning, understanding that their role is to facilitate the former and own the latter? TPM signal: Do they talk about Architecture Decision Records (ADRs) or similar mechanisms for documenting decisions? TPM signal: Do they create decision deadlines to prevent analysis paralysis? Red flag: Tries to make the architecture decision themselves. Red flag: Milestones are vague or not verifiable.

**Example strong answer**

"Technical planning for a six-month programme needs to be structured but time-boxed. I'd aim to complete the technical planning phase in 3-4 weeks, not let it expand indefinitely. Questions I'd ask engineering leads: What existing systems does this need to integrate with, and what state are those systems in? What are the expected scale requirements at launch and 12 months post-launch? Are we designing for 10,000 users or 10 million? What are the non-negotiable technical constraints security compliance, data residency, existing technology standards? Where are the biggest technical unknowns, and what do we need to prove out early? What are the team's current capabilities do we have the skills for the proposed architecture, or do we need to upskill or hire? What's the data model, and where does data flow in and out of this platform? Architecture review process: I'd structure it in two stages: 1. _High-level design review (Week 1-2): engineering leads present the proposed_ architecture to a cross-functional group - other tech leads, security, infrastructure, and product. Focus on: does this meet the requirements? What are the biggest risks? _What alternatives were considered? Document the decision in an ADR._ 2. _Detailed design reviews (Week 2-4): each component or service gets a focused_ _review with the implementing team and key stakeholders. These are lighter-weight_ and can run in parallel. I'd avoid a single architecture review board that becomes a bottleneck. Instead, I'd define clear criteria for what needs a full review (new services, new data stores, securitysensitive components) versus what teams can decide independently. Translating to a programme roadmap: I'd work with engineering leads to identify natural milestones that are verifiable: Week 4: API contracts defined and validated by all consumer teams. Week 8: Core services deployed to integration environment, end-to-end smoke test passing. Week 12: Feature-complete in staging, performance testing started. Week 16: UAT complete, production readiness review passed. Week 20: Controlled launch to beta users. Week 24: General availability. Each milestone has clear, measurable criteria not 'mostly done' but specific checkpoints. For the architecture disagreement: my role isn't to pick the winner; it's to facilitate a structured decision. I'd ask both leads to present their case against the same criteria: time to first delivery, scalability needs at launch vs. 12 months, team capability, operational complexity. I'd set a decision deadline 'We're making this call by Friday' because architecture debates can go on forever. If they can't agree, I'd escalate to a chief architect or engineering director for a tiebreaker. What I wouldn't do is let the programme stall while two engineers debate in Slack for three weeks."

</details>

<details>
<summary><strong>TPM-T005 · Your programme is consistently slipping on delivery dates. Engineering leads tell you technical debt is a major contributor but product leadership won't prioritise it.</strong></summary>

**Interview prompts**

1. How would you quantify the impact of technical debt in terms that product leadership understands?
2. What strategies would you use to negotiate dedicated capacity for debt reduction without stalling feature delivery?
3. How would you build a sustainable approach to managing technical debt across the programme going forward?

**Follow-up**

An engineering lead wants to pause all feature work for a full sprint to address tech debt. Product leadership says absolutely not. How do you broker a path forward that both sides can support?

**What this tests**

Tests the TPM's ability to translate between engineering and business language, and to broker compromises between competing priorities.

**Evaluator guide**

Do they quantify debt in business terms: velocity decline, increased time-to-market, deployment risk, incident frequency? Do they propose allocation strategies: percentage of capacity, boy scout rule, dedicated sprints? Do they address the relationship between technical debt and programme predictability? TPM signal: Do they act as a bridge between engineering and product, not taking sides? TPM signal: Do they use data velocity trends, incident rates, deployment times to make the case? TPM signal: Do they make debt visible on the programme board rather than hiding it? Red flag: Takes engineering's side without translating to business impact. Red flag: Dismisses technical debt as an engineering concern, not a programme concern.

**Example strong answer**

"This is one of the most common and important challenges a TPM faces bridging the gap between engineering reality and product expectations. The key is to stop calling it 'technical debt' when talking to product leadership and start talking about delivery predictability and velocity. Quantifying impact: I'd gather concrete data: Velocity trend: 'Six months ago, our teams averaged 40 story points per sprint. Today it's 25. That's a 37% decline. At this rate, the Q4 roadmap is not deliverable.' Time-to-market: 'Adding a new payment method used to take 2 weeks. The last one took 6 weeks because of the state of the payments module.' Incident frequency: 'We've had 8 production incidents this quarter caused by fragile code. Each one cost N hours of engineering time and affected X customers.' Deployment risk: 'Our deployment failure rate has increased from 5% to 25%. Each failed deployment delays the release by a day.' I'd present this as a programme risk, not an engineering wish: 'At our current trajectory, we will miss the Q4 deadline by 6 weeks unless we invest in reducing the friction that's slowing teams down.' Negotiation strategies: Percentage allocation: propose dedicating 15-20% of each sprint to technical improvements. Product still gets 80% of capacity. Frame it as an investment in future velocity 'Spending 20% now means we'll be 30% faster in three months.' Boy scout rule: every PR leaves the code better than it found it. This costs almost nothing incrementally and prevents debt from growing. Targeted tech debt sprints: every 4th sprint is dedicated to tech improvements. Product gets a predictable schedule they can plan around. Making it sustainable: I'd make tech debt visible. It goes on the same programme board as feature work, with the same tracking and reporting. I'd classify debt items by impact (how much it slows us down) and effort (how long to fix), then work with engineering leads to prioritise the highest-impact, lowest-effort items first quick wins that demonstrate value to product stakeholders. For the follow-up brokering scenario: I'd propose a compromise. Instead of a full feature freeze sprint, dedicate 50% of the next sprint to the highest-impact debt items and 50% to features. Track the velocity improvement. If the data shows a meaningful bounce-back, use that as evidence to negotiate ongoing capacity. I'd frame it to product as: 'We're not stopping feature work. We're investing one sprint at reduced capacity so the next six sprints are significantly faster.' That's a proposition most product leaders can accept."

</details>

<details>
<summary><strong>TPM-T006 · How do you evaluate whether a feature is truly ready for production release?</strong></summary>

**Interview prompts**

1. What production readiness criteria would you define, and how would you ensure teams follow them consistently?
2. How do you balance thoroughness in readiness checks with the pressure to release quickly?
3. What role should monitoring, rollback plans, and operational runbooks play in your definition of "done"?

**Follow-up**

A team insists their feature is ready because all tests pass, but there's no monitoring, no alerting, and no runbook. How do you handle this conversation without blocking the release indefinitely?

**What this tests**

Tests whether the TPM thinks about production readiness as a programme concern, not just an engineering checkbox. The follow-up tests their ability to hold the line on quality while maintaining relationships.

**Evaluator guide**

Do they define concrete readiness criteria beyond "tests pass" monitoring, alerting, runbooks, rollback plans, performance validation? Do they mention production readiness reviews or checklists? Do they distinguish between "feature complete" and "production ready"? TPM signal: Do they talk about building production readiness into the delivery process rather than checking it at the end? TPM signal: Do they mention creating a shared Definition of Done across the programme? TPM signal: Do they balance risk with delivery pragmatism? Red flag: Production readiness is just "QA signed off." Red flag: No awareness of operational readiness (monitoring, rollback, runbooks).

**Example strong answer**

_"Tests passing is necessary but nowhere near sufficient for production readiness. I define production readiness across four dimensions:_ _1. Functional readiness: all acceptance criteria met, tests passing, no open P1/P2 bugs, UAT signed off by product._ _2. Operational readiness: this is where most teams fall short._ Monitoring: key metrics dashboards exist request rate, error rate, latency percentiles. Alerting: alerts configured for anomalous behaviour with clear thresholds and routing to the right on-call team. Runbook: documented steps for the most likely failure scenarios. Not a novel a onepage decision tree. 'If you see X, do Y.' Rollback plan: documented and tested. 'If this goes wrong, here's how we revert within 15 minutes.' _3. Performance readiness: load tested at expected traffic levels plus headroom. No regressions in latency or resource consumption._ _4. Security readiness: security review completed for any new attack surface. No open_ critical or high vulnerabilities. _To ensure teams follow these consistently, I'd create a production readiness checklist that's part of the release process. Before any feature enters the release candidate, the team reviews the checklist. I'd make the checklist lightweight a one-page document, not_ a 50-item audit. The goal is to make it easy to follow, not bureaucratic. _Balancing thoroughness with speed: I tier the requirements based on risk. A cosmetic UI_ change doesn't need load testing and an operational runbook. A new payment processing feature absolutely does. The checklist has mandatory items (applies to everything) and _risk-based items (applies based on the nature of the change). This prevents over-_ engineering the process for low-risk changes. _For the follow-up conversation: I wouldn't block the release, but I would have a direct_ conversation. I'd say: 'If this feature breaks in production at 2am and we have no monitoring, no alerts, and no runbook, how do we find out and how do we fix it? The answer right now is we don't find out until a customer complains, and then whoever is on call has to figure it out from scratch.' Then I'd offer a pragmatic compromise: 'Can we get basic monitoring and a one-page runbook in place by tomorrow? That's four hours of work and it transforms our ability to operate this feature safely.' I'd frame it as protecting the _team, not blocking them."_

</details>

<details>
<summary><strong>TPM-T007 · Your programme includes a new data-intensive feature. The engineering team is debating between SQL and NoSQL approaches. As a TPM, how do you facilitate this decision?</strong></summary>

**Interview prompts**

1. What factors would you want the team to consider when evaluating SQL versus NoSQL for this use case?
2. How would you ensure the decision is made based on data and requirements rather than personal preference?
3. What would you do if the teams involved have different levels of expertise with the chosen technology?

**Follow-up**

The team picks a technology the organisation has never used in production before. What additional programme risks does this introduce, and how would you mitigate them?

**What this tests**

Tests whether the TPM understands enough about data technology to ask the right questions, without needing to make the decision themselves. The follow-up tests risk awareness around technology adoption.

**Evaluator guide**

Do they identify key factors: data structure (structured vs. unstructured), query patterns, consistency requirements, scale, team expertise? Do they understand the trade-offs at a conceptual level (ACID vs. eventual consistency, joins vs. denormalization)? Do they mention documenting the decision rationale? TPM signal: Do they facilitate the decision by ensuring the right criteria are evaluated, rather than making the call themselves? TPM signal: Do they consider team capability as a factor alongside technical fit? TPM signal: Do they think about programme risk when introducing new technology learning curve, operational unknowns, hiring implications? Red flag: Has no understanding of the difference between SQL and NoSQL. Red flag: Makes the decision without involving engineering/ technical leads.

**Example strong answer**

"My role here isn't to pick the database it's to ensure the decision is well-structured, considers the right factors, and is documented. I've seen too many technology decisions made based on what engineers used at their last company rather than what fits the current problem. Factors I'd want the team to evaluate: Data model: is the data highly structured with relationships (orders, customers, products) or more document-oriented with variable schemas (user profiles, content, logs)? Query patterns: do we need complex joins and aggregations, or primarily key-value lookups and full-document reads? Consistency requirements: does this use case require strong consistency (financial transactions) or is eventual consistency acceptable (social media feeds)? Scale projections: what's the expected data volume at launch and at 12 months? What's the read-to-write ratio? Team expertise: which technology does the team already know? What's the cost of ramping up on something new? Operational maturity: do we have the infrastructure and tooling to operate this in production? Backups, monitoring, alerting, scaling procedures? To ensure the decision is data-driven, I'd structure it as a decision record: 1. _Define evaluation criteria upfront - before anyone proposes a solution._ 2. _Each option (SQL, NoSQL, or hybrid) is evaluated against the same criteria._ 3. _Weight the criteria based on this specific use case._ 4. _Document the decision, rationale, and trade-offs accepted in an ADR._ If teams have different expertise levels: this is a programme risk I'd flag early. If we choose a technology that one team knows well but another doesn't, the less experienced team will be slower and may make costly mistakes. I'd factor in ramp-up time, plan for knowledge transfer sessions between teams, and potentially adjust the timeline for the less experienced team's deliverables. On the follow-up introducing technology the organisation has never run in production introduces several programme risks: Learning curve: development will be slower initially. I'd add 30-50% buffer to estimates for the first features built on the new technology. Operational unknowns: we don't know how it behaves under real production load, what the failure modes are, or how to troubleshoot it. I'd require a production readiness spike before the first real feature goes live. Hiring risk: if the only person who understands the technology leaves, we have a single point of failure. I'd ensure at least 2-3 engineers get deep exposure. Mitigation: I'd propose a proof of concept with a non-critical use case before committing the programme to it. Let the team build confidence with real experience before putting it on the critical path."

</details>

<details>
<summary><strong>TPM-T008 · Your programme involves building a new customer-facing service that handles authentication. What security considerations would you ensure are part of the technical planning?</strong></summary>

**Interview prompts**

1. What authentication and authorisation patterns would you expect the team to evaluate, and what questions would you ask to validate their approach?
2. How would you ensure security is addressed throughout the delivery lifecycle rather than as a last-minute review?
3. How do you validate that security requirements are met when you are not a security expert yourself?

**Follow-up**

A security review reveals a critical vulnerability two days before launch. The fix requires a significant redesign of the authentication flow. How do you manage the programme impact?

**What this tests**

Tests whether the TPM understands security enough to ask the right questions and ensure it's not an afterthought. The follow-up tests programme management under crisis.

**Evaluator guide**

Do they mention authentication patterns: OAuth 2.0, OIDC, SAML, token-based auth, MFA? Do they talk about security throughout the SDLC, not just at the end? TPM signal: Do they lean on security experts rather than trying to be one? TPM signal: Do they build security checkpoints into the delivery plan threat modelling, security review, pen testing? TPM signal: Do they manage the follow-up scenario as a programme decision, not a technical one? Red flag: No awareness of authentication standards or common vulnerability patterns. Red flag: Treats security as a gate at the end rather than integrated throughout.

**Example strong answer**

"I'm not a security expert, and I don't pretend to be. But I know enough to ask the right questions and ensure the programme treats security as a first-class concern, not a lastminute audit. Authentication patterns I'd expect the team to evaluate: OAuth 2.0/OIDC for customer-facing auth this is the industry standard. I'd ask: are we using an established identity provider or building our own? Building your own auth is one of the highest-risk things a team can do. Token management: how are access tokens and refresh tokens handled? What are the expiry policies? How are tokens stored on the client side? MFA: is multi-factor authentication required? What factors are supported? For authorisation: what model are we using RBAC, ABAC? How are permissions managed and audited? Questions I'd ask to validate the approach: What happens if a token is stolen? What's the blast radius and how do we revoke access? How do we handle session management across multiple services? What data does the token contain, and is any of it sensitive? How do we log and audit authentication events? Ensuring security throughout delivery: Week 1-2 of planning: threat modelling session with a security specialist. Identify attack vectors before writing code. Design phase: security review of the authentication architecture before implementation starts. During development: SAST/DAST tools running in CI to catch common vulnerabilities automatically. Before release: penetration testing on the authentication flows. I'd schedule this 3-4 weeks before launch, not 3 days before, to allow time for fixes. Post-launch: security monitoring and incident response plan. How I validate security without being an expert: I build relationships with security specialists and involve them early. I create checkpoints in the programme plan for security reviews. I ask the team to explain their approach in plain language if they can't explain why something is secure, that's a red flag. I track security review findings and fix rates the same way I track any other programme risk. For the follow-up a critical vulnerability two days before launch is a programme crisis, not just a technical one. My immediate actions: 1. _Assess the risk: how exploitable is this vulnerability? What's the potential impact if it's_ exploited? Get the security team to classify it. 2. _If it's truly critical and exploitable: delay the launch. No amount of business pressure is_ _worth a security breach that affects customer data. I'd communicate to leadership:_ _'Here's the vulnerability, here's the risk of launching with it, here's the timeline for the_ fix.' 3. _Explore mitigations: can we launch with a temporary workaround - a WAF rule, rate_ limiting, reduced scope - while the proper fix is implemented? 4. _Replan: how long does the fix take? What can we ship now and what waits?_ _Communicate a revised timeline immediately._ 5. _Post-mortem: why was this found two days before launch and not four weeks ago? Fix_ _the process that allowed it."_

</details>

<details>
<summary><strong>TPM-T009 · A critical release is planned for Friday. During final testing, one of five features in the release is found to have a significant bug. Walk me through how you would manage this situation.</strong></summary>

**Interview prompts**

1. What decision process would you follow to determine whether to delay the release, remove the broken feature, or proceed as-is?
2. How would you communicate the situation to stakeholders, and what information would they need to make an informed decision?
3. What mechanisms should have been in place to make isolating and removing a single feature straightforward?

**Follow-up**

The feature with the bug belongs to a team that insists they can fix it by Thursday night. The other four teams are nervous about last-minute changes. How do you navigate this?

**What this tests**

Tests real-time programme decision-making under pressure. The follow-up tests the TPM's ability to manage competing team interests.

**Evaluator guide**

Do they have a clear decision framework: assess severity, evaluate isolation options, make a go/no-go call? Do they consider risk of proceeding vs. risk of delaying? Do they mention feature flags, branch-based isolation, or configuration-based toggles? TPM signal: Do they communicate proactively with clear options and recommendations, not just the problem? TPM signal: Do they consider the impact on all teams, not just the one with the bug? TPM signal: Do they have a process for this type of decision rather than making it ad-hoc? Red flag: Panics and delays the entire release without evaluating options. Red flag: Mentions to manually revert the code with / without mentioning re-testing efforts. Red flag: Ships the bug to meet the date.

**Example strong answer**

"This is a situation where having a structured decision process prevents panic-driven choices. Here's how I'd handle it: Step 1 Assess: What's the severity of the bug? Is it a data corruption risk, a user-facing error, or a cosmetic issue? Can the buggy feature be isolated from the other four? Are they independent, or does the release require all five to work together? What's the business impact of delaying the entire release? Are there contractual deadlines, regulatory dates, or customer commitments? Step 2 Evaluate options: Option A: Remove the broken feature and release the other four. This is my preferred option if the feature can be isolated. Feature flags or configuration toggles make this trivial. Without them, it depends on whether the feature's code changes can be reverted cleanly. Option B: Delay the release by a defined period (not indefinitely) to fix the bug. Only if the fix is low-risk and the timeline is short (hours, not days). Option C: Proceed with the bug if it's low severity and doesn't affect the other features. Document it as a known issue with a fix timeline. Option D: Delay the entire release. Only if the features are tightly coupled and can't be separated, and the bug is severe. Step 3 Decide and communicate: I'd present the options to the release decision group (product owner, engineering leads, operations) with my recommendation, the risks of each option, and the timeline. The decision should take 30 minutes, not 3 hours. Communication to stakeholders would include: what happened, what our options are, what we recommend, and what the revised timeline is. No surprises. What should have been in place: Feature flags: every feature in the release should be independently toggleable. This makes 'remove the broken feature' a configuration change, not a code change. Independent deployability: features should be deployable independently so that one team's issue doesn't block others. Release candidate testing: the final testing should happen early enough in the week to allow for this type of decision, not on Thursday night for a Friday release. For the follow-up: I'd set a hard deadline 'If the fix is merged, tested, and passing all CI checks by 5pm Thursday, it's in the release. If not, the feature is pulled and ships in the next release.' I'd make this decision based on the programme's risk tolerance, not the team's optimism. I'd also ensure the fix goes through the same testing and review process as any other change a rushed, untested fix on Thursday night is higher risk than shipping without the feature. The other four teams' concerns are valid; I'd protect the integrity of their work."

</details>

<details>
<summary><strong>TPM-T010 · An urgent production fix needs to be applied to multiple release branches. Walk me through how you would coordinate this across teams.</strong></summary>

**Interview prompts**

1. How would you ensure the fix is applied consistently across all affected branches without introducing regressions?
2. What communication and coordination steps would you take to align the teams involved?
3. How would you prevent this type of multi-branch hotfix situation from recurring?

**Follow-up**

During the hotfix rollout, a team discovers that applying the fix to their branch conflicts with a feature they're mid-way through developing. How do you resolve this?

**What this tests**

Tests coordination under time pressure. Multi-branch hotfixes are a common source of programme-level chaos this reveals whether the TPM has dealt with it before.

**Evaluator guide**

Do they describe a coordination process: single owner for the fix, consistent cherry-pick or merge strategy, verification on each branch? Do they mention testing the fix on each branch independently before deploying? Do they talk about communication channels: dedicated Slack channel, war room, status updates at defined intervals? TPM signal: Do they think about who coordinates, not just what happens technically? TPM signal: Do they consider preventing recurrence, not just fighting the fire? TPM signal: Do they manage the conflict in the follow-up by balancing urgency with team impact? Red flag: Assumes the fix can be applied identically to all branches without testing. Red flag: No communication plan just tells everyone to "fix it."

**Example strong answer**

"Multi-branch hotfixes are high-risk because the same logical fix may need different implementations on different branches, and each one can introduce new issues. Here's how I'd coordinate: Immediate coordination: 1. _Designate a single incident owner - typically the senior engineer who wrote or best_ _understands the fix. They're responsible for the technical consistency of the fix across_ branches. 2. _Identify all affected branches and environments. Map them out: which release_ branches are in production, which are in staging, which are in active development? 3. _Establish a dedicated communication channel (Slack channel, Teams bridge) for this_ hotfix. All updates go there. This prevents fragmented information across multiple _threads._ Applying the fix: The fix is developed and tested against one branch first typically the production branch, since that's the highest priority. Once validated, it's cherry-picked (not merged) to each additional branch. Cherrypicking is cleaner because it applies just the fix, not any unrelated changes. Each branch gets its own test cycle. The fix must be tested on each branch independently because different branches may have different code states. Each team that owns a branch is responsible for verifying the fix works in their context. The incident owner provides guidance, but each team knows their branch best. Communication cadence: Initial message: what the issue is, which branches are affected, who is coordinating, expected timeline. Updates every 2 hours (or more frequently for severe incidents): which branches have the fix, which are pending, any issues encountered. Final all-clear: fix confirmed on all branches, monitoring confirms no regressions. Preventing recurrence: This type of situation usually indicates too many long-lived branches. I'd push for trunk-based development with feature flags to reduce the number of active branches. Implement automated backporting: when a fix is merged to main, automated tooling creates PRs for active release branches. Review the branching strategy in the next retrospective: do we need this many active branches? Can we consolidate? For the conflict follow-up: the production fix takes priority production stability always wins over in-progress features. I'd work with the team to: first, apply the hotfix on a clean branch to verify it works, then help the team resolve the merge conflict with their feature branch. If the conflict is significant, the team may need to rebase their feature branch on top of the hotfix. I'd adjust their sprint commitment if this costs them time, and communicate the impact to stakeholders proactively." ### Part 2: Programme Management

</details>
