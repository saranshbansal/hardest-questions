<p align="center">
  <img src="banner.jpeg" alt="A glowing idea among hanging light bulbs" width="720">
</p>

<h1 align="center">Hardest Questions</h1>

<p align="center">
  A curated question bank for senior, staff, and principal engineers.
</p>

The collection focuses on the reasoning, trade-offs, and leadership judgment expected in difficult technical interviews.

## What's inside

The repository currently contains **203 questions** across two distinct role tracks:

- **Engineering:** architecture, systems, data, front-end, hardware, and technical leadership.
- **Technical Program / Project Management:** technical credibility, programme delivery, stakeholder leadership, risk, and agile ways of working.

Alongside those question cards, the Engineering Track now includes **16 in-depth solution guides**. These are detailed architecture chapters rather than additional interview questions, so the **203 interview-question total remains unchanged**.

Questions are organized by reusable concepts within each role track, so each question has one canonical home and future contributions have a clear place. Open a card in any file to reveal the answer guidance and evaluator notes.

The system-design set is screened against the existing system-design, distributed-systems, and data-and-storage cards. Overlapping prompts are folded into canonical cards: cache hit ratio into `SD-F015`, data-centre failure into `SD-F026`, and live schema changes into `SD-F032`. The five new prompts are `SD-F039`–`SD-F043`; no duplicate cards were added.

## Choose a role track

### Engineering Track

The engineering collection contains 173 questions for senior, staff, and principal engineers.

Every concept file uses compact, collapsible Q&A cards. Open a card to reveal the answer guidance, trade-offs, and code examples; the `Q001`-style serial numbers make it easy to reference a specific prompt during study or discussion.

<table>
  <thead>
    <tr>
      <th align="center"> </th>
      <th align="left">Concept</th>
      <th align="center">Questions</th>
      <th align="left">Coverage</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td align="center"><span style="font-size: 1.6em;">◈</span></td>
      <td><a href="engineering/coding-and-data-structures.md">Coding and data structures</a></td>
      <td align="center">30</td>
      <td>Algorithms, data structures, coding fundamentals, and performance</td>
    </tr>
    <tr>
      <td align="center"><span style="font-size: 1.6em;">△</span></td>
      <td><a href="engineering/system-design-and-architecture.md">System design and architecture</a></td>
      <td align="center">30</td>
      <td>Platforms, APIs, architecture decisions, SaaS, and fraud systems</td>
    </tr>
    <tr>
      <td align="center"><span style="font-size: 1.6em;">◇</span></td>
      <td><a href="engineering/system-design-fundamentals.md">System design fundamentals</a></td>
      <td align="center">43</td>
      <td>Requirements, platform patterns, data flows, resilience, security, and product-scale designs</td>
    </tr>
    <tr>
      <td align="center"><span style="font-size: 1.6em;">≋</span></td>
      <td><a href="engineering/distributed-systems-and-reliability.md">Distributed systems and reliability</a></td>
      <td align="center">20</td>
      <td>Networking, consistency, failure modes, observability, and scale</td>
    </tr>
    <tr>
      <td align="center"><span style="font-size: 1.6em;">▦</span></td>
      <td><a href="engineering/data-and-storage.md">Data and storage</a></td>
      <td align="center">10</td>
      <td>SQL, schemas, indexing, partitioning, migrations, and analytics</td>
    </tr>
    <tr>
      <td align="center"><span style="font-size: 1.6em;">◌</span></td>
      <td><a href="engineering/frontend-and-platform-engineering.md">Front-end and platform engineering</a></td>
      <td align="center">10</td>
      <td>JavaScript, rendering, design systems, performance, and technical debt</td>
    </tr>
    <tr>
      <td align="center"><span style="font-size: 1.6em;">◉</span></td>
      <td><a href="engineering/gpu-parallelism-and-hardware.md">GPU, parallelism, and hardware</a></td>
      <td align="center">20</td>
      <td>CUDA, GPU systems, memory, hardware/software boundaries, and digital logic</td>
    </tr>
    <tr>
      <td align="center"><span style="font-size: 1.6em;">✦</span></td>
      <td><a href="engineering/technical-leadership-and-behavioral.md">Technical leadership and behavioral</a></td>
      <td align="center">10</td>
      <td>Influence, mentoring, judgment, communication, and principal-level scope</td>
    </tr>
  </tbody>
</table>

#### In-depth system design solutions

The separate [`engineering/in-depth-system-design-solutions/`](engineering/in-depth-system-design-solutions/) division contains one Markdown chapter per system. Each guide is a senior/staff/principal-level walkthrough with explicit assumptions, requirements, architecture, APIs, data models, correctness, failure handling, security, observability, scaling, trade-offs, and interviewer follow-ups.

<table>
  <thead>
    <tr>
      <th align="center"> </th>
      <th align="left">Solution guide</th>
      <th align="center">Guides</th>
      <th align="left">Coverage</th>
    </tr>
  </thead>
  <tbody>
    <tr><td align="center">01</td><td><a href="engineering/in-depth-system-design-solutions/design-twitter.md">Design Twitter</a></td><td align="center">1</td><td>Social graph, posts, hybrid timeline fan-out, ranking, and moderation</td></tr>
    <tr><td align="center">02</td><td><a href="engineering/in-depth-system-design-solutions/design-aarogya-setu.md">Design Aarogya Setu</a></td><td align="center">1</td><td>Consent, privacy-preserving exposure notification, epidemiology, and regional resilience</td></tr>
    <tr><td align="center">03</td><td><a href="engineering/in-depth-system-design-solutions/design-load-balancer.md">Design a load balancer</a></td><td align="center">1</td><td>L4/L7 forwarding, health, draining, configuration, and failover</td></tr>
    <tr><td align="center">04</td><td><a href="engineering/in-depth-system-design-solutions/design-url-shortener.md">Design a URL shortener</a></td><td align="center">1</td><td>Short-code generation, redirects, aliases, analytics, and abuse controls</td></tr>
    <tr><td align="center">05</td><td><a href="engineering/in-depth-system-design-solutions/design-logging-system.md">Design a logging system</a></td><td align="center">1</td><td>Durable ingestion, indexing, retention, query isolation, and audit integrity</td></tr>
    <tr><td align="center">06</td><td><a href="engineering/in-depth-system-design-solutions/design-google-play-store.md">Design Google Play Store</a></td><td align="center">1</td><td>App publishing, scanning, catalog, staged rollout, delivery, and compatibility</td></tr>
    <tr><td align="center">07</td><td><a href="engineering/in-depth-system-design-solutions/design-zoomcar-app-lld.md">Design Zoom car app LLD</a></td><td align="center">1</td><td>Domain objects, reservation/trip state machines, concurrency, and sagas</td></tr>
    <tr><td align="center">08</td><td><a href="engineering/in-depth-system-design-solutions/design-e-commerce-platform.md">Design an e-commerce platform</a></td><td align="center">1</td><td>Marketplace boundaries, checkout, inventory reservation, payment, and fulfillment</td></tr>
    <tr><td align="center">09</td><td><a href="engineering/in-depth-system-design-solutions/design-recommendation-system.md">Design a recommendation system</a></td><td align="center">1</td><td>Offline/online ML, candidate generation, ranking, experimentation, and safety</td></tr>
    <tr><td align="center">10</td><td><a href="engineering/in-depth-system-design-solutions/design-order-management-system.md">Design an order management system</a></td><td align="center">1</td><td>Order state, fulfillment orchestration, payment/refund correctness, and reconciliation</td></tr>
    <tr><td align="center">11</td><td><a href="engineering/in-depth-system-design-solutions/design-waste-management-app.md">Design an app for waste management</a></td><td align="center">1</td><td>Municipal scheduling, routing, telemetry, offline sync, and public metrics</td></tr>
    <tr><td align="center">12</td><td><a href="engineering/in-depth-system-design-solutions/design-library-management-system-lld.md">Design library management system LLD</a></td><td align="center">1</td><td>Catalog/circulation objects, loan and hold states, locking, and testable policies</td></tr>
    <tr><td align="center">13</td><td><a href="engineering/in-depth-system-design-solutions/design-warehouse-management-system.md">Design a warehouse management system</a></td><td align="center">1</td><td>Receiving, putaway, picking, ledger integrity, task leases, and cycle counts</td></tr>
    <tr><td align="center">14</td><td><a href="engineering/in-depth-system-design-solutions/design-parking-lot-reservation-system.md">Design a parking-lot reservation system</a></td><td align="center">1</td><td>Capacity holds, interval allocation, gates, payment, and occupancy reconciliation</td></tr>
    <tr><td align="center">15</td><td><a href="engineering/in-depth-system-design-solutions/design-real-time-inventory-tracking-system.md">Design a real-time inventory tracking system</a></td><td align="center">1</td><td>Event-sourced stock, projections, sequencing, reservations, and drift repair</td></tr>
    <tr><td align="center">16</td><td><a href="engineering/in-depth-system-design-solutions/design-ecommerce-rating-system.md">Design a rating system for an e-commerce website</a></td><td align="center">1</td><td>Reviews, verified purchase, moderation, aggregation, voting, and fraud controls</td></tr>
  </tbody>
</table>

### Technical Program / Project Management Track

The Technical Program / Project Management collection contains 30 questions adapted from the supplied interviewer-preparation set. These are intentionally separate from engineering questions: program and project manager answers should demonstrate technical credibility and delivery judgment without pretending to own implementation-level engineering decisions.

| Concept | Questions | Coverage |
| --- | ---: | --- |
| [Technical credibility](technical-program-manager/technical-credibility.md) | 10 | APIs, releases, architecture awareness, data, security, and production readiness |
| [Programme management](technical-program-manager/programme-management.md) | 10 | Recovery, ambiguity, prioritisation, stakeholders, ownership, and vendors |
| [Agile and ways of working](technical-program-manager/agile-and-ways-of-working.md) | 10 | Programme setup, ceremonies, speed versus quality, estimation, scaling, and influence |

## How to use it

1. Choose a concept and answer a question aloud before reading the model guidance.
2. Explain the requirements and constraints before proposing a solution.
3. Make trade-offs explicit: consistency versus availability, latency versus cost, or speed versus maintainability.
4. Cover failure modes, observability, rollout, and rollback for design questions.
5. Replace the model guidance with your own experience, metrics, and examples.

The original interview source—Atlassian, NVIDIA, or Booking.com—is retained inside each concept file as context. The guidance is a scaffold for reasoning, not a script to memorize.
