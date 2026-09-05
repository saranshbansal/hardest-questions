# Front-End & Platform Engineering

Questions about front-end architecture, JavaScript platforms, performance, standards, and technical debt.

## Atlassian

### 3. Front-End & Modern JavaScript Practices

<details>
<summary><strong>Q001 · How do you evaluate whether a feature should be built as a micro-frontend vs. part of a monolithic front-end app?</strong></summary>

**Answer guidance**: I look at team topology first, not technology — micro-frontends solve an organizational scaling problem (independent deploy cadence for separate teams) more than a technical one, and they add real complexity (shared design system drift, bundle duplication, cross-app navigation). If a single team owns the feature and it doesn't need independent deployment, I'd keep it in the monolith. I'd only reach for micro-frontends when the coordination cost of shared deploys is demonstrably higher than the runtime/complexity cost of splitting.


</details>

<details>
<summary><strong>Q002 · What's your strategy for enforcing performance budgets across dozens of teams contributing to a shared front-end platform?</strong></summary>

**Answer guidance**: Budgets need to be automated and enforced in CI, not aspirational documentation — bundle size and key metrics (LCP, TTI) checked on every PR with hard failure thresholds, not just dashboards nobody looks at. I'd also make the cost visible at the point of decision (e.g., a bundle-size diff comment on the PR) rather than surfaced only after merge. For legitimate exceptions, I'd have a lightweight override process with an owner and expiry date, so exceptions don't silently become permanent.


</details>

<details>
<summary><strong>Q003 · How would you design a component library/design system that scales across multiple product teams while allowing local customization?</strong></summary>

**Answer guidance**: I'd separate core primitives (strict, versioned, rarely broken) from composable patterns (looser, teams can extend) so teams aren't forced to choose between full conformance and full escape-hatch. Theming/customization should go through defined design tokens rather than arbitrary CSS overrides, which preserves visual consistency while allowing brand or product-specific variation. Governance matters as much as code: a clear contribution process and a small core team reviewing breaking changes prevents the library from fragmenting into per-team forks.


</details>

<details>
<summary><strong>Q004 · Explain trade-offs between server-side rendering, static generation, and client-side rendering for a B2B SaaS dashboard.</strong></summary>

**Answer guidance**: A B2B dashboard is typically behind auth, personalized, and not SEO-sensitive, which removes the strongest argument for SSR/SSG (crawlability, fast first paint for anonymous users). I'd lean client-side rendering with aggressive code-splitting and a fast API, since the data is highly dynamic and per-user anyway — pre-rendering personalized dashboard content offers little benefit. I'd reconsider SSR only if initial load time on data-heavy views becomes a measured pain point for users on poor connections.


</details>

<details>
<summary><strong>Q005 · How do you approach state management in a large, long-lived single-page application with many contributing teams?</strong></summary>

**Answer guidance**: I'd draw a clear line between server state (data fetched from APIs — best handled by a dedicated caching/fetching library) and client/UI state (local to a component or a small feature), since conflating them into one global store is a common source of complexity in large apps. For genuinely shared client state, I'd scope it narrowly to the features that need it rather than a single global store everyone touches, since that shared mutable surface becomes a coordination bottleneck across teams.


</details>

<details>
<summary><strong>Q006 · What's your approach to migrating legacy front-end code (e.g., jQuery to a modern framework) incrementally in production?</strong></summary>

**Answer guidance**: I'd migrate page-by-page or feature-by-feature behind the existing routing, never attempt a big-bang rewrite of the whole app at once — that's a classic way to stall for a year and ship nothing. A strangler pattern where the new framework mounts into isolated DOM regions alongside legacy code lets both coexist during the transition, with automated visual/functional regression tests protecting the surfaces not yet migrated.


</details>

<details>
<summary><strong>Q007 · How would you architect a front-end plugin system so third-party UI extensions can't degrade core app performance?</strong></summary>

**Answer guidance**: I'd load third-party UI in isolated contexts (iframes, or web components with strict resource budgets) rather than sharing the host app's JS execution context directly, so a slow or buggy extension can't block the main thread. I'd enforce a resource/timeout budget per extension and fail gracefully (hide or placeholder the extension) rather than let it hang the whole page.


</details>

<details>
<summary><strong>Q008 · Describe your approach to setting front-end engineering standards (linting, testing, accessibility) across a large organization.</strong></summary>

**Answer guidance**: Standards need to be enforced by tooling, not convention — shared lint configs, CI gates, and accessibility checks (automated axe-core scans plus manual review for complex flows) baked into the shared build pipeline every team inherits by default. I'd version these standards like a product, with a changelog and migration guides when rules tighten, so teams aren't blindsided by a sudden wave of CI failures.


</details>

<details>
<summary><strong>Q009 · How do you balance technical debt reduction against feature delivery pressure as a technical leader?</strong></summary>

**Answer guidance**: I treat debt paydown as an ongoing cost of doing business, not a special project that competes for a separate budget — I push for a standing allocation (e.g., a percentage of each sprint) rather than periodic "debt sprints" that get deprioritized under pressure. I also prioritize debt by actual cost (velocity drag, incident frequency) rather than aesthetic discomfort, and I'm explicit with stakeholders about which debt is genuinely blocking future work versus which is safe to defer.


</details>

<details>
<summary><strong>Q010 · How would you mentor a front-end team through adopting a new rendering architecture with minimal disruption?</strong></summary>

**Answer guidance**: I'd start with a small, low-risk pilot surface, pair closely with a couple of engineers to build internal expertise before broad rollout, and document the concrete patterns (not just theory) that emerged from the pilot. I'd resist the urge to mandate the new architecture everywhere immediately — mentorship works better as "here's how we did it and what we learned" than "everyone must do this now," which builds genuine buy-in rather than compliance.


</details>
