---
name: radio-interferometry-visibility-data-handling
description: Use when processing large radio-interferometer visibility datasets (e.g.
  CHIME Pathfinder correlator frames in HDF5) that require noise-relative precision
  reduction to a power-of-two granularity and/or a vectorized-rounding throughput
  benchmark on x86.
when_to_use_negative:
- Datasets where adjacent elements are not highly correlated — Bitshuffle relies on
  neighbouring-element correlation (spans of at least 24 elements) to improve compression,
  so the precision-reduction + Bitshuffle pipeline gains little
- Applications requiring exact float/int32 fidelity — the rounding step is 'nearly
  lossless' only because noise-variance increase is bounded by f; use it only when
  that controlled precision loss is acceptable
- Non-x86 platforms without SSE2 — the vectorized variant and Bitshuffle's fast path
  depend on SSE/AVX instruction sets
tools:
- name: Bitshuffle
  role: HDF5 filter (number 32008) providing the vectorized SSE2/AVX2 bit-transpose
    pre-filter applied after precision reduction; its 4096-byte block size frames
    the benchmarking loop
  repo: https://github.com/kiyo-masui/bitshuffle
- name: HDF5
  role: Data format and library used to store and read the CHIME Pathfinder correlator
    visibility dataset
- name: CHIME Pathfinder
  role: Instrument whose correlator output (paired 32-bit signed integers) is the
    benchmark input data
- name: NumPy
  role: Array framework within which the scalar reference rounding routine is implemented
    (Bitshuffle's own Python bindings use the Numpy framework)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2015.07.002
    title: A compression scheme for radio data in high performance computing
  source_content_hash: sha256:4be7ef44b5688df3efd2bf0cc6ea74f08f0794f72ca31a37693ef0d69c4b94a5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/kiyo-masui/bitshuffle
  tools:
  - Bitshuffle
  - CHIME Pathfinder
  - HDF5
  - NumPy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# radio-interferometry-visibility-data-handling

## Summary

Handles CHIME Pathfinder-style radio-interferometer visibility data by applying the paper's nearly lossless, noise-relative precision-reduction (rounding) step and benchmarking a vectorized SSE2/AVX2 implementation against a scalar baseline, storing results alongside Bitshuffle/HDF5 compression.

## When to use

Use when processing large radio-interferometer visibility datasets (e.g. CHIME Pathfinder correlator frames in HDF5) that require noise-relative precision reduction to a power-of-two granularity and/or a vectorized-rounding throughput benchmark on x86.

## When NOT to use

- Datasets where adjacent elements are not highly correlated — Bitshuffle relies on neighbouring-element correlation (spans of at least 24 elements) to improve compression, so the precision-reduction + Bitshuffle pipeline gains little
- Applications requiring exact float/int32 fidelity — the rounding step is 'nearly lossless' only because noise-variance increase is bounded by f; use it only when that controlled precision loss is acceptable
- Non-x86 platforms without SSE2 — the vectorized variant and Bitshuffle's fast path depend on SSE/AVX instruction sets

## Inputs

- CHIME Pathfinder correlator dataset: 68 MiB HDF5 file, array of shape (64, 136, 1024) of paired 32-bit signed little-endian integers (collected January 25, 2015, ~2:10AM PDT)
- Fractional noise-variance increase parameter f (e.g. 10^-3)
- Element-wise thermal noise estimates from Equation 14

## Outputs

- Precision-reduced (rounded) visibility array, each element rounded to the largest possible power-of-two multiple allowed by Equation 16
- vectorized_rounding_benchmark.csv with scalar throughput (MiB/s), vectorized throughput (MiB/s), and speed-up factor columns

## How to apply

Load the CHIME Pathfinder correlator dataset (68 MiB, shape (64, 136, 1024) of paired 32-bit signed little-endian integers) from the HDF5 file. Estimate the thermal noise on a data-element-by-element basis using Equation 14, choose an acceptable fractional increase in noise variance f (recommended range 10^-5 > f > 10^-2; the benchmark task uses f = 10^-3), and round all data to a multiple of the largest possible power of two subject to the granularity limit of Equation 16. Implement a scalar reference routine (baseline throughput roughly 300 MiB/s single-core) and a vectorized variant using SSE2 intrinsics (16-byte batches) or AVX2 intrinsics (32-byte batches), processing data in blocks matching the default Bitshuffle block size of 4096 bytes. Benchmark both single-threaded on the reference x86 CPU (Intel Core i7-3770 at 3.40 GHz, SSE2 support, no AVX2) over repeated passes, compute MiB/s throughput and the scalar-to-vectorized speed-up factor, and write results to vectorized_rounding_benchmark.csv. The rationale is that precision reduction lowers information entropy before Bitshuffle's bit-transpose (vectorized with SSE2 since it is present on x86 processors since 2001) and LZ77-style lossless compression.

## Related tools

- **Bitshuffle** (HDF5 filter (number 32008) providing the vectorized SSE2/AVX2 bit-transpose pre-filter applied after precision reduction; its 4096-byte block size frames the benchmarking loop) — https://github.com/kiyo-masui/bitshuffle
- **HDF5** (Data format and library used to store and read the CHIME Pathfinder correlator visibility dataset)
- **CHIME Pathfinder** (Instrument whose correlator output (paired 32-bit signed integers) is the benchmark input data)
- **NumPy** (Array framework within which the scalar reference rounding routine is implemented (Bitshuffle's own Python bindings use the Numpy framework))

## Examples

```
import h5py, numpy as np; f = h5py.File('chime_pf_correlator.h5','r'); vis = f['vis'][:]; f_param = 1e-3; gran = noise_granularity(vis, f_param); vis_rounded = np.round(vis / gran) * gran
```

## Evaluation signals

- vectorized_rounding_benchmark.csv exists and contains throughput columns for both scalar and vectorized variants plus a speed-up factor column
- Measured speed-up factor is a positive value within the plausible range 1 to 4, consistent with the paper's factor-of-four vectorization hypothesis
- Scalar baseline throughput is roughly 300 MiB/s on a single core of the reference i7-3770 environment
- Vectorized variant produces output bit-identical to the scalar reference rounding routine (same power-of-two rounding under Equation 16 with f = 10^-3)
- Benchmark timing passes over blocks matching the default Bitshuffle block size of 4096 bytes, single-threaded

## Limitations

- The reference environment (Intel Core i7-3770, SSE2, no AVX2) validates only SSE2 vectorization; AVX2 results are extrapolated, and the factor-of-four speed-up is a hypothesis, not a guaranteed outcome
- Prebuilt binary wheels of Bitshuffle are built with AVX2 support and will crash with SIGILL on unsupported processors; source builds are needed for older or non-AVX2 hardware
- The rounding granularity limit (Equation 16) bounds how aggressively data can be rounded; too large an f visibly degrades data quality, and the recommended range is 10^-5 > f > 10^-2
- Achieved vectorized speed-up depends on block size, cache behaviour, and CPU generation (roughly 1 byte/clock on most x86, up to 2 bytes/clock on Haswell and later)

## Evidence

- [methods] rounding rule and granularity limit: [evidence span withheld — anchor status: not_found]
