---
name: sampling-rate-cost-scaling-analysis
description: Use when comparing wall-clock timing of a fast surrogate waveform evaluation
  against the standard EOB solver as a function of time-sampling rate, to verify O(mL
  + m*c_fit) linear-in-L scaling and compute per-rate speedup ratios.
when_to_use_negative:
- The goal is waveform accuracy validation rather than timing/scaling analysis — this
  skill measures cost, not surrogate fidelity
- The baseline solver is not the EOB differential-equation code (e.g. comparing two
  surrogates); the ~2,300x anchor is specific to EOB-LAL
- Only a single sampling rate is of interest and no scaling-in-L check is needed;
  the sweep across rates is the core of this skill
edam_operation: http://edamontology.org/operation_3860
edam_topics: []
tools:
- name: gwsurrogate
  role: Pulls and loads the surrogate model and evaluates it at each sampling rate
    for timing
  repo: https://github.com/sxs-collaboration/gwsurrogate
- name: LIGO Analysis Library (LAL)
  role: EOB solver used as the EOB-LAL timing baseline for waveform generation
- name: gwtools
  role: Required dependency of gwsurrogate for waveform utilities
  repo: https://bitbucket.org/chadgalley/gwtools/
- name: NumPy
  role: Numerical arrays and timing of surrogate evaluation at each sampling rate
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1103/physrevx.4.031006
    title: Fast Prediction and Evaluation of Gravitational Waveforms Using Surrogate
      Models
  source_content_hash: sha256:653bbb63b7875ec44d8456d44742e04817982d01b2c10d691ee5aff71b3cfe95
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
  edam_operation: http://edamontology.org/operation_3860
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# sampling-rate-cost-scaling-analysis

## Summary

Benchmark how a surrogate model's online evaluation cost scales with the number of time samples L and quantify its speedup versus the fiducial differential-equation solver (EOB-LAL) across sampling rates, e.g. up to 2,048 Hz.

## When to use

Use when comparing wall-clock timing of a fast surrogate waveform evaluation against the standard EOB solver as a function of time-sampling rate, to verify O(mL + m*c_fit) linear-in-L scaling and compute per-rate speedup ratios.

## When NOT to use

- The goal is waveform accuracy validation rather than timing/scaling analysis — this skill measures cost, not surrogate fidelity
- The baseline solver is not the EOB differential-equation code (e.g. comparing two surrogates); the ~2,300x anchor is specific to EOB-LAL
- Only a single sampling rate is of interest and no scaling-in-L check is needed; the sweep across rates is the core of this skill

## Inputs

- EOBNRv2 time-domain surrogate model data (m=10 greedy elements) downloaded via gwsurrogate.catalog.pull
- Binary parameter choice for evaluation (e.g. mass ratio q, spins, timestep dt in units of total mass M)
- EOB-LAL waveform generation code for the same parameters and sampling rates

## Outputs

- Per-sampling-rate wall-clock timings for surrogate evaluation and EOB-LAL generation
- Surrogate-to-EOB-LAL speedup ratio at each rate (expect ~2,300 at 2,048 Hz)
- Empirical scaling check that surrogate cost is linear in the number of time samples L, written to timing_comparison.csv

## How to apply

Pull the EOBNRv2 time-domain surrogate via gwsurrogate.catalog.pull and load it with gwsurrogate.LoadSurrogate; the surrogate is built on m=10 greedy-selected empirical-interpolant elements, so its online prediction cost is predicted to be O(mL + m*c_fit) operations with m << L. Sweep the sampling rate from low values up to 2,048 Hz (dt in units of total mass M controls the number of samples L), timing surrogate evaluation for one generic parameter choice at each rate. Time EOB-LAL (EOB solver as implemented in the LIGO Analysis Library) waveform generation at the same rates and parameters to establish the fiducial baseline. Compute the surrogate-to-EOB-LAL speedup ratio at each rate and check that surrogate timing grows linearly in L. Validation anchor: the speedup at 2,048 Hz should be approximately 2,300x, with overall speedups spanning 2 to nearly 4 orders of magnitude.

## Related tools

- **gwsurrogate** (Pulls and loads the surrogate model and evaluates it at each sampling rate for timing) — https://github.com/sxs-collaboration/gwsurrogate
- **LIGO Analysis Library (LAL)** (EOB solver used as the EOB-LAL timing baseline for waveform generation)
- **gwtools** (Required dependency of gwsurrogate for waveform utilities) — https://bitbucket.org/chadgalley/gwtools/
- **NumPy** (Numerical arrays and timing of surrogate evaluation at each sampling rate)

## Examples

```
import gwsurrogate; gwsurrogate.catalog.pull('EOBNRv2'); sur = gwsurrogate.LoadSurrogate('EOBNRv2'); t, h, dyn = sur(q, dt=dt, f_low=f_low)
```

## Evaluation signals

- Measured speedup at 2,048 Hz sampling rate is approximately 2,300x over EOB-LAL
- Overall speedup factors fall between 2 and nearly 4 orders of magnitude across the swept rates
- Surrogate evaluation wall-clock time scales linearly with the number of time samples L (consistent with O(mL + m*c_fit) with m << L)
- Same binary parameters and sampling rates used for both surrogate and EOB-LAL timings so the ratio is apples-to-apples
- Per-rate timings, speedup factors, and scaling check are recorded in timing_comparison.csv

## Limitations

- The reported speedup depends on the sampling rate: between 2 and almost 4 orders of magnitude, so a single-rate measurement is not representative
- The ~2,300x figure at 2,048 Hz is tied to the m=10 EOBNRv2 surrogate and this specific EOB-LAL baseline; other models or codes will differ
- gwsurrogate modules are C-extensions needing NumPy's C-API headers at build time; NumPy 1.x vs 2.x build choices affect installation
- Timing is hardware-dependent; only the scaling behavior and order-of-magnitude ratio are robust, not absolute wall-clock numbers

## Evidence

- [results] speedup ~2,300 at 2,048 Hz with 2 to nearly 4 orders of magnitude range: [evidence span withheld — anchor status: not_found]
- [abstract] predicted online cost O(mL + m*c_fit): [evidence span withheld — anchor status: not_found]
- [abstract] surrogates are more than 3 orders of magnitude faster than standard EOB waveform generation: "We find that these surrogates are more than 3 orders of magnitude faster to evaluate as compared to the cost of generating effective-one-body waveforms in standard ways."
- [abstract] baseline defined as EOB solver in LAL: "generating a fiducial waveform using the EOB solver as implemented in the LIGO Analysis Library (LAL) software [54], which we refer to as the EOB-LAL code."
- [readme] download-then-load usage pattern in gwsurrogate: "gwsurrogate.catalog.pull('NRSur7dq4')       # This can take a few minutes"
