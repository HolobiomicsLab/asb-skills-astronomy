---
name: uniform-integer-array-generation
description: Use when you need reproducible synthetic integer test data of known bit-depth
  occupancy (e.g. 2048x2048, 8192x8192 images or a (10000, 200, 200) cube) to benchmark
  storage formats or compression filters.
when_to_use_negative:
- The benchmark needs lossy compression comparison — the paper explicitly states 'we
  have not compared lossy compression algorithms', so synthetic data tuned for lossless
  ratio/read benchmarks should not be presented as covering lossy codecs.
- The task requires realistic astronomical signal (e.g. correlated interferometric
  data for bitshuffle-style filters); uniform random integers lack the element-to-element
  correlations bitshuffle relies on.
- Input is already existing observational data files (FITS/MeasurementSet) — use the
  fits2hdf conversion path instead of synthetic generation.
tools:
- name: Python
  role: NumPy-based generation of seeded uniform int32 arrays with the specified shapes
    and (-2^N, 2^N) bounds
- name: fits2hdf
  role: 'downstream consumer context: generated arrays feed the FITS vs HDFITS compression
    and read-speed benchmarks'
  repo: https://github.com/telegraphic/fits2hdf
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2015.05.001
    title: 'HDFITS: Porting the FITS data model to HDF5'
  source_content_hash: sha256:a86a7f49d41e48b38b436d54f76a5c6f5597c73de6984dcc9d2f783b9413620e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/telegraphic/fits2hdf
  tools:
  - Python
  - fits2hdf
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# uniform-integer-array-generation

## Summary

Generate deterministic synthetic 2D and 3D int32 image/dataset arrays with values drawn uniformly from (-2^N, 2^N) for N in {7, 15, 23, 31}, used as controlled inputs to benchmark FITS vs HDF5 (HDFITS) compression ratios and read speeds.

## When to use

Use when you need reproducible synthetic integer test data of known bit-depth occupancy (e.g. 2048x2048, 8192x8192 images or a (10000, 200, 200) cube) to benchmark storage formats or compression filters.

## When NOT to use

- The benchmark needs lossy compression comparison — the paper explicitly states 'we have not compared lossy compression algorithms', so synthetic data tuned for lossless ratio/read benchmarks should not be presented as covering lossy codecs.
- The task requires realistic astronomical signal (e.g. correlated interferometric data for bitshuffle-style filters); uniform random integers lack the element-to-element correlations bitshuffle relies on.
- Input is already existing observational data files (FITS/MeasurementSet) — use the fits2hdf conversion path instead of synthetic generation.

## Inputs

- seed value for the deterministic NumPy random generator
- list of bit depths N in {7, 15, 23, 31} defining the (-2^N, 2^N) sampling bounds
- target array shapes: 2048x2048, 8192x8192 (2D images) and (10000, 200, 200) (3D read benchmark)

## Outputs

- 2D int32 image arrays of shape 2048x2048 and 8192x8192 for each bit depth N
- 3D int32 read-benchmark array of shape (10000, 200, 200) sampled from (-2^23, 2^23)
- serialized .npy files (or a single .npz archive) consumed by downstream compression and read-speed benchmarks

## How to apply

Instantiate a seeded NumPy random generator so every run produces identical arrays. For each bit depth N in {7, 15, 23, 31}, draw 32-bit integers uniformly from the open interval (-2^N, 2^N) and fill 2D image arrays of shape 2048x2048 and 8192x8192 (matching Table 1 sizes). Separately generate the read-speed benchmark array of shape (10000, 200, 200) with dtype int32, sampling uniformly from (-2^23, 2^23). Serialize each array to raw NumPy binary files (.npy or a single .npz archive) so downstream compression and read-speed tasks consume byte-identical inputs. The varying bit depth is the rationale: it lets compression ratios be compared as a function of how many bits of each int32 sample actually carry information, mirroring the HDFITS claim of higher compression ratios and throughput over FITS.

## Related tools

- **Python** (NumPy-based generation of seeded uniform int32 arrays with the specified shapes and (-2^N, 2^N) bounds)
- **fits2hdf** (downstream consumer context: generated arrays feed the FITS vs HDFITS compression and read-speed benchmarks) — https://github.com/telegraphic/fits2hdf

## Examples

```
import numpy as np; rng = np.random.default_rng(42); arr = rng.integers(-2**23, 2**23, size=(10000,200,200), dtype='int32'); np.save('readbench_int23.npy', arr)
```

## Evaluation signals

- Every output array has the exact required shape: 2048x2048, 8192x8192, or (10000, 200, 200).
- Every output array has dtype int32/uint32.
- For each array assigned bit depth N, min and max values lie strictly within the specified (-2^N, 2^N) bounds.
- Re-running with the same seed reproduces byte-identical arrays (deterministic generation).
- All generated arrays are serialized to .npy/.npz and load back with identical values before being handed to the compression/read benchmarks.

## Limitations

- Uniform random integers are information-dense and lack spatial correlation, so compression gains measured on them will not transfer to correlated radio-astronomy data (e.g. bitshuffle is designed for data where 'neighbouring elements of a dataset [are] highly correlated').
- Values are stored as 32-bit integers even for low bit depths (N=7, 15), so raw file sizes overstate information content; compression-ratio comparisons depend on this deliberate padding.
- Only 2D image sizes (2048x2048, 8192x8192 per Table 1) and one 3D read array are covered; other shapes or access patterns are out of scope ('full characterization of the effect of data access pattern upon read performance is beyond the scope of the paper').

## Evidence

- [other] benchmark datasets bounds: [evidence span withheld — anchor status: not_found]
- [readme] hickle numpy arrays: [evidence span withheld — anchor status: not_found]
