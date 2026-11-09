---
name: surrogate-online-evaluation-benchmarking
description: Use when a reduced-basis/empirical-interpolant surrogate has been built
  offline (m basis waveforms, m empirical time nodes, c_fit spline coefficient fits)
  and you must evaluate it at generic parameter values and quantify fidelity-vs-speedup
  against the fiducial solver.
when_to_use_negative:
- No trained surrogate exists yet — the greedy reduced basis, empirical interpolant,
  and c_fit fits must be built offline first (see the 4-step construction loop skill)
- The requested parameter point lies outside the training interval of the surrogate
  (e.g., q beyond the surrogate's fitted range); extrapolation is not supported by
  the empirical-interpolant construction
- Validation requires reproducing waveform physics beyond the fiducial solver's fidelity
  (e.g., precessing or memory effects) — use the corresponding NR/CCE-trained models
  like NRSur7dq4 or NRHybSur3dq8_CCE instead of the EOB surrogates here
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: GWSurrogate
  role: Loads and evaluates the surrogate model online for waveform prediction at
    generic parameter choices
  repo: https://github.com/sxs-collaboration/gwsurrogate
- name: gwtools
  role: Computes mismatch/errors between surrogate and directly generated EOB-LAL
    waveforms
  repo: https://bitbucket.org/chadgalley/gwtools/
- name: NumPy
  role: Holds waveform arrays and performs the O(mL + m*c_fit) online reconstruction
    arithmetic
- name: LIGO Analysis Library (LAL)
  role: Generates the reference EOB-LAL fiducial waveforms used as the benchmark for
    accuracy and timing
- name: pytest
  role: Runs the regression harness (test_model_regression.py) to verify surrogate
    evaluation reproduces expected results
- name: PyCBC
  role: Alternative interface to evaluate the gwsurrogate model through PyCBC's waveform
    API
provenance:
  source_task_ids:
  - task_001
  - task_008
  - task_007
  source_papers:
  - doi: 10.1103/physrevx.4.031006
    title: Fast Prediction and Evaluation of Gravitational Waveforms Using Surrogate
      Models
  source_content_hash: sha256:c77eb4601325133c64e98c0174a95c7b50d2810a0dad8a5eb32a1bf1744fc134
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sxs-collaboration/gwsurrogate
  tools:
  - GWSurrogate
  - LAL
  - LIGO Analysis Library (LAL)
  - NumPy
  - PyCBC
  - Python
  - gwtools
  - pytest
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# surrogate-online-evaluation-benchmarking

## Summary

Evaluate a reduced-basis surrogate gravitational waveform online at generic parameter choices and benchmark its speedup and accuracy (L2/L∞ errors, evaluation time) against direct fiducial EOB-LAL waveform generation, targeting >3 orders of magnitude faster evaluation.

## When to use

Use when a reduced-basis/empirical-interpolant surrogate has been built offline (m basis waveforms, m empirical time nodes, c_fit spline coefficient fits) and you must evaluate it at generic parameter values and quantify fidelity-vs-speedup against the fiducial solver.

## When NOT to use

- No trained surrogate exists yet — the greedy reduced basis, empirical interpolant, and c_fit fits must be built offline first (see the 4-step construction loop skill)
- The requested parameter point lies outside the training interval of the surrogate (e.g., q beyond the surrogate's fitted range); extrapolation is not supported by the empirical-interpolant construction
- Validation requires reproducing waveform physics beyond the fiducial solver's fidelity (e.g., precessing or memory effects) — use the corresponding NR/CCE-trained models like NRSur7dq4 or NRHybSur3dq8_CCE instead of the EOB surrogates here

## Inputs

- Trained surrogate model: m reduced basis waveforms, m empirical time nodes, and c_fit coefficient spline fits (optionally saved as HDF5 via gwsurrogate.new.saveH5Object)
- Held-out parameter values within the training range (e.g., q values in [3,4] or [1,10])
- Reference EOB-LAL waveforms generated directly at the validation parameter points (via the LIGO Analysis Library EOB solver)

## Outputs

- Surrogate-predicted waveform time series at each validation parameter choice
- L2 and L∞ error metrics against directly generated EOB-LAL waveforms
- Surrogate evaluation time, direct EOB-LAL generation time, and computed speedup factor (e.g., 1×10⁻⁴ s evaluation, speedup 1800 for the q∈[1,4] case)
- Structured metrics CSV (e.g., eob22_surrogate_metrics.csv or surrogate_validation_metrics.csv) and a passing pytest regression harness (test_model_regression.py)

## How to apply

Reconstruct the waveform at a held-out parameter choice using Eq. (30): combine the fitted coefficients (c_fit spline fits at each of the m empirical times) with the reduced basis, so predicting L time samples costs O(mL + m*c_fit) online operations with typically m << L. Evaluate at held-out points spanning the training interval (e.g., q in [3,4] inside the q in [1,4] surrogate, or the extended Table I cases with q up to 10, durations ~1e5 M, and modes (2,1), (2,2), (3,3), (4,4), (5,5)). Compute L2 and L∞ errors against directly generated EOB-LAL waveforms using gwtools for the comparison. Time the surrogate evaluation and the direct EOB-LAL generation to compute the speedup factor, then log the metrics into a structured record (e.g., eob22_surrogate_metrics.csv or surrogate_validation_metrics.csv) and exercise the module through pytest. Accept the surrogate only if errors are consistent with Table I values (e.g., L2 ~ 2×10⁻⁷, L∞ ~ 2×10⁻³ for the m=25 q∈[1,4] case) and the >3-orders-of-magnitude speedup is retained.

## Related tools

- **GWSurrogate** (Loads and evaluates the surrogate model online for waveform prediction at generic parameter choices) — https://github.com/sxs-collaboration/gwsurrogate
- **gwtools** (Computes mismatch/errors between surrogate and directly generated EOB-LAL waveforms) — https://bitbucket.org/chadgalley/gwtools/
- **NumPy** (Holds waveform arrays and performs the O(mL + m*c_fit) online reconstruction arithmetic)
- **LIGO Analysis Library (LAL)** (Generates the reference EOB-LAL fiducial waveforms used as the benchmark for accuracy and timing)
- **pytest** (Runs the regression harness (test_model_regression.py) to verify surrogate evaluation reproduces expected results)
- **PyCBC** (Alternative interface to evaluate the gwsurrogate model through PyCBC's waveform API)

## Examples

```
sur = gwsurrogate.LoadSurrogate('EOB_surrogate'); t, h, dyn = sur(q=3.5, dt=0.1, f_low=0)
```

## Evaluation signals

- Surrogate evaluation reproduces fiducial EOB-LAL waveforms with errors consistent with Table I (e.g., L2 ~ 2×10⁻⁷, L∞ ~ 2×10⁻³ for the m=25 q∈[1,4] surrogate)
- Online evaluation is >3 orders of magnitude faster than standard EOB-LAL generation (e.g., ~1×10⁻⁴ s vs speedup ~1800)
- Timing scales as O(mL + m*c_fit), confirming m << L (e.g., m=25 basis elements)
- Regression harness passes: generate regression data locally via test_model_regression.py and pytest -v -s runs cleanly
- Validation metrics CSV (surrogate_validation_metrics.csv) is populated with greedy error, surrogate mismatch, and speedup for each tested case

## Limitations

- Surrogate accuracy and speedup are only valid within the parameter interval and duration used for the offline build (e.g., q∈[1,2] or extended q∈[1,4] with m=25 basis elements); performance outside the training domain is not guaranteed
- The fiducial waveforms must be generated with the EOB solver as implemented in LAL (EOB-LAL code); surrogates built on a different fiducial family are not interchangeable
- The online cost formula O(mL + m*c_fit) assumes the m basis elements and coefficient fits are preloaded; large surrogates require catalog pull and load time not counted in the evaluation benchmark

## Evidence

- [abstract] Online cost scaling: [evidence span withheld — anchor status: not_found]
- [abstract] Speedup claim: "We find that these surrogates are more than 3 orders of magnitude faster to evaluate as compared to the cost of generating effective-one-body waveforms in standard ways."
- [abstract] Fiducial solver: "generating a fiducial waveform using the EOB solver as implemented in the LIGO Analysis Library (LAL) software [54], which we refer to as the EOB-LAL code."
- [readme] Install path: "`gwsurrogate` is [on conda-forge](https://anaconda.org/conda-forge/gwsurrogate), and can be installed with"
- [methods] Regression testing: ">>> pytest                               # run all tests"
