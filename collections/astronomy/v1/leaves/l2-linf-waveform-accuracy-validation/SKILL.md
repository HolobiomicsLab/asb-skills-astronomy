---
name: l2-linf-waveform-accuracy-validation
description: Use when a rebuilt or extended surrogate (e.g., larger mass ratios, longer
  durations, higher harmonic modes) must be certified accurate and fast before release
  or reuse.
when_to_use_negative:
- The waveform model was not produced by the offline-online greedy-basis/empirical-interpolant/fit
  surrogate pipeline, so there is no compact surrogate prediction cost O(mL + m*c_fit)
  to compare
- No fiducial EOB-LAL (or equivalent) waveforms are available at validation points,
  so L2/Linf errors cannot be computed against a reference
- The goal is interpolating or fitting new waveform data rather than validating an
  existing surrogate's accuracy and evaluation speed
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_3452
tools:
- name: GWSurrogate
  role: Evaluates the surrogate online at validation parameter points and times predictions
  repo: https://github.com/sxs-collaboration/gwsurrogate
- name: LIGO Analysis Library (LAL)
  role: Generates fiducial EOB-LAL waveforms as the accuracy and timing reference
- name: NumPy
  role: Numerical backend for computing L2/Linf waveform errors at validation points
- name: pytest
  role: Runs the surrogate regression harness (pytest -v -s) after generating regression
    data via test_model_regression.py
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1103/physrevx.4.031006
    title: Fast Prediction and Evaluation of Gravitational Waveforms Using Surrogate
      Models
  source_content_hash: sha256:02575bd3b4652f90027108e7d59282d30933785d62cad62b6dc66995a2abcece
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
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# l2-linf-waveform-accuracy-validation

## Summary

Validates a rebuilt gravitational-wave surrogate model by computing L2 and Linf waveform errors at validation parameter points against fiducial EOB-LAL waveforms, and checking evaluation times and speedups against Table I values, accepting the surrogate only if the >3-orders-of-magnitude speedup is retained.

## When to use

Use when a rebuilt or extended surrogate (e.g., larger mass ratios, longer durations, higher harmonic modes) must be certified accurate and fast before release or reuse.

## When NOT to use

- The waveform model was not produced by the offline-online greedy-basis/empirical-interpolant/fit surrogate pipeline, so there is no compact surrogate prediction cost O(mL + m*c_fit) to compare
- No fiducial EOB-LAL (or equivalent) waveforms are available at validation points, so L2/Linf errors cannot be computed against a reference
- The goal is interpolating or fitting new waveform data rather than validating an existing surrogate's accuracy and evaluation speed

## Inputs

- Rebuilt surrogate model (HDF5 file saved via gwsurrogate.new.saveH5Object) covering modes (2,2), (2,1), (3,3), (4,4), (5,5)
- Validation parameter points (mass ratios q in [1,10], durations up to 10^5 M)
- Fiducial EOB-LAL waveforms generated with the LAL EOB solver for error comparison

## Outputs

- L2 and Linf waveform errors at each validation parameter point per mode
- Surrogate evaluation times and speedups vs. EOB-LAL generation
- surrogate_validation_metrics.csv with errors, evaluation times, and speedups
- pytest regression test results from running pytest -v -s

## How to apply

Evaluate the surrogate online at generic parameter choices with GWSurrogate in Python/NumPy, timing the online prediction cost O(mL + m*c_fit) against fiducial EOB-LAL waveform generation. At each validation parameter point, compute the L2 and Linf norms of the difference between surrogate and fiducial waveform for each mode ((2,2), (2,1), (3,3), (4,4), (5,5)). Exercise the regression harness by generating regression data locally via test_model_regression.py and running pytest -v -s. Write the L2/Linf errors, evaluation times, and speedups to surrogate_validation_metrics.csv. Accept the rebuilt surrogate only if its errors, evaluation times, and speedups are consistent with the paper's Table I values and it retains the >3-orders-of-magnitude evaluation speedup over standard EOB-LAL generation.

## Related tools

- **GWSurrogate** (Evaluates the surrogate online at validation parameter points and times predictions) — https://github.com/sxs-collaboration/gwsurrogate
- **LIGO Analysis Library (LAL)** (Generates fiducial EOB-LAL waveforms as the accuracy and timing reference)
- **NumPy** (Numerical backend for computing L2/Linf waveform errors at validation points)
- **pytest** (Runs the surrogate regression harness (pytest -v -s) after generating regression data via test_model_regression.py)

## Examples

```
python test_model_regression.py; pytest -v -s
```

## Evaluation signals

- L2 and Linf waveform errors at validation points are consistent with the paper's Table I values
- Online evaluation retains the >3-orders-of-magnitude speedup over standard EOB-LAL generation
- surrogate_validation_metrics.csv is produced with populated L2/Linf error, evaluation time, and speedup columns
- pytest -v -s regression tests all pass after generating regression data locally via test_model_regression.py
- Online prediction cost scales as O(mL + m*c_fit) with m << L time samples

## Limitations

- Validation is tied to the fiducial family (nonspinning BBHs, q in [1,10], durations up to 10^5 M, EOB-LAL); accuracy claims do not automatically transfer to other waveform families or parameter ranges
- Speedup depends on m << L; for surrogates with large basis sizes or short time series the O(mL + m*c_fit) advantage can shrink
- Acceptance is comparative: a surrogate whose metrics deviate from Table I values must be rejected even if errors look small in absolute terms

## Evidence

- [abstract] abstract speedup finding: "We find that these surrogates are more than 3 orders of magnitude faster to evaluate as compared to the cost of generating effective-one-body waveforms in standard ways."
- [abstract] online prediction cost: [evidence span withheld — anchor status: not_found]
- [methods] pytest usage: ">>> pytest                               # run all tests"
- [intro] surrogate purpose: "Surrogates provide a fast and accurate evaluation mechanism for gravitational
waveforms, which would otherwise be found through solving differential
equations."
