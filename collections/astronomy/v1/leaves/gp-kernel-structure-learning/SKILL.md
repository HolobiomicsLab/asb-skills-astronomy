---
name: gp-kernel-structure-learning
description: Use when a time series (e.g. stellar variability, exoplanet transit light
  curve, transmission spectroscopy) requires GP regression but no suitable fixed covariance
  kernel is known, and accurate predictive accuracy plus uncertainty estimation are
  needed.
when_to_use_negative:
- When a domain-standard fixed covariance kernel is already known to fit the data
  well and only point estimates are needed
- When the dataset is very large — the paper identifies scalability to larger datasets
  (e.g. PLATO-scale missions) as future work
- When the analysis target is not a time series / GP regression task — gallifrey is
  focused on time series modelling, analysis, and forecasting
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3315
tools:
- name: gallifrey
  role: 'Core package: defines the prior over kernel structures/hyperparameters and
    runs SMC-based Bayesian structure learning with involutive MCMC; pip install gallifrey
    (Python >=3.10, Apache-2.0)'
  repo: github.com/ChrisBoettner/gallifrey
- name: JAX
  role: Numerical backend enabling efficient vectorized computation of the GP posterior
    and SMC loop (pinned jax >=0.5.0,<0.6.0)
- name: blackjax
  role: MCMC dependency used for the HMC rejuvenation steps inside the SMC iterations
    (blackjax >=1.2.5,<2.0.0)
- name: AutoGP.jl
  role: Original Julia package by Feras Saad that gallifrey re-implements in Python/JAX;
    its SMC structure-learning algorithm defines this skill's method
  repo: https://probsys.github.io/AutoGP.jl/stable/index.html
- name: GPJax
  role: Reference implementation that inspired gallifrey's Gaussian process implementation
    (alongside tinygp)
  repo: https://docs.jaxgaussianprocesses.com/
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_005
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202554518
    title: 'gallifrey: JAX-based Gaussian process structure learning for astronomical
      time series'
  source_content_hash: sha256:a84bad88b7db0ab22c262e0a21435bfdd1e58b251ec2315c7a891b22c9d271c7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoGP.jl
  - GPJax
  - JAX
  - Python
  - blackjax
  - gallifrey
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# gp-kernel-structure-learning

## Summary

Bayesian GP kernel structure learning with gallifrey: a prior over kernel structures and hyperparameters is combined with Sequential Monte Carlo sampling (involutive MCMC rejuvenation moves) under JAX to infer the GP covariance structure from time series data instead of fixing it a priori.

## When to use

Use when a time series (e.g. stellar variability, exoplanet transit light curve, transmission spectroscopy) requires GP regression but no suitable fixed covariance kernel is known, and accurate predictive accuracy plus uncertainty estimation are needed.

## When NOT to use

- When a domain-standard fixed covariance kernel is already known to fit the data well and only point estimates are needed
- When the dataset is very large — the paper identifies scalability to larger datasets (e.g. PLATO-scale missions) as future work
- When the analysis target is not a time series / GP regression task — gallifrey is focused on time series modelling, analysis, and forecasting

## Inputs

- Time series arrays (time stamps and measurements) as JAX arrays, e.g. synthetic benchmark series with ground-truth kernel labels
- gallifrey example datasets: cond_exoplanet_transit, cond_transmission_spectroscopy, cond_stellar_variability
- Fixed-kernel GP baseline configurations for comparison conditions

## Outputs

- Learned GP kernel structure and particle posterior over kernels and hyperparameters
- Predictive mean, standard deviation, and credible intervals at held-out/extrapolated points
- Posterior samples of transit parameters (depth, duration, ephemeris) with credible intervals
- Comparison metric tables (CSV, e.g. RMSE and interval coverage per method; transit_parameters.csv; transmission_spectrum.csv; stellar_variability_metrics.csv)

## How to apply

Load the time series (e.g. synthetic benchmarks from known kernel compositions, cond_exoplanet_transit, cond_transmission_spectroscopy, or cond_stellar_variability example data) as JAX arrays and split into training and held-out points. Initialize a gallifrey GPModel with a prior distribution over kernel structures and hyperparameters (num_particles controlling the SMC particle count) rather than a fixed covariance function. Generate an annealing schedule (e.g. LinearSchedule) and run fit_smc, which samples the GP posterior via SMC with n_mcmc MCMC steps per iteration and n_hmc HMC steps within each MCMC step, using involutive MCMC moves for structure rejuvenation. Update the model with the final SMC state, then obtain the predictive mixture distribution at test points for interpolation/extrapolation or posterior samples of parameters such as transit depth, duration, and ephemeris. Rationale: unlike packages requiring an explicitly specified covariance function, gallifrey infers the covariance structure from the data, which the paper shows yields advantages in accuracy and uncertainty estimation versus traditional fixed-kernel baselines.

## Related tools

- **gallifrey** (Core package: defines the prior over kernel structures/hyperparameters and runs SMC-based Bayesian structure learning with involutive MCMC; pip install gallifrey (Python >=3.10, Apache-2.0)) — github.com/ChrisBoettner/gallifrey
- **JAX** (Numerical backend enabling efficient vectorized computation of the GP posterior and SMC loop (pinned jax >=0.5.0,<0.6.0))
- **blackjax** (MCMC dependency used for the HMC rejuvenation steps inside the SMC iterations (blackjax >=1.2.5,<2.0.0))
- **AutoGP.jl** (Original Julia package by Feras Saad that gallifrey re-implements in Python/JAX; its SMC structure-learning algorithm defines this skill's method) — https://probsys.github.io/AutoGP.jl/stable/index.html
- **GPJax** (Reference implementation that inspired gallifrey's Gaussian process implementation (alongside tinygp)) — https://docs.jaxgaussianprocesses.com/

## Examples

```
from gallifrey.model import GPConfig, GPModel; from gallifrey.schedule import LinearSchedule; gpmodel = GPModel(model_key, x=xtrain, y=ytrain, num_particles=8, config=GPConfig()); final_smc_state, history = gpmodel.fit_smc(smc_key, annealing_schedule=LinearSchedule().generate(len(xtrain), 10), n_mcmc=50, n_hmc=10)
```

## Evaluation signals

- Predictive RMSE and prediction-interval coverage of the learned-kernel model should beat a fixed-kernel GP baseline fit to the same series (reproducing the paper's accuracy/uncertainty advantage finding)
- Injected ground-truth transit parameter values (depth, duration, ephemeris) must fall inside the posterior credible intervals reported by the particle posterior
- Recovered wavelength-dependent transit depths in transmission_spectrum.csv must agree with injected ground truth within gallifrey's posterior uncertainties
- Held-out interpolation and beyond-window extrapolation RMSE and credible-interval coverage on the quasi-periodic stellar variability series should be populated in stellar_variability_metrics.csv
- Pipeline invariants: model must be updated with the final SMC state (gpmodel.update_state(final_smc_state)) before prediction, and the annealing schedule must be generated for the number of training points

## Limitations

- Scalability to larger datasets is an open issue; the paper discusses a road map for algorithmic improvements needed for missions such as PLATO
- SMC results depend on particle count, annealing schedule length, n_mcmc and n_hmc settings; too-few particles/steps can under-sample the structure posterior
- Requires Python >=3.10 and pinned dependency versions (jax >=0.5.0,<0.6.0, blackjax >=1.2.5,<2.0.0, etc.)
- CPU-parallel SMC particles require setting --xla_force_host_platform_device_count to the CPU count

## Evidence

- [abstract] SMC + involutive MCMC method: "defines a prior distribution over kernel structures and hyperparameters, and efficiently samples the GP posterior distribution using a novel involutive Markov chain Monte Carlo procedure"
- [abstract] Structure learning advantage over fixed kernels: "When compared with traditional fixed-kernel approaches, we show that structure learning has advantages in terms of accuracy and uncertainty estimation"
- [intro] Covariance inferred, not specified: [evidence span withheld — anchor status: not_found]
- [readme] SMC fitting API and parameters (readme): "gpmodel.fit_smc(
    smc_key,
    annealing_schedule=annealing_schedule,
    n_mcmc=50,      # Number of MCMC steps per SMC iteration
    n_hmc=10,       # Number of HMC steps within each MCMC step"
- [readme] Prediction via mixture distribution (readme): "xtest = gpmodel.x_transform(jnp.linspace(0, 18, 60)) # Create x values for prediction
dist = gpmodel.get_mixture_distribution(xtest) # Get the predictive distribution"
