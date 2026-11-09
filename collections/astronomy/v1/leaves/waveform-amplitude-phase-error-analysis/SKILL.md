---
name: waveform-amplitude-phase-error-analysis
description: Use when validating a gwsurrogate waveform model by comparing surrogate
  evaluations against a fiducial/reference family to bound reconstruction error and
  locate the worst-case parameter point.
when_to_use_negative:
- When the surrogate domain differs from the validation range — e.g., precessing-spin
  models like NRSur7dq4 (q≤4) or high-mass-ratio models like BHPTNRSur1dq1e4 (q≤10000)
  — since the sampled q∈[1,2] nonspinning protocol applies to the EOB fiducial family
- When a direct fiducial/reference waveform family is unavailable, since Eq. 31 and
  amplitude/phase comparisons require a reference to diff against
- When L (time samples) or f_low differ from the fiducial family's sampling — the
  16,384-sample, f_low=0 setting is specific to this validation protocol
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: gwsurrogate
  role: Pulls surrogate catalog data, loads the model (LoadSurrogate), and evaluates
    sur(q, chiA, chiB, dt, f_low) to produce time arrays and waveform modes
  repo: https://github.com/sxs-collaboration/gwsurrogate
- name: gwtools
  role: Waveform utilities used to compute pointwise amplitude and phase differences
    against the reference family
  repo: https://bitbucket.org/chadgalley/gwtools/
- name: NumPy
  role: Draws 1,000 mass ratios uniformly at random from [1,2] and supports array-based
    error aggregation
- name: pytest
  role: Package regression tests used to cross-check stored waveform outputs during
    validation
  repo: https://github.com/sxs-collaboration/gwsurrogate
- name: Python
  role: Environment for the validation script (e.g., conda env python=3.11, numpy<2.0)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1103/physrevx.4.031006
    title: Fast Prediction and Evaluation of Gravitational Waveforms Using Surrogate
      Models
  source_content_hash: sha256:0080affc7101970abbbc9752b50478eb8052f21533a9223ad4a7e571043bb618
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sxs-collaboration/gwsurrogate
  tools:
  - NumPy
  - Python
  - gwsurrogate
  - gwtools
  - pytest
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# waveform-amplitude-phase-error-analysis

## Summary

Quantifies surrogate gravitational-waveform reconstruction error (Eq. 31) plus pointwise amplitude and phase deviations against a reference waveform family, using gwsurrogate evaluations over randomly sampled mass ratios with gwtools waveform utilities.

## When to use

Use when validating a gwsurrogate waveform model by comparing surrogate evaluations against a fiducial/reference family to bound reconstruction error and locate the worst-case parameter point.

## When NOT to use

- When the surrogate domain differs from the validation range — e.g., precessing-spin models like NRSur7dq4 (q≤4) or high-mass-ratio models like BHPTNRSur1dq1e4 (q≤10000) — since the sampled q∈[1,2] nonspinning protocol applies to the EOB fiducial family
- When a direct fiducial/reference waveform family is unavailable, since Eq. 31 and amplitude/phase comparisons require a reference to diff against
- When L (time samples) or f_low differ from the fiducial family's sampling — the 16,384-sample, f_low=0 setting is specific to this validation protocol

## Inputs

- Loaded gwsurrogate surrogate model (fitted coefficients + empirical interpolant basis)
- 1,000 mass-ratio values drawn uniformly at random from [1,2] (numpy.random)
- Reference/fiducial waveform family (e.g., EOB-LAL waveforms) for error comparison
- Timestep dt and f_low=0 evaluation settings with L=16,384 time samples

## Outputs

- Per-point surrogate reconstruction error (Eq. 31) values
- Pointwise amplitude and phase error arrays per sampled mass ratio
- surrogate_validation_results.csv with max surrogate error, max amplitude/phase error, and q of largest error
- Worst-case mass ratio (expected q=1.068) and max-error magnitude (expected < 1e-7)

## How to apply

Pull the surrogate data with gwsurrogate.catalog.pull and load it via gwsurrogate.LoadSurrogate so fitted coefficients and the empirical interpolant basis are in memory. Draw 1,000 mass ratios uniformly at random from [1,2] with numpy.random, keeping spins at zero for the nonspinning fiducial family, and evaluate each via sur(q, chiA, chiB, dt, f_low) with L=16,384 time samples and f_low=0 to return the full surrogate in units of total mass M. For each parameter point, compute the surrogate reconstruction error (Eq. 31) against the reference family and the pointwise amplitude and phase differences using gwtools waveform utilities. Aggregate max surrogate error, max pointwise amplitude/phase error, and the q of the largest error into surrogate_validation_results.csv. Pass criteria: surrogate error uniformly below 1e-7, pointwise amplitude/phase errors at or below ~1e-3, and largest error at q=1.068 (the expected worst case for the EOB test family).

## Related tools

- **gwsurrogate** (Pulls surrogate catalog data, loads the model (LoadSurrogate), and evaluates sur(q, chiA, chiB, dt, f_low) to produce time arrays and waveform modes) — https://github.com/sxs-collaboration/gwsurrogate
- **gwtools** (Waveform utilities used to compute pointwise amplitude and phase differences against the reference family) — https://bitbucket.org/chadgalley/gwtools/
- **NumPy** (Draws 1,000 mass ratios uniformly at random from [1,2] and supports array-based error aggregation)
- **pytest** (Package regression tests used to cross-check stored waveform outputs during validation) — https://github.com/sxs-collaboration/gwsurrogate
- **Python** (Environment for the validation script (e.g., conda env python=3.11, numpy<2.0))

## Examples

```
gwsurrogate.catalog.pull('NRSur7dq4'); sur = gwsurrogate.LoadSurrogate('NRSur7dq4'); t, h, dyn = sur(q, chiA, chiB, dt=dt, f_low=0)
```

## Evaluation signals

- Max surrogate error (Eq. 31) over all 1,000 sampled mass ratios must fall below 1e-7
- Max pointwise amplitude and phase errors must be at or below ~1e-3
- Largest surrogate error must occur at q = 1.068 for the EOB test case
- surrogate_validation_results.csv is produced with the max-error and worst-q fields populated
- Stored waveform outputs agree with the gwsurrogate package's pytest regression tests

## Limitations

- Random sampling of q means the reported worst-case q=1.068 is a Monte Carlo estimate; a different random draw may locate a nearby but not identical peak
- The protocol holds spins at zero and covers q∈[1,2] only, matching the nonspinning fiducial EOB family validated in the paper, not precessing or high-mass-ratio models
- The paper's own validation covers mass ratios 1–10; extending beyond the fiducial family's trained domain invalidates the error bounds
- gwsurrogate modules are C-extensions requiring NumPy C-API headers at build time; NumPy 1.x vs 2.x builds can complicate environment setup

## Evidence

- [methods] surrogate evaluation protocol: "t, h, dyn = sur(q, chiA, chiB, dt=dt, f_low=f_low, precessing_opts=precessing_opts)"
- [readme] model loading workflow: "sur = gwsurrogate.LoadSurrogate('NRSur7dq4')"
- [readme] data download step: "gwsurrogate.catalog.pull('NRSur7dq4')       # This can take a few minutes"
- [readme] test execution: ">>> pytest                               # run all tests"
- [abstract] surrogate accuracy claims: "We find that these surrogates are more than 3 orders of magnitude faster to evaluate as compared to the cost of generating effective-one-body waveforms in standard ways."
- [full_text] validation workflow source card: [evidence span withheld — anchor status: not_found]
