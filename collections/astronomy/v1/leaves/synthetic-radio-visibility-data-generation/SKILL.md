---
name: synthetic-radio-visibility-data-generation
description: Use when benchmarking the paper's compression pipeline (precision reduction
  with granularity parameter f, then Bitshuffle) and no real correlator dump is available
  — construct a synthetic (64, 136, 1024) visibility array instead.
when_to_use_negative:
- Real CHIME Pathfinder correlator data (e.g. the January 25, 2015 dump) is available
  — the paper's benchmark uses the actual instrument data, so synthetic data is only
  a stand-in
- The data's fastest-varying axis is not time-ordered with correlated neighbouring
  elements, since Bitshuffle relies on adjacent-element correlation for compression
  gain
- The goal is lossy compression rather than the paper's 'nearly lossless' rounding
  scheme bounded by the radiometer-equation noise constraint
tools:
- name: Bitshuffle
  role: Lossless compression applied after precision reduction; bit-transposes data
    into element-by-bit matrix blocks then applies LZ4 or LZF, exposed as HDF5 filter
    32008
  repo: https://github.com/kiyo-masui/bitshuffle
- name: HDF5
  role: Storage layer for the chunked (8, 8, 1024) visibility array; Bitshuffle is
    integrated as a dynamically loaded HDF5 filter
- name: LZ4
  role: Fast LZ77-style back-end integrated directly into Bitshuffle to compress each
    block after the bit transpose
- name: LZF
  role: LZ77-class comparison back-end (chosen for the availability of an HDF5 filter)
    benchmarked against LZ4
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2015.07.002
    title: A compression scheme for radio data in high performance computing
  source_content_hash: sha256:5e65f97515da4b8d3393c794cfd62c719798875993ff2ff9b27c70c79c9117fa
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
yes_rate: 0.6666666666666666
n_claims: 6
---

# synthetic-radio-visibility-data-generation

## Summary

Generate a synthetic noise-dominated radio visibility dataset mimicking CHIME Pathfinder correlator output (struct of two 32-bit signed little-endian integers, time fastest-varying, frequency slowest), stored in chunked HDF5, to benchmark the two-step precision-reduction + Bitshuffle compression scheme from Masui et al. 2015.

## When to use

Use when benchmarking the paper's compression pipeline (precision reduction with granularity parameter f, then Bitshuffle) and no real correlator dump is available — construct a synthetic (64, 136, 1024) visibility array instead.

## When NOT to use

- Real CHIME Pathfinder correlator data (e.g. the January 25, 2015 dump) is available — the paper's benchmark uses the actual instrument data, so synthetic data is only a stand-in
- The data's fastest-varying axis is not time-ordered with correlated neighbouring elements, since Bitshuffle relies on adjacent-element correlation for compression gain
- The goal is lossy compression rather than the paper's 'nearly lossless' rounding scheme bounded by the radiometer-equation noise constraint

## Inputs

- Synthetic visibility dataset of shape (64, 136, 1024), each element a struct of two 32-bit signed little-endian integers (real, imaginary), ~68 MiB total
- Fractional noise increase parameter f (0 and 10^-2)
- Bitshuffle back-end choice (LZ4, LZF) and block size (default 4096 bytes)

## Outputs

- HDF5 files with chunked (8, 8, 1024) datasets compressed via Bitshuffle/LZ4 and Bitshuffle/LZF
- compression_ratio_table.csv with rows per (f, back-end) combination, ratios expressed as percentages

## How to apply

Create a noise-dominated visibility array of shape (64, 136, 1024) where each element is a struct of two 32-bit signed little-endian integers (real and imaginary parts), matching the 68 MiB CHIME Pathfinder correlator data described in the paper, with time as the fastest varying index and frequency as the slowest. Store the array in HDF5 with chunk shape (8, 8, 1024), yielding 512 KiB chunks, since neighbouring elements along the fastest-varying (time) axis must be highly correlated for Bitshuffle to work. Apply the precision-reduction step by estimating thermal noise per element (Equation 14) and rounding each value to a multiple of the largest possible power of two g subject to the Equation 16 limit, using a fractional noise increase f = 10^-2 (the most aggressive value within the recommended 10^-5 > f > 10^-2 range). Compress both native-precision and f=10^-2 versions with the Bitshuffle HDF5 filter (ID 32008) using the LZ4 back-end and default block size, and with the LZF back-end as the LZ77-class comparison, then compute compression ratios as compressed/original expressed as percentages across the f x back-end grid.

## Related tools

- **Bitshuffle** (Lossless compression applied after precision reduction; bit-transposes data into element-by-bit matrix blocks then applies LZ4 or LZF, exposed as HDF5 filter 32008) — https://github.com/kiyo-masui/bitshuffle
- **HDF5** (Storage layer for the chunked (8, 8, 1024) visibility array; Bitshuffle is integrated as a dynamically loaded HDF5 filter)
- **LZ4** (Fast LZ77-style back-end integrated directly into Bitshuffle to compress each block after the bit transpose)
- **LZF** (LZ77-class comparison back-end (chosen for the availability of an HDF5 filter) benchmarked against LZ4)

## Examples

```
import h5py, bitshuffle, bitshuffle.h5; f = h5py.File('vis.h5', 'w'); f.create_dataset('vis', data=vis, chunks=(8,8,1024), compression=32008, compression_opts=(0, bitshuffle.BITSHUFFLE_LZ4, 4096))
```

## Evaluation signals

- Compression ratio at f=10^-2 must improve (decrease) relative to f=0 for both LZ4 and LZF back-ends, consistent with the paper's finding that precision reduction improves compression in all cases
- Even without precision reduction (f=0), Bitshuffle should outperform other lossless compressors, per the paper's intro claim
- HDF5 chunks must be exactly 512 KiB ((8, 8, 1024) of 4-byte structs x2 = 8*8*1024*8 bytes)
- Total uncompressed dataset size must be ~68 MiB, matching the CHIME Pathfinder correlator dump described in the methods
- Rounding granularity g at each element must equal the largest power of two satisfying the Equation 16 constraint given the estimated noise and chosen f

## Limitations

- Bitshuffle cannot compress unless neighbouring elements along the fastest-varying axis are highly correlated; on uncorrelated synthetic data ratios will be poor
- Precision reduction is only 'nearly lossless': it is bounded by a fractional noise-variance increase f, recommended within 10^-5 > f > 10^-2, and should be justified against the radiometer equation
- Bitshuffle operates at the bit level, so sophisticated entropy coders (GZIP, BZIP) are unlikely to beat simple duplicate-string eliminators (LZF, LZ4, ZSTD) after shuffling

## Evidence

- [methods] Round all data to a multiple of the largest possible power of two subject to the limit on rounding granu- larity given in Equation 16.: [evidence span withheld — anchor status: not_found]
