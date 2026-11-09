---
name: smc-posterior-approximation
description: Use when you need to sample the posterior of a GP model whose covariance
  structure is itself uncertain — e.g. inferring kernel compositions from noisy time
  series (stellar variability, exoplanet transit light curves) rather than fixing
  a kernel a priori.
when_to_use_negative:
- The dataset is very large — the article lists scalability to larger datasets as
  future work needed for missions such as PLATO
- The covariance function is already known and fixed — a conventional fixed-kernel
  GP package is the appropriate choice, and structure learning adds cost without benefit
- The problem is not time series / GP regression — gallifrey is scoped to Bayesian
  structure learning with Gaussian Process models for time series data
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: gallifrey
  role: Implements the SMC loop (fit_smc) with involutive MCMC for GP kernel structure
    learning and posterior sampling
  repo: github.com/ChrisBoettner/gallifrey
- name: JAX
  role: Backend for efficient numerical computation; configure host platform device
    count for CPU parallelism across SMC particles
- name: GPJax
  role: Inspiration for the underlying Gaussian Process implementation used inside
    the SMC sampler
  repo: https://docs.jaxgaussianprocesses.com/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202554518
    title: 'gallifrey: JAX-based Gaussian process structure learning for astronomical
      time series'
  source_content_hash: sha256:d8a91a6fc1c62b39652a9005e3ede09f65e85d1361f93aa0a64aaaea3ceb49b4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GPJax
  - JAX
  - Python
  - gallifrey
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# smc-posterior-approximation

## Summary

Robust and fast approximation of the GP posterior over kernel structures and hyperparameters using Sequential Monte Carlo with involutive MCMC moves, as implemented in the gallifrey JAX package for time series structure learning.

## When to use

Use when you need to sample the posterior of a GP model whose covariance structure is itself uncertain — e.g. inferring kernel compositions from noisy time series (stellar variability, exoplanet transit light curves) rather than fixing a kernel a priori.

## When NOT to use

- The dataset is very large — the article lists scalability to larger datasets as future work needed for missions such as PLATO
- The covariance function is already known and fixed — a conventional fixed-kernel GP package is the appropriate choice, and structure learning adds cost without benefit
- The problem is not time series / GP regression — gallifrey is scoped to Bayesian structure learning with Gaussian Process models for time series data

## Inputs

- Training time series (x, y) as JAX arrays, optionally split into train/test portions
- GPConfig defining the kernel vocabulary / prior over kernel structures and hyperparameters
- Number of SMC particles (e.g. num_particles=8)
- Annealing schedule (e.g. LinearSchedule over data points)

## Outputs

- Final SMC particle state (final_smc_state) and iteration history
- Updated GPModel with posterior-weighted kernel particle ensemble
- Predictive mixture distribution (mean and standard deviation) at test points

## How to apply

Initialize a gallifrey GPModel with training x/y (JAX arrays), a GPConfig, and a chosen number of SMC particles (README example uses num_particles=8). Generate an annealing schedule (e.g. LinearSchedule().generate(len(xtrain), 10)) since tempering is important for SMC stability, then run gpmodel.fit_smc with n_mcmc MCMC steps per SMC iteration and n_hmc HMC steps within each (README defaults: n_mcmc=50, n_hmc=10). SMC sequentially reweights and resamples particles over kernel structures and hyperparameters, using involutive MCMC proposals; the rationale is robust posterior approximation over a trans-dimensional space that single-chain MCMC handles poorly. After fitting, call gpmodel.update_state(final_smc_state) and obtain the predictive mixture distribution via gpmodel.get_mixture_distribution(xtest), from which predictive mean and standard deviation are computed for accuracy and uncertainty-coverage evaluation.

## Related tools

- **gallifrey** (Implements the SMC loop (fit_smc) with involutive MCMC for GP kernel structure learning and posterior sampling) — github.com/ChrisBoettner/gallifrey
- **JAX** (Backend for efficient numerical computation; configure host platform device count for CPU parallelism across SMC particles)
- **GPJax** (Inspiration for the underlying Gaussian Process implementation used inside the SMC sampler) — https://docs.jaxgaussianprocesses.com/

## Examples

```
final_smc_state, history = gpmodel.fit_smc(smc_key, annealing_schedule=LinearSchedule().generate(len(xtrain), 10), n_mcmc=50, n_hmc=10, verbosity=1)
```

## Evaluation signals

- The annealing schedule is non-empty and matches the training set length (LinearSchedule().generate(len(xtrain), 10) yields 10 stages)
- fit_smc returns a final_smc_state that converts into a valid GPModel via gpmodel.update_state without shape errors
- get_mixture_distribution(xtest) returns a distribution whose mean and stddev are finite over the test grid
- Predictive RMSE and prediction-interval coverage on held-out data should favour structure learning over fixed-kernel baselines, reproducing the paper's reported advantage in accuracy and uncertainty estimation
- Per-series metrics aggregated into a comparison table with columns for method, RMSE, and interval coverage should match the benchmark's acceptance criterion

## Limitations

- Scalability to larger datasets is an open algorithmic challenge; the paper discusses a road map for improvements needed for missions such as PLATO
- Requires Python 3.10+ and pinned dependency versions (jax >=0.5.0,<0.6.0; blackjax >=1.2.5,<2.0.0; flax, equinox, beartype, tensorflow-probability with strict version caps)
- Posterior quality depends on particle count, annealing schedule, and MCMC/HMC step settings; too few particles or MCMC steps degrade approximation

## Evidence

- [abstract] We present gallifrey, a JAX-based Python package that implements a sequential Monte Carlo algorithm for Bayesian kernel structure learning: "We present gallifrey, a JAX-based Python package that implements a sequential Monte Carlo algorithm for Bayesian kernel structure learning"
- [abstract] defines a prior distribution over kernel structures and hyperparameters, and efficiently samples the GP posterior distribution using a novel involutive Markov chain Monte Carlo procedure: "defines a prior distribution over kernel structures and hyperparameters, and efficiently samples the GP posterior distribution using a novel involutive Markov chain Monte Carlo procedure"
- [intro] gallifrey utilizes JAX for efficient numerical computation and Sequential Monte Carlo (SMC) methods for robust posterior approximation.: "gallifrey utilizes JAX for efficient numerical computation and Sequential Monte Carlo (SMC) methods for robust posterior approximation."
- [readme] Employs SMC for robust and fast posterior approximations.: [evidence span withheld — anchor status: external_doc]
- [readme] Generate an annealing schedule (important for SMC): [evidence span withheld — anchor status: external_doc]
