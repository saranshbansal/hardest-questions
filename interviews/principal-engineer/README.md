# Principal Engineer Interview Prep

This collection contains principal-level questions from Atlassian, NVIDIA, and Booking.com interview loops. The questions are organized into canonical high-level concepts, so related questions live together and future contributions have a clear home. Each question appears once; the original company loop is retained as source context inside the concept file. Each question is followed by **model answer guidance**: the important reasoning path, trade-offs, risks, and leadership signals an excellent answer should surface.

## Quick navigation

| Interview loop | Focus areas |
| --- | --- |
| [Coding and data structures](concepts/coding-and-data-structures.md) | Algorithms, data structures, coding fundamentals, and performance |
| [System design and architecture](concepts/system-design-and-architecture.md) | Platforms, APIs, architecture decisions, SaaS, and fraud systems |
| [Distributed systems and reliability](concepts/distributed-systems-and-reliability.md) | Networking, consistency, failure modes, observability, and scale |
| [Data and storage](concepts/data-and-storage.md) | SQL, schemas, indexing, partitioning, migrations, and analytics |
| [Front-end and platform engineering](concepts/frontend-and-platform-engineering.md) | JavaScript, rendering, design systems, performance, and technical debt |
| [GPU, parallelism, and hardware](concepts/gpu-parallelism-and-hardware.md) | CUDA, GPU systems, memory, hardware/software boundaries, and digital logic |
| [Technical leadership and behavioral](concepts/technical-leadership-and-behavioral.md) | Influence, mentoring, judgment, communication, and principal-level scope |

Use the concept files above as the primary navigation. Each file keeps source labels (Atlassian, NVIDIA, or Booking.com) so readers can still understand the context without duplicating questions across company files.

## How to practice

1. Pick a round and answer the question aloud before opening the guidance.
2. Structure the response around requirements, scale, consistency, failure modes, observability, and rollout.
3. Compare your answer with the model guidance. Treat it as a checklist of ideas, not a script.
4. Explain at least one trade-off and name the condition that would make you choose the alternative.
5. End with clarifying questions, measurable success criteria, or a staged rollout plan.

## What principal-level answers demonstrate

- **Systems thinking:** Connect APIs, data, infrastructure, operations, security, and user impact.
- **Trade-off judgment:** Make constraints explicit instead of presenting one design as universally correct.
- **Operational ownership:** Cover SLOs, capacity, observability, incident behavior, migration, and rollback.
- **Organizational influence:** Show how you align teams, create paved roads, and earn adoption without relying on authority.
- **Pragmatism:** Prefer reversible, incremental changes and validate assumptions with production-shaped evidence.

## Answer framework

Use this lightweight framework when a prompt is unfamiliar:

1. **Clarify:** Identify users, workload, scale, latency, consistency, compliance, and cost constraints.
2. **Sketch:** State the simplest architecture or algorithm that satisfies the core requirement.
3. **Stress:** Walk through hot paths, failure modes, abuse cases, and uneven load.
4. **Choose:** Compare alternatives and explain the trade-off behind the decision.
5. **Operate:** Describe metrics, alerting, capacity planning, rollout, migration, and rollback.
6. **Lead:** Explain ownership boundaries, adoption strategy, and how you would resolve disagreement.

Model answers in this collection are intentionally concise. Expand them with a real project, a concrete metric, and an example of a decision you changed after learning something new.
