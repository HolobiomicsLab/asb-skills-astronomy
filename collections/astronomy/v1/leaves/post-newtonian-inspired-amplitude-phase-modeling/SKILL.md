---
name: post-newtonian-inspired-amplitude-phase-modeling
description: Use when building a reduced-basis/empirical-interpolant surrogate for
  a waveform family and you need cheap parametric fits of amplitude and phase at each
  empirical time node across a mass-ratio training interval (e.g., q = 1 to 10).
when_to_use_negative:
- The goal is evaluating an already-built surrogate (e.g., via gwsurrogate.LoadSurrogate)
  rather than constructing the parametric fits themselves
- The waveform parameter space includes precession or spins, since the fits here are
  one-dimensional functions of mass ratio for nonspinning BBHs with q = 1 to 10
- Empirical time nodes have not yet been selected (the fits are defined pointwise
  at the m nodes of the empirical interpolant)
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: NumPy
  role: Least-squares regression used for the polynomial fits of amplitude and phase
    versus mass ratio at each empirical time node
- name: gwtools
  role: Companion utilities for writing fit coefficients and error diagnostics to
    fit_coefficients_and_errors.csv
  repo: https://bitbucket.org/chadgalley/gwtools/
- name: GWSurrogate
  role: Public package distributing surrogate models built with this fitting procedure;
    installed via pip or conda-forge
  repo: https://github.com/sxs-collaboration/gwsurrogate
- name: Python
  role: Implementation environment for the fitting workflow (e.g., conda env with
    python=3.11 and numpy<2.0)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1103/physrevx.4.031006
    title: Fast Prediction and Evaluation of Gravitational Waveforms Using Surrogate
      Models
  source_content_hash: sha256:c2e4f8f5236d76acd997b20177f36f6cbb1918f8a94f25ada470bb17d6d17d6b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GWSurrogate
  - NumPy
  - Python
  - gwtools
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# post-newtonian-inspired-amplitude-phase-modeling

## Summary

Fits the waveform amplitude and phase, evaluated at each of the m greedy-selected empirical time nodes, as functions of mass ratio using either polynomial least-squares (Eq. 25) or a post-Newtonian-inspired ansatz (Eqs. 28-29) combining polynomial terms with inverse powers of the symmetric mass ratio, selecting the better family per node for a compact surrogate model.

## When to use

Use when building a reduced-basis/empirical-interpolant surrogate for a waveform family and you need cheap parametric fits of amplitude and phase at each empirical time node across a mass-ratio training interval (e.g., q = 1 to 10).

## When NOT to use

- The goal is evaluating an already-built surrogate (e.g., via gwsurrogate.LoadSurrogate) rather than constructing the parametric fits themselves
- The waveform parameter space includes precession or spins, since the fits here are one-dimensional functions of mass ratio for nonspinning BBHs with q = 1 to 10
- Empirical time nodes have not yet been selected (the fits are defined pointwise at the m nodes of the empirical interpolant)

## Inputs

- Greedy-selected training data: amplitude and phase of the reduced-basis-represented EOB-LAL waveform at each of the m empirical time nodes, over mass ratio q = 1 to 10
- Fiducial EOB-LAL waveform values at each empirical node for validation (e.g., 1,000 randomly selected EOB waveforms)

## Outputs

- Per-node fit coefficients for amplitude and phase (polynomial or PN-inspired ansatz, Eq. 25 / Eqs. 28-29)
- Per-node error diagnostics (pointwise fit residuals vs. fiducial waveforms)
- fit_coefficients_and_errors.csv

## How to apply

Load the greedy-selected training data: the amplitude and phase of the reduced-basis-represented EOB-LAL waveform evaluated at each of the m empirical time nodes, sampled over the mass-ratio interval q = 1 to 10. At each node, perform a polynomial least-squares fit (Eq. 25) of amplitude and phase versus mass ratio with NumPy least-squares regression, and in parallel fit the PN-inspired ansatz (Eqs. 28-29) that combines polynomial terms with inverse-power dependence on the symmetric mass ratio; select the better-performing fit family per node. Compute pointwise fit residuals between the fitted surrogate values and the fiducial EOB-LAL waveform values at each node on ~1,000 randomly selected EOB waveforms, then write all fit coefficients and per-node error diagnostics to fit_coefficients_and_errors.csv using Python/gwtools. Accept fits only when pointwise errors are uniformly below 3e-3 and amplitude errors are better than 1e-5 in the inspiral.

## Related tools

- **NumPy** (Least-squares regression used for the polynomial fits of amplitude and phase versus mass ratio at each empirical time node)
- **gwtools** (Companion utilities for writing fit coefficients and error diagnostics to fit_coefficients_and_errors.csv) — https://bitbucket.org/chadgalley/gwtools/
- **GWSurrogate** (Public package distributing surrogate models built with this fitting procedure; installed via pip or conda-forge) — https://github.com/sxs-collaboration/gwsurrogate
- **Python** (Implementation environment for the fitting workflow (e.g., conda env with python=3.11 and numpy<2.0))

## Examples

```
sur = gwsurrogate.LoadSurrogate('NRSur7dq4'); t, h, dyn = sur(q, chiA, chiB, dt=dt, f_low=f_low)
```

## Evaluation signals

- Pointwise least-squares errors for amplitude and phase at the 15th empirical node, evaluated on 1,000 randomly selected EOB waveforms, are uniformly below 3 x 10^-3
- Amplitude fit errors are better than 1e-5 in the inspiral, rising to approximately 1e-3 at merger/ringdown
- Per-node comparison confirms the PN-inspired ansatz (Eqs. 28-29) outperforms or matches the pure polynomial fit (Eq. 25) at each node
- fit_coefficients_and_errors.csv contains coefficients for all m nodes together with per-node error diagnostics

## Limitations

- Fit accuracy degrades toward merger/ringdown: amplitude errors rise from <1e-5 in the inspiral to ~1e-3 at merger/ringdown
- The fits are parameterized only by mass ratio for nonspinning binary black holes (q = 1 to 10), so they do not cover spin or precessing parameter spaces
- gwsurrogate C-extensions require NumPy C-API headers at build time; use numpy<2.0 (e.g., conda create -n myenv python=3.11 'numpy<2.0') if explicit control is needed

## Evidence

- [abstract] polynomial least-squares fits at empirical nodes: [evidence span withheld — anchor status: not_found]
- [methods] fiducial EOB-LAL waveforms: "generating a fiducial waveform using the EOB solver as implemented in the LIGO Analysis Library (LAL) software [54], which we refer to as the EOB-LAL code."
- [abstract] surrogate mass-ratio coverage: [evidence span withheld — anchor status: not_found]
- [readme] gwsurrogate availability: "Surrogates provide a fast and accurate evaluation mechanism for gravitational
waveforms, which would otherwise be found through solving differential
equations."
- [readme] numpy version constraint: "conda create -n myenv python=3.11 "numpy<2.0""
