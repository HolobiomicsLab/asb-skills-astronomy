---
name: astronomical-time-series-modelling
description: Use when fitting GP models to astronomical time series (time and flux
  measurements) — e.g. stellar variability, exoplanet transit light curves, transmission
  spectroscopy — and the covariance structure should be inferred from the data rather
  than fixed by hand.
when_to_use_negative:
- When the user already requires a specific fixed covariance function — gallifrey's
  value is inferring the covariance structure from the time series, unlike packages
  where 'a covariance function needs to be specified explicitly'
- 'For very large datasets: the paper identifies ''a road map for algorithmic improvements
  in terms of scalability to larger datasets'' as needed for future missions such
  as PLATO'
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: gallifrey
  role: Python/JAX package implementing the SMC Bayesian kernel structure learning
    algorithm with involutive MCMC transitions
  repo: github.com/ChrisBoettner/gallifrey
- name: JAX
  role: Numerical computation backend executing the SMC and involutive MCMC kernels
    (supports multi-core via --xla_force_host_platform_device_count)
- name: AutoGP.jl
  role: Original Julia package by Feras Saad that gallifrey directly re-implements
    in Python/JAX
  repo: https://probsys.github.io/AutoGP.jl/stable/index.html
- name: GPJax
  role: Reference package whose Gaussian process implementation inspired gallifrey's
    GP code (together with tinygp)
  repo: https://docs.jaxgaussianprocesses.com/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202554518
    title: 'gallifrey: JAX-based Gaussian process structure learning for astronomical
      time series'
  source_content_hash: sha256:d5d5952a760e32e6ef85979f8e933ed3f0b15675fe84a9d29e450c9b7edab937
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoGP.jl
  - GPJax
  - JAX
  - Python
  - gallifrey
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# astronomical-time-series-modelling

## Summary

Bayesian Gaussian process structure learning for astronomical time series: instead of specifying a covariance function explicitly, place a prior over kernel structures and hyperparameters and sample the GP posterior with Sequential Monte Carlo using a novel involutive MCMC transition kernel, implemented in gallifrey/JAX.

## When to use

Use when fitting GP models to astronomical time series (time and flux measurements) — e.g. stellar variability, exoplanet transit light curves, transmission spectroscopy — and the covariance structure should be inferred from the data rather than fixed by hand.

## When NOT to use

- When the user already requires a specific fixed covariance function — gallifrey's value is inferring the covariance structure from the time series, unlike packages where 'a covariance function needs to be specified explicitly'
- For very large datasets: the paper identifies 'a road map for algorithmic improvements in terms of scalability to larger datasets' as needed for future missions such as PLATO

## Inputs

- Astronomical time series: arrays of time and flux measurements (numpy/pandas, converted to jnp arrays for x and y)
- Training/test split of the series (e.g. xtrain = x[x < 10])
- SMC configuration: number of particles, annealing schedule (LinearSchedule), n_mcmc and n_hmc step counts
- GPConfig kernel vocabulary defining the prior over kernel structures

## Outputs

- Posterior particle set over GP kernel structures and hyperparameters (final_smc_state plus iteration history from fit_smc)
- Posterior kernel structure samples with SMC log weights (e.g. saved to smc_kernel_samples.csv per the source workflow)
- Predictive distribution (GP mixture) with predictive mean and standard deviation for interpolation/extrapolation and uncertainty estimates

## How to apply

Load the target time series (time and flux measurements) and convert to jnp arrays, splitting into training and test sets. Initialize a GPModel via gallifrey.model.GPModel with a GPConfig, num_particles for the SMC population, treating the kernel structure as a latent discrete variable drawn from the prior over kernel structures and hyperparameters. Generate an annealing schedule (gallifrey.schedule.LinearSchedule().generate(len(xtrain), n_schedule)) — the README notes this is important for SMC — then run gpmodel.fit_smc with n_mcmc=50 MCMC steps per SMC iteration and n_hmc=10 HMC steps within each MCMC step; the transitions are the involutive MCMC kernel that jointly updates discrete structure and continuous hyperparameters. Reweight particles by the GP marginal likelihood, resample adaptively when particle weights degenerate, and iterate reweight–resample–rejuvenate over adaptive SMC steps until the terminal iteration. Finally call gpmodel.update_state(final_smc_state) and obtain the predictive mixture via gpmodel.get_mixture_distribution(xtest), extracting predictive_mean and predictive_stddev for interpolation/extrapolation and uncertainty estimation.

## Related tools

- **gallifrey** (Python/JAX package implementing the SMC Bayesian kernel structure learning algorithm with involutive MCMC transitions) — github.com/ChrisBoettner/gallifrey
- **JAX** (Numerical computation backend executing the SMC and involutive MCMC kernels (supports multi-core via --xla_force_host_platform_device_count))
- **AutoGP.jl** (Original Julia package by Feras Saad that gallifrey directly re-implements in Python/JAX) — https://probsys.github.io/AutoGP.jl/stable/index.html
- **GPJax** (Reference package whose Gaussian process implementation inspired gallifrey's GP code (together with tinygp)) — https://docs.jaxgaussianprocesses.com/

## Examples

```
from gallifrey.model import GPConfig, GPModel; gpmodel = GPModel(model_key, x=xtrain, y=ytrain, num_particles=8, config=GPConfig()); final_smc_state, history = gpmodel.fit_smc(smc_key, annealing_schedule=LinearSchedule().generate(len(xtrain), 10), n_mcmc=50, n_hmc=10)
```

## Evaluation signals

- The pipeline runs end-to-end in gallifrey/JAX without errors (fit_smc completes over the full annealing schedule).
- The terminal SMC posterior particle set is non-degenerate (adaptive resampling triggered on particle weight degeneracy did not collapse to a single particle).
- Predictive outputs are a proper GP mixture: get_mixture_distribution(xtest) yields finite predictive_mean and predictive_stddev, with uncertainty covering held-out test points.
- Qualitative paper criterion: learned structures improve accuracy and uncertainty estimation relative to traditional fixed-kernel approaches (no numeric threshold is reported).
- Compare against the reference AutoGP.jl behaviour, since gallifrey is a direct re-implementation and Saad et al. (2023) should be cited alongside.

## Limitations

- Scalability to larger datasets is future work; a road map for algorithmic improvements is discussed for missions such as PLATO.
- The validation criterion is qualitative (improved accuracy/uncertainty vs fixed kernels) — no numeric threshold is reported in the paper.
- Particle weight degeneracy in SMC requires adaptive resampling; a degenerate particle set invalidates the posterior approximation.

## Evidence

- [abstract] Prior over kernel structures and involutive MCMC sampling: "defines a prior distribution over kernel structures and hyperparameters, and efficiently samples the GP posterior distribution using a novel involutive Markov chain Monte Carlo procedure"
- [abstract] Structure learning beats fixed kernels: "When compared with traditional fixed-kernel approaches, we show that structure learning has advantages in terms of accuracy and uncertainty estimation"
- [abstract] Applied astronomical tasks: "We applied gallifrey to common astronomical time series tasks, including stellar variability modelling, exoplanet transit modelling, and transmission spectroscopy"
- [readme] Covariance inferred, not specified: "Unlike most Gaussian Process packages, where a covariance function needs to be specified explicitly, $\texttt{gallifrey}$ infers the covariance structure from the time series."
- [readme] SMC for posterior approximation: "gallifrey utilizes JAX for efficient numerical computation and Sequential Monte Carlo (SMC) methods for robust posterior approximation."
- [other] Scalability limitation: "We discuss a road map for algorithmic improvements in terms of scalability to larger datasets"
