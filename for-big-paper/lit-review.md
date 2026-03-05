Hybrid Instruction Set Computing: A Comprehensive Literature Review

Hybrid Instruction Set Computing (HISC) refers to processor designs that combine, blend, or dynamically switch between two or more instruction set architectures (ISAs), execution modes, or instruction-width encodings — either at the chip level, within the pipeline, or across heterogeneous core clusters. Far from a niche curiosity, it underpins almost every modern processor you interact with today, from the phone in your pocket to a cloud datacenter. This review surveys its full intellectual landscape: history, theory, implementations, academic literature, key players, and speculative futures.



Foundational Concepts

An Instruction Set Architecture (ISA) is the contract between software and hardware — it defines the instruction opcodes, register file, memory model, and addressing modes a program can rely on. The two dominant philosophies have historically been:

​



CISC (Complex Instruction Set Computing): Rich, variable-length instructions capable of complex operations in a single opcode (e.g., x86, IBM 360/370). Code density is high; hardware decoders are complex.



RISC (Reduced Instruction Set Computing): Small, fixed-length, load-store instructions. Hardware is simpler; compilers bear the complexity. (e.g., MIPS, SPARC, early ARM, RISC-V).



Hybrid ISC arose because neither paradigm is universally optimal. Real workloads simultaneously demand high code density, low power, peak performance, legacy compatibility, and specialised acceleration — constraints that no single ISA philosophy cleanly satisfies. Hybridisation is the engineering response to that tension.

​



Historical Background

The RISC/CISC Wars (1960s–1980s)

The IBM System/360 (1964) established CISC as the dominant commercial paradigm, prioritising programmer productivity through rich instruction sets. IBM's 360/370 series remains notable for being virtualizable at very low overhead, and its descendants underpin IBM mainframes to this day.

​



The RISC counter-revolution emerged simultaneously from two academic centres in the early 1980s: David Patterson's RISC-I at UC Berkeley (1981) and John Hennessy's MIPS at Stanford. The core insight — that simple instructions, a large uniform register file, and pipeline-friendly fixed-width encodings could outperform complex ISAs when combined with optimising compilers — had profound and enduring consequences. Crucially, early studies found that the instruction count and instruction mix were "ISA-independent to first order," meaning that RISC and CISC compilers tend to generate broadly similar computational work per unit of high-level code; the real differentiators are microarchitecture and power management.



ARM and the Embedded Hybrid Problem (1985–2003)

The Advanced RISC Machine (ARM) architecture, born at Acorn Computers in 1985, achieved remarkable results with a clean 32-bit RISC ISA optimised for power efficiency and embedded applications. However, its 32-bit fixed instruction width meant poor code density — a serious problem in memory-constrained embedded systems. ARM's solution was architecturally groundbreaking: the Thumb instruction set (introduced in ARMv4T, ~1994), a 16-bit compressed re-encoding of the most-used ARM instructions. This created the first mainstream commercially deployed hybrid ISA — a single processor that could execute two instruction widths, switching mode via a branch instruction. The processor's CPSR (Current Program Status Register) tracked which mode was active.



Thumb offered code sizes approximately 65–70% of equivalent ARM code, at the cost of some execution efficiency due to the restricted register access (only 8 of the 16 registers were directly accessible in Thumb). This trade-off was context-dependent but highly valuable for flash-based microcontrollers where memory cost and footprint dominated.

​



Thumb-2, introduced with the ARM1156 in 2003, resolved the efficiency penalty by extending Thumb with 32-bit instructions freely intermixable with 16-bit ones — a variable-length hybrid capable of matching ARM32's performance while preserving Thumb's code density. ARM also introduced Unified Assembly Language (UAL) at this time, allowing a single source assembly to target either ISA.



x86 Goes RISC Internally (1995–Present)

Intel's P6 microarchitecture (Pentium Pro, 1995) is one of the most consequential hidden hybridisations in computing history. Rather than simplifying the x86 ISA (which would have shattered binary compatibility), Intel instead retained the CISC interface while internally translating x86 instructions into fixed-width RISC-like micro-operations (micro-ops) before execution. The x86 front-end became a CISC-to-RISC translator; the back-end execution engine is effectively RISC. This architectural schizophrenia allowed Intel to preserve backwards compatibility with the enormous x86 software ecosystem while exploiting the microarchitectural benefits of a regular, pipelined internal representation.



The P6 design split memory operations into separate micro-ops (e.g., a memory store became a store-address and a store-data micro-op). Intel's subsequent Pentium M introduced micro-fusion — fusing pairs of related micro-ops back into single reorder buffer entries to reduce scheduling pressure — and the Core architecture extended this to full macro-fusion of compare-and-branch pairs. This iterative refinement of the internal RISC representation, invisible to application programmers, has continued through every Intel generation since. AMD adopted the same strategy with its K5 (1996), which used 86-bit RISC86 internal micro-ops.

​



VLIW and EPIC: Compiler-Driven Hybrids (1990s–2000s)

Very Long Instruction Word (VLIW) and Explicitly Parallel Instruction Computing (EPIC) architectures represented a different kind of hybrid: rather than hiding complexity in the decoder, they exposed instruction-level parallelism explicitly in the ISA, placing the burden of scheduling on the compiler. The Intel Itanium (IA-64), co-developed with HP and launched in 2001, was the flagship EPIC processor. Its ISA included 128 integer registers, predicated execution (allowing the compiler to eliminate branches), speculative non-faulting loads, a rotating register frame for software-pipelined loops, and software-assisted branch prediction. Itanium issued instructions in 128-bit bundles of three, with explicit stop bits indicating dependency boundaries.

​



While Itanium's technical architecture was sophisticated, its commercial fate was poor — the compiler burden was immense, legacy x86 binary emulation was slow, and the ecosystem never achieved the critical mass needed to amortise software porting costs. VLIW survives successfully in Digital Signal Processors (DSPs), where workloads are static, structured, and compiler-friendly — a crucial qualification about the domain-specificity of hybrid ISA strategies.

​



Taxonomising Hybrid ISA Approaches

It is worth establishing a clear taxonomy, as "hybrid" is used loosely across the literature. Five primary categories can be identified:



Category	Mechanism	Key Examples

Intra-ISA width hybrids	Variable-length instruction encodings in a single ISA	ARM Thumb/Thumb-2, RISC-V C extension

Internal micro-op translation	CISC ISA frontend translating to RISC backend	x86 P6 and all descendants, AMD K5+

Heterogeneous core hybrids	Multiple ISA-compatible core designs in one package	ARM big.LITTLE, Intel Alder Lake (P+E)

Cross-ISA binary translation	Running code compiled for ISA-A on hardware ISA-B	Apple Rosetta 2, QEMU, FX!32

ISA extension/DSA hybrids	Base ISA augmented with domain-specific instructions	RISC-V custom extensions, Intel AMX, ARM SVE

Intra-ISA Width Hybrids

The RISC-V Compressed (RVC) "C" extension implements the same principle as ARM Thumb: a 16-bit compressed encoding of the most frequently used RISC-V instructions. As noted in Waterman's foundational 2016 Berkeley dissertation Design of the RISC-V Instruction Set Architecture, ARM's Thumb offers "competitive code size but low orthogonality" compared to the cleaner RVC design. Unlike ARM's Thumb, which required explicit mode-switching, RISC-V's compressed instructions are interleaved in the instruction stream at a per-instruction granularity — the decoder identifies instruction width from the two least-significant bits of each fetch, making the hybrid completely transparent to the programmer and compiler.

​



Internal Micro-Op Translation

The P6 story is now well-established, but subtler hybridisation occurs even within RISC processors. Modern ARM Cortex-A cores internally decompose some A64 instructions into multiple micro-ops for the out-of-order execution engine. The external ISA is a clean 32-bit fixed-width RISC (AArch64), but the internal execution engine operates on a different, wider, partially fused representation. The distinction between "ISA" and "microarchitecture" becomes genuinely blurry here — the internal representation is a form of micro-ISA invisible above the pipeline.

​



Heterogeneous Core Hybrids

ARM's big.LITTLE architecture (introduced with the Cortex-A7/A15 pairing in 2011) was the first mainstream realisation of heterogeneous-core, same-ISA multiprocessing. Crucially, both core types execute identical ISA binary code — the hybridisation is in performance and power profile, not instruction semantics. The OS scheduler migrates threads between core clusters. ARM's DynamIQ (2017+) refined this by allowing the big and LITTLE clusters to share an L3 cache and communicate at lower latency, enabling finer-grained migration.

​



Intel's Alder Lake (12th Gen Core, 2021) brought this concept to x86 desktop computing for the first time — though technically Intel had trialled it in the mobile-only Lakefield (2020). Alder Lake combined up to eight Golden Cove P-cores (optimised for single-thread performance with Hyper-Threading) and up to eight Gracemont E-cores (Atom-lineage, optimised for throughput and background tasks). An E-core delivers "equivalent performance at 40% less power than Skylake," or alternatively 40% more performance at the same power. The Intel Thread Director technology provides hardware telemetry to the OS scheduler, classifying threads by their instruction mix (e.g., heavy SSE/AVX vs lightweight integer work) and recommending appropriate core placement. A SPEC 2024 paper from TU Dresden found that while the heterogeneity of Alder Lake can improve performance and energy efficiency, it also introduces scheduling complexity that imposes real overhead if the OS does not correctly partition workloads.

​



Apple Silicon M-series chips represent arguably the most sophisticated heterogeneous hybrid currently commercially deployed. Beyond CPU P-core/E-core hybridisation (running AArch64), the M-series SoC incorporates: a Neural Engine (a fixed-function neural network accelerator with its own dataflow), an AMX (Apple Matrix coprocessor) controlled by CPU instructions, a tiled GPU with a different instruction set, and a unified memory architecture where all components share a single physical DRAM pool. The AMX is particularly instructive for hybrid ISA research — it is not a separate MMIO-mapped peripheral but receives commands via special CPU instructions (a form of ISA extension), making it architecturally analogous to RISC-V custom extensions but at product scale. The M4 reaches 103 GB/s peak GPU memory bandwidth and 2.9 FP32 TFLOPS.



Key Academic Literature and Research Groups

The ISA Wars: Does ISA Choice Matter?

The most important empirical paper on RISC vs. CISC is "Power Struggles: Revisiting the RISC vs. CISC Debate on Contemporary ARM and x86 Processors" (Blem, Menon, Sankaralingam; HPCA 2013). The key finding was that instruction count and mix are "ISA-independent to first order," and that performance differences are generated by ISA-independent microarchitecture differences — not the ISA itself. This is a crucial theoretical underpinning for hybrid ISA research: if ISA is relatively neutral with respect to computation, then the real value of hybrids lies in power, code density, compatibility, and specialisation rather than in raw performance.



A complementary analysis, "ISA Wars" (Blem et al., TOCS 2015), extended this finding and quantified the convergence of RISC and CISC implementations — finding cycle count gaps of no more than 2.5× across ISA families, with the gap attributable to microarchitecture.

​



CHERI: Capability Hardware Enhanced RISC Instructions

CHERI (Capability Hardware Enhanced RISC Instructions), developed at the University of Cambridge since 2010, is one of the most intellectually ambitious hybrid ISA projects in academia. It extends standard ISAs (MIPS, RISC-V, and an experimental Morello ARM64 implementation) with hardware capability registers — pointer-like values that carry cryptographically unforgeable bounds and permission metadata. A CHERI-aware process can only access memory through valid capabilities, providing hardware-enforced spatial and temporal memory safety. The CHERI ISA v9 technical report (released 2023) documents thirteen years of iterative refinement, formal analysis, and hardware validation. CHERI is notable because it is a security-motivated hybrid: the base ISA remains unchanged; the capability instructions form an orthogonal extension layer. It has been adopted by DARPA's MORPHEUS programme and influenced the design of Arm's experimental Morello research chip (2022).

​



HASTE: Hybrid Architectures at CMU

HASTE (Hybrid Architectures for Software and Temporal Execution) from Carnegie Mellon University represents an early academic investigation of fine-grained multi-ISA hybridisation. The HASTE architecture explored combining a general-purpose processor core with a reconfigurable fabric and a specialised Hybrid Computation Unit (HCU), developing a novel application representation that could target all three substrates. The 2003 FCCM paper described how HASTE's application representation allowed efficient scheduling across the fabric, ISA cores, and HCU without the programmer needing to manually partition computation — an early statement of what would later be called heterogeneous compilation.

​



RISC-V VLIW Hybrid Scheduling (2024)

A 2024 paper from the Journal of Software (China), "Hybrid Instruction Scheduling Algorithm for RISC-V VLIW Architecture," addresses the problem of instruction-level parallelism optimisation in RISC-V VLIW designs for DSP applications. The research proposed combining Integer Linear Programming (ILP) scheduling (optimal but computationally expensive) with list scheduling (fast but suboptimal) using a theoretical IPC upper-bound model to identify only those scheduling regions where list scheduling fails to reach optimality. The IPC model achieved 95.74% accuracy, and the hybrid algorithm identified that 94.62% of scheduling regions already reached optimality with list scheduling — applying expensive ILP only to the remaining 5.38%. This achieves ILP-quality results with complexity comparable to list scheduling, and is representative of a broader trend: using RISC-V as a research substrate for hybrid architectural investigations.

​



HDC-CNN Hybrid Acceleration on Custom RISC-V (arXiv 2025)

A 2025 arXiv paper (arXiv:2511.05053) proposes a custom GPU architecture extending RISC-V with four specialised Hyperdimensional Computing (HDC) instructions. HDC is a brain-inspired computing paradigm that encodes information in high-dimensional binary vectors; integrating it with CNN inference requires both vector arithmetic and tensor operations. By extending the RISC-V ISA with custom opcodes for HDC's core operations (binding, bundling, similarity), the authors achieved significant reduction in memory access overhead and speedup over a baseline GPU. This paper exemplifies the post-RISC-V research era: a stable, open, extensible base ISA used as a foundation for domain-specific hybrid instruction design.

​



Cross-ISA Binary Translation Research

The problem of running code compiled for one ISA on hardware implementing another — cross-ISA Dynamic Binary Translation (DBT) — is a vibrant research area with direct practical relevance. A 2024 USENIX ATC paper addresses the challenge that the increasing prevalence of new ISAs necessitates migration of closed-source binaries across ISAs, with DBT as the key technology. The NSF-funded work "Cross-ISA Machine Instrumentation using Fast and Scalable Dynamic Binary Translation" introduced three novel techniques: leveraging the host FPU for floating-point emulation (observing that most FP operations can be correctly emulated by wrapping them with minimal non-FP code), designing a shared code cache that scales for high parallelism, and providing ISA-agnostic instrumentation. Apple's Rosetta 2 (2020), which runs x86-64 macOS applications on Apple Silicon at near-native performance via ahead-of-time translation, is the highest-profile commercial deployment of these principles.



X-HEEP: Open-Source Extensible RISC-V Platform (2024)

X-HEEP (eXtendible Heterogeneous Energy-Efficient Platform) is an open-source RISC-V SoC platform designed explicitly for hybrid ISA experimentation through custom accelerator extensions. It uses the CORE-V-XIF (eXtension Interface) to allow custom accelerators to extend the RISC-V instruction set transparently — the processor core offloads unrecognised instructions to a connected accelerator rather than trapping. This "disaggregated ISA extension" model allows researchers to prototype ISA hybrids without modifying the processor itself, and represents an important infrastructure contribution to the field.

​



Major Industry Players

Intel

Intel's hybrid ISA journey spans: P6 micro-op translation (1995), IA-64/EPIC (2001), AVX/AVX-512 vector extensions, AMX matrix extensions (Sapphire Rapids, 2023), Lakefield (2020, first hybrid desktop mobile), Alder Lake (2021), Raptor Lake (2022), Meteor Lake (2023), and Arrow Lake/Lunar Lake (2024–25). Intel's Thread Director hardware-OS co-design for workload scheduling across P and E cores continues to evolve, with each generation adding more telemetry classes and scheduler hints.

​



ARM / Qualcomm / Apple

ARM Holdings controls the ISA IP; its DynamIQ architecture underpins big.LITTLE implementations in every Android flagship. Qualcomm's Oryon cores (Snapdragon X Elite, 2024) are custom ARM64 microarchitectures combining performance-optimised Oryon P-cores with efficiency clusters and a dedicated Hexagon NPU for AI workloads. Apple's M-series represents the furthest integration, with ISA hybridisation effectively operating at SoC scale.



AMD

AMD historically resisted explicit P+E heterogeneous core designs on the x86 desktop (unlike Intel), instead pursuing symmetric multi-core performance. However, AMD's recent Strix Halo and Strix Point APU designs (2024–25) incorporate RDNA GPU cores, XDNA AI engines (NPUs), and traditional Zen 5 CPU cores on a unified die, representing a DSA-hybrid approach even without differentiated CPU core types.



SiFive and the RISC-V Ecosystem

SiFive is the leading commercial RISC-V IP provider, offering cores ranging from ultra-low-power E-series microcontrollers to high-performance P-series application processors. Its Intelligence X280 vector core implemented the RISC-V V extension (vector instructions) as a hybrid overlay on the base integer ISA. The broader RISC-V International consortium (600+ member organisations as of 2024) governs the open ISA standard, managing a modular extension system where hybrid instruction capabilities are first-class citizens — the "base + extensions" model is architecturally hybrid by design.



Esperanto Technologies

Esperanto Technologies (founded 2014) takes an extreme RISC-V hybrid approach: its ET-SoC-1 chip (2022) packed 1,088 low-power RISC-V ET-Minion cores alongside four high-performance ET-Maxion RISC-V cores on a single die for ML inference. The massive parallelism of the minion array is directed by the maxion cores — a heterogeneous hybrid where the ISA is constant but microarchitectural diversity is extreme.



Tenstorrent

Tenstorrent (CEO Jim Keller, architect of AMD Zen and Apple A-series) develops AI accelerator chips using a RISC-V + custom Tensix processor architecture. The Grayskull and Wormhole chips use RISC-V cores to manage data movement and control flow, with Tensix cores executing tensor operations — an explicit hybrid ISA system where two distinct instruction sets divide the computation.

​



Compiler and OS-Level Considerations

Hybrid ISA architectures impose non-trivial burdens above the hardware level. As McKinsey's 2023 domain-specific architecture report observes, "higher-level compilers will need to account efficiently for the potential coexistence of multiple ISAs in a single package". Key challenges include:

​



Instruction selection and scheduling: The compiler must decide when to emit compressed vs. full-width instructions (RISC-V C extension, ARM Thumb-2), balancing code density against decode efficiency.



Thread affinity: OSes must correctly classify workloads for P-core vs. E-core placement. Incorrect scheduling can negate hybrid architecture benefits, as the TU Dresden Alder Lake energy paper demonstrated.

​



Binary translation fidelity: Subtle ISA semantic mismatches (floating-point rounding, exception ordering, memory model differences) are the primary source of correctness bugs in cross-ISA DBT.

​



Toolchain fragmentation: Each RISC-V custom extension requires custom compiler support, linker relocation types, and ABI extensions — a significant ecosystem cost for hybrid ISA designers.

​



Smaller and Experimental Research Threads

Heterogeneous ISA Frameworks (1993)

An early 1993 Computer magazine paper, "An Architectural Framework for Supporting Heterogeneous Instruction-Set Architectures," was ahead of its time in addressing how RISC, superscalar, and VLIW architectures could be integrated in a single system with hardware that optimises across instruction-set options. It represents the academic community grappling with the reality that no single ISA paradigm would dominate all workloads — the very premise of HISC.

​



CHERI Morello (ARM + Capability Hybrid)

Arm's Morello research chip (2022), developed with the University of Cambridge and UKRI funding, is the most advanced realisation of a CHERI capability ISA hybridised with a production ARM64 microarchitecture. It demonstrates that capability extensions can be added to a modern out-of-order superscalar pipeline with manageable area and performance overhead, making it a realistic candidate for future security-critical ISA hybridisation in product silicon.

​



Hyperdimensional Computing ISA Integration

As noted above, arXiv:2511.05053 (2025) demonstrates RISC-V ISA extension for HDC-CNN hybrid models. HDC's robustness to bit-level noise and its fixed-dimension binary vector operations make it architecturally very different from conventional FP32 neural networks — exploring how its specific computational primitives can be encoded as ISA extensions is an open and active research area.

​



ARM-RISC-V Comparative Embedded Analysis (TU Munich, 2024)

A February 2024 paper from TU Munich's Faculty of Information Systems provides a direct, holistic ISA comparison between ARM Cortex-M0+ (Thumb subset) and RISC-V RV32IMC for embedded systems. Notably, it found that while RISC-V can emulate all ARM Thumb instructions, it cannot always do so as efficiently — some Thumb instructions require multiple RISC-V instructions, highlighting that even within the hybrid ISA design space, coverage ≠ efficiency.

​



Future Directions

Chiplets and Multi-Die ISA Heterogeneity

The chiplet trend (disaggregated dies connected via high-speed interconnects like UCIe, TSMC CoWoS, or Intel EMIB) is pushing ISA hybridisation to the package level. A future SoC package may contain: a general-purpose CPU die (x86 or ARM64), a RISC-V-based security co-processor, an NPU die, a CXL-attached memory compute die with its own ISA, and a photonic I/O die — all presented to software as a coherent ISA through abstraction layers. Managing multi-ISA dispatch across chiplet boundaries is an open research problem.

​



AI-Driven and Adaptive ISAs

Several research groups are exploring ISA design automation using machine learning — generating candidate instruction set extensions from profiling data, evaluating them in simulation, and selecting those offering the best efficiency/performance trade-off for a target workload distribution. This points toward workload-adaptive hybrid ISAs that could reconfigure instruction decode semantics at runtime.



Quantum-Classical Hybrid ISAs

Quantum computing requires classical control processors to issue gate sequences, manage error correction, and process measurement outcomes. The interface ISA between a classical host CPU and a quantum processing unit (QPU) is an emerging and unsettled area. Research prototypes such as those from Delft, MIT, and Intel Labs are experimenting with instruction set extensions that allow a classical RISC-V or ARM core to dispatch quantum gate operations as if they were specialised instructions — the deepest possible form of hybrid ISC.



Near-Memory and In-Memory Computing

As memory bandwidth becomes the dominant bottleneck for AI and data analytics workloads, Processing-In-Memory (PIM) and Processing-Near-Memory (PNM) architectures are proliferating. Samsung's HBM-PIM and SK Hynix's AiM DRAM integrate simple RISC-like compute elements into memory dies, each with their own minimal ISA. The host processor must dispatch to these in-memory ISAs through specialised load/store extensions — yet another dimension of HISC. Domain-specific architecture research identifies at least 95 start-ups working on DSAs for AI, collectively raising over $10.6 billion.

​



Neuromorphic and Event-Driven ISAs

Intel's Loihi 2 neuromorphic chip and IBM's NorthPole architecture use spiking neural network computation models that bear no resemblance to conventional scalar or vector instruction sets. Hybridising these with conventional RISC or ARM cores — directing spike processing from conventional code — represents an architectural challenge being actively explored in both academia and industry.



Critical Synthesis: What Does the Research Show?

The weight of evidence across this literature supports several conclusions. First, ISA choice per se is a secondary factor in performance; microarchitecture and specialisation are primary. This means hybrid ISA strategies are most valuable when they enable power-efficiency trade-offs (big.LITTLE, P+E cores), code density improvements (Thumb, RISC-V C), security enforcement (CHERI), or domain-specific acceleration (AMX, custom RISC-V extensions) — not when they are used to chase raw FLOPS through instruction set design alone.



Second, the RISC-V extensibility model is arguably the most important current structural development in HISC. By providing a stable, royalty-free, formally specified base ISA with a well-governed extension mechanism, RISC-V allows research groups to build hybrid ISA systems without the ecosystem barrier that killed Itanium. This is catalysing an explosion of DSA-hybrid architectures in academia and industry.



Third, the software and tooling layer — compilers, OSes, binary translators — consistently emerges as the binding constraint on hybrid ISA adoption. Hardware innovation in hybrid ISC routinely outpaces the software ecosystem's ability to exploit it, and this gap remains the dominant practical challenge in the field.



Key Reference Works

For deeper reading, the following primary sources are foundational:



Patterson \& Hennessy, Computer Architecture: A Quantitative Approach, 6th ed. — The canonical textbook; Appendix K surveys ISA families

​



Blem, Menon, Sankaralingam (HPCA 2013), "Power Struggles: Revisiting the RISC vs. CISC Debate" — Empirical ISA-neutrality finding



Waterman (UC Berkeley PhD dissertation, 2016), Design of the RISC-V Instruction Set Architecture — Definitive RISC-V design rationale

​



Watson et al., CHERI ISA Specification v9 (Cambridge Technical Report, 2023) — The CHERI capability hybrid architecture

​



Intel, 12th Gen Intel Core Processors (Alder Lake) Optimization Reference Manual (2021) — P+E heterogeneous core design

​



arXiv:2511.05053 (2025), "Accelerating HDC-CNN Hybrid Models Using Custom Instructions on RISC-V" — Latest hybrid ISA extension research

​



arXiv:2502.05317 (2025), "Apple vs. Oranges: Evaluating Apple Silicon M-Series for HPC" — Modern heterogeneous SoC analysis

​



McKinsey Global Institute (2023), "Domain-Specific Architectures and the Future of Compute" — Industry landscape and DSA economics

​



X-HEEP platform (open source, 2024) — RISC-V hybrid ISA research infrastructure

​

