---
name: waveform-parameter-space-fitting
description: Use when a reduced-basis/empirical-interpolant surrogate construction
  needs the waveform's value at each of the m empirical time nodes fitted across the
  parameter dimension, and fit residuals must be validated against fiducial EOB waveforms.
when_to_use_negative:
- The goal is building the reduced basis or selecting empirical time nodes via the
  greedy algorithm — this skill covers only the third (fitting) stage of the surrogate
  pipeline
- The waveforms are numerical-relativity or precessing-spin surrogates (e.g. NRSur7dq4,
  NRHybSur3dq8) rather than the nonspinning EOB fiducial family; the PN-inspired ansatz
  and error thresholds were derived for nonspinning EOB-LAL waveforms
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: NumPy
  role: least-squares regression for the polynomial and PN-inspired fits of amplitude
    and phase versus mass ratio at each empirical node
  repo: https://pypi.org/project/gwsurrogate
- name: gwtools
  role: writing fit coefficients and per-node error diagnostics to CSV; dependency
    installed automatically with gwsurrogate
  repo: https://bitbucket.org/chadgalley/gwtools/
- name: GWSurrogate
  role: public package providing surrogate models and the environment (Python, gwtools
    dependency) in which the fits and diagnostics are executed
  repo: https://github.com/sxs-collaboration/gwsurrogate
- name: LIGO Analysis Library (LAL)
  role: generates the fiducial EOB waveforms (EOB-LAL) against which pointwise fit
    residuals are evaluated
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1103/physrevx.4.031006
    title: Fast Prediction and Evaluation of Gravitational Waveforms Using Surrogate
      Models
  source_content_hash: sha256:e1f02a20215b497d47ddb4255c291980867f9b77ee847ef9094aedb5802dc162
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GWSurrogate
  - LIGO Analysis Library (LAL)
  - NumPy
  - Python
  - gwtools
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# waveform-parameter-space-fitting

## Summary

Pointwise least-squares fitting of waveform amplitude and phase at each empirical time node of a reduced-basis surrogate, comparing polynomial versus PN-inspired ansatz families versus mass ratio (q = 1 to 10). This is the third stage of the surrogate pipeline, turning greedy-selected empirical data into a fast parameter-space interpolant for EOB-LAL waveforms.

## When to use

Use when a reduced-basis/empirical-interpolant surrogate construction needs the waveform's value at each of the m empirical time nodes fitted across the parameter dimension, and fit residuals must be validated against fiducial EOB waveforms.

## When NOT to use

- The goal is building the reduced basis or selecting empirical time nodes via the greedy algorithm — this skill covers only the third (fitting) stage of the surrogate pipeline
- The waveforms are numerical-relativity or precessing-spin surrogates (e.g. NRSur7dq4, NRHybSur3dq8) rather than the nonspinning EOB fiducial family; the PN-inspired ansatz and error thresholds were derived for nonspinning EOB-LAL waveforms

## Inputs

- Amplitude and phase of the reduced-basis-represented waveform evaluated at each of the m empirical time nodes
- Mass-ratio training grid over q = 1 to 10
- Fiducial EOB waveforms generated with the EOB solver as implemented in LAL (EOB-LAL code)

## Outputs

- Per-node fit coefficients for amplitude and phase (polynomial or PN-inspired ansatz, whichever performs better at that node)
- Per-node error diagnostics (pointwise fit residuals vs mass ratio)
- fit_coefficients_and_errors.csv

## How to apply

Load the greedy-selected training data: amplitude and phase of the reduced-basis-represented waveform evaluated at each of the m empirical time nodes, sampled over the mass-ratio training interval (q = 1 to 10). At each node, perform a polynomial least-squares fit of amplitude and phase versus mass ratio using NumPy least-squares regression (Eq. 25). In parallel, fit the PN-inspired ansatz (Eqs. 28-29) that combines polynomial terms with inverse-power dependence on the symmetric mass ratio, and select the better-performing fit family per node. Compute pointwise fit residuals between the fitted surrogate values and the fiducial (EOB-LAL) waveform values across the parameter grid; expect pointwise errors uniformly below 3e-3 and amplitude fit errors better than 1e-5 in the inspiral, rising to roughly 1e-3 at merger/ringdown. Write the per-node fit coefficients and error diagnostics to fit_coefficients_and_errors.csv with Python/gwtools.

## Related tools

- **NumPy** (least-squares regression for the polynomial and PN-inspired fits of amplitude and phase versus mass ratio at each empirical node) — https://pypi.org/project/gwsurrogate
- **gwtools** (writing fit coefficients and per-node error diagnostics to CSV; dependency installed automatically with gwsurrogate) — https://bitbucket.org/chadgalley/gwtools/
- **GWSurrogate** (public package providing surrogate models and the environment (Python, gwtools dependency) in which the fits and diagnostics are executed) — https://github.com/sxs-collaboration/gwsurrogate
- **LIGO Analysis Library (LAL)** (generates the fiducial EOB waveforms (EOB-LAL) against which pointwise fit residuals are evaluated)

## Examples

```
coeffs, errs = np.linalg.lstsq(A(q_grid), amp_nodes[i, :], rcond=None)
```

## Evaluation signals

- Pointwise least-squares errors for amplitude and phase at the 15th empirical node, evaluated on 1,000 randomly selected EOB waveforms, are uniformly below 3e-3
- Amplitude fit errors better than 1e-5 in the inspiral, rising to approximately 1e-3 at merger/ringdown
- Per-node comparison of polynomial (Eq. 25) vs PN-inspired (Eqs. 28-29) fit residuals confirms the better family is selected per node
- fit_coefficients_and_errors.csv contains coefficients for all m nodes together with per-node error diagnostics
- The fitted surrogate, when evaluated online, reproduces fiducial EOB waveforms across the full mass-ratio range q = 1 to 10

## Limitations

- Fit accuracy degrades at merger/ringdown: amplitude errors rise from ~1e-5 (inspiral) to ~1e-3, so the fit quality is phase-dependent along the waveform
- The training interval is limited to nonspinning binaries with mass ratios q = 1 to 10 and durations up to 10^5M; extrapolation beyond this range is not covered by the fit validation
- Fits are performed only at the m empirical time nodes with m << L, so accuracy between nodes relies on the empirical interpolant, not on the fits themselves

## Evidence

- [abstract] Third stage: pointwise fits at empirical nodes: [evidence span withheld — anchor status: not_found]
- [abstract] Empirical node selection: "Second, these m parameters induce the selection of m time values for interpolating a waveform time series using an empirical interpolant"
- [abstract] EOB-LAL fiducial waveforms: "generating a fiducial waveform using the EOB solver as implemented in the LIGO Analysis Library (LAL) software [54], which we refer to as the EOB-LAL code."
- [abstract] Mass-ratio training range: [evidence span withheld — anchor status: not_found]
- [readme] gwtools dependency: "gwtools. If you are installing gwsurrogate with pip you will automatically get gwtools."
- [results] Fit error threshold: [evidence span withheld — anchor status: not_found]
