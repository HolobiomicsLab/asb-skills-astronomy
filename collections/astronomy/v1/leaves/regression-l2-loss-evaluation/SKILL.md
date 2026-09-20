---
name: regression-l2-loss-evaluation
description: Use when comparing neural differential equation models on irregularly
  sampled time-series regression tasks (e.g. BIDMC respiratory rate, heart rate, SpO2)
  and you must report test-set L2 loss (mean ± std over seeds) plus training time.
when_to_use_negative:
- The task is classification or probabilistic/generative evaluation rather than regression
  — L2 loss and this mean ± std protocol are specific to regression error.
- There is no held-out test split or no multiple seeds available, so the mean ± standard
  deviation over seeds cannot be computed as the protocol requires.
- The datasets are not normalised (zero mean, unit variance per channel), making L2
  losses incomparable across channels and models.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_2269
tools:
- name: PyTorch
  role: Framework in which the neural RDE, neural CDE, and ODE-RNN models are implemented
    and trained for the L2-loss comparison
- name: torchcde
  role: Provides differentiable GPU-capable CDE solvers and path interpolation used
    for the driving path before test L2 loss is computed
  repo: https://github.com/patrick-kidger/torchcde
- name: torchdiffeq
  role: PyTorch ODE solvers (adjoint backpropagation) used for the neural CDE and
    ODE-RNN baselines evaluated on the same splits
  repo: https://github.com/rtqichen/torchdiffeq
- name: Diffrax
  role: JAX-based alternative library where the corresponding experiments are implemented
    as examples
  repo: https://github.com/patrick-kidger/diffrax
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:d0c0851d9065b9a590583ca29942bada006e13dff0cc3b8b5ec8f611b99833bf
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
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# regression-l2-loss-evaluation

## Summary

Evaluate neural time-series regression models (neural RDE, neural CDE, ODE-RNN) on the three BIDMC vital-sign regression datasets by recording test-set L2 loss as mean ± standard deviation over seeds, alongside wall-clock training time, per the protocol of Kidger's 'On Neural Differential Equations'.

## When to use

Use when comparing neural differential equation models on irregularly sampled time-series regression tasks (e.g. BIDMC respiratory rate, heart rate, SpO2) and you must report test-set L2 loss (mean ± std over seeds) plus training time.

## When NOT to use

- The task is classification or probabilistic/generative evaluation rather than regression — L2 loss and this mean ± std protocol are specific to regression error.
- There is no held-out test split or no multiple seeds available, so the mean ± standard deviation over seeds cannot be computed as the protocol requires.
- The datasets are not normalised (zero mean, unit variance per channel), making L2 losses incomparable across channels and models.

## Inputs

- BIDMC regression datasets (respiratory rate, heart rate, SpO2) from the TSR archive
- Per-channel normalised time series (zero mean, unit variance)
- Test-set predictions and targets from each trained model (neural RDE, neural CDE, ODE-RNN)
- Multiple training runs (seeds) per model per dataset

## Outputs

- Test-set L2 loss reported as mean ± standard deviation over seeds for each model on each of the three BIDMC datasets
- Wall-clock training time for each model, supporting the ~10x speed-up comparison

## How to apply

Download the three BIDMC regression datasets (respiratory rate, heart rate, SpO2) from the TSR archive and normalise each channel to zero mean and unit variance. Train the neural RDE (log-ODE preprocessing: logsignatures of depth 2-3 over windows with step sizes 8/128/512, path interpolated with torchcde) and the baselines — neural CDE solved by reduction to an ODE and ODE-RNN — on identical train/test splits with matching hyperparameter budgets, backpropagating via discretise-then-optimise. On the held-out test set, compute the L2 loss for every seed and report mean ± standard deviation per model per dataset, together with wall-clock training time. Validation criterion: the neural RDE must attain test L2 loss within the baselines' reported mean ± standard deviation while achieving the ~10x wall-clock speed-up (Section B.2.3 / Appendix D.6).

## Related tools

- **PyTorch** (Framework in which the neural RDE, neural CDE, and ODE-RNN models are implemented and trained for the L2-loss comparison)
- **torchcde** (Provides differentiable GPU-capable CDE solvers and path interpolation used for the driving path before test L2 loss is computed) — https://github.com/patrick-kidger/torchcde
- **torchdiffeq** (PyTorch ODE solvers (adjoint backpropagation) used for the neural CDE and ODE-RNN baselines evaluated on the same splits) — https://github.com/rtqichen/torchdiffeq
- **Diffrax** (JAX-based alternative library where the corresponding experiments are implemented as examples) — https://github.com/patrick-kidger/diffrax

## Examples

```
import torch; l2 = ((y_pred - y_true) ** 2).sum(-1)  # per-seed test L2 loss; report l2.mean(), l2.std() as 'mean ± standard deviation of test set L2 loss'
```

## Evaluation signals

- Reported result is test-set L2 loss as mean ± standard deviation over seeds, per model per BIDMC dataset (as in the paper's results table).
- The neural RDE's test L2 loss falls within the reported mean ± standard deviation of the neural CDE and ODE-RNN baselines on the same train/test splits.
- Wall-clock training time for the neural RDE is roughly an order of magnitude (~10x) less than the baselines.
- Each dataset/channel was normalised to zero mean and unit variance before training, so L2 losses are comparable.
- All models used matching hyperparameter budgets and identical train/test splits; the neural CDE was solved by reduction to an ODE with discretise-then-optimise backpropagation.

## Limitations

- Software version numbers are not stated for any tool, so exact reproducibility of the loss numbers may vary across library versions.
- On long time series, optimise-then-discretise backpropagation was needed simply to avoid running out of memory, which may affect training-time measurements.
- Validation is stated as loss within the baselines' mean ± std rather than a strict improvement — the neural RDE need not beat every baseline in loss.

## Evidence

- [results] BIDMC results table protocol: [evidence span withheld — anchor status: not_found]
- [readme] torchcde capability supporting the modelling pipeline: [evidence span withheld — anchor status: not_found]
