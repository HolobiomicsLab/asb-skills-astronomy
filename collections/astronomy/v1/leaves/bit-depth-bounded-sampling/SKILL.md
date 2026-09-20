---
name: bit-depth-bounded-sampling
description: Use when building benchmark datasets to compare lossless compression
  ratios or I/O read speeds across file formats (FITS vs HDF5), and you need controlled
  bit-depth content ranges that are reproducible across runs.
when_to_use_negative:
- When the benchmark needs realistic astronomical data with spatial correlations —
  random integers give worst-case compression, and the paper notes bitshuffle-style
  compression 'relies on neighbouring elements of a dataset being highly correlated'
  to improve compression
- When evaluating lossy compression — the paper explicitly states 'we have not compared
  lossy compression algorithms'
- When the goal is simply converting existing FITS files to HDFITS — use the fits2hdf/hdf2fits
  conversion tools instead of generating synthetic data
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Python
  role: Used with NumPy's seeded random generator to draw uniform 32-bit integers
    within (-2^N, 2^N) bounds and to serialize arrays to .npy/.npz
- name: fits2hdf
  role: 'Downstream consumer: the generated benchmark arrays feed compression and
    read-speed comparisons between FITS and the HDFITS (HDF5) format via fits2hdf
    conversion'
  repo: https://github.com/telegraphic/fits2hdf
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2015.05.001
    title: 'HDFITS: Porting the FITS data model to HDF5'
  source_content_hash: sha256:81afccba3fcc1e09f2c7222ade3e02047772538453127f927fdf02f852804b41
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/telegraphic/fits2hdf
  tools:
  - Python
  - fits2hdf
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# bit-depth-bounded-sampling

## Summary

Generates deterministic, reproducible synthetic test images of random 32-bit integers drawn from bounded ranges (-2^N, 2^N) for N = 7, 15, 23, 31, used to benchmark lossless compression ratios and read speeds between FITS and HDF5 (HDFITS) formats.

## When to use

Use when building benchmark datasets to compare lossless compression ratios or I/O read speeds across file formats (FITS vs HDF5), and you need controlled bit-depth content ranges that are reproducible across runs.

## When NOT to use

- When the benchmark needs realistic astronomical data with spatial correlations — random integers give worst-case compression, and the paper notes bitshuffle-style compression 'relies on neighbouring elements of a dataset being highly correlated' to improve compression
- When evaluating lossy compression — the paper explicitly states 'we have not compared lossy compression algorithms'
- When the goal is simply converting existing FITS files to HDFITS — use the fits2hdf/hdf2fits conversion tools instead of generating synthetic data

## Inputs

- Seed value for the NumPy random generator (for reproducibility)
- List of bit depths N in {7, 15, 23, 31}
- Target array shapes: 2D images of 2048x2048 and 8192x8192; 3D read-benchmark cube (10000, 200, 200)

## Outputs

- .npy or .npz files containing int32 arrays of shape 2048x2048 for each bit depth N
- .npy or .npz files containing int32 arrays of shape 8192x8192 for each bit depth N
- .npy file containing an int32 array of shape (10000, 200, 200) drawn from (-2^23, 2^23) for read-speed benchmarking

## How to apply

Instantiate a seeded NumPy random generator so every run reproduces identical arrays. For each bit depth N in {7, 15, 23, 31}, draw 32-bit integers uniformly from (-2^N, 2^N) and fill 2D image arrays of shape 2048x2048 and 8192x8192 (per Table 1 of the HDFITS paper). Additionally generate the 3D read-speed benchmark array of shape (10000, 200, 200) with dtype int32 drawn from (-2^23, 2^23), matching the paper's finding that HDFITS can read data up to 100x faster than FITS. Serialize each array to raw NumPy binaries (.npy or .npz) for downstream compression and read-speed benchmark tasks. The rationale is that bounding values to specific bit depths lets you measure how compression filters (e.g. shuffle/bitshuffle-style filters) exploit the unused high bits, giving controlled, comparable compression ratios across formats.

## Related tools

- **Python** (Used with NumPy's seeded random generator to draw uniform 32-bit integers within (-2^N, 2^N) bounds and to serialize arrays to .npy/.npz)
- **fits2hdf** (Downstream consumer: the generated benchmark arrays feed compression and read-speed comparisons between FITS and the HDFITS (HDF5) format via fits2hdf conversion) — https://github.com/telegraphic/fits2hdf

## Examples

```
import numpy as np; rng = np.random.default_rng(42); a = rng.integers(-2**23 + 1, 2**23 - 1, size=(10000, 200, 200), dtype=np.int32); np.save('readbench_n23.npy', a)
```

## Evaluation signals

- Every output array has the exact required shape: 2048x2048, 8192x8192, or (10000, 200, 200)
- Every output array has dtype int32 (or equivalent 32-bit integer dtype)
- For each array, min and max values lie strictly within the assigned (-2^N, 2^N) bounds
- Re-running with the same seed reproduces bit-identical arrays (determinism check)
- Downstream benchmark reads the .npy/.npz files without reshaping or dtype coercion

## Limitations

- Uniform random integers are uncorrelated, so they represent a worst case for compression filters that exploit element correlation (e.g. bitshuffle)
- The paper's read-speed and compression conclusions are tied to the specific shapes (2048x2048, 8192x8192) and bit depths (N = 7, 15, 23, 31); other configurations were not characterized
- Full characterization of the effect of data access pattern upon read performance was beyond the scope of the paper, so the 3D benchmark does not cover all access patterns

## Evidence

- [other] For each bit depth N in {7, 15, 23, 31}, draw 32-bit integers uniformly from the interval (-2^N, 2^N) and fill 2D image arrays of shape 2048x2048 and 8192x8192.: [evidence span withheld — anchor status: not_found]
- [other] Generate the 3D read-speed benchmark array of shape (10000, 200, 200) with dtype int32, drawing values uniformly from (-2^23, 2^23).: [evidence span withheld — anchor status: not_found]
- [abstract] HDFITS allows faster reading of data, up to 100x of FITS in some use cases, and improved compression (higher compression ratios and higher throughput): "We show that HDFITS allows faster reading of data (up to 100x of FITS in some use cases), and improved compression (higher compression ratios and higher throughput)"
- [readme] Bitshuffle relies on correlated neighbouring elements, relevant because the synthetic data are uniform random integers: [evidence span withheld — anchor status: external_doc]
- [other] Instantiate a deterministic (seeded) NumPy random generator so every run reproduces identical arrays.: [evidence span withheld — anchor status: not_found]
