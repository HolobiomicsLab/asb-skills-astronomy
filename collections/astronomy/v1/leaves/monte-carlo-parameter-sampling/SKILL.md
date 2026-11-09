---
name: monte-carlo-parameter-sampling
description: Use when a surrogate or empirical interpolant has been built (e.g., 19-element
  reduced basis for q in [1,2] EOB waveforms) and you need a randomized, distribution-aware
  validation of interpolation error against a theoretical bound such as 1e-8 from
  Eq. (20).
when_to_use_negative:
- You need training-set construction rather than validation — greedy parameter selection
  and EIM node selection are separate upstream steps, not Monte Carlo sampling
- The parameter domain differs from the surrogate's training range (e.g., precessing/spinning
  surrogates like NRSur7dq4 trained on q≤4, NRHybSur3dq8 on q≤8) — extrapolation outside
  the training range invalidates the bound
- You only need a single fast online surrogate evaluation rather than a distribution-level
  error study
edam_operation: http://edamontology.org/operation_0244
tools:
- name: NumPy
  role: Generate the 1,000 uniformly random q samples and compute pointwise errors
    and bounds vectorized over waveforms
  repo: https://github.com/sxs-collaboration/gwsurrogate
- name: Python
  role: Implementation language for the sampling, interpolation evaluation, and error
    diagnostics loop
  repo: https://github.com/sxs-collaboration/gwsurrogate
- name: GWSurrogate
  role: Provides access to the surrogate models and evaluation interface used alongside
    the reduced-basis/EIM machinery
  repo: https://github.com/sxs-collaboration/gwsurrogate
- name: gwtools
  role: Supporting waveform/reduced-basis utilities required by the gwsurrogate environment
  repo: https://bitbucket.org/chadgalley/gwtools/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1103/physrevx.4.031006
    title: Fast Prediction and Evaluation of Gravitational Waveforms Using Surrogate
      Models
  source_content_hash: sha256:fd79af54db8fe8c962f6ad056d5ec8b2cb31e05fb821cc089ac4c3efee4d78b7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sxs-collaboration/gwsurrogate
  tools:
  - GWSurrogate
  - NumPy
  - Python
  - gwtools
  edam_operation: http://edamontology.org/operation_0244
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# monte-carlo-parameter-sampling

## Summary

Randomized Monte Carlo validation of a reduced-basis/EIM surrogate for nonspinning EOB binary-black-hole waveforms: draw mass ratios q uniformly at random in [1,2] and measure how well the empirical interpolant and its a posteriori bound reproduce the fiducial waveform family over the parameter space.

## When to use

Use when a surrogate or empirical interpolant has been built (e.g., 19-element reduced basis for q in [1,2] EOB waveforms) and you need a randomized, distribution-aware validation of interpolation error against a theoretical bound such as 1e-8 from Eq. (20).

## When NOT to use

- You need training-set construction rather than validation — greedy parameter selection and EIM node selection are separate upstream steps, not Monte Carlo sampling
- The parameter domain differs from the surrogate's training range (e.g., precessing/spinning surrogates like NRSur7dq4 trained on q≤4, NRHybSur3dq8 on q≤8) — extrapolation outside the training range invalidates the bound
- You only need a single fast online surrogate evaluation rather than a distribution-level error study

## Inputs

- 19-element reduced basis for the fiducial nonspinning EOB waveform family (q in [1,2])
- 19 empirical time nodes and B_i(t) interpolation coefficient functions from EIM
- Parameter-space fits of the waveform value at each of the m=19 time nodes
- Access to exact reduced-basis (greedy) waveforms for arbitrary q in [1,2]

## Outputs

- 1,000 randomly sampled q values in [1,2]
- Maximum pointwise empirical interpolation error over the 1,000 validation waveforms
- Comparison of empirical interpolation error vs. greedy (reduced-basis) error
- Lebesgue-constant-based a posteriori error bound (expected < 1e-8)
- Diagnostics CSV (eim_validation.csv) containing sampled q, errors, and bound

## How to apply

Draw validation parameters by Monte Carlo: sample 1,000 mass ratios q uniformly at random from the training interval [1,2] using numpy.random.uniform. For each sampled q, evaluate the empirical interpolant built from the 19 empirical time nodes, the precomputed B_i(t) coefficient functions, and the parameter-space fits at each node, then compare to the exact reduced-basis waveform for the same q. Compute the maximum pointwise error |empirical interpolant - exact reduced-basis waveform| across the 1,000 waveforms and compare it against the greedy (reduced-basis) error, which the paper reports is remarkably close to the empirical interpolation error. Finally compute the Lebesgue-constant-based a posteriori error bound and confirm it guarantees error below 1e-8 as claimed for Eq. (20). Persist the sampled q values and error/bound diagnostics (e.g., to eim_validation.csv) so the validation is reproducible.

## Related tools

- **NumPy** (Generate the 1,000 uniformly random q samples and compute pointwise errors and bounds vectorized over waveforms) — https://github.com/sxs-collaboration/gwsurrogate
- **Python** (Implementation language for the sampling, interpolation evaluation, and error diagnostics loop) — https://github.com/sxs-collaboration/gwsurrogate
- **GWSurrogate** (Provides access to the surrogate models and evaluation interface used alongside the reduced-basis/EIM machinery) — https://github.com/sxs-collaboration/gwsurrogate
- **gwtools** (Supporting waveform/reduced-basis utilities required by the gwsurrogate environment) — https://bitbucket.org/chadgalley/gwtools/

## Examples

```
import numpy as np; qs = np.random.uniform(1.0, 2.0, 1000)
```

## Evaluation signals

- The maximum empirical interpolation error over the 1,000 randomly sampled q in [1,2] sits at the greedy (reduced-basis) error level
- The Lebesgue-constant-based a posteriori bound guarantees error better than 1e-8 (Eq. (20) guarantee)
- The sampled q values are reproducibly uniform on [1,2] and the full set of 1,000 draws is recorded in the output CSV
- Error diagnostics (max pointwise error per waveform) are present for every sampled q with no missing entries

## Limitations

- Validation is limited to the fiducial nonspinning EOB family with q in [1,2]; the bound does not transfer to other parameter ranges or spinning/precessing surrogate families
- The a posteriori bound involves the Lebesgue constant of the interpolation nodes; ill-conditioning of the EIM linear system at the nodes can degrade the guarantee
- 1,000 random draws give a statistical picture, not a worst-case certificate over the continuum of q

## Evidence

- [other] Monte Carlo draw of validation parameters: [evidence span withheld — anchor status: not_found]
- [other] Error bound validation target: [evidence span withheld — anchor status: not_found]
- [abstract] Surrogate validation against fiducial family: "Second, these m parameters induce the selection of m time values for interpolating a waveform time series using an empirical interpolant that is built for the fiducial waveform family."
- [readme] Environment requirement for running the sampling/evaluation code: "Certain gwsurrogate modules are implemented as C-extensions and require NumPy’s C-API headers at build time."
