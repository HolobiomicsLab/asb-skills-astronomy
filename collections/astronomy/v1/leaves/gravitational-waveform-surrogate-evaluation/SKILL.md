---
name: gravitational-waveform-surrogate-evaluation
description: Use when waveform generation cost dominates an analysis (e.g., parameter
  estimation or parameter-space exploration) and fast, accurate evaluation of effective-one-body
  or NR-trained surrogate models is needed.
when_to_use_negative:
- When a new fiducial waveform family must be constructed from scratch — the surrogate
  building phase (greedy reduced basis, empirical interpolation, parameter fits) must
  be performed offline with other codes; GWSurrogate only provides fast evaluation
  of pre-built surrogates
- When binary parameters fall outside a surrogate's training domain (e.g., q>4 for
  NRSur7dq4, q>8 for NRHybSur3dq8, q≤2 for the paper's EOB test case) — extrapolation
  accuracy is not guaranteed
- When the required output is the EOB evolution dynamics itself rather than the waveform
  modes; surrogates reproduce the waveform, not the differential-equation solution
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_3399
tools:
- name: gwsurrogate
  role: 'Public interface to gravitational-wave surrogate models: catalog.pull downloads
    surrogate data, LoadSurrogate loads fitted coefficients and basis, and model call
    evaluates waveform modes'
  repo: https://github.com/sxs-collaboration/gwsurrogate
- name: gwtools
  role: Waveform utilities for computing pointwise amplitude and phase differences
    against the fiducial family during validation
  repo: https://bitbucket.org/chadgalley/gwtools/
- name: NumPy
  role: Random mass-ratio sampling from [1,2] and array handling; required C-API headers
    at build time for gwsurrogate's C-extension modules
- name: LIGO Analysis Library (LAL)
  role: EOB-LAL code generates fiducial EOB waveforms as the timing baseline for speedup
    measurements
- name: pytest
  role: Runs the package's regression tests to cross-check stored waveform outputs
  repo: https://github.com/sxs-collaboration/gwsurrogate
- name: PyCBC
  role: Alternative route to evaluate any gwsurrogate model through PyCBC's waveform
    API
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_003
  source_papers:
  - doi: 10.1103/physrevx.4.031006
    title: Fast Prediction and Evaluation of Gravitational Waveforms Using Surrogate
      Models
  source_content_hash: sha256:7ef52caa01cbe27953ef56921ea4e850e357e84167ce24729707c60c5ef38ee8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sxs-collaboration/gwsurrogate
  tools:
  - GWSurrogate
  - LIGO Analysis Library (LAL)
  - NumPy
  - PyCBC
  - Python
  - gwsurrogate
  - gwtools
  - pytest
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# gravitational-waveform-surrogate-evaluation

## Summary

Evaluates gravitational-wave surrogate models — reduced-basis + empirical-interpolation + parameter-fit surrogates — with GWSurrogate to produce waveform modes orders of magnitude faster than solving EOB differential equations, while validating surrogate error and timing against fiducial codes.

## When to use

Use when waveform generation cost dominates an analysis (e.g., parameter estimation or parameter-space exploration) and fast, accurate evaluation of effective-one-body or NR-trained surrogate models is needed.

## When NOT to use

- When a new fiducial waveform family must be constructed from scratch — the surrogate building phase (greedy reduced basis, empirical interpolation, parameter fits) must be performed offline with other codes; GWSurrogate only provides fast evaluation of pre-built surrogates
- When binary parameters fall outside a surrogate's training domain (e.g., q>4 for NRSur7dq4, q>8 for NRHybSur3dq8, q≤2 for the paper's EOB test case) — extrapolation accuracy is not guaranteed
- When the required output is the EOB evolution dynamics itself rather than the waveform modes; surrogates reproduce the waveform, not the differential-equation solution

## Inputs

- Binary parameters: mass ratio q (mA/mB >= 1), dimensionless spin vectors chiA and chiB
- Timestep dt in units of total mass M and initial frequency f_low
- Number of time samples L (e.g., L=16,384)
- Random validation parameter draws (e.g., 1,000 q values uniform in [1,2], spins zero for the nonspinning fiducial family)
- A loaded surrogate model (fitted coefficients + empirical interpolant basis, e.g., the EOB test-case surrogate with m=10 greedy elements)

## Outputs

- Time array (units of total mass M) and waveform spherical-harmonic modes from sur(q, chiA, chiB, dt, f_low)
- Per-parameter surrogate reconstruction errors and pointwise amplitude/phase differences vs. the fiducial family
- Max surrogate error, max amplitude/phase error, and q of largest error (e.g., written to surrogate_validation_results.csv)
- Per-sampling-rate wall-clock timings and surrogate-to-EOB-LAL speedup factors with linear-scaling check (e.g., timing_comparison.csv)

## How to apply

Pull the surrogate data once with gwsurrogate.catalog.pull('MODEL') and load it with gwsurrogate.LoadSurrogate, which brings the fitted coefficients and empirical-interpolant basis into memory. Evaluate a generic parameter choice via the model call sur(q, chiA, chiB, dt, f_low) — e.g., q the mass ratio, dimensionless spins chiA/chiB, dt the timestep in units of total mass M, f_low=0 to return the full surrogate — obtaining the time array and waveform modes; L=16,384 time samples was used for validation in q∈[1,2]. The online prediction cost is O(mL + m*c_fit) with m greedy basis elements typically m << L, so evaluation cost is linear in the number of time samples L. Validate by checking the surrogate reconstruction error against the fiducial family (uniformly below 10^-7 over 1,000 random mass ratios in the paper's EOB test case, largest error at q=1.068), pointwise amplitude/phase errors with gwtools utilities (~1e-3 or below), and timing against EOB-LAL generation (≈2,300x speedup at a 2,048 Hz sampling rate). Cross-check stored outputs against the package's pytest regression tests.

## Related tools

- **gwsurrogate** (Public interface to gravitational-wave surrogate models: catalog.pull downloads surrogate data, LoadSurrogate loads fitted coefficients and basis, and model call evaluates waveform modes) — https://github.com/sxs-collaboration/gwsurrogate
- **gwtools** (Waveform utilities for computing pointwise amplitude and phase differences against the fiducial family during validation) — https://bitbucket.org/chadgalley/gwtools/
- **NumPy** (Random mass-ratio sampling from [1,2] and array handling; required C-API headers at build time for gwsurrogate's C-extension modules)
- **LIGO Analysis Library (LAL)** (EOB-LAL code generates fiducial EOB waveforms as the timing baseline for speedup measurements)
- **pytest** (Runs the package's regression tests to cross-check stored waveform outputs) — https://github.com/sxs-collaboration/gwsurrogate
- **PyCBC** (Alternative route to evaluate any gwsurrogate model through PyCBC's waveform API)

## Examples

```
import gwsurrogate; gwsurrogate.catalog.pull('NRSur7dq4'); sur = gwsurrogate.LoadSurrogate('NRSur7dq4'); t, h, dyn = sur(4, [-0.2,0.4,0.1], [-0.5,0.2,-0.4], dt=0.1, f_low=0)
```

## Evaluation signals

- Surrogate error must fall below 1e-7 over the 1,000 randomly sampled mass ratios in [1,2]
- Largest surrogate model error must occur at q = 1.068 for the EOB test case
- Pointwise amplitude/phase errors must be at or below ~1e-3
- Measured speedup at a 2,048 Hz sampling rate must be approximately 2,300x, with overall speedups between 2 and nearly 4 orders of magnitude
- Surrogate online timing must be linear in the number of samples L, consistent with the O(mL + m*c_fit) cost prediction, and stored outputs must match the package's pytest regression tests

## Limitations

- Surrogate building (greedy basis, EIM nodes, fits) is a separate offline phase using other codes; GWSurrogate provides evaluation only
- Accuracy claims apply within the trained parameter domain — e.g., the EOB test case covers nonspinning q∈[1,2] with durations as long as 10^5M; models like NRSur7dq4 cover q≤4 and NRHybSur3dq8 q≤8
- The paper's surrogate validation used the nonspinning fiducial family (spins held at zero); precessing evaluation requires models trained for it (e.g., NRSur7dq4, SEOBNRv4PHMSur)
- Building gwsurrogate from source requires NumPy C-API headers; NumPy 1.x vs 2.x compatibility may require pinning (e.g., conda create -n myenv python=3.11 "numpy<2.0")

## Evidence

- [readme] gwsurrogate usage: "gwsurrogate.catalog.pull('NRSur7dq4')       # This can take a few minutes"
- [readme] model loading: "sur = gwsurrogate.LoadSurrogate('NRSur7dq4')"
- [readme] surrogate call signature: "t, h, dyn = sur(q, chiA, chiB, dt=dt, f_low=f_low, precessing_opts=precessing_opts)"
- [abstract] speedup claim: "We find that these surrogates are more than 3 orders of magnitude faster to evaluate as compared to the cost of generating effective-one-body waveforms in standard ways."
- [abstract] online cost model: [evidence span withheld — anchor status: not_found]
- [abstract] parameter-estimation motivation: "As waveform generation is one of the dominant costs in parameter estimation algorithms and parameter space exploration, surrogate models offer a new and practical way to dramatically accelerate such"
