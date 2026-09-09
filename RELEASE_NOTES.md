# Webchan 1.0.0

Build: `090920262021`

This is the first release using the stable public semantic version `1.0.0`. Historical development milestone names and numbered internal-IR branding are not part of the release identity.

## Included systems

- temporal-safe single-host-thread execution
- proof-directed structural optimization
- Structural Linear Algebra
- Structural Iteration Algebra
- Structural Data Layout Algebra
- Structural Topology
- matrix-free and structural non-materialization paths
- Inverse Parallel Expansion
- External Capability Interface
- generic bounded indexed kernels
- proof-gated gather, scatter, predication, reduction/scan structure, and layout rewrites
- LLVM-backed generic bit-reverse lowering where the backend provides it
- zero-import portable core Wasm profile

## Release identity

- Release: `1.0.0`
- Build identifier: `090920262021`
- Build identifier format: `DDMMYYYYHHMM`
- Public archive: `webchan-1.0.0-090920262021.zip`
- Textual intermediate representation: `Webchan IR`
- Intermediate representation extension: `.webir`

The compiler does not introduce workload-specific FFT, butterfly, radix, stencil, sort, particle, RGB, AoS, or SoA opcodes for this release.

## Validation

The release archive was produced after a complete `build_and_test.sh` run with zero process exit status. The package includes the validation reports and negative compiler gates used by that build.

Archive SHA-256:

```text
71ce26e8c4881b02aa1a00d46598dfff960684e60b02291e1b148b3deaeeb784
```