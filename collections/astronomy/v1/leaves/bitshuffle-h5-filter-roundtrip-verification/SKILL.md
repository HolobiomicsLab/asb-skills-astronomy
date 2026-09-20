---
name: bitshuffle-h5-filter-roundtrip-verification
description: Use when a typed, correlated binary dataset (e.g. radio-interferometry
  or float32/int arrays) must be stored losslessly in chunked HDF5 and you need to
  confirm the Bitshuffle+LZ4 filter pipeline round-trips exactly and actually shrinks
  on-disk storage.
when_to_use_negative:
- Dataset elements are not correlated along the fastest-varying index — the README
  states Bitshuffle 'relies on neighbouring elements of a dataset being highly correlated
  to improve data compression', so random/uncorrelated data yields poor or no gains
- Data is text or otherwise suited to sophisticated entropy coders — the README notes
  GZIP and BZIP are unlikely to beat LZ4/LZF/ZSTD on bit-shuffled data, but the algorithm
  targets typed binary arrays
- Storage must be readable outside HDF5 (e.g. plain NumPy .npy or raw binary) — the
  filter only applies within HDF5 (inside Python, or anywhere via the dynamically
  loaded filter / h5dump)
tools:
- name: Bitshuffle
  role: Provides the H5FILTER (32008) bit-transpose HDF5 filter and Python shuffle/unshuffle
    routines used to write and read back the chunked dataset
  repo: kiyo-masui/bitshuffle
- name: h5py
  role: Creates the chunked HDF5 dataset with the 32008 filter and compression_opts=(block_size,
    H5_COMPRESS_LZ4), and reads it back for verification
- name: HDF5
  role: Chunked storage and the dynamically loaded filters framework through which
    the Bitshuffle plugin is registered and invoked transparently on read
- name: LZ4
  role: Fast LZ77-style lossless compressor applied to each bit-shuffled block; integrated
    directly into Bitshuffle rather than applied as a sequential HDF5 filter
- name: NumPy
  role: Supplies the typed synthetic test array and the element-wise equality check
    for the round-trip validation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2015.07.002
    title: A compression scheme for radio data in high performance computing
  source_content_hash: sha256:44ab48ae9cd3e465da37f3636a36a573956b5fd5f0d88f8fe804096ffd00e659
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/kiyo-masui/bitshuffle
  tools:
  - Bitshuffle
  - HDF5
  - LZ4
  - NumPy
  - h5py
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# bitshuffle-h5-filter-roundtrip-verification

## Summary

Verifies the lossless round trip of a typed NumPy array through a chunked HDF5 dataset compressed with the Bitshuffle filter (HDF5 filter 32008) plus LZ4, and quantifies the compression ratio against the paper's ~28% CHIME Pathfinder benchmark.

## When to use

Use when a typed, correlated binary dataset (e.g. radio-interferometry or float32/int arrays) must be stored losslessly in chunked HDF5 and you need to confirm the Bitshuffle+LZ4 filter pipeline round-trips exactly and actually shrinks on-disk storage.

## When NOT to use

- Dataset elements are not correlated along the fastest-varying index — the README states Bitshuffle 'relies on neighbouring elements of a dataset being highly correlated to improve data compression', so random/uncorrelated data yields poor or no gains
- Data is text or otherwise suited to sophisticated entropy coders — the README notes GZIP and BZIP are unlikely to beat LZ4/LZF/ZSTD on bit-shuffled data, but the algorithm targets typed binary arrays
- Storage must be readable outside HDF5 (e.g. plain NumPy .npy or raw binary) — the filter only applies within HDF5 (inside Python, or anywhere via the dynamically loaded filter / h5dump)

## Inputs

- Synthetic or real typed NumPy array with correlated neighbouring elements (e.g. float32 or integer radio-correlator samples)
- h5py/HDF5 file handle with the Bitshuffle dynamically loaded filter plugin available (HDF5 1.8.11+ for the plugin path)

## Outputs

- Chunked HDF5 dataset encoded with Bitshuffle filter 32008 and LZ4 block compression
- Element-wise verification result (read-back array equals original, lossless)
- compression_report recording on-disk storage size and compression ratio (compressed/original as a percentage)

## How to apply

Create a chunked HDF5 dataset with h5py specifying compression=bitshuffle.h5.H5FILTER (filter number 32008) and compression_opts=(block_size, bitshuffle.h5.H5_COMPRESS_LZ4), passing block_size=0 so Bitshuffle chooses its default ~8 kB block tuned for L1 cache. Bitshuffle only rearranges data (a bit-level transpose of the elements-by-bits matrix, vectorized with SSE2/AVX and OpenMP-parallelized); the LZ4 pass then compresses each shuffled block while it is still in L1 cache, which is why LZ4 is integrated directly into Bitshuffle rather than applied as a sequential HDF5 filter. Write the synthetic correlated array, close the file, reopen it, and read the dataset back — decoding through the dynamically loaded filter is transparent when HDF5_PLUGIN_PATH points at the plugin directory. Compare element-wise (e.g. np.array_equal) against the original array to confirm a lossless round trip, then query the on-disk storage size of the chunked dataset to compute the compression ratio (compressed/original as a percentage), expecting well below 100% for correlated data like the CHIME Pathfinder correlator data compressed to 28% in the paper.

## Related tools

- **Bitshuffle** (Provides the H5FILTER (32008) bit-transpose HDF5 filter and Python shuffle/unshuffle routines used to write and read back the chunked dataset) — kiyo-masui/bitshuffle
- **h5py** (Creates the chunked HDF5 dataset with the 32008 filter and compression_opts=(block_size, H5_COMPRESS_LZ4), and reads it back for verification)
- **HDF5** (Chunked storage and the dynamically loaded filters framework through which the Bitshuffle plugin is registered and invoked transparently on read)
- **LZ4** (Fast LZ77-style lossless compressor applied to each bit-shuffled block; integrated directly into Bitshuffle rather than applied as a sequential HDF5 filter)
- **NumPy** (Supplies the typed synthetic test array and the element-wise equality check for the round-trip validation)

## Examples

```
import h5py, numpy as np, bitshuffle.h5; d = arr.shape; f = h5py.File('t.h5','w'); ds = f.create_dataset('x', shape=d, chunks=d, compression=bitshuffle.h5.H5FILTER, compression_opts=(0, bitshuffle.h5.H5_COMPRESS_LZ4)); ds[...] = arr
```

## Evaluation signals

- Read-back array compared with np.array_equal against the original must be exactly True (lossless round trip through filter 32008)
- Reported compression ratio (compressed on-disk size / original array size as a percentage) must be below 100%, consistent with the paper's ~28% for CHIME-like correlated data
- The dataset's HDF5 filter pipeline must report the Bitshuffle filter id 32008 with compression_opts (block_size, H5_COMPRESS_LZ4)
- Closing and reopening the file must not require any manual decode step — reading the dataset transparently returns the original data when the plugin is on HDF5_PLUGIN_PATH
- The bit-transpose runs per ~8 kB block sized for L1 cache; per-chunk (not whole-dataset) shuffle+LZ4 keeps the block cache-resident, so throughput should stay near out-of-cache memory-copy speed (~1 byte/clock, up to 2 on Haswell+)

## Limitations

- Bitshuffle does not compress by itself — it only rearranges bits; a companion LZ77-style compressor (LZF, LZ4, ZSTD) is required, and GZIP/BZIP add little on top of bit-shuffled data
- Compression gains require adjacent elements along the fastest-varying index to be highly correlated; only correlations spanning at least 24 elements can be exploited
- Binary wheels are built with AVX2 and will crash with SIGILL on older/unsupported x86_64 processors; a source build (optionally with OpenMP and --zstd) is needed for other targets
- The dynamically loaded HDF5 filter requires HDF5 1.8.11+ and HDF5_PLUGIN_PATH (or /usr/local/hdf5/lib/plugin); without it the filter works only inside Python after importing bitshuffle.h5

## Evidence

- [readme] HDF5 filter registration: "is integrated through the `dynamically loaded filters`_ framework. Bitshuffle is HDF5 filter number ``32008``."
- [readme] Bitshuffle only rearranges: "This does not in itself compress data, only rearranges it for more efficient compression. To perform the actual compression you will need a compression library."
- [readme] Per-block LZ4 integration: [evidence span withheld — anchor status: external_doc]
- [readme] Correlation requirement: "The Bitshuffle algorithm relies on neighbouring elements of a dataset being highly correlated to improve data compression."
