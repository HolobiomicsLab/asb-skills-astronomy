---
name: reference-implementation-consistency-check
description: Use when you have independently derived a rounding granularity g and
  precision-reduced radio data from Equations 14-16 and must confirm your values match
  the authors' kiyo-masui/bitshuffle example before trusting downstream compression
  results.
when_to_use_negative:
- Data are not radiometer-equation-limited (e.g. post-calibration or systematics-dominated
  data), since the noise scale s from Equation 14 no longer bounds rounding noise
  meaningfully.
- Data are uncorrelated between adjacent elements, where Bitshuffle-style reference
  tooling gains nothing and the consistency check target codepath does not apply.
- A quick lossy-compression comparison is needed instead of exact consistency with
  the paper's nearly-lossless rounding step.
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: Bitshuffle
  role: Provides the authors' example implementation of the precision-reduction and
    HDF5 filter (number 32008) against which derived granularity and rounded values
    are compared
  repo: https://github.com/kiyo-masui/bitshuffle
- name: HDF5
  role: Data format used to load the CHIME Pathfinder visibility dataset and integrate
    the reference filter via h5py
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2015.07.002
    title: A compression scheme for radio data in high performance computing
  source_content_hash: sha256:26c880ad19afaeec0febe0196034125de4459f3e1273462a4fdd4e2eea80ac5f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/kiyo-masui/bitshuffle
  tools:
  - Bitshuffle
  - CHIME Pathfinder
  - HDF5
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# Reference implementation consistency check

## Summary

Validates a self-implemented controlled precision-reduction (radiometer-relative rounding) procedure by comparing the chosen power-of-two granularity and the rounded integer values against the authors' released example code in the kiyo-masui/bitshuffle repository.

## When to use

Use when you have independently derived a rounding granularity g and precision-reduced radio data from Equations 14-16 and must confirm your values match the authors' kiyo-masui/bitshuffle example before trusting downstream compression results.

## When NOT to use

- Data are not radiometer-equation-limited (e.g. post-calibration or systematics-dominated data), since the noise scale s from Equation 14 no longer bounds rounding noise meaningfully.
- Data are uncorrelated between adjacent elements, where Bitshuffle-style reference tooling gains nothing and the consistency check target codepath does not apply.
- A quick lossy-compression comparison is needed instead of exact consistency with the paper's nearly-lossless rounding step.

## Inputs

- CHIME Pathfinder correlator visibility dataset (shape (64, 136, 1024), each element a struct of two 32-bit signed little-endian integers, time fastest-varying, 68 MiB total, HDF5/h5py)
- Per-element thermal noise estimates s from Equation 14 with Equation 4 (or 5-7) covariance
- Chosen fractional noise-variance increase f (e.g. 10^-3)
- Self-derived power-of-two rounding granularity g and rounded array
- Authors' example precision-reduction code from the kiyo-masui/bitshuffle repository

## Outputs

- Verified rounding granularity g (largest power of two with sigma_r^2 < f*s^2)
- rounded_data.npy containing the precision-reduced array matching the reference implementation
- Rounding-error array (original minus reduced) with empirical distribution statistics

## How to apply

First reproduce the precision-reduction pipeline: load the CHIME Pathfinder visibility dataset (64, 136, 1024) via h5py/HDF5, estimate thermal noise s per data element using Equation 14 with the calibration-independent noise covariance from Equation 4 (or 5-7), fix the fractional noise-variance increase at f = 10^-3 (recommended range 10^-5 > f > 10^-2), and select the largest power-of-two granularity satisfying sigma_r^2 < f*s^2 (Equations 15-16). Round every element to the nearest multiple of g using ties-to-even (e.g. numpy.rint on values scaled by 1/g), subtracting the original array to obtain rounding errors for verification. Then diff the chosen g and the rounded array element-by-element against the output of the authors' example implementation in kiyo-masui/bitshuffle: granularities must agree exactly, and rounded values should be identical because ties-to-even makes the rounding deterministic and unbiased. Any mismatch in granularity signals a mis-derived f, a mis-estimated s (often from using a calibration-dependent covariance), or a different tie-breaking convention.

## Related tools

- **Bitshuffle** (Provides the authors' example implementation of the precision-reduction and HDF5 filter (number 32008) against which derived granularity and rounded values are compared) — https://github.com/kiyo-masui/bitshuffle
- **HDF5** (Data format used to load the CHIME Pathfinder visibility dataset and integrate the reference filter via h5py)

## Examples

```
import numpy as np; err = data - (g*np.rint(data/1.0/g)).astype(data.dtype); np.save('rounded_data.npy', g*np.rint(data/1.0/g))
```

## Evaluation signals

- Chosen granularity g exactly equals the granularity used by the kiyo-masui/bitshuffle example for the same f and dataset
- rounded_data.npy matches the reference implementation's output bit-for-bit (ties-to-even is deterministic)
- Rounding errors are unbiased: probability density symmetric about zero
- No absolute rounding error exceeds half the maximum granularity bound sqrt(12 f s^2)
- Fractional increase in noise variance stays below f = 10^-3 (and within the recommended 10^-5 > f > 10^-2 band)

## Limitations

- The consistency check is only valid for the specific CHIME Pathfinder dataset (Jan 25 2015, 68 MiB) and f used in the paper; other datasets require re-estimating s per element.
- Ties-to-even rounding must match the reference exactly; a different tie-breaking (e.g. round-half-up) will produce small but systematic mismatches.
- Bitshuffle requires adjacent elements along the fastest-varying index to be highly correlated; correlations spanning fewer than 24 elements cannot be exploited by the reference filter.
- Bitshuffle binary wheels are built with AVX2 support and will crash with SIGILL on unsupported x86_64 processors, requiring a source build for comparison runs.

## Evidence

- [methods] round to largest power of two: [evidence span withheld — anchor status: not_found]
- [abstract] nearly lossless rounding: [evidence span withheld — anchor status: not_found]
