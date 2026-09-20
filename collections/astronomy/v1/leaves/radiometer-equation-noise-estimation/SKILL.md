---
name: radiometer-equation-noise-estimation
description: Use when compressing correlated radio-astronomy visibility data (e.g.
  CHIME Pathfinder HDF5 visibility stacks) and you must bound rounding-induced noise
  to a chosen fraction f of the radiometer-equation thermal noise before lossless
  compression.
when_to_use_negative:
- Data whose adjacent elements are not highly correlated over the fastest-varying
  index — Bitshuffle and the precision-reduction scheme both rely on neighbouring-element
  correlation for compression gains
- Use cases requiring strictly lossless compression of the raw integer data — the
  rounding step is only 'nearly lossless' and irreversibly reduces precision
- Data lacking the integration/calibration metadata needed to evaluate the radiometer-equation
  noise (Equation 14) on a per-element basis
tools:
- name: h5py/HDF5
  role: Read the CHIME Pathfinder visibility dataset whose per-element noise scale
    is estimated before rounding
- name: Bitshuffle
  role: Reference implementation in the kiyo-masui/bitshuffle repository against which
    the chosen granularity and rounded values are compared
  repo: https://github.com/kiyo-masui/bitshuffle
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2015.07.002
    title: A compression scheme for radio data in high performance computing
  source_content_hash: sha256:9ccbda3a4d4e24ef6d8ceed70b5c3253e898851dffc14149c403c75e9d6db2ad
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/kiyo-masui/bitshuffle
  tools:
  - Bitshuffle
  - CHIME Pathfinder
  - h5py/HDF5
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# radiometer-equation-noise-estimation

## Summary

Estimates thermal noise per data element using the paper's generalized, calibration-independent form of the radiometer equation (Equation 14 with the noise covariance of Equations 4-7), which sets the granularity limit for the nearly-lossless rounding step of the radio-data compression scheme.

## When to use

Use when compressing correlated radio-astronomy visibility data (e.g. CHIME Pathfinder HDF5 visibility stacks) and you must bound rounding-induced noise to a chosen fraction f of the radiometer-equation thermal noise before lossless compression.

## When NOT to use

- Data whose adjacent elements are not highly correlated over the fastest-varying index — Bitshuffle and the precision-reduction scheme both rely on neighbouring-element correlation for compression gains
- Use cases requiring strictly lossless compression of the raw integer data — the rounding step is only 'nearly lossless' and irreversibly reduces precision
- Data lacking the integration/calibration metadata needed to evaluate the radiometer-equation noise (Equation 14) on a per-element basis

## Inputs

- CHIME Pathfinder correlator visibility data (Jan 25 2015, 68 MiB) stored in HDF5 via h5py; shape (64, 136, 1024) with each element a struct of two 32-bit signed little-endian integers, time fastest-varying
- Choice of fractional noise-variance increase f (e.g. 10^-3, within recommended range 10^-5 > f > 10^-2)
- Radiometer-equation parameters needed for Equation 14 and the covariance matrix of Equation 4 (or Equations 5-7)

## Outputs

- Per-data-element thermal noise scale s estimated from Equation 14 with the Eq. 4 covariance
- Maximum power-of-two rounding granularity g satisfying sigma_r^2 < f*s^2 (Equations 15-16)
- Rounded array written to rounded_data.npy, with rounding errors verifiable as unbiased

## How to apply

Load the CHIME Pathfinder visibility dataset (shape (64, 136, 1024), each element a struct of two 32-bit signed little-endian integers, time fastest-varying, 68 MiB) via h5py/HDF5. Estimate the thermal noise s on a data-element-by-data-element basis using Equation 14 together with the noise covariance matrix defined by Equation 4 (or Equations 5-7); this estimate is calibration-independent because it derives from the radiometer equation rather than a calibration solution. Choose an acceptable fractional increase in noise variance f (recommended 10^-5 > f > 10^-2; the paper uses f = 10^-3) and derive the maximum rounding granularity from the constraint sigma_r^2 < f*s^2 of Equations 15-16, selecting the largest power of two not exceeding that limit. The rationale: rounding noise per element is bounded by the granularity, so tying granularity to the per-element thermal-noise scale keeps the induced variance negligible while minimizing entropy ahead of Bitshuffle.

## Related tools

- **h5py/HDF5** (Read the CHIME Pathfinder visibility dataset whose per-element noise scale is estimated before rounding)
- **Bitshuffle** (Reference implementation in the kiyo-masui/bitshuffle repository against which the chosen granularity and rounded values are compared) — https://github.com/kiyo-masui/bitshuffle

## Examples

```
g = 2**int(np.floor(np.log2(np.sqrt(12*f*s)))); rounded_data = (np.rint(data.astype(np.float64)/g)*g).astype(data.dtype); np.save('rounded_data.npy', rounded_data)
```

## Evaluation signals

- Rounding errors computed by direct subtraction of the original dataset from the precision-reduced dataset have a probability density symmetric about zero (unbiased, ties rounded to even)
- No absolute rounding error exceeds half the maximum granularity sqrt(12 f s^2)
- The chosen granularity g is the largest power of two not exceeding the limit implied by sigma_r^2 < f*s^2 in Equations 15-16
- With f = 10^-3 the increase in per-element noise variance stays within 10^-3 of the radiometer-equation estimate s^2
- Granularity and rounded values match the authors' released example implementation in kiyo-masui/bitshuffle

## Limitations

- The noise estimate is only valid for integrated, post-correlation data where the generalized radiometer equation applies; it requires the calibration-independent covariance form of Equations 4-7
- Bitshuffle relies on neighbouring elements being highly correlated, so the downstream benefit of noise-scaled rounding degrades for uncorrelated data
- Rounding is irreversible ('nearly lossless'); once data are rounded to granularity g, sub-g information is permanently discarded

## Evidence
