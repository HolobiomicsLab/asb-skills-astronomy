---
name: rounding-error-distribution-validation
description: Use when nearly-losslessly rounding integer visibility/correlator data
  (e.g. CHIME Pathfinder HDF5 visibility sets) and you must verify the rounding noise
  stays negligible relative to thermal noise.
when_to_use_negative:
- Datasets for which no calibration-independent thermal noise estimate (Equation 14
  with Equation 4/5-7 covariance) can be computed, since the granularity limit cannot
  be justified
- Applications requiring exact lossless recovery of the original integer data — rounding
  is intentionally lossy (nearly lossless) by design
- Data already compressed or entropy-reduced by a prior transform, where applying
  the radiometer-relative rounding again would double-count noise budget
tools:
- name: Bitshuffle
  role: Authors' released example implementation of the compression scheme used to
    cross-check the chosen granularity and rounded values
  repo: https://github.com/kiyo-masui/bitshuffle
- name: HDF5
  role: Data format/library used with h5py to load the visibility dataset before rounding
- name: numpy
  role: Provides numpy.rint for nearest-multiple rounding with ties-to-even after
    scaling by 1/g
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2015.07.002
    title: A compression scheme for radio data in high performance computing
  source_content_hash: sha256:fdf21c6d2228f3a24c50106dc8a1ae967ed4505d1de4afe457b7c9be4f8b725e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/kiyo-masui/bitshuffle
  tools:
  - Bitshuffle
  - CHIME Pathfinder
  - HDF5
  - numpy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# rounding-error-distribution-validation

## Summary

Validates the paper's controlled precision-reduction step by checking that rounding integer radio data to a power-of-two granularity (derived from the radiometer-equation noise scale) produces an unbiased error distribution with errors bounded by half the maximum granularity.

## When to use

Use when nearly-losslessly rounding integer visibility/correlator data (e.g. CHIME Pathfinder HDF5 visibility sets) and you must verify the rounding noise stays negligible relative to thermal noise.

## When NOT to use

- Datasets for which no calibration-independent thermal noise estimate (Equation 14 with Equation 4/5-7 covariance) can be computed, since the granularity limit cannot be justified
- Applications requiring exact lossless recovery of the original integer data — rounding is intentionally lossy (nearly lossless) by design
- Data already compressed or entropy-reduced by a prior transform, where applying the radiometer-relative rounding again would double-count noise budget

## Inputs

- CHIME Pathfinder visibility dataset (shape (64, 136, 1024); each element a struct of two 32-bit signed little-endian integers, time fastest-varying; 68 MiB total) read via h5py/HDF5
- Per-element thermal noise scale s estimated from Equation 14 and the Equation 4 (or 5-7) covariance matrix
- Acceptable fractional increase in noise variance f (recommended 10^-5 > f > 10^-2)

## Outputs

- Maximum rounding granularity g (largest permissible power of two)
- Precision-reduced (rounded) array written to rounded_data.npy
- Rounding-error array (original minus rounded) with verified symmetric, bounded distribution

## How to apply

First estimate the thermal noise s on a data-element-by-data-element basis using Equation 14 together with the calibration-independent noise covariance of Equation 4 (or Equations 5-7). Fix the acceptable fractional increase in noise variance at f (recommended range 10^-5 > f > 10^-2) and derive the maximum rounding granularity from the constraint sigma_r^2 < f*s^2 in Equations 15-16. Select the largest power of two not exceeding that limit and round every element to the nearest multiple of it, breaking ties to even (e.g. numpy.rint on values scaled by 1/g) so the rounding is unbiased. Finally, compute rounding errors by direct subtraction of the original dataset from the precision-reduced dataset and verify the error probability density is symmetric about zero and that no error exceeds half the granularity bound.

## Related tools

- **Bitshuffle** (Authors' released example implementation of the compression scheme used to cross-check the chosen granularity and rounded values) — https://github.com/kiyo-masui/bitshuffle
- **HDF5** (Data format/library used with h5py to load the visibility dataset before rounding)
- **numpy** (Provides numpy.rint for nearest-multiple rounding with ties-to-even after scaling by 1/g)

## Examples

```
import numpy as np; rounded = (np.rint(vis.astype(np.float64) / g) * g).astype(vis.dtype); np.save('rounded_data.npy', rounded)
```

## Evaluation signals

- Rounding-error probability density is symmetric about zero (errors are unbiased)
- No rounding error exceeds half the maximum granularity in absolute value, consistent with the sqrt(12 f s^2) variance bound
- Selected granularity g is a power of two and is the largest one satisfying sigma_r^2 < f*s^2 (Equations 15-16) for the chosen f
- Chosen granularity and rounded values match the authors' released implementation in the kiyo-masui/bitshuffle repository
- Round-tripped output array saved as rounded_data.npy reconstructs the expected multiples of g

## Limitations

- The granularity must be a power of two, so the achieved rounding noise may be well below the f-budget if the permissible limit falls between powers of two
- Validity depends on the noise estimate s being calibration-independent (radiometer-equation based); miscalibrated noise invalidates the error bound
- Ties-to-even is required for unbiasedness; biased tie-breaking shifts the error distribution off zero
- f should lie in the recommended range 10^-5 > f > 10^-2; values outside this range trade storage savings against noise degradation

## Evidence

- [methods] methods: granularity limit: [evidence span withheld — anchor status: not_found]
