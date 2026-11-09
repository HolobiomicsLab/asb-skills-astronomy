---
name: bit-matrix-transpose-implementation
description: Use when you need to reproduce or interoperate with Bitshuffle's byte-level
  rearrangement of typed binary arrays — e.g. verifying an in-house transpose against
  github.com/kiyo-masui/bitshuffle before applying LZ4/ZSTD/LZF compression.
when_to_use_negative:
- When the goal is actual compression — Bitshuffle only rearranges data and must be
  followed by an LZ77-style compressor (LZF, LZ4, ZSTD)
- When consecutive elements of the dataset are not highly correlated (correlations
  spanning fewer than ~24 elements cannot be exploited), since the transpose yields
  no compression benefit
- When the data are not typed binary arrays with a fixed element size in bits
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Bitshuffle
  role: Reference implementation of the bit-transpose; its python/C package provides
    shuffle/unshuffle routines and an HDF5 filter (filter number 32008) used to validate
    the in-house transpose byte-for-byte
  repo: https://github.com/kiyo-masui/bitshuffle
- name: HDF5
  role: Integration framework through which the Bitshuffle pre-filter is applied to
    datasets via dynamically loaded filters
- name: LZ4
  role: Downstream lossless compressor applied to each bit-transposed block after
    the pre-filter
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2015.07.002
    title: A compression scheme for radio data in high performance computing
  source_content_hash: sha256:1f1a2652f8fcf4a784c2a0b72d34c6adec45bcde154b4a7961687e386b70cb8a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/kiyo-masui/bitshuffle
  tools:
  - Bitshuffle
  - HDF5
  - LZ4
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# bit-matrix-transpose-implementation

## Summary

Implements and validates the Bitshuffle bit-transpose pre-filter, which rearranges typed binary data (e.g. 32-bit signed integer CHIME visibility elements) by viewing each block as a matrix of elements (rows) by bits (columns) and transposing it, without compressing data itself.

## When to use

Use when you need to reproduce or interoperate with Bitshuffle's byte-level rearrangement of typed binary arrays — e.g. verifying an in-house transpose against github.com/kiyo-masui/bitshuffle before applying LZ4/ZSTD/LZF compression.

## When NOT to use

- When the goal is actual compression — Bitshuffle only rearranges data and must be followed by an LZ77-style compressor (LZF, LZ4, ZSTD)
- When consecutive elements of the dataset are not highly correlated (correlations spanning fewer than ~24 elements cannot be exploited), since the transpose yields no compression benefit
- When the data are not typed binary arrays with a fixed element size in bits

## Inputs

- Typed binary data array (e.g. 32-bit signed integer visibility elements from CHIME Pathfinder correlator data)
- Block size parameter (default Bitshuffle block of 4096 bytes, roughly 8 kB blocks as described in the README)

## Outputs

- Bit-transposed byte buffer (same total size as input, rearranged at bit level)
- Per-block round-trip equality and reference-agreement flags with throughput timings (e.g. bit_transpose_results.npz)

## How to apply

Load a typed binary array (e.g. 32-bit signed integer correlator visibilities from CHIME Pathfinder) and split it into blocks of the default Bitshuffle block size (4096 bytes, chosen to fit L1 cache and match the LZF window). For each block, arrange the bytes as a matrix with dimensions = number of elements x element size in bits and transpose it so all least-significant-bits end up in a row; on x86 this is vectorized with SSE2/AVX for ~1 byte/clock (up to 2 on Haswell+). Apply the inverse transpose (unshuffle) to each transposed block and assert byte-identical round-trip equality with the original input. Then compute the transpose with the reference library and compare byte-for-byte with the in-house implementation, recording per-block round-trip flags, reference-agreement flags, and transpose throughput timings. The transpose itself does not compress — it only groups correlated bits to improve subsequent LZ77-style compression.

## Related tools

- **Bitshuffle** (Reference implementation of the bit-transpose; its python/C package provides shuffle/unshuffle routines and an HDF5 filter (filter number 32008) used to validate the in-house transpose byte-for-byte) — https://github.com/kiyo-masui/bitshuffle
- **HDF5** (Integration framework through which the Bitshuffle pre-filter is applied to datasets via dynamically loaded filters)
- **LZ4** (Downstream lossless compressor applied to each bit-transposed block after the pre-filter)

## Examples

```
import bitshuffle, numpy as np; arr = np.fromfile('visibilities.int32', dtype=np.int32); out = bitshuffle.bitshuffle(arr, 4096); rec = bitshuffle.unbitshuffle(out); assert np.array_equal(rec, arr)
```

## Evaluation signals

- Untransposing the transposed blocks reproduces the original input byte-identically (round-trip invariant)
- In-house transposed output matches the reference Bitshuffle library output byte-for-byte on the same blocks
- Output buffer size equals input size (transpose is a rearrangement, not a compression)
- All least-significant bits of elements within a block are grouped contiguously after transposition
- Transpose throughput is in the expected range (~1 byte/clock on modern x86, up to 2 bytes/clock on Haswell+)

## Limitations

- The transpose is performed only within blocks of roughly 8 kB; correlations must span at least 24 elements within a block to be exploited
- Sophisticated entropy coders (GZIP, BZIP) gain little after bit-level transposition; simpler duplicate-string-elimination algorithms (LZF, LZ4, ZSTD) are the intended pairings
- Binary wheels of Bitshuffle are built with AVX2 and will crash with SIGILL on pre-2015 x86_64 processors lacking those instructions

## Evidence

- [methods] bit-transpose within blocks: "4. Rearrange the bits within blocks of data by arrang- ing them in a matrix with dimensions of the number of elements by the size of each element (in bits), then perform a transpose."
- [methods] SSE2 vectorization: "Bitshuffle implements the bit-transpose using the vec- torized SSE2 (present on x86 processors since 2001)"
- [readme] README algorithm description: "Arranging a typed data array in to a matrix with the elements as the rows and the bits within the elements as the columns, Bitshuffle "transposes" the matrix"
- [readme] README block size rationale: "This transpose is performed within blocks of data roughly 8 kB long [1]_."
- [readme] README no-compression caveat: "This does not in itself compress data, only rearranges it for more efficient compression."
