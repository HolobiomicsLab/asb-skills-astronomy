---
name: nonspinning-binary-black-hole-waveform-processing
description: Use when the goal is to quantify reduced-basis compressibility of a fiducial
  waveform family — e.g. reproducing Fig. 3 greedy-error decay for 501 EOB-LAL nonspinning
  waveforms with q in [1,2] and checking exponential decay to machine precision.
when_to_use_negative:
- The waveform family is precessing, higher-mode, or NR-trained (e.g. NRSur7dq4, NRHybSur3dq8)
  rather than the nonspinning EOBNRv2 (2,2) fiducial family — the greedy decay thresholds
  differ
- Mass ratios extend beyond q in [1,2]; the 19-RB machine-precision finding is validated
  only on the 501-waveform q∈[1,2] training set
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: LIGO Analysis Library (LAL)
  role: EOB-LAL solver used to generate the EOBNRv2 fiducial training waveforms
- name: gwtools
  role: Waveform utilities for peak-aligning training waveforms on a common time axis
  repo: https://bitbucket.org/chadgalley/gwtools/
- name: NumPy
  role: Norm evaluations used to compute the greedy error at each greedy iteration
- name: GWSurrogate
  role: Publicly available Python package providing the surrogate-building interface
    and dependencies (gwtools) used in this workflow
  repo: https://github.com/sxs-collaboration/gwsurrogate
- name: Python
  role: Implementation language for the greedy algorithm, alignment, and error-decay
    plotting
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1103/physrevx.4.031006
    title: Fast Prediction and Evaluation of Gravitational Waveforms Using Surrogate
      Models
  source_content_hash: sha256:0a99d2fd2b1df0fe87b3aca5a578433f1ca174fec90831eeb857484921d6a13a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GWSurrogate
  - LIGO Analysis Library (LAL)
  - NumPy
  - Python
  - gwtools
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# nonspinning-binary-black-hole-waveform-processing

## Summary

Generates a training set of EOBNRv2 nonspinning binary-black-hole (2,2) waveforms, peak-aligns them, and runs a greedy algorithm to build a reduced basis whose error decay quantifies how compactly the EOB model can be represented for mass ratios q in [1,2].

## When to use

Use when the goal is to quantify reduced-basis compressibility of a fiducial waveform family — e.g. reproducing Fig. 3 greedy-error decay for 501 EOB-LAL nonspinning waveforms with q in [1,2] and checking exponential decay to machine precision.

## When NOT to use

- The waveform family is precessing, higher-mode, or NR-trained (e.g. NRSur7dq4, NRHybSur3dq8) rather than the nonspinning EOBNRv2 (2,2) fiducial family — the greedy decay thresholds differ
- Mass ratios extend beyond q in [1,2]; the 19-RB machine-precision finding is validated only on the 501-waveform q∈[1,2] training set

## Inputs

- EOBNRv2 nonspinning BBH (2,2) waveform training set of 501 waveforms, mass ratios q spanning 1 to 2, generated with EOB-LAL
- Peak-alignment convention (Appendix E) applied via gwtools waveform utilities

## Outputs

- reduced_basis.h5 containing the greedy-selected reduced-basis waveforms and their parameter values
- greedy_error_decay.png plotting greedy error versus number of reduced-basis waveforms
- Verified exponential greedy-error decay (~1e-3 at m=5, machine precision at m=19)

## How to apply

Generate the EOBNRv2 fiducial training set of 501 nonspinning waveforms with mass ratios q spanning 1 to 2 using the EOB solver implemented in the LIGO Analysis Library (EOB-LAL). Peak-align all training waveforms on a common time axis following the Appendix E alignment convention, using gwtools waveform utilities. Run the greedy algorithm over the training set, sequentially selecting reduced-basis parameter values and computing the greedy error at each iteration with NumPy norm evaluations. Monitor greedy error versus number of reduced-basis (RB) waveforms: accept the result only if the error decays exponentially, attains roughly 1e-3 at m=5 RB elements, and reaches machine precision by m=19 RB elements. Save the selected RB waveforms to reduced_basis.h5 and record the decay curve to greedy_error_decay.png.

## Related tools

- **LIGO Analysis Library (LAL)** (EOB-LAL solver used to generate the EOBNRv2 fiducial training waveforms)
- **gwtools** (Waveform utilities for peak-aligning training waveforms on a common time axis) — https://bitbucket.org/chadgalley/gwtools/
- **NumPy** (Norm evaluations used to compute the greedy error at each greedy iteration)
- **GWSurrogate** (Publicly available Python package providing the surrogate-building interface and dependencies (gwtools) used in this workflow) — https://github.com/sxs-collaboration/gwsurrogate
- **Python** (Implementation language for the greedy algorithm, alignment, and error-decay plotting)

## Examples

```
conda create -n myenv python=3.11 "numpy<2.0"; pip install gwsurrogate; # then run greedy over the 501 EOB-LAL training waveforms and export reduced_basis.h5 + greedy_error_decay.png
```

## Evaluation signals

- Greedy error decays exponentially (straight line on semilog plot of error vs number of RB waveforms)
- Greedy error reaches ~1e-3 with about 5 reduced-basis waveforms
- Greedy error reaches machine precision with about 19 reduced-basis waveforms for q in [1,2]
- All 501 training waveforms are peak-aligned on a common time axis before greedy selection
- reduced_basis.h5 stores exactly the m greedily-selected RB waveforms and their parameter values

## Limitations

- Machine-precision representation with 19 RB elements is validated only for the considered mass ratios q in [1,2] on the 501-waveform EOB training set
- Training-set generation is expensive: the fiducial EOB waveforms must be solved with EOB-LAL before greedy selection can run
- gwsurrogate C-extension modules require NumPy C-API headers at build time; NumPy 1.x vs 2.x builds need explicit handling (e.g. conda create -n myenv python=3.11 "numpy<2.0")

## Evidence

- [abstract] EOB-LAL training set generation: "generating a fiducial waveform using the EOB solver as implemented in the LIGO Analysis Library (LAL) software [54], which we refer to as the EOB-LAL code."
- [abstract] greedy algorithm and reduced basis: "First, a set of m parameter values is determined using a greedy algorithm from which a reduced basis representation is constructed."
- [abstract] surrogate speedup rationale: "We find that these surrogates are more than 3 orders of magnitude faster to evaluate as compared to the cost of generating effective-one-body waveforms in standard ways."
- [readme] gwtools dependency: "gwsurrogate requires:

1)  gwtools. If you are installing gwsurrogate with pip you will automatically get gwtools."
- [readme] NumPy C-API build constraint: "Certain gwsurrogate modules are implemented as C-extensions and require NumPy’s C-API headers at build time."
