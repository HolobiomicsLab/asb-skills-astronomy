---
name: empirical-interpolant-node-selection
description: Use when a reduced basis of m greedy parameters has been built for a
  fiducial waveform family and you must induce m time nodes for empirical interpolation
  to construct a surrogate's compact offline representation.
when_to_use_negative:
- No reduced basis exists yet for the waveform family — run greedy parameter selection
  first; EIM nodes are induced by the m greedy parameters, not chosen independently.
- Only a handful of fiducial waveform evaluations are needed — the offline EIM/EIM-solve
  cost is not justified and direct EOB-LAL generation is simpler.
- The empirical nodes are requested for a different fiducial family or extended parameter
  range (e.g., moving from q in [1,2] to q up to 10, longer ~1e5M durations, or additional
  (2,1), (3,3), (4,4), (5,5) modes) — the nodes and B_i(t) must be rebuilt per family
  and per mode.
edam_operation: http://edamontology.org/operation_2426
tools:
- name: GWSurrogate
  role: Public Python package through which surrogates built with this offline-online
    method are stored and evaluated; used to exercise the EIM-based surrogate online
    at generic parameter choices.
  repo: https://github.com/sxs-collaboration/gwsurrogate
- name: NumPy
  role: Implements the greedy EIM node selection loop and solves the EIM linear system
    for the B_i(t) coefficient functions; also computes the 1,000-waveform pointwise
    error statistics.
- name: gwtools
  role: Dependency of gwsurrogate used for reduced-basis/EIM tooling in the offline
    construction workflow.
  repo: https://bitbucket.org/chadgalley/gwtools/
- name: LIGO Analysis Library (LAL)
  role: EOB-LAL code generating the fiducial waveform training set from which the
    reduced basis (and hence EIM nodes) is built.
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_007
  - task_004
  source_papers:
  - doi: 10.1103/physrevx.4.031006
    title: Fast Prediction and Evaluation of Gravitational Waveforms Using Surrogate
      Models
  source_content_hash: sha256:42a184a8917907889a552bdcc85199dcdc538966c5f04a402d0caedac2b2ab74
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# empirical-interpolant-node-selection

## Summary

Greedily selects m empirical time nodes from a reduced-basis waveform family so that the fiducial waveform can be reconstructed at any time and parameter value from only m samples, with a provable interpolation error bound (Eq. 20).

## When to use

Use when a reduced basis of m greedy parameters has been built for a fiducial waveform family and you must induce m time nodes for empirical interpolation to construct a surrogate's compact offline representation.

## When NOT to use

- No reduced basis exists yet for the waveform family — run greedy parameter selection first; EIM nodes are induced by the m greedy parameters, not chosen independently.
- Only a handful of fiducial waveform evaluations are needed — the offline EIM/EIM-solve cost is not justified and direct EOB-LAL generation is simpler.
- The empirical nodes are requested for a different fiducial family or extended parameter range (e.g., moving from q in [1,2] to q up to 10, longer ~1e5M durations, or additional (2,1), (3,3), (4,4), (5,5) modes) — the nodes and B_i(t) must be rebuilt per family and per mode.

## Inputs

- Reduced basis waveforms from the greedy algorithm (e.g., a 19-element basis for the EOB fiducial waveform family sampled on a mass-ratio grid)
- Fiducial waveform family time series (amplitude and phase) generated with an EOB-LAL-style solver on the parameter grid
- Greedy-selected m parameter values

## Outputs

- m empirical time nodes (e.g., the 19 EIM time values for q in [1,2])
- B_i(t) empirical interpolation coefficient arrays from the solved EIM linear system
- Validation diagnostics: maximum pointwise empirical interpolation error over ~1,000 random q, comparison against greedy error, and the Eq. (20) Lebesgue-constant error bound (CSV such as eim_validation.csv)

## How to apply

Load the m-element reduced basis built by greedy parameter selection (e.g., the 19-element basis for the EOB fiducial family, q in [1,2] or up to q=10). Run the empirical interpolation method: iteratively select each empirical time node by greedily choosing the time sample where the reduced-basis residual is maximal, until m nodes are fixed. At each step, solve the EIM linear system to precompute the B_i(t) interpolation coefficient functions. Validate by drawing ~1,000 mass ratios q uniformly at random over the training interval, evaluating the empirical interpolant built from B_i(t) and the parameter fits, and checking that the maximum pointwise error |empirical interpolant - exact reduced-basis waveform| is close to the greedy error and that the Lebesgue-constant-based bound from Eq. (20) guarantees error below 1e-8. Write the selected time nodes, B_i(t) arrays, sampled q values, and error/bound diagnostics (e.g., eim_validation.csv) for downstream fitting and online evaluation, which costs O(mL + m*c_fit) operations with m << L.

## Related tools

- **GWSurrogate** (Public Python package through which surrogates built with this offline-online method are stored and evaluated; used to exercise the EIM-based surrogate online at generic parameter choices.) — https://github.com/sxs-collaboration/gwsurrogate
- **NumPy** (Implements the greedy EIM node selection loop and solves the EIM linear system for the B_i(t) coefficient functions; also computes the 1,000-waveform pointwise error statistics.)
- **gwtools** (Dependency of gwsurrogate used for reduced-basis/EIM tooling in the offline construction workflow.) — https://bitbucket.org/chadgalley/gwtools/
- **LIGO Analysis Library (LAL)** (EOB-LAL code generating the fiducial waveform training set from which the reduced basis (and hence EIM nodes) is built.)

## Evaluation signals

- Maximum empirical interpolation error over ~1,000 randomly drawn q in [1,2] waveforms is 'remarkably close to the greedy error' (reduced-basis error level).
- The Eq. (20) Lebesgue-constant-based a posteriori bound guarantees interpolation error better than 1e-8.
- Number of selected nodes equals the reduced-basis dimension m (e.g., exactly 19 nodes for the q in [1,2] EOB example).
- B_i(t) coefficient functions exactly satisfy the EIM linear system at the selected nodes (interpolation reproduces the basis values at the nodes).
- Downstream surrogate evaluation from the m nodes reproduces fiducial EOB waveforms accurately and is more than 3 orders of magnitude faster than standard EOB-LAL generation; pytest regression harness passes.

## Limitations

- The guarantee relies on the greedy/reduced-basis error: the empirical interpolant error can only track the greedy error, so a poorly converged basis yields poorly converged nodes.
- The Eq. (20) bound involves the Lebesgue constant of the selected nodes; if nodes are ill-conditioned, the a posteriori bound may be loose even when actual errors are small.
- Nodes are specific to the fiducial family, parameter range, duration (up to 1e5M), and harmonic mode — surrogates for q up to 10 or higher modes require rebuilding the basis and nodes.

## Evidence

- [abstract] induce EIM node selection from greedy parameters: "Second, these m parameters induce the selection of m time values for interpolating a waveform time series using an empirical interpolant that is built for the fiducial waveform family."
- [abstract] greedy reduced basis precedes EIM: "First, a set of m parameter values is determined using a greedy algorithm from which a reduced basis representation is constructed."
- [readme] surrogates replace differential-equation solves with fast evaluation: "Surrogates provide a fast and accurate evaluation mechanism for gravitational waveforms, which would otherwise be found through solving differential equations."
