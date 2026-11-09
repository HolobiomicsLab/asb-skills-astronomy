---
name: precision-reduction-rounding
description: Use when processing integrated, post-correlation radio data (e.g. CHIME
  Pathfinder correlator frames stored as 32-bit integers) where controlled precision
  reduction relative to thermal noise is needed to improve subsequent Bitshuffle/LZ77
  compression ratios.
when_to_use_negative:
- Data are not integrated post-correlation visibilities with a meaningful thermal-noise
  estimate per element (e.g. raw un-correlated telemetry or already-quantized low-bit
  data)
- The application cannot tolerate any fractional increase in noise variance (f = 0);
  the rounding step is deliberately lossy at the chosen granularity
- Downstream analysis depends on the exact unrounded integer values rather than noise-limited
  precision
tools:
- name: Bitshuffle
  role: Receives the precision-reduced data; performs the vectorized SSE2/AVX2 bit-transpose
    and applies LZ4/ZSTD lossless compression via its HDF5 filter (filter ID 32008)
  repo: https://github.com/kiyo-masui/bitshuffle
- name: HDF5
  role: Storage format for the input correlator dataset and integration point for
    the Bitshuffle and precision-reduction filters
- name: CHIME Pathfinder
  role: Source instrument of the benchmark correlator data used to validate the precision-reduction
    behaviour on real data
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2015.07.002
    title: A compression scheme for radio data in high performance computing
  source_content_hash: sha256:e994093733d8b06f30c2d3a4388d370ce3d3a68a32752c9a31247b2120093c9e
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
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# precision-reduction-rounding

## Summary

A nearly lossless rounding step that reduces the precision of integrated radio-correlator data by rounding each element to the largest possible power of two permitted by a noise-relative granularity limit, lowering information entropy before lossless compression.

## When to use

Use when processing integrated, post-correlation radio data (e.g. CHIME Pathfinder correlator frames stored as 32-bit integers) where controlled precision reduction relative to thermal noise is needed to improve subsequent Bitshuffle/LZ77 compression ratios.

## When NOT to use

- Data are not integrated post-correlation visibilities with a meaningful thermal-noise estimate per element (e.g. raw un-correlated telemetry or already-quantized low-bit data)
- The application cannot tolerate any fractional increase in noise variance (f = 0); the rounding step is deliberately lossy at the chosen granularity
- Downstream analysis depends on the exact unrounded integer values rather than noise-limited precision

## Inputs

- Integrated, post-correlation radio data as an array of paired 32-bit signed little-endian integers (e.g. CHIME Pathfinder correlator dataset of shape (64, 136, 1024), 68 MiB, stored in HDF5)
- Chosen fractional noise-variance increase f (recommended 10^-5 > f > 10^-2; benchmark uses 10^-3)
- Per-element thermal noise estimates (Equation 14)

## Outputs

- Precision-reduced (rounded) integer array suitable for bit-transpose and lossless compression
- Optional benchmark CSV (e.g. vectorized_rounding_benchmark.csv) with scalar and vectorized throughput in MiB/s and the measured speed-up factor

## How to apply

First estimate the thermal noise on a data-element by data-element basis using the generalized, calibration-independent form of the radiometer equation (Equation 14). Choose an acceptable fractional increase in noise variance f within the recommended range 10^-5 > f > 10^-2 (the benchmark workflow uses f = 10^-3). Then round all data elements to a multiple of the largest possible power of two subject to the maximum rounding granularity limit of Equation 16, implemented as a scalar reference routine (roughly 300 MiB/s single-core throughput) and optionally vectorized with SSE2 (16-byte batches) or AVX2 (32-byte batches) intrinsics, hypothesizing up to a factor-of-four speed-up. The entropy-reduced output feeds directly into the Bitshuffle bit-transpose plus LZ4/LZF lossless compression; the rounding step is independent of Bitshuffle and can be used alone.

## Related tools

- **Bitshuffle** (Receives the precision-reduced data; performs the vectorized SSE2/AVX2 bit-transpose and applies LZ4/ZSTD lossless compression via its HDF5 filter (filter ID 32008)) — https://github.com/kiyo-masui/bitshuffle
- **HDF5** (Storage format for the input correlator dataset and integration point for the Bitshuffle and precision-reduction filters)
- **CHIME Pathfinder** (Source instrument of the benchmark correlator data used to validate the precision-reduction behaviour on real data)

## Examples

```
import h5py, numpy as np; f = 1e-3; d = h5py.File('chimepf_20150125.h5', 'r')['data'][:]; p = np.maximum(np.int32(1), 2**np.floor(np.log2(f * d.astype(np.float64)**2 + 1))); rounded = (d // p) * p
```

## Evaluation signals

- Rounded output, when decompressed, differs from the input only by the bounded fractional noise-variance increase implied by the chosen f
- Scalar throughput is in the ballpark of the ~300 MiB/s single-core baseline on a modern x86 processor
- Vectorized (SSE2/AVX2) speed-up factor is a positive value in the plausible range 1 to 4, consistent with the paper's factor-of-four hypothesis
- Benchmark CSV contains throughput columns for both scalar and vectorized variants plus a speed-up factor column
- Compression ratio after Bitshuffle+LZ4 improves relative to compressing unrounded data

## Limitations

- The rounding granularity limit (Equation 16) constrains the maximum power-of-two multiple; too aggressive an f inflates noise variance beyond acceptable limits
- AVX2 paths require appropriate hardware: binary wheels with AVX2 support crash with SIGILL on processors from before ~2015 (pre-Haswell), so a source build or SSE2 fallback is needed
- Bitshuffle (and by extension the pipeline) relies on neighbouring elements being highly correlated; correlations must span at least 24 elements to be exploited, so the benefit of prior rounding is data-dependent

## Evidence

- [methods] Round all data to a multiple of the largest possible power of two subject to the limit on rounding granularity given in Equation 16.: [evidence span withheld — anchor status: not_found]
