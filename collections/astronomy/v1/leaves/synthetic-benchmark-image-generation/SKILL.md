---
name: synthetic-benchmark-image-generation
description: Use when you need reproducible synthetic test data with controlled value
  ranges (bit depths N = 7, 15, 23, 31) to benchmark lossless compression ratios and
  read speeds between FITS and HDF5 formats.
when_to_use_negative:
- Your goal is to compare lossy compression algorithms — the HDFITS paper explicitly
  states 'we have not compared lossy compression algorithms', so these synthetic datasets
  only support lossless benchmarks
- You need realistic astronomical data with correlations between neighbouring elements
  — bitshuffle-style compression 'relies on neighbouring elements of a dataset being
  highly correlated', which uniform random integers deliberately lack
- The input is already an existing FITS or MeasurementSet file — use fits2hdf conversion
  (fits2hdf/hdf2fits) instead of generating synthetic arrays
tools:
- name: Python
  role: Used (with NumPy) to instantiate a seeded generator, draw int32 values uniformly
    from (-2^N, 2^N), fill the 2048x2048, 8192x8192 and (10000, 200, 200) arrays,
    and serialize them to .npy/.npz
- name: fits2hdf
  role: 'Downstream consumer of the generated arrays: converts the synthetic images
    into HDFITS (HDF5) format for the compression and read-speed benchmarks, with
    hdf2fits providing the round-trip back to FITS'
  repo: https://github.com/telegraphic/fits2hdf
- name: bitshuffle
  role: Optional HDF5 filter (filter number 32008) applied to the generated arrays
    downstream to improve lossless compression ratios in the benchmark comparison
  repo: https://github.com/kiyo-masui/bitshuffle
- name: hickle
  role: Alternative HDF5 serialization path for the generated NumPy arrays, supporting
    compression options such as gzip/lzf if benchmarking via hickle instead of fits2hdf
  repo: https://github.com/telegraphic/hickle
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2015.05.001
    title: 'HDFITS: Porting the FITS data model to HDF5'
  source_content_hash: sha256:d567fe92b67a4ce6833ec68a3b90da140fd136ab303dc1f7461d317c3f71f9ac
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/telegraphic/fits2hdf
  tools:
  - Python
  - bitshuffle
  - fits2hdf
  - hickle
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# synthetic-benchmark-image-generation

## Summary

Generates deterministic synthetic integer image arrays (2D 2048x2048 and 8192x8192, plus a 3D (10000, 200, 200) read-speed cube) as int32 data drawn from (-2^N, 2^N) for N = 7, 15, 23, 31, so the FITS-vs-HDF5 compression and read benchmarks in the HDFITS paper run on reproducible, well-defined inputs.

## When to use

Use when you need reproducible synthetic test data with controlled value ranges (bit depths N = 7, 15, 23, 31) to benchmark lossless compression ratios and read speeds between FITS and HDF5 formats.

## When NOT to use

- Your goal is to compare lossy compression algorithms — the HDFITS paper explicitly states 'we have not compared lossy compression algorithms', so these synthetic datasets only support lossless benchmarks
- You need realistic astronomical data with correlations between neighbouring elements — bitshuffle-style compression 'relies on neighbouring elements of a dataset being highly correlated', which uniform random integers deliberately lack
- The input is already an existing FITS or MeasurementSet file — use fits2hdf conversion (fits2hdf/hdf2fits) instead of generating synthetic arrays

## Inputs

- Bit-depth specification N ∈ {7, 15, 23, 31} defining the uniform sampling interval (-2^N, 2^N)
- Image shape specification: 2048x2048 and 8192x8192 2D arrays per Table 1
- Read-speed cube shape specification: (10000, 200, 200) with dtype int32
- Random seed for the deterministic NumPy generator

## Outputs

- 2D int32 image arrays of shape 2048x2048 for each N in {7, 15, 23, 31}
- 2D int32 image arrays of shape 8192x8192 for each N in {7, 15, 23, 31}
- 3D int32 array of shape (10000, 200, 200) drawn from (-2^23, 2^23) for the read benchmark
- NumPy binary files (.npy or a single .npz archive) consumed by the downstream compression and read-speed benchmark tasks

## How to apply

Instantiate a seeded NumPy random generator so every run produces identical arrays. For each bit depth N in {7, 15, 23, 31}, draw 32-bit integers uniformly from (-2^N, 2^N) and fill 2D image arrays of shape 2048x2048 and 8192x8192 (per Table 1 of the HDFITS paper). Separately generate the 3D read-speed benchmark array of shape (10000, 200, 200) as int32 drawn uniformly from (-2^23, 2^23), which lets HDF5's chunked reads be compared against FITS reads. Serialize every array to raw NumPy binary files (.npy, or a single .npz archive) so downstream tasks can convert them with fits2hdf and apply HDF5 compression filters (e.g. bitshuffle) without regenerating data. Rationale: bounding values by 2^N makes compression ratios comparable across effective bit depths, since low-N data contain more compressible high-order zero bits.

## Related tools

- **Python** (Used (with NumPy) to instantiate a seeded generator, draw int32 values uniformly from (-2^N, 2^N), fill the 2048x2048, 8192x8192 and (10000, 200, 200) arrays, and serialize them to .npy/.npz)
- **fits2hdf** (Downstream consumer of the generated arrays: converts the synthetic images into HDFITS (HDF5) format for the compression and read-speed benchmarks, with hdf2fits providing the round-trip back to FITS) — https://github.com/telegraphic/fits2hdf
- **bitshuffle** (Optional HDF5 filter (filter number 32008) applied to the generated arrays downstream to improve lossless compression ratios in the benchmark comparison) — https://github.com/kiyo-masui/bitshuffle
- **hickle** (Alternative HDF5 serialization path for the generated NumPy arrays, supporting compression options such as gzip/lzf if benchmarking via hickle instead of fits2hdf) — https://github.com/telegraphic/hickle

## Examples

```
import numpy as np; rng = np.random.default_rng(0); a = rng.integers(-2**23, 2**23, size=(10000,200,200), dtype=np.int32); np.save('read_bench.npy', a)
```

## Evaluation signals

- Every output array has exactly the required shape: 2048x2048, 8192x8192, or (10000, 200, 200)
- Every output array has dtype int32 (or equivalent 32-bit integer dtype such as uint32 where appropriate)
- For each N, min(array) > -2^N and max(array) < 2^N, i.e. values lie strictly within the specified (-2^N, 2^N) bounds
- Re-running the generator with the same seed produces byte-identical arrays (deterministic reproduction)
- Serialized .npy/.npz files load back with identical shape, dtype, and min/max values as the in-memory arrays

## Limitations

- Uniform random integer data lack correlations, so compression results will not reflect correlated (e.g. radio interferometric) data where bitshuffle performs best
- Only lossless compression is exercised; the paper notes 'we have not compared lossy compression algorithms'
- HDF5 currently has no equivalent to the FITS subtractive dithering technique, so dithered-integer compression schemes are out of scope for these benchmarks
- Values are stored as 32-bit integers regardless of the effective bit depth N, so the generator does not emit smaller dtypes

## Evidence

- [methods] Bit-depth bounds: [evidence span withheld — anchor status: not_found]
- [methods] Read-speed cube: [evidence span withheld — anchor status: not_found]
- [methods] Serialization for downstream benchmarks: [evidence span withheld — anchor status: not_found]
