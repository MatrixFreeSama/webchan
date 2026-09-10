# Webchan 1.0.6

Build: `091020261435`

Webchan is an experimental proof-directed systems and numerical language for finite, structured computation. Version 1.0.6 joins two previously separate concerns:

- structure-equivalent programs may share optimization permissions when the compiler can prove the same properties;
- long browser work may execute through resumable, bounded, cancellable and backpressured paths instead of occupying the host until completion.

> Structure, not spelling, grants optimization. Bounded host occupation, not eventual completion, governs responsive web execution.

The 1.0.5 Minimal Web Surface remains unchanged: ordinary source can stay short, while resolved contracts and proof information remain explicit in generated IR.

## Download

- [Webchan 1.0.6 archive](dist/webchan-1.0.6-091020261435.zip)
- [SHA-256 checksum](dist/webchan-1.0.6-091020261435.zip.sha256)
- Archive size: `986577` bytes
- SHA-256: `641724ef23adba094e40be5bd049a8f142c4a38b4c663d8a3efdd6067ba91ab4`

Published archives are indexed in [`dist/`](dist/).

## Minimal source

```text
module demo

array x f64 1000000
array y f64 1000000

kernel scale
for i in 0..1000000
y[i] = 2 * x[i]
end
end

fn identity
repeat 1 affine 1 0
end
entry identity
```

The compiler still accepts the older explicit contract and block-specific terminators.

## What is new in 1.0.6

### Structural Passport

Ordinary indexed kernels are no longer confined to the generic-kernel proof domain when their arithmetic and index relations prove a stronger structure.

For example:

```text
for i in 0..256
y[i] = 4 * x[i]
end
```

is proved as a diagonal linear operator and receives the same legal solver capability set as an equivalent explicitly declared diagonal matrix. A three-point stencil can likewise be promoted to a banded linear operator. Nonlinear forms such as `x[i] * x[i] + 1` do not receive a linear passport.

The passport uses a separate proof namespace from generic kernel facts. Workload names, browser names and benchmark identities are not optimization predicates.

### Cross-domain topology evidence

Finite graph Laplacians now export a typed linear passport. For the supported undirected graph representation the compiler can prove linearity, symmetry, static sparsity, matrix-free action and positive semidefiniteness, while reusing the topology component proof for Laplacian nullity.

An ungrounded graph Laplacian is **not** silently promoted to SPD and receives no solver unlock merely because it is symmetric. Its zero-space remains explicit.

### Resumable kernels

The original `wc_kernel_run()` fast path remains available. Compiler-proved suitable kernels additionally expose a resumable companion ABI:

```text
reset -> resume(budget) -> yield -> resume(...) -> done
```

Continuation state records progress rather than restarting the kernel. The browser host uses returned slices plus observed wall-clock time to adapt the next work-unit budget. This is cooperative safepoint scheduling, not preemptive interruption in the middle of an indivisible operation.

### Async ECI lifecycle

The External Capability Interface now has a real asynchronous lifecycle:

```text
async_start -> pending Future -> host completion/failure -> Future terminal state
                         \-> cancel -> provider AbortController
```

Source-level `eci_async` and `eci_cancel` lower into the same generic ABI. Generation tokens prevent a late completion from overwriting a cancelled or recycled Future.

### Bounded streams and backpressure

The runtime includes fixed-capacity byte streams with read/write cursors, high/low watermarks, close/fail/cancel states and host-visible contiguous spans. Producers pause when the high-water mark is reached and resume after consumers drain below the low-water mark.

This primitive is application-neutral. It can carry HTTP response bodies, WebSocket/SSE payloads, files or generated data without an unbounded queue.

### Incremental text/framing

The browser host includes incremental UTF-8 decoding, line framing and SSE framing. Decoder/parser state survives arbitrary chunk boundaries, including a UTF-8 code point split across network chunks.

### Event mailbox and active-set bound

Host events enter a bounded FIFO mailbox. The mailbox capacity is tied to the active-set contract rather than growing without limit. Overflow is explicit and counted.

### Coalesced DOM commit

DOM text, attribute and property writes are queued and coalesced by target/property key, then committed at a frame boundary. Repeated writes to the same observable location within one batch collapse to the final write.

### Deferred repair

Repeated invalidations are accumulated in a dirty set without immediately destroying completed state. Repair commit invalidates each affected task once and then recomputes through the existing causal scheduler. Repeated invalidations of the same cone therefore coalesce.

### Optional Worker host

`host/browser_worker_host.mjs` is an optional off-main-thread profile for sustained computation. It is an additional throughput placement, not a fallback for a blocking main-thread design. The main-thread browser runtime is still required to be resumable and bounded.

## Optimization policy

Webchan retains the same two optimization classes.

**Housekeeping Optimization** contains classical compiler infrastructure such as constant propagation, DCE, CSE, LICM, induction-variable simplification, standard SIMD lowering, locality costing and normal inlining/outlining profitability.

**Luxury Optimization** remains the Webchan-origin set: temporal safety, causal ready-frontier/rank execution, active working sets, proof-directed structural work deletion, matrix-free storage-to-compute substitution, IPE, progressive completion/deferred repair, and structural-algebra unification.

1.0.6 does not add a ninth Luxury Optimization. It expands where existing structural permissions can legally flow.

**Benchmarks are probes, never teachers.** A probe may expose a missing generic structure; workload, provider, browser, product or competitor identity may not enter an optimizer predicate.

## Browser runtime boundary

1.0.6 makes the non-blocking primitives real, but it is not an ecosystem-complete browser framework. The release provides the generic runtime, async ECI, streaming/backpressure, DOM mutation batching, event ingress and optional Worker host. It does not claim React/Vue/Svelte-equivalent component ecosystems, a complete npm replacement, or automatic proof recovery from arbitrary opaque runtime data.

A structure can only receive a passport when the AOT compiler has sufficient evidence. Runtime-generated adjacency arrays, pointer-obscured aliasing or unknown external effects are not guessed into stronger mathematical domains.

## Build and test

```sh
./build_and_test.sh
```

The release build covers the inherited compiler/runtime regressions plus 1.0.6 gates for:

- minimal/explicit surface equivalence;
- ordinary-kernel structural passports and nonlinear rejection;
- fast/resumable exact output parity;
- topology-to-linear typed proof transfer without false SPD permission;
- bounded streams and backpressure;
- async Future completion, cancellation and stale-completion rejection;
- incremental UTF-8 and SSE framing;
- active-set-bounded event mailbox;
- deferred repair coalescing;
- Chromium responsive-kernel and DOM batching behavior;
- optional Chromium Worker execution;
- scalar/SIMD128 parity and the inherited 1.0.4 housekeeping probes;
- compiler specialization/provenance policy.

Final 1.0.6 release validation: **234 PASS / 0 FAIL**, **30/30 compiler negative gates**, and **312/312 packaged-file SHA-256 checks**.

## Performance preservation

The old synchronous fast kernel remains a separate path rather than being routed through the resumable host loop. A direct Chromium A/B used the same 1.0.5 `profile_parity.webchan` source, forced scalar Wasm on both versions, and alternated the two modules in one browser process:

- Webchan 1.0.5 median: `81.7 ms`
- Webchan 1.0.6 median: `80.8 ms`
- 1.0.6 / 1.0.5: `0.989x`
- output mismatch: `0`
- SIMD instructions in both A/B modules: `0`

This probe supports "no measured fast-path regression" for that workload; it is not a universal performance claim.

## Boundaries

Webchan 1.0.6 remains experimental software rather than an ecosystem-complete replacement for C, Rust, JavaScript, or mature production web toolchains. A missing proof does not authorize a guess. The compiler keeps an exact fallback where one exists or rejects the stronger transformation when the required property cannot be established.
