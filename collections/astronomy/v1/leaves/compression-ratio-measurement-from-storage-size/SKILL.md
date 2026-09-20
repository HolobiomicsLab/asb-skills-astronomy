---
name: compression-ratio-measurement-from-storage-size
description: Use when a chunked HDF5 dataset has been written through an HDF5 filter
  (e.g. Bitshuffle filter 32008 with LZ4) and you need to report how much on-disk
  space the filtered data occupies relative to the raw typed array.
when_to_use_negative:
- The data is uncorrelated or already entropy-coded (random values, existing compressed
  stream) — Bitshuffle relies on highly correlated neighbouring elements, so the ratio
  may exceed 100%
- You intend to read the ratio from the HDF5 file's total byte size rather than the
  specific dataset's chunk storage size, which conflates other datasets, metadata,
  and alignment padding
- You need a general-purpose compressor benchmark (e.g. against GZIP, BZIP2, snappy)
  rather than measuring the Bitshuffle+LZ4 filter pipeline on typed binary data
edam_operation: http://edamontology.org/operation_2409
tools:
- name: h5py
  role: Creates the chunked dataset with the Bitshuffle filter options and exposes
    ds.id.get_storage_size() for querying the on-disk storage size
- name: Bitshuffle
  role: Registers as HDF5 filter 32008 and performs the bit-transpose plus LZ4 compression
    whose output size is measured
  repo: https://github.com/kiyo-masui/bitshuffle
- name: HDF5
  role: Chunked-storage filter pipeline framework that tracks how much compressed
    data each chunk occupies on disk
- name: NumPy
  role: Supplies the typed array, its nbytes baseline, and the element-wise equality
    check for lossless verification
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2015.07.002
    title: A compression scheme for radio data in high performance computing
  source_content_hash: sha256:079632741f818fd74e9d2e43381e5df990171410bd85a71af3a442797c095a13
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# compression-ratio-measurement-from-storage-size

## Summary

Quantifies the effectiveness of a Bitshuffle+LZ4 HDF5 filter pipeline by querying the on-disk storage size of a chunked dataset and expressing it as a percentage of the original array size, alongside a lossless read-back verification.

## When to use

Use when a chunked HDF5 dataset has been written through an HDF5 filter (e.g. Bitshuffle filter 32008 with LZ4) and you need to report how much on-disk space the filtered data occupies relative to the raw typed array.

## When NOT to use

- The data is uncorrelated or already entropy-coded (random values, existing compressed stream) — Bitshuffle relies on highly correlated neighbouring elements, so the ratio may exceed 100%
- You intend to read the ratio from the HDF5 file's total byte size rather than the specific dataset's chunk storage size, which conflates other datasets, metadata, and alignment padding
- You need a general-purpose compressor benchmark (e.g. against GZIP, BZIP2, snappy) rather than measuring the Bitshuffle+LZ4 filter pipeline on typed binary data

## Inputs

- Synthetic or real typed NumPy array with correlated adjacent elements (e.g. integer or float32 radio data)
- h5py chunked HDF5 dataset created with bitshuffle.h5.H5FILTER and H5_COMPRESS_LZ4 compression options
- Original array in memory for the round-trip equality check and nbytes denominator

## Outputs

- Compression ratio (compressed on-disk size / original size as a percentage)
- Boolean lossless-verification result (read-back array exactly equals original)
- compression_report recording ratio and verification status

## How to apply

Write the typed NumPy array to a chunked HDF5 dataset with compression=bitshuffle.h5.H5FILTER (filter number 32008) and compression_opts=(block_size, bitshuffle.h5.H5_COMPRESS_LZ4), passing block_size=0 so Bitshuffle chooses its default ~8 kB block. Close and reopen the file so the reported storage reflects what was actually serialized, then query the dataset's on-disk storage size (h5py's ds.id.get_storage_size()) rather than the file size, since a file can contain multiple datasets. Compute the ratio as (compressed storage size / original array nbytes) * 100%; the paper reports ~28% for CHIME Pathfinder correlator data, so anything below 100% is consistent while ~28% is the reference point for radio-telescope-like correlated data. Pair the ratio with an element-wise read-back comparison (reopened dataset == original array) to confirm the compression was lossless, and record both in a compression report.

## Related tools

- **h5py** (Creates the chunked dataset with the Bitshuffle filter options and exposes ds.id.get_storage_size() for querying the on-disk storage size)
- **Bitshuffle** (Registers as HDF5 filter 32008 and performs the bit-transpose plus LZ4 compression whose output size is measured) — https://github.com/kiyo-masui/bitshuffle
- **HDF5** (Chunked-storage filter pipeline framework that tracks how much compressed data each chunk occupies on disk)
- **NumPy** (Supplies the typed array, its nbytes baseline, and the element-wise equality check for lossless verification)

## Examples

```
import h5py, numpy, bitshuffle.h5; f=h5py.File('t.h5','w'); ds=f.create_dataset('d',data=arr,chunks=arr.shape,compression=bitshuffle.h5.H5FILTER,compression_opts=(0,bitshuffle.h5.H5_COMPRESS_LZ4)); ratio=ds.id.get_storage_size()/arr.nbytes*100
```

## Evaluation signals

- The read-back array from the reopened HDF5 file must exactly equal the original array (lossless round trip)
- The reported compression ratio must be below 100%, i.e. the filtered dataset actually occupies less on-disk space than the raw array
- For CHIME-like correlated radio data the ratio should approach the paper's reported ~28% of original size
- The size denominator must be the dataset's own on-disk storage (e.g. ds.id.get_storage_size()), not the enclosing file size
- The dataset must be chunked (not contiguous); with a contiguous layout the filter pipeline never executes per chunk and the measurement is meaningless

## Limitations

- Bitshuffle does not itself compress — it only rearranges bits for a downstream LZ77-style compressor (LZ4/ZSTD/LZF), so the measured ratio depends on the paired compressor
- Compression depends on neighbouring dataset elements being highly correlated; correlations spanning at least 24 elements can be exploited, weaker structure yields poor ratios
- Bitshuffle's block-oriented transpose (blocks chosen to fit L1 cache, ~8 kB) means per-chunk compression differs from applying Bitshuffle to the full dataset then compressing

## Evidence

- [methods] workflow sub-step: bit transpose: [evidence span withheld — anchor status: not_found]
