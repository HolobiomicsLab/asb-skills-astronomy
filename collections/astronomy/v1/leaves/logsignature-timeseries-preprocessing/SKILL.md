---
name: logsignature-timeseries-preprocessing
description: Use when an irregularly sampled, long time series must drive a neural
  CDE/RDE whose integration step size is far larger than the data sampling rate, and
  naive interpolation would discard sub-step dynamics (the log-ODE method of Appendix
  B).
when_to_use_negative:
- The series is short or densely sampled such that a standard neural CDE on direct
  interpolation already trains fast — the log-ODE speed-up targets long series with
  sampling rates far finer than the desired integration step.
- Natural cubic splines are preferred for measurability reasons in a specific task
  — the article notes neural CDEs using natural cubic splines produce slightly worse
  results, so they should not replace logsignature preprocessing here.
- The target framework is JAX and logsignatures must be computed in-pipeline — at
  time of writing there does not exist a JAX library for computing logsignatures.
edam_operation: http://edamontology.org/operation_2945
tools:
- name: torchcde
  role: Differentiable GPU-capable CDE solvers and path interpolation used to build
    the driving path from logsignatures and train the neural RDE/CDE
  repo: github.com/patrick-kidger/torchcde
- name: torchdiffeq
  role: PyTorch differentiable ODE solvers with adjoint backpropagation, used for
    solving the neural CDE reduced to an ODE
  repo: github.com/rtqichen/torchdiffeq
- name: PyTorch
  role: Framework in which the neural RDE is implemented and trained
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:657b8b95d7a9a532943c6eb9b9f26f6884e9898b2fb5b1ece4fa3de8012acf43
  source_papers:
  - doi: 10.48550/arxiv.2202.02435
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - PyTorch
  - torchcde
  - torchdiffeq
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# logsignature-timeseries-preprocessing

## Summary

Preprocess irregularly sampled, long time series (e.g. the BIDMC respiratory-rate, heart-rate and SpO2 regression tasks from the TSR archive) into logsignatures over windows, so a neural CDE/RDE can take integration steps (8/128/512) much larger than the sampling rate while retaining sub-step information.

## When to use

Use when an irregularly sampled, long time series must drive a neural CDE/RDE whose integration step size is far larger than the data sampling rate, and naive interpolation would discard sub-step dynamics (the log-ODE method of Appendix B).

## When NOT to use

- The series is short or densely sampled such that a standard neural CDE on direct interpolation already trains fast — the log-ODE speed-up targets long series with sampling rates far finer than the desired integration step.
- Natural cubic splines are preferred for measurability reasons in a specific task — the article notes neural CDEs using natural cubic splines produce slightly worse results, so they should not replace logsignature preprocessing here.
- The target framework is JAX and logsignatures must be computed in-pipeline — at time of writing there does not exist a JAX library for computing logsignatures.

## Inputs

- Irregularly sampled multivariate time series from the TSR archive (BIDMC respiratory rate, heart rate, SpO2 regression datasets)
- Window step size (one of 8, 128, 512) and logsignature depth (2 or 3)

## Outputs

- Logsignature-augmented continuous driving path (piecewise-interpolated from window logsignatures) suitable for torchcde CDE/RDE solvers
- Normalised channel tensors (zero mean, unit variance) per dataset

## How to apply

First normalise each dataset and each channel to zero mean and unit variance. Then compute logsignatures of depth 2–3 over windows defined by the chosen step sizes (8/128/512 on the BIDMC tasks), per the log-ODE method of Appendix B, and interpolate the resulting logsignatures into a continuous driving path (using torchcde for path interpolation in PyTorch). Train the neural RDE with backpropagation via discretise-then-optimise; train the neural CDE by reducing it to an ODE as the comparison. Success is judged by test-set L2 loss (mean ± standard deviation over seeds) within the neural CDE/ODE-RNN baseline range while achieving the ~10x wall-clock training speed-up reported in Section B.2.3 / Appendix D.6.

## Related tools

- **torchcde** (Differentiable GPU-capable CDE solvers and path interpolation used to build the driving path from logsignatures and train the neural RDE/CDE) — github.com/patrick-kidger/torchcde
- **torchdiffeq** (PyTorch differentiable ODE solvers with adjoint backpropagation, used for solving the neural CDE reduced to an ODE) — github.com/rtqichen/torchdiffeq
- **PyTorch** (Framework in which the neural RDE is implemented and trained)

## Evaluation signals

- Every channel of every dataset is normalised to exactly zero mean and unit variance before preprocessing.
- Logsignatures are computed to depth 2–3 over windows matching one of the step sizes 8/128/512, then interpolated into a continuous driving path.
- Test-set L2 loss (mean ± standard deviation over seeds) falls within the neural CDE and ODE-RNN baseline ranges on all three BIDMC datasets.
- Wall-clock training time for the neural RDE is roughly an order of magnitude (~10x) lower than the neural CDE baseline.
- Backpropagation uses discretise-then-optimise as specified in Section 5.

## Limitations

- At time of writing there does not exist a JAX library for computing logsignatures, blocking the same preprocessing in JAX/Diffrax pipelines.
- torchcde was a research prototype — its README recommends Diffrax for new projects as 'much faster, and production-quality'.
- On very long time series, optimise-then-discretise backpropagation may be required simply to avoid running out of memory, which changes the default discretise-then-optimise setup.

## Evidence

- [readme] torchcde deprecation: [evidence span withheld — anchor status: not_found]
