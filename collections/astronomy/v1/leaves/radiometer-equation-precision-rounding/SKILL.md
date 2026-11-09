---
name: radiometer-equation-precision-rounding
description: Use when correlator/visibility data (e.g. int32 real+imaginary pairs
  from instruments like the CHIME Pathfinder) must be shrunk with insignificant scientific
  impact by rounding precision relative to per-element thermal noise before lossless
  compression.
when_to_use_negative:
- When strict losslessness is required — the rounding step intentionally discards
  bits and is only 'nearly lossless' relative to thermal noise
- When adjacent elements over the fastest-varying index are not highly correlated,
  since the downstream Bitshuffle step relies on neighbouring-element correlation
  to compress (README scope statement)
- When data are not noise-dominated visibility/radio data with a meaningful per-element
  noise estimate for the radiometer-equation criterion
tools:
- name: Bitshuffle
  role: Lossless compression applied after rounding; its bit-transpose exploits the
    entropy reduction from precision reduction, distributed with an HDF5 filter (filter
    number 32008)
  repo: kiyo-masui/bitshuffle
- name: HDF5
  role: Storage container used with the compression algorithm; datasets chunked (e.g.
    (8, 8, 1024), 512 KiB chunks), optionally with the 'core' in-memory driver
- name: LZ4
  role: Fast LZ77-class back-end compressor integrated directly into Bitshuffle and
    applied to each shuffled block
  repo: https://code.google.com/p/lz4/
- name: LZF
  role: LZ77-class encoder chosen due to the availability of an HDF5 filter; used
    for comparison against LZ4 after precision reduction
  repo: http://oldhome.schmorp.de/marc/liblzf.html
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2015.07.002
    title: A compression scheme for radio data in high performance computing
  source_content_hash: sha256:68570c29d179e7a751ceeb7ff5b6bc63a17bc6f2a52010fddcda0a78c60c0a6e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/kiyo-masui/bitshuffle
  tools:
  - Bitshuffle
  - HDF5
  - LZ4
  - LZF
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# radiometer-equation-precision-rounding

## Summary

A nearly lossless precision-reduction step for radio visibility data in which each data element is rounded to a multiple of the largest possible power of two, with the rounding granularity limited by a calibration-independent form of the radiometer equation, before applying Bitshuffle lossless compression.

## When to use

Use when correlator/visibility data (e.g. int32 real+imaginary pairs from instruments like the CHIME Pathfinder) must be shrunk with insignificant scientific impact by rounding precision relative to per-element thermal noise before lossless compression.

## When NOT to use

- When strict losslessness is required — the rounding step intentionally discards bits and is only 'nearly lossless' relative to thermal noise
- When adjacent elements over the fastest-varying index are not highly correlated, since the downstream Bitshuffle step relies on neighbouring-element correlation to compress (README scope statement)
- When data are not noise-dominated visibility/radio data with a meaningful per-element noise estimate for the radiometer-equation criterion

## Inputs

- Correlator visibility array of shape (64, 136, 1024) with each element a struct of two signed 32-bit little-endian integers (real and imaginary), ~68 MiB total (CHIME Pathfinder-style data)
- Fractional noise increase parameter f (recommended range 10^-5 > f > 10^-2)
- Per-element thermal-noise estimates from Equation 14

## Outputs

- Precision-reduced dataset in which every element is a multiple of the power-of-two granularity g
- HDF5 dataset ready for Bitshuffle (LZ4/LZF) lossless compression
- Compression-ratio comparison (e.g. rows for f = 0 and f = 10^-2 per back-end) showing improvement from rounding

## How to apply

First estimate the thermal noise on a data-element-by-data-element basis using Equation 14 of the paper. Choose an acceptable fractional increase in noise variance f; the authors recommend 10^-5 > f > 10^-2, with f = 10^-2 being the most aggressive value. Compute the rounding granularity g as the largest possible power of two subject to the limit given in Equation 16, then round all data (e.g. the signed 32-bit real and imaginary components of each visibility element) to a multiple of g. Store the result via HDF5 (e.g. chunk shape (8, 8, 1024) giving 512 KiB chunks for a (64, 136, 1024) int32-pair array) so that the subsequent Bitshuffle bit-transpose and LZ77-style compressor (LZ4 or LZF) can exploit the reduced entropy. This step is independent of the lossless step — either can be used without the other — and precision reduction improves compression ratios for most lossless compressors.

## Related tools

- **Bitshuffle** (Lossless compression applied after rounding; its bit-transpose exploits the entropy reduction from precision reduction, distributed with an HDF5 filter (filter number 32008)) — kiyo-masui/bitshuffle
- **HDF5** (Storage container used with the compression algorithm; datasets chunked (e.g. (8, 8, 1024), 512 KiB chunks), optionally with the 'core' in-memory driver)
- **LZ4** (Fast LZ77-class back-end compressor integrated directly into Bitshuffle and applied to each shuffled block) — https://code.google.com/p/lz4/
- **LZF** (LZ77-class encoder chosen due to the availability of an HDF5 filter; used for comparison against LZ4 after precision reduction) — http://oldhome.schmorp.de/marc/liblzf.html

## Examples

```
import h5py, bitshuffle.h5, numpy as np; f = h5py.File('vis.h5', 'w', driver='core'); dset = f.create_dataset('vis', data=vis.astype([('re','<i4'),('im','<i4')]), chunks=(8,8,1024), compression=32008, compression_opts=(0, bitshuffle.BITSHUFFLE_LZ4, 4096))
```

## Evaluation signals

- Measured fractional increase in noise variance after rounding does not exceed the chosen f (e.g. f = 10^-2)
- Every rounded element is an exact multiple of the power-of-two granularity g satisfying the Equation 16 limit
- Compression ratio improves (compressed size / original size decreases) at f = 10^-2 relative to f = 0 for both LZ4 and LZF back-ends
- Throughput of the lightly optimized rounding implementation on a single core is of order 300 MiB/s (paper's rounding condition: 289 MiB/s write, 2574 MiB/s read at 100.0% compression ratio), reproducible across trials to within a few percent
- Decompression throughput exceeds 1 GB/s on a single core after the full two-step pipeline

## Limitations

- The rounding granularity g is capped by the maximum granularity limit of Equation 16, so the most aggressive rounding is bounded even for very large f
- The scheme assumes per-element noise estimates are meaningful (data-element-by-data-element estimation via Equation 14); mis-estimated noise makes the 'nearly lossless' claim invalid
- Per the README, binary wheels of Bitshuffle require AVX2-capable processors and can crash with SIGILL on unsupported x86_64 CPUs; sophisticated entropy compressors such as GZIP/BZIP are unlikely to beat LZ4/LZF/ZSTD on bit-shuffled data

## Evidence

- [methods] rounding granularity rule: [evidence span withheld — anchor status: not_found]
