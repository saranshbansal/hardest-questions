# GPU, Parallelism & Hardware

Questions about CUDA, GPU systems, parallel execution, memory, hardware/software boundaries, and digital logic.

## NVIDIA: GPU and parallelism

### 2. GPU, Parallelism & Memory Depth

<details>
<summary><strong>Q001 · Explain how you'd optimize a CUDA kernel that is memory-bandwidth bound rather than compute bound.</strong></summary>

**Answer guidance**: I'd focus on maximizing memory coalescing (ensuring threads in a warp access contiguous memory), reducing redundant global memory traffic by staging data in shared memory when reused across threads, and increasing arithmetic intensity where possible (doing more compute per byte moved) so the kernel shifts closer to compute-bound. Profiling tools (Nsight Compute) would confirm whether bandwidth utilization is actually near the hardware ceiling before further tuning.


</details>

<details>
<summary><strong>Q002 · Walk through the trade-offs between different GPU memory hierarchies (registers, shared memory, global memory) for a given workload.</strong></summary>

**Answer guidance**: Registers are fastest but extremely limited and per-thread, so overuse causes register spilling to slower memory. Shared memory is fast and shared within a thread block, ideal for data reused across threads (e.g., tiling in matrix multiply), but limited in size and requires explicit management including bank-conflict avoidance. Global memory is large but far higher latency, so the general strategy is minimizing global memory round-trips by staging reusable data in shared memory and keeping hot per-thread values in registers without spilling.


</details>

<details>
<summary><strong>Q003 · How would you design a system to efficiently schedule thousands of concurrent GPU workloads across a multi-tenant cluster?</strong></summary>

**Answer guidance**: I'd separate scheduling policy (fair-share, priority, or SLA-based queuing) from the underlying resource manager (e.g., Kubernetes with GPU device plugins, or a purpose-built scheduler), and support both time-slicing and spatial partitioning (e.g., MIG) depending on whether workloads need full-GPU performance or can share fractional GPU resources. Multi-tenant fairness also requires guarding against a single tenant monopolizing scarce GPU memory or queue slots, typically via quotas.


</details>

<details>
<summary><strong>Q004 · Explain your approach to diagnosing a race condition in a highly parallel, multi-GPU training pipeline.</strong></summary>

**Answer guidance**: I'd first try to make the failure deterministic and reproducible by fixing seeds and reducing parallelism/scale until it still occurs, since debugging non-deterministic races at full scale is far harder. Tools like CUDA's race detector (compute-sanitizer) or careful placement of synchronization barriers to bisect where state diverges help narrow down the offending section, rather than guessing from stack traces alone.


</details>

<details>
<summary><strong>Q005 · How would you approach optimizing data transfer between host and device to minimize PCIe bottlenecks?</strong></summary>

**Answer guidance**: I'd use pinned (page-locked) host memory to enable faster, asynchronous DMA transfers, overlap transfer with compute using CUDA streams so the GPU isn't idle waiting on PCIe, and batch smaller transfers into fewer, larger ones since per-transfer overhead dominates for small payloads. If the workload allows, minimizing total data movement (e.g., keeping intermediate results on-device across pipeline stages) is often a bigger win than optimizing the transfer mechanism itself.


</details>

<details>
<summary><strong>Q006 · What strategies would you use to balance load across heterogeneous GPU generations in the same cluster?</strong></summary>

**Answer guidance**: I'd avoid naive even-splitting of work across GPUs and instead weight allocation by measured throughput per GPU generation, ideally with dynamic work-stealing so faster GPUs pick up more work rather than idling while waiting on slower ones in a lockstep design. For training specifically, this often means avoiding synchronous all-reduce patterns that force every GPU to wait for the slowest, in favor of more asynchronous or heterogeneity-aware parallelism strategies.


</details>

<details>
<summary><strong>Q007 · How do you evaluate when to use model/data parallelism vs. pipeline parallelism for a large-scale training job?</strong></summary>

**Answer guidance**: Data parallelism is the default when the model fits on a single device and you're scaling throughput across many devices with the same replica. Model parallelism becomes necessary when the model itself doesn't fit in one device's memory, splitting layers or tensors across devices. Pipeline parallelism helps when model parallelism alone leaves devices idle waiting on sequential dependencies, by overlapping micro-batches across pipeline stages — the trade-off is added complexity and "bubble" overhead that needs micro-batch tuning to minimize.


</details>

<details>
<summary><strong>Q008 · Explain your approach to designing fault-tolerant checkpointing for a multi-day distributed training run.</strong></summary>

**Answer guidance**: I'd checkpoint at a cadence balancing recovery cost against checkpoint overhead (too frequent hurts training throughput, too infrequent means large recompute loss on failure), write checkpoints asynchronously so they don't block training steps, and validate checkpoint integrity (not just existence) before relying on them for recovery. For very large models, I'd also consider sharded/distributed checkpoint writes rather than funneling everything through one node, which becomes a bottleneck and single point of failure.


</details>

<details>
<summary><strong>Q009 · As a technical leader, how would you set architectural direction for a team building a new GPU orchestration layer (e.g., on Kubernetes)?</strong></summary>

**Answer guidance**: I'd start from concrete workload requirements (latency-sensitive inference vs. long-running batch training have very different scheduling needs) rather than a generic "build a Kubernetes GPU scheduler" mandate, and evaluate what Kubernetes's existing device plugin ecosystem already solves versus what's genuinely novel to our workloads. I'd push the team to prototype against real workload traces early rather than architecting from first principles in a vacuum.


</details>

<details>
<summary><strong>Q010 · How do you balance the tension between low-level hardware optimization and maintainable, portable software architecture?</strong></summary>

**Answer guidance**: I'd isolate the hardware-specific optimized code behind a clean abstraction boundary (e.g., a well-defined interface for the optimized kernel/path) so the rest of the system stays portable and testable, and reserve deep hardware-specific tuning for the actual hot paths proven by profiling, not applied speculatively throughout the codebase. Over-optimizing broadly at the cost of maintainability is rarely worth it outside the small set of paths where it actually matters.



</details>

## NVIDIA: Hardware and digital logic

### 4. Hardware-Adjacent / Digital Logic

<details>
<summary><strong>Q011 · Explain how you'd design an asynchronous control sequencing circuit to avoid metastability issues.</strong></summary>

**Answer guidance**: I'd use synchronizer chains (typically two or more flip-flops in series) at every clock domain crossing to reduce the probability of metastability propagating downstream, and design the sequencing logic so it tolerates the added latency those synchronizers introduce rather than assuming instantaneous signal crossing. For control signals specifically, I'd favor Gray-coded or single-bit-change encodings when crossing domains, since simultaneous multi-bit changes risk transient invalid states being sampled.


</details>

<details>
<summary><strong>Q012 · Walk through the trade-offs of different flip-flop constructions for a high-speed pipeline stage.</strong></summary>

**Answer guidance**: Master-slave flip-flops are robust and widely used but have higher setup/hold overhead; pulse-triggered or transmission-gate-based designs can reduce that overhead for higher clock speeds at the cost of more careful timing analysis and higher sensitivity to clock skew. The choice depends on the target clock frequency and how much margin the process/technology gives you — I'd defer to the specific timing constraints and let simulation data drive the final choice rather than picking based on general preference.


</details>

<details>
<summary><strong>Q013 · How would you approach verifying correctness of a hardware/software co-designed feature under tight schedule pressure?</strong></summary>

**Answer guidance**: I'd prioritize verification effort toward the highest-risk interfaces (the actual hardware/software boundary, since that's where co-design bugs concentrate) rather than spreading effort evenly, and push for early integration testing on emulation/simulation platforms rather than waiting for real silicon, since schedule pressure makes late-discovered integration bugs far more costly than early ones.


</details>

<details>
<summary><strong>Q014 · Explain your approach to debugging a race condition that only manifests under specific clock domain crossings.</strong></summary>

**Answer guidance**: I'd use clock-domain-crossing-specific static analysis tools (CDC checkers) to flag unsynchronized crossings systematically rather than manually inspecting the whole design, and where a suspected crossing is found, verify with timing simulation under the specific clock phase relationships that trigger the failure, since these bugs are often intermittent and phase-dependent in ways that generic simulation misses.


</details>

<details>
<summary><strong>Q015 · How do you evaluate the right level of hardware abstraction to expose to firmware/driver engineers?</strong></summary>

**Answer guidance**: I'd expose the minimum interface needed for firmware to correctly and efficiently control the hardware, hiding implementation details that are likely to change across hardware revisions behind a stable API, so firmware doesn't need a rewrite every hardware generation. Where firmware genuinely needs low-level control for performance reasons, I'd version that lower-level access explicitly rather than baking hardware-version assumptions silently into the interface.


</details>

<details>
<summary><strong>Q016 · What's your strategy for ensuring software teams can iterate quickly against hardware that's still in development (simulation/emulation)?</strong></summary>

**Answer guidance**: I'd invest early in a functionally accurate simulation/emulation environment that software teams can use before silicon is available, even if it's slower than real hardware, since blocking software development until silicon arrives compresses the whole program's timeline unnecessarily. I'd also track and communicate divergences between the emulation model and evolving real hardware so software teams know what to re-validate once real hardware lands.


</details>

<details>
<summary><strong>Q017 · How would you approach a performance regression that only appears on certain hardware revisions?</strong></summary>

**Answer guidance**: I'd first isolate whether the regression correlates with a specific hardware change (new stepping, different memory timing, etc.) by systematically comparing revisions rather than assuming software is at fault, and use hardware performance counters to compare where time is actually spent across revisions. This is a case where premature blame between hardware and software teams wastes time — data first.


</details>

<details>
<summary><strong>Q018 · Explain your process for setting technical standards across teams that operate at different levels of the stack (hardware, firmware, driver, application).</strong></summary>

**Answer guidance**: I'd focus standards on the interfaces between layers (well-defined APIs, versioning conventions, timing/interface contracts) rather than trying to impose uniform practices within each layer, since hardware, firmware, driver, and application teams legitimately have very different constraints and tooling. Interface contracts are where cross-layer bugs concentrate, so that's where standardization pays off most.


</details>

<details>
<summary><strong>Q019 · As a Principal Engineer, how do you make the call on whether a bug should be fixed in hardware, firmware, or software?</strong></summary>

**Answer guidance**: I'd weigh fix cost and blast radius at each layer — hardware fixes are usually impossible or extremely expensive post-tape-out, firmware fixes can sometimes be field-updated, and software fixes are cheapest and fastest to deploy — and prefer the highest layer that can correctly and durably solve the problem, reserving hardware-level fixes for cases where lower layers genuinely cannot compensate.


</details>

<details>
<summary><strong>Q020 · How do you build trust and alignment between hardware and software engineering orgs with different velocity expectations?</strong></summary>

**Answer guidance**: I'd make each side's constraints and timelines visible to the other early and often (shared roadmaps, joint planning reviews) rather than letting velocity mismatches surface only as friction during a crunch, and explicitly acknowledge that hardware's slower iteration cycle isn't a lack of urgency — it's a different risk profile that software's faster cycle doesn't share. Trust comes from consistently honoring commitments made across that boundary, even small ones.


</details>
