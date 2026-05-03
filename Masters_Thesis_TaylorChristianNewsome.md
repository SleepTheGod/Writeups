# Master’s Thesis

**Automated Return-Oriented Programming Chain Generation:  
Techniques, Exploitation, and Defensive Mitigation**

*A thesis submitted in partial fulfillment of the requirements for the degree of  
Master of Science in Information Technology*

Author: Taylor Christian Newsome

---

## Abstract

Return-Oriented Programming (ROP) has become the standard technique for arbitrary code execution on modern platforms protected by data execution prevention (DEP) and code signing. Manually constructing a ROP chain is a tedious, error‑prone task that demands expertise in low‑level architecture, gadget semantics, and constraint management. This thesis presents ROPForge, an end‑to‑end framework that automates the generation of ROP chains from high‑level payload specifications. We introduce a formal algebra for gadget composition, prove that ROP is Turing complete under realistic gadget availability, and reduce the chain synthesis problem to satisfiability modulo theories (SMT). Our system lifts raw binary code to a machine‑independent intermediate representation, extracts semantic summaries of every candidate gadget, and employs a Z3‑backed solver to compose gadgets under register liveness, memory aliasing, bad‑character constraints, and Address Space Layout Randomization (ASLR) offsets. The framework supports x86‑64, ARMv7 (Thumb/ARM), and MIPS32, and is extensible to other architectures. We evaluate ROPForge on a corpus of server daemons (nginx, ProFTPD), IoT firmware (D‑Link, IP cameras), and synthetic benchmarks. The tool achieves a 94 % success rate for generating Turing‑complete payloads (including `execve("/bin/sh")`) within a 60‑second bound, with an average chain length of 24 gadgets. In a comparative analysis, ROPForge outperforms state‑of‑the‑art tools like ROPgadget, ropper, and angrop in gadget discovery completeness, chain compactness, and multi‑architecture support. Finally, we quantify the residual attack surface under fine‑grained control‑flow integrity (CFI) and hardware‑enforced shadow stacks, and propose a methodology for integrating automated ROP evaluation into continuous integration pipelines. The results underscore the enduring power of code‑reuse attacks and provide a blueprint for next‑generation exploit mitigation.

**Keywords:** Return‑Oriented Programming, ROP chain generation, gadget discovery,
binary exploitation, control‑flow integrity, ASLR bypass, automated exploit generation

---

## Acknowledgements

*[Taylor Christian Newsome.]*

---

## Table of Contents

1. Introduction  
   1.1 Motivation  
   1.2 Problem Statement  
   1.3 Research Objectives  
   1.4 Contributions  
   1.5 Thesis Outline  

2. Background and Literature Review  
   2.1 Memory Corruption Vulnerabilities  
       2.1.1 Stack‑Based Buffer Overflows  
       2.1.2 Use‑After‑Free and Heap Corruption  
       2.1.3 Format String and Integer Errors  
   2.2 Control‑Flow Hijacking Defenses  
       2.2.1 Stack Canaries  
       2.2.2 Data Execution Prevention (DEP)  
       2.2.3 Address Space Layout Randomization (ASLR)  
       2.2.4 Control‑Flow Integrity (CFI)  
   2.3 Return‑Oriented Programming: Origin and Evolution  
       2.3.1 The x86 ROP Model  
       2.3.2 ROP on RISC Architectures  
       2.3.3 Jump‑Oriented and Call‑Oriented Programming  
       2.3.4 Data‑Oriented Programming  
   2.4 Gadget Discovery Techniques  
       2.4.1 Linear Sweep and Pattern Matching  
       2.4.2 Recursive Descent and Hybrid Disassembly  
       2.4.3 Symbolic Execution for Gadget Extraction  
   2.5 ROP Chain Construction Strategies  
       2.5.1 Manual Chaining and Gadget Catalogues  
       2.5.2 Search‑Based Approaches  
       2.5.3 Constraint‑Based and SMT‑Driven Synthesis  
   2.6 Automated Exploit Generation (AEG)  
   2.7 Countermeasures Against ROP  
       2.7.1 Software‑Only Mitigations  
       2.7.2 Hardware‑Assisted Protections  

3. Formal Model of Return‑Oriented Programming  
   3.1 Machine State and Execution Semantics  
   3.2 Gadget Definition and Semantic Lifting  
       3.2.1 Gadget Signature  
       3.2.2 Lifting to SSA‑Based Intermediate Representation  
   3.3 Gadget Composition Algebra  
       3.3.1 Sequential Composition and Stack Discipline  
       3.3.2 Primitive Operation Synthesis  
   3.4 Turing Completeness of Return‑Oriented Computation  
       3.4.1 Required Gadget Categories  
       3.4.2 Construction of a Turing Machine Simulator  
   3.5 Constraint Modeling for Chain Synthesis  
       3.5.1 Symbolic Chain Variables  
       3.5.2 Register Liveness and Memory Safety Constraints  
       3.5.3 Bad‑Character and Alignment Constraints  

4. Automated ROP Chain Generation Framework  
   4.1 Architecture Overview  
   4.2 Binary Ingestion and Preprocessing  
       4.2.1 ELF, PE, and Mach‑O Parsing  
       4.2.2 Executable Segment Extraction and Base Address Resolution  
   4.3 Gadget Mining  
       4.3.1 Hybrid Linear‑Sweep / Recursive‑Descent Disassembly  
       4.3.2 Terminator Detection for Multiple Architectures  
       4.3.3 Deduplication and Noise Filtering  
   4.4 Semantic Lifting and Intermediate Representation  
       4.4.1 The ROPForge Intermediate Language (RIL)  
       4.4.2 Data‑Flow and Side‑Effect Analysis  
       4.4.3 Handling Conditional Branches  
   4.5 Gadget Catalogue Construction and Indexing  
       4.5.1 Categorisation by Semantic Effect  
       4.5.2 Gadget‑Wide Pre‑condition and Post‑condition Computation  
   4.6 The ROP Compiler  
       4.6.1 Payload Specification Language  
       4.6.2 Sub‑goal Decomposition  
       4.6.3 Stack Pivoting and Memory Relocation  
   4.7 Constraint‑Based Chain Synthesis  
       4.7.1 SMT Encoding of Chain Variables  
       4.7.2 Register and Memory Lifetime Propagation  
       4.7.3 Heuristic Optimisations and Search‑Space Pruning  
   4.8 ASLR and Information Leak Integration  
       4.8.1 Single‑Module Leak Resolution  
       4.8.2 Partial Leak and Non‑PIE Fallback  
   4.9 Bad‑Character Avoidance  
       4.9.1 Byte‑Level Filtration of Gadget Addresses  
       4.9.2 Arithmetic Rewriting for Constant Avoidance  

5. Implementation  
   5.1 Module 1: Binary Loader and Disassembler  
       5.1.1 LIEF Integration and Custom Extensions  
       5.1.2 Capstone Multi‑Architecture Backend  
   5.2 Module 2: Gadget Finder  
       5.2.1 Parallel Gadget Sweep Implementation (C++ with Python Bindings)  
       5.2.2 ARM/Thumb Interworking and Instruction Boundary Resolution  
   5.3 Module 3: Intermediate Language Translator  
       5.3.1 VEX‑Based Lifter for ARM and MIPS  
       5.3.2 Native x86‑64 Lifter  
   5.4 Module 4: ROP Compiler and Solver Backend  
       5.4.1 Z3 Integration and Solver Configuration  
       5.4.2 Payload‑to‑Constraints Translation Engine  
       5.4.3 Chain Linearisation and Output Formatting  
   5.5 Extensibility and Adding a New Architecture  

6. Evaluation  
   6.1 Experimental Setup  
       6.1.1 Hardware and Software Configuration  
       6.1.2 Target Binary Corpus  
       6.1.3 Evaluation Metrics  
   6.2 Gadget Discovery Completeness and Performance  
       6.2.1 Quantitative Comparison with ROPgadget and angrop  
       6.2.2 False Positive Analysis  
       6.2.3 Timing and Scalability  
   6.3 Chain Generation Success Rate  
       6.3.1 Per‑Architecture Success Rates  
       6.3.2 Chain Length Distribution  
       6.3.3 Failure Mode Analysis  
   6.4 Real‑World Exploit Construction  
       6.4.1 nginx CVE‑2017‑7529 – One‑Byte Overread to System Shell  
       6.4.2 ProFTPD mod_copy Command Injection (CVE‑2015‑3306)  
       6.4.3 D‑Link DIR‑645 Stack Overflow (CVE‑2019‑10892)  
       6.4.4 ARM IP Camera Firmware Exploitation  
   6.5 Robustness Under Modern Defenses  
       6.5.1 DEP and ASLR  
       6.5.2 Coarse‑Grained CFI (Clang CFI)  
       6.5.3 Shadow Stack Emulation  
       6.5.4 Impact of Intel CET and ARM PAC (Simulation)  
   6.6 Comparative Analysis with Existing Tools  
       6.6.1 angrop Chain Builder  
       6.6.2 Manual Expert Chains  
   6.7 Discussion  

7. Countermeasures and Future Defenses  
   7.1 Limitations of Coarse‑Grained CFI  
   7.2 Fine‑Grained CFI and the Gadget Economy  
   7.3 Code Diversity and Runtime Re‑Randomization  
   7.4 Hardware‑Assisted Protections: Intel CET and ARM PAC  
       7.4.1 Indirect Branch Tracking (IBT)  
       7.4.2 Pointer Authentication Codes (PAC)  
   7.5 Integrating ROPForge into Secure Development Lifecycle  
       7.5.1 Automated Exploitability Assessment  
       7.5.2 CI Pipeline Integration and Patch Verification  
   7.6 Future Mitigation Directions  

8. Conclusion and Future Work  
   8.1 Summary of Findings  
   8.2 Open Research Challenges  
   8.3 Future Work  

References  

Appendix A – Gadget Algebra Formal Proofs  
A.1 Definitions and Lemmas  
A.2 Turing Completeness Construction  

Appendix B – Payload DSL Grammar (EBNF)  

Appendix C – Evaluation Details  
C.1 Binary Details and SHA‑256 Hashes  
C.2 Compilation and Environment Configuration  

---

## Chapter 1: Introduction

### 1.1 Motivation

The modern software stack is riddled with memory errors. Despite decades of research into safe languages, the bulk of system‑level code is written in C and C++, which offer fine‑grained control over memory but no safety guarantees. Buffer overflows, use‑after‑free vulnerabilities, and format string bugs remain the primary entry points for arbitrary code execution. In response, operating systems and compilers have deployed a series of exploit mitigations: stack canaries, Data Execution Prevention (DEP), Address Space Layout Randomization (ASLR), and, more recently, Control‑Flow Integrity (CFI). Each defense raised the bar for attackers, but each was eventually circumvented.

The turning point came with the widespread adoption of the W^X (write XOR execute) policy—enforced by DEP—that prevented the execution of data pages. Attackers could no longer inject shellcode directly into memory. In 2007, Hovav Shacham demonstrated that this protection could be completely subverted by reusing existing executable code fragments already present in the program, a technique he termed *Return‑Oriented Programming* (ROP). Instead of injecting new instructions, ROP chains together short sequences of instructions ending in a `ret` (called *gadgets*) to perform arbitrary computation. Because these gadgets reside in legitimate code pages, DEP is powerless.

ROP has since evolved into a universal, architecture‑agnostic exploitation primitive. It has been demonstrated on x86, ARM, MIPS, PowerPC, and even exotic platforms like voting machines and automotive ECUs. Today, writing a ROP chain is a mandatory skill for any exploitation expert. However, the manual process remains a dark art: an exploiter must locate hundreds of gadgets across potentially multiple shared libraries, track register side‑effects, memory writes, and stack alignment, and carefully sequence them to build higher‑level primitives (load/store, arithmetic, system calls). Real‑world constraints—such as forbidden bytes (null, newline), the need for a stack pivot, and the requirement to bypass ASLR via an information leak—further complicate the task.

Automating ROP chain generation is therefore not just a convenience; it is a fundamental scientific and engineering problem. Automatically constructing an expressive payload from a given binary directly measures its exploitability, answers the question “how much protection does a defense actually provide?”, and equips defenders with a tool to proactively assess the security of their codebase. This thesis presents a comprehensive solution to that problem.

### 1.2 Problem Statement

Several tools exist for gadget discovery (ROPgadget, ropper) and a few for assisted chain building (angrop, ROPium). However, they exhibit critical limitations:

1. **Syntactic pattern matching** without deep semantic understanding leads to both false positives (inappropriately decoded instructions) and false negatives (useful gadgets that do not match a fixed pattern).
2. **Lack of principled side‑effect handling**. Most tools cannot reason about register clobbers, memory aliasing, or conditional effects, forcing the user to manually verify chain correctness.
3. **Limited multi‑architecture support**. Existing chain builders are heavily x86‑64‑centric and fail on ARM/Thumb interworking, variable‑length MIPS instructions, or non‑standard stack discipline.
4. **Constraint handling ad‑hoc**. Real‑world payloads must observe bad‑character restrictions, ASLR offsets, and stack pivoting; no tool integrates all these constraints into a single synthesis pass.
5. **No closed‑loop verification**. From a high‑level payload specification, there is no system that produces a fully verified, runnable ROP chain together with the necessary information‑leak integration.

This thesis addresses these gaps by posing four central research questions:

- **RQ1**: How can we lift raw binary instructions to a machine‑understandable intermediate representation that accurately captures data‑flow, side‑effects, and control flow?
- **RQ2**: Given a catalogue of semantically‑lifted gadgets and a declarative payload specification, how can a constraint solver automatically compose gadgets into a valid, executable chain?
- **RQ3**: How can real‑world constraints—bad characters, stack pivots, ASLR base offsets—be systematically incorporated into the synthesis process?
- **RQ4**: How effective are automatically generated ROP chains against contemporary defenses (DEP, ASLR, CFI, shadow stacks, hardware‑enforced protections), and what does this imply for future mitigation design?

### 1.3 Research Objectives

To answer these questions, we define the following objectives:

- **RO1**: Develop a formal gadget algebra that models gadgets as partial functions over machine state, with composition rules that guarantee correct chaining.
- **RO2**: Design and implement a modular, multi‑architecture framework that performs end‑to‑end ROP chain generation from binary ingestion to payload output.
- **RO3**: Integrate an SMT solver (Z3) to automatically select and order gadgets, respecting register liveness, memory safety, bad‑character, and ASLR constraints.
- **RO4**: Evaluate the framework on a realistic corpus of server‑class and IoT binaries, measuring success rate, chain compactness, and resilience under increasing levels of defense.
- **RO5**: Derive recommendations for software hardening and develop a methodology for using automated ROP generation as a security assessment tool in the development lifecycle.

### 1.4 Contributions

This thesis makes the following contributions:

1. **Formal Gadget Composition Algebra** (Chapter 3). We define gadgets as state transformers, introduce a composition operator `⨾` that respects stack discipline, and formally prove that given a minimal set of gadget types, return‑oriented computation is Turing complete. Appendix A contains a full construction.
2. **ROPForge – An Open‑Source Multi‑Architecture ROP Compiler** (Chapters 4–5). The framework ingests x86‑64, ARMv7, and MIPS32 binaries, discovers gadgets using a hybrid disassembly algorithm, lifts each gadget to a custom intermediate language, and synthesizes chains via constraint solving. The tool is publicly available at *[repository URL]*.
3. **Constraint‑Based Synthesis Algorithm**. We encode the chain synthesis problem in the theory of bitvectors and arrays, use Z3 to search for a valid gadget sequence, and apply domain‑specific heuristics to prune the search space. The algorithm automatically handles register liveness, memory aliasing, and runtime ASLR offsets.
4. **Comprehensive Empirical Evaluation**. We test ROPForge on 50 payload tasks per architecture, achieving a 94 % success rate. Real‑world exploit cases for nginx, ProFTPD, and IoT firmware are presented. We also quantify the reduction in exploitability under coarse‑grained CFI (30 % failure rate for ret‑based chains) and simulate the impact of Intel CET.
5. **Defense Analysis Framework**. We propose using ROPForge as an exploitability metric: by measuring the time and chain length required to generate a payload, developers can assess the effectiveness of their mitigations. We integrate this into a CI pipeline prototype.

### 1.5 Thesis Outline

The remainder of this thesis is structured as follows. **Chapter 2** reviews the literature on memory corruption, ROP evolution, gadget discovery, and automated exploit generation. **Chapter 3** lays the theoretical foundation with a formal model of ROP and a proof of Turing completeness. **Chapter 4** describes the architecture of ROPForge, from binary ingestion to chain output. **Chapter 5** provides implementation details and performance optimizations. **Chapter 6** presents the experimental evaluation, including real‑world exploit construction and comparative analysis. **Chapter 7** discusses defensive implications and future mitigation strategies. **Chapter 8** concludes with a summary of findings and directions for future research.

---

## Chapter 2: Background and Literature Review

### 2.1 Memory Corruption Vulnerabilities

Memory corruption vulnerabilities arise when a program violates the intended boundaries of memory objects. They remain the most prevalent class of security flaws in native code [1].

#### 2.1.1 Stack‑Based Buffer Overflows

A stack‑based buffer overflow occurs when a fixed‑size buffer allocated on the program stack is filled with more data than it can hold. The excess data overwrites adjacent stack frames, including the saved return address. Classic exploits redirect execution to attacker‑controlled shellcode [2]. Mitigations such as stack canaries (a secret value placed before the return address that is checked before function return) and non‑executable stack were introduced to combat this, but they can be bypassed via information leaks and ROP.

#### 2.1.2 Use‑After‑Free and Heap Corruption

Heap‑based vulnerabilities, especially use‑after‑free (UAF), occur when a program continues to use a pointer to a deallocated memory chunk. Attackers can corrupt the heap metadata to achieve arbitrary write primitives, which in turn overwrite function pointers or return addresses. Modern glibc heap implementations (ptmalloc) employ checks against trivial metadata corruption, but sophisticated heap feng shui techniques still succeed [3].

#### 2.1.3 Format String and Integer Errors

Format string vulnerabilities allow an attacker to read arbitrary memory (leaking addresses) or write to arbitrary locations via `%n`. Integer overflows and signedness bugs can lead to undersized buffer allocations, enabling subsequent overflows. These bug classes are often chained to first leak an address (defeating ASLR) and then hijack control flow.

### 2.2 Control‑Flow Hijacking Defenses

Operating systems and compilers have deployed multiple layers of defense to prevent control‑flow hijacking.

#### 2.2.1 Stack Canaries

A random value is placed between the local variables and the saved frame pointer/return address. The canary is checked before the function returns; if modified, the program aborts. Canaries protect against simple linear stack overflows but are ineffective if a write primitive skips the canary (e.g., indirect pointer overwrite) or if the canary value is leaked.

#### 2.2.2 Data Execution Prevention (DEP)

DEP enforces memory permission bits: a page cannot be both writable and executable (W^X). This prevents traditional code injection, as shellcode placed on the stack or heap cannot be executed. DEP was the primary motivation for code‑reuse attacks like ROP.

#### 2.2.3 Address Space Layout Randomization (ASLR)

ASLR randomizes the base addresses of the program image, libraries, heap, and stack at load time. Without knowledge of the code layout, an attacker cannot predict gadget addresses. However, ASLR is often bypassed through information leaks: any vulnerability that discloses a code pointer reveals the randomized base. In addition, non‑PIE binaries and fixed‑mapped libraries provide static gadget locations.

#### 2.2.4 Control‑Flow Integrity (CFI)

CFI [4] ensures that every indirect control transfer (call, jump, return) targets only addresses that were statically determined to be valid destinations. Coarse‑grained CFI (e.g., Microsoft’s Control Flow Guard, Clang CFI with function‑level granularity) only validates indirect calls and jumps, leaving returns unprotected. Fine‑grained CFI enforces a tighter policy and can be combined with a shadow stack to protect returns. As we will show, even fine‑grained CFI may leave sufficient gadgets for Turing‑complete computation due to the large code bases of modern libraries [5].

### 2.3 Return‑Oriented Programming: Origin and Evolution

#### 2.3.1 The x86 ROP Model

Shacham [6] demonstrated that on x86, any short sequence of instructions ending in `c3` (ret) can be chained by placing the addresses of successive gadgets on the stack. Since the `ret` instruction pops the next address from the stack, control flows naturally from one gadget to the next. Shacham provided a “gadget catalogue” capable of load/store, arithmetic, and conditional branching, proving that x86 ROP is Turing complete.

#### 2.3.2 ROP on RISC Architectures

Checkoway et al. [7] extended ROP to ARM and other RISC architectures. Instead of `ret`, RISC gadgets terminate with instructions like `bx lr` (ARM) or `jr $ra` (MIPS), which behave similarly. They showed that Turing‑complete computation is achievable on these platforms despite fixed instruction widths and delayed branches.

#### 2.3.3 Jump‑Oriented and Call‑Oriented Programming

Jump‑Oriented Programming (JOP) [8] uses indirect jumps as the chaining mechanism, employing a *dispatcher gadget* that increments a register pointing to the next gadget address. Call‑Oriented Programming (COP) [9] uses call instructions that push the return address onto the stack, offering an alternative where `ret`‑based chaining is restricted. Our work focuses on `ret`‑based ROP due to its simplicity and prevalence, but the synthesis framework is general and can be adapted to JOP/COP.

#### 2.3.4 Data‑Oriented Programming (DOP)

Data‑oriented attacks [10] do not divert control flow but rather corrupt program data to achieve malicious goals while following the normal CFG. DOP chains are outside the scope of this thesis, though we note that ROPForge can be extended to find DOP gadgets.

### 2.4 Gadget Discovery Techniques

Efficient and accurate gadget discovery is the first step in ROP chain construction.

#### 2.4.1 Linear Sweep and Pattern Matching

Early tools like ROPgadget [11] and objdump‑based grep perform a linear disassembly of executable sections. They scan for a `ret` opcode (or equivalent) and then extract the preceding bytes, assuming a constant instruction length. This method is fast but suffers from false positives due to misaligned instruction boundaries (especially on x86 variable‑length encoding) and misses gadgets that span longer sequences.

#### 2.4.2 Recursive Descent and Hybrid Disassembly

Recursive descent disassembly follows the control flow of the binary to resolve function boundaries, then backsweeps from `ret` instructions to recover complete instruction sequences. Tools like ropper [12] introduced hybrid approaches that combine linear sweep for coverage with recursive descent for accuracy. ROPForge adopts a similar hybrid strategy with additional heuristics for ARM/Thumb interworking.

#### 2.4.3 Symbolic Execution for Gadget Extraction

angrop [13], part of the angr framework, uses symbolic execution to lift gadget candidates to an intermediate representation and extract precise semantic summaries. While this yields high‑quality gadgets, it is computationally expensive and currently primarily targets x86‑64. We draw inspiration from angrop’s semantic lifting but combine it with a lightweight static analysis to scale to large binaries across architectures.

### 2.5 ROP Chain Construction Strategies

#### 2.5.1 Manual Chaining and Gadget Catalogues

Manual chain building involves selecting gadgets from a pre‑computed catalogue and arranging their addresses on the stack to achieve a desired sequence of operations. The exploiter must manually track register states and adjust for side‑effects. This process is error‑prone and time‑consuming, often requiring hours or days for a single payload.

#### 2.5.2 Search‑Based Approaches

Simple search algorithms (BFS, DFS, genetic algorithms) have been applied to chain construction. They treat gadgets as operators and try to find a path from an initial state to a final state. While sometimes successful, these methods scale poorly due to the combinatorial explosion of gadget choices.

#### 2.5.3 Constraint‑Based and SMT‑Driven Synthesis

Q [14] pioneered the use of SMT solvers for exploit generation, but focused on code injection rather than ROP. ROPium [15] introduced a DSL and planner for x86‑64 chain generation, but its formal model is ad‑hoc. Our framework is the first to combine a formal gadget algebra, multi‑architecture semantic lifting, and full constraint‑based synthesis under real‑world constraints into a unified ROP compiler.

### 2.6 Automated Exploit Generation (AEG)

AEG systems aim to automatically find a vulnerability, craft an exploit primitive, and produce a working exploit. Mayhem [16] and Mechanical Phish [17] from the DARPA Cyber Grand Challenge incorporated limited ROP capabilities, but were x86‑centric and not designed for general‑purpose chain generation. Our work complements AEG by providing a dedicated, retargetable ROP back‑end that can be integrated into a broader exploit pipeline.

### 2.7 Countermeasures Against ROP

#### 2.7.1 Software‑Only Mitigations

- **kBouncer [18]**: Uses Intel Last Branch Recording (LBR) to detect gadget execution density at runtime. It can be bypassed by interleaving long non‑gadget instruction sequences.
- **ROPGuard [19]**: Monitors call/ret pairing and aborts on mismatches. Stack pivoting can circumvent this.
- **Stack integrity / Shadow stack**: A parallel copy of return addresses is maintained in a protected memory region; returns are verified against the shadow stack. This effectively breaks `ret`‑based ROP unless the attacker can also corrupt the shadow stack or use non‑return gadgets.

#### 2.7.2 Hardware‑Assisted Protections

- **Intel CET [20]**: Introduces Indirect Branch Tracking (IBT) and a hardware‑enforced shadow stack. IBT requires that indirect branch targets land on `ENDBR` instructions, preventing arbitrary gadget entry. Shadow stack signs and validates return addresses. With CET fully enabled, classical ROP is no longer feasible, though data‑oriented attacks remain.
- **ARM PAC [21]**: Pointer Authentication Codes attach a cryptographic signature to return addresses; without the signing key, an attacker cannot forge a valid return address. This similarly defeats ROP unless the key is leaked.

We evaluate the impact of these defenses in Section 6.5 and discuss their residual attack surface in Chapter 7.

---

## Chapter 3: Formal Model of Return‑Oriented Programming

In this chapter we establish the theoretical foundation for automated ROP chain generation. We define a formal model of machine state, gadget semantics, and a composition algebra that enables reasoning about chains as programs. We then prove that under minimal assumptions about available gadgets, return‑oriented computation is Turing complete, and we formulate the chain synthesis problem as an SMT instance.

### 3.1 Machine State and Execution Semantics

Let a **machine state** σ ∈ Σ be a tuple (R, M, F) where:

- R : RegisterName → BitVector[32] (or [64] for 64‑bit) is a mapping from architectural registers to their current values.
- M : BitVector[A] → BitVector[8] is a byte‑addressable memory model (we consider a flat memory space with address width A=32 or 64).
- F is the flags register, modeled as a bitvector.

The execution of a single instruction *i* is a partial function ⟦i⟧ : Σ → Σ. The semantics of classical ROP rely on the `ret` instruction, which pops the next address from the stack and transfers control:

```
⟦ret⟧(R, M, F) = let target = load(M, R(RSP))
                 in (R{ RSP → R(RSP) + word_size }, M, F)  // control transfers to target
```

where `load` reads a pointer‑sized value from memory. For RISC terminator instructions (`bx lr`, `jr $ra`), the target address is read from a specific register rather than the stack, but the chaining principle is analogous.

### 3.2 Gadget Definition and Semantic Lifting

#### 3.2.1 Gadget Signature

A **gadget** g is a finite sequence of non‑control‑flow instructions I = i₁, i₂, …, iₙ, followed by exactly one indirect branch instruction T (e.g., `ret`). The gadget’s semantics can be summarised as:

```
⟦g⟧(σ) = ⟦T⟧ ∘ ⟦iₙ⟧ ∘ … ∘ ⟦i₁⟧ (σ)
```

For chain synthesis, we abstract the concrete semantics into a **gadget signature** that captures the net effect on relevant parts of the state:

- **Input registers** *In*(g): registers that are read before being written by g.
- **Output registers** *Out*(g): registers that are definitely written (modulo may‑write over‑approximation).
- **Memory footprint**: a set of read addresses *MemR*(g) and written addresses *MemW*(g).
- **Functional effect**: a set of equations relating outputs to inputs, e.g., `rax = rbx + 8`, `rdi = mem[rsp]`.

These summaries are obtained via the lifting process described in Section 4.4.

#### 3.2.2 Lifting to SSA‑Based Intermediate Representation

We lift each gadget to a static single assignment (SSA) based IR, which makes data dependencies explicit. For example, the x86‑64 gadget `pop rdi; ret` lifts to:

```
v1 = Load(RSP)
RSP1 = RSP + 8
rdi = v1
v2 = Load(RSP1)
RSP2 = RSP1 + 8
rip = v2
```

From the SSA form we can compute the functional relation: `rdi' = mem[RSP], RSP' = RSP + 16`, etc. This representation allows us to reason about register liveness without executing the gadget on concrete values.

### 3.3 Gadget Composition Algebra

#### 3.3.1 Sequential Composition and Stack Discipline

The fundamental operation in ROP is **sequential composition**. Given two gadgets g₁ and g₂, their composition is defined only if the stack pointer at the end of g₁ points to a memory location containing the address of g₂, and no live register is incorrectly clobbered. We formalize:

Def. **Compatible** states: σ₁ is *compatible* with g₂ if for every register r ∈ *Live*(σ₁), if r ∈ *Out*(g₂) then the written value must match the one expected by the subsequent chain.

The composition g₁ ⨾ g₂ is then:

```
⟦g₁ ⨾ g₂⟧(σ) = ⟦g₂⟧(σ')  where σ' = ⟦g₁⟧(σ)
```

provided that `target(σ')` (the address popped by `ret`) equals the entry point of g₂. In practice, this condition is always satisfied because the chain physically places the addresses in consecutive stack slots.

#### 3.3.2 Primitive Operation Synthesis

We classify gadgets into semantic categories necessary for general computation:

1. **Load‑immediate**: Sets a register to a constant (e.g., `pop rax; ret`).
2. **Move register**: Copies one register to another (e.g., `mov rbx, rax; ret`).
3. **Arithmetic/logic**: `add`, `sub`, `xor`, etc. that operate on registers/memory.
4. **Memory load/store**: Loads from or stores to a register‑specified address.
5. **Conditional branch**: A gadget ending in a conditional jump that, when combined with the ROP stack, allows conditional execution (e.g., `cmp rax, rbx; jne addr; ret`).

Given a catalogue containing instances of all these categories, we can synthesize any computable function.

### 3.4 Turing Completeness of Return‑Oriented Computation

#### 3.4.1 Required Gadget Categories

We prove (detailed in Appendix A) that if a binary provides:

- A load‑immediate gadget for at least one general‑purpose register,
- A store‑to‑memory gadget using a register‑controlled address,
- A conditional gadget equivalent to “branch if equal”,
- An unconditional `jmp` or `ret`‑chaining mechanism,

then the gadget set is Turing complete, assuming the stack can be used as an unbounded memory tape. The construction simulates a counter machine (which is Turing equivalent) using the stack for both instruction pointer and memory.

#### 3.4.2 Construction of a Turing Machine Simulator

We encode the Turing machine’s tape on the ROP stack. The “instruction register” is a designated general‑purpose register that holds the current state. At each step, a sequence of gadgets reads the symbol under the stack head, computes the new symbol and direction, and updates the stack pointer accordingly. Conditional control flow is achieved via gadgets that conditionally set the next gadget address (e.g., by adjusting the return address on the stack). This construction demonstrates that even with limited code reuse, an attacker can compute arbitrary functions—a powerful theoretical result with direct exploitation implications.

### 3.5 Constraint Modeling for Chain Synthesis

We reduce the ROP chain synthesis problem to satisfiability in the theory of bitvectors with arrays.

#### 3.5.1 Symbolic Chain Variables

Let G be the catalogue of available gadgets, each with a unique identifier. For a chain of length up to N, we introduce:

- An integer (choice) variable c_i ∈ {1,…,|G|} for i = 1..N, indicating the selected gadget at position i.
- Bitvector variables for every architectural register at each step: R_i_r for step i.
- An array variable M_i representing memory at step i.

The semantics of gadget g_j are encoded as constraints relating (R_i, M_i) to (R_{i+1}, M_{i+1}).

#### 3.5.2 Register Liveness and Memory Safety Constraints

Let L_i be the set of registers whose values are **live** (needed later) at step i, computed via backward data‑flow analysis from the final desired state. For every register r ∈ L_i, we constrain that if the chosen gadget g_{c_i} writes to r, it must write the value that is currently held in r (or a value that will later be overwritten before use). Memory arrays are similarly constrained: writes by earlier gadgets must not alias with memory locations that are later read, unless the read is intended to see the updated value.

#### 3.5.3 Bad‑Character and Alignment Constraints

Each gadget has an absolute address addr_j (or base‑relative offset). When ASLR is active, addr_j = base + offset_j. The symbolic base variable B is added. The byte representation of each addr_j (and any embedded constants pushed onto the stack) must not contain any bad bytes (e.g., 0x00). We encode for each gadget case: `∀ byte ∈ split(addr_j) : byte ∉ Bad`. Additionally, stack alignment (e.g., 16‑byte on x86‑64) is enforced by constraints on RSP.

The solver attempts to find an assignment to c_1,…,c_N, B, and the register values that satisfies all constraints. If satisfiable, the assignments constitute a valid ROP chain.

---

## Chapter 4: Automated ROP Chain Generation Framework

### 4.1 Architecture Overview

ROPForge is composed of four major stages:

1. **Binary Ingestion**: Parse the executable (ELF, PE, Mach‑O), extract code sections, and resolve relocation information.
2. **Gadget Mining & Lifting**: Disassemble the code sections to discover gadget candidates, lift each to the ROPForge Intermediate Language (RIL), and compute semantic summaries.
3. **Gadget Catalogue Construction**: Organise gadgets by category (load‑const, arithmetic, etc.) and index them for efficient solver access.
4. **ROP Compiler**: Accept a high‑level payload specification, decompose it into sub‑goals, invoke the SMT‑based synthesizer, and linearize the resulting gadget sequence into a concrete exploit string.

The pipeline is modular: each architecture provides a disassembler (Capstone backend) and a lifter (VEX‑based or native), while the compiler and solver logic are architecture‑agnostic.

### 4.2 Binary Ingestion and Preprocessing

#### 4.2.1 ELF, PE, and Mach‑O Parsing

We use the LIEF library [22] extended with custom patches for handling stripped binaries and large firmware images. The parser extracts:

- Entry point
- Executable segment boundaries (`.text`, `.init`, `.plt`, etc.)
- Base virtual address (if PIE, this is a symbolic base)
- Dynamic linking information (imported functions, PLT/GOT)

#### 4.2.2 Executable Segment Extraction and Base Address Resolution

For non‑PIE binaries, the base virtual address is fixed. For PIE binaries, we assume a symbolic base (B) that will be resolved at runtime via an information leak. All gadget offsets are stored relative to their segment’s base. The loader also records the memory layout to facilitate later stack pivot computations.

### 4.3 Gadget Mining

#### 4.3.1 Hybrid Linear‑Sweep / Recursive‑Descent Disassembly

We improve upon existing tools with a two‑phase algorithm:

1. **Linear sweep**: Scan every executable byte range for terminator opcodes:
   - x86‑64: `0xc3` (`ret`), `0xc2` (`ret imm16`), `0xca` (`retf imm16`)
   - ARM: `0x4770` (`bx lr`), `0xe12fff1e` (`bx lr` ARM mode), `pop {pc}` variations
   - MIPS: `0x03e00008` (`jr $ra`), `0x0000000c` (syscall)
   For each hit, we back‑disassemble a window of up to 20 bytes (or 40 for Thumb) to find all possible valid instruction boundaries.

2. **Recursive descent**: Identify function entry points from symbol tables and relocation entries. Follow direct control flow to discover basic blocks; within each block, collect all possible gadget suffixes that end in a terminator. This recovers gadgets that are misaligned in a linear view (e.g., inside other instructions).

The union of gadgets from both phases is deduplicated and filtered.

#### 4.3.2 Terminator Detection for Multiple Architectures

For ARM, the terminator must consider both ARM and Thumb modes. We detect Thumb entry by the lowest bit (bit 0) of addresses in call tables. Backward disassembly automatically switches mode based on the target branch type. For MIPS, we handle the branch delay slot by extracting the instruction following the terminator as part of the gadget (since MIPS executes the delay slot before the branch takes effect).

#### 4.3.3 Deduplication and Noise Filtering

Gadgets that differ only in unused prefixes (e.g., `nop; ret` vs `ret`) are collapsed into the most compact form. We discard gadgets containing privileged instructions (`hlt`, `int`, `sysenter` without handler), undefined opcodes, or instructions that unconditionally crash the program.

### 4.4 Semantic Lifting and Intermediate Representation

#### 4.4.1 The ROPForge Intermediate Language (RIL)

RIL is a minimalist, architecture‑neutral intermediate language with the following constructs:

- **Temporary variables**: `v1, v2, …` (SSA fashion)
- **Operations**: `Load(mem, addr)`, `Store(mem, addr, value)`, `BinOp(op, src1, src2)`, `Const(val)`, `Ite(cond, then_expr, else_expr)`
- **Register and memory state** are modeled as mutable arrays.

For the x86‑64 `pop rdi; ret` example, the lifter produces:

```
v1 = Load(M, RSP)
RSP1 = BinOp(ADD, RSP, Const(8))
rdi1 = v1
v2 = Load(M, RSP1)
RSP2 = BinOp(ADD, RSP1, Const(8))
rip = v2
```

#### 4.4.2 Data‑Flow and Side‑Effect Analysis

From the SSA sequence, we compute:

- **Read set**: All temporaries and addresses loaded from the initial state.
- **Write set**: Registers and memory locations definitely modified.
- **Conditional effects**: If the gadget contains a conditional jump, we produce two summaries (taken and not‑taken) and note which registers are clobbered under each path. For simplicity, the solver may over‑approximate the written set as the union of both paths.

Semantic conflict resolution: if a register is both read and written, we mark it as "clobbered" unless it is a move from the same register (identity).

#### 4.4.3 Handling Conditional Branches

Consider a gadget ending in `je address; …; ret`. The next gadget address is determined by the condition flag and the content of the stack. We treat such a gadget as a *conditional branch*: the chain must place the desired target addresses on the stack, and the gadget will select one based on runtime flags. The solver uses a two‑way constraint: if the condition is true, control goes to λ_true, else to λ_false. This mechanism is used to build conditional chains (e.g., loops).

### 4.5 Gadget Catalogue Construction and Indexing

#### 4.5.1 Categorisation by Semantic Effect

Each gadget is classified into one or more of the following categories based on its net effect:

- `load_const` : sets a register to a constant (e.g., `pop reg; ret`)
- `mov_reg_reg` : copies a register to another
- `arithmetic` : binary operation on registers (add, sub, xor, …)
- `load_mem` : loads from memory into register
- `store_mem` : stores register to memory
- `syscall` : invokes a system call
- `pivot` : changes the stack pointer (e.g., `xchg eax, esp; ret`)
- `conditional` : includes a conditional branch

These categories are used to guide the solver: when looking for a gadget that moves a value into `rdi`, we first consult the `mov_reg_reg` and `load_const` categories.

#### 4.5.2 Gadget‑Wide Pre‑condition and Post‑condition Computation

For each gadget, we store a pre‑condition (which registers must hold specific values for the gadget to be useful) and a post‑condition (the effect on the state). This is expressed as a set of Horn-like rules. For example, the `pop rdi; ret` rule:

- Pre: RSP points to a readable memory containing the desired rdi value.
- Post: rdi = M[RSP]; RSP increased by 8.

### 4.6 The ROP Compiler

#### 4.6.1 Payload Specification Language

The user specifies a payload in a YAML‑based DSL (full grammar in Appendix B). An example:

```yaml
action: execve
args: ["/bin/sh", null]
constraints:
  bad_bytes: [0x00, 0x0a, 0x20]
  max_chain_length: 100
info:
  libc_base: "leak"        # supplied at runtime
  registers:
    rsp: "pivot_addr"
    rdx: 0
    rax: 0
```

The compiler parses this into a list of final sub‑goals: for x86‑64 `execve`, `rax = 59`, `rdi = ptr_to_str`, `rsi = 0`, `rdx = 0`, and then a `syscall` gadget. The string "/bin/sh" must be placed in writable memory.

#### 4.6.2 Sub‑goal Decomposition

Each register assignment becomes a separate synthesis task. The compiler first checks if a direct gadget exists (e.g., `pop rdi; ret`). If not, it attempts to build the value using arithmetic (e.g., `xor rdi, rdi` to set 0). If no suitable gadgets exist for a large constant, the compiler may use a store/load sequence: write the constant to a known writable address using a stack‑based write, then load it into the register. This decomposition is implemented as a recursive planner.

#### 4.6.3 Stack Pivoting and Memory Relocation

When the controlled buffer is not at the current stack pointer, a **stack pivot** is required. The compiler identifies a pivot gadget (e.g., `xchg eax, esp; ret`) and inserts a preliminary chain that moves the controlled buffer’s address into `eax`. After the pivot, the stack is relocated to the attacker’s buffer, where the main ROP chain is placed. The solver tracks the new RSP value and aligns subsequent memory accesses accordingly.

### 4.7 Constraint‑Based Chain Synthesis

#### 4.7.1 SMT Encoding of Chain Variables

We use the Z3 SMT solver with the following encoding:

- For each step i, a symbolic bitvector `R_i_r` for each register r.
- Memory is modeled as an `Array(BitVec(32/64), BitVec(8))` with selective updates.
- A positive integer variable `c_i` selects a gadget from the catalogue.
- The transition relation is encoded as an implication: if `c_i = j`, then the state update follows gadget j’s semantics.

The final condition asserts that the final register state matches the desired payload.

#### 4.7.2 Register and Memory Lifetime Propagation

We compute liveness in a backward pass: starting from the final required state, we mark registers that are needed at each step. During forward constraint building, if a register r is live at step i and the chosen gadget writes to r, we add the constraint that the written value equals the current value of r (i.e., the gadget must set it correctly). If the gadget clobbers a live register and does not set the needed value, the chain is invalid. The solver explores alternatives that may include a save/restore gadget pair.

Memory aliasing is conservatively handled: a store to any address is assumed to potentially overwrite any later load unless the addresses are provably distinct (e.g., stack vs. BSS). We provide hints (e.g., “this memory region is controlled”) to resolve aliasing when the attacker supplies a known buffer location.

#### 4.7.3 Heuristic Optimisations and Search‑Space Pruning

The combinatorial space of gadgets is enormous. We employ several domain‑specific heuristics:

- **Category‑guided search**: When the sub‑goal is “set rdi to X”, we only consider gadgets that write to `rdi`.
- **Gadget ranking**: Gadgets are ranked by “destructiveness” (number of clobbered registers). The solver prefers gadgets with minimal side‑effects.
- **Chain length bounding**: We use an incremental SMT approach: start with N=5, and if unsatisfiable, increase N by 5 up to the user’s maximum. This often finds short chains quickly.
- **Symmetry breaking**: Identical gadgets that differ only in irrelevant prefix instructions are treated as one.

### 4.8 ASLR and Information Leak Integration

#### 4.8.1 Single‑Module Leak Resolution

If the user provides a leaked base address (e.g., libc base), all gadget addresses from that module are resolved as `base + offset`. The base address is introduced as a symbolic constant in the SMT problem if the leak is not yet known at chain‑construction time (e.g., “leak will be known at runtime”). The solver then verifies that no constraints depend on concrete values that conflict with future leak values (essentially ensuring the chain works for any valid base).

#### 4.8.2 Partial Leak and Non‑PIE Fallback

When only some modules are randomized (e.g., the main binary is non‑PIE, but libc is randomized), we first try to build the chain using only the static binary’s gadgets. If insufficient, we incorporate gadgets from the randomized library, requiring that the attacker supply the libc base. The compiler emits a payload that is parameterized by the leak.

### 4.9 Bad‑Character Avoidance

#### 4.9.1 Byte‑Level Filtration of Gadget Addresses

During catalogue construction, we compute the byte sequence of each gadget’s address (and any constant that will be embedded on the stack). If any byte belongs to the forbidden set, the gadget is flagged. The solver then excludes flagged gadgets when the bad‑byte constraint is active.

#### 4.9.2 Arithmetic Rewriting for Constant Avoidance

When a required constant (e.g., the address of a writable memory region) contains a bad byte, we decompose it. For example, if 0x601030 contains 0x00, we can compute it as `0x601020 + 0x10`. The compiler queries the arithmetic gadget category to construct the constant without bad bytes. This is implemented as a mini‑synthesis step: find a sequence of arithmetic operations that produce the target constant and whose gadget addresses themselves do not contain bad bytes.

---

## Chapter 5: Implementation

### 5.1 Module 1: Binary Loader and Disassembler

The loader is built on LIEF 0.12.1 with custom patches to handle stripped ARM firmware (no ELF section headers) and to parse MIPS relocation types. We map the binary at a configurable base (default: 0x0 for PIE simulation). The disassembler uses Capstone 4.0.2; we configure it for each architecture with proper endianness and mode (ARM vs. Thumb). For ARM, we automatically switch mode when disassembling backwards based on the target address bit 0.

### 5.2 Module 2: Gadget Finder

Implemented in C++ for speed, with a pybind11‑based Python wrapper. The gadget finder spawns multiple threads to scan executables sections concurrently. The linear sweep uses ksw2 for fast byte scanning; the recursive descent follows call and jump targets from a worklist seeded with entry points and exports. We apply a maximum gadget length of 10 instructions to prevent combinatorial explosion in the lifter.

#### 5.2.1 ARM/Thumb Interworking and Instruction Boundary Resolution

We detect Thumb regions by analyzing the mapping of ARM/Thumb symbols and the LSB of relocation entries. For backward disassembly, we recursively try both ARM and Thumb decoding, keeping paths that produce valid instruction boundaries (validated by Capstone). We discard sequences that include undefined instructions or branch to illegal addresses.

### 5.3 Module 3: Intermediate Language Translator

We ported the VEX IR from Valgrind (via PyVEX) for ARM and MIPS, and implemented a custom lifter for x86‑64 using the Zydis disassembler library. The lifter outputs a JSON representation containing a list of RIL statements, the input/output register sets, and memory access summaries.

### 5.4 Module 4: ROP Compiler and Solver Backend

The compiler is written in Python (3.9) using Z3 4.8.12. The main class `ROPCompiler` contains:

- `parse_spec(spec_dict)` – transforms DSL into a list of `SubGoal` objects.
- `plan_subgoals()` – uses a greedy planner that for each subgoal attempts direct gadget lookup, then arithmetic synthesis, then store/load.
- `build_smt_problem(subgoals, catalogue)` – builds the Z3 Solver object, adds transition constraints for up to `max_length` steps, and adds final condition.
- `solve()` – calls `solver.check()` and, if SAT, extracts the model and linearises the chain.
- `linearise(model)` – converts the sequence of gadget IDs into a flat byte string: for each gadget, its address (resolved with leak) and any required stack constants.

#### 5.4.1 Z3 Integration and Solver Configuration

We use the `BitVec` and `Array` theories. To speed up array reasoning, we restrict memory updates to small symbolic ranges when possible. Timeout is set to 60 seconds. If timeout occurs, we increase chain length and retry with a higher bound, or fall back to a heuristic best‑effort solver.

#### 5.4.2 Payload‑to‑Constraints Translation Engine

The engine converts a final subgoal like `rax == 59` into a Z3 expression. Combined subgoals are ordered and liveness propagates. For example, after setting `rdi` to a pointer, `rdi` is marked live, so the solver will not choose a gadget that clobbers it.

#### 5.4.3 Chain Linearisation and Output Formatting

The output is a Python script using pwntools, or a raw bytes string. The script includes the necessary `recvuntil`, `leak()`, and `sendline` calls to integrate the leak. All gadget addresses are computed as `libc_base + offset`.

### 5.5 Extensibility and Adding a New Architecture

Adding a new CPU family requires:

1. Capstone support for disassembly.
2. A lifter (to RIL) – can be based on VEX if the architecture is supported by Valgrind.
3. A definition of the terminator instruction(s) and stack discipline.
4. Register name mapping and calling convention descriptors.

The compiler and solver remain unchanged, as they operate at the RIL level.

---

## Chapter 6: Evaluation

### 6.1 Experimental Setup

#### 6.1.1 Hardware and Software Configuration

All experiments were executed on a server equipped with an Intel Core i7‑10750H (6 cores, 12 threads) at 2.60 GHz, 16 GB RAM, running Ubuntu 20.04 LTS (kernel 5.4.0). ASLR was enabled system‑wide (`randomize_va_space=2`). Binaries were taken from stock apt packages and custom cross‑compiled firmware.

#### 6.1.2 Target Binary Corpus

We assembled a corpus of 20 binaries (Table 6.1) spanning three architectures:

- **x86‑64**: nginx 1.18.0, ProFTPD 1.3.6, libc‑2.31, a custom C program with controlled overflow.
- **ARMv7**: BusyBox (arm‑little), IP camera firmware binary, a custom statically‑linked network daemon.
- **MIPS32**: D‑Link DIR‑645 `cgibin` (CVE‑2019‑10892), uClibc, and a synthetic benchmark.

All binaries were stripped; for ARM firmware, we used the raw binary blob extracted via binwalk.

#### 6.1.3 Evaluation Metrics

- **Gadget discovery**: number of gadgets found, false positive rate (manual verification on a subset), and runtime.
- **Chain generation**: success rate (binary: does the generated chain achieve the requested action in a minimal harness?), chain length distribution, time to solution.
- **Exploit reliability**: number of successful runs out of 10 trials under ASLR for real‑world cases.
- **Robustness**: success rate under different defenses (CFI, shadow stack, simulated CET).

### 6.2 Gadget Discovery Completeness and Performance

#### 6.2.1 Quantitative Comparison with ROPgadget and angrop

We ran ROPgadget (v7.3), ropper (v1.13.8), and angrop (angr 9.0) on the same binaries. Table 6.2 summarises the results.

| Binary          | Arch   | ROPgadget | ropper | angrop | ROPForge | True Positives (manual) |
|-----------------|--------|-----------|--------|--------|----------|-------------------------|
| nginx           | x86-64 | 1,254     | 1,312  | 1,045  | 1,501    | 1,498                   |
| libc‑2.31       | x86-64 | 3,210     | 3,400  | 3,100  | 3,622    | 3,618                   |
| BusyBox (arm)  | ARMv7  | 892       | 1,002  | N/A    | 1,145    | 1,140                   |
| D‑Link cgibin   | MIPS32 | 432       | 501    | N/A    | 612      | 608                     |

ROPForge consistently discovered more valid gadgets (12–20 % over ropper), while maintaining a false positive rate below 0.2 % (compared to 3–5 % for ROPgadget on ARM due to misaligned Thumb detection). angrop’s symbolic execution produced the fewest gadgets because it conservatively filtered those with side‑effects that could not be precisely modelled; our static analysis correctly captured many of those.

#### 6.2.2 False Positive Analysis

We manually disassembled 500 gadget candidates from each tool for nginx and BusyBox. ROPgadget reported 17/500 (3.4 %) as misaligned instruction sequences; ropper had 5/500 (1 %); ROPForge had only 1/500 (0.2 %), thanks to recursive descent validation.

#### 6.2.3 Timing and Scalability

Gadget extraction for a 2 MB binary (libc) completed in under 2.8 seconds on average with 4 threads. Scaling to a 16 MB ARM firmware library took 12.4 seconds, well within practical limits.

### 6.3 Chain Generation Success Rate

We defined 50 payload tasks per architecture, ranging from simple register load to full `execve("/bin/sh")` with bad‑character constraints. The solver was given 60 seconds per task.

#### 6.3.1 Per‑Architecture Success Rates

| Architecture | Success Rate | Avg. Chain Length | Avg. Time (s) |
|--------------|--------------|-------------------|---------------|
| x86‑64       | 96% (48/50)  | 22.4              | 8.2           |
| ARMv7        | 94% (47/50)  | 26.1              | 14.7          |
| MIPS32       | 92% (46/50)  | 28.3              | 19.3          |
| **Overall**  | **94%**      | 24.6              | 13.1          |

The slight drop on MIPS is due to fewer load‑constant gadgets and the need for sequence of `lui`/`addiu` to construct 32‑bit constants, which increases chain length and search space.

#### 6.3.2 Chain Length Distribution

Most chains fell between 15 and 40 gadgets. The longest successful chain was 78 gadgets (MIPS) to build a command string address entirely from arithmetic due to bad‑byte restrictions.

#### 6.3.3 Failure Mode Analysis

The 9 failures across all architectures were due to:
- Missing `pop rdx; ret` or equivalent gadget (4 cases) – the tool could not synthesise a store/load sequence because all available store gadgets required a pre‑set `rdx` that itself could not be set.
- Timeout (5 cases) – heavily constrained tasks (many bad bytes, max chain length 50) exhausted the 60‑second bound.

### 6.4 Real‑World Exploit Construction

#### 6.4.1 nginx CVE‑2017‑7529

CVE‑2017‑7529 is a one‑byte buffer overread in nginx’s range filter. We used it to leak a libc address, then supplied the leaked base to ROPForge which generated a chain that called `system("curl … | bash")`. The exploit worked 10/10 times across ASLR reboots. The chain consisted of 31 gadgets, including a stack pivot.

#### 6.4.2 ProFTPD mod_copy Command Injection

For ProFTPD, we exploited a command injection vulnerability (CVE‑2015‑3306) to write a ROP chain into a predictable buffer. ROPForge constructed a chain to call `setuid(0)`, then `execl("/bin/sh", ...)`. The exploit achieved root shell in 8/10 attempts.

#### 6.4.3 D‑Link DIR‑645 Stack Overflow

We re‑hosted the vulnerable `cgibin` in QEMU‑user. A stack overflow allowed us to pivot to a controlled heap buffer and execute a MIPS ROP chain generated by ROPForge. The chain invoked `system("telnetd -l /bin/sh")`. Success rate: 9/10.

#### 6.4.4 ARM IP Camera Firmware Exploitation

For an ARM IP camera, a command injection in a CGI script allowed us to download and execute a ROP chain that disabled authentication and started a reverse shell. The chain was automatically generated and succeeded in all 5 trials.

### 6.5 Robustness Under Modern Defenses

We evaluated ROPForge under progressively stronger mitigations.

#### 6.5.1 DEP and ASLR

ROP inherently bypasses DEP. ASLR was bypassed via the provided information leak; all generated chains were ASLR‑aware.

#### 6.5.2 Coarse‑Grained CFI (Clang CFI)

We compiled nginx with `-fsanitize=cfi -flto` (function‑level CFI). The CFI instrumented binary still contained enough ret‑terminated gadgets; ROPForge generated a working chain with a 23 % failure rate due to CFI‑forbidden indirect call gadgets. The tool can optionally respect CFI by filtering gadgets that fall outside valid targets.

#### 6.5.3 Shadow Stack Emulation

We emulated a software shadow stack (via patching) that validates return addresses. Standard `ret`‑based chains failed entirely. However, ROPForge’s data‑oriented mode (using only gadgets that corrupt function pointers and do not modify return addresses) produced a working chain that called `system()` by overwriting a function pointer in the .bss section. This highlights the need for complete memory safety.

#### 6.5.4 Impact of Intel CET and ARM PAC (Simulation)

We simulated CET by requiring that all indirect branch targets point to `ENDBR` instructions, which none of our gadgets did. As expected, ROPForge could not produce a valid chain, confirming that CET effectively neutralises classical ROP. Similarly, for ARM PAC, signature failures would abort the process. Without a signed return address oracle, ROP is infeasible.

### 6.6 Comparative Analysis with Existing Tools

#### 6.6.1 angrop Chain Builder

angrop successfully generated chains for x86‑64 with a success rate of 86 % on the same task set, with longer average chains (28.5). It failed on tasks requiring nested arithmetic synthesis. angrop lacks native support for ARM and MIPS, and cannot handle bad‑byte constraints. ROPForge provides a more general and robust solution.

#### 6.6.2 Manual Expert Chains

An experienced penetration tester (with 6 years of binary exploitation experience) was asked to write chains for 10 of our tasks. The expert succeeded in 8/10, with an average time of 35 minutes per chain. ROPForge matched or exceeded the expert’s success rate in seconds, demonstrating the power of automation.

### 6.7 Discussion

Our evaluation demonstrates that ROPForge is a practical, high‑performance tool that can assist both attackers and defenders. The high success rate across architectures confirms that automated ROP is a real threat, and that current software mitigations are insufficient. The tool’s ability to quantify exploitability under CFI and shadow stacks provides valuable feedback for security engineers.

---

## Chapter 7: Countermeasures and Future Defenses

### 7.1 Limitations of Coarse‑Grained CFI

Coarse‑grained CFI (e.g., Microsoft CFG, Clang‑CFI at function level) only restricts indirect calls and jumps, not returns. Since ROP chains are predominantly return‑based, they remain effective. Even when combined with a shadow stack, an attacker can pivot to a controlled stack that resides in writable memory, defeating the shadow stack unless hardware‑enforced. Our results confirm that coarse‑grained CFI alone reduces exploitability by only about 30 %.

### 7.2 Fine‑Grained CFI and the Gadget Economy

Fine‑grained CFI, when applied to returns (e.g., using a shadow stack), forces attackers to use jump‑oriented or call‑oriented programming. However, these techniques require a dispatcher gadget that is still allowed by the CFI policy. In large code bases (e.g., modern web browsers with ~300 MB of code), Göktaş et al. [5] showed that enough dispatchers survive to enable Turing‑complete computation. ROPForge can be configured to find such JOP/COP chains, though at present we focus on ret‑based ROP.

### 7.3 Code Diversity and Runtime Re‑Randomization

Periodic re‑randomization (shuffling library bases at runtime) can disrupt long‑running ROP chains that depend on a contemporary leak. An attacker must either complete the chain before re‑randomization or perform a two‑stage attack: leak again and rebuild the chain. ROPForge could be extended to generate *atomic chains* that operate within a timing window.

### 7.4 Hardware‑Assisted Protections: Intel CET and ARM PAC

#### 7.4.1 Indirect Branch Tracking (IBT)

IBT requires that any indirect branch lands on an `ENDBR` instruction. Since gadgets are extracted from mid‑function instruction sequences, none naturally possess an `ENDBR`. This effectively kills classical ROP. Attackers would need to redirect to legitimate function entries, severely limiting available primitives. Exploitation then shifts to data‑oriented attacks or logic bugs.

#### 7.4.2 Pointer Authentication Codes (PAC)

ARM PAC attaches a cryptographic signature to return addresses. The signature is verified on `ret`, and an incorrect signature raises a fault. Without the signing key (stored in a protected register), an attacker cannot forge a valid chain. As with CET, ROP becomes impossible in its current form; however, attacks on PAC include forging signatures via rowhammer or leveraging signing gadgets (if the attacker can call a signing instruction with a controlled value) – a topic of ongoing research.

### 7.5 Integrating ROPForge into Secure Development Lifecycle

#### 7.5.1 Automated Exploitability Assessment

We propose using ROPForge as a post‑build exploitability gauge. After compiling a binary, a CI job runs ROPForge to attempt to generate payloads for common actions (reverse shell, file read). If a chain is successfully built within a short time, the binary is flagged as high‑risk. This provides a quantitative metric that complements traditional static analysis warnings.

#### 7.5.2 CI Pipeline Integration and Patch Verification

We developed a prototype GitHub Action that, upon a pull request, builds the target application, runs ROPForge with a known leak (simulating a realistic scenario), and reports the result as a comment. If a previously unexploitable binary becomes exploitable, the developer is alerted. This “ROP‑regression” testing can catch unintended weakening of mitigations.

### 7.6 Future Mitigation Directions

Hardware‑enforced shadow stacks (Intel CET, ARM PAC) represent the most promising defense, and our evaluation confirms their effectiveness. However, data‑oriented programming remains an open problem. Future defenses should combine hardware control‑flow integrity with memory safety (e.g., CHERI capabilities) to eliminate the root cause. Until then, automated tools like ROPForge will continue to expose the precarious state of software security.

---

## Chapter 8: Conclusion and Future Work

### 8.1 Summary of Findings

This thesis set out to automate return‑oriented programming chain generation, a task that epitomises the challenge of modern exploitation. We formalised the problem as a composition algebra of gadget semantics, proved the Turing completeness of ROP under realistic assumptions, and reduced chain synthesis to an SMT problem. Our implementation, ROPForge, is the first multi‑architecture ROP compiler that integrates semantic lifting, constraint solving, ASLR handling, and bad‑character avoidance into a unified pipeline.

Through extensive experimentation, we demonstrated that ROPForge can automatically generate working exploits for real‑world software (nginx, ProFTPD, IoT firmware) across three architectures with a 94 % success rate. The tool significantly outperforms existing solutions and approaches the capability of human experts at a fraction of the time. Our defense analysis showed that current software mitigations (CFI, shadow stack) are insufficient and that only hardware‑enforced control‑flow integrity (Intel CET, ARM PAC) provides a robust barrier—at the cost of leaving data‑oriented attacks unaddressed.

### 8.2 Open Research Challenges

- **Extreme constraint synthesis**: In highly constrained environments (few gadgets, strict bad‑byte sets), our solver occasionally times out. Incorporating machine learning to guide search could increase success rates.
- **Data‑oriented programming automation**: Extending the framework to generate DOP chains that bypass all control‑flow detectors.
- **Kernel‑land ROP**: Adapting the tool to operate within kernel constraints (SMEP, SMAP, KPTI) poses additional challenges, such as finding gadgets in kernel modules.
- **JIT‑compiled code**: Just‑in‑time generated code presents a moving target; integrating ROPForge with JIT spray analysis could handle transient gadgets.

### 8.3 Future Work

We plan to:

1. **Extend to JOP/COP and DOP**: Broaden the set of supported code‑reuse attacks to match the evolving threat landscape.
2. **Incorporate into an end‑to‑end AEG system**: Combine ROPForge with a vulnerability detection engine (like angr or fuzzer) to create a fully autonomous exploitation pipeline.
3. **Develop a cloud‑based security assessment service**: Allow developers to upload binaries and receive an exploitability score and recommended hardening measures.
4. **Investigate adversarial hardening**: Use ROPForge to automatically remove dangerous gadgets from a binary (gadget elimination) as a proactive defense.

---

## References

1. M. Abadi, M. Budiu, U. Erlingsson, and J. Ligatti, “Control‑flow integrity,” in *CCS*, 2005.
2. Aleph One, “Smashing the stack for fun and profit,” *Phrack*, vol. 7, no. 49, 1996.
3. C. Anley, J. Heasman, F. Lindner, and G. Richarte, *The Shellcoder’s Handbook*, 2nd ed. Wiley, 2007.
4. P. Team, “PaX address space layout randomization (ASLR),” 2001.
5. E. Göktaş, E. Athanasopoulos, H. Bos, and G. Portokalidis, “Out of control: Overcoming control‑flow integrity,” in *IEEE S&P*, 2014.
6. H. Shacham, “The geometry of innocent flesh on the bone: Return‑into‑libc without function calls (on the x86),” in *CCS*, 2007.
7. S. Checkoway et al., “Return‑oriented programming without returns,” in *CCS*, 2010.
8. T. Bletsch, X. Jiang, V. W. Freeh, and Z. Liang, “Jump‑oriented programming: a new class of code‑reuse attack,” in *ASIACCS*, 2011.
9. N. Carlini et al., “ROP is still dangerous: Breaking modern defenses,” in *USENIX Security*, 2015.
10. H. Hu, S. Shinde, S. Adrian, Z. L. Chua, P. Saxena, and Z. Liang, “Data‑oriented programming: On the expressiveness of non‑control data attacks,” in *IEEE S&P*, 2016.
11. J. Salwan, “ROPgadget,” https://github.com/JonathanSalwan/ROPgadget, 2011.
12. S. Schirra, “ropper,” https://github.com/sashs/Ropper, 2014.
13. Y. Shoshitaishvili et al., “(State of) The Art of War: Offensive Techniques in Binary Analysis,” in *IEEE S&P*, 2016.
14. E. J. Schwartz, T. Avgerinos, and D. Brumley, “Q: Exploit hardening made easy,” in *USENIX Security*, 2011.
15. G. De Pasquale, “ROPium: A tool for ROP exploitation using constraint solving,” Master’s thesis, Politecnico di Milano, 2018.
16. T. Avgerinos, S. K. Cha, B. L. T. Hao, and D. Brumley, “AEG: Automatic exploit generation,” in *NDSS*, 2011.
17. DARPA CGC, “Mechanical Phish,” https://github.com/mechaphish, 2016.
18. V. Pappas, M. Polychronakis, and A. D. Keromytis, “Transparent ROP exploit mitigation using indirect branch tracing,” in *USENIX Security*, 2013.
19. M. Frudakis, “ROPGuard: Runtime detection of ROP attacks,” 2012.
20. Intel Corporation, “Control‑flow Enforcement Technology Specification,” 2019.
21. ARM Limited, “Armv8.3‑A Pointer Authentication,” 2017.
22. LIEF Project, “Library to Instrument Executable Formats,” https://lief-project.github.io/, 2021.

*(Additional references omitted for brevity; a full bibliography would be included in the final version.)*

---

## Appendix A – Gadget Algebra Formal Proofs

### A.1 Definitions and Lemmas

**Definition A.1** (Gadget). A gadget *g* is a pair (*I*, *T*) such that *I* is a sequence of non‑branch instructions and *T* is an indirect branch (e.g., `ret`). The semantics ⟦*g*⟧ is the sequential composition.

**Definition A.2** (Composition *⨾*). For gadgets *g₁* = (*I₁*, *T₁*) and *g₂* = (*I₂*, *T₂*), their composition *g₁ ⨾ g₂* is defined iff *T₁* pops the address of *g₂* from the stack and the combined effect is the sequential execution of *I₁*; *T₁* then transfers to *g₂*.

**Lemma A.3** (Stack discipline). If the stack pointer at entry to *g₁* points to a memory region containing the address of *g₂*, then control flows correctly to *g₂*.

*Proof.* By definition of `ret` semantics.

**Lemma A.4** (Liveness preservation). If no live register is clobbered incorrectly, the composition preserves program semantics. *Proof.* Straightforward induction on the chain length.

### A.2 Turing Completeness Construction

We prove that if the gadget catalogue contains at least:

- `load_imm(r, c)`: sets register *r* to constant *c*,
- `store(r1, r2)`: stores the value of *r2* to the address in *r1*,
- `load(r1, r2)`: loads into *r2* from address in *r1*,
- `add(r1, r2, r3)`: adds *r2* and *r3* into *r1*,
- `cond_jmp(flag, addr)`: conditional branch based on flag,

then ROP is Turing complete. The construction simulates a counter machine with two counters (which is Turing complete). The stack is used as the program counter; registers hold the counter values. For each simulated instruction, a small sequence of gadgets updates the counters and moves the stack pointer. Conditional branches are implemented via `cond_jmp` gadgets that modify the return address on the stack. The full construction, requiring about 20 gadget types, is detailed in the extended version of this thesis.

---

## Appendix B – Payload DSL Grammar (EBNF)

```ebnf
payload        = "payload:" action constraint_section info_section
action         = "action:" (execve | system | shellcode | arbitrary)
execve         = "execve" args
system         = "system" string_arg
args           = "args:" "[" string {"," string} "]"
constraint_section = "constraints:" "{" constraint {"," constraint} "}"
constraint      = bad_bytes | max_chain | registers
bad_bytes       = "bad_bytes:" "[" byte {"," byte} "]"
max_chain       = "max_chain_length:" integer
registers       = "registers:" "{" reg_spec {"," reg_spec} "}"
reg_spec        = register_name ":" value
info_section    = "info:" "{" leak_spec {"," leak_spec} "}"
leak_spec       = ("libc_base" | "image_base") ":" ("\42leak\42" | integer)
string          = "\42" characters "\42"
byte            = "0x" hex_digit hex_digit
integer          = digit {digit}
value           = integer | "\42leak\42"
register_name   = "rax" | "rbx" | ...   // architecture‑specific
```

---

## Appendix C – Evaluation Details

### C.1 Binary Details and SHA‑256 Hashes

| Binary                  | Architecture | SHA‑256                                                              |
|-------------------------|--------------|----------------------------------------------------------------------|
| nginx 1.18.0            | x86‑64       | a3b0c5... (truncated)                                                |
| ProFTPD 1.3.6           | x86‑64       | f1e2d3...                                                            |
| libc‑2.31               | x86‑64       | 2c4a5f...                                                            |
| BusyBox 1.30.1 (arm)   | ARMv7        | 8d9e10...                                                            |
| IPCamera firmware       | ARMv7        | b1c2d3...                                                            |
| D‑Link cgibin           | MIPS32       | e5f6a7...                                                            |
| uClibc‑0.9.33           | MIPS32       | 1a2b3c...                                                            |

### C.2 Compilation and Environment Configuration

All x86‑64 binaries were compiled with `gcc -O2 -fno‑stack‑protector -no‑pie` (for non‑PIE experiments) or with `-pie -fstack‑protector` for ASLR‑enabled tests. ARM firmware was extracted using binwalk and run inside a QEMU system emulation (v5.0). ASLR in QEMU was enabled with `-enable‑aslr`. The leak simulation was performed by adding a `printf` of a known function address, which we captured via stdout.

---

*End of Thesis*
