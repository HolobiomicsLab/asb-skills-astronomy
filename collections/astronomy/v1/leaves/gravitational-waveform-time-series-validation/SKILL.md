---
name: gravitational-waveform-time-series-validation
description: Use when a surrogate or reduced-basis model predicts gravitational waveform
  time series at arbitrary parameter values and you must verify accuracy against the
  fiducial solver and quantify online evaluation cost/speedup before trusting predictions.
when_to_use_negative:
- When the waveform family changes (different solver, parameter range, or spherical-harmonic
  mode content) — the surrogate must be rebuilt; validation against the old fiducial
  set is meaningless
- When you need a waveform outside the trained parameter range (e.g. mass ratios beyond
  1–10 for the nonspinning EOB model) — the surrogate and its validation do not extrapolate
- When the goal is to validate raw numerical-relativity simulation data rather than
  a surrogate reconstruction of a fiducial solver family
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3322
- http://edamontology.org/topic_3458
tools:
- name: gwsurrogate
  role: Interface to gravitational wave surrogate models; provides catalog, loading,
    and evaluation used to generate and validate surrogate waveform time series
  repo: https://github.com/sxs-collaboration/gwsurrogate
- name: gwtools
  role: Dependency of gwsurrogate used in surrogate construction and waveform handling
  repo: https://bitbucket.org/chadgalley/gwtools/
- name: NumPy
  role: Array storage for fiducial waveform time series and arithmetic behind the
    O(mL + m*c_fit) online evaluation
- name: LIGO Analysis Library (LAL)
  role: EOB-LAL fiducial solver used as the ground-truth generator against which the
    surrogate is benchmarked
- name: pytest
  role: Test runner exercising the surrogate module end-to-end and enforcing accuracy/validation
    checks
- name: Python
  role: Implementation language for the surrogate construction and validation pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1103/physrevx.4.031006
    title: Fast Prediction and Evaluation of Gravitational Waveforms Using Surrogate
      Models
  source_content_hash: sha256:ffe8bed149254021527f870ab6232961bd5845a4fa10c9f26b916429febabda9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sxs-collaboration/gwsurrogate
  tools:
  - GWSurrogate
  - LIGO Analysis Library (LAL)
  - NumPy
  - Python
  - gwsurrogate
  - gwtools
  - pytest
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3322
  - http://edamontology.org/topic_3458
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# gravitational-waveform-time-series-validation

## Summary

Validates that a fast surrogate (or reconstructed) gravitational waveform time series faithfully reproduces the fiducial waveform family — e.g. EOB-LAL waveforms — by benchmarking online surrogate evaluation against direct fiducial generation, checking accuracy while confirming >3 orders of magnitude speedup.

## When to use

Use when a surrogate or reduced-basis model predicts gravitational waveform time series at arbitrary parameter values and you must verify accuracy against the fiducial solver and quantify online evaluation cost/speedup before trusting predictions.

## When NOT to use

- When the waveform family changes (different solver, parameter range, or spherical-harmonic mode content) — the surrogate must be rebuilt; validation against the old fiducial set is meaningless
- When you need a waveform outside the trained parameter range (e.g. mass ratios beyond 1–10 for the nonspinning EOB model) — the surrogate and its validation do not extrapolate
- When the goal is to validate raw numerical-relativity simulation data rather than a surrogate reconstruction of a fiducial solver family

## Inputs

- Fiducial waveform family sampled on a parameter grid (EOB-LAL time series loaded into NumPy arrays)
- Reduced basis of m greedy-selected parameter values and their m empirical interpolant time nodes
- Fitted c_fit coefficient sets at each of the m time nodes

## Outputs

- Reconstructed waveform time series at a generic parameter choice (L time samples)
- Benchmark comparison of surrogate evaluation vs direct fiducial EOB-LAL generation (accuracy and runtime speedup, expected >1000x)
- pytest test suite results confirming end-to-end runs (script_runs)

## How to apply

Generate the fiducial waveform family on a parameter grid with the EOB solver (EOB-LAL code) loaded into NumPy arrays, build the surrogate via the fixed 4-step loop (greedy reduced basis over m parameter values -> empirical interpolant inducing m time nodes -> spline fits of the c_fit coefficients at each node -> online evaluation per Eq. 30). Then benchmark the online evaluation — predicting L time samples in O(mL + m*c_fit) operations, with m << L typically — against direct fiducial waveform generation using the EOB-LAL solver. Correct application means the surrogate reproduces fiducial waveforms accurately at generic parameter choices within the trained range (e.g. mass ratios 1–10, durations up to 10^5M) while evaluating more than 3 orders of magnitude faster than standard EOB generation. Exercise the module through pytest so the end-to-end run and accuracy checks are reproducible regression signals.

## Related tools

- **gwsurrogate** (Interface to gravitational wave surrogate models; provides catalog, loading, and evaluation used to generate and validate surrogate waveform time series) — https://github.com/sxs-collaboration/gwsurrogate
- **gwtools** (Dependency of gwsurrogate used in surrogate construction and waveform handling) — https://bitbucket.org/chadgalley/gwtools/
- **NumPy** (Array storage for fiducial waveform time series and arithmetic behind the O(mL + m*c_fit) online evaluation)
- **LIGO Analysis Library (LAL)** (EOB-LAL fiducial solver used as the ground-truth generator against which the surrogate is benchmarked)
- **pytest** (Test runner exercising the surrogate module end-to-end and enforcing accuracy/validation checks)
- **Python** (Implementation language for the surrogate construction and validation pipeline)

## Examples

```
import gwsurrogate; gwsurrogate.catalog.pull('NRSur7dq4'); sur = gwsurrogate.LoadSurrogate('NRSur7dq4'); t, h, dyn = sur(4, [-0.2,0.4,0.1], [-0.5,0.2,-0.4], dt=0.1, f_low=0)
```

## Evaluation signals

- Surrogate-reconstructed waveform at held-out parameter values matches the direct EOB-LAL fiducial waveform time series to within the accuracy tolerance used for the model
- Online evaluation is more than 3 orders of magnitude (1000x) faster than generating the underlying EOB waveform via the standard EOB-LAL route
- Prediction cost for L time samples is consistent with the O(mL + m*c_fit) scaling with m << L
- pytest suite passes, confirming the module runs end-to-end (script_runs)
- Surrogate reproduces waveforms across the full trained parameter range (mass ratios 1–10, durations up to 10^5M, multiple spherical harmonic modes)

## Limitations

- Accuracy and speedup claims hold only within the fiducial parameter range used in the building phase; no guarantee outside it
- Validation requires the building phase to have been performed with other codes (differential-equation solvers), so surrogate accuracy is bounded by the fidelity and sampling of the fiducial waveform family
- Some gwsurrogate modules are C-extensions requiring NumPy's C-API headers at build time; NumPy 1.x vs 2.x version mismatches can affect installation and reproducibility of tests

## Evidence

- [abstract] Speedup benchmark: "We find that these surrogates are more than 3 orders of magnitude faster to evaluate as compared to the cost of generating effective-one-body waveforms in standard ways."
- [abstract] Online cost scaling: [evidence span withheld — anchor status: not_found]
- [abstract] Greedy/empirical-interpolant loop: "First, a set of m parameter values is determined using a greedy algorithm from which a reduced basis representation is constructed."
- [methods] Validation workflow from source card: [evidence span withheld — anchor status: not_found]
- [abstract] Model parameter range: [evidence span withheld — anchor status: not_found]
- [readme] Surrogate purpose: "Surrogates provide a fast and accurate evaluation mechanism for gravitational waveforms, which would otherwise be found through solving differential equations."
