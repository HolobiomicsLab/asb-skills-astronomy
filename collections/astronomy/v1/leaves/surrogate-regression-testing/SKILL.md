---
name: surrogate-regression-testing
description: 'Use when a built or updated GW surrogate model must be validated: sample
  parameters randomly (e.g., 1,000 mass ratios in [1,2]), evaluate the surrogate,
  and check error magnitudes and their parameter dependence against published thresholds.'
when_to_use_negative:
- Evaluating surrogates for precessing or high-spin systems when the validated family
  is the nonspinning fiducial one (spins are held at zero here)
- Mass ratios outside the surrogate's training domain (this check covers [1,2]; other
  models cover e.g. q≤4, q≤8, or q≤10000 but with different accuracy guarantees)
- Timing/benchmark studies of surrogate speed (3+ orders of magnitude faster than
  EOB generation) rather than accuracy validation
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: gwsurrogate
  role: Catalog pull, model loading (LoadSurrogate), and surrogate evaluation sur(q,
    chiA, chiB, dt, f_low); pytest suite provides regression tests for stored waveform
    outputs
  repo: https://github.com/sxs-collaboration/gwsurrogate
- name: gwtools
  role: Waveform utilities for computing pointwise amplitude and phase differences
    against the reference family
  repo: https://bitbucket.org/chadgalley/gwtools/
- name: NumPy
  role: Random sampling of 1,000 mass ratios from [1,2] and array-level error aggregation
- name: PyCBC
  role: Alternative API to evaluate gwsurrogate models when cross-checking outputs
- name: pytest
  role: Run the package's regression tests to cross-check stored waveform outputs
  repo: https://github.com/sxs-collaboration/gwsurrogate
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1103/physrevx.4.031006
    title: Fast Prediction and Evaluation of Gravitational Waveforms Using Surrogate
      Models
  source_content_hash: sha256:f00366d3a9d7548a9518e85a9a5e95d83a7dc810a6935efc09679b3b45df70b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sxs-collaboration/gwsurrogate
  tools:
  - NumPy
  - PyCBC
  - Python
  - gwsurrogate
  - gwtools
  - pytest
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# surrogate-regression-testing

## Summary

Regression-test a gravitational-wave surrogate model by evaluating it over randomly sampled mass ratios, computing surrogate reconstruction and pointwise amplitude/phase errors against the fiducial family, and validating against known error thresholds and the parameter location of the largest error.

## When to use

Use when a built or updated GW surrogate model must be validated: sample parameters randomly (e.g., 1,000 mass ratios in [1,2]), evaluate the surrogate, and check error magnitudes and their parameter dependence against published thresholds.

## When NOT to use

- Evaluating surrogates for precessing or high-spin systems when the validated family is the nonspinning fiducial one (spins are held at zero here)
- Mass ratios outside the surrogate's training domain (this check covers [1,2]; other models cover e.g. q≤4, q≤8, or q≤10000 but with different accuracy guarantees)
- Timing/benchmark studies of surrogate speed (3+ orders of magnitude faster than EOB generation) rather than accuracy validation

## Inputs

- Loaded gwsurrogate model (fitted coefficients + empirical interpolant basis) for the EOB nonspinning fiducial family
- 1,000 mass-ratio values drawn uniformly at random from [1,2] (spins chiA=chiB=0)
- Timestep dt sized so the full waveform is covered with L=16,384 time samples, f_low=0

## Outputs

- Per-sample surrogate reconstruction error (Eq. 31) magnitudes
- Max pointwise amplitude and phase errors per parameter point
- Mass ratio q where the largest surrogate error occurs (expected q=1.068)
- surrogate_validation_results.csv summarizing errors and worst-case parameter

## How to apply

Pull the surrogate data with gwsurrogate.catalog.pull and load it via gwsurrogate.LoadSurrogate so fitted coefficients and the empirical interpolant basis are in memory. Draw 1,000 mass-ratio values uniformly at random from [1,2] with numpy.random, keeping spins at zero for the nonspinning fiducial family. Evaluate each sample via sur(q, chiA, chiB, dt, f_low) with L=16,384 time samples and f_low=0 to return the full surrogate, obtaining the time array in units of total mass M and the waveform modes. Compute the surrogate reconstruction error (Eq. 31) against the reference family and pointwise amplitude/phase differences using gwtools waveform utilities, aggregate max errors plus the q where the largest error occurs, and write the table to surrogate_validation_results.csv. Validate: surrogate error below 1e-7, pointwise amplitude/phase errors at or below ~1e-3, largest error at q=1.068; cross-check stored waveform outputs against the package's pytest regression tests.

## Related tools

- **gwsurrogate** (Catalog pull, model loading (LoadSurrogate), and surrogate evaluation sur(q, chiA, chiB, dt, f_low); pytest suite provides regression tests for stored waveform outputs) — https://github.com/sxs-collaboration/gwsurrogate
- **gwtools** (Waveform utilities for computing pointwise amplitude and phase differences against the reference family) — https://bitbucket.org/chadgalley/gwtools/
- **NumPy** (Random sampling of 1,000 mass ratios from [1,2] and array-level error aggregation)
- **PyCBC** (Alternative API to evaluate gwsurrogate models when cross-checking outputs)
- **pytest** (Run the package's regression tests to cross-check stored waveform outputs) — https://github.com/sxs-collaboration/gwsurrogate

## Examples

```
import gwsurrogate; gwsurrogate.catalog.pull('NRSur7dq4'); sur = gwsurrogate.LoadSurrogate('NRSur7dq4'); t, h, dyn = sur(q, chiA, chiB, dt=0.1, f_low=0)
```

## Evaluation signals

- Surrogate error (Eq. 31) is uniformly below 1e-7 over all 1,000 sampled mass ratios
- Pointwise amplitude/phase errors are at or below ~1e-3
- Largest surrogate error occurs at q=1.068
- surrogate_validation_results.csv contains max surrogate error, max pointwise amplitude/phase error, and the worst-case q for all 1,000 points
- Stored waveform outputs match gwsurrogate's pytest regression tests

## Limitations

- Accuracy thresholds (error < 1e-7, amplitude/phase ~1e-3) are specific to this EOB nonspinning surrogate with L=16,384 samples and q in [1,2]; other models (e.g., NRSur7dq4 trained on q≤4, NRHybSur3dq8 on q≤8) have different domains and accuracies
- Some gwsurrogate modules are C-extensions requiring NumPy C-API headers at build time; NumPy 1.x vs 2.x compatibility must be handled (e.g., conda create -n myenv python=3.11 'numpy<2.0')
- Random sampling means the exact worst-case q depends on the sample; the published q=1.068 is for the paper's specific draw of 1,000 ratios

## Evidence

- [methods] Pull and load workflow: "gwsurrogate.catalog.pull('NRSur7dq4')       # This can take a few minutes"
- [readme] Load surrogate: "sur = gwsurrogate.LoadSurrogate('NRSur7dq4')"
- [readme] Evaluate surrogate call signature: "t, h, dyn = sur(q, chiA, chiB, dt=dt, f_low=f_low, precessing_opts=precessing_opts)"
- [methods] Regression testing via pytest: ">>> pytest                               # run all tests"
- [abstract] Fiducial waveform family definition: "generating a fiducial waveform using the EOB solver as implemented in the LIGO Analysis Library (LAL) software [54], which we refer to as the EOB-LAL code"
- [readme] Time samples in units of total mass: "dt = 0.1                        # timestep size, Units of total mass M"
