---
name: bayesian-credible-interval-coverage-analysis
description: Use when a Bayesian model (e.g. gallifrey's SMC-sampled GP posterior)
  yields parameter posteriors and you must confirm the credible intervals actually
  cover the true injected values before trusting downstream products like transmission
  spectra.
when_to_use_negative:
- The covariance function is already fixed and known a priori (gallifrey's structure
  learning, and hence this coverage workflow, targets cases where the kernel structure
  must be inferred from the time series itself)
- The dataset is too large for SMC-based structure learning — the paper explicitly
  flags scalability to larger datasets (e.g. PLATO-scale) as future work
- The data are not a time series (e.g. i.i.d. feature tables); gallifrey is focused
  on time series modelling, analysis, and forecasting
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: gallifrey
  role: Provides the Bayesian kernel structure-learning GP model and SMC posterior
    sampling (involutive MCMC) whose posterior credible intervals over transit parameters
    are coverage-tested
  repo: github.com/ChrisBoettner/gallifrey
- name: JAX
  role: Numerical backend for the light-curve arrays and the GP/SMC computations underlying
    the posterior sampling
- name: Python
  role: Language in which the coverage check and the transit_parameters.csv export
    are implemented
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202554518
    title: 'gallifrey: JAX-based Gaussian process structure learning for astronomical
      time series'
  source_content_hash: sha256:c552956184a453455fd61bf8262a5f8aeeb27291ff273a430dc30e2243343613
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - JAX
  - Python
  - gallifrey
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# Bayesian credible interval coverage analysis

## Summary

Verifies that Bayesian posterior uncertainties are trustworthy by checking whether injected (ground-truth) parameter values fall inside the reported posterior credible intervals. Here it is applied to gallifrey GP structure-learning posteriors over exoplanet transit parameters (depth, duration, ephemeris) from a synthetic transit light curve.

## When to use

Use when a Bayesian model (e.g. gallifrey's SMC-sampled GP posterior) yields parameter posteriors and you must confirm the credible intervals actually cover the true injected values before trusting downstream products like transmission spectra.

## When NOT to use

- The covariance function is already fixed and known a priori (gallifrey's structure learning, and hence this coverage workflow, targets cases where the kernel structure must be inferred from the time series itself)
- The dataset is too large for SMC-based structure learning — the paper explicitly flags scalability to larger datasets (e.g. PLATO-scale) as future work
- The data are not a time series (e.g. i.i.d. feature tables); gallifrey is focused on time series modelling, analysis, and forecasting

## Inputs

- Synthetic exoplanet transit light curve (time stamps + flux measurements with injected transit depth, duration, and ephemeris), e.g. the cond_exoplanet_transit dataset, loaded as a JAX array
- gallifrey GPModel posterior samples over kernel structures, hyperparameters, and transit parameters

## Outputs

- Posterior credible intervals for each transit parameter (depth, duration, ephemeris)
- Recovered parameter table with intervals saved as transit_parameters.csv
- Coverage verdict: whether each injected transit value falls inside its credible interval

## How to apply

Load the synthetic light curve (e.g. the cond_exoplanet_transit dataset with injected transit depth, duration, and ephemeris) as a JAX array, then fit a gallifrey GPModel whose prior is a distribution over kernel structures and hyperparameters rather than a fixed covariance function. Sample the GP posterior with gallifrey's sequential Monte Carlo algorithm (involutive MCMC steps, using an annealing schedule such as LinearSchedule and multiple particles/HMC steps per iteration). Extract posterior samples of each transit parameter and compute posterior credible intervals for depth, duration, and ephemeris. The coverage check is the pass criterion: each injected transit value must fall inside its reported credible interval, and the recovered parameter table with intervals is saved as transit_parameters.csv. This coverage test is the mechanism by which the paper claims structure learning has 'advantages in terms of accuracy and uncertainty estimation' over traditional fixed-kernel approaches.

## Related tools

- **gallifrey** (Provides the Bayesian kernel structure-learning GP model and SMC posterior sampling (involutive MCMC) whose posterior credible intervals over transit parameters are coverage-tested) — github.com/ChrisBoettner/gallifrey
- **JAX** (Numerical backend for the light-curve arrays and the GP/SMC computations underlying the posterior sampling)
- **Python** (Language in which the coverage check and the transit_parameters.csv export are implemented)

## Examples

```
gpmodel = GPModel(model_key, x=xtrain, y=ytrain, num_particles=8, config=GPConfig()); final_smc_state, history = gpmodel.fit_smc(smc_key, annealing_schedule=LinearSchedule().generate(len(xtrain), 10), n_mcmc=50)
```

## Evaluation signals

- Each injected transit value (depth, duration, ephemeris) falls inside its reported posterior credible interval
- transit_parameters.csv is produced with one row per parameter and explicit interval bounds
- Posterior uncertainty is compared against a traditional fixed-kernel GP baseline, where structure learning should show advantages in accuracy and uncertainty estimation
- SMC run converges sensibly: multiple particles with an annealing schedule and n_mcmc/n_hmc steps per iteration, and the final state is applied via gpmodel.update_state(final_smc_state)
- Background/transit separation is visually consistent with the injected signal (predictive mean tracks the transit, intervals quantify uncertainty)

## Limitations

- Scalability to larger datasets (e.g. future missions such as PLATO) is identified as needed future algorithmic work
- SMC-based inference is stochastic: results depend on particle count, annealing schedule, and MCMC/HMC step counts, so coverage should be rechecked for different run configurations
- Requires known injected/ground-truth parameter values to perform the coverage test; without ground truth, interval correctness cannot be directly verified

## Evidence

- [abstract] defines a prior distribution over kernel structures and hyperparameters, and efficiently samples the GP posterior distribution using a novel involutive Markov chain Monte Carlo procedure: "defines a prior distribution over kernel structures and hyperparameters, and efficiently samples the GP posterior distribution using a novel involutive Markov chain Monte Carlo procedure"
- [abstract] structure learning has advantages in terms of accuracy and uncertainty estimation: "When compared with traditional fixed-kernel approaches, we show that structure learning has advantages in terms of accuracy and uncertainty estimation"
- [readme] gallifrey utilizes JAX for efficient numerical computation and Sequential Monte Carlo (SMC) methods for robust posterior approximation.: "gallifrey utilizes JAX for efficient numerical computation and Sequential Monte Carlo (SMC) methods for robust posterior approximation."
- [readme] Implements Gaussian Processes, leveraging JAX for efficient computation, with a particular focus on accurate uncertainty estimation.: [evidence span withheld — anchor status: external_doc]
