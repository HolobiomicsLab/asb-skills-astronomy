---
name: sse2-avx2-vectorized-transpose-optimization
description: Use when a lossless compression pipeline needs a high-throughput bit-level
  transpose of typed arrays (e.g. CHIME 32-bit visibility integers) and naive per-bit
  looping is too slow.
when_to_use_negative:
- Consecutive elements of the dataset are not highly correlated (Bitshuffle requires
  neighbouring elements to be correlated, with exploitable correlations spanning at
  least 24 elements) — the transpose overhead yields no compression benefit
- Data must remain byte-addressable for byte-level shuffle filters or for sophisticated
  entropy coders like GZIP/BZIP2, which are unlikely to beat simple duplicate-string
  elimination after bit-level transposition
- Target CPU lacks the required instruction set — binary wheels are built with AVX2
  support and will crash with SIGILL on older processors (pre-Haswell Intel / pre-Excavator
  AMD)
tools:
- name: Bitshuffle
  role: Provides the SSE2/AVX-vectorized bit-transpose implementation, its inverse,
    and an HDF5 filter (number 32008) that applies the transpose per block before
    LZ4/LZF/ZSTD compression
  repo: https://github.com/kiyo-masui/bitshuffle
- name: HDF5
  role: Integrates the vectorized transpose as a dynamically loaded HDF5 filter so
    transposition/compression is transparent on read and write of datasets
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2015.07.002
    title: A compression scheme for radio data in high performance computing
  source_content_hash: sha256:72939b7b5b5fdfccbd2b3aa32f3bb4e05508ef7d89562b1d48100dd29ce8afc7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/kiyo-masui/bitshuffle
  tools:
  - Bitshuffle
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

# sse2-avx2-vectorized-transpose-optimization

## Summary

Implements the Bitshuffle bit-transpose pre-filter using vectorized SSE2/SSE and AVX instructions so that rearranging typed binary data into bit-plane-major order runs at roughly 1 byte per clock cycle on modern x86 CPUs, keeping the pre-filter cost negligible relative to I/O.

## When to use

Use when a lossless compression pipeline needs a high-throughput bit-level transpose of typed arrays (e.g. CHIME 32-bit visibility integers) and naive per-bit looping is too slow.

## When NOT to use

- Consecutive elements of the dataset are not highly correlated (Bitshuffle requires neighbouring elements to be correlated, with exploitable correlations spanning at least 24 elements) — the transpose overhead yields no compression benefit
- Data must remain byte-addressable for byte-level shuffle filters or for sophisticated entropy coders like GZIP/BZIP2, which are unlikely to beat simple duplicate-string elimination after bit-level transposition
- Target CPU lacks the required instruction set — binary wheels are built with AVX2 support and will crash with SIGILL on older processors (pre-Haswell Intel / pre-Excavator AMD)

## Inputs

- Typed binary data array (Numpy array, e.g. 32-bit signed integer visibility elements)
- Block size in bytes (Bitshuffle default blocks are roughly 8 kB / 4096-byte chunks)
- Element bit width (e.g. 32 bits per element)

## Outputs

- Bit-transposed byte stream (all least-significant bits grouped into contiguous rows, suitable for LZ4/LZF/ZSTD compression)
- Bit-untransposed reconstruction for round-trip verification
- Per-block throughput timings of the transpose step

## How to apply

Load the typed binary array (e.g. 32-bit signed integer CHIME Pathfinder visibility elements) and split it into blocks sized to fit comfortably in L1 cache — Bitshuffle's blocks are roughly 8 kB long, matching the LZ4/LZF window. Within each block, arrange the data as a matrix with elements as rows and the bits within elements as columns, then transpose so all least-significant bits end up in a contiguous row. Implement the transpose with vectorized SSE2 (available on x86 since 2001) or AVX/AVX2 so the inner bit-gather loop moves multiple bits per instruction; on Haswell (2013) and later, throughput can reach up to 2 bytes per clock cycle. Verify correctness by applying the inverse transpose (untranspose) and asserting byte-identical round-trip output, and compare against the reference kiyo-masui/bitshuffle library byte-for-byte. Because the block stays in L1 cache, apply the LZ77-style compressor (LZ4, ZSTD, or LZF) immediately after the transpose on the same block rather than shuffling the whole dataset first.

## Related tools

- **Bitshuffle** (Provides the SSE2/AVX-vectorized bit-transpose implementation, its inverse, and an HDF5 filter (number 32008) that applies the transpose per block before LZ4/LZF/ZSTD compression) — https://github.com/kiyo-masui/bitshuffle
- **HDF5** (Integrates the vectorized transpose as a dynamically loaded HDF5 filter so transposition/compression is transparent on read and write of datasets)

## Examples

```
import numpy as np, bitshuffle; arr = np.fromfile('vis.bin', dtype=np.int32); out = bitshuffle.bitshuffle(arr, arr.itemsize); rt = bitshuffle.bitunshuffle(out, arr.itemsize); assert rt.tobytes() == arr.tobytes()
```

## Evaluation signals

- Round-trip invariant: applying the inverse transpose (untranspose) to the transposed blocks reproduces the original input byte-identically
- Reference agreement: output of the in-house vectorized transpose matches the reference Bitshuffle library (github.com/kiyo-masui/bitshuffle) byte-for-byte on the same blocks
- Throughput sanity check: on modern x86 processors the transpose achieves roughly 1 byte per clock cycle (up to 2 bytes/cycle on Haswell and later), comparable to an out-of-cache memory copy
- Compression downstream benefit: post-transpose data compresses better under LZ77-style compressors (LZ4/LZF/ZSTD), e.g. CHIME Pathfinder data compressed to 28% of original size
- Block-size check: transposition is confined to blocks of roughly 8 kB, chosen to fit L1 cache and the LZF compression window

## Limitations

- Vectorized transpose requires SSE2/AVX-capable x86 hardware; binary wheels built with AVX2 will crash with SIGILL on unsupported processors
- The transpose is a pre-filter only — it rearranges bits for compression but does not itself compress; a separate LZ77-style compressor is required
- Only correlations spanning at least 24 consecutive elements can be exploited; correlations below that threshold are lost by the blocking
- OpenMP parallelization and instruction-set target (e.g. AVX512) must be configured at build time via setup.cfg

## Evidence
