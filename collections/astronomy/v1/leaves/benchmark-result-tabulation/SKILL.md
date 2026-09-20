---
name: benchmark-result-tabulation
description: Use when comparing measured throughput of a scalar reference routine
  against an SSE2/AVX2-intrinsics vectorized variant (e.g. precision-reduction rounding
  of 32-bit integers), or when a speed-up factor must be tabulated and validated against
  a paper's hypothesis.
when_to_use_negative:
- When the goal is comparing compression ratios or compressed sizes of Bitshuffle/LZ4/LZF/DEFLATE
  rather than runtime throughput of a rounding routine
- When the question is OpenMP multi-threaded scaling or disk I/O speed rather than
  single-core scalar-versus-vectorized speed-up
- When the input is not typed binary data amenable to the precision-reduction rounding
  step (e.g. data already entropy-reduced or non-integer scientific arrays with different
  rounding semantics)
edam_operation: http://edamontology.org/operation_2945
tools:
- name: Bitshuffle
  role: Provides the vectorized SSE2/AVX2 implementation pattern being benchmarked
    and the 4096-byte block size used to time repeated passes
  repo: kiyo-masui/bitshuffle
- name: HDF5
  role: File format of the 68 MiB CHIME Pathfinder correlator dataset loaded for the
    benchmark
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2015.07.002
    title: A compression scheme for radio data in high performance computing
  source_content_hash: sha256:d35d2e24686179d18f495405bc9aa4d7e43913d423a6b92a17d41e84d8245cb5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/kiyo-masui/bitshuffle
  tools:
  - Bitshuffle
  - CHIME Pathfinder
  - HDF5
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# benchmark-result-tabulation

## Summary

Tabulates measured single-core throughput (MiB/s) of scalar versus SSE2/AVX2-vectorized implementations of the noise-relative precision-reduction (rounding) routine from the Masui et al. 2015 radio-data compression scheme, producing a CSV with a scalar-to-vectorized speed-up factor used to test the paper's factor-of-four vectorization hypothesis.

## When to use

Use when comparing measured throughput of a scalar reference routine against an SSE2/AVX2-intrinsics vectorized variant (e.g. precision-reduction rounding of 32-bit integers), or when a speed-up factor must be tabulated and validated against a paper's hypothesis.

## When NOT to use

- When the goal is comparing compression ratios or compressed sizes of Bitshuffle/LZ4/LZF/DEFLATE rather than runtime throughput of a rounding routine
- When the question is OpenMP multi-threaded scaling or disk I/O speed rather than single-core scalar-versus-vectorized speed-up
- When the input is not typed binary data amenable to the precision-reduction rounding step (e.g. data already entropy-reduced or non-integer scientific arrays with different rounding semantics)

## Inputs

- CHIME Pathfinder correlator dataset, Jan 25 2015, 68 MiB, shape (64, 136, 1024) of paired 32-bit signed little-endian integers, stored in HDF5
- Scalar reference precision-reduction (rounding) implementation with fractional noise-variance increase f = 10^-3 (Equation 16 granularity limit)
- Vectorized (SSE2 16-byte or AVX2 32-byte) variant of the same rounding routine
- Default Bitshuffle block size of 4096 bytes used as the benchmarking block

## Outputs

- vectorized_rounding_benchmark.csv with scalar throughput (MiB/s), vectorized throughput (MiB/s), and speed-up factor columns
- Per-variant timing measurements from repeated single-threaded passes over the 68 MiB dataset

## How to apply

Load the CHIME Pathfinder correlator dataset (68 MiB, shape (64, 136, 1024) of paired 32-bit signed little-endian integers) from its HDF5 file, keeping the Bitshuffle block size of 4096 bytes as the timing unit. Implement the scalar reference rounding routine that rounds each element to the largest possible power of two subject to the granularity limit of Equation 16 with f = 10^-3, then implement the same routine with SSE2 intrinsics (16 bytes per batch) or AVX2 intrinsics (32 bytes per batch). Benchmark both variants single-threaded on an x86 CPU (reference environment: Intel Core i7-3770 at 3.40 GHz, SSE2 only), timing repeated passes over the dataset in 4096-byte blocks. Compute throughput in MiB/s for each variant and the scalar-to-vectorized speed-up factor, and write the results to vectorized_rounding_benchmark.csv. Validate that the scalar baseline sits near the lightly optimized figure of roughly 300 MiB/s on a modern single core and that the speed-up factor is positive and within the plausible 1-to-4 range consistent with the paper's hypothesis.

## Related tools

- **Bitshuffle** (Provides the vectorized SSE2/AVX2 implementation pattern being benchmarked and the 4096-byte block size used to time repeated passes) — kiyo-masui/bitshuffle
- **HDF5** (File format of the 68 MiB CHIME Pathfinder correlator dataset loaded for the benchmark)

## Examples

```
python benchmark_rounding.py --input chime_pathfinder_20150125.h5 --f 1e-3 --block 4096 --simd sse2 --out vectorized_rounding_benchmark.csv
```

## Evaluation signals

- vectorized_rounding_benchmark.csv exists and contains throughput columns for both the scalar and vectorized variants plus a speed-up factor column
- Measured speed-up factor is a positive value within the plausible range of 1 to 4, consistent with the paper's factor-of-four vectorization hypothesis
- Scalar baseline throughput is in the neighborhood of the lightly optimized reference of roughly 300 MiB/s on a single core of a modern processor
- Timing is performed single-threaded, over repeated passes, in blocks matching the default Bitshuffle block size of 4096 bytes
- Vectorized output data is identical (rounding-equivalent) to the scalar reference output for f = 10^-3, confirming a fair throughput comparison

## Limitations

- Maximum achievable speed-up is bounded by the SIMD width: roughly 4x for SSE2 (16-byte batches of 32-bit integers) and the paper's hypothesis, so results far outside the 1-to-4 range indicate a benchmarking or implementation error
- SSE2/AVX2 availability is processor-dependent: per the Bitshuffle README, binary wheels built with AVX2 crash with SIGILL on unsupported processors, so builds must match the target CPU (reference environment supports SSE2 but not AVX2)
- Throughput figures are hardware- and compiler-dependent; the ~300 MiB/s scalar baseline and 1 GB/s decompression figures are single-core reference numbers, not portable guarantees
- Block-size choice strongly affects cache behaviour; Bitshuffle blocks are sized to fit comfortably within L1 cache, so benchmarking must use the same block size as deployment to be meaningful

## Evidence

- [methods] Bitshuffle implements the bit-transpose using the vectorized SSE2: "Bitshuffle implements the bit-transpose using the vec- torized SSE2 (present on x86 processors since 2001)"
- [methods] Round all data to a multiple of the largest possible power of two: [evidence span withheld — anchor status: not_found]
- [readme] Because it is able to exploit the SSE and AVX instruction sets: "Because it is able to exploit the SSE and AVX instruction sets present on modern Intel and AMD processors, on these machines compression is only marginally slower than an out-of-cache memory copy."
- [abstract] decompression throughputs in excess of 1 GB/s are obtained on a single core: "decompression throughputs in excess of 1 GB/s are obtained on a single core"
