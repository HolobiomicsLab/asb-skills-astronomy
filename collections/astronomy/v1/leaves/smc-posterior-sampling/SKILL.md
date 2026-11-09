---
name: smc-posterior-sampling
description: Use when a time series (e.g. a stellar-variability curve or transit light
  curve loaded as a JAX array) needs GP regression where the covariance structure
  should be inferred from the data rather than specified explicitly, and posterior
  samples with uncertainty estimates are required.
when_to_use_negative:
- The user already has a physically justified fixed covariance function and only needs
  hyperparameter optimization — traditional fixed-kernel GP packages are appropriate,
  and structure learning adds cost
- The dataset is very large and scalability is critical — the paper identifies scalability
  to larger datasets (e.g. PLATO-scale missions) as future work needing algorithmic
  improvements
- The input is not a time series but, e.g., already-aggregated features or non-sequential
  data where GP covariance-structure learning does not apply
edam_operation: http://edamontology.org/operation_3659
tools:
- name: gallifrey
  role: Provides GPModel, GPConfig and the fit_smc SMC loop with involutive MCMC rejuvenation
    used for Bayesian kernel structure learning
  repo: github.com/ChrisBoettner/gallifrey
- name: JAX
  role: Numerical backend enabling efficient computation of the GP posterior and particle-parallel
    SMC across CPU cores
- name: blackjax
  role: MCMC/HMC dependency underlying the n_mcmc and n_hmc rejuvenation steps inside
    each SMC iteration (pinned >=1.2.5,<2.0.0 in gallifrey's dependencies)
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202554518
    title: 'gallifrey: JAX-based Gaussian process structure learning for astronomical
      time series'
  source_content_hash: sha256:b7dce9298e452bec9f59fcf149467c97a2a05ef6028b0178233c4448e3fc3bd2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - JAX
  - Python
  - blackjax
  - gallifrey
  edam_operation: http://edamontology.org/operation_3659
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# smc-posterior-sampling

## Summary

Approximate the posterior over GP kernel structures and hyperparameters with gallifrey's Sequential Monte Carlo sampler, which pairs particle resampling with an involutive MCMC rejuvenation procedure instead of committing to a fixed covariance function. This is the core inference mechanism behind structure learning for astronomical time series such as stellar variability and exoplanet transit light curves.

## When to use

Use when a time series (e.g. a stellar-variability curve or transit light curve loaded as a JAX array) needs GP regression where the covariance structure should be inferred from the data rather than specified explicitly, and posterior samples with uncertainty estimates are required.

## When NOT to use

- The user already has a physically justified fixed covariance function and only needs hyperparameter optimization — traditional fixed-kernel GP packages are appropriate, and structure learning adds cost
- The dataset is very large and scalability is critical — the paper identifies scalability to larger datasets (e.g. PLATO-scale missions) as future work needing algorithmic improvements
- The input is not a time series but, e.g., already-aggregated features or non-sequential data where GP covariance-structure learning does not apply

## Inputs

- Time series data as JAX arrays (x timestamps, y flux/measurements), e.g. from the cond_stellar_variability or cond_exoplanet_transit example datasets
- Train/test split of the time series (e.g. observed training window x < 10 with held-out points beyond)
- GPConfig specifying the prior over kernel structures and hyperparameters (e.g. periodic and stochastic components for stellar variability)
- PRNGKey for reproducible SMC sampling
- Annealing schedule from gallifrey.schedule.LinearSchedule

## Outputs

- final_smc_state and sampling history from gpmodel.fit_smc
- Updated GPModel particle set (gpmodel.update_state(final_smc_state)) representing the learned kernel-structure mixture
- Predictive mixture distribution with mean and standard deviation at test/extrapolation points (get_mixture_distribution)
- Posterior samples of downstream parameters (e.g. transit depth, duration, ephemeris) with credible intervals
- Interpolation/extrapolation RMSE and credible-interval coverage metrics (e.g. written to stellar_variability_metrics.csv or transit_parameters.csv)

## How to apply

Initialize a GPModel with a GPConfig prior over kernel structures and hyperparameters, providing the training x/y arrays and a num_particles setting for the SMC population. Before fitting, generate an annealing schedule with LinearSchedule().generate(len(xtrain), n_schedule), since the schedule is 'important for SMC'. Run gpmodel.fit_smc(smc_key, annealing_schedule=..., n_mcmc=50, n_hmc=10), where each SMC iteration performs involutive MCMC rejuvenation with n_hmc HMC steps inside each MCMC step; JAX should be forced to expose all CPU cores via XLA_FLAGS='--xla_force_host_platform_device_count={cpu_count}' for particle-parallel execution. After the final SMC iteration, update the model with gpmodel.update_state(final_smc_state) so the particle set represents the learned kernel mixture; the posterior is then a mixture distribution over particles rather than a single fitted kernel. Prediction at held-out or extrapolation times uses gpmodel.get_mixture_distribution(xtest), returning predictive mean and stddev that reflect both kernel and hyperparameter uncertainty.

## Related tools

- **gallifrey** (Provides GPModel, GPConfig and the fit_smc SMC loop with involutive MCMC rejuvenation used for Bayesian kernel structure learning) — github.com/ChrisBoettner/gallifrey
- **JAX** (Numerical backend enabling efficient computation of the GP posterior and particle-parallel SMC across CPU cores)
- **blackjax** (MCMC/HMC dependency underlying the n_mcmc and n_hmc rejuvenation steps inside each SMC iteration (pinned >=1.2.5,<2.0.0 in gallifrey's dependencies))

## Examples

```
gpmodel = GPModel(model_key, x=xtrain, y=ytrain, num_particles=8, config=GPConfig()); annealing_schedule = LinearSchedule().generate(len(xtrain), 10); final_smc_state, history = gpmodel.fit_smc(smc_key, annealing_schedule=annealing_schedule, n_mcmc=50, n_hmc=10); gpmodel = gpmodel.update_state(final_smc_state)
```

## Evaluation signals

- fit_smc returns both final_smc_state and history; the model must be updated via gpmodel.update_state(final_smc_state) before prediction or the particle set used is stale
- Predictions must come from gpmodel.get_mixture_distribution(xtest), yielding both predictive_mean and predictive_std — a single mean curve without uncertainty indicates the SMC mixture was not used
- Interpolation and extrapolation metrics (RMSE against held-out truth and credible-interval coverage) should reproduce the paper's finding of accurate interpolation/extrapolation of stellar variability
- Structure-learning results should show better accuracy and uncertainty estimation than fixed-kernel baselines (the article's EnrichedIndex comparison)
- The annealing schedule must be generated with LinearSchedule before fitting; running fit_smc without it deviates from the documented workflow

## Limitations

- Scalability to larger datasets is identified as future work; the paper 'discuss[es] a road map for algorithmic improvements in terms of scalability to larger datasets' for missions such as PLATO
- Requires Python >= 3.10 with tightly pinned dependencies (jax >=0.5.0,<0.6.0; blackjax >=1.2.5,<2.0.0; flax, equinox, beartype, tensorflow-probability pins)
- The SMC run needs an explicit annealing schedule (LinearSchedule().generate(len(xtrain), n)) — omitting it deviates from the documented workflow
- Posterior quality depends on num_particles, n_mcmc, and n_hmc; the README quick start uses num_particles=8, n_mcmc=50, n_hmc=10, which may need scaling for harder problems

## Evidence

- [abstract] defines a prior distribution over kernel structures and hyperparameters, and efficiently samples the GP posterior distribution using a novel involutive Markov chain Monte Carlo procedure: "defines a prior distribution over kernel structures and hyperparameters, and efficiently samples the GP posterior distribution using a novel involutive Markov chain Monte Carlo procedure"
- [intro] Unlike most Gaussian Process packages, where a covariance function needs to be specified explicitly, gallifrey infers the covariance structure from the time series.: [evidence span withheld — anchor status: not_found]
- [readme] gallifrey utilizes JAX for efficient numerical computation and Sequential Monte Carlo (SMC) methods for robust posterior approximation.: "gallifrey utilizes JAX for efficient numerical computation and Sequential Monte Carlo (SMC) methods for robust posterior approximation."
- [readme] Generate an annealing schedule (important for SMC): [evidence span withheld — anchor status: external_doc]
- [readme] Employs SMC for robust and fast posterior approximations.: [evidence span withheld — anchor status: external_doc]
