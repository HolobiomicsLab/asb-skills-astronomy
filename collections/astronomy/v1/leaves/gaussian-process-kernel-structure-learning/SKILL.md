---
name: gaussian-process-kernel-structure-learning
description: Use when a time series (e.g., astronomical light curves of stellar variability
  or exoplanet transits) must be modelled with a Gaussian Process but no covariance
  function can be justified a priori, and accurate uncertainty estimation is required.
when_to_use_negative:
- When a physically motivated covariance function is already known and fixed-kernel
  GP regression suffices — structure learning is aimed at inferring the covariance
  structure from the series itself
- 'When the dataset is very large: the paper explicitly flags scalability to larger
  datasets (e.g., for missions such as PLATO) as future algorithmic work, not a current
  capability'
- When the workflow requires non-JAX environments or Python < 3.10, since gallifrey
  depends on jax (>=0.5.0,<0.6.0) and Python 3.10+
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: gallifrey
  role: Python package providing the GP model, kernel-structure prior, and the SMC
    sampler with involutive MCMC transitions used to learn the covariance structure
    from the time series
  repo: github.com/ChrisBoettner/gallifrey
- name: JAX
  role: Numerical backend for JIT-compiled GP evaluation and MCMC/SMC transitions;
    parallelism across CPU cores is enabled via xla_force_host_platform_device_count
- name: AutoGP.jl
  role: Original Julia package whose SMC structure-learning algorithm gallifrey re-implements;
    cite alongside gallifrey
- name: GPJax
  role: Inspiration for gallifrey's Gaussian Process implementation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202554518
    title: 'gallifrey: JAX-based Gaussian process structure learning for astronomical
      time series'
  source_content_hash: sha256:e170d41418835fe04322edc60559de56c28e6fec297c523a1a18528664cfea54
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoGP.jl
  - GPJax
  - JAX
  - Python
  - gallifrey
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# gaussian-process-kernel-structure-learning

## Summary

Bayesian GP kernel structure learning in which the covariance function is inferred from the time series instead of specified explicitly: a prior is placed over kernel structures and hyperparameters, and the GP posterior is sampled with Sequential Monte Carlo whose transitions use a novel involutive MCMC procedure, implemented in gallifrey on JAX.

## When to use

Use when a time series (e.g., astronomical light curves of stellar variability or exoplanet transits) must be modelled with a Gaussian Process but no covariance function can be justified a priori, and accurate uncertainty estimation is required.

## When NOT to use

- When a physically motivated covariance function is already known and fixed-kernel GP regression suffices — structure learning is aimed at inferring the covariance structure from the series itself
- When the dataset is very large: the paper explicitly flags scalability to larger datasets (e.g., for missions such as PLATO) as future algorithmic work, not a current capability
- When the workflow requires non-JAX environments or Python < 3.10, since gallifrey depends on jax (>=0.5.0,<0.6.0) and Python 3.10+

## Inputs

- Astronomical time series as arrays of time and flux measurements (x, y) in jax.numpy format (e.g., transit light curves or stellar variability data)
- GPConfig specifying gallifrey's kernel vocabulary and prior settings
- SMC settings: num_particles, annealing_schedule (e.g., LinearSchedule), n_mcmc and n_hmc step counts

## Outputs

- Final SMC state / posterior particle set of kernel structures and hyperparameters with SMC log weights
- Predictive mixture distribution (predictive mean and stddev) over test points from get_mixture_distribution
- SMC fit history from the fit_smc run

## How to apply

Load the target time series (time and flux) as jax.numpy arrays and split into training and test sets. Initialize a GPModel with gallifrey's GPConfig, choosing an SMC particle count (e.g., num_particles=8), which implicitly defines the prior over kernel structures and hyperparameters with the covariance structure treated as a latent discrete variable. Generate an annealing schedule (e.g., LinearSchedule().generate(len(xtrain), 10)) — the README marks this as important for SMC — then run gpmodel.fit_smc with n_mcmc=50 MCMC steps per SMC iteration and n_hmc=10 HMC steps within each MCMC step. Each SMC iteration reweights particles by the GP marginal likelihood, resamples adaptively when weights degenerate, and rejuvenates particles with the involutive MCMC transition kernel that jointly updates discrete structure and continuous hyperparameters, all JIT-compiled in JAX. Finally call gpmodel.update_state(final_smc_state) and obtain predictions via get_mixture_distribution(xtest), which mixes the per-particle GP posteriors weighted by SMC log weights.

## Related tools

- **gallifrey** (Python package providing the GP model, kernel-structure prior, and the SMC sampler with involutive MCMC transitions used to learn the covariance structure from the time series) — github.com/ChrisBoettner/gallifrey
- **JAX** (Numerical backend for JIT-compiled GP evaluation and MCMC/SMC transitions; parallelism across CPU cores is enabled via xla_force_host_platform_device_count)
- **AutoGP.jl** (Original Julia package whose SMC structure-learning algorithm gallifrey re-implements; cite alongside gallifrey)
- **GPJax** (Inspiration for gallifrey's Gaussian Process implementation)

## Examples

```
from gallifrey.model import GPConfig, GPModel; gpmodel = GPModel(key, x=xtrain, y=ytrain, num_particles=8, config=GPConfig()); final_smc_state, history = gpmodel.fit_smc(smc_key, annealing_schedule=LinearSchedule().generate(len(xtrain), 10), n_mcmc=50, n_hmc=10)
```

## Evaluation signals

- The pipeline runs end-to-end in gallifrey/JAX (fit_smc -> update_state -> get_mixture_distribution) without errors on Python >= 3.10 with jax >=0.5.0,<0.6.0
- The posterior particle set is non-degenerate: adaptive resampling has prevented all particle weight from collapsing onto a single kernel structure
- Predictive distribution is a proper GP mixture: dist.mean() and dist.stddev() return finite values over the test grid and uncertainty bands widen appropriately away from training data (extrapolation regime)
- Learned structures improve accuracy and uncertainty estimation relative to traditional fixed-kernel approaches (the paper's qualitative acceptance criterion; no numeric threshold is reported)
- Trace the fit history returned by fit_smc to confirm the annealing schedule advanced over the full dataset length (e.g., LinearSchedule().generate(len(xtrain), 10) with 10 steps)

## Limitations

- Scalability to larger datasets is identified as future work needed for missions such as PLATO
- Performance and reliability depend on SMC particle count and MCMC step counts; too few particles or rejuvenation steps leads to weight degeneracy
- Validation of learned structures is qualitative (accuracy/uncertainty improvement over fixed kernels); the paper reports no numeric threshold
- gallifrey is a re-implementation of AutoGP.jl; results inherit that algorithm's design and should cite Saad et al. as well

## Evidence

- [abstract] defines a prior distribution over kernel structures and hyperparameters, and efficiently samples the GP posterior distribution using a novel involutive Markov chain Monte Carlo procedure: "defines a prior distribution over kernel structures and hyperparameters, and efficiently samples the GP posterior distribution using a novel involutive Markov chain Monte Carlo procedure"
- [intro] Unlike most Gaussian Process packages, where a covariance function needs to be specified explicitly, gallifrey infers the covariance structure from the time series.: [evidence span withheld — anchor status: not_found]
- [readme] utilizes JAX for efficient numerical computation and Sequential Monte Carlo (SMC) methods for robust posterior approximation: "gallifrey utilizes JAX for efficient numerical computation and Sequential Monte Carlo (SMC) methods for robust posterior approximation."
- [readme] Generate an annealing schedule (important for SMC) ... final_smc_state, history = gpmodel.fit_smc(smc_key, annealing_schedule=annealing_schedule, n_mcmc=50, n_hmc=10): "# Generate an annealing schedule (important for SMC)
annealing_schedule = LinearSchedule().generate(len(xtrain), 10)"
- [abstract] We discuss a road map for algorithmic improvements in terms of scalability to larger datasets: "We discuss a road map for algorithmic improvements in terms of scalability to larger datasets"
