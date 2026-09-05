# TPM · Programme Management

Delivery ownership, prioritisation, stakeholders, risk, recovery, and execution.

> **Role track:** Technical Program Manager (TPM) — these questions assess programme leadership, technical credibility, and cross-team execution. They are intentionally separate from the engineering interview question bank.

<details>
<summary><strong>TPM-P011 · Tell me about a programme you were responsible for that was significantly at risk behind schedule, over budget, or failing to deliver on its objectives.</strong></summary>

**Interview prompts**

1. What were the early warning signs you identified (or missed), and what recovery actions did you take?
2. How did you communicate the situation to senior stakeholders, and how did you rebuild their confidence?
3. What structural or process changes did you make to prevent similar situations in the future?

**Follow-up**

Looking back, what would you have done differently in the first two weeks of that programme to catch the risk earlier?

**What this tests**

Behavioural question that reveals real experience. The follow-up tests self-awareness and learning ability.

**Evaluator guide**

Do they give a specific, real example with concrete details? Do they describe structured recovery actions, not just "worked harder"? Do they show honest communication with stakeholders, including bad news? TPM signal: Do they take ownership rather than blaming teams or circumstances? TPM signal: Do they describe systemic fixes, not just heroics? TPM signal: Do they show vulnerability and learning in the follow-up? Red flag: Vague or hypothetical answer with no real example. Red flag: Recovery plan was "I stayed late and did it myself." Red flag: Blames everyone else for the situation.

**Example strong answer**

"I led a programme to migrate our payment processing to a new provider across three regions. Six weeks in, we were already three weeks behind schedule. The original estimate was 16 weeks, and at the current pace, we were tracking to 24. Early warning signs I missed: The team estimated based on the first region's migration but didn't account for regulatory differences in region 2 and 3 that added significant complexity. A dependency on the new provider's sandbox environment was on the risk register, but I treated it as low probability. The sandbox was delivered two weeks late and with bugs, which blocked our testing entirely. I was tracking task completion but not integration testing progress. Teams were completing their individual work but nothing was working end-to-end. Recovery actions: 1. _Re-baseline: I stopped pretending the original plan was achievable. I worked with the_ engineering leads to create a realistic revised plan based on actual velocity, not hoped-for velocity. 2. _Scope negotiation: I identified which regions and which payment methods were_ highest priority and proposed a phased approach - region 1 with core payment methods first, then expand. This gave us a meaningful first delivery in 12 weeks instead of a full delivery in 24. 3. _Dependency management: I escalated the sandbox issue directly to the provider's_ account manager and established a weekly sync. I also built a mock environment as a fallback so testing could continue even when the sandbox was down. 4. _Increased integration cadence: I moved from bi-weekly to daily integration testing,_ _which surfaced issues much faster._ Communicating to stakeholders: I requested a 30-minute meeting with the steering committee. I presented: here's where we are (honest status, no spin), here's why we're behind (my analysis, not excuses), here are the options (phased delivery with earlier value vs. delayed full delivery), and here's my recommendation. I didn't wait to be asked I brought the problem and the solution together. The steering committee appreciated the transparency and approved the phased approach. Rebuilding confidence: I moved to weekly stakeholder updates with a simple dashboard showing planned vs. actual progress. When we hit the revised milestones on time, confidence rebuilt naturally. Actions, not words, restored trust. Structural changes for prevention: I now require a dependency validation exercise in the first two weeks of any programme. Every external dependency gets a 'proof of life' test. I track integration test pass rates as a programme metric from week 1, not just individual team progress. I build risk-adjusted timelines adding explicit buffer for identified risks rather than hoping they won't materialise. For the follow-up in the first two weeks, I should have insisted on a proof-of-concept that exercised the end-to-end flow in at least one region, including the provider's sandbox. That would have surfaced both the sandbox reliability issue and the regulatory complexity immediately, when we had time to adjust."

</details>

<details>
<summary><strong>TPM-P012 · Tell me about a time you had to drive delivery forward when the requirements were incomplete, ambiguous, or constantly changing.</strong></summary>

**Interview prompts**

1. How did you create enough structure to keep teams moving without waiting for perfect clarity?
2. What techniques did you use to manage stakeholder expectations while requirements were still evolving?
3. How did you decide what to commit to and what to keep flexible?

**Follow-up**

A key stakeholder adds a major requirement mid-programme and insists it's nonnegotiable. How do you assess the impact and respond?

**What this tests**

Tests the TPM's core skill of navigating ambiguity one of the most valuable TPM traits. The follow-up tests scope management under stakeholder pressure.

**Evaluator guide**

Do they describe specific techniques: timeboxed discovery, decision logs, assumption tracking, progressive elaboration? Do they show a structured approach to managing unknowns rather than just "winging it"? Do they manage expectations by being transparent about what's known vs. unknown? TPM signal: Do they separate what's fixed from what's flexible protecting the critical path while allowing flexibility elsewhere? TPM signal: Do they use assumptions as a tool documenting assumptions explicitly so they can be validated or invalidated? TPM signal: Do they manage the follow-up with impact analysis, not just a yes or no? Red flag: Waited for complete requirements before starting any work. Red flag: Just built whatever they thought was right without stakeholder alignment. Red flag: Demonstrates rigidity in approach, using processes as a shield rather than being flexible with proper documentation of risks & assumptions

**Example strong answer**

"I was leading a programme to build a customer self-service portal for a financial services client. The product vision was clear 'customers should be able to manage their accounts online' but the detailed requirements were evolving weekly as the client's compliance team, UX team, and business stakeholders all had input. How I created structure: I divided the requirements into three tiers: Tier 1 (clear and agreed), Tier 2 (directionally agreed but details evolving), and Tier 3 (unknown or contested). Teams worked on Tier 1 immediately, prepared designs for Tier 2, and I ran discovery sessions for Tier 3. Assumption log: for every area where requirements were unclear, I documented our working assumption. 'We assume self-service password reset is in scope. If this changes, it impacts the auth service by 2 weeks.' This made uncertainty visible and forced stakeholders to validate or challenge assumptions early. Two-week decision cadence: every two weeks, I brought the outstanding requirement questions to a stakeholder decision meeting. Items couldn't stay in limbo forever they got decided, deferred, or explicitly acknowledged as unknown with a fallback plan. Managing stakeholder expectations: I was transparent: 'Here's what we're confident about, here's what's still being defined, and here's what we don't know yet. Our plan reflects this committed items on the roadmap, tentative items flagged, and unknown items in a backlog that we'll plan as clarity emerges.' I resisted the pressure to commit to a full scope and date simultaneously. When pressed, I'd say: 'I can commit to this scope by this date, or that scope by that date. I can't commit to everything by the earliest date because requirements are still evolving.' Deciding what to commit to vs. keep flexible: I committed to the core user journeys that were stable and clear account viewing, balance checks, transaction history. These were on the critical path and had stable requirements. I kept flexible the features that were still being designed notification preferences, reporting dashboards, account customisation. These were built on modular architecture so they could adapt as requirements solidified. The rule was: commit to things that are on the critical path and well-understood. Keep flexible everything else. For the follow-up when a stakeholder adds a major requirement mid-programme, I don't say yes or no immediately. I say: 'Let me assess the impact.' Then I'd work with engineering leads to understand: how much effort does this require, what does it displace, what's the risk to the timeline? I'd come back to the stakeholder with: 'This requirement adds 4 weeks to the programme. We can accommodate it if we defer features X and Y, or we can extend the timeline by 3 weeks. Which trade-off do you prefer?' The stakeholder gets their requirement, but they also see the cost. Often, when the cost is visible, the 'non-negotiable' becomes negotiable."

</details>

<details>
<summary><strong>TPM-P013 · Imagine you manage a programme serving multiple customers. Your largest customer representing 80% of revenue reports a critical issue at the same time a smaller but strategic customer escalates an unrelated incident.</strong></summary>

**Interview prompts**

1. How do you prioritise between these two situations, and what factors drive your decision?
2. How do you communicate your prioritisation to the customer who isn't getting immediate attention?
3. What programme structures would you put in place to handle competing customer priorities without ad-hoc decision-making every time?

**Follow-up**

The smaller customer threatens to leave if their issue isn't resolved within 24 hours. Your leadership team is divided on what to do. How do you facilitate a decision?

**What this tests**

Tests pragmatic decision-making under pressure and communication skills. There's no perfect right answer what matters is the framework.

**Evaluator guide**

Do they consider multiple factors: revenue impact, contractual SLAs, strategic importance, severity, resolution complexity? Do they avoid a purely revenue-based decision without considering strategic value? Do they show empathy and transparency in communicating with both customers? TPM signal: Do they propose a prioritisation framework that works beyond this one situation? TPM signal: Do they consider whether both issues can be worked in parallel with different resources? TPM signal: Do they manage the follow-up by facilitating a decision with data, not just escalating the problem? Red flag: Purely revenue-based decision with no nuance. Red flag: Avoids making a prioritisation call. Red flag: Jumps on an answer without providing the background and assumptions.

**Example strong answer**

_"First, I'd check whether this is actually an either-or choice. Can we assign separate resources to each issue? If I have two engineers with the right expertise, both can be worked in parallel. The worst thing a TPM can do is create a false constraint._ _If resources are genuinely limited, I'd evaluate based on:_ Severity and blast radius: how many users are affected? Is data at risk? Is the customer's business blocked? Contractual obligations: do we have SLAs with either customer that define response and resolution times? Breaking an SLA has contractual consequences. Revenue and strategic value: the 80% revenue customer has obvious financial weight, but the smaller customer may be strategic a reference account, a new market entry, or a pilot for a major expansion. Resolution complexity: if one issue is a quick fix and the other is a multi-day investigation, I'd do the quick fix first regardless of customer size. _In this specific scenario, assuming both issues are equally severe and complex, I'd prioritise the 80% revenue customer first while immediately communicating to the smaller_ customer. But I'd also set a hard commitment for when we'll start on their issue not 'we'll get to it eventually' but 'we're starting on your issue at 2pm today.' _Communication to the customer not getting immediate attention:_ Be honest: 'We're dealing with a critical issue for another customer that's currently consuming our incident response team. Your issue is our next priority and we'll begin working on it at [time].' Provide a specific person they can reach out to for updates. Give a meaningful interim update even if there's no resolution yet. _Programme structures for the future:_ Tiered support model: define customer tiers based on revenue, strategic value, and SLA. Each tier has defined response times and resource allocation. On-call rotation with enough depth: if one incident consumes one engineer, there's another available for a second incident. Incident prioritisation matrix: documented criteria for how we prioritise competing issues, agreed upon with leadership in advance, not during a crisis. Escalation path: clear criteria for when an incident gets escalated to leadership for a prioritisation call. _For the follow-up: the smaller customer threatening to leave changes the calculus. I'd_ facilitate a decision meeting with leadership, presenting: here's the revenue at risk from each customer, here's the cost of losing the smaller customer (not just their revenue but _the relationship, the market signal, the reference value), here are our options (split resources, bring in additional help, negotiate timeline with the larger customer). I'd present a recommendation, but this is a business decision that leadership needs to own."_

</details>

<details>
<summary><strong>TPM-P014 · Tell me about a time when senior leadership disagreed with a recommendation you made about a programme direction, timeline, or resource allocation.</strong></summary>

**Interview prompts**

1. How did you present your recommendation, and what evidence did you use to support it?
2. How did you handle the disagreement - did you push back, adapt, or find a middle ground?
3. What did you learn about influencing senior stakeholders from that experience?

**Follow-up**

If you could go back to that moment, would you change your approach? What would you do differently to be more persuasive?

**What this tests**

Tests stakeholder management maturity and self-awareness. The best TPMs know when to push back, when to adapt, and when to disagree and commit.

**Evaluator guide**

Do they give a specific, real example? Do they use data and evidence, not just opinion? Do they show adaptability willing to adjust their approach based on feedback? TPM signal: Do they understand the difference between "I disagree" and "I have data that suggests a different approach"? TPM signal: Do they show learning and self-reflection? Red flag: Caved immediately without presenting their case. Red flag: Refused to adapt and created conflict. Red flag: No real example only hypothetical.

**Example strong answer**

"I recommended delaying a programme launch by four weeks to address performance issues that were surfacing in load testing. Our response times under expected load were 3x the target, and I was confident we'd see customer complaints and potential SLA breaches within the first week of launch. I presented the data clearly: load test results showing response time degradation under projected traffic, projected customer impact based on our SLA commitments, and the estimated cost of post-launch firefighting (engineer hours, customer escalation handling, potential SLA penalties). I also presented the fix timeline our engineers had identified the bottleneck and estimated four weeks to resolve it properly. Leadership disagreed. They had committed the launch date to external stakeholders and a major industry conference was planned around it. Their position was: 'Launch on time, fix it fast after launch.' I pushed back once more with data: 'Based on our load tests, we'll breach SLAs for the top 3 customers within the first 48 hours. The cost of that exceeds the cost of a four-week delay.' But I also listened to their reasoning. The external commitment was real and had business consequences I hadn't fully weighed partnership agreements, press coverage, investor expectations. We found a middle ground: launch on the original date but with a reduced user base. We'd onboard customers in cohorts over four weeks, starting with 10% of the target load. This gave us a real launch for the conference while keeping load within our performance envelope. Meanwhile, the engineering team worked on the performance fix. By the time we'd onboarded all customers, the fix was in place. What I learned: Present your recommendation with data, but also understand the constraints that leadership is operating under. They had information I didn't have the external commitments and their implications. When you disagree, propose alternatives rather than just objecting. 'No, we should delay' is less useful than 'Here are three options that address both the performance risk and the launch commitment.' Sometimes the answer isn't your plan or their plan it's a third option that addresses both concerns. Looking back, I'd have involved leadership earlier. By the time I brought the performance issue to them, the launch date was two weeks away and options were limited. If I'd flagged it a month earlier, we'd have had more flexibility to adjust."

</details>

<details>
<summary><strong>TPM-P015 · Tell me about a time you took ownership of something that was clearly outside your defined role or responsibility.</strong></summary>

**Interview prompts**

1. What prompted you to step in, and what was the situation?
2. How did you navigate the politics of owning something outside your scope without stepping on toes?
3. What was the outcome, and how did it change how your role was perceived?

**Follow-up**

Did taking on that additional ownership create any tension with the person or team who was originally responsible? How did you handle it?

**What this tests**

Tests initiative and political awareness. The best TPMs expand their impact by owning gaps, but the best of the best do it without creating territorial conflict.

**Evaluator guide**

Do they give a specific example that shows genuine initiative, not just doing extra work they were assigned? Do they show awareness of organisational dynamics and how their actions affected others? Do they describe a positive outcome that benefited the programme, not just their reputation? TPM signal: Do they identify gaps in ownership proactively rather than waiting for permission? TPM signal: Do they bring others along rather than going solo? TPM signal: Do they handle the follow-up tension with empathy and collaboration? TPM signal: Do they exert influence without authority? Red flag: Example is just "I worked extra hours" rather than taking ownership of a gap. Red flag: Took over someone else's work without awareness of the political impact.

**Example strong answer**

"Our programme had a critical gap in data migration planning. We were building a new platform, but nobody owned the migration of existing customer data from the legacy system. Engineering assumed product would define the migration requirements. Product assumed engineering would figure it out. And neither team had started. We were 8 weeks from launch. I stepped in because this was a programme-level risk that didn't have an owner, and waiting for someone to claim it would have cost us weeks. As a TPM, I'm responsible for the programme delivering end-to-end, and a launch without customer data isn't a launch. How I navigated it: I didn't claim ownership publicly. Instead, I scheduled a meeting with both the engineering lead and the product owner and said: 'I've identified a gap in our plan around data migration. Can we spend 30 minutes mapping out what needs to happen and who should own each piece?' I facilitated the session, captured the work breakdown, and offered to coordinate the cross-team aspects since I was already tracking programme dependencies. Engineering owned the technical migration scripts, product owned the data mapping and validation rules, and I owned the programme plan sequencing, testing, rollback, and stakeholder communication. I framed my involvement as 'connecting the dots across teams,' not 'taking over because no one else was doing it.' This was important because both teams would have felt called out if I'd made it about their failure to plan. Outcome: the data migration was planned, tested, and executed on time. We identified data quality issues during testing that would have caused customer-facing errors on launch day. The programme launched with clean data because we caught the issues early. How it changed perception: the engineering and product leads both acknowledged that the migration would have been a disaster without someone pulling it together. My director started involving me earlier in programme formation because I'd demonstrated that I could identify and fill ownership gaps without being asked. For the follow-up there was initial discomfort from the engineering lead, who felt I was implying his team had dropped the ball. I addressed it directly in a 1:1: 'This wasn't about blame. Data migration sits between engineering and product, and neither team's scope explicitly included it. My job is to make sure the programme has no gaps. Your team's technical execution on the migration was excellent once we had a plan.' He appreciated the directness, and it actually strengthened our working relationship."

</details>

<details>
<summary><strong>TPM-P016 · Tell me about a process or initiative you championed that ultimately failed or didn't deliver the results you expected.</strong></summary>

**Interview prompts**

1. What was the process, and what outcome were you hoping to achieve?
2. At what point did you realise it wasn't working, and what signals told you?
3. What did you learn, and how did that change your approach to introducing new processes?

**Follow-up**

How do you now decide when to persevere with a struggling process versus when to abandon it? What's your threshold?

**What this tests**

Tests self-awareness, humility, and learning ability. TPMs who can't admit failure are dangerous because they'll cling to broken processes.

**Evaluator guide**

Do they give an honest, specific example of a genuine failure? Do they take personal responsibility rather than blaming resistance or lack of support? Do they describe concrete lessons learned and how they changed their approach? TPM signal: Do they show awareness that process must serve the teams, not the other way around? TPM signal: Do they have a framework for evaluating process effectiveness? TPM signal: Do they describe checking for signals of failure proactively rather than waiting until it's obvious? TPM signal: Did they invest in upfront homework—such as planning, stakeholder discussions, and piloting use cases—before rolling out the process at full scale? Red flag: Can't think of a process that failed suggests either lack of experience or lack of self-awareness. Red flag: Starts explaining a process that worked fine. Red flag: Blames the teams for not following the process.

**Example strong answer**

"I introduced a programme-level change control board (CCB) to manage scope changes across five teams. The idea was that any scope change above a certain size threshold would go through a weekly CCB meeting for impact assessment and approval. I was trying to solve a real problem uncontrolled scope creep was causing missed deadlines. The process failed. After six weeks, teams were routing around the CCB by splitting large changes into small ones that fell below the threshold. The CCB meetings became a formality pre-decided outcomes, rubber-stamped approvals. The actual scope discussions were happening in hallway conversations. Scope creep continued. Signals that told me it wasn't working: Attendance dropped. Senior engineers stopped coming to CCB meetings and sent proxies. The number of 'small' scope changes (below threshold) tripled, while 'large' changes (requiring CCB) dropped to near zero. Teams were gaming the system. Delivery dates were still slipping, which was the original problem I was trying to solve. I realised the root cause: I'd introduced a governance process that felt like bureaucracy because it didn't align with how teams actually made decisions. The CCB created a bottleneck without adding value because the approval was a formality, not a genuine decision point. What I learned: Process must serve the people, not the other way around. If teams are routing around your process, the process is wrong, not the teams. Start with the lightest-weight solution that addresses the problem. A full CCB was overkill for what was fundamentally a visibility problem. Co-design the process with the teams who will use it. I designed the CCB top-down and rolled it out. I should have said: 'We have a scope creep problem how should we manage it?' What I replaced it with: a simple scope change log visible to all teams and stakeholders. Any scope change gets logged with: what changed, who requested it, estimated impact on timeline, and what was deprioritised to make room. No approval gate, just transparency. This turned out to be far more effective because stakeholders could see the cumulative impact of their changes in real time, and that visibility naturally reduced unnecessary changes. For the follow-up my threshold now is: if after 4-6 weeks the process isn't showing early signs of the intended behaviour change, I reassess. I look for leading indicators, not lagging ones. If teams are engaged and following the process but results haven't materialised yet, I persevere. If teams are disengaged, working around the process, or complaining that it's not helping, I pivot. The fastest way to kill a team's trust in your judgement is to force a process that everyone can see isn't working."

</details>

<details>
<summary><strong>TPM-P017 · Tell me about a time when critical unplanned work landed in the middle of a sprint something that couldn't wait but also couldn't be absorbed without impact.</strong></summary>

**Interview prompts**

1. How did you assess the urgency and decide whether to pull it into the current sprint?
2. What did you deprioritise, and how did you communicate the trade-off to stakeholders who were expecting the original deliverables?
3. What changes did you make to your planning process to account for unplanned work in the future?

**Follow-up**

The stakeholder whose feature was deprioritised escalates to your leadership. How do you handle that conversation?

**What this tests**

Tests real-world sprint management and stakeholder communication. Every TPM deals with this regularly the question reveals whether they have a mature approach or just react.

**Evaluator guide**

Do they have criteria for assessing urgency: business impact, customer impact, SLA risk, time sensitivity? Do they communicate trade-offs proactively rather than just silently dropping planned work? Do they build buffers or unplanned capacity into future sprints? Do they mention finding the root cause once the situation is handled? TPM signal: Do they make the trade-off decision explicit and transparent, not hidden? TPM signal: Do they have a process for sprint disruption rather than handling it ad-hoc every time? TPM signal: Do they handle the escalation calmly with data rather than defensively? Red flag: Accepts all unplanned work without pushing back on urgency. Red flag: Never communicates the impact to stakeholders.

**Example strong answer**

"This happens regularly, and having a structured approach is what separates reactive firefighting from programme management. A specific example: mid-sprint, our operations team identified that a regulatory reporting feature was producing incorrect data for a subset of customers. The regulator had a reporting deadline in 10 days. If we missed it, the customer faced fines. This was genuinely urgent and couldn't wait for the next sprint. How I assessed urgency: Is this time-sensitive? Yes regulatory deadline in 10 days. What's the impact of not acting now? Customer faces regulatory fines, potential contract breach, reputational damage to us. Can it be solved with a quick fix, or does it require significant effort? Engineering assessed it as 5-7 days of work for 2 engineers. Can it be handled by someone outside the sprint team? No the engineers with domain knowledge were in the current sprint. Decision: pull it into the sprint. But I was explicit about the trade-off. I met with the product owner and said: 'We need to pull 2 engineers off feature work for 7 days. That means either feature A or feature B gets pushed to the next sprint. Here are the options which feature is lower priority this sprint?' We agreed to push feature B. Communication to stakeholders: I immediately reached out to the stakeholder who owned feature B: 'A regulatory emergency requires us to redirect capacity this sprint. Feature B will be delivered in the next sprint instead. Here's the revised timeline. I wanted you to hear this from me directly rather than discover it later.' I was transparent about the reason without oversharing details. The stakeholder wasn't happy, but they appreciated the early communication and understood the priority. Process changes for the future: I introduced a 15-20% unplanned capacity buffer in sprint planning. If the team's velocity is 40 points, we plan 32-34 points of committed work. The buffer absorbs disruptions without requiring deprioritisation. I created a disruption escalation criteria: only items meeting defined urgency criteria (production impact, regulatory risk, SLA breach) can disrupt the sprint. Everything else goes into the backlog for the next sprint. I started tracking sprint disruption frequency as a programme metric. If disruptions exceed 20% of sprint capacity regularly, that's a signal of a deeper problem poor production stability, inadequate on-call coverage, or unrealistic planning. For the escalation follow-up: I'd approach it calmly with data. I'd explain to my leadership: 'Here's the situation that required us to reprioritise. Here's the impact of not acting on the regulatory issue. Here's the revised timeline for the feature. I communicated the change to the stakeholder proactively.' I'd also acknowledge the stakeholder's frustration and offer a specific plan: 'Feature B is the first item in next sprint and I'll provide daily progress updates until it's delivered.' The key is showing that the decision was deliberate, transparent, and made based on programme priorities, not arbitrary."

</details>

<details>
<summary><strong>TPM-P018 · Tell me about a time you missed a significant risk in a programme, and it materialised into a real problem.</strong></summary>

**Interview prompts**

1. What was the risk, and why did you miss it?
2. What were the consequences, and how did you recover?
3. What changes did you make to your risk management approach as a result?

**Follow-up**

How do you now distinguish between risks that are genuinely unlikely and risks you're simply choosing not to see because they're inconvenient?

**What this tests**

Tests risk management maturity and intellectual honesty. TPMs who can't admit to missing risks haven't managed enough programmes.

**Evaluator guide**

Do they give a specific, honest example? Do they explain why they missed it not just what happened? Do they describe concrete changes to their risk management approach? TPM signal: Do they take personal responsibility for the miss rather than blaming information availability? TPM signal: Do they describe systemic improvements, not just "I'll be more careful next time"? TPM signal: Do they have a thoughtful answer to the follow-up about confirmation bias? Red flag: Can't think of a missed risk suggesting either lack of experience or lack of honesty. Red flag: Lesson learned is just "I'll pay more attention" with no process change.

**Example strong answer**

"I was leading a programme to integrate with a new third-party data provider. The integration was technically straightforward well-documented API, sandbox environment, responsive support team. I assessed the technical risk as low and didn't put any buffer in the plan for integration complications. What I missed was the contractual and procurement risk. Our legal team needed to review the data processing agreement because the provider would be handling customer PII. Legal had a 6-week review backlog that I didn't know about. By the time the contract review started, we were already three weeks into development. Legal flagged data residency concerns that required the provider to make changes to their infrastructure, which added another four weeks to the timeline. Why I missed it: I was focused on technical risk and didn't consider the end-to-end dependency chain. I assumed procurement and legal were parallel activities that would be resolved before engineering needed the production API keys. I also didn't ask the right questions early enough I should have checked with legal in week one, not week four. Consequences: the programme was delayed by six weeks. We had engineers idle for two weeks while waiting for the contract to be signed. The engineering idle time cost the programme credibility with leadership and affected team morale. Recovery: I escalated the legal review to both my leadership and the legal team's leadership, explaining the business impact of the delay. I got the review prioritised. Meanwhile, I redirected idle engineers to other programme work items to minimise wasted capacity. I also negotiated a parallel path with the provider where they began their infrastructure changes while the contract was being finalised. Changes to my risk management approach: I now map the entire dependency chain for any external integration: technical, contractual, procurement, security, compliance, and operations. Each gets a timeline and an owner. I do a 'pre-mortem' at programme kick-off: 'Imagine it's six months from now and the programme has failed. What went wrong?' This surfaces risks that people know about but don't raise because they seem unlikely. I check in with non-engineering dependencies (legal, procurement, security, compliance) in the first week, not after development starts. For the follow-up distinguishing genuinely unlikely risks from inconvenient ones: I've learned to ask myself: 'Am I rating this risk as low probability because I have evidence, or because acknowledging it would require changing the plan?' If the answer is the latter, that's a red flag. I also rely on pre-mortems and diverse perspectives when I ask the team 'what could go wrong?', someone almost always names the risk I was subconsciously avoiding."

</details>

<details>
<summary><strong>TPM-P019 · Your programme is behind on a critical feature delivery, but the engineering team is also dealing with a high volume of production bugs that are affecting existing customers.</strong></summary>

**Interview prompts**

1. How do you decide how to split the team's capacity between bug fixes and new feature development?
2. How do you communicate this trade-off to product leadership who want the feature, and to customer success who want the bugs fixed?
3. What data or metrics would you use to make this a structured decision rather than a gut call?

**Follow-up**

Product leadership tells you the feature must ship on time regardless. Customer success tells you churn will increase if bugs aren't fixed. How do you navigate this?

**What this tests**

Tests the TPM's ability to manage competing priorities with data and to communicate trade-offs honestly to multiple stakeholders.

**Evaluator guide**

Do they use data to drive the decision: bug severity, customer impact, churn risk, feature revenue projection? Do they avoid a binary choice and look for creative solutions: dedicated bug team, severitybased triage, parallel workstreams? Do they communicate the trade-off honestly to both stakeholder groups? TPM signal: Do they elevate the decision to leadership when it requires a business call, not just a programme call? TPM signal: Do they present options with trade-offs rather than just asking "what should I do?" TPM signal: Do they use metrics like MTTR, bug escape rate, customer satisfaction scores? Red flag: Ignores bugs entirely to hit the feature deadline. Red flag: Cannot articulate the trade-off in business terms. Red flag: Takes the prioritisation call without involving product / business.

**Example strong answer**

"This is never a binary choice. I'd approach it by first understanding the actual impact of each priority, then finding a split that addresses both. Step 1 Quantify both sides: Bugs: how many customers are affected? What's the severity distribution? Are any bugs causing SLA breaches, revenue loss, or churn risk? What's the customer sentiment? Feature: what's the revenue projection or strategic importance? What's the cost of delay is this tied to a customer commitment, a market window, or a contractual deadline? Step 2 Evaluate capacity options: Can I split the team? Dedicate 2-3 engineers to critical bug fixes while the rest continue feature work. This is often the most pragmatic solution. Severity-based triage: fix P1 bugs (data loss, security, complete outage) immediately. P2 bugs (significant functionality broken) get fixed within the sprint. P3 bugs (degraded experience) go to backlog. Time-box bug fixing: dedicate the first 3 days of the sprint to the most critical bugs, then shift to feature work. This gives customer success a timeline for fixes without abandoning the feature. Step 3 Communicate the trade-off: To product leadership: 'We have 15 open bugs affecting 2,000 customers. Three of these are causing SLA breaches that cost us $X per month in penalties. If we allocate zero capacity to bugs, we hit the feature date but risk losing customer Y who has threatened to churn. Here's my recommended split and the impact on the feature timeline.' To customer success: 'We're dedicating 30% of the team's capacity to the top 5 bugs by customer impact. Here's the fix timeline for each. The remaining bugs are triaged and will be addressed in the next sprint.' Metrics I'd use: Bug escape rate: how many bugs are reaching production per sprint? If it's increasing, there's a quality problem that more bug-fixing won't solve. MTTR by severity: how long does it take to resolve P1, P2, P3 bugs? Customer impact: number of customers affected, support ticket volume, NPS/CSAT trend. Feature delay cost: revenue impact of delaying the feature by 1 week, 2 weeks, etc. For the follow-up: I'd bring both stakeholders together rather than trying to mediate separately. I'd present the data: 'Here's the feature timeline at 100% feature focus. Here's the projected churn cost of ignoring bugs. Here's the recommended allocation and its impact on both. This is a business decision that we need to make together.' If they can't agree, I'd escalate to whoever owns both P&L lines with a clear recommendation. What I wouldn't do is quietly sacrifice one priority to satisfy the other that destroys trust when the consequences become visible."

</details>

<details>
<summary><strong>TPM-P020 · Tell me about a time you worked with a vendor or external partner that was underperforming delivering late, delivering poor quality, or both and you couldn't replace them in the short term.</strong></summary>

**Interview prompts**

1. What steps did you take to improve the vendor's performance?
2. How did you manage the programme risk created by the vendor's underperformance?
3. How did you communicate the situation to your stakeholders without undermining the vendor relationship?

**Follow-up**

The vendor blames your organisation for scope changes and unclear requirements. There's some truth to it. How do you handle this?

**What this tests**

Tests programme management in situations where the TPM doesn't control the resources. Vendor management is a common TPM challenge that reveals coordination and negotiation skills.

**Evaluator guide**

Do they describe structured performance management: clear expectations, regular checkpoints, documented issues? Do they build programme-level mitigations: buffer, parallel paths, internal fallback? Do they balance accountability with relationship preservation? TPM signal: Do they own the relationship and work to improve it rather than just complaining? TPM signal: Do they have a plan B while working to improve plan A? TPM signal: Do they handle the follow-up with honesty acknowledging their own organisation's contribution to the problem? Red flag: Purely adversarial approach with no attempt to improve the relationship. Red flag: Blames the vendor without looking at internal contributions to the problem. Red flag: Starts blame game / finding root cause even before resolving the problem at hand.

**Example strong answer**

"I managed a programme where a vendor was responsible for building a key integration component. They were consistently delivering 2-3 weeks late on every milestone and the quality required significant rework by our team before it was usable. Steps to improve performance: 1. _I reset expectations explicitly. I scheduled a meeting with the vendor's project_ manager and delivery lead and said: 'Here's the current state - 3 of 4 milestones have been late, and each delivery has required N days of rework by our team. We need to _understand why and agree on a path forward.'_ 2. _I introduced tighter checkpoints. Instead of monthly milestone reviews, we moved to_ _weekly demos of work in progress. This surfaced issues earlier - if they were going in_ _the wrong direction, we caught it in week 1, not week 4._ 3. _I clarified acceptance criteria. I worked with our engineering team to create detailed_ acceptance criteria for each deliverable, including test cases. This eliminated ambiguity about what 'done' meant. 4. _I established a shared definition of quality: code review standards, test coverage_ expectations, documentation requirements. These went into the SOW as an addendum. _Programme risk management:_ I added 3-week buffers to every vendor dependency on the programme timeline. I communicated to stakeholders that the vendor-dependent path had higher uncertainty. I identified which components our internal team could build as a fallback if the vendor failed to deliver. I didn't start building them (that would be wasteful), but I had a plan ready. I created a risk register entry specifically for vendor performance and reviewed it weekly with the programme steering committee. _Stakeholder communication:_ I was transparent with stakeholders about the risk without being inflammatory: 'Vendor delivery has been inconsistent. I've put mitigations in place including tighter checkpoints and buffer in the timeline. I'm working with the vendor to improve, and I'll escalate if we don't see improvement in the next two milestones.' I avoided badmouthing the vendor because it's unprofessional and unhelpful. My goal was to fix the problem, not assign blame. _For the follow-up: if the vendor has legitimate points about scope changes and unclear requirements from our side, I'd own that honestly. I'd say: 'You're right our requirements_ changed three times in the first month, and that contributed to the delays. Here's what I'm going to do about it: requirements for the next phase will be signed off by both sides before work begins, and any changes go through a formal change request process with impact assessment.' I'd turn it into a mutual improvement plan rather than a blame game. _Taking accountability for our side of the problem usually motivates the vendor to do the same."_ ### Part 3: Agile & Ways of Working

</details>
