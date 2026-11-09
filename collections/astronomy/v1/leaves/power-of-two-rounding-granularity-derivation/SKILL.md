---
name: power-of-two-rounding-granularity-derivation
description: Use when compressing correlated integer visibility/correlator data (e.g.
  CHIME Pathfinder HDF5 visibilities) and you need to quantize values to a power-of-two
  grid without meaningfully raising thermal noise.
when_to_use_negative:
- Data are floating-point or already low-entropy, so the rounding step offers no entropy
  reduction and only destroys precision
- Correlations between adjacent elements are weak, so precision reduction will not
  improve downstream lossless compression ratios enough to justify the noise increase
- You cannot estimate thermal noise per element (no calibration-independent radiometer-equation
  covariance available), making the sigma_r^2 < f*s^2 constraint inapplicable
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Bitshuffle
  role: Reference implementation whose kiyo-masui/bitshuffle repository contains the
    authors' released example of the precision-reduction (nearly lossless rounding)
    step, against which the chosen granularity and rounded values are compared
  repo: https://github.com/kiyo-masui/bitshuffle
- name: HDF5
  role: Data format/library used to load the CHIME Pathfinder visibility dataset (via
    h5py) before rounding
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2015.07.002
    title: A compression scheme for radio data in high performance computing
  source_content_hash: sha256:c99a9810594336fc126a4fc4a37aa021bbfd2555ca79546a015c0e814e498184
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/kiyo-masui/bitshuffle
  tools:
  - Bitshuffle
  - CHIME Pathfinder
  - HDF5
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# power-of-two-rounding-granularity-derivation

## Summary

Derives the largest power-of-two rounding granularity g for integer radio data such that the induced rounding noise variance stays below a chosen fraction f of the element-wise thermal noise estimated from a calibration-independent radiometer equation (Eqs. 14-16), enabling nearly lossless precision reduction before lossless compression.

## When to use

Use when compressing correlated integer visibility/correlator data (e.g. CHIME Pathfinder HDF5 visibilities) and you need to quantize values to a power-of-two grid without meaningfully raising thermal noise.

## When NOT to use

- Data are floating-point or already low-entropy, so the rounding step offers no entropy reduction and only destroys precision
- Correlations between adjacent elements are weak, so precision reduction will not improve downstream lossless compression ratios enough to justify the noise increase
- You cannot estimate thermal noise per element (no calibration-independent radiometer-equation covariance available), making the sigma_r^2 < f*s^2 constraint inapplicable

## Inputs

- CHIME Pathfinder visibility dataset stored as HDF5 (shape (64, 136, 1024); each element a struct of two 32-bit signed little-endian integers, time fastest-varying; 68 MiB total)
- Element-wise thermal noise estimates s from the radiometer-equation covariance (Equation 14 with Equation 4 or Equations 5-7)
- Chosen acceptable fractional increase in noise variance f

## Outputs

- Largest permissible power-of-two rounding granularity g
- Precision-reduced (rounded) integer array written to rounded_data.npy
- Rounding-error statistics (symmetry about zero, max absolute error vs sqrt(12 f s^2))

## How to apply

Estimate the thermal noise scale s on a data-element-by-data-element basis using Equation 14 together with the noise covariance matrix defined by Equation 4 (or Equations 5-7), which is calibration-independent. Fix the acceptable fractional increase in noise variance at f (recommended range 10^-5 > f > 10^-2; the CHIME Pathfinder example uses f = 10^-3) and apply the constraint sigma_r^2 < f*s^2 from Equations 15-16 to derive the maximum rounding granularity. Select the largest power of two not exceeding that limit and round every element to the nearest multiple of it, breaking ties by rounding to even (e.g. via numpy.rint on values scaled by 1/g) so the rounding noise is unbiased. Verify by subtracting the original from the rounded dataset: the error distribution should be symmetric about zero and no error should exceed half the granularity implied by sqrt(12 f s^2) in absolute value.

## Related tools

- **Bitshuffle** (Reference implementation whose kiyo-masui/bitshuffle repository contains the authors' released example of the precision-reduction (nearly lossless rounding) step, against which the chosen granularity and rounded values are compared) — https://github.com/kiyo-masui/bitshuffle
- **HDF5** (Data format/library used to load the CHIME Pathfinder visibility dataset (via h5py) before rounding)

## Examples

```
g = 2**int(np.floor(np.log2(np.sqrt(f * s**2 / 12.0)))); rounded = np.rint(data.astype(np.float64) / g).astype(np.int32) * g; np.save('rounded_data.npy', rounded)
```

## Evaluation signals

- Rounding-error probability density is symmetric about zero, confirming unbiased rounding (ties-to-even)
- No rounding error exceeds half the maximum granularity sqrt(12 f s^2) in absolute value
- Chosen g is the largest power of two satisfying sigma_r^2 < f*s^2 from Equations 15-16
- Rounded values match the authors' released example implementation in kiyo-masui/bitshuffle
- Realized fractional increase in noise variance stays within the recommended range 10^-5 > f > 10^-2 (f = 10^-3 in the worked example)

## Limitations

- The constraint relies on a calibration-independent radiometer-equation noise estimate; if the assumed covariance (Eq. 4 / Eqs. 5-7) mis-models the data, rounding noise may exceed the intended fraction of thermal noise
- Bitshuffle relies on neighbouring elements being highly correlated; correlations spanning fewer than at least 24 elements cannot be exploited, so rounding gains may not translate into compression
- Granularity is limited to powers of two, so the achieved noise increase is bounded by the largest power of two below the constraint, which may be much smaller than the f*s^2 allowance allows

## Evidence

- [methods] Round all data to a multiple of the largest possible power of two subject to Equation 16.: [evidence span withheld — anchor status: not_found]
