---
name: empirical-interpolant-reconstruction
description: Use when a fiducial waveform family (e.g. EOB-LAL nonspinning BBH modes)
  has already been compressed into greedy-reduced-basis coefficients plus empirical
  interpolant times, and fast reconstruction at new parameters (mass ratio q, spins
  chiA/chiB) with pointwise error checks is required.
when_to_use_negative:
- When a suitable pre-built surrogate already exists in the gwsurrogate catalog (e.g.
  NRSur7dq4, NRHybSur3dq8) covering the requested q and spin range — pull and evaluate
  it instead of rebuilding the interpolant offline
- When the target parameter domain or waveform physics differs from the fiducial family
  the reduced basis and empirical interpolant were built for (the interpolant is only
  valid for its training domain, e.g. nonspinning q in [1,4])
- When offline solve cost is irrelevant and only a handful of waveforms are needed
  — direct EOB-LAL generation avoids the entire greedy/interpolant/fitting construction
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: gwsurrogate
  role: Pulls surrogate catalog data and loads the fitted coefficients and empirical
    interpolant basis; provides the sur(q, chiA, chiB, dt, f_low) evaluation API
  repo: https://github.com/sxs-collaboration/gwsurrogate
- name: gwtools
  role: Waveform utilities for computing pointwise amplitude/phase differences and
    mismatches between surrogate reconstructions and direct EOB-LAL waveforms
  repo: https://bitbucket.org/chadgalley/gwtools/
- name: NumPy
  role: Draws the random held-out mass ratios (e.g. 1,000 samples uniform in [1,2])
    and handles the array-level reconstruction arithmetic
- name: LIGO Analysis Library (LAL)
  role: EOB-LAL code generating the fiducial waveforms used for greedy basis building
    and held-out error validation
- name: pytest
  role: Package regression tests used to cross-check stored waveform outputs of the
    surrogate evaluations
provenance:
  source_task_ids:
  - task_005
  - task_008
  source_papers:
  - doi: 10.1103/physrevx.4.031006
    title: Fast Prediction and Evaluation of Gravitational Waveforms Using Surrogate
      Models
  source_content_hash: sha256:558eaf9560838bc0ae2769b793989d408fc4573695abdc0d0531dbe830312e75
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sxs-collaboration/gwsurrogate
  tools:
  - GWSurrogate
  - LAL
  - LIGO Analysis Library (LAL)
  - NumPy
  - Python
  - gwsurrogate
  - gwtools
  - pytest
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# empirical-interpolant-reconstruction

## Summary

Reconstruct gravitational-waveform time series at arbitrary binary parameters from a compressed surrogate: an m-point reduced basis, an empirical interpolant selecting m waveform times, and parameter-dimension fits, yielding O(mL + m*c_fit) online evaluation instead of solving the underlying EOB/NR equations.

## When to use

Use when a fiducial waveform family (e.g. EOB-LAL nonspinning BBH modes) has already been compressed into greedy-reduced-basis coefficients plus empirical interpolant times, and fast reconstruction at new parameters (mass ratio q, spins chiA/chiB) with pointwise error checks is required.

## When NOT to use

- When a suitable pre-built surrogate already exists in the gwsurrogate catalog (e.g. NRSur7dq4, NRHybSur3dq8) covering the requested q and spin range — pull and evaluate it instead of rebuilding the interpolant offline
- When the target parameter domain or waveform physics differs from the fiducial family the reduced basis and empirical interpolant were built for (the interpolant is only valid for its training domain, e.g. nonspinning q in [1,4])
- When offline solve cost is irrelevant and only a handful of waveforms are needed — direct EOB-LAL generation avoids the entire greedy/interpolant/fitting construction

## Inputs

- Training set of fiducial waveform time series (e.g. (2,2)-mode EOB-LAL waveforms) spanning a parameter domain such as q in [1,2] or [3,4]
- Mass ratio q and dimensionless spins chiA, chiB at which to reconstruct the waveform
- Time-sampling parameters: dt (units of total mass M), f_low (0 to return full surrogate), target number of samples L (e.g. L=16,384)

## Outputs

- Reconstructed waveform modes h(t) and the time array t at the requested parameter point
- Greedy residuals and reduced-basis selection logs per iteration
- Empirical interpolant time/parameter selections and per-time coefficient fits
- Validation table of surrogate errors (L2, L-infinity, pointwise amplitude/phase) vs direct EOB-LAL generation, e.g. surrogate_validation_results.csv or eob22_surrogate_metrics.csv

## How to apply

Run the offline pipeline in the paper's three stages: (1) greedily select m parameter values from the training set and build the reduced basis; (2) let those m parameters induce m empirical time values interpolating the fiducial waveform family; (3) fit the waveform value at each of the m empirical times across the parameter dimension to produce online-ready coefficient fits. Online, load the surrogate with gwsurrogate.LoadSurrogate after gwsurrogate.catalog.pull, then call sur(q, chiA, chiB, dt, f_low) with dt in units of total mass M and f_low=0 to return the full waveform and L time samples (e.g. L=16,384). Reconstruct at held-out parameters, compute L2/L-infinity surrogate errors and pointwise amplitude/phase differences against directly generated EOB-LAL waveforms (via gwtools), and accept the surrogate when errors are below thresholds such as the paper's ~1e-7 surrogate error or Table-I-level L2 error 2e-7 / L-infinity 2e-3. The rationale is that m << L, so online cost is O(mL + m*c_fit) rather than solving the EOB differential equations, and direct waveform generation only supplies the offline training and validation data.

## Related tools

- **gwsurrogate** (Pulls surrogate catalog data and loads the fitted coefficients and empirical interpolant basis; provides the sur(q, chiA, chiB, dt, f_low) evaluation API) — https://github.com/sxs-collaboration/gwsurrogate
- **gwtools** (Waveform utilities for computing pointwise amplitude/phase differences and mismatches between surrogate reconstructions and direct EOB-LAL waveforms) — https://bitbucket.org/chadgalley/gwtools/
- **NumPy** (Draws the random held-out mass ratios (e.g. 1,000 samples uniform in [1,2]) and handles the array-level reconstruction arithmetic)
- **LIGO Analysis Library (LAL)** (EOB-LAL code generating the fiducial waveforms used for greedy basis building and held-out error validation)
- **pytest** (Package regression tests used to cross-check stored waveform outputs of the surrogate evaluations)

## Examples

```
import gwsurrogate; gwsurrogate.catalog.pull('NRSur7dq4'); sur = gwsurrogate.LoadSurrogate('NRSur7dq4'); t, h, dyn = sur(q, chiA, chiB, dt=0.1, f_low=0)
```

## Evaluation signals

- Surrogate error must be uniformly below ~1e-7 over 1,000 randomly sampled mass ratios in the paper's EOB test case, with the largest error at q = 1.068
- Extended-domain (q in [3,4], (2,2)-mode, m=25 basis) benchmarks must reach L2 error ~2e-7 and L-infinity error ~2e-3 against directly generated EOB-LAL waveforms
- Pointwise amplitude and phase errors between surrogate and reference waveforms must be at or below ~1e-3 (computed with gwtools)
- Surrogate evaluation time must be orders of magnitude faster than direct generation (paper: ~1e-4 s, speedup ~1,800 vs EOB-LAL); online cost must scale as O(mL + m*c_fit) with m << L
- Reconstructed waveforms must match the outputs of gwsurrogate's pytest regression suite

## Limitations

- The empirical interpolant and coefficient fits are only valid inside the training domain of the fiducial family (e.g. the nominal nonspinning q in [1,2] example); extending to q in [3,4] requires rebuilding the basis with new greedy selections
- Offline construction requires solving the underlying equations (EOB via LAL or NR simulations); gwsurrogate only accelerates the online evaluation phase
- The paper's headline error figures apply to nonspinning fiducial families; precessing/spinning models rely on separately trained surrogates and additional precessing_opts

## Evidence

- [abstract] greedy reduced-basis selection: "First, a set of m parameter values is determined using a greedy algorithm from which a reduced basis representation is constructed."
- [abstract] empirical interpolant time selection: "Second, these m parameters induce the selection of m time values for interpolating a waveform time series using an empirical interpolant that is built for the fiducial waveform family."
- [abstract] online cost model: [evidence span withheld — anchor status: not_found]
- [intro] offline vs online separation: "These equations must be solved in the ``building" phase, which was performed using other codes."
- [readme] catalog pull and load invocation: "gwsurrogate.catalog.pull('NRSur7dq4')       # This can take a few minutes"
