---
name: single-core-throughput-benchmarking
description: Use when measuring single-core write/read throughput (MiB/s) of a rounding
  or compression routine on radio-telemetry visibility data, or when validating a
  vectorized (SSE2/AVX2) variant against a scalar baseline.
when_to_use_negative:
- Input data are not strongly correlated typed binary data (e.g. adjacent elements
  of the fastest-varying index are dissimilar) — Bitshuffle and its rounding/benchmarks
  assume high inter-element correlation.
- The goal is multi-threaded throughput measurement — this skill benchmarks a single
  core explicitly, whereas Bitshuffle itself is parallelized with OpenMP.
- The data are float or otherwise non-integer typed arrays where the power-of-two
  rounding granularity routine for signed 32-bit integers does not apply.
tools:
- name: Bitshuffle
  role: Provides the block-based (8 kB / 4096-byte default) processing model and SSE2/AVX2-vectorized
    bit-transpose whose throughput behaviour the benchmark replicates; source used
    as reference for vectorized intrinsics.
  repo: kiyo-masui/bitshuffle
- name: HDF5
  role: Used with the 'core' in-memory file driver to hold the (64, 136, 1024) visibility
    dataset with 512 KiB chunks so that timing is not confounded by disk I/O.
- name: CHIME Pathfinder correlator data
  role: Real 68 MiB Jan 25 2015 correlator snapshot serving as the benchmark workload
    against which ~300 MiB/s scalar and >1 GB/s decompression throughputs are compared.
provenance:
  source_task_ids:
  - task_006
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.07.002
    title: A compression scheme for radio data in high performance computing
  source_content_hash: sha256:1b567160be0d136130a1a8c0cc8137d68ae55d5a80eac732c5f87b49fdf4bcb6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/kiyo-masui/bitshuffle
  tools:
  - Bitshuffle
  - CHIME Pathfinder
  - CHIME Pathfinder correlator data
  - HDF5
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# single-core-throughput-benchmarking

## Summary

Benchmark the throughput of the paper's precision-reduction (rounding) and Bitshuffle-style routines on a single core, using the CHIME Pathfinder correlator dataset as the test workload, and quantify scalar-to-SSE2/AVX2 vectorized speed-up factors.

## When to use

Use when measuring single-core write/read throughput (MiB/s) of a rounding or compression routine on radio-telemetry visibility data, or when validating a vectorized (SSE2/AVX2) variant against a scalar baseline.

## When NOT to use

- Input data are not strongly correlated typed binary data (e.g. adjacent elements of the fastest-varying index are dissimilar) — Bitshuffle and its rounding/benchmarks assume high inter-element correlation.
- The goal is multi-threaded throughput measurement — this skill benchmarks a single core explicitly, whereas Bitshuffle itself is parallelized with OpenMP.
- The data are float or otherwise non-integer typed arrays where the power-of-two rounding granularity routine for signed 32-bit integers does not apply.

## Inputs

- HDF5 file with CHIME Pathfinder correlator dataset (68 MiB, shape (64, 136, 1024), paired 32-bit signed little-endian integers), or an equivalent synthetic dataset of the same shape and dtype
- Precision-reduction parameter f (e.g. 10^-3) and rounding-granularity limit from Equation 16
- Bitshuffle block size (4096 bytes) for timing-pass blocking

## Outputs

- Measured scalar and vectorized throughputs in MiB/s written to vectorized_rounding_benchmark.csv with a scalar-to-vectorized speed-up factor column
- Reproducible single-core timing measurements (within a few percent across repeated trials)

## How to apply

Generate or load a 68 MiB visibility dataset of shape (64, 136, 1024) of paired 32-bit signed little-endian integers (CHIME Pathfinder correlator data from Jan 25 2015), storing it via HDF5 using the 'core' in-memory driver with chunk shape (8, 8, 1024) giving 512 KiB chunks. Implement the scalar reference rounding routine — rounding each element to the largest possible power of two subject to the maximum granularity limit of Equation 16 with f = 10^-3 (recommended range 10^-5 > f > 10^-2) — then a vectorized variant using SSE2 intrinsics (16-byte batches) or AVX2 intrinsics (32-byte batches). Benchmark both variants single-threaded on an x86 CPU (reference: Intel Core i7-3770 at 3.40 GHz, SSE2 only), timing repeated passes over blocks matching the default Bitshuffle block size of 4096 bytes, and compute throughput in MiB/s and the scalar-to-vectorized speed-up factor. Judge success by reproducibility of timings to within a few percent and by agreement with the paper's baseline of ~300 MiB/s scalar throughput (rounding-only condition: 289 MiB/s write, 2574 MiB/s read at 100.0% compression ratio) and the hypothesized ~4x vectorization speed-up.

## Related tools

- **Bitshuffle** (Provides the block-based (8 kB / 4096-byte default) processing model and SSE2/AVX2-vectorized bit-transpose whose throughput behaviour the benchmark replicates; source used as reference for vectorized intrinsics.) — kiyo-masui/bitshuffle
- **HDF5** (Used with the 'core' in-memory file driver to hold the (64, 136, 1024) visibility dataset with 512 KiB chunks so that timing is not confounded by disk I/O.)
- **CHIME Pathfinder correlator data** (Real 68 MiB Jan 25 2015 correlator snapshot serving as the benchmark workload against which ~300 MiB/s scalar and >1 GB/s decompression throughputs are compared.)

## Examples

```
python benchmark_rounding.py --input chime_pathfinder.h5 --f 1e-3 --block-size 4096 --scalar --sse2 --out vectorized_rounding_benchmark.csv
```

## Evaluation signals

- Measured scalar throughput falls near the paper's reported ~300 MiB/s (rounding-only condition: 289 MiB/s write, 2574 MiB/s read at 100.0% compression ratio) within a few percent across repeated trials.
- vectorized_rounding_benchmark.csv exists and contains throughput columns for both scalar and vectorized variants plus a speed-up factor column.
- Measured scalar-to-vectorized speed-up factor is a positive value in the plausible range of 1 to 4, consistent with the paper's factor-of-four vectorization hypothesis.
- Timings are reproducible across repeated trials to within a few percent.
- Rounding output verified to round each element to the largest possible power of two subject to the Equation 16 granularity limit with f = 10^-3.

## Limitations

- The factor-of-four vectorization speed-up is a hypothesis from the authors; measured speed-up depends on the reference CPU (SSE2-only i7-3770 vs AVX2-capable Haswell+) and may fall below 4x.
- Binary wheels of Bitshuffle are built with AVX2 and crash with SIGILL on processors predating Haswell (2013)/AMD Excavator, so the benchmark environment must match instruction-set support or build from source.
- Throughput numbers are environment-specific: the paper reports on 'a single core of a modern processor' and Bitshuffle README claims ~1 byte per clock cycle (up to 2 on Haswell), so results should be reported alongside CPU model and instruction-set availability.

## Evidence

- [results] The lightly optimized scalar precision-reduction implementation achieves roughly 300 MiB/s throughput on a single core of a modern processor: [evidence span withheld — anchor status: not_found]
