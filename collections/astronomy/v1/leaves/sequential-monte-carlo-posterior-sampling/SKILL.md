---
name: sequential-monte-carlo-posterior-sampling
description: Use when the input is a time series (e.g. stellar variability or exoplanet
  transit light-curve time/flux measurements) and the GP covariance function should
  be inferred from the data rather than specified explicitly by the user.
when_to_use_negative:
- When the GP covariance function is already known and explicitly specified by the
  user — gallifrey's added machinery is for inferring unknown covariance structure.
- 'When very large datasets are the target: the paper flags scalability to larger
  datasets (e.g. future PLATO missions) as a road-map item, not a current capability.'
- 'When the data are not time series (e.g. static feature tables or images): gallifrey
  is designed for time series modelling, analysis, and forecasting tasks such as light
  curves.'
tools:
- name: gallifrey
  role: Core package implementing the SMC structure-learning algorithm, GPModel, GPConfig,
    and LinearSchedule
  repo: github.com/ChrisBoettner/gallifrey
- name: JAX
  role: Numerical backend for efficient GP computation and compilation of the SMC/involutive-MCMC
    kernels
- name: Python
  role: Runtime environment (requires Python 3.10+; installable via pip install gallifrey)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202554518
    title: 'gallifrey: JAX-based Gaussian process structure learning for astronomical
      time series'
  source_content_hash: sha256:724e4fc95ec788d0505b47acb1efcf0472c9fa8a624ff9dd5b88a88a507355c9
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
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# sequential-monte-carlo-posterior-sampling

## Summary

Performs Bayesian inference over Gaussian Process covariance structure for astronomical time series by placing a prior over kernel structures and hyperparameters and sampling the GP posterior with Sequential Monte Carlo whose transitions use a novel involutive MCMC kernel, implemented in JAX via the gallifrey package.

## When to use

Use when the input is a time series (e.g. stellar variability or exoplanet transit light-curve time/flux measurements) and the GP covariance function should be inferred from the data rather than specified explicitly by the user.

## When NOT to use

- When the GP covariance function is already known and explicitly specified by the user — gallifrey's added machinery is for inferring unknown covariance structure.
- When very large datasets are the target: the paper flags scalability to larger datasets (e.g. future PLATO missions) as a road-map item, not a current capability.
- When the data are not time series (e.g. static feature tables or images): gallifrey is designed for time series modelling, analysis, and forecasting tasks such as light curves.

## Inputs

- Time series arrays (x = time, y = flux/measurements) as jnp arrays, e.g. astronomical light-curve data
- GPConfig object specifying the kernel vocabulary / default configuration
- Annealing schedule from gallifrey.schedule.LinearSchedule (generated for len(xtrain) points)
- SMC settings: num_particles, n_mcmc (MCMC steps per iteration), n_hmc (HMC steps per MCMC step)
- JAX PRNGKey

## Outputs

- final_smc_state containing the posterior particle population over kernel structures with SMC log weights
- history of the SMC iterations returned by fit_smc
- Predictive mixture distribution (dist.mean(), dist.stddev()) at test points via get_mixture_distribution

## How to apply

Load the time series as x/y arrays (JAX numpy) and instantiate a GPModel with a GPConfig, a particle count (e.g. num_particles=8) and the covariance structure treated as a latent discrete variable drawn from the kernel-structure prior. Build an annealing schedule with LinearSchedule().generate(len(xtrain), n_schedule) — important for SMC stability — then call fit_smc with n_mcmc MCMC steps per SMC iteration and n_hmc HMC steps inside each MCMC step (README defaults: n_mcmc=50, n_hmc=10). Each SMC iteration reweights particles by the GP marginal likelihood, adaptively resamples when weights degenerate, and rejuvenates particles with the involutive MCMC transition kernel that jointly updates discrete structure and continuous hyperparameters. After the terminal iteration, update the model with the final SMC state (gpmodel.update_state(final_smc_state)) and obtain predictions as a particle mixture via gpmodel.get_mixture_distribution(xtest), from which predictive mean and stddev are read. Rationale: annealed SMC over a prior over structures plus involutive-MCMC rejuvenation avoids getting trapped in single fixed kernels and yields both a posterior over structures and well-calibrated uncertainty.

## Related tools

- **gallifrey** (Core package implementing the SMC structure-learning algorithm, GPModel, GPConfig, and LinearSchedule) — github.com/ChrisBoettner/gallifrey
- **JAX** (Numerical backend for efficient GP computation and compilation of the SMC/involutive-MCMC kernels)
- **Python** (Runtime environment (requires Python 3.10+; installable via pip install gallifrey))

## Examples

```
from gallifrey.model import GPModel, GPConfig; from gallifrey.schedule import LinearSchedule
final_smc_state, history = GPModel(key, x=xtrain, y=ytrain, num_particles=8, config=GPConfig()).fit_smc(smc_key, annealing_schedule=LinearSchedule().generate(len(xtrain), 10), n_mcmc=50, n_hmc=10)
```

## Evaluation signals

- Pipeline runs end-to-end in gallifrey/JAX without errors, including JAX device configuration via --xla_force_host_platform_device_count.
- The terminal SMC particle set is non-degenerate (particles retain diverse kernel structures rather than collapsing to a single structure).
- Learned structures qualitatively improve accuracy and uncertainty estimation relative to traditional fixed-kernel approaches (paper's stated acceptance criterion; no numeric threshold reported).
- Predictive output is a proper mixture distribution: dist.mean() and dist.stddev() return finite values over interpolated and extrapolated test points.
- update_state(final_smc_state) succeeds, confirming the fit_smc history and final state are consistent with the model.

## Limitations

- Scalability to larger datasets is identified as needed future work for missions such as PLATO.
- The advantage over fixed-kernel approaches is qualitative (accuracy and uncertainty estimation); no numeric performance threshold is reported in the paper.
- Package is a JAX-based re-implementation of AutoGP.jl; GP implementation is inspired by GPJax and tinygp, so behaviour may differ from those packages.
- Requires Python 3.10+ and pinned dependency ranges (e.g. jax >=0.5.0,<0.6.0; blackjax >=1.2.5,<2.0.0).

## Evidence

- [abstract] defines a prior distribution over kernel structures and hyperparameters, and efficiently samples the GP posterior distribution using a novel involutive Markov chain Monte Carlo procedure: "defines a prior distribution over kernel structures and hyperparameters, and efficiently samples the GP posterior distribution using a novel involutive Markov chain Monte Carlo procedure"
- [intro] Unlike most Gaussian Process packages, where a covariance function needs to be specified explicitly, gallifrey infers the covariance structure from the time series.: [evidence span withheld — anchor status: not_found]
- [readme] gallifrey utilizes JAX for efficient numerical computation and Sequential Monte Carlo (SMC) methods for robust posterior approximation.: "gallifrey utilizes JAX for efficient numerical computation and Sequential Monte Carlo (SMC) methods for robust posterior approximation."
