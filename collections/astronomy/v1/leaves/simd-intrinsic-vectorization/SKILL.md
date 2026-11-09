---
name: simd-intrinsic-vectorization
description: Use when a scalar per-element transform (rounding to powers of two per
  Equation 16, or bit-transpose of typed arrays) is the throughput bottleneck and
  the target x86 CPU supports SSE2 or AVX2.
when_to_use_negative:
- The target architecture lacks SSE2/AVX support (e.g. non-x86 CPUs) — README warns
  AVX2-only binary wheels of bitshuffle crash with SIGILL on unsupported processors
- The data is not dense typed binary with adjacent correlated elements; Bitshuffle-style
  vectorized transforms need highly correlated neighbouring elements to be worthwhile
- A library-provided HDF5 filter (Bitshuffle filter number 32008) already performs
  the transform transparently, so hand-written intrinsics would duplicate existing
  optimized code
tools:
- name: Bitshuffle
  role: Provides the SSE2/AVX2-vectorized bit-transpose implementation and OpenMP-parallelized
    shuffle routines used as the model and baseline for vectorizing the precision-reduction
    rounding routine
  repo: kiyo-masui/bitshuffle
- name: HDF5
  role: File format/library through which the 68 MiB CHIME Pathfinder correlator dataset
    is loaded and through which the Bitshuffle filter (number 32008) is integrated
- name: CHIME Pathfinder
  role: Source of the real interferometric correlation data used as the benchmark
    input for scalar-vs-vectorized throughput measurement
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2015.07.002
    title: A compression scheme for radio data in high performance computing
  source_content_hash: sha256:c53c9b7169aa95adc30a469c00682a8ed85d52db3b323fb7593db8d695c10995
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/kiyo-masui/bitshuffle
  tools:
  - Bitshuffle
  - CHIME Pathfinder
  - HDF5
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# simd-intrinsic-vectorization

## Summary

Accelerates byte- and word-level data transforms (Bitshuffle's bit-transpose and the noise-relative precision-reduction rounding routine) on x86 CPUs using SSE2/AVX2 intrinsics, testing the paper's hypothesis of a ~4x speed-up over the ~300 MiB/s scalar baseline.

## When to use

Use when a scalar per-element transform (rounding to powers of two per Equation 16, or bit-transpose of typed arrays) is the throughput bottleneck and the target x86 CPU supports SSE2 or AVX2.

## When NOT to use

- The target architecture lacks SSE2/AVX support (e.g. non-x86 CPUs) — README warns AVX2-only binary wheels of bitshuffle crash with SIGILL on unsupported processors
- The data is not dense typed binary with adjacent correlated elements; Bitshuffle-style vectorized transforms need highly correlated neighbouring elements to be worthwhile
- A library-provided HDF5 filter (Bitshuffle filter number 32008) already performs the transform transparently, so hand-written intrinsics would duplicate existing optimized code

## Inputs

- CHIME Pathfinder correlator dataset (Jan 25 2015, 68 MiB, shape (64, 136, 1024)) of paired 32-bit signed little-endian integers stored in an HDF5 file
- Scalar precision-reduction (rounding) implementation with granularity parameter f = 10^-3 per Equation 16
- Block size parameter matching Bitshuffle's default 4096-byte blocks

## Outputs

- vectorized_rounding_benchmark.csv with scalar throughput (MiB/s), vectorized throughput (MiB/s), and speed-up factor columns
- Vectorized SSE2/AVX2 rounding routine (C intrinsics) producing bit-identical output to the scalar reference

## How to apply

Load the CHIME Pathfinder correlator dataset (68 MiB, shape (64, 136, 1024), paired 32-bit signed little-endian integers) from HDF5, and implement the scalar reference rounding routine that rounds each element to the largest possible power of two subject to the Equation 16 granularity limit with f = 10^-3. Then re-implement the same routine with SSE2 intrinsics (16 bytes per instruction batch) or AVX2 intrinsics (32 bytes per batch), processing data in blocks matching the default Bitshuffle block size of 4096 bytes so blocks stay in L1 cache. Benchmark both variants single-threaded (reference: Intel Core i7-3770 at 3.40 GHz, SSE2 only, no AVX2) over repeated passes, compute MiB/s throughput for each, and derive the scalar-to-vectorized speed-up factor. The rationale is that Bitshuffle exploits SSE/AVX instruction sets to run at roughly 1 byte per clock on modern x86, so element-wise rounding should see a comparable (factor-of-four-hypothesized) vector speed-up.

## Related tools

- **Bitshuffle** (Provides the SSE2/AVX2-vectorized bit-transpose implementation and OpenMP-parallelized shuffle routines used as the model and baseline for vectorizing the precision-reduction rounding routine) — kiyo-masui/bitshuffle
- **HDF5** (File format/library through which the 68 MiB CHIME Pathfinder correlator dataset is loaded and through which the Bitshuffle filter (number 32008) is integrated)
- **CHIME Pathfinder** (Source of the real interferometric correlation data used as the benchmark input for scalar-vs-vectorized throughput measurement)

## Examples

```
gcc -msse2 -O2 -o bench benchmark_rounding.c && ./bench --input chime_pathfinder_20150125.h5 --f 1e-3 --block 4096 --output vectorized_rounding_benchmark.csv
```

## Evaluation signals

- vectorized_rounding_benchmark.csv exists and contains throughput columns for both scalar and vectorized variants plus a speed-up factor column
- Measured scalar baseline throughput is roughly 300 MiB/s on a single core, consistent with the lightly optimized scalar reference
- Measured speed-up factor is positive and lies within the plausible 1 to 4 range consistent with the paper's factor-of-four vectorization hypothesis
- Vectorized routine output is bit-identical to the scalar rounding of every element to the largest allowed power of two under the Equation 16 limit with f = 10^-3
- Timing is performed in 4096-byte blocks (Bitshuffle default) single-threaded, so results are not confounded by multi-threading or cache-mismatched block sizes

## Limitations

- The factor-of-four speed-up is a hypothesis; actual gain depends on instruction set availability (reference CPU i7-3770 has SSE2 but no AVX2)
- Bitshuffle binary wheels built with AVX2 support will crash with SIGILL on processors older than ~2015 (pre-Haswell/Excavator), forcing a source build with --no-binary=bitshuffle
- Vectorized transforms only pay off when consecutive data elements are highly correlated; the throughput benefit vanishes on uncorrelated data
- Throughput figures are single-core and machine-dependent; Bitshuffle's own expected rate is roughly 1 byte per clock on x86 (up to 2 bytes on Haswell+)

## Evidence

- [methods] Rounding to power of two: [evidence span withheld — anchor status: not_found]
- [other] Scalar throughput baseline: [evidence span withheld — anchor status: not_found]
