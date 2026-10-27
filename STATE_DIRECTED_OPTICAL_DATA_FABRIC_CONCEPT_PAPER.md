# A State-Directed Optical Data Fabric for Heterogeneous Processors
## A High-Level Concept Architecture

**Author:** Emerson Star (Shane A. Fortner)  
**Version:** 0.6 — 27 October 2026  
**Status:** Concept proposal; no processor hardware has been built.

### Abstract

Modern processors move large volumes of information among compute units, caches, memory, and other devices. Optical links can carry data at high aggregate rates, while electronic processors remain effective at general-purpose instruction execution. This paper proposes a hybrid architecture that combines those strengths: conventional electronic cores and memory connected by an optical data fabric, coordinated by a hierarchical, state-directed routing engine.

The proposed routing engine would track the condition and demand of the fabric, then coordinate local child routers and shared routes as traffic changes. Programs or operation graphs can be configured before a stream arrives; after the task header and route are accepted, a processing node can begin on the first valid beat and pipeline later beats without waiting for the full file to be staged. Active capacity should respond to admitted work, queue backlog, deadlines, receiver credits, and physical reconfiguration limits. The concept does not assume that light itself replaces a CPU, that a wavelength inherently represents a fixed number of bits, or that light is a persistent cache. Optical signals would be framed, detected, checked, and delivered to electronic processing and storage, except where a separately defined photonic operation processes an optical signal directly.

The concept now keeps two related architectural paths in view. A **hybrid CPU-fabric** uses conventional programmable cores and applies the fabric to selected data movement. A **network-native processor** makes processing elements nodes of the routed network, with streams carrying operands and partial results between them. Both use hierarchical state-directed routing, flow control, and 512-bit logical beats; they differ in where application computation occurs and how programs are mapped. An exploratory capacity model sizes a hypothetical fabric for 12 TB/s, with a 100× scale-out calculation to 1.2 PB/s. These are assumption-based projections, not physical throughput or end-to-end speedup.

### 1. Motivation

As processors add cores and accelerators, moving data between them can limit performance and energy efficiency. Photonics offers a way to carry information across links using optical channels, including multiple wavelengths and other signal dimensions. The practical data rate depends on the transmitter, receiver, modulation, coding, signal quality, channel spacing, and implementation. A wavelength is a transport channel; it does not by itself encode an arbitrary multi-bit “color word.”

A useful processor design must account for the full path. It must include the compute work, memory access, routing, queueing, signal conversion, control traffic, and error handling. Increasing the capacity of one link only helps an application when that link is a limiting stage and the rest of the system can keep up.

### 2. Proposed architecture

The paper studies two configurations built around a shared fabric and control foundation. They are related, but they are not the same processor model.

#### Shared fabric and control foundation

1. **512-bit logical stream beat.** A beat groups 64 bytes of payload for framing and processing. It is encoded on an optical signal over time; it is not 512 bits carried by one wavelength or one flash of light.
2. **Hierarchical state-directed network.** A master state engine maintains global routing policy and capacity information. Child routers manage local routes, queues, and credits. Route setup and burst descriptors amortize control over a stream; compatible beats may be coalesced while preserving address, permission, integrity, sequence, and ordering semantics.
3. **Electronic endpoints and buffering.** Transceivers convert between electrical and optical signals. Electronic FIFOs/SRAM hold mutable data and retry state. A bounded optical loop may be tested as a short delay element, not assumed to be persistent storage or an L1 cache.
4. **Flow control and recovery.** Frames carry source/destination, stream and sequence identity, payload length, route epoch, and integrity information. Credits, acknowledgements, retry, duplicate suppression, and backpressure protect data when queues or paths fail.
5. **Stream-first execution and elastic allocation.** Configure the operation and route before payload arrival where possible. An ingress controller reads enough framing/header information to validate the task and select the route; intermediate optical switches may forward payload without fully decoding it. A compute endpoint converts/detects the payload or invokes a defined photonic operation, then starts processing at the first complete valid beat. Results can flow onward as they are produced. Queue occupancy, accepted rate, task deadlines, receiver credits, and known burst demand guide active lane allocation. Keep installed peak capacity distinct from active rate and useful delivered rate. Channel or laser turn-on, wavelength settling, and power savings are hardware questions to measure; do not assume instantaneous reconfiguration or proportional power savings.

#### Architecture A: hybrid CPU-fabric

Conventional programmable cores execute application instructions and retain ordinary register, cache, and memory semantics. The optical fabric carries selected bulk transfers among CPU, accelerator, memory, and I/O domains. The state engine chooses and checks routes but does not execute the application program. This is the lower-risk integration path and the reference point for measuring whether optical transport helps a workload.

#### Architecture B: network-native processor

Processing elements are nodes in the routed graph. The network is the computer's execution topology: stream operands move to nodes, nodes apply defined operations using local state, and partial results flow to other nodes or destinations. The state engine coordinates routes, dependencies, and flow, but routing alone does not perform arithmetic. This path requires a distributed operation/instruction model, local state, synchronization, precise error behavior, and a compiler or mapping system. `DOT16.I16`, a signed 16-lane dot-product primitive over one 512-bit beat, is the first functional example.

Both configurations use the same general transport ideas. Results for one configuration should not be presented as proof for the other. For example, a transport capacity projection does not establish that the network-native nodes can execute a workload at that rate.

A valid-byte mask marks partial beats; padding is overhead and is excluded from useful-bandwidth totals. A timeout or size limit prevents a small flow from waiting indefinitely for a full burst.

The route descriptor and the payload batch are separate concepts. A long-lived route can be set up once while beats are processed continuously; the 512 KiB value in the capacity model is an assumed amount of payload amortized per controller lookup, not a requirement to buffer 512 KiB before execution begins. A kernel that requires a complete frame, temporal window, or global dependency must still wait for that required state. The first FPGA/software timing model should measure route setup, first-beat latency, steady-state valid beats per second, queue stalls, and end-of-stream completion separately.

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

The proposed contribution is narrower and architectural: a hierarchical master/child routing organization directed by explicit system state, evaluated in both a conventional CPU-fabric integration and a network-native processor topology. Whether either configuration or their shared control organization is novel requires a more complete literature and patent review. This paper makes no claim that the concept is the first optical CPU or the first optical router.

### 4. Working hypotheses

The architecture is intended to test the following hypotheses:

- A state-directed hierarchy can allocate optical path capacity to changing traffic demand while preserving local routing decisions.
- Coalescing compatible 512-bit beats into bursts can amortize route setup while preserving per-beat address, permission, and ordering semantics.
- Using optical links for selected high-volume transfers can raise available inter-domain bandwidth when electrical links are the limiting stage.
- Keeping instruction execution and ordinary cache state electronic can reduce the scope of the first implementation and preserve access to conventional software tools.
- Route selection should account for queue delay, contention, integrity failures, and conversion overhead, rather than maximizing raw link rate alone.
- The same state-directed fabric may support two distinct compute placements: conventional cores attached to the fabric, or processing elements distributed as network nodes. Each configuration needs separate workload and correctness tests.

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

The RV64 integration test establishes a small functional connection between a conventional instruction interpreter and lossless 512-bit stream framing. Separately, the `DOT16.I16` test maps a signed 16-lane dot-product primitive to 333,334 virtual processing nodes and an eight-ary reduction tree. The result matched an independent scalar reference. A 75-million-node software stress run also produced an exact tree-reduction result. These runs validate specific software functions and accounting; they do not encode or detect photons or validate physical routing.

A further **software scale-out test** executed 100,000 deterministic `DOT16.I16` tasks, totaling 6.4 MB of logical input, against a scalar reference. Four virtual nodes with different assumed service rates and a three-node retry produced exact results. An illustrative timing calculation under declared hypothetical rates estimated 100 ms on one virtual host, 39.412 ms with four nodes and local input, 61.200 ms when the input crossed a modeled shared link, and 55.455 ms when one node was unavailable and retained input was reassigned. These are not measured CPU, network, FPGA, or optical timings: the processing rates, 1-Gbit/s link, 10-ms control round trip, overlap, and retry-buffer availability are scenario assumptions. The test illustrates locality sensitivity; it does not establish speedup.

For the 100× scale-out target (1.2 PB/s), direct arithmetic using the prior per-resource assumptions calls for about 333,334 child paths, 500 active parent links plus one spare, 297 router shards, and 733 memory domains. The network-native `DOT16.I16` workload performs 31 scalar operations per 64-byte beat, or about 581 TOP/s aggregate at 1.2 PB/s if every beat is processed. These figures are conditional scale calculations, not evidence that this node count, memory system, compute rate, or throughput can be built. The 75-million-node stress used 4.8 GB of software payload; its 270 PB/s figure is only node count multiplied by an assumed per-node path rate.

Neither approach has a physical optical link, validated optical buffer, RTL processor, full-system memory model, or end-to-end workload benchmark. The two configurations remain separate evaluation tracks.

Accordingly, the 12 TB/s and 1.2 PB/s figures are **design targets that the assumptions are chosen to meet**, not achieved throughput. No processor prototype or operating-system execution exists. The models do not establish performance for gaming, servers, video editing, or general software. The hybrid path should first be evaluated on accelerator-style bulk transfers; the network-native path should first be evaluated on defined streaming primitives. Fine-grained CPU traffic and general-purpose distributed execution remain open questions, not claimed capabilities. The models do not show a CPU speedup.

A defensible performance claim will compare each architecture with an appropriate baseline using the same workload and input: a conventional CPU/NoC for the hybrid path, and a conventional distributed processor or accelerator for the network-native path. Report correctness, throughput, latency, and power separately. The model also needs calibration against measured processor and interconnect data before its predicted execution times can support comparative claims.

The demand-driven design should be evaluated with time-series counters for offered payload, active lane/channel count, useful valid bytes delivered, queue depth, stall cycles, first-beat latency, route setup/reconfiguration time, error/retry rate, and measured power. Report peak provisioned capacity separately from time-windowed active capacity and useful throughput. A fixed optical clock or carrier frequency is not the same as useful application throughput. The target controller should provision for backlog and deadlines, not just instantaneous arrivals, and preserve a minimum control path plus any reserve needed to meet latency objectives.

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
6. Report useful bandwidth separately from padded/raw bandwidth, plus median and tail latency, queue occupancy, correctness, and energy assumptions. Also measure first-beat start, steady-state beat rate, and end-of-stream time.
7. Sweep offered traffic and queue backlog; measure active lane/channel duty cycle, allocation response, hysteresis, optical/electronic reconfiguration delay, retry cost, and power. Keep logical allocation tests separate from physical laser/wavelength switching tests.
8. Connect instruction-generated memory transactions during execution to a cycle-accurate or RTL core/cache/router model; compare with a conventional electronic NoC and memory baseline using identical traces.
9. Calibrate against published hardware data, then against a physical link or prototype when available; run application benchmarks only after the model can represent their compute and memory behavior.

A result should be reported as a modeled projection until a calibrated model or hardware test supports it. A bandwidth gain should be reported separately from whole-application speedup.

### Conclusion

A hybrid and network-native processor fabric combining electronic computation, optical data movement, stream-first execution, demand-driven capacity allocation, and hierarchical state-directed routing is a concrete architecture concept worth evaluating for data-intensive workloads. Its plausible contribution lies in the execution and traffic organization, not in the general use of light or in biological novelty. A configured stream may begin processing at its first valid beat; the 512-bit beat defines a logical transfer unit, while the 12 TB/s figure remains a peak design target based on unvalidated assumptions. Effective throughput depends on offered load, data locality, compute capacity, queueing, and the full memory/link path.

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

