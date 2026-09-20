---
name: neural-rde-model-training
description: Use when irregularly sampled, long time series (e.g. BIDMC respiratory
  rate, heart rate, SpO2) must be modelled by a differential-equation-based regressor
  while taking integration steps much larger than the data sampling rate without discarding
  sub-step information.
when_to_use_negative:
- Data are short or densely and regularly sampled, so large integration steps are
  unnecessary and a standard neural CDE suffices
- The task requires an interpolation scheme with better measurability properties,
  e.g. rectilinear interpolation, rather than logsignature preprocessing
- 'Working in JAX: at the time of writing there does not exist a JAX library for computing
  logsignatures, so the PyTorch/torchcde route does not apply'
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: torchcde
  role: Interpolates the logsignature-preprocessed driving path and provides differentiable
    GPU-capable CDE solvers in PyTorch
  repo: https://github.com/patrick-kidger/torchcde
- name: torchdiffeq
  role: Solves the neural CDE baseline by reducing it to an ODE with differentiable
    PyTorch ODE solvers
  repo: https://github.com/rtqichen/torchdiffeq
- name: PyTorch
  role: Framework in which the neural RDE and baselines are implemented and trained
- name: Diffrax
  role: JAX-based alternative solver library referenced by the thesis; recommended
    in the torchcde README for new projects
  repo: https://github.com/patrick-kidger/diffrax
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:9f3030618685aef3b768d703ebcb4c26bfcfe152ad2aa0aace3747ff833b3693
  source_papers:
  - doi: 10.48550/arxiv.2202.02435
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/patrick-kidger/torchcde
  tools:
  - Diffrax
  - PyTorch
  - torchcde
  - torchdiffeq
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# neural-rde-model-training

## Summary

Trains a neural RDE driven by a log-ODE preprocessed path (logsignatures of depth 2-3 over large integration steps) and benchmarks it against neural CDE and ODE-RNN baselines on the three BIDMC vital-sign regression datasets, targeting the reported ~10x wall-clock training speed-up.

## When to use

Use when irregularly sampled, long time series (e.g. BIDMC respiratory rate, heart rate, SpO2) must be modelled by a differential-equation-based regressor while taking integration steps much larger than the data sampling rate without discarding sub-step information.

## When NOT to use

- Data are short or densely and regularly sampled, so large integration steps are unnecessary and a standard neural CDE suffices
- The task requires an interpolation scheme with better measurability properties, e.g. rectilinear interpolation, rather than logsignature preprocessing
- Working in JAX: at the time of writing there does not exist a JAX library for computing logsignatures, so the PyTorch/torchcde route does not apply

## Inputs

- BIDMC regression datasets (respiratory rate, heart rate, SpO2) from the TSR archive
- Irregularly sampled time series with per-channel values needing zero-mean/unit-variance normalisation
- Logsignature windows (depth 2-3) over step sizes 8/128/512
- Shared train/test splits and hyperparameter budget across models

## Outputs

- Test-set L2 loss (mean ± standard deviation over seeds) for neural RDE, neural CDE, and ODE-RNN
- Wall-clock training times per model per dataset
- Trained PyTorch neural RDE model with torchcde-interpolated driving path

## How to apply

Download the three BIDMC regression datasets (respiratory rate, heart rate, SpO2) from the TSR archive and normalise each channel to zero mean and unit variance. Preprocess each irregularly sampled series via the log-ODE method of Appendix B: compute logsignatures of depth 2-3 over windows defined by step sizes 8/128/512, then interpolate them into a continuous driving path. Implement the neural RDE in PyTorch with torchcde for path interpolation, backpropagating via discretise-then-optimise. Train the neural CDE (solved by reducing it to an ODE) and the ODE-RNN baselines on the same train/test splits under a matching hyperparameter budget. Record test-set L2 loss (mean ± standard deviation over seeds) and wall-clock training time for all three models on all three datasets.

## Related tools

- **torchcde** (Interpolates the logsignature-preprocessed driving path and provides differentiable GPU-capable CDE solvers in PyTorch) — https://github.com/patrick-kidger/torchcde
- **torchdiffeq** (Solves the neural CDE baseline by reducing it to an ODE with differentiable PyTorch ODE solvers) — https://github.com/rtqichen/torchdiffeq
- **PyTorch** (Framework in which the neural RDE and baselines are implemented and trained)
- **Diffrax** (JAX-based alternative solver library referenced by the thesis; recommended in the torchcde README for new projects) — https://github.com/patrick-kidger/diffrax

## Examples

```
import torchcde; coeffs = torchcde.natural_cubic_coeffs(logsignature_windows); driving_path = torchcde.CubicSpline(coeffs)
```

## Evaluation signals

- Neural RDE attains test L2 loss within the reported mean ± standard deviation of the neural CDE and ODE-RNN baselines on each BIDMC dataset
- Neural RDE achieves roughly an order of magnitude (~10x) less wall-clock training time than the baselines
- Every channel of every dataset has zero mean and unit variance before preprocessing
- Logsignature depth is 2-3 and windows use step sizes 8/128/512
- Backpropagation runs via discretise-then-optimise without running out of memory on the long BIDMC series

## Limitations

- On long time series, optimise-then-discretise backpropagation was needed simply to avoid running out of memory
- At the time of writing there does not exist a JAX library for computing logsignatures, limiting portability of the log-ODE preprocessing to JAX/Diffrax
- Natural cubic splines produce slightly worse results than other interpolation schemes for neural CDEs, so spline choice must be handled carefully
- The torchcde README recommends Diffrax instead for new projects, as torchcde was its research prototype

## Evidence

- [readme] torchcde role: [evidence span withheld — anchor status: not_found]
