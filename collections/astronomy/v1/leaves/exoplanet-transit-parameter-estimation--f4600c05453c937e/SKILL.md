---
name: exoplanet-transit-parameter-estimation
description: Use when you have a (synthetic or real) transit light curve as time/flux
  arrays and need posterior credible intervals on transit parameters such as depth,
  duration, and ephemeris, or a transmission spectrum, without pre-specifying a GP
  covariance kernel.
when_to_use_negative:
- When the GP covariance function is already known and fixed-kernel GP regression
  suffices — the article notes structure learning's advantages apply relative to traditional
  fixed-kernel approaches, but a simple fixed kernel may be adequate for well-characterized,
  high-SNR transits
- When the dataset is very large and scalability is critical — the paper flags scalability
  to larger datasets as future work on its road map for missions such as PLATO
- When the input is not a time series (e.g. already a parameter table or feature matrix)
  — gallifrey is focused on time series data
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3391
- http://edamontology.org/topic_3318
tools:
- name: gallifrey
  role: 'Core package: defines a prior over kernel structures and hyperparameters
    and samples the GP posterior via SMC with involutive MCMC for transit light curve
    fitting'
  repo: github.com/ChrisBoettner/gallifrey
- name: JAX
  role: Backend for efficient numerical computation of the GP model and SMC sampler
    on the light curve arrays
  repo: github.com/ChrisBoettner/gallifrey
- name: Python
  role: Language in which the transit fitting workflow is scripted (requires Python
    >= 3.10)
- name: AutoGP.jl
  role: Original Julia SMC structure-learning implementation that gallifrey re-implements;
    reference for the algorithm
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202554518
    title: 'gallifrey: JAX-based Gaussian process structure learning for astronomical
      time series'
  source_content_hash: sha256:11d10a52c635685f0cba9cdc8de44871c2e9ff43f302bc330d62fbff7ddbfca1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoGP.jl
  - JAX
  - Python
  - gallifrey
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3391
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# exoplanet-transit-parameter-estimation

## Summary

Recover exoplanet transit parameters (depth, duration, ephemeris) with robust posterior uncertainties from a light curve by separating the transit signal from the background using gallifrey's Bayesian Gaussian process structure learning with Sequential Monte Carlo.

## When to use

Use when you have a (synthetic or real) transit light curve as time/flux arrays and need posterior credible intervals on transit parameters such as depth, duration, and ephemeris, or a transmission spectrum, without pre-specifying a GP covariance kernel.

## When NOT to use

- When the GP covariance function is already known and fixed-kernel GP regression suffices — the article notes structure learning's advantages apply relative to traditional fixed-kernel approaches, but a simple fixed kernel may be adequate for well-characterized, high-SNR transits
- When the dataset is very large and scalability is critical — the paper flags scalability to larger datasets as future work on its road map for missions such as PLATO
- When the input is not a time series (e.g. already a parameter table or feature matrix) — gallifrey is focused on time series data

## Inputs

- Transit light curve as time stamps and flux measurements (loaded as JAX arrays), e.g. a synthetic cond_exoplanet_transit curve with injected transit depth, duration, and ephemeris
- Optional GPConfig for kernel prior settings and num_particles for the SMC sampler

## Outputs

- Posterior samples of transit parameters (depth, duration, ephemeris) from the learned GP model
- Posterior credible intervals per transit parameter, exported as a table (e.g. transit_parameters.csv)
- Background/transit signal separation usable for transmission spectra derivation; predictive mean and stddev for the fitted light curve

## How to apply

Load the light curve (time stamps and flux measurements with injected or measured transit depth, duration, and ephemeris) as a JAX array. Initialize a gallifrey GPModel with a prior distribution over kernel structures and hyperparameters (GPConfig) rather than a fixed covariance function, choosing a particle count for SMC (e.g. 8 in the README quick start). Sample the GP posterior with gallifrey's Sequential Monte Carlo algorithm, using the involutive MCMC procedure and a generated annealing schedule (e.g. LinearSchedule) with per-iteration MCMC/HMC step counts. Extract posterior samples of the transit parameters (depth, duration, ephemeris) from the learned GP model, which separates the background from the transit signal, and compute posterior credible intervals for each parameter. Verify each injected transit value falls inside its reported credible interval and export the recovered parameter table with intervals (e.g. transit_parameters.csv); the same background/transit separation supports deriving transmission spectra.

## Related tools

- **gallifrey** (Core package: defines a prior over kernel structures and hyperparameters and samples the GP posterior via SMC with involutive MCMC for transit light curve fitting) — github.com/ChrisBoettner/gallifrey
- **JAX** (Backend for efficient numerical computation of the GP model and SMC sampler on the light curve arrays) — github.com/ChrisBoettner/gallifrey
- **Python** (Language in which the transit fitting workflow is scripted (requires Python >= 3.10))
- **AutoGP.jl** (Original Julia SMC structure-learning implementation that gallifrey re-implements; reference for the algorithm)

## Examples

```
gpmodel = GPModel(model_key, x=xtrain, y=ytrain, num_particles=8, config=GPConfig()); final_smc_state, history = gpmodel.fit_smc(smc_key, annealing_schedule=LinearSchedule().generate(len(xtrain), 10), n_mcmc=50, n_hmc=10)
```

## Evaluation signals

- Each injected (or known) transit parameter value falls inside its reported posterior credible interval
- The learned GP successfully separates the background from the transit signal, enabling transmission spectra derivation
- Structure learning yields tighter/more accurate uncertainty estimation than a fixed-kernel GP baseline on the same light curve
- Predictive distribution from get_mixture_distribution gives a mean that tracks the transit shape with a sensible predictive stddev band
- SMC run converges with an annealing schedule over the training points (e.g. LinearSchedule with ~10 iterations) and updates the model from the final SMC state before prediction

## Limitations

- Scalability to larger datasets is identified as future work needed for missions such as PLATO
- SMC quality depends on the number of particles and MCMC/HMC steps per iteration; too few particles or steps can give poor posterior approximation
- Requires Python >= 3.10 and pinned dependency versions (jax >=0.5.0,<0.6.0; blackjax >=1.2.5,<2.0.0), which may constrain environments

## Evidence

- [abstract] defines a prior distribution over kernel structures and hyperparameters, and efficiently samples the GP posterior distribution using a novel involutive Markov chain Monte Carlo procedure: "defines a prior distribution over kernel structures and hyperparameters, and efficiently samples the GP posterior distribution using a novel involutive Markov chain Monte Carlo procedure"
- [abstract] We applied gallifrey to common astronomical time series tasks, including stellar variability modelling, exoplanet transit modelling, and transmission spectroscopy: "We applied gallifrey to common astronomical time series tasks, including stellar variability modelling, exoplanet transit modelling, and transmission spectroscopy"
- [abstract] When compared with traditional fixed-kernel approaches, we show that structure learning has advantages in terms of accuracy and uncertainty estimation: "When compared with traditional fixed-kernel approaches, we show that structure learning has advantages in terms of accuracy and uncertainty estimation"
- [intro] gallifrey was created with exoplanet transit light curves in mind, but is applicable to a wide variety of time series modelling, analysis, and forecasting tasks.: "gallifrey was created with exoplanet transit light curves in mind, but is applicable to a wide variety of time series modelling, analysis, and forecasting tasks."
- [readme] final_smc_state, history = gpmodel.fit_smc(smc_key, annealing_schedule=annealing_schedule, n_mcmc=50, n_hmc=10, verbosity=1): "final_smc_state, history = gpmodel.fit_smc(
    smc_key,
    annealing_schedule=annealing_schedule,
    n_mcmc=50,      # Number of MCMC steps per SMC iteration"
- [readme] dist = gpmodel.get_mixture_distribution(xtest); predictive_mean = dist.mean(); predictive_std = dist.stddev(): [evidence span withheld — anchor status: external_doc]
