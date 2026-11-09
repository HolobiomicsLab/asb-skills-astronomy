---
name: synthetic-lightcurve-fitting
description: Use when the input is a time series (time stamps + flux measurements,
  e.g. a synthetic transit light curve) and the goal is to separate a transit/background
  signal while recovering parameters with reliable posterior uncertainties.
when_to_use_negative:
- The covariance function is already known and fixed-kernel GP regression suffices
  — the paper's advantage is for cases where the kernel structure must be learned
- 'The dataset is very large: the paper flags scalability to larger datasets (e.g.
  PLATO-scale) as future work needing algorithmic improvements'
- Input is not a time series (gallifrey is focused on time series modelling, analysis,
  and forecasting)
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3391
- http://edamontology.org/topic_3318
tools:
- name: gallifrey
  role: 'Core package: GP model with Bayesian kernel structure learning, SMC fitting
    (fit_smc), and predictive mixture distribution for light curve fitting'
  repo: https://github.com/ChrisBoettner/gallifrey
- name: JAX
  role: Numerical backend for efficient GP computation and SMC sampling; arrays are
    loaded as jnp arrays
- name: Python
  role: Implementation language; requires Python 3.10+ and dependencies like blackjax,
    flax, equinox
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202554518
    title: 'gallifrey: JAX-based Gaussian process structure learning for astronomical
      time series'
  source_content_hash: sha256:db22d0d8a7ced57135c3b0bcdc617edd8f2eaf94c19d383f2e69274f70979040
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChrisBoettner/gallifrey
  tools:
  - JAX
  - Python
  - gallifrey
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3391
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# synthetic-lightcurve-fitting

## Summary

Fits astronomical time series (e.g. synthetic exoplanet transit light curves) with gallifrey's Bayesian GP structure learning: a prior over kernel structures and hyperparameters is sampled via SMC with involutive MCMC, yielding posterior predictive distributions and parameter credible intervals rather than a single fixed-kernel fit.

## When to use

Use when the input is a time series (time stamps + flux measurements, e.g. a synthetic transit light curve) and the goal is to separate a transit/background signal while recovering parameters with reliable posterior uncertainties.

## When NOT to use

- The covariance function is already known and fixed-kernel GP regression suffices — the paper's advantage is for cases where the kernel structure must be learned
- The dataset is very large: the paper flags scalability to larger datasets (e.g. PLATO-scale) as future work needing algorithmic improvements
- Input is not a time series (gallifrey is focused on time series modelling, analysis, and forecasting)

## Inputs

- Light curve as JAX arrays: time stamps x and flux measurements y (e.g. synthetic cond_exoplanet_transit with injected transit depth, duration, and ephemeris)
- Optional train/test split of the time series (e.g. xtrain = x[x < 10])
- GPConfig prior specification and SMC settings (num_particles, n_mcmc, n_hmc, annealing schedule)

## Outputs

- Fitted GPModel with final SMC state over kernel structures and hyperparameters
- Posterior predictive distribution over a test grid: predictive mean and standard deviation (uncertainty band)
- Posterior samples of transit parameters (depth, duration, ephemeris) with credible intervals, e.g. saved as transit_parameters.csv

## How to apply

Load the light curve as JAX arrays (x, y) and instantiate a gallifrey GPModel with a GPConfig prior over kernel structures and hyperparameters instead of fixing a covariance function; choose the number of SMC particles (e.g. 8 in the README quick start, increase for reliability). Fit with gpmodel.fit_smc using an annealing schedule such as LinearSchedule().generate(len(xtrain), 10), specifying n_mcmc (e.g. 50) and n_hmc (e.g. 10) steps per SMC iteration, then update the model state with gpmodel.update_state(final_smc_state). Obtain the predictive distribution with gpmodel.get_mixture_distribution(xtest) and extract posterior means and standard deviations (dist.mean(), dist.stddev()) to build credible intervals for transit parameters (depth, duration, ephemeris). The rationale is that structure learning infers the covariance from the data itself, which the paper shows improves accuracy and uncertainty estimation over fixed-kernel approaches and cleanly separates background variability from the transit signal.

## Related tools

- **gallifrey** (Core package: GP model with Bayesian kernel structure learning, SMC fitting (fit_smc), and predictive mixture distribution for light curve fitting) — https://github.com/ChrisBoettner/gallifrey
- **JAX** (Numerical backend for efficient GP computation and SMC sampling; arrays are loaded as jnp arrays)
- **Python** (Implementation language; requires Python 3.10+ and dependencies like blackjax, flax, equinox)

## Examples

```
gpmodel = GPModel(model_key, x=xtrain, y=ytrain, num_particles=8, config=GPConfig()); final_smc_state, history = gpmodel.fit_smc(smc_key, annealing_schedule=LinearSchedule().generate(len(xtrain), 10), n_mcmc=50, n_hmc=10)
```

## Evaluation signals

- Each injected (known) transit value falls inside its reported posterior credible interval
- Recovered parameter table (transit_parameters.csv) contains depth, duration, and ephemeris with finite credible intervals
- Predictive distribution is a valid GP mixture: dist.mean() and dist.stddev() over xtest form a smooth mean curve with a non-degenerate uncertainty band
- SMC run converges: final_smc_state produced by gpmodel.fit_smc with the annealing schedule, then gpmodel.update_state applied without error
- Structure-learning fit shows better accuracy/uncertainty than a fixed-kernel GP baseline, as the paper claims for structure learning

## Limitations

- Scalability to larger datasets is identified as future work needed for missions such as PLATO
- SMC requires an annealing schedule and enough particles/MCMC steps; too few (e.g. num_particles=8, n_mcmc=50 as in the demo) may give rough posterior approximations
- Version pinning: requires Python >= 3.10, jax >=0.5.0,<0.6.0, and specific blackjax/flax/equinox versions

## Evidence

- [abstract] defines a prior distribution over kernel structures and hyperparameters, and efficiently samples the GP posterior distribution using a novel involutive Markov chain Monte Carlo procedure: "defines a prior distribution over kernel structures and hyperparameters, and efficiently samples the GP posterior distribution using a novel involutive Markov chain Monte Carlo procedure"
- [abstract] structure learning has advantages in accuracy and uncertainty estimation compared with fixed kernels: "we show that structure learning has advantages in terms of accuracy and uncertainty estimation"
- [intro] Unlike most Gaussian Process packages, where a covariance function needs to be specified explicitly, gallifrey infers the covariance structure from the time series.: [evidence span withheld — anchor status: not_found]
- [readme] README quick start: SMC fitting call with annealing schedule and MCMC step counts: "final_smc_state, history = gpmodel.fit_smc(
    smc_key,
    annealing_schedule=annealing_schedule,
    n_mcmc=50,      # Number of MCMC steps per SMC iteration"
- [readme] README: prediction via mixture distribution: [evidence span withheld — anchor status: external_doc]
