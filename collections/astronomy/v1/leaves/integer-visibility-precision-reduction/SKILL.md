---
name: integer-visibility-precision-reduction
description: Use when handling large integer visibility datasets (e.g. CHIME Pathfinder
  correlator output) where information entropy must be reduced before lossless compression
  without measurably increasing thermal noise.
when_to_use_negative:
- Data elements are uncorrelated or not integrated/post-correlation radio data — the
  radiometer-equation noise model of Equations 4-7 and 14 does not apply
- The task requires strictly lossless preservation of the original integers — the
  rounding step is 'nearly lossless' and irreversibly discards low-order bits
- Storage compression alone is the goal and precision reduction is unnecessary — the
  paper notes the two steps are independent and Bitshuffle can be used without rounding
edam_operation: http://edamontology.org/operation_3281
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: Bitshuffle
  role: 'Reference implementation context: the precision-reduction step reduces entropy
    so that the subsequent Bitshuffle lossless compression (HDF5 filter 32008) achieves
    higher ratios; example implementation lives in the kiyo-masui/bitshuffle repository'
  repo: https://github.com/kiyo-masui/bitshuffle
- name: HDF5
  role: Data format and library through which the visibility data are read and the
    compression scheme is applied
- name: CHIME Pathfinder
  role: Source telescope of the integer visibility dataset used to validate the rounding
    procedure
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2015.07.002
    title: A compression scheme for radio data in high performance computing
  source_content_hash: sha256:8e21174197edb395aaf0513b64ec2f1362c90295e235dbb546c2459c9cf8da66
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/kiyo-masui/bitshuffle
  tools:
  - Bitshuffle
  - CHIME Pathfinder
  - HDF5
  edam_operation: http://edamontology.org/operation_3281
  edam_topics:
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# integer-visibility-precision-reduction

## Summary

A nearly lossless rounding procedure that reduces the precision of integrated radio-interferometric data by rounding each element to a multiple of the largest power-of-two granularity permitted by a calibration-independent form of the radiometer equation, so that induced rounding noise stays a negligible, user-chosen fraction of the thermal noise.

## When to use

Use when handling large integer visibility datasets (e.g. CHIME Pathfinder correlator output) where information entropy must be reduced before lossless compression without measurably increasing thermal noise.

## When NOT to use

- Data elements are uncorrelated or not integrated/post-correlation radio data — the radiometer-equation noise model of Equations 4-7 and 14 does not apply
- The task requires strictly lossless preservation of the original integers — the rounding step is 'nearly lossless' and irreversibly discards low-order bits
- Storage compression alone is the goal and precision reduction is unnecessary — the paper notes the two steps are independent and Bitshuffle can be used without rounding

## Inputs

- CHIME Pathfinder correlator visibility dataset (shape (64, 136, 1024); each element a struct of two 32-bit signed little-endian integers, time fastest-varying; 68 MiB), stored in HDF5 and read via h5py
- Noise covariance matrix parameters from Equations 4-7 (calibration-independent radiometer-equation inputs)
- Choice of acceptable fractional noise-variance increase f (e.g. 10^-3)

## Outputs

- Precision-reduced (rounded) integer array written to rounded_data.npy
- Element-wise rounding-error array (original minus rounded) used for validation
- Chosen power-of-two granularity g consistent with sigma_r^2 < f*s^2

## How to apply

Load the integer visibility data (CHIME Pathfinder example: shape (64, 136, 1024), each element a struct of two 32-bit signed little-endian integers, 68 MiB total) via h5py/HDF5. Estimate the thermal noise scale s on a data-element-by-data-element basis using Equation 14 together with the noise covariance matrix of Equation 4 (or Equations 5-7), which is calibration-independent. Choose an acceptable fractional increase in noise variance f (recommended 10^-5 > f > 10^-2; the paper's example uses f = 10^-3) and derive the maximum rounding granularity from the constraint sigma_r^2 < f*s^2 in Equations 15-16. Round every data element to the nearest multiple of the largest power of two not exceeding that limit, breaking ties to even (e.g. numpy.rint on values scaled by 1/g), which keeps the rounding unbiased. Verify by subtracting the original data from the rounded data that errors are symmetric about zero and that no error exceeds half the granularity bound sqrt(12 f s^2).

## Related tools

- **Bitshuffle** (Reference implementation context: the precision-reduction step reduces entropy so that the subsequent Bitshuffle lossless compression (HDF5 filter 32008) achieves higher ratios; example implementation lives in the kiyo-masui/bitshuffle repository) — https://github.com/kiyo-masui/bitshuffle
- **HDF5** (Data format and library through which the visibility data are read and the compression scheme is applied)
- **CHIME Pathfinder** (Source telescope of the integer visibility dataset used to validate the rounding procedure)

## Examples

```
g = largest power of two with g**2 <= 12*f*s**2; rounded = (np.rint(data / g) * g).astype(data.dtype)  # f=1e-3, s from Eq.14 per element
```

## Evaluation signals

- Rounding-error distribution (original minus rounded) is symmetric about zero (unbiased probability density)
- No rounding error exceeds half the maximum granularity sqrt(12 f s^2) in absolute value
- Chosen granularity g is the largest power of two satisfying sigma_r^2 < f*s^2 for the per-element noise estimate s
- Fractional increase in noise variance stays within the chosen f (paper example f = 10^-3, within recommended 10^-5 > f > 10^-2)
- Granularity and rounded values match the authors' released implementation in kiyo-masui/bitshuffle

## Limitations

- Rounding is 'nearly lossless' rather than lossless: low-order bits are irreversibly discarded, so downstream analyses needing exact integer values must use the original data
- The noise estimate relies on the radiometer-equation covariance (Equations 4-7, 14); if the instrument noise deviates from that model, the granularity bound may be inappropriate
- The choice of f trades compression ratio against added noise; recommended range is 10^-5 > f > 10^-2, and values outside it are not validated
- The benefit assumes adjacent/fast-varying elements are highly correlated; Bitshuffle itself relies on correlations spanning at least 24 elements and needs SSE2/AVX-class x86 processors for the vectorized bit-transpose

## Evidence

- [abstract] Integrated, post-correlation data are first passed through a nearly lossless rounding step: [evidence span withheld — anchor status: not_found]
- [methods] rounding to power-of-two granularity: [evidence span withheld — anchor status: not_found]
- [methods] CHIME Pathfinder dataset provenance: [evidence span withheld — anchor status: not_found]
