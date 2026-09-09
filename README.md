# Webchan 1.0.0

**Build:** `090920262021`  
**Source extension:** `.webchan`  
**Textual IR extension:** `.webir`  
**Primary target:** single-host-thread WebAssembly and native integration

Webchan is an experimental proof-directed systems and numerical language for finite, structured computation. The compiler treats bounded work, dependence, locality, topology, data layout, and external effects as semantic information rather than leaving all of them to a low-level optimizer.

The central rule is:

> Do not execute, store, synchronize, or materialize work that can be proved unnecessary.

The complete 1.0.0 source package is distributed in [`dist/webchan-1.0.0-090920262021.zip`](dist/webchan-1.0.0-090920262021.zip). This repository keeps the public landing page and release artifact small; the archive contains the compiler, runtime, examples, tests, generated outputs, validation reports, and full reference documentation.

---

## 1. Architecture

The compilation path is:

```text
.webchan source
      |
      v
semantic + termination + temporal checks
      |
      v
structural proof and profitability analysis
      |
      v
Webchan IR (.webir)
      |
      v
internal generated C
      |
      v
Clang / LLVM
      |
      v
WebAssembly or native host integration
```

Webchan owns source semantics, proof obligations, structural transformations, and execution contracts. Clang/LLVM is used as the machine-level backend for instruction selection, register allocation, vectorization, target scheduling, and final code generation.

The compiler is not organized around workload names. A matrix, a graph, a bounded indexed kernel, and an imported dependence region are different source objects, but they can contribute evidence to the same structural optimizer.

---

## 2. Main structural systems

### Structural Iteration Algebra

Structural Iteration Algebra, or SIA, represents finite indexed computation through:

- finite induction domains;
- nested bounded loops;
- local scalar state;
- indexed reads and writes;
- gather and scatter relations;
- arithmetic and bitwise index maps;
- local predication;
- dependence topology;
- reduction and scan structure;
- legality gates for fusion, re-linearization, and other loop transforms.

The generic rule is:

```text
finite domain
    -> bounds/dependence/locality proof
    -> legal transformation
    -> profitability test
    -> exact fallback
```

No FFT, stencil, sort, rendering, simulation, or other workload-specific opcode is required.

### Structural Data Layout Algebra

Structural Data Layout Algebra, or SDLA, separates the logical array model from physical storage.

Two logical arrays may remain separate, become interleaved, become tiled, or be scalarized when the compiler has enough evidence to prove that the rewrite preserves semantics and is profitable.

For example, source code may keep two independent logical arrays:

```text
array re f64 1024
array im f64 1024
```

If matched-index access strongly favors interleaving, the backend may represent them physically as:

```text
re0 im0 re1 im1 re2 im2 ...
```

The source-level bounds and array identities remain unchanged.

### Structural Linear Algebra

The linear-algebra proof system recognizes useful operator properties rather than forcing every problem through one solver or one matrix representation. Current paths include:

- diagonal structure;
- lower and upper triangular structure;
- banded and sparse structure;
- symmetry;
- strict diagonal dominance;
- SPD evidence;
- matrix-free operator actions;
- direct structural collapse when a cheaper exact path exists.

A matrix-free action is preferred when assembling a global matrix would only create storage and memory traffic without adding information required by the computation.

### Structural Topology

The topology layer supports finite graph and selected two-dimensional simplicial-complex reasoning. Implemented graph capabilities include:

- connected components;
- bridges;
- articulation vertices;
- Euler characteristic;
- first Betti number;
- graph homotopy rank;
- tree contractibility and simple connectivity;
- topology-aware planning and pruning.

For finite 2-complexes, Webchan computes GF(2) boundary ranks and Betti numbers and supports elementary collapse-to-point certificates. `H1 = 0` is not treated as a proof of simple connectivity.

### Inverse Parallel Expansion

Inverse Parallel Expansion imports finite fork-join dependence structure as evidence. If purity, acyclicity, and dependence constraints are proved, orchestration that is not part of the program's meaning can be removed and a legal single-thread representation can be generated.

This is not a claim that every parallel program can be serialized without cost. The transformation is proof-gated and preserves dependence.

### External Capability Interface

The External Capability Interface, or ECI, describes external work in terms of semantic capabilities instead of hard-wiring the language to a vendor API.

The policy is:

```text
authority first
C ABI preferred when eligible
providers replaceable
vendor-specific path as fallback
```

Potentially blocking foreign work must be declared as blocking. Asynchronous work cannot silently masquerade as an immediate direct call.

---

## 3. Release contents

After extracting the distribution archive, the important paths are:

```text
compiler/                 compiler source
runtime/                  portable runtime
examples/                 complete Webchan programs
host/                     Node, browser, and native host probes
tests/                    positive and negative regression tests
docs/                     language and architecture documents
generated/                generated output from the main example
results/                  validation and benchmark reports
build_and_test.sh          complete build and regression entry point
README.md                  full release tutorial
RELEASE_NOTES.md           release identity
SHA256SUMS.txt             per-file release checksums
```

Generated output is included so the lowering boundary can be inspected directly.

---

## 4. Build the compiler

From the extracted release directory:

```bash
cc -O2 -std=c11 compiler/webchanc.c -lm -o build/webchanc
```

The command-line interface is intentionally small:

```text
webchanc <input.webchan> <output-dir>
```

For example:

```bash
mkdir -p my_generated
./build/webchanc examples/universal.webchan my_generated
```

The compiler emits files including:

```text
my_generated/program.webir
my_generated/webchan_program.c
my_generated/webchan_program.h
my_generated/procedural_fields.glsl
my_generated/external_capabilities.json
my_generated/webchan_eci_native.c
```

`program.webir` is usually the best first file to inspect when studying a compilation result because it records the finite-work and structural decisions made before C/LLVM lowering.

---

## 5. Minimal program

A Webchan source file begins with an explicit execution contract. The following is a minimal 1.0.0 program:

```text
webchan 1.0.0
module hello_webchan

target portable_wasm
profile universal
backend c_llvm_wasm
c_backend authoritative
generated_c internal_only
host_abi portable
execution single_host_thread

time_safety strict
frame_budget_ms 16.667
slice_budget_ms 2.0
max_continuous_ms 4.0
max_step_units 2048
default_step_units 512
yield_policy input_fair
group_commit progressive
implicit_barrier forbidden
causal_topology true
active_set_cap 16
chebyshev_order 4
chebyshev_tolerance 1e-8

optimization proof_directed
proof_policy profitable_only
wild_project_policy textbook_only
parallel_migration inverse_parallel_expansion

external_interface authority_first
c_abi_policy prefer
provider_policy replaceable
vendor_policy fallback_only

memory_mode dense
memory_recompute_budget 1
memory_scratch_cap 16

fn affine_demo effect bounded 8
repeat 1 affine 2.0 1.0
endfn

task main_task shed_rank 0 cost 1 deadline_ms 10 class visible fn affine_demo
```

Compile it with:

```bash
mkdir -p out
./build/webchanc hello.webchan out
```

Webchan 1.0.0 rejects a mismatched language header, invalid execution contract, recursive call cycle, or strict-path unbounded execution.

---

## 6. Generic bounded indexed kernels

The indexed-kernel surface is intended for ordinary finite numerical work, not a closed numerical DSL.

A simple elementwise kernel has the form:

```text
array input f64 1024
array output f64 1024

kernel scale effect bounded 1024
for i from 0 to 1024 step 1
    local f64 x = input[i]
    set output[i] = x * 2.0
endfor
endkernel
```

The compiler proves the induction domain and each indexed access before lowering. A write such as `output[i + 1]` in the same domain is rejected because `i = 1023` would produce an out-of-range index.

Nested domains are supported when their bounds remain statically finite. Local scalar state can carry values between iterations. Arithmetic and bitwise index expressions can describe permutations and local neighborhood maps.

Predication is written with the generic kernel form:

```text
if condition
    set output[i] = value
endif
```

The compiler does not attach workload meaning to the branch. It is simply a conditional local update whose bounds and dependence consequences must remain valid.

---

## 7. Temporal execution

A memory-safe program can still freeze a browser main thread. Webchan therefore treats finite work and continuous host-thread occupation as separate concerns.

The release configuration contains explicit values such as:

```text
frame_budget_ms 16.667
slice_budget_ms 2.0
max_continuous_ms 4.0
max_step_units 2048
yield_policy input_fair
```

These values are part of the execution contract, not informal comments.

A large synchronous Wasm call can still block the browser. Webchan's temporal model is useful when work is represented as bounded steps that can return control to the host between steps. Throughput and responsiveness are therefore measured separately.

---

## 8. Matrix-free and non-materialized computation

Webchan treats materialization as a cost.

If an operator can be applied exactly without building a global matrix, the compiler can preserve a matrix-free representation. If a temporary state can be reconstructed, fused, scalarized, or removed without changing semantics, physical storage can be reduced.

This has three practical effects:

1. less memory traffic;
2. a smaller active working set;
3. fewer intermediate values retained in memory.

The third property is an attack-surface reduction, not cryptographic encryption. Confidential data still requires real cryptography.

---

## 9. Safety model

Webchan uses several safety dimensions that should not be conflated.

### Bounds safety

Indexed kernel accesses are interval-proved against fixed extents before lowering.

### Termination and temporal safety

Bounded functions and kernels carry finite-work evidence. Recursive call cycles and unsupported unbounded strict-path loops are rejected.

### Causal safety

Task and IPE dependency cycles are rejected. Scheduling metadata never grants permission to violate dependence.

### Capability safety

External work crosses explicit capability boundaries. Blocking behavior must be declared.

### Structural non-materialization

The compiler may remove storage only when exact reconstruction or elimination is proved and the transformation is profitable.

---

## 10. Full validation

The authoritative regression entry point is:

```bash
./build_and_test.sh
```

The suite builds the compiler and examples, produces WebAssembly, runs host probes, checks structural paths, and verifies compiler rejection cases.

Negative tests cover cases including:

- unbounded loops;
- recursive cycles;
- invalid task/dependence cycles;
- invalid memory contracts;
- out-of-bounds indexed writes;
- zero loop steps;
- unknown index symbols;
- declared kernel bounds smaller than the computed bound;
- unclosed kernels;
- invalid topology declarations;
- invalid ECI blocking declarations.

After a complete build, useful reports include:

```text
results/VALIDATION.txt
results/REPORT.md
results/TOOLCHAIN.txt
results/compiler_negative.txt
```

The 1.0.0 release package was validated with a zero-exit full build before publication.

---

## 11. Performance guidance

Webchan does not define performance as "every instruction is faster." Most of its higher-level optimization opportunities come from changing how much work and storage reaches the low-level backend.

When writing performance-sensitive code:

1. express the finite structure directly;
2. prefer exact locality and dependence information over opaque helpers;
3. avoid manually assembling global matrices when an operator action is sufficient;
4. let logical arrays describe program meaning instead of manually flattening every layout;
5. keep legality and profitability separate;
6. verify results before timing;
7. keep orchestration languages and measurement scripts outside timed regions;
8. compare equivalent algorithms and equivalent numerical semantics.

LLVM remains responsible for the final machine-level optimization. Webchan's role is to deliver a smaller and structurally clearer program to that backend when proof permits it.

---

## 12. Current boundaries

Webchan 1.0.0 does not claim to provide:

- unrestricted pointer alias mutation;
- a mature dynamic heap ownership model;
- general strings and a production text-processing ecosystem;
- mature packages, modules, or generics;
- arbitrary unbounded loops;
- recursive call cycles;
- complete arbitrary fundamental-group decision for finite 2-complexes;
- higher homotopy groups;
- persistent homology or Morse theory;
- a universal optimal data-layout cost model;
- guaranteed optimal SIMD lowering;
- a cross-backend exact fused-multiply-add source primitive;
- a built-in FFT, stencil, sorting, rendering, simulation, or benchmark-specific opcode.

A missing proof does not authorize a guess. The compiler keeps an exact fallback where one exists or rejects the operation when the required safety property cannot be established.

---

## 13. Reference documentation

The distribution archive contains the complete manual set. Start with:

```text
docs/LANGUAGE_SPEC.md
docs/WEBCHAN_CONTRACT.md
docs/WEBCHAN_MANIFESTO.md
docs/STRUCTURAL_ITERATION_ALGEBRA.md
docs/STRUCTURAL_DATA_LAYOUT_ALGEBRA.md
docs/STRUCTURAL_LINEAR_ALGEBRA.md
docs/STRUCTURAL_TOPOLOGY.md
docs/INVERSE_PARALLEL_EXPANSION.md
docs/EXTERNAL_CAPABILITY_INTERFACE.md
docs/MATRIX_FREE_MEMORY.md
```

`examples/universal.webchan` is the broadest executable source example in the release, and `results/REPORT.md` records the release validation surface.

---

## Release artifact

- Version: `1.0.0`
- Build: `090920262021`
- Build identifier format: `DDMMYYYYHHMM`
- Archive: [`dist/webchan-1.0.0-090920262021.zip`](dist/webchan-1.0.0-090920262021.zip)
- Archive SHA-256: `71ce26e8c4881b02aa1a00d46598dfff960684e60b02291e1b148b3deaeeb784`

Webchan 1.0.0 is an experimental compiler release. The implementation is published for inspection, reproduction, and technical evaluation.