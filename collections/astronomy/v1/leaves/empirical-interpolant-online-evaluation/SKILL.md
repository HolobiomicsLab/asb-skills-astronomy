---
name: empirical-interpolant-online-evaluation
description: Use when a fast, accurate waveform prediction is needed at a generic
  parameter choice after a surrogate (reduced basis + empirical interpolant + fits)
  has already been built, and solving EOB/NR equations online is too costly.
when_to_use_negative:
- No surrogate has been built yet for the waveform family — the offline building phase
  (greedy reduced basis, empirical interpolant, parameter fits) must precede online
  evaluation
- Parameters fall outside the surrogate's training domain (e.g. mass ratios beyond
  q<=8 for NRHybSur3dq8 or q<=4 for NRSur7dq4)
- A full, solver-level EOB evolution with dynamics beyond the interpolated waveform
  samples is required
edam_operation: http://edamontology.org/operation_0226
tools:
- name: GWSurrogate
  role: Public Python package providing catalog.pull for downloading surrogate data,
    LoadSurrogate for loading it, and the evaluation API used for online prediction
  repo: https://github.com/sxs-collaboration/gwsurrogate
- name: gwtools
  role: Required dependency of gwsurrogate, pulled in automatically via pip
  repo: https://bitbucket.org/chadgalley/gwtools/
- name: NumPy
  role: Provides the C-API used by gwsurrogate's C-extension modules during surrogate
    evaluation
  repo: https://pypi.org/project/gwsurrogate
- name: LIGO Analysis Library (LAL)
  role: EOB-LAL waveform generation serves as the timing baseline against which surrogate
    evaluation speedup is measured
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1103/physrevx.4.031006
    title: Fast Prediction and Evaluation of Gravitational Waveforms Using Surrogate
      Models
  source_content_hash: sha256:10d50c8bfa8c3af01aa82f2abb47c206ff0af1e43bf37e39473570c00d524011
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
  edam_operation: http://edamontology.org/operation_0226
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 18
---

# empirical-interpolant-online-evaluation

## Summary

Evaluates a pre-built gravitational-wave surrogate model online — interpolating the waveform at m empirically selected time values and applying parameter-dimension fits — to predict L time samples in O(mL + m*c_fit) operations instead of solving EOB differential equations.

## When to use

Use when a fast, accurate waveform prediction is needed at a generic parameter choice after a surrogate (reduced basis + empirical interpolant + fits) has already been built, and solving EOB/NR equations online is too costly.

## When NOT to use

- No surrogate has been built yet for the waveform family — the offline building phase (greedy reduced basis, empirical interpolant, parameter fits) must precede online evaluation
- Parameters fall outside the surrogate's training domain (e.g. mass ratios beyond q<=8 for NRHybSur3dq8 or q<=4 for NRSur7dq4)
- A full, solver-level EOB evolution with dynamics beyond the interpolated waveform samples is required

## Inputs

- Binary parameters: mass ratio q (mA/mB >= 1), dimensionless spin vectors chiA and chiB, timestep dt (units of total mass M), initial frequency f_low
- Pre-built surrogate model data downloaded via gwsurrogate.catalog.pull (e.g. NRSur7dq4, NRHybSur3dq8, EOBNRv2-style EOB surrogates)
- Number of time samples L / target sampling rate (e.g. 2,048 Hz) for timing and scaling studies

## Outputs

- Time array t, waveform modes h (multiple spherical-harmonic modes), and dynamics dyn returned by the surrogate evaluation call
- Wall-clock timings and speedup ratios (surrogate vs. EOB-LAL) as a function of number of samples L, e.g. written to timing_comparison.csv

## How to apply

Load the pre-built surrogate data (e.g. an EOBNRv2 model with m=10 greedy-selected empirical-interpolant elements) once per script with gwsurrogate.catalog.pull followed by gwsurrogate.LoadSurrogate. Evaluate it at the target parameters (mass ratio q, spin vectors chiA/chiB, timestep dt in units of total mass M, low frequency f_low) to obtain the time series and waveform modes; the online cost is O(mL + m*c_fit) with m << L, so wall time scales linearly in the number of samples L rather than with the EOB solver cost. For timing studies, sweep sampling rates up to 2,048 Hz, time both surrogate and EOB-LAL generation, and compute the per-rate speedup ratio (expect ~2,300x at 2,048 Hz and between 2 and nearly 4 orders of magnitude overall). The offline building phase (greedy parameter selection, empirical interpolant construction, parameter fits) must already be complete — this skill only covers online evaluation.

## Related tools

- **GWSurrogate** (Public Python package providing catalog.pull for downloading surrogate data, LoadSurrogate for loading it, and the evaluation API used for online prediction) — https://github.com/sxs-collaboration/gwsurrogate
- **gwtools** (Required dependency of gwsurrogate, pulled in automatically via pip) — https://bitbucket.org/chadgalley/gwtools/
- **NumPy** (Provides the C-API used by gwsurrogate's C-extension modules during surrogate evaluation) — https://pypi.org/project/gwsurrogate
- **LIGO Analysis Library (LAL)** (EOB-LAL waveform generation serves as the timing baseline against which surrogate evaluation speedup is measured)

## Examples

```
gwsurrogate.catalog.pull('EOBNRv2'); sur = gwsurrogate.LoadSurrogate('EOBNRv2'); t, h, dyn = sur(q=10, chiA=[0,0,0], chiB=[0,0,0], dt=1/2048, f_low=0)
```

## Evaluation signals

- Measured surrogate evaluation wall time scales linearly with the number of time samples L, consistent with the O(mL + m*c_fit) online cost with m << L
- At a 2,048 Hz sampling rate the surrogate-to-EOB-LAL speedup is approximately 2,300x
- Overall speedups fall between 2 and nearly 4 orders of magnitude (consistent with the reported 'more than 3 orders of magnitude faster' result)
- Surrogate prediction reproduces the fiducial waveform family's physics — output modes h match EOB-LAL waveforms within the surrogate's stated accuracy at the same parameters

## Limitations

- The m greedy parameter values and m empirical time values are fixed offline; online evaluation cannot improve accuracy beyond the fits built during the building phase
- Speedup depends on sampling rate — only ~2 orders of magnitude at low rates versus nearly 4 at high rates, since cost is linear in L
- gwsurrogate modules are C-extensions requiring NumPy C-API headers at build time; NumPy 1.x vs 2.x compatibility needs attention (e.g. conda env with python=3.11 and numpy<2.0)

## Evidence

- [abstract] Online cost scaling: [evidence span withheld — anchor status: not_found]
- [abstract] Empirical interpolant construction: "Second, these m parameters induce the selection of m time values for interpolating a waveform time series using an empirical interpolant that is built for the fiducial waveform family."
- [abstract] Speedup over standard EOB generation: "We find that these surrogates are more than 3 orders of magnitude faster to evaluate as compared to the cost of generating effective-one-body waveforms in standard ways."
- [readme] Surrogate vs. solving differential equations: "Surrogates provide a fast and accurate evaluation mechanism for gravitational waveforms, which would otherwise be found through solving differential equations."
- [methods] Concrete evaluation call: "t, h, dyn = sur(q, chiA, chiB, dt=dt, f_low=f_low, precessing_opts=precessing_opts)"
- [readme] Load the surrogate once: "Load the surrogate, this only needs to be done once at the start of a script"
