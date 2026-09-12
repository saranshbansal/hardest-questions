<p align="center">
  <img src="banner.jpeg" alt="A glowing idea among hanging light bulbs" width="720">
</p>

<h1 align="center">Hardest Questions</h1>

<p align="center">
  A curated question bank for senior, staff, and principal engineers.
</p>

A collection of **212 interview questions** focused on reasoning, trade-offs, and leadership judgment expected in difficult technical interviews. Questions are organized by reusable concepts within each role track, so each question has one canonical home.

---

## Quick start

Choose your track and jump in:

- **[Engineering Track](#engineering-track)** — 182 questions across 8 concept areas + 16 in-depth solution guides
- **[Technical Program / Project Management Track](#technical--project-management-track)** — 30 questions across 3 concept areas

---

## How to use this bank

1. Choose a concept and answer a question aloud before reading the model guidance.
2. Explain the requirements and constraints before proposing a solution.
3. Make trade-offs explicit: consistency vs. availability, latency vs. cost, speed vs. maintainability.
4. Cover failure modes, observability, rollout, and rollback for design questions.
5. Replace the model guidance with your own experience, metrics, and examples.

The original interview source—Atlassian, NVIDIA, or Booking.com—is retained inside each concept file as context. The guidance is a scaffold for reasoning, not a script to memorize.

---

## Engineering Track

182 questions for senior, staff, and principal engineers. Every concept file uses compact, collapsible Q&A cards. Open a card to reveal the answer guidance, trade-offs, and code examples; the `Q001`-style serial numbers make it easy to reference a specific question.

| | Concept | Questions | Coverage |
|---|---|---:|---|
| ◈ | [Coding and data structures](engineering/coding-and-data-structures.md) | 30 | Algorithms, data structures, coding fundamentals, and performance |
| △ | [Applied system design case studies](engineering/applied-system-design-case-studies.md) | 30 | Product/SaaS platforms, infrastructure and ML systems, fraud/risk, and applied architecture decisions |
| ◇ | [System design fundamentals and patterns](engineering/system-design-fundamentals-and-patterns.md) | 52 | Requirements, reusable platform patterns, data flows, resilience, security, traffic management, and interview judgment |
| ≋ | [Distributed systems and reliability](engineering/distributed-systems-and-reliability.md) | 20 | Networking, consistency, failure modes, observability, and scale |
| ▦ | [Data and storage](engineering/data-and-storage.md) | 10 | SQL, schemas, indexing, partitioning, migrations, and analytics |
| ◌ | [Front-end and platform engineering](engineering/frontend-and-platform-engineering.md) | 10 | JavaScript, rendering, design systems, performance, and technical debt |
| ◉ | [GPU, parallelism, and hardware](engineering/gpu-parallelism-and-hardware.md) | 20 | CUDA, GPU systems, memory, hardware/software boundaries, and digital logic |
| ✦ | [Technical leadership and behavioral](engineering/technical-leadership-and-behavioral.md) | 10 | Influence, mentoring, judgment, communication, and principal-level scope |

### In-depth system design solutions

The [`engineering/in-depth-system-design-solutions/`](engineering/in-depth-system-design-solutions/) directory contains detailed architecture chapters—one Markdown guide per system. Each guide is a senior/staff/principal-level walkthrough rather than a quick question card, useful for deep-dive preparation or reference.

| | Solution guide | Coverage |
|---|---|---|
| 01 | [Design Twitter](engineering/in-depth-system-design-solutions/design-twitter.md) | Social graph, posts, hybrid time, search, and notifications |
| 02 | [Design Aarogya Setu](engineering/in-depth-system-design-solutions/design-aarogya-setu.md) | Consent, privacy-preserving architecture, and scale-to-millions |
| 03 | [Design a load balancer](engineering/in-depth-system-design-solutions/design-load-balancer.md) | L4/L7 forwarding, sticky sessions, and health checks |
| 04 | [Design a URL shortener](engineering/in-depth-system-design-solutions/design-url-shortener.md) | Short-code generation, collision handling, and analytics |
| 05 | [Design a logging system](engineering/in-depth-system-design-solutions/design-logging-system.md) | Durable ingestion, streaming, storage, and query interfaces |
| 06 | [Design Google Play Store](engineering/in-depth-system-design-solutions/design-google-play-store.md) | App publishing, delivery, versioning, and rollout strategies |
| 07 | [Design Zoom car app LLD](engineering/in-depth-system-design-solutions/design-zoomcar-app-lld.md) | Domain objects, state machines, and rental workflows |
| 08 | [Design an e-commerce platform](engineering/in-depth-system-design-solutions/design-e-commerce-platform.md) | Marketplace, payments, inventory, and fulfillment |
| 09 | [Design a recommendation system](engineering/in-depth-system-design-solutions/design-recommendation-system.md) | Offline models, online serving, feedback loops, and diversity |
| 10 | [Design an order management system](engineering/in-depth-system-design-solutions/design-order-management-system.md) | Order lifecycle, state transitions, and vendor integration |
| 11 | [Design an app for waste management](engineering/in-depth-system-design-solutions/design-waste-management-app.md) | Scheduling, routing, and resource optimization |
| 12 | [Design library management system LLD](engineering/in-depth-system-design-solutions/design-library-management-system-lld.md) | Inventory, checkouts, and reservations |
| 13 | [Design a warehouse management system](engineering/in-depth-system-design-solutions/design-warehouse-management-system.md) | Storage, picking, packing, and shipment tracking |
| 14 | [Design a parking-lot reservation system](engineering/in-depth-system-design-solutions/design-parking-lot-reservation-system.md) | Availability, pricing, and real-time updates |
| 15 | [Design a real-time inventory tracking system](engineering/in-depth-system-design-solutions/design-real-time-inventory-tracking-system.md) | Event streams, analytics, and alerting |
| 16 | [Design a rating system for an e-commerce website](engineering/in-depth-system-design-solutions/design-ecommerce-rating-system.md) | Consensus, fraud detection, and display logic |

---

## Technical Program / Project Management Track

30 questions for technical program managers and project leads. Organized by three core competencies, separate from engineering questions because they emphasize delivery, stakeholder judgment, and programme scope.

| Concept | Questions | Coverage |
|---|---:|---|
| [Technical credibility](technical-program-manager/technical-credibility.md) | 10 | APIs, releases, architecture awareness, data, security, and production readiness |
| [Programme management](technical-program-manager/programme-management.md) | 10 | Recovery, ambiguity, prioritisation, stakeholders, ownership, and vendors |
| [Agile and ways of working](technical-program-manager/agile-and-ways-of-working.md) | 10 | Programme setup, ceremonies, speed vs. quality, estimation, scaling, and influence |
