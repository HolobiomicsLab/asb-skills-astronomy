---
name: predictive-accuracy-evaluation
description: Use when comparing a learned GP covariance structure against a fixed-kernel
  GP baseline on time series (e.g. synthetic series with known ground-truth kernel
  labels), and you need both point-prediction accuracy (RMSE) and uncertainty-calibration
  (interval coverage) metrics.
when_to_use_negative:
- The covariance structure is already known and fixed — gallifrey exists precisely
  for cases where the covariance function cannot be specified explicitly, so a learned-structure
  benchmark adds cost without information.
- No held-out data or ground-truth kernel labels exist, so RMSE and coverage against
  known structure cannot be computed and the paper's validation criterion cannot be
  evaluated.
- 'Datasets are large: the paper flags scalability to larger datasets (e.g. PLATO-scale)
  as future algorithmic work, so extrapolating the benchmark result to very large
  series is out of scope.'
tools:
- name: gallifrey
  role: 'Runs the SMC structure-learning condition: GPModel with prior over kernel
    structures/hyperparameters, involutive MCMC within SMC, and predictive mixture
    distribution used to compute RMSE and interval coverage'
  repo: github.com/ChrisBoettner/gallifrey
- name: JAX
  role: Backend for efficient numerical computation of the GP posterior and SMC sampling;
    CPU device count is configured to parallelize particles
- name: Python
  role: 'Language for the benchmark driver: data generation, baseline fitting, metric
    computation, and CSV aggregation'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202554518
    title: 'gallifrey: JAX-based Gaussian process structure learning for astronomical
      time series'
  source_content_hash: sha256:5000f16d56d11b99d31b30319c34eeaae9d2f6c9073b23b98ae9c5fff9ae10e4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - JAX
  - Python
  - gallifrey
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# predictive-accuracy-evaluation

## Summary

Benchmark protocol for evaluating GP structure learning (gallifrey's SMC with involutive MCMC) against traditional fixed-kernel GP baselines using predictive RMSE and prediction-interval coverage, aggregated into a per-method comparison table. It operationalizes the paper's claim that structure learning has advantages in accuracy and uncertainty estimation.

## When to use

Use when comparing a learned GP covariance structure against a fixed-kernel GP baseline on time series (e.g. synthetic series with known ground-truth kernel labels), and you need both point-prediction accuracy (RMSE) and uncertainty-calibration (interval coverage) metrics.

## When NOT to use

- The covariance structure is already known and fixed — gallifrey exists precisely for cases where the covariance function cannot be specified explicitly, so a learned-structure benchmark adds cost without information.
- No held-out data or ground-truth kernel labels exist, so RMSE and coverage against known structure cannot be computed and the paper's validation criterion cannot be evaluated.
- Datasets are large: the paper flags scalability to larger datasets (e.g. PLATO-scale) as future algorithmic work, so extrapolating the benchmark result to very large series is out of scope.

## Inputs

- Synthetic benchmark time series (x, y arrays) generated from known GP kernel compositions, with ground-truth kernel labels per series
- Train/test split of each series (e.g. train x < 10, extrapolate to x ~ 18 as in the README example)
- A fixed-kernel GP baseline configuration fitted to the same series

## Outputs

- Per-series predictive RMSE for structure-learning and fixed-kernel conditions
- Per-series prediction-interval coverage for both conditions
- Aggregated CSV comparison table with columns: method, RMSE, interval coverage

## How to apply

Load synthetic benchmark time series generated from known GP kernel compositions, retaining the ground-truth kernel label for each series. Fit gallifrey's GPModel with SMC structure learning: initialize with num_particles particles, generate an annealing schedule (e.g. LinearSchedule over the training points), then run fit_smc with n_mcmc MCMC steps per SMC iteration and n_hmc HMC steps inside each, updating the model from the final SMC state. Fit the traditional fixed-kernel GP baseline to the identical series. For both conditions, form the predictive mixture distribution via get_mixture_distribution on held-out x values and compute predictive RMSE (mean vs held-out y) and prediction-interval coverage from dist.mean() and dist.stddev(). Aggregate per-series metrics into a single CSV with columns method, RMSE, and interval coverage; the benchmark is accepted only if the table reproduces the paper's reported result that structure learning outperforms fixed-kernel approaches in both accuracy and uncertainty estimation.

## Related tools

- **gallifrey** (Runs the SMC structure-learning condition: GPModel with prior over kernel structures/hyperparameters, involutive MCMC within SMC, and predictive mixture distribution used to compute RMSE and interval coverage) — github.com/ChrisBoettner/gallifrey
- **JAX** (Backend for efficient numerical computation of the GP posterior and SMC sampling; CPU device count is configured to parallelize particles)
- **Python** (Language for the benchmark driver: data generation, baseline fitting, metric computation, and CSV aggregation)

## Examples

```
dist = gpmodel.get_mixture_distribution(xtest); predictive_mean = dist.mean(); predictive_std = dist.stddev()
```

## Evaluation signals

- The aggregated CSV contains both conditions (structure-learning and fixed-kernel) with non-null method, RMSE, and interval-coverage columns for every benchmark series.
- Structure-learning RMSE is lower than the fixed-kernel baseline RMSE across the synthetic series, reproducing the paper's reported accuracy advantage.
- Prediction-interval coverage of the structure-learning condition is better (nearer nominal coverage) than the fixed-kernel baseline, reproducing the reported uncertainty-estimation advantage.
- The SMC fit converged: gpmodel.update_state(final_smc_state) was called on the final SMC state from fit_smc before predictions, with a valid annealing schedule of length matching the training points.
- Held-out predictions extend beyond the training domain (train x < 10, predict to x ~ 18) so extrapolation quality, not just interpolation, is tested.

## Limitations

- Scalability to larger datasets is identified as future work needed for missions such as PLATO, so benchmark conclusions may not transfer to large-N series.
- The comparison is validated on synthetic series generated from known GP kernel compositions with ground-truth labels; performance on real astronomical signals without such labels is not directly certified by this benchmark.
- Results depend on SMC hyperparameters (num_particles, n_mcmc, n_hmc, annealing schedule); under-resourced SMC runs may fail to reproduce the reported advantage.

## Evidence

- [abstract] Structure learning has advantages in accuracy and uncertainty estimation compared with traditional fixed-kernel approaches: "When compared with traditional fixed-kernel approaches, we show that structure learning has advantages in terms of accuracy and uncertainty estimation"
- [abstract] SMC with involutive MCMC defines the structure-learning condition: "defines a prior distribution over kernel structures and hyperparameters, and efficiently samples the GP posterior distribution using a novel involutive Markov chain Monte Carlo procedure"
- [readme] README prediction interface for RMSE/coverage computation: [evidence span withheld — anchor status: external_doc]
- [readme] README fit_smc parameters governing benchmark reproducibility: "annealing_schedule=annealing_schedule,
    n_mcmc=50,      # Number of MCMC steps per SMC iteration
    n_hmc=10,       # Number of HMC steps within each MCMC step"
- [readme] README motivation for uncertainty focus: [evidence span withheld — anchor status: external_doc]
