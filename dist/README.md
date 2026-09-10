# Distribution

This directory contains published Webchan release archives for reproducibility and technical evaluation.

## Webchan 1.0.6

- Build: `091020261435`
- Archive: [`webchan-1.0.6-091020261435.zip`](webchan-1.0.6-091020261435.zip)
- Archive size: `986577` bytes
- SHA-256: `641724ef23adba094e40be5bd049a8f142c4a38b4c663d8a3efdd6067ba91ab4`
- Checksum file: [`webchan-1.0.6-091020261435.zip.sha256`](webchan-1.0.6-091020261435.zip.sha256)

Webchan 1.0.6 adds structural-passport proof transfer across equivalent representations and a bounded browser runtime path with resumable kernels, async ECI completion/cancellation, bounded streams with backpressure, incremental UTF-8/SSE handling, event mailboxes, DOM mutation coalescing, deferred repair, and an optional Worker host. The synchronous fast kernel path is retained.

Final release validation: `234 PASS / 0 FAIL`, `30/30` compiler negative gates, and `312/312` packaged-file SHA-256 checks.

Verify the archive with:

```bash
sha256sum -c webchan-1.0.6-091020261435.zip.sha256
```

## Historical release

### Webchan 1.0.0

- Build: `090920262021`
- Archive: [`webchan-1.0.0-090920262021.zip`](webchan-1.0.0-090920262021.zip)
- Archive size: `410051` bytes
- SHA-256: `71ce26e8c4881b02aa1a00d46598dfff960684e60b02291e1b148b3deaeeb784`
- Checksum file: [`webchan-1.0.0-090920262021.zip.sha256`](webchan-1.0.0-090920262021.zip.sha256)

The release archives contain compiler source, runtime code, examples, host probes, generated outputs, regression tests, validation reports, and documentation. Webchan remains experimental software intended for inspection, reproduction, and technical evaluation.
