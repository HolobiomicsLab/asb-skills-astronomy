---
name: stellar-variability-gp-modelling
description: Use when a time series (e.g. stellar light curves from the cond_stellar_variability
  example data) must be modelled and forecast without hand-picking a covariance kernel,
  and when interpolation/extrapolation accuracy and credible-interval coverage should
  be quantified.
when_to_use_negative:
- 'Very large datasets: the paper lists scalability to larger datasets (e.g. PLATO-scale
  missions) as a future-work road map, so SMC structure learning may be too costly
  there.'
- A fixed covariance kernel is already explicitly specified and well justified for
  the task, so structure learning offers no benefit over a traditional fixed-kernel
  GP.
- The data are not time series (gallifrey is focused on time series modelling, analysis,
  and forecasting).
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: gallifrey
  role: 'Primary package: GP modelling, Bayesian kernel structure learning via SMC
    with involutive MCMC rejuvenation moves, and predictive mixture distributions
    for interpolation/extrapolation'
  repo: https://github.com/ChrisBoettner/gallifrey
- name: JAX
  role: Numerical computation backend for the GP and SMC algorithm; configure XLA
    device count to parallelise particles across CPU cores
- name: Python
  role: Language for the workflow (gallifrey is a Python package; requires Python
    3.10+)
- name: AutoGP.jl
  role: Julia predecessor whose SMC structure-learning algorithm gallifrey re-implements;
    useful cross-reference for the method
  repo: https://probsys.github.io/AutoGP.jl/stable/index.html
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202554518
    title: 'gallifrey: JAX-based Gaussian process structure learning for astronomical
      time series'
  source_content_hash: sha256:9a80d30d1e8315de9e6627035ccf845a722dd9eec7ee19b25cd1047158673e81
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChrisBoettner/gallifrey
  tools:
  - AutoGP.jl
  - JAX
  - Python
  - gallifrey
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# stellar-variability-gp-modelling

## Summary

Model stellar variability time series with gallifrey, a JAX-based Gaussian process package that learns the GP covariance structure via Sequential Monte Carlo (SMC) with involutive MCMC, rather than fixing a kernel by hand. It is used to benchmark learned-kernel GP mixtures on interpolation within the observed window and extrapolation beyond it, e.g. against held-out truth.

## When to use

Use when a time series (e.g. stellar light curves from the cond_stellar_variability example data) must be modelled and forecast without hand-picking a covariance kernel, and when interpolation/extrapolation accuracy and credible-interval coverage should be quantified.

## When NOT to use

- Very large datasets: the paper lists scalability to larger datasets (e.g. PLATO-scale missions) as a future-work road map, so SMC structure learning may be too costly there.
- A fixed covariance kernel is already explicitly specified and well justified for the task, so structure learning offers no benefit over a traditional fixed-kernel GP.
- The data are not time series (gallifrey is focused on time series modelling, analysis, and forecasting).

## Inputs

- Stellar-variability time series (synthetic or observed light-curve y(t) with noise), e.g. the cond_stellar_variability example data
- Train/test split: an observed training window and held-out test points inside and beyond the window

## Outputs

- Predictive GP mixture distribution over test times (predictive mean and standard deviation, sampled with uncertainties)
- stellar_variability_metrics.csv with RMSE against held-out truth and credible-interval coverage, populated for both interpolation and extrapolation regimes

## How to apply

Load the stellar-variability series (e.g. from the cond_stellar_variability example data), split it into an observed training window and held-out test points, and initialise a GPModel in gallifrey (Python/JAX) with an SMC particle count (README quick start uses 8 particles). Configure the kernel prior over structures and hyperparameters, including periodic and stochastic components suited to stellar variability. Fit via the SMC loop with an annealing schedule (LinearSchedule over the training points), n_mcmc=50 MCMC steps per iteration and n_hmc=10 HMC steps, then update the model with the final SMC state. Use the learned-kernel GP mixture (get_mixture_distribution) to predict at held-out points (interpolation) and at times beyond the observed window (extrapolation), extracting predictive means and standard deviations. Rationale: the paper shows structure learning over kernel structures and hyperparameters, sampled with a novel involutive MCMC procedure, gives better accuracy and uncertainty estimation than traditional fixed-kernel approaches.

## Related tools

- **gallifrey** (Primary package: GP modelling, Bayesian kernel structure learning via SMC with involutive MCMC rejuvenation moves, and predictive mixture distributions for interpolation/extrapolation) — https://github.com/ChrisBoettner/gallifrey
- **JAX** (Numerical computation backend for the GP and SMC algorithm; configure XLA device count to parallelise particles across CPU cores)
- **Python** (Language for the workflow (gallifrey is a Python package; requires Python 3.10+))
- **AutoGP.jl** (Julia predecessor whose SMC structure-learning algorithm gallifrey re-implements; useful cross-reference for the method) — https://probsys.github.io/AutoGP.jl/stable/index.html

## Examples

```
final_smc_state, history = gpmodel.fit_smc(smc_key, annealing_schedule=LinearSchedule().generate(len(xtrain), 10), n_mcmc=50, n_hmc=10); dist = gpmodel.get_mixture_distribution(xtest)
```

## Evaluation signals

- stellar_variability_metrics.csv exists with error and coverage fields populated for both interpolation and extrapolation regimes
- RMSE against held-out truth is computed for in-window (interpolation) and out-of-window (extrapolation) predictions
- Credible-interval coverage of the learned-kernel predictive distribution is measured, not just point errors
- Learned-kernel results reproduce the paper's qualitative finding of accurate interpolation and extrapolation of stellar variability
- Accuracy and uncertainty estimation compare favourably against traditional fixed-kernel baselines

## Limitations

- Scalability to larger datasets is future work: the paper discusses a road map for algorithmic improvements in scalability, motivated by missions such as PLATO.
- Requires Python 3.10+ and pinned JAX ecosystem versions (jax >=0.5.0,<0.6.0, blackjax >=1.2.5,<2.0.0, etc.), so environment conflicts may arise.
- The method is a re-implementation of AutoGP.jl; behaviour differences versus the Julia original may exist.

## Evidence

- [abstract] When compared with traditional fixed-kernel approaches, we show that structure learning has advantages in terms of accuracy and uncertainty estimation: "When compared with traditional fixed-kernel approaches, we show that structure learning has advantages in terms of accuracy and uncertainty estimation"
- [abstract] defines a prior distribution over kernel structures and hyperparameters, and efficiently samples the GP posterior distribution using a novel involutive Markov chain Monte Carlo procedure: "defines a prior distribution over kernel structures and hyperparameters, and efficiently samples the GP posterior distribution using a novel involutive M"
- [results] gallifrey was applied to common astronomical time series tasks, including stellar variability modelling, exoplanet transit modelling, and transmission spectroscopy: [evidence span withheld — anchor status: not_found]
- [readme] Employs SMC for robust and fast posterior approximations.: [evidence span withheld — anchor status: external_doc]
- [readme] Unlike most Gaussian Process packages, where a covariance function needs to be specified explicitly, gallifrey infers the covariance structure from the time series.: "Unlike most Gaussian Process packages, where a covariance function needs to be specified explicitly, $\texttt{gallifrey}$ infers the covariance structure"
