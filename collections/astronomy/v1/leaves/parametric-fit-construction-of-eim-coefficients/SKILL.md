---
name: parametric-fit-construction-of-eim-coefficients
description: Use when a reduced-basis + empirical-interpolant representation of a
  waveform family (e.g., (2,2)-mode EOB-LAL over q∈[1,4]) already exists and the m
  waveform values at the m empirical time points must be fitted across the parameter
  dimension to produce an online-ready surrogate.
when_to_use_negative:
- No reduced basis or empirical interpolant has been built yet — the fits require
  the m greedy parameters and m empirical time values as prerequisites
- The waveform family depends on many strongly coupled parameters with sparse sampling;
  univariate fits per empirical time may not capture the parameter dependence without
  additional modelling
- Goal is direct waveform generation from the EOB solver (EOB-LAL) rather than fast
  surrogate evaluation — the fit stage only accelerates evaluation after the offline
  build
tools:
- name: Python
  role: Implementation language for performing the per-empirical-time parametric fits
    and assembling the surrogate coefficients
  repo: https://github.com/sxs-collaboration/gwsurrogate
- name: NumPy
  role: Numerical library used for the fitting and array operations in the offline
    build of coefficient fits
  repo: https://github.com/sxs-collaboration/gwsurrogate
- name: GWSurrogate
  role: Publicly available Python package in which built surrogates (with their coefficient
    fits) are distributed and evaluated
  repo: https://github.com/sxs-collaboration/gwsurrogate
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1103/physrevx.4.031006
    title: Fast Prediction and Evaluation of Gravitational Waveforms Using Surrogate
      Models
  source_content_hash: sha256:bdf01298410ca8bbe1bd019e09096c7d718039588ae056e1c68cc94c122ef9d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sxs-collaboration/gwsurrogate
  tools:
  - GWSurrogate
  - LAL
  - NumPy
  - Python
  - gwtools
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# Parametric fit construction of EIM coefficients

## Summary

Third phase of the gravitational-wave surrogate construction pipeline of Field et al. (PRX 4, 031006): for each of the m empirical-interpolant time values selected from the reduced basis, fit the fiducial waveform's value as a smooth function of the parameter (e.g., mass ratio q), yielding compact coefficient fits that make the surrogate fast to evaluate online.

## When to use

Use when a reduced-basis + empirical-interpolant representation of a waveform family (e.g., (2,2)-mode EOB-LAL over q∈[1,4]) already exists and the m waveform values at the m empirical time points must be fitted across the parameter dimension to produce an online-ready surrogate.

## When NOT to use

- No reduced basis or empirical interpolant has been built yet — the fits require the m greedy parameters and m empirical time values as prerequisites
- The waveform family depends on many strongly coupled parameters with sparse sampling; univariate fits per empirical time may not capture the parameter dependence without additional modelling
- Goal is direct waveform generation from the EOB solver (EOB-LAL) rather than fast surrogate evaluation — the fit stage only accelerates evaluation after the offline build

## Inputs

- Reduced basis of m greedy-selected fiducial (2,2)-mode EOB-LAL waveforms spanning the target parameter domain (e.g., q∈[3,4])
- Empirical interpolant with its induced m time values of the waveform
- Vector of fiducial waveform amplitudes at each empirical time point for every training parameter value (e.g., mass ratio q)
- Parameter grid / greedy parameter values used to anchor the fits

## Outputs

- Coefficient fits (parametric representations, one per empirical time) stored as offline surrogate data
- Online-ready surrogate model callable at any parameter value in the trained domain
- Metrics such as L2/L∞ surrogate error and evaluation speedup (e.g., assembled into eob22_surrogate_metrics.csv)

## How to apply

After the greedy algorithm has selected m parameter values and built the reduced basis, and the empirical interpolant has induced m time values for the waveform, extract the fiducial waveform's value at each of those m times for every training parameter point. Fit each column of these values as a smooth function of the parameter (e.g., mass ratio q in the q∈[3,4] EOB extension task, using Python/NumPy), storing the fit coefficients as the surrogate's offline data. The rationale is that online evaluation then reduces to m cheap univariate evaluations plus one evaluation of the reduced basis, giving online cost O(mL + m*c_fit) for L time samples — dramatically cheaper than solving the EOB equations. Judge success by evaluating the fitted surrogate at held-out parameter values and confirming surrogate errors (e.g., L2 ≈ 2×10⁻⁷, L∞ ≈ 2×10⁻³ for the q∈[1,4] (2,2)-mode EOB case) and speedups (≈1,800× vs EOB-LAL) consistent with the reference build.

## Related tools

- **Python** (Implementation language for performing the per-empirical-time parametric fits and assembling the surrogate coefficients) — https://github.com/sxs-collaboration/gwsurrogate
- **NumPy** (Numerical library used for the fitting and array operations in the offline build of coefficient fits) — https://github.com/sxs-collaboration/gwsurrogate
- **GWSurrogate** (Publicly available Python package in which built surrogates (with their coefficient fits) are distributed and evaluated) — https://github.com/sxs-collaboration/gwsurrogate

## Examples

```
import numpy as np; coeffs = np.polyfit(q_greedy, h_at_empirical_times, deg=deg)  # one fit per empirical time value
```

## Evaluation signals

- Number of coefficient fits equals the greedy basis size m — one fit per empirical interpolation time value
- Evaluating fitted coefficients reproduces the fiducial waveform values at the m empirical times to high precision on training parameters
- Surrogate error at held-out q values in the target domain is small (reference case: L2 error 2×10⁻⁷, L∞ error 2×10⁻³)
- Online evaluation time is of order O(mL + m*c_fit) and yields a large speedup (reference: ~1,800× vs EOB-LAL, evaluation ~1×10⁻⁴ s)
- Mismatch against directly generated EOB-LAL waveforms (computed with gwtools) remains negligible at validation parameters

## Limitations

- The fits are built offline against a specific fiducial waveform family (e.g., EOB-LAL (2,2) mode); extending to a new parameter interval requires regenerating the training set, basis, and fits
- Accuracy is bounded by the greedy error of the reduced basis and the quality of the parameter-space fits; coverage outside the trained domain (e.g., beyond q∈[1,4]) is not guaranteed
- Surrogate data must be downloaded/packaged separately (e.g., via gwsurrogate.catalog.pull) before evaluation

## Evidence

- [abstract] fit at each empirical time is the third pipeline stage: [evidence span withheld — anchor status: not_found]
- [abstract] online cost is driven by the coefficient fits and reduced basis: [evidence span withheld — anchor status: not_found]
- [intro] surrogates replace differential-equation solves with fast evaluation: "Surrogates provide a fast and accurate evaluation mechanism for gravitational
waveforms, which would otherwise be found through solving differential
equations."
- [full_text] workflow step for the coefficient fits: [evidence span withheld — anchor status: not_found]
