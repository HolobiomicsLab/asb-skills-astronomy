---
name: reduced-basis-error-comparison
description: Use when a greedy reduced basis has been built over a waveform training
  set (e.g., 501 EOBNRv2 waveforms with q in [1,2]) and you must verify that m empirical
  time nodes reconstruct waveforms to within the greedy error and a guaranteed error
  bound.
when_to_use_negative:
- When no reduced basis exists yet — run the greedy reduced-basis construction first;
  error comparison is meaningless without the greedy error reference
- When validating a fully assembled surrogate's online evaluation cost (O(mL + m*c_fit)
  operations) rather than the interpolation error of the fiducial waveform family
- When the waveform family or parameter range differs from the fiducial training set
  (e.g., precessing or q>2), since the greedy error and Eq. (20) bound were calibrated
  on the q∈[1,2] nonspinning EOB model
tools:
- name: GWSurrogate
  role: Public Python package providing the surrogate-model interface and catalog
    used to build and evaluate the reduced-basis/empirical-interpolant surrogate
  repo: https://github.com/sxs-collaboration/gwsurrogate
- name: gwtools
  role: Waveform utilities for peak-aligning training waveforms on a common time axis
    per the Appendix E alignment convention
  repo: https://bitbucket.org/chadgalley/gwtools/
- name: NumPy
  role: Norm evaluations for greedy error computation and pointwise interpolation-error
    arrays over the validation sample
- name: LIGO Analysis Library (LAL)
  role: EOB-LAL solver used to generate the fiducial EOBNRv2 waveform training set
    whose reconstruction error is compared
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1103/physrevx.4.031006
    title: Fast Prediction and Evaluation of Gravitational Waveforms Using Surrogate
      Models
  source_content_hash: sha256:a1f3165599bc259f7edbb00997672e7d8bf1ca72de12978d81addabf9fc5ce95
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
yes_rate: 0.5454545454545454
n_claims: 11
---

# reduced-basis-error-comparison

## Summary

Quantifies how well an empirical-interpolant surrogate reproduces a reduced-basis representation of EOB waveforms by comparing the empirical interpolation error against the greedy error decay, and validates the a posteriori (Lebesgue-constant / Eq. 20) bound on that error.

## When to use

Use when a greedy reduced basis has been built over a waveform training set (e.g., 501 EOBNRv2 waveforms with q in [1,2]) and you must verify that m empirical time nodes reconstruct waveforms to within the greedy error and a guaranteed error bound.

## When NOT to use

- When no reduced basis exists yet — run the greedy reduced-basis construction first; error comparison is meaningless without the greedy error reference
- When validating a fully assembled surrogate's online evaluation cost (O(mL + m*c_fit) operations) rather than the interpolation error of the fiducial waveform family
- When the waveform family or parameter range differs from the fiducial training set (e.g., precessing or q>2), since the greedy error and Eq. (20) bound were calibrated on the q∈[1,2] nonspinning EOB model

## Inputs

- Reduced basis of m (e.g., 19) greedy-selected EOB waveform representations (e.g., stored in reduced_basis.h5)
- Training set of EOBNRv2 fiducial waveforms (501 nonspinning waveforms, q spanning 1 to 2), peak-aligned per the Appendix E convention
- Randomly sampled validation parameters (e.g., 1,000 q values uniform in [1,2])
- Greedy error at each reduced-basis iteration

## Outputs

- Empirical time nodes (19) and B_i(t) interpolation coefficient arrays
- Maximum pointwise empirical interpolation error over the validation parameter sample
- Lebesgue-constant-based a posteriori error bound from Eq. (20)
- Diagnostics file (e.g., eim_validation.csv) and error-decay plot (e.g., greedy_error_decay.png)

## How to apply

After running the greedy algorithm to select the reduced-basis parameters (exponential error decay reaching ~1e-3 at m=5 and machine precision at m=19 for the EOB-LAL q∈[1,2] training set), run the empirical interpolation method (EIM) to greedily select the m=19 empirical time nodes where the reduced-basis residual is maximal. Solve the EIM linear system at those nodes to precompute the B_i(t) interpolation coefficient functions, then evaluate the empirical interpolant on a large validation sample (e.g., 1,000 mass ratios q drawn uniformly at random in [1,2]) and compute the maximum pointwise error |empirical interpolant − exact reduced-basis waveform|. Compare this maximum against the greedy error — they should be remarkably close — and evaluate the Eq. (20) Lebesgue-constant-based a posteriori bound, which must guarantee error better than 1e-8. Persist the nodes, B_i(t) arrays, sampled q values, and error diagnostics (e.g., to eim_validation.csv) so the error decay and bound can be audited.

## Related tools

- **GWSurrogate** (Public Python package providing the surrogate-model interface and catalog used to build and evaluate the reduced-basis/empirical-interpolant surrogate) — https://github.com/sxs-collaboration/gwsurrogate
- **gwtools** (Waveform utilities for peak-aligning training waveforms on a common time axis per the Appendix E alignment convention) — https://bitbucket.org/chadgalley/gwtools/
- **NumPy** (Norm evaluations for greedy error computation and pointwise interpolation-error arrays over the validation sample)
- **LIGO Analysis Library (LAL)** (EOB-LAL solver used to generate the fiducial EOBNRv2 waveform training set whose reconstruction error is compared)

## Examples

```
sur = gwsurrogate.LoadSurrogate('NRSur7dq4'); t, h, dyn = sur(q, chiA, chiB, dt=dt, f_low=f_low)
```

## Evaluation signals

- Empirical interpolation error decays alongside and stays remarkably close to the greedy error as nodes are added
- Greedy error is exponentially decaying, reaching ~1e-3 at m=5 reduced-basis elements and machine precision at m=19
- Lebesgue-constant bound from Eq. (20) guarantees an error better than 1e-8 for the m=19 node EIM system
- Over 1,000 randomly sampled q∈[1,2] waveforms, the maximum pointwise error |empirical interpolant − exact reduced-basis waveform| is consistent with the greedy error level
- Output files (eim_validation.csv, reduced_basis.h5) contain the 19 nodes, B_i(t) arrays, sampled q values, and error/bound diagnostics with consistent shapes

## Limitations

- The guarantee (error < 1e-8) is specific to the fiducial EOB-LAL family, the q∈[1,2] training range, and the 19-element basis; it does not automatically transfer to other parameter ranges or waveform models
- The a posteriori bound is an upper guarantee — actual maximum errors can be smaller, so bound tightness relative to the greedy error should be reported rather than assumed
- Building-phase waveforms must first be generated by solving differential equations (EOB-LAL), so the comparison cannot be run without access to the fiducial waveform family

## Evidence

- [abstract] Greedy selection of m parameter values to construct a reduced basis representation: "First, a set of m parameter values is determined using a greedy algorithm from which a reduced basis representation is constructed."
- [abstract] Empirical interpolation node selection from the reduced basis: "Second, these m parameters induce the selection of m time values for interpolating a waveform time series using an empirical interpolant that is built for the fiducial waveform family."
- [readme] Surrogate accuracy relative to solving the underlying equations: "Surrogates provide a fast and accurate evaluation mechanism for gravitational waveforms, which would otherwise be found through solving differential equations."
- [intro] Surrogate interface used for building and evaluating the model: "GWSurrogate is an easy-to-use interface to gravitational wave surrogate models."
- [abstract] Online evaluation cost context for the fitted surrogate: [evidence span withheld — anchor status: not_found]
