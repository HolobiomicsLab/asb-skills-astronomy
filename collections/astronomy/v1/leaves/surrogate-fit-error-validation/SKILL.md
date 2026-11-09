---
name: surrogate-fit-error-validation
description: Use when building or auditing a reduced-basis surrogate where waveform
  amplitude/phase values at each of m empirical time nodes must be fit across a parameter
  grid (e.g. mass ratio q = 1 to 10) and the fit errors must be quantified and bounded.
when_to_use_negative:
- The waveform data is not decomposed onto a reduced basis with empirical interpolant
  nodes — direct full-time-series fits would not respect the O(mL + m*c_fit) surrogate
  evaluation structure
- The task is online surrogate evaluation at a generic parameter choice, not the offline
  building/validation of the fits themselves
- Parameters include precessing spins; the polynomial / PN-inspired fits here are
  validated only over the nonspinning mass-ratio training interval (q = 1 to 10)
tools:
- name: NumPy
  role: Least-squares regression used to perform the polynomial fit (Eq. 25) of amplitude
    and phase versus mass ratio at each empirical node
- name: GWSurrogate
  role: Interface to gravitational wave surrogate models; provides the reduced-basis/empirical-interpolant
    surrogate context in which the node-wise fits live
  repo: https://github.com/sxs-collaboration/gwsurrogate
- name: gwtools
  role: Companion utility library (pulled in automatically with gwsurrogate via pip)
    used for waveform utilities in the fit and error-diagnostic workflow
  repo: https://bitbucket.org/chadgalley/gwtools/
- name: LIGO Analysis Library (LAL)
  role: Source of the fiducial EOB waveforms (EOB-LAL code) against which the fit
    residuals are computed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1103/physrevx.4.031006
    title: Fast Prediction and Evaluation of Gravitational Waveforms Using Surrogate
      Models
  source_content_hash: sha256:adcc6d45e836a2c48cfd2171f9e0d0b1609bb413fd2857780206a9efb605bb4c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sxs-collaboration/gwsurrogate
  tools:
  - GWSurrogate
  - LIGO Analysis Library (LAL)
  - NumPy
  - Python
  - gwtools
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# surrogate-fit-error-validation

## Summary

Validates the parameter-dimension fits of a surrogate model by least-squares fitting waveform amplitude and phase at each empirical time node of a greedy/reduced-basis surrogate, comparing polynomial vs PN-inspired ansätze, and checking pointwise residuals against explicit error thresholds (e.g. < 3e-3 pointwise).

## When to use

Use when building or auditing a reduced-basis surrogate where waveform amplitude/phase values at each of m empirical time nodes must be fit across a parameter grid (e.g. mass ratio q = 1 to 10) and the fit errors must be quantified and bounded.

## When NOT to use

- The waveform data is not decomposed onto a reduced basis with empirical interpolant nodes — direct full-time-series fits would not respect the O(mL + m*c_fit) surrogate evaluation structure
- The task is online surrogate evaluation at a generic parameter choice, not the offline building/validation of the fits themselves
- Parameters include precessing spins; the polynomial / PN-inspired fits here are validated only over the nonspinning mass-ratio training interval (q = 1 to 10)

## Inputs

- Greedy-selected training data: amplitude and phase of the reduced-basis-represented waveform evaluated at each of the m empirical time nodes, over the mass-ratio training interval q = 1 to 10
- Fiducial EOB-LAL waveform values at each empirical node across the parameter grid (for residual computation)
- Mass-ratio parameter grid and symmetric mass ratio values for the fit ansätze

## Outputs

- fit_coefficients_and_errors.csv containing per-node fit coefficients and per-node error diagnostics
- Pointwise fit residual reports (amplitude and phase) evaluated on randomly selected EOB waveforms
- Per-node selection record of polynomial vs PN-inspired fit family

## How to apply

Load the greedy-selected training data: amplitude and phase of the reduced-basis-represented EOB waveform evaluated at each of the m empirical time nodes, sampled over the mass-ratio training interval (q = 1 to 10). At each node, perform a polynomial least-squares fit (Eq. 25) of amplitude and phase versus mass ratio using NumPy least-squares regression, and in parallel fit the PN-inspired ansatz (Eqs. 28-29) combining polynomial terms with inverse-power dependence on the symmetric mass ratio, selecting the better-performing family per node. Compute pointwise fit residuals between the surrogate fitted values and the fiducial EOB-LAL waveform values at each node across the parameter grid, evaluated on a large held-out set (e.g. 1,000 randomly selected EOB waveforms). Verify that pointwise fit errors are uniformly below 3e-3 and that amplitude fit errors are better than 1e-5 in the inspiral, rising to approximately 1e-3 at merger/ringdown. Write the fit coefficients for all m nodes together with the per-node error diagnostics to fit_coefficients_and_errors.csv.

## Related tools

- **NumPy** (Least-squares regression used to perform the polynomial fit (Eq. 25) of amplitude and phase versus mass ratio at each empirical node)
- **GWSurrogate** (Interface to gravitational wave surrogate models; provides the reduced-basis/empirical-interpolant surrogate context in which the node-wise fits live) — https://github.com/sxs-collaboration/gwsurrogate
- **gwtools** (Companion utility library (pulled in automatically with gwsurrogate via pip) used for waveform utilities in the fit and error-diagnostic workflow) — https://bitbucket.org/chadgalley/gwtools/
- **LIGO Analysis Library (LAL)** (Source of the fiducial EOB waveforms (EOB-LAL code) against which the fit residuals are computed)

## Examples

```
import numpy as np; coeffs = np.linalg.lstsq(design_matrix, A_node, rcond=None)[0]  # polynomial fit (Eq. 25) of amplitude vs mass ratio at each empirical node
```

## Evaluation signals

- Pointwise least-squares errors for amplitude and phase at the 15th empirical node, evaluated on 1,000 randomly selected EOB waveforms, are uniformly below 3e-3
- Amplitude fit errors are better than 1e-5 in the inspiral, rising to approximately 1e-3 at merger/ringdown
- Per-node comparison shows the selected fit family (polynomial vs PN-inspired ansatz) achieves lower residual than the alternative at each node
- fit_coefficients_and_errors.csv exists and contains coefficients for all m nodes plus per-node error diagnostics

## Limitations

- Fit accuracy is parameter-family dependent: the polynomial and PN-inspired ansätze are validated over the nonspinning mass-ratio range q = 1 to 10 only
- Fit errors are not uniform in waveform phase of evolution: amplitude errors of ~1e-3 occur at merger/ringdown versus 1e-5 in the inspiral, so threshold checks must be phase-dependent
- Residuals must be checked against fiducial EOB-LAL waveforms on a random held-out set; passing on the greedy training parameters alone does not guarantee uniform accuracy
- gwsurrogate modules are C-extensions requiring NumPy C-API headers at build time; NumPy 1.x vs 2.x compatibility must be managed (e.g. conda create -n myenv python=3.11 "numpy<2.0")

## Evidence

- [abstract] Third, a fit in the parameter dimension is performed for the waveform's value at each of these m times.: [evidence span withheld — anchor status: not_found]
- [abstract] these m parameters induce the selection of m time values for interpolating a waveform time series using an empirical interpolant that is built for the fiducial waveform family.: "Second, these m parameters induce the selection of m time values for interpolating a waveform time series using an empirical interpolant that is built for the fiducial waveform family."
- [abstract] a set of m parameter values is determined using a greedy algorithm from which a reduced basis representation is constructed.: "First, a set of m parameter values is determined using a greedy algorithm from which a reduced basis representation is constructed."
- [abstract] generating a fiducial waveform using the EOB solver as implemented in the LIGO Analysis Library (LAL) software [54], which we refer to as the EOB-LAL code.: "generating a fiducial waveform using the EOB solver as implemented in the LIGO Analysis Library (LAL) software [54], which we refer to as the EOB-LAL code."
- [readme] gwsurrogate requires: 1) gwtools. If you are installing gwsurrogate with pip you will automatically get gwtools.: "gwsurrogate requires:

1)  gwtools. If you are installing gwsurrogate with pip you
will automatically get gwtools."
- [readme] Certain gwsurrogate modules are implemented as C-extensions and require NumPy’s C-API headers at build time.: "Certain gwsurrogate modules are implemented as C-extensions and require NumPy’s C-API headers at build time."
