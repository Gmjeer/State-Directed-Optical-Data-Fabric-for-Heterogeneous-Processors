# A State-Directed Optical Data Fabric for Heterogeneous Processors
## A High-Level Concept Architecture

**Author:** Emerson Star (Shane A. Fortner)  
**Version:** 0.4 — 27 October 2026  
**Status:** Concept proposal; no processor hardware has been built.

### Abstract

Modern processors move large volumes of information among compute units, caches, memory, and other devices. Optical links can carry data at high aggregate rates, while electronic processors remain effective at general-purpose instruction execution. This paper proposes a hybrid architecture that combines those strengths: conventional electronic cores and memory connected by an optical data fabric, coordinated by a hierarchical, state-directed routing engine.

The proposed routing engine would track the condition and demand of the fabric, then coordinate local child routers and shared routes as traffic changes. Its purpose is to organize data movement among compute and memory domains. The concept does not assume that light itself replaces a CPU, that a wavelength inherently represents a fixed number of bits, or that light is a persistent cache. Optical signals would be framed, detected, checked, and delivered to ordinary electronic processing and storage.

The potential contribution is the system-level organization of optical paths and routing state around heterogeneous compute and memory domains. The first evaluation target is bulk data movement for accelerators and other data-intensive workloads, not a demonstrated general-purpose CPU replacement. An exploratory capacity model sizes a hypothetical configuration for a 12 TB/s useful-aggregate-bandwidth target, using a 512-bit logical transfer beat and much larger bursts per route decision. It reaches the target only under specified, unmeasured assumptions; it does not demonstrate physical throughput or end-to-end speedup.

### 1. Motivation

As processors add cores and accelerators, moving data between them can limit performance and energy efficiency. Photonics offers a way to carry information across links using optical channels, including multiple wavelengths and other signal dimensions. The practical data rate depends on the transmitter, receiver, modulation, coding, signal quality, channel spacing, and implementation. A wavelength is a transport channel; it does not by itself encode an arbitrary multi-bit “color word.”

A useful processor design must account for the full path. It must include the compute work, memory access, routing, queueing, signal conversion, control traffic, and error handling. Increasing the capacity of one link only helps an application when that link is a limiting stage and the rest of the system can keep up.

### 2. Proposed architecture

The concept has five cooperating parts:

1. **Electronic compute and memory.** General-purpose CPU cores, caches, memory controllers, and RAM retain conventional digital state and execute ordinary instructions.
2. **Optical data fabric.** Optical links carry framed transfers among selected compute, cache, memory, and I/O endpoints. The initial interface uses a **512-bit (64-byte) logical beat**. This is a group of payload bits encoded onto an optical signal over time, not 512 bits carried by one wavelength or one flash of light.
3. **Hierarchical routing control.** A master state engine maintains system-level routing policy and capacity information. Child routers manage local paths and queues, reporting availability and demand to the master. The proposed fast path uses local admission and burst descriptors so the master does not need to make a software-level decision for every 512-bit beat.
4. **Electronic endpoints.** At each boundary, transceivers encode electrical data into optical signals and detect incoming signals back into electrical data. Endpoint logic checks frames and places accepted transfers into electronic queues for the destination.
5. **Hybrid buffering.** Electronic FIFOs/SRAM hold mutable data and route descriptors. A bounded optical loop may be tested as a short delay element for in-flight light, but it is not assumed to be persistent storage or an L1 cache; recirculation would need measured loss, noise, and timing limits.

The state engine’s job is to select among available routes, coordinate capacity, and respond to congestion or path faults. It does not execute application instructions or replace cache and memory state. A frame would carry enough control information for its destination and integrity to be checked, while the payload travels over an assigned optical path. A local coalescer may combine compatible beats into a larger burst under one route descriptor. It must preserve each beat’s address, permissions, integrity status, and required ordering. A valid-byte mask marks partial beats; padding is overhead and is excluded from useful-bandwidth totals. A timeout or size limit prevents a small flow from waiting indefinitely for a full burst.

#### Bio-inspired control heuristics

These are engineering analogies, not claims that the processor reproduces biological mechanisms:

- **Heart: one-way admission.** Credit-based gates admit a burst only when the next buffer has room, preventing a sender from overrunning its receiver.
- **Brain: local demand response.** Child routers adjust their service shares from local queue and receiver-credit signals; the master sets slower system-wide quotas and priorities.
- **Lungs: hierarchical distribution.** Parent paths branch into child groups and endpoints. Capacity must scale at every level with downstream demand; extra leaf paths do not help if an upstream branch is undersized. Lung studies show that matching flow depends on more than branch geometry, so equal utilization of all child paths must be measured, not assumed.
- **Liver: selective first-pass checks.** Every ingress frame gets inexpensive framing, destination, authorization, and integrity checks. Suspect frames use a separate repair or quarantine path. A transparent CPU fabric preserves valid payloads; it does not discard data merely because it appears unnecessary.

The model should treat queue occupancy and receiver credits as control signals, not literal fluid pressure. Physiology motivates the control pattern; digital queueing and optical link models must evaluate it.

This hybrid boundary keeps the proposal compatible in principle with established processor software models. It also makes conversion and buffering costs visible. Persistent cache contents remain electronic in the initial architecture; optical transport is not treated as a store unless a separate storage technology is specified and evaluated.

### 3. Potential contribution and relationship to prior work

Optical data transport and optical computation are established research areas. A 2025 photonic-link study reports an 80-channel transmitter array with an aggregate rate of 800 Gb/s, demonstrating progress in dense optical interconnect components. Photonic tensor-core research demonstrates specialized optical processing for operations such as neural-network matrix calculations. Other work has demonstrated dynamically switched photonic networks-on-chip. Prior optical network-on-wafer research has also examined control mechanisms for multi-chip systems and discussed terabyte-per-second-class per-module bandwidth as an architectural goal.

There is also a direct conceptual neighbor: a 2024 paper presents an all-optical general-purpose CPU architecture and a photonic integrated circuit implementation of a minimal processor design. These results show that the broader idea of optical computing, optical processor proposals, and photonic routing are not new by themselves.

The proposed contribution is narrower and architectural: a hierarchical master/child routing organization, directed by explicit system state, integrated with conventional electronic cores and cache/memory domains. Whether this combination is novel requires a more complete literature and patent review. This paper makes no claim that the concept is the first optical CPU or the first optical router.

### 4. Working hypotheses

The architecture is intended to test the following hypotheses:

- A state-directed hierarchy can allocate optical path capacity to changing traffic demand while preserving local routing decisions.
- Coalescing compatible 512-bit beats into bursts can amortize route setup while preserving per-beat address, permission, and ordering semantics.
- Using optical links for selected high-volume transfers can raise available inter-domain bandwidth when electrical links are the limiting stage.
- Keeping instruction execution and ordinary cache state electronic can reduce the scope of the first implementation and preserve access to conventional software tools.
- Route selection should account for queue delay, contention, integrity failures, and conversion overhead, rather than maximizing raw link rate alone.

These are hypotheses, not established findings. A successful design may improve bandwidth for some workloads while providing little benefit for workloads limited by computation, latency, or serial dependencies.

### 5. Preliminary model and evidence boundary

The software models represent compute and memory domains, optical paths, state/action dispatch, and routing limits using configurable assumptions. An earlier two-domain comparison projected twice the one-domain memory-bandwidth ceiling, assuming both domains sustain their configured rates and the fabric can carry the traffic. A newer scenario sizes a hypothetical system for **12 TB/s of useful aggregate payload bandwidth**. Here, 512 bits is the logical beat width; it is distinct from the much larger **512 KiB burst** assumed per state lookup:

| Modeled stage | Scenario assumption | Modeled ceiling |
|---|---|---:|
| Child paths | 3,334 paths × 4 GB/s raw × 90% efficiency | 12.0024 TB/s |
| Parent links | Six links × 2.666667 TB/s raw × 90%; five remain after one failure | 12.0000 TB/s |
| Router shards | Three × 4.5 TB/s raw × 90% | 12.15 TB/s |
| Memory domains | Eight × 1.638 TB/s | 13.104 TB/s |
| State/action dispatch | Four assumed 2.9 GHz control cores at 500 cycles per lookup; one lookup per 512 KiB batch | 12.163 TB/s equivalent |

In the healthy capacity scenario, the calculated system ceiling is **12.0024 TB/s**, limited by the child-path assumption. With one parent link unavailable, the calculation is **12.0000 TB/s**, leaving effectively no margin. A 512-bit beat is 64 bytes; if the modeled controller performs one lookup for each beat, its 23.2 million lookups per second limit payload throughput to **1.485 GB/s**. The 512 KiB burst assumption combines 8,192 such beats under one lookup. Halving the burst to 256 KiB lowers the modeled ceiling to 6.082 TB/s. Routing each useful payload byte across the fabric twice or four times lowers the modeled ceiling to 6.0012 TB/s or 3.0006 TB/s. Staging every payload byte through the modeled 4.096 TB/s SRAM also limits the model to 4.096 TB/s.

These are capacity calculations from scenario inputs, not measured device characteristics. The model assumes, among other things, that one state lookup can authorize a 512 KiB payload batch; it does not establish that this is correct or safe for general-purpose CPU workloads.

A new **functional software integration test** connects the RV64 state-engine interpreter to the stream-framing model. A machine-code loop using `LUI`, `ADDI`, `SD`, `BNE`, and `EBREAK` retired 4,099 instructions and emitted 1,024 sequential 64-bit stores (8,192 bytes). The test grouped eight 64-bit values into each of 128 logical 512-bit beats; SHA-256 hashes matched before and after packing, confirming that the payload was preserved. The harness captures stores after the interpreter executes them. It does not send instruction-generated transactions through a timed cache, memory hierarchy, or optical router, and it does not measure throughput or energy.

The integration test then reapplies the existing route-setup capacity assumptions across payload sizes. Under those assumptions, the setup-controller ceiling is 1.485 GB/s when each route setup covers one 64-byte beat, 0.095 TB/s for 4 KiB, 1.520 TB/s for 64 KiB, and 12.163 TB/s for 512 KiB; at the last size, the modeled child-path ceiling limits delivered capacity to 12.0024 TB/s. These are arithmetic projections, not measurements, and the long-stream case does not show that arbitrary workloads can combine data into such streams.

The test establishes a small functional connection between RV64 instruction execution and lossless 512-bit stream framing. It does not encode or detect photons. The project still has no physical optical link, validated optical buffer, RTL processor, full-system memory model, or end-to-end workload benchmark.

Accordingly, the 12 TB/s figure is a **design target that the assumptions are chosen to meet**, not achieved throughput. No processor prototype or operating-system execution exists, and the models do not establish performance for gaming, servers, video editing, or general software. The first practical evaluation should focus on accelerator-style streaming and bulk transfers. Fine-grained CPU traffic remains an open question, not a claimed capability. The models do not show a CPU speedup.

A defensible performance claim will require the same named workloads and inputs on both the proposed system and a modern reference processor, with correctness, throughput, latency, and power reported. The model also needs calibration against measured processor and interconnect data before its predicted execution times can support comparative claims.

### 6. Engineering questions

A complete design would need to define the physical meaning of an optical path, transceiver rates, channel count, framing, coding, error correction, and the optical power budget. It would also need detailed queue and arbitration behavior, routing update time, retry policy, cache coherence, memory ordering, and failure recovery.

System-level evaluation must include end-to-end energy and latency: laser generation, modulation, detection, electrical drivers, control logic, buffering, and cooling. The design must also address optical loss, crosstalk, thermal drift, manufacturing variation, and the cost of converting signals between electrical and optical form. Optical buffering and persistent optical storage require separate device evidence.

### 7. Evaluation plan

A staged evaluation can make the concept testable:

1. Define one reference processor configuration and a documented memory hierarchy.
2. Sweep transfer sizes from 64-byte beats through multi-kilobyte and 256/512 KiB bursts, using address traces and cache/coherence transactions.
3. Compare centralized per-beat control with local credit gates, child-router demand response, and batched route descriptors.
4. Model route allocation, congestion, failures, exception traffic, and fallback behavior.
5. Compare one, two, and additional memory domains under identical accelerator-style and general-purpose traces.
6. Report useful bandwidth separately from padded/raw bandwidth, plus median and tail latency, queue occupancy, correctness, and energy assumptions.
7. Connect instruction-generated memory transactions during execution to a cycle-accurate or RTL core/cache/router model; compare with a conventional electronic NoC and memory baseline using identical traces.
8. Calibrate against published hardware data, then against a physical link or prototype when available; run application benchmarks only after the model can represent their compute and memory behavior.

A result should be reported as a modeled projection until a calibrated model or hardware test supports it. A bandwidth gain should be reported separately from whole-application speedup.

### Conclusion

A hybrid processor fabric that combines electronic computation with optical data movement, burst coalescing, and hierarchical state-directed routing is a concrete architecture concept worth evaluating first for data-intensive accelerator transfers. Its plausible contribution lies in the control and traffic organization, not in the general use of light or in biological novelty. The 512-bit beat defines a transfer unit; the 12 TB/s projection additionally depends on amortizing route decisions over large bursts.

The next step is a calibrated core, cache, coherence, and router model with real workload traces. Until a processor is implemented and benchmarked, performance advantages remain unproven.

### Selected references

- M. Kissner, L. Del Bino, F. Päsler, P. Caruana, and G. Ghalanos, “An All-Optical General-Purpose CPU and Optical Computer Architecture,” *Journal of Lightwave Technology*, vol. 42, pp. 7999–8013, 2024. [Optica Publishing Group](https://opg.optica.org/jlt/abstract.cfm?uri=jlt-42-22-7999)
- “Three-dimensional photonic integration for ultra-low-energy, high-bandwidth interchip data links,” *Nature Photonics*, 2025. [Nature Photonics](https://www.nature.com/articles/s41566-025-01633-0)
- “Dynamic switching of a packaged photonic integrated network-on-chip using an FPGA controller,” *Optics Letters*, 2018. [Optica Publishing Group](https://opg.optica.org/ol/abstract.cfm?uri=ol-43-21-5471)
- “On the network design and control of an optical network: interconnecting multiple chips on a wafer,” *Journal of Optical Communications and Networking*, vol. 15, no. 2, 2023. [Optica Publishing Group](https://opg.optica.org/jocn/abstract.cfm?uri=jocn-15-2-119)
- “120 GOPS Photonic tensor core in thin-film lithium niobate for inference and in situ training,” *Nature Communications*, 2024. [Nature Communications](https://www.nature.com/articles/s41467-024-53261-x)
- J. A. H. R. Claassen et al., “Regulation of cerebral blood flow in humans: physiology and clinical implications of autoregulation,” *Physiological Reviews*, vol. 101, no. 4, 2021. [American Physiological Society](https://doi.org/10.1152/physrev.00022.2020)
- W. Kang et al., “Gravity outweighs the contribution of structure to passive ventilation-perfusion matching in the supine adult human lung,” *Journal of Applied Physiology*, vol. 124, no. 1, pp. 23–33, 2018. [American Physiological Society](https://doi.org/10.1152/japplphysiol.00791.2016)
- J. M. Mates et al., “Mouse Liver Sinusoidal Endothelium Eliminates HIV-Like Particles from Blood at a Rate of 100 Million per Minute by a Second-Order Kinetic Process,” *Frontiers in Immunology*, vol. 8, article 35, 2017. [Frontiers](https://doi.org/10.3389/fimmu.2017.00035)
- National Heart, Lung, and Blood Institute, “How the Heart Works: Blood Flow Through the Heart.” [NHLBI](https://www.nhlbi.nih.gov/health/heart/blood-flow)

