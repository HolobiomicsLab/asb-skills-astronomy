---
name: wallclock-runtime-profiling
description: Use when comparing training cost of neural differential equation variants
  on irregularly sampled time series (e.g. BIDMC respiratory rate, heart rate, SpO2)
  and you need to substantiate an efficiency claim with measured wall-clock time plus
  test L2 loss over seeds.
when_to_use_negative:
- The task involves neural SDEs solved with the reversible Heun method — that experiment
  uses a different solver/backprop setup, not the log-ODE preprocessing benchmarked
  here
- The time series are short or densely and regularly sampled, so the step-size speed-up
  motivation of the log-ODE method (steps much larger than the sampling rate) does
  not apply
- Only accuracy matters and training-time comparison is out of scope — then standard
  CDE evaluation suffices without wall-clock profiling
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: PyTorch
  role: Implementation framework in which the neural RDE, neural CDE, and ODE-RNN
    models are trained and timed
- name: torchcde
  role: Interpolates the logsignature-augmented driving path for the neural RDE/CDE
    experiments
  repo: https://github.com/patrick-kidger/torchcde
- name: torchdiffeq
  role: Provides the differentiable ODE solvers used to solve the neural CDE (reduced
    to an ODE) and support backpropagation during the timing runs
  repo: https://github.com/rtqichen/torchdiffeq
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:fa8662fe7a95941c264d8308ba21b9c36caa501ec23a429dddf22905ad91bacf
  source_papers:
  - doi: 10.48550/arxiv.2202.02435
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/patrick-kidger/torchcde
  tools:
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
yes_rate: 0.42857142857142855
n_claims: 7
---

# wallclock-runtime-profiling

## Summary

Benchmark neural differential equation models (neural RDE/log-ODE, neural CDE, ODE-RNN) on the BIDMC vital-sign regression tasks by recording test-set L2 loss (mean ± standard deviation over seeds) and wall-clock training time, to validate the ~10x speed-up claim of the neural RDE over CDE-style baselines.

## When to use

Use when comparing training cost of neural differential equation variants on irregularly sampled time series (e.g. BIDMC respiratory rate, heart rate, SpO2) and you need to substantiate an efficiency claim with measured wall-clock time plus test L2 loss over seeds.

## When NOT to use

- The task involves neural SDEs solved with the reversible Heun method — that experiment uses a different solver/backprop setup, not the log-ODE preprocessing benchmarked here
- The time series are short or densely and regularly sampled, so the step-size speed-up motivation of the log-ODE method (steps much larger than the sampling rate) does not apply
- Only accuracy matters and training-time comparison is out of scope — then standard CDE evaluation suffices without wall-clock profiling

## Inputs

- Three BIDMC regression datasets (respiratory rate, heart rate, SpO2) from the TSR archive
- Irregularly sampled time series, preprocessed via log-ODE logsignatures (depth 2-3, window step sizes 8/128/512)
- Train/test splits and a fixed hyperparameter budget shared across models

## Outputs

- Test-set L2 loss (mean ± standard deviation over seeds) for neural RDE, neural CDE, and ODE-RNN on each BIDMC dataset
- Wall-clock training time per model per dataset
- A speed-up ratio (target: ~order of magnitude) for neural RDE vs neural CDE

## How to apply

Preprocess the three BIDMC TSR-archive datasets by normalising each channel to zero mean and unit variance, then compute logsignatures of depth 2-3 over windows defined by step sizes 8/128/512 (log-ODE method) and interpolate them into a continuous driving path with torchcde. Train the neural RDE in PyTorch with backpropagation via discretise-then-optimise, and the baselines — the neural CDE solved by reduction to an ODE and the ODE-RNN — on the same train/test splits with matching hyperparameter budgets. For each model/dataset pair record test-set L2 loss as mean ± standard deviation over seeds and wall-clock training time. Validation is two-sided: the neural RDE must attain test L2 loss within the reported mean ± standard deviation of the baselines while achieving the ~10x wall-clock speed-up reported in Section B.2.3 / Appendix D.6.

## Related tools

- **PyTorch** (Implementation framework in which the neural RDE, neural CDE, and ODE-RNN models are trained and timed)
- **torchcde** (Interpolates the logsignature-augmented driving path for the neural RDE/CDE experiments) — https://github.com/patrick-kidger/torchcde
- **torchdiffeq** (Provides the differentiable ODE solvers used to solve the neural CDE (reduced to an ODE) and support backpropagation during the timing runs) — https://github.com/rtqichen/torchdiffeq

## Examples

```
pip install torchcde torchdiffeq  # then train neural RDE / neural CDE / ODE-RNN on BIDMC splits, timing each run with time.time() around the training loop
```

## Evaluation signals

- Neural RDE test L2 loss falls within the reported mean ± standard deviation of the neural CDE and ODE-RNN baselines on all three BIDMC datasets
- Measured wall-clock training time of the neural RDE is roughly an order of magnitude (~10x) lower than the neural CDE, matching Section B.2.3 / Appendix D.6
- All three models are trained on identical train/test splits with matching hyperparameter budgets, so the timing comparison is apples-to-apples
- Each dataset channel is confirmed normalised to zero mean and unit variance before training
- Loss and timing are aggregated over multiple seeds (mean ± standard deviation), not single runs

## Limitations

- Backpropagation scheme matters for both cost and memory: on long time series, optimise-then-discretise backpropagation was needed simply to avoid running out of memory
- At the time of writing there did not exist a JAX library for computing logsignatures, so the log-ODE preprocessing is tied to the PyTorch (torchcde) stack
- Software version numbers are not stated for any tool, so exact wall-clock timings are not directly reproducible across environments
- torchcde is a research prototype; its author recommends Diffrax for new projects, which may change achievable runtimes

## Evidence

- [results] Reported metrics for the BIDMC benchmark: [evidence span withheld — anchor status: not_found]
- [readme] torchdiffeq README on supported backpropagation: [evidence span withheld — anchor status: not_found]
