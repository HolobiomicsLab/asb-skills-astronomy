---
name: eob-waveform-generation
description: Use when you need fiducial EOB waveforms as training data for reduced-basis/empirical-interpolant
  surrogate building, or as a timing and accuracy baseline against a surrogate model.
when_to_use_negative:
- You only need a fast waveform prediction for downstream analysis — load a pre-built
  surrogate via gwsurrogate.catalog.pull and gwsurrogate.LoadSurrogate instead of
  generating EOB-LAL waveforms directly
- The target binary is precessing or spans parameter regimes outside the nonspinning
  EOB training space described here (e.g. use NRSur7dq4 or SEOBNRv4PHMSur models instead)
- You lack the EOB-LAL code or license; offline surrogate building requires fiducial
  EOB generation that surrogate evaluation was designed to bypass
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: LIGO Analysis Library (LAL)
  role: Provides the EOB solver (EOB-LAL code) used to generate fiducial effective-one-body
    waveforms for training sets and timing baselines
- name: GWSurrogate
  role: Python package used to download and load the resulting surrogate models built
    from the generated EOB training waveforms, and to time surrogate evaluation against
    EOB-LAL generation
  repo: https://github.com/sxs-collaboration/gwsurrogate
- name: gwtools
  role: Waveform utilities used for peak-aligning training waveforms on a common time
    axis (Appendix E alignment convention)
  repo: https://bitbucket.org/chadgalley/gwtools/
- name: NumPy
  role: Numerical backend for norm evaluations, greedy-error computation, and L2/Linf
    error calculations against generated waveforms
provenance:
  source_task_ids:
  - task_002
  - task_007
  - task_006
  source_papers:
  - doi: 10.1103/physrevx.4.031006
    title: Fast Prediction and Evaluation of Gravitational Waveforms Using Surrogate
      Models
  source_content_hash: sha256:84b3fa2b38ed24a5b9d212be2615aac82828f86463f4758a3550bc422c34d769
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
  - pytest
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# eob-waveform-generation

## Summary

Generates fiducial effective-one-body (EOB) gravitational waveforms — e.g. nonspinning BBH EOBNRv2 waveforms across mass ratios q in [1,10] and durations up to 10^5 M — using the EOB solver implemented in the LIGO Analysis Library (EOB-LAL code), serving as the training/validation backbone for reduced-basis surrogate construction.

## When to use

Use when you need fiducial EOB waveforms as training data for reduced-basis/empirical-interpolant surrogate building, or as a timing and accuracy baseline against a surrogate model.

## When NOT to use

- You only need a fast waveform prediction for downstream analysis — load a pre-built surrogate via gwsurrogate.catalog.pull and gwsurrogate.LoadSurrogate instead of generating EOB-LAL waveforms directly
- The target binary is precessing or spans parameter regimes outside the nonspinning EOB training space described here (e.g. use NRSur7dq4 or SEOBNRv4PHMSur models instead)
- You lack the EOB-LAL code or license; offline surrogate building requires fiducial EOB generation that surrogate evaluation was designed to bypass

## Inputs

- Binary parameters: mass ratio q (e.g. [1,2] or [1,10]), nonspinning (zero spins)
- Waveform duration in total mass M (up to 10^5 M)
- Sampling rate / timestep dt (e.g. 2,048 Hz)
- Spherical-harmonic mode list ((2,2), (2,1), (3,3), (4,4), (5,5))

## Outputs

- Training set of fiducial time-domain EOB waveforms (e.g. 501 waveforms for q in [1,2])
- Peak-aligned waveform time series on a common time axis
- EOB-LAL timing baselines per sampling rate for speedup comparison
- Validation waveforms at generic parameter points for computing surrogate L2/Linf errors

## How to apply

Use the EOB solver as implemented in the LIGO Analysis Library (the EOB-LAL code) to generate time-domain waveforms for a training set spanning the target parameter space — e.g. 501 nonspinning waveforms with q in [1,2] for the (2,2) greedy-error study, or q up to 10 with durations up to 10^5 M for the Table I surrogate suite. Peak-align all training waveforms on a common time axis (Appendix E alignment convention) so that reduced-basis and empirical-interpolant constructions operate on consistently aligned series. The same generation path provides the timing baseline for speedup measurements: time EOB-LAL generation at each sampling rate (e.g. up to 2,048 Hz) and compare against surrogate evaluation cost O(mL + m*c_fit). Validate generated waveforms by checking they support exponential greedy-error decay (~1e-3 at m=5 RB elements, machine precision at m=19 for q in [1,2]) and that surrogate L2/Linf errors against fiducial EOB-LAL waveforms are consistent with reported values.

## Related tools

- **LIGO Analysis Library (LAL)** (Provides the EOB solver (EOB-LAL code) used to generate fiducial effective-one-body waveforms for training sets and timing baselines)
- **GWSurrogate** (Python package used to download and load the resulting surrogate models built from the generated EOB training waveforms, and to time surrogate evaluation against EOB-LAL generation) — https://github.com/sxs-collaboration/gwsurrogate
- **gwtools** (Waveform utilities used for peak-aligning training waveforms on a common time axis (Appendix E alignment convention)) — https://bitbucket.org/chadgalley/gwtools/
- **NumPy** (Numerical backend for norm evaluations, greedy-error computation, and L2/Linf error calculations against generated waveforms)

## Examples

```
sur = gwsurrogate.LoadSurrogate('EOBNRv2'); t, h, dyn = sur(q=2, dt=dt, f_low=f_low)
```

## Evaluation signals

- Training set of 501 nonspinning EOB waveforms with q in [1,2] supports a greedy error that decays exponentially, reaching ~1e-3 with about 5 reduced-basis waveforms and machine precision with about 19
- Surrogate L2/Linf errors at validation parameter points are consistent with the Table I values of the paper
- Surrogate-to-EOB-LAL speedup at 2,048 Hz sampling is approximately 2,300x, with overall speedups between 2 and nearly 4 orders of magnitude
- Surrogate online evaluation cost scales linearly in the number of time samples L, consistent with the O(mL + m*c_fit) prediction
- All training waveforms are peak-aligned on a common time axis per the Appendix E alignment convention

## Limitations

- Surrogates built this way target nonspinning BBHs; the fiducial EOB family described covers mass ratios 1 to 10 and durations up to 10^5 M, not generically precessing systems
- EOB-LAL waveform generation is the slow baseline (surrogates are more than 3 orders of magnitude faster), so it is impractical for parameter estimation or large parameter-space exploration
- This is the offline building phase — surrogate building requires solving differential equations with other codes and should be done once; online evaluation uses the saved surrogate (e.g. reduced_basis.h5 / gwsurrogate H5 objects)

## Evidence

- [abstract] generating a fiducial waveform using the EOB solver as implemented in the LIGO Analysis Library (LAL) software [54], which we refer to as the EOB-LAL code.: "generating a fiducial waveform using the EOB solver as implemented in the LIGO Analysis Library (LAL) software"
- [abstract] We generate accurate surrogate models for effective-one-body waveforms of nonspinning binary black hole coalescences with durations as long as 105M, mass ratios from 1 to 10, and for multiple: [evidence span withheld — anchor status: not_found]
- [readme] Surrogates provide a fast and accurate evaluation mechanism for gravitational waveforms, which would otherwise be found through solving differential equations. These equations must be solved in the ``building" phase, which was performed using other codes.: "These equations must be solved in the ``building" phase, which was performed using other codes"
- [abstract] First, a set of m parameter values is determined using a greedy algorithm from which a reduced basis representation is constructed.: "First, a set of m parameter values is determined using a greedy algorithm from which a reduced basis representation is constructed."
