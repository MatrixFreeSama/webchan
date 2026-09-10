# Webchan

Webchan is an experimental proof-directed systems and numerical language for finite, structured computation. It targets single-host-thread WebAssembly and native integration while keeping bounded work, dependence, locality, topology, data layout, and external effects visible to the compiler.

> Do not execute, store, synchronize, or materialize work that can be proved unnecessary.

Current validated build: **1.0.4** (`091020261219`)

- Source extension: `.webchan`
- IR extension: `.webir`
- Backend: internal C lowered through Clang/LLVM
- Portable Wasm: scalar baseline plus optional standard SIMD128
- 1.0.4 archive SHA-256: `57c97f8b7377f8b66ec0f61d527329b921e67b1dc0e4d34557cfc928d819bc1c`

## Compiler path

```text
.webchan
   |
   v
semantic / termination / temporal checks
   |
   v
structural proof + profitability analysis
   |
   v
.webir
   |
   v
internal generated C
   |
   v
Clang / LLVM
   |
   v
WebAssembly or native integration
```

Webchan owns source semantics, proof obligations, structural transformations, and execution contracts. LLVM handles low-level instruction selection, register allocation, vectorization, target scheduling, and final code generation.

## Structural systems

### Structural Iteration Algebra

Finite induction domains, nested bounded loops, dependence, gather/scatter relations, reductions, scans, local predication, and legality checks for loop transformations.

### Structural Data Layout Algebra

Logical arrays are separated from physical storage. Separate, interleaved, tiled, or scalarized layouts are selected only when the rewrite is legal and profitable.

### Structural Linear Algebra

The compiler can reason about diagonal, triangular, banded, sparse, symmetric, diagonally dominant, and SPD structure. Matrix-free operator actions are retained when materializing a global matrix would add storage and traffic without adding required information.

### Structural Topology

Finite graph analysis includes connected components, bridges, articulation vertices, Euler characteristic, first Betti number, graph homotopy rank, and topology-aware planning. Selected finite 2-complex operations use GF(2) boundary ranks and elementary collapse certificates.

### Inverse Parallel Expansion

Finite fork-join dependence can be imported as evidence. If purity, acyclicity, and dependence constraints are proved, orchestration that is not part of program meaning can be removed without violating causality.

### External Capability Interface

External work is described by capability rather than vendor identity. The default policy is authority-first, C ABI when eligible, replaceable providers, and vendor-specific fallback only when needed.

## Optimization policy

Webchan separates optimization into two classes.

**Housekeeping Optimization** covers classical compiler machinery: dead-code elimination, common-subexpression elimination, conventional loop optimization, induction-variable simplification, ordinary auto-vectorization, SIMD lowering, locality costing, target-feature selection, and similar established techniques.

**Luxury Optimization** is reserved for mechanisms originating in Webchan's own design. Current examples include temporal safety, causal ready-frontier execution, active working sets, structural work deletion, matrix-free substitution, Inverse Parallel Expansion, and related structural systems.

Distinctive advanced optimizers from other languages or research systems are not imported as Luxury Optimization.

Benchmarks are probes, not optimizer specifications. A benchmark may expose a missing general rule, but workload names and competitor identities are not valid optimization predicates. Probe-driven changes require an origin case, a cross-domain witness, and a negative witness.

## 1.0.4

1.0.4 adds no new Luxury Optimization. It closes a scalar-loop lowering gap with Housekeeping Optimization only:

- proved induction-variable simplification for direct `u32` affine relations;
- exact modulo-`u32` sentinel induction for fixed unit-stride finite loops, including wrap across `0xffffffff -> 0`;
- exact fallback when the proof does not hold;
- profitability-gated selective kernel outlining instead of blanket `noinline`;
- regression witnesses for affine, non-affine, mutating-base, runtime-bound, tiny, and oversized cases.

The 1.0.4 release regression suite completed with **214 PASS / 0 FAIL**, **30/30 negative gates**, and **30/30 scalar-loop housekeeping probes**. The scalar no-SIMD probe was checked to contain zero Wasm SIMD instructions.

## Build

From an extracted release tree:

```bash
./build_and_test.sh
```

Compiler only:

```bash
cc -O2 -std=c11 compiler/webchanc.c -lm -o build/webchanc
```

Compile a source file:

```bash
./build/webchanc input.webchan output_dir
```

For portable Wasm, the release can build a conservative scalar module and a standard SIMD128 variant from the same generated source. Host selection is capability-based; unsupported SIMD128 falls back to scalar.

## Example

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

The compiler proves loop domains and indexed accesses before lowering. Unsupported unbounded strict-path work, recursive call cycles, invalid dependency cycles, and unproved out-of-range accesses are rejected rather than guessed through.

## Documentation

The release archive contains the compiler source, runtime, examples, host probes, generated outputs, tests, validation reports, and reference documentation. Useful starting points are:

```text
docs/LANGUAGE_SPEC.md
docs/WEBCHAN_CONTRACT.md
docs/WEBCHAN_MANIFESTO.md
docs/OPTIMIZATION_PROVENANCE.md
docs/STRUCTURAL_ITERATION_ALGEBRA.md
docs/STRUCTURAL_DATA_LAYOUT_ALGEBRA.md
docs/STRUCTURAL_LINEAR_ALGEBRA.md
docs/STRUCTURAL_TOPOLOGY.md
docs/INVERSE_PARALLEL_EXPANSION.md
docs/EXTERNAL_CAPABILITY_INTERFACE.md
```

## Boundaries

Webchan 1.0.4 is a compiler prototype, not an ecosystem-complete replacement for C, Rust, JavaScript, or mature production toolchains. Current boundaries include unrestricted pointer alias mutation, arbitrary unbounded loops, recursive call cycles, a mature general heap/ownership system, a full package ecosystem, and complete arbitrary higher-topology reasoning.

A missing proof does not authorize a guess. The compiler keeps an exact fallback where one exists or rejects the operation when the required property cannot be established.

Published archives are kept in [`dist/`](dist/) for reproducibility.
