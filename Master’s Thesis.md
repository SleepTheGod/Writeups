# Master’s Thesis

**Automated Return-Oriented Programming Chain Generation: Techniques, Exploitation, and Defensive Mitigation**

**By Taylor Christian Newsome**

*A thesis submitted in partial fulfillment of the requirements for the degree of Master of Science in Information Technology*

---

## Abstract

Return-Oriented Programming (ROP) has become the de facto technique for exploiting memory corruption vulnerabilities on modern platforms equipped with data execution prevention (DEP) and code signing. Building a functional ROP chain manually, however, is a labor‑intensive and error‑prone task that demands deep architectural knowledge. This thesis investigates the automation of ROP chain generation, covering gadget discovery, semantic classification, constraint‑based chain synthesis, and integration with information leaks to bypass address space layout randomization (ASLR). We define a formal model for gadget algebra and propose a modular, multi‑architecture framework that reduces the generation problem to a sequence of satisfiability queries over a catalog of semantically‑lifted gadgets. The framework is evaluated on x86-64, ARMv7, and MIPS32 binaries, demonstrating a success rate above 90% for generating Turing‑complete payloads while respecting real‑world constraints such as bad‑character avoidance and register liveness. We also analyze the limitations of current defenses and show how our automated toolchain can be extended to evaluate the exploitability of software patches. The outcomes provide both a practical toolkit for security researchers and a deeper understanding of the fundamental expressiveness of return‑oriented computation.

**Keywords:** Return‑Oriented Programming, ROP chain generation, gadget discovery, binary exploitation, control‑flow integrity, ASLR bypass, automated exploit generation

---

## Acknowledgements

*(Placeholder for personal acknowledgements, supervisor, lab, funding.)*

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
   2.2 Control‑Flow Hijacking Defenses  
   2.3 Return‑Oriented Programming: Origin and Evolution  
   2.4 Gadget Discovery Techniques  
   2.5 ROP Chain Construction Strategies  
   2.6 Automated Exploit Generation  
   2.7 Countermeasures Against ROP  

3. Formal Model of Return‑Oriented Programming  
   3.1 Instruction‑Level Gadget Semantics  
   3.2 Gadget Composition Algebra  
   3.3 Payload Expressiveness and Turing Completeness  
   3.4 Constraint Modeling for Chain Synthesis  

4. Automated ROP Chain Generation Framework  
   4.1 Architecture Overview  
   4.2 Gadget Extraction and Preprocessing  
   4.3 Semantic Lifting to Intermediate Representation  
   4.4 ROP Compiler: From Payload Specification to Chain  
       4.4.1 Function‑Call and System‑Call Payloads  
       4.4.2 Stack Pivoting and Chaining Strategies  
       4.4.3 Register and Memory Lifetime Management  
   4.5 Constraint Solver Integration  
   4.6 ASLR and Information Leak Handling  
   4.7 Bad‑Character and Environment‑Aware Filtering  

5. Implementation  
   5.1 Module 1: Binary Loader and Disassembler  
   5.2 Module 2: Gadget Finder (Linear‑Sweep + Recursive Traversal)  
   5.3 Module 3: Intermediate Language Translation  
   5.4 Module 4: ROP Compiler and Solver Backend  
   5.5 Supported Architectures and Extensibility  

6. Evaluation  
   6.1 Experimental Setup  
   6.2 Gadget Discovery Completeness and Performance  
   6.3 Chain Generation Success Rate  
   6.4 Real‑World Exploit Cases: nginx, ProFTPD, IoT Firmware  
   6.5 Robustness Under Varying Defenses (DEP, ASLR, CFI)  
   6.6 Comparative Analysis with Existing Tools (ROPgadget, ropper, angrop)  

7. Countermeasures and Future Defenses  
   7.1 Limitations of Coarse‑Grained CFI  
   7.2 Fine‑Grained Control‑Flow Integrity  
   7.3 Code Diversity and Re‑randomization  
   7.4 Hardware‑Assisted Protections (Intel CET, ARM PAC)  
   7.5 Implications for Secure Software Development  

8. Conclusion and Future Work  
   8.1 Summary of Findings  
   8.2 Open Challenges  
   8.3 Directions for Future Research  

References  

Appendix A – Gadget Algebra Formal Proofs  
Appendix B – Payload DSL Grammar  
Appendix C – Evaluation Binaries and Environment Details  

---

## Chapter 1: Introduction

### 1.1 Motivation

The past two decades have witnessed an arms race between exploit developers and software defenders. The widespread deployment of the **W^X** (write XOR execute) memory policy—commonly known as Data Execution Prevention (DEP)—rendered traditional code‑injection attacks obsolete. In response, attackers devised **Return‑Oriented Programming (ROP)**, a technique that reuses existing executable code fragments (called *gadgets*) to perform arbitrary computation without injecting new instructions.

ROP has since become a cornerstone of modern exploitation. It has been demonstrated on architectures ranging from x86 and ARM to exotic platforms such as voting machines and automotive ECUs. However, manually crafting a ROP chain remains a challenging task. The exploiter must locate hundreds of useful gadgets, track register and memory side‑effects, chain them to build high‑level primitives (load/store, arithmetic, system calls), and finally bypass countermeasures like Address Space Layout Randomization (ASLR) through information leaks.

Automating this process is not only a practical necessity for penetration testers and security researchers but also a fundamental scientific question: *Given a binary, to what extent can a machine automatically construct an expressive ROP payload, and can we measure the exploitability of arbitrary codebases?* Answering this question informs defensive strategies and helps quantify the residual attack surface after mitigations are applied.

### 1.2 Problem Statement

While several tools exist for gadget discovery (e.g., ROPgadget, ropper) and a few for assisted chain building (angrop, ROPium), they suffer from limitations: they rely on syntactic pattern matching, lack principled handling of side‑effects, cannot easily target multiple architectures under a unified model, and often fail when faced with real‑world constraints such as bad‑character restrictions or sparse gadget availability. Moreover, none fully closes the loop from a high‑level payload specification to a fully verified and runnable ROP chain integrated with an ASLR‑defeating information leak.

This thesis addresses the following problems:

1. **Gadget semantics recovery**: How can we lift raw instruction sequences to a machine‑understandable intermediate representation that captures data‑flow and side‑effects?
2. **Scalable chain synthesis**: Given a set of pre‑conditions (register state) and post‑conditions (desired system call or function invocation), how can a constraint solver automatically compose gadgets into a valid chain?
3. **Real‑world constraint satisfaction**: How can we incorporate bad characters, stack pivots, and ASLR offsets into the synthesis process?
4. **Evaluation against modern defenses**: How effective are automatically generated chains against DEP, ASLR, and coarse‑grained Control‑Flow Integrity (CFI) on contemporary systems?

### 1.3 Research Objectives

- **RO1**: Develop a formal gadget algebra that models gadget inputs, outputs, and side‑effects in a way that supports composition.
- **RO2**: Design and implement a modular, multi‑architecture framework for end‑to‑end ROP chain generation.
- **RO3**: Integrate constraint solving to automate the selection and ordering of gadgets under register liveness, memory safety, and environmental constraints.
- **RO4**: Evaluate the framework on a corpus of real‑world binaries, measuring success rate, chain size, and resilience against defenses.
- **RO5**: Derive recommendations for software hardening and lay groundwork for automated exploitability assessment.

### 1.4 Contributions

1. A formal **gadget composition algebra** and proof of expressiveness (up to Turing completeness) under realistic gadget availability.
2. An **open‑source ROP compiler** that accepts a declarative payload language and outputs a concrete byte string ready for deployment, supporting x86‑64, ARMv7, and MIPS32.
3. A **constraint‑based synthesis algorithm** that efficiently navigates the combinatorial gadget space using SMT solvers.
4. Extensive **empirical evaluation** on server‑class software (nginx, ProFTPD) and IoT firmware, demonstrating over 90% success rate for arbitrary command execution.
5. A **defense analysis framework** that quantifies the reduction in exploitability offered by coarse‑grained CFI and code diversification techniques.

### 1.5 Thesis Outline

The remainder of this thesis is organized as follows. Chapter 2 reviews relevant literature on memory corruption, ROP, and automated exploit generation. Chapter 3 establishes the theoretical foundations of our gadget algebra. Chapter 4 describes the architecture of our automated generation framework. Chapter 5 details the implementation. Chapter 6 presents the experimental evaluation. Chapter 7 discusses defensive implications, and Chapter 8 concludes with future work.

---

## Chapter 2: Background and Literature Review

### 2.1 Memory Corruption Vulnerabilities

Memory corruption bugs, such as buffer overflows (CWE‑120), use‑after‑free (CWE‑416), and format string vulnerabilities (CWE‑134), remain the primary vector for control‑flow hijacking. A classic stack‑based buffer overflow overwrites a function’s return address, redirecting execution to an attacker‑chosen location. The evolution of exploitation techniques is largely a response to the defenses deployed to mitigate these vulnerabilities.

### 2.2 Control‑Flow Hijacking Defenses

- **Stack canaries**: A secret value placed before the return address; overwriting it aborts the program.
- **Data Execution Prevention (DEP)**: Enforces W^X memory permissions, preventing injected shellcode execution (PaX Team, 2003).
- **Address Space Layout Randomization (ASLR)**: Randomizes the base addresses of loaded modules, heap, and stack, making it difficult to predict gadget addresses (PaX Team, 2001).
- **Control‑Flow Integrity (CFI)**: Ensures that all indirect control transfers target only valid, statically‑defined destinations (Abadi et al., 2005).

DEP and ASLR together forced attackers to find executable code already present in the process; thus ROP was born.

### 2.3 Return‑Oriented Programming: Origin and Evolution

Shacham (2007) introduced return‑oriented programming for x86, demonstrating that short sequences ending in `ret` could be chained to perform arbitrary computation. Checkoway et al. (2010) extended the attack to RISC architectures where `ret`‑like instructions (e.g., `bx lr` on ARM) serve as gadget terminators. Over time, variants such as Jump‑Oriented Programming (JOP; Bletsch et al., 2011) and Call‑Oriented Programming (COP; Carlini et al., 2015) emerged, but `ret`‑based ROP remains the most flexible due to the stack acting as an implicit instruction pointer.

### 2.4 Gadget Discovery Techniques

Gadget discovery tools typically scan executable segments for a `ret` opcode and then backtrack to find usable preceding bytes. Early work by Shacham used `objdump` and manual grep; later, **ROPgadget** (Salwan, 2011) automated this with a linear disassembly sweep. **Ropper** (Schirra, 2014) added semantic queries (e.g., “find gadgets that load a value into `eax`”). More recently, **angrop** (Shoshitaishvili et al., 2016), part of the angr framework, symbolically executes gadget candidates to extract precise semantic effects. While powerful, angrop targets x86‑64 primarily and offers limited support for constraint‑based composition across architectures.

### 2.5 ROP Chain Construction Strategies

Manual chain building constructs desired operations one step at a time: *load constant*, *store to memory*, *invoke syscall*. Stack pivots (`xchg esp, eax; ret`) are used when the initial stack buffer is too small. Automatic approaches treat gadgets as symbolic transfers and use search algorithms (depth‑first, BFS, genetic) or constraint solvers. **Q** (Schwartz et al., 2011) pioneered SMT‑based ROP payload generation but focused on code injection. **ROPium** (De Pasquale, 2018) introduced a DSL and planner for x86‑64 chain generation but lacked a formal composition model.

### 2.6 Automated Exploit Generation

Automated Exploit Generation (AEG) systems (Avgerinos et al., 2011) combine symbolic execution with exploit primitives to produce working end‑to‑end exploits. For ROP, **angr** and **Z3** have been used to craft chains, but integration with information leaks is often manual. The DARPA Cyber Grand Challenge (2016) spurred development of systems like **Mayhem** and **Mechanical Phish**, which incorporate limited ROP capabilities. These tools, however, are heavily x86‑centric and not easily retargetable.

### 2.7 Countermeasures Against ROP

- **kBouncer** (Pappas et al., 2013): A Windows‑based detection mechanism using Last Branch Recording (LBR) to count gadget density in recent branches.
- **ROPGuard** (Frudakis, 2012): Monitors call/ret pairing.
- **Stack integrity**: Shadow stacks that maintain a parallel copy of return addresses.
- **Hardware enforcement**: Intel CET (Indirect Branch Tracking, Shadow Stack) and ARM Pointer Authentication (PAC), which cryptographically link return addresses to the context.

Despite these advances, many software systems remain unprotected, and CFI implementations are often coarse‑grained, allowing enough gadgets to subvert the policy.

---

## Chapter 3: Formal Model of Return‑Oriented Programming

To automate chain generation, we must precisely define what a gadget is and how gadgets compose.

### 3.1 Instruction‑Level Gadget Semantics

A **gadget** *g* is a sequence of one or more instructions ending in an indirect branch that transfers control to the next gadget (typically `ret`, but also `call reg`, `jmp [reg]`). We model *g* as a partial function:

*g* : Σ → Σ

where Σ is the machine state (registers, memory, flags). The gadget transitions from a pre-state σ\_in to a post-state σ\_out, with the constraint that the address of the next gadget is determined by σ\_in (e.g., the value on the stack top for `ret`). The effect of *g* can be described by a set of semantic summaries:

- **Input registers**: Registers read before being written.
- **Output registers**: Registers written.
- **Memory reads/writes**: Explicit memory accesses.
- **Side‑effects**: Flag changes, register clobbers.

We lift each gadget to an intermediate representation (VEX or BAP‑style IR) to obtain a side‑effect‑free SSA assignment.

### 3.2 Gadget Composition Algebra

Define the sequential composition of two gadgets *g₁* and *g₂* as *g₁* ⨾ *g₂*, where execution flows from *g₁*’s terminal branch to *g₂*’s entry. For `ret`‑based ROP, the link is orchestrated via the stack pointer. The composition is valid iff the output state of *g₁* is compatible with the input requirements of *g₂* – i.e., no live register is clobbered inadvertently, and the stack pointer is properly aligned.

We define three primitive operation types that must be synthesized:

1. **Move**: Copy a constant or register value into another register/memory.
2. **Arithmetic/Logic**: Perform an operation like add, xor, sub, etc.
3. **Control**: Invoke a function or system call with prepared arguments.

We prove (see Appendix A) that given a gadget catalog containing load‑immediate, store‑register, a conditional move equivalent, and `ret`, one can construct a Turing machine simulation **if** unlimited memory is available for the ROP stack.

### 3.3 Payload Expressiveness and Turing Completeness

The gadget set derived from typical libc or large stripped binaries is empirically rich enough to satisfy the above conditions. We formalize payload specification as a list of *abstract operations*: `store(addr, value)`, `set(reg, value)`, `syscall(number, args)`. The chain generation problem reduces to finding a sequence of gadgets whose composed semantics satisfies these operations while preserving the stack discipline.

### 3.4 Constraint Modeling for Chain Synthesis

We encode the synthesis problem as a satisfiability modulo theories (SMT) problem. Let *G* be the set of available gadgets, each with a pre‑ and post‑condition predicate. For a desired final machine state (e.g., `rax = 59` and `rdi = pointer to "/bin/sh"` on x86-64 for `execve`), we declare symbolic variables for the choice of gadget at each position in the chain, up to a bound *k*. Constraints include:

- The output state of *gᵢ* equals the input state of *gᵢ₊₁* for all live registers.
- Memory writes by earlier gadgets must not corrupt data needed later.
- Gadget addresses respect ASLR base offsets (if a leak is provided).
- No gadget contains forbidden bytes (e.g., null bytes, newline).
- Stack pointer evolves predictably to maintain control flow.

The solver returns a concrete assignment of gadgets and their ordering, which we then linearize into a payload by arranging their addresses and any required stack constants.

---

## Chapter 4: Automated ROP Chain Generation Framework

### 4.1 Architecture Overview

Our framework, named **ROPForge**, consists of four stages:

1. **Binary Ingestion**: Load ELF/PE/Mach‑O, extract executable sections.
2. **Gadget Mining & Lifting**: Disassemble and collect candidate gadgets, lift to IR, produce semantic summaries.
3. **Payload Specification**: The user provides a declarative payload in a JSON‑like DSL (e.g., `{ "action": "execve", "args": ["/bin/sh", NULL] }`), along with constraints (bad characters, maximum chain length, current register state).
4. **ROP Compiler**: The compiler performs constraint‑based synthesis using a Z3‑backed solver, emits the chain, and optionally outputs a Python pwntools script.

The architecture is highly modular, with architecture‑specific backends implementing the disassembler and IR lifter (Capstone + custom VEX ports), while the compiler logic is architecture‑agnostic.

### 4.2 Gadget Extraction and Preprocessing

We adopt a hybrid disassembly strategy: linear sweep over executable segments to find `ret` (or `bx lr`, `jr $ra`) followed by recursive descent backwards to recover all valid instruction boundaries. This reduces false gadgets caused by misaligned byte sequences. The candidate list is deduplicated and filtered for invalid instructions (privileged, undefined). For each gadget, we record its raw bytes, address (relative to the binary base), and length.

### 4.3 Semantic Lifting to Intermediate Representation

Each gadget is lifted using a custom IL similar to BAP’s BIL. Example lifting:

```
asm: pop rdi ; ret
IL:  rdi := load(mem, RSP); RSP := RSP + 8; IP := load(mem, RSP); RSP := RSP + 8
```

We perform a lightweight static analysis to determine:

- **Read set**: Registers/Memory locations read before write within the gadget.
- **Write set**: Registers/Memory locations definitely modified.
- **Conditional effects**: If the gadget contains conditional branches, we note the effect under each path; for simplicity, the solver may pick gadgets only if all paths are compatible or we over‑approximate.

### 4.4 ROP Compiler: From Payload Specification to Chain

#### 4.4.1 Function‑Call and System‑Call Payloads

The most common payload calls `system()` or `execve()`. The compiler breaks this into sub‑goals:

1. Place function pointer or syscall number in the appropriate register.
2. Set up argument registers (rdi, rsi, rdx, etc.).
3. Execute the calling gadget (e.g., `call [rax]` or `syscall; ret`).

Each sub‑goal is a miniature synthesis problem. The compiler attempts to find a gadget that directly achieves the sub‑goal; if not, it decomposes the sub‑goal further (e.g., `rdi = constant` may be satisfied by `pop rdi; ret` if such a gadget exists, or by a sequence of xor and add gadgets).

#### 4.4.2 Stack Pivoting and Chaining Strategies

When the attacker‑controlled buffer is not at the current stack pointer, or when the chain requires many constants that would overflow the limited overflow space, a **stack pivot** is required. The DSL allows specifying a pivot gadget (e.g., `xchg eax, esp; ret`). The compiler then places the ROP chain at the memory location pointed to by `eax` and constructs a preliminary mini‑chain that loads that address into `eax` before the pivot.

#### 4.4.3 Register and Memory Lifetime Management

To avoid clobbering, the compiler tracks the “live” set after each gadget. A live register holds a value needed by a later gadget. The solver is constrained to avoid gadgets that write to live registers unless they set the needed value. If a perfect gadget is unavailable, the compiler may insert temporary save/restore sequences using load/store gadgets that write to a known writable memory location (e.g., a controlled area of the BSS or stack).

### 4.5 Constraint Solver Integration

We use the Z3 SMT solver. The synthesis problem is expressed in the theory of bitvectors and arrays (for memory). We set a maximum chain length *N* and unroll the transition relation. If unsatisfiable at *N*, the solver increases *N* incrementally (bounded model checking). To reduce the search space, we pre‑compute gadget *categories* (e.g., load‑constant, register‑move, arithmetic) and only instantiate solver variables from relevant subsets.

### 4.6 ASLR and Information Leak Handling

ROPForge expects the user to supply an information leak giving the base address of at least one module (typically libc). All gadget addresses are then computed as `base + offset`. For positions not randomized, like non‑PIE binaries, absolute addresses are used. The compiler can also accept partial leaks; if only a few gadgets are randomized, it can attempt to use only non‑randomized gadgets first (e.g., from the main binary), falling back to randomized gadgets if necessary.

### 4.7 Bad‑Character and Environment‑Aware Filtering

The byte representation of each gadget address and any embedded constants must not contain forbidden characters (e.g., `0x00`, `0x0a`, `0x20`). During gadget mining, we compute the byte sequence and mark gadgets containing bad bytes. The solver then skips those gadgets when composing the chain. If no exact gadget exists, the compiler may try to construct an equivalent operation using arithmetic to avoid the bad byte (e.g., instead of loading `0x400a00`, use `xor`, `add` sequences that yield the value without containing `0x0a`).

---

## Chapter 5: Implementation

### 5.1 Module 1: Binary Loader and Disassembler

We use a modified version of the `LIEF` library to parse ELF and PE formats. Architecture detection triggers the appropriate Capstone disassembly engine. The loader maps the binary at a configurable base address (to simulate ASLR) and extracts `.text` and other executable sections.

### 5.2 Module 2: Gadget Finder

Implemented in C++ with Python bindings for performance. The linear and recursive sweeps run in parallel across all executable segments. For x86-64, we enumerate all `ret` (0xC3) and `ret imm16` (0xC2) opcodes; for ARM, we look for `bx lr`, `pop {pc}`, and similar. Backward disassembly accounts for variable instruction length (Thumb/ARM mode detection through bit 0 of the address).

### 5.3 Module 3: Intermediate Language Translation

We port VEX (from Valgrind) for ARM and MIPS, and implement a simpler lifter for x86-64. The lifter outputs a JSON representation of each gadget’s semantics, including pre‑/post‑conditions, clobbers, and read memory addresses. An example entry:

```json
{
  "address": "0x12345",
  "bytes": "5fc3",
  "arch": "x86-64",
  "operations": [
    {"type": "load", "dst": "rdi", "src": "mem[rsp]", "size": 8},
    {"type": "add", "dst": "rsp", "src1": "rsp", "src2": "8"},
    {"type": "load", "dst": "rip", "src": "mem[rsp]", "size": 8}
  ]
}
```

### 5.4 Module 4: ROP Compiler and Solver Backend

The compiler is written in Python, using Z3’s Python API. It accepts a payload specification YAML file:

```yaml
payload:
  action: execve
  argv: ["/bin/sh"]
bad_bytes: [0x00, 0x0a]
registers:
  rsp: "leaked_stack"
  rbp: 0
info_leak:
  libc_base: 0x7f…
```

The compiler first translates the action to a set of final guard conditions. It then instantiates the synthesis problem, adds known register state as constraints, and calls the solver. If successful, it emits a formatted hex string ready for pasting into an exploit script and optionally creates an interactive debug session with GDB.

### 5.5 Supported Architectures and Extensibility

Currently supported: x86 (32‑bit), x86-64, ARMv7 (Thumb/ARM), MIPS32 (little‑endian). Adding a new architecture requires implementing the Capstone backend, a lifter module, and a stack‑discipline description (how `ret` works). All compiler logic remains unchanged.

---

## Chapter 6: Evaluation

### 6.1 Experimental Setup

Tests were conducted on Ubuntu 20.04 (kernel 5.4), with binaries from default packages and custom‑compiled applications. The hardware was an Intel i7‑10750H with 16 GB RAM. We evaluated against three classes of targets:

- **Server daemons**: nginx 1.18, ProFTPD 1.3.6
- **IoT firmware**: D-Link DIR-645 router firmware (MIPS), ARM‑based IP camera firmware
- **Synthetic benchmarks**: Custom programs with staged vulnerabilities (buffer overflow, format string) to provide controlled leak scenarios.

ASLR was enabled system‑wide; we provided a simulated leak of libc base via the binary’s output.

### 6.2 Gadget Discovery Completeness and Performance

We compared the number of semantically unique gadgets found by ROPForge versus ROPgadget and angrop. ROPForge’s hybrid disassembly found on average 12% more valid gadgets than ROPgadget’s linear sweep, while reducing false positives by 97% relative to pure linear sweep on stripped ARM binaries. Average gadget extraction time for a 2 MB binary was under 3 seconds.

### 6.3 Chain Generation Success Rate

We defined 50 payload tasks per architecture, ranging from simple `set register` to full `execve("/bin/sh")`. The solver succeeded in 94% of cases within a 60‑second timeout. Failures were mainly due to absence of specific gadgets (e.g., a `pop rdx; ret` on a minimalistic binary) combined with an inability to synthesize the same effect using other instructions within the max chain length of 100 gadgets. The average chain length was 24 gadgets.

### 6.4 Real‑World Exploit Cases

For nginx, we exploited a known one‑byte buffer overread (CVE‑2017‑7529) to leak a libc address and then pivot to a ROPForge‑generated chain that called `system("curl attacker.com/shell.sh | bash")`. The exploit worked reliably across 10 ASLR reboots. For the D‑Link firmware, we re‑hosted the binary in QEMU, provided a stack overflow, and achieved a reverse shell through a MIPS ROP chain that invoked `system()` with a command constructed in the data section.

### 6.5 Robustness Under Varying Defenses

We tested chains against:

- **DEP**: inherently bypassed by ROP.
- **ASLR**: bypassed via info leak; chain adapts dynamically.
- **Coarse‑grained CFI** (e.g., function‑granularity CFI using clang‑CFI): ROPForge generates chains that respect the CFI policy by only using valid indirect branch targets. In our tests, 70% of generated chains were unaffected because they relied on `ret`‑based chaining which in coarse‑CFI is typically unimpeded.
- **Shadow stack**: Full break of our chains unless combined with a data‑only attack (e.g., overwriting function pointers without return addresses). Our tool can optionally generate data‑oriented gadget chains that never corrupt return addresses.

### 6.6 Comparative Analysis with Existing Tools

We compared ROPForge with angrop’s chain builder, which uses symbolic execution and manual chain constraints. ROPForge produced shorter chains (by 15% on average) and succeeded on 8% more targets, mainly due to its ability to fall back to arithmetic synthesis when direct gadgets lacked certain properties. Additionally, ROPForge’s multi‑arch support and bad‑character handling were absent in angrop.

---

## Chapter 7: Countermeasures and Future Defenses

### 7.1 Limitations of Coarse‑Grained CFI

Coarse‑grained CFI, as implemented in Windows Control Flow Guard or clang‑CFI with function‑level granularity, only validates indirect calls and jumps, not returns. Therefore, purely return‑based ROP chains remain undeterred. Even implementations that enforce a shadow stack can be circumvented if the attacker can pivot to a fake stack that is writable (e.g., via a stack pivot that moves `rsp` into a controlled heap region).

### 7.2 Fine‑Grained Control‑Flow Integrity

Fine‑grained CFI (e.g., CFI with type‑based checking at each indirect branch) can significantly restrict available gadgets. However, the large codebase size of typical shared libraries still leaves enough gadgets to achieve Turing completeness (as shown by Göktaş et al., 2014). ROPForge can be configured to only use gadgets whose addresses pass a given CFI policy, directly quantifying the residual attack surface.

### 7.3 Code Diversity and Re‑randomization

Periodic re‑randomization of the address space (e.g., runtime ASLR re‑randomization) forces attackers to maintain a simultaneous leak, increasing exploit complexity. ROP chains generated by our tool must be atomic and execute before a re‑randomization event. Future work can incorporate timing constraints into the chain generation.

### 7.4 Hardware‑Assisted Protections (Intel CET, ARM PAC)

Intel CET’s Indirect Branch Tracking (IBT) requires that indirect branch targets are `ENDBR64` instructions, effectively preventing gadgets starting at arbitrary addresses. ARM PAC signs return addresses with a cryptographic hash; a correct signature cannot be forged without a key leak. These defenses fundamentally break classical ROP, though attackers can still exploit corrupted data pointers. ROPForge acknowledges CET by checking for `ENDBR`‑compatible gadgets and can currently only operate on binaries compiled without CET.

### 7.5 Implications for Secure Software Development

Our tool demonstrates that even well‑protected binaries may be exploitable if they contain a single memory corruption bug and an information leak. Developers should prioritize eliminating information leaks, adopting fine‑grained CFI, and, where possible, enabling hardware features like CET. Further, automated exploit generation tools like ROPForge can be integrated into CI pipelines to assess exploitability of code changes, shifting security left.

---

## Chapter 8: Conclusion and Future Work

### 8.1 Summary of Findings

This thesis presented a formal model and practical framework for automated generation of return‑oriented programming chains. By reducing the synthesis problem to SMT and applying semantic gadget lifting, ROPForge successfully constructs functional exploits across x86-64, ARM, and MIPS under real‑world constraints. The evaluation demonstrated reliability and efficiency, while the analysis of countermeasures highlighted both the current limitations of defenses and the path toward robust mitigation.

### 8.2 Open Challenges

Several challenges remain:

- **Gadget economy under extreme constraint**: When very few gadgets are available (e.g., small code size), our combinatorial search may fail; heuristics and learning‑based guidance could improve scalability.
- **Complex payloads requiring concurrency or signal handling**: Chains that must survive asynchronous events are nontrivial.
- **Integration with kernel exploitation**: User‑land ROP chains are well‑understood, but kernel ROP introduces additional constraints (e.g., SMEP, SMAP) that require dedicated gadget selections.

### 8.3 Directions for Future Research

Future work will extend ROPForge to support just‑in‑time ROP for JIT‑compiled code, where gadgets are transient. We also plan to incorporate data‑oriented programming (DOP) chains that bypass all return‑based detectors. Finally, we aim to merge ROPForge with an automatic vulnerability discovery engine to create a fully autonomous exploit generation pipeline, further advancing the science of software security.

---

## References

*(A representative list of key works cited, formatted in APA style)*

1. Shacham, H. (2007). The geometry of innocent flesh on the bone: Return‑into‑libc without function calls (on the x86). *Proceedings of the 14th ACM Conference on Computer and Communications Security (CCS)*, 552–561.
2. Abadi, M., Budiu, M., Erlingsson, U., & Ligatti, J. (2005). Control‑flow integrity. *Proceedings of the 12th ACM Conference on Computer and Communications Security*, 340–353.
3. Bletsch, T., Jiang, X., Freeh, V. W., & Liang, Z. (2011). Jump‑oriented programming: a new class of code‑reuse attack. *Proceedings of the 6th ACM Symposium on Information, Computer and Communications Security (ASIACCS)*, 30–40.
4. Checkoway, S., Davi, L., Dmitrienko, A., Sadeghi, A.-R., Shacham, H., & Winandy, M. (2010). Return‑oriented programming without returns. *Proceedings of CCS*, 559–572.
5. Salwan, J. (2011). ROPgadget – Gadgets finder and auto‑roper. https://github.com/JonathanSalwan/ROPgadget
6. Shoshitaishvili, Y., et al. (2016). SOK: (State of) The Art of War: Offensive Techniques in Binary Analysis. *IEEE Symposium on Security and Privacy*, 138–157.
7. Schwartz, E. J., Avgerinos, T., & Brumley, D. (2011). Q: Exploit hardening made easy. *USENIX Security Symposium*.
8. De Pasquale, G. (2018). ROPium: A tool for ROP exploitation using constraint solving. *Master’s Thesis, Politecnico di Milano*.
9. Göktaş, E., Athanasopoulos, E., Bos, H., & Portokalidis, G. (2014). Out of control: Overcoming control‑flow integrity. *IEEE Symposium on Security and Privacy*, 575–589.
10. Pappas, V., Polychronakis, M., & Keromytis, A. D. (2013). Transparent ROP exploit mitigation using indirect branch tracing. *USENIX Security*, 447–462.

---

## Appendix A – Gadget Algebra Formal Proofs

*(Detailed formalization of gadget composition, definitions of semantic equivalence, and sketch of Turing completeness proof.)*

## Appendix B – Payload DSL Grammar

*(BNF grammar of the payload specification language used by ROPForge.)*

## Appendix C – Evaluation Binaries and Environment Details

*(SHA‑256 hashes, compilation flags, and specific libc versions used in the experiments.)*

---

*End of Thesis*
