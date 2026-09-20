---
name: typed-binary-buffer-manipulation
description: Use when typed binary arrays with highly correlated consecutive elements
  (e.g. radio-telescope correlator data) must be bit-transposed as a pre-filter before
  LZ4/LZF/ZSTD compression, or validated against the reference Bitshuffle library.
when_to_use_negative:
- Consecutive elements of the dataset are not highly correlated — Bitshuffle relies
  on neighbouring elements being correlated (correlations spanning at least 24 elements)
  to improve compression
- You need an algorithm that compresses data itself — Bitshuffle only rearranges bits;
  a separate LZ77-class compressor (LZF, LZ4, ZSTD) must be applied afterwards
- Sophisticated entropy coders like GZIP or BZIP are the only available downstream
  compressors — because Bitshuffle modifies data at the bit level, they are unlikely
  to beat simpler duplicate-string-elimination algorithms
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Bitshuffle
  role: Reference implementation of the bit-transpose pre-filter, with C/Python bindings
    and HDF5 filter number 32008
  repo: https://github.com/kiyo-masui/bitshuffle
- name: HDF5
  role: Data format and dynamically loaded filter framework through which the Bitshuffle
    filter is integrated and applied to datasets
- name: LZ4
  role: Fast LZ77-style lossless compressor applied to each block after shuffling;
    integrated directly into Bitshuffle
- name: ZSTD
  role: Alternative fast lossless compressor Bitshuffle includes routines and HDF5
    filter options to apply after shuffling
  repo: https://github.com/facebook/zstd
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2015.07.002
    title: A compression scheme for radio data in high performance computing
  source_content_hash: sha256:fac1ecb070144ab470492369e4e84a6bc0932a9a19790c032f03eb24c8dff847
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/kiyo-masui/bitshuffle
  tools:
  - Bitshuffle
  - HDF5
  - LZ4
  - ZSTD
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# typed-binary-buffer-manipulation

## Summary

Implements Bitshuffle's bit-level transpose pre-filter, which rearranges typed binary data (e.g. 32-bit signed integer CHIME visibility elements) so that like-significance bits are grouped, improving downstream LZ77-class compression without compressing data itself.

## When to use

Use when typed binary arrays with highly correlated consecutive elements (e.g. radio-telescope correlator data) must be bit-transposed as a pre-filter before LZ4/LZF/ZSTD compression, or validated against the reference Bitshuffle library.

## When NOT to use

- Consecutive elements of the dataset are not highly correlated — Bitshuffle relies on neighbouring elements being correlated (correlations spanning at least 24 elements) to improve compression
- You need an algorithm that compresses data itself — Bitshuffle only rearranges bits; a separate LZ77-class compressor (LZF, LZ4, ZSTD) must be applied afterwards
- Sophisticated entropy coders like GZIP or BZIP are the only available downstream compressors — because Bitshuffle modifies data at the bit level, they are unlikely to beat simpler duplicate-string-elimination algorithms

## Inputs

- Typed binary data array (e.g. 32-bit signed integer CHIME visibility elements) as a Numpy array or raw buffer
- Block size parameter (default Bitshuffle block size, roughly 8 kB / 4096-element blocks)

## Outputs

- Bit-transposed buffer with all like-significance bits grouped per row
- bit_transpose_results.npz containing per-block round-trip equality flags, reference-agreement flags, and throughput timings

## How to apply

Load a typed binary array (e.g. 32-bit signed integer CHIME Pathfinder visibility elements) and split it into blocks of the Bitshuffle block size — the README says the transpose is 'performed within blocks of data roughly 8 kB long', chosen to fit L1 cache and match the LZF compression window. For each block, arrange the bytes as a matrix with the array elements as rows and the bits within elements as columns, then transpose so all least-significant-bits land in a row, etc.; the reference implementation vectorizes this with SSE2 (and SSE/AVX) instructions. Validate the in-house transpose by applying the inverse transpose (untranspose) and asserting byte-identity with the original input, and compare against the reference library (github.com/kiyo-masui/bitshuffle) byte-for-byte on the same blocks. Record per-block round-trip equality and reference-agreement flags plus throughput timings, since expected throughput is roughly 1 byte per clock cycle on modern x86 (up to 2 on Haswell+).

## Related tools

- **Bitshuffle** (Reference implementation of the bit-transpose pre-filter, with C/Python bindings and HDF5 filter number 32008) — https://github.com/kiyo-masui/bitshuffle
- **HDF5** (Data format and dynamically loaded filter framework through which the Bitshuffle filter is integrated and applied to datasets)
- **LZ4** (Fast LZ77-style lossless compressor applied to each block after shuffling; integrated directly into Bitshuffle)
- **ZSTD** (Alternative fast lossless compressor Bitshuffle includes routines and HDF5 filter options to apply after shuffling) — https://github.com/facebook/zstd

## Examples

```
import numpy as np, bitshuffle; a = np.arange(1024, dtype=np.int32); out = bitshuffle.bitshuffle(a.view(np.uint8), (8192, 0)); rec = bitshuffle.bitunshuffle(out, (8192, 0)); assert np.array_equal(rec, a.view(np.uint8))
```

## Evaluation signals

- Round-trip invariant: applying the inverse transpose (untranspose) to the bit-transposed blocks yields a result byte-identical to the original input
- Reference agreement: in-house transposed output is byte-for-byte identical to output of the reference Bitshuffle library (github.com/kiyo-masui/bitshuffle) on the same blocks
- Throughput check: transpose throughput is roughly 1 byte per clock cycle on modern x86 (up to 2 bytes/cycle on Haswell and later); timings well below that suggest a faulty implementation
- Compression gain sanity check: downstream LZ4/LZF/ZSTD compression of the transposed blocks should outperform compression of the raw blocks for correlated radio data
- Block size check: transpose is performed within blocks of roughly 8 kB, matching L1 cache and the LZF window

## Limitations

- Bitshuffle does not itself compress data — it only rearranges it for more efficient compression; a separate compression library (LZF, LZ4, ZSTD) is required
- Less general than many compression algorithms: consecutive elements must be highly correlated (correlations spanning at least 24 elements) to improve compression
- Binary wheels are built with AVX2 support and will crash with SIGILL on unsupported processors (pre-2015); source builds are needed for unsupported x86_64 or AVX512 targets
- Sophisticated entropy-reducing compressors such as GZIP and BZIP are unlikely to achieve significantly better compression than simpler and faster duplicate-string-elimination algorithms after bit-level rearrangement

## Evidence
