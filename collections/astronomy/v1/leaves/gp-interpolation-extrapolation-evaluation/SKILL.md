---
name: gp-interpolation-extrapolation-evaluation
description: Use when a time series (e.g. stellar variability) must be modelled with
  a GP whose covariance structure is learned rather than fixed, and you must verify
  predictive accuracy and uncertainty coverage for both held-out interpolation points
  and times beyond the observed window.
when_to_use_negative:
- The covariance function is already known and an explicitly specified fixed kernel
  is sufficient — gallifrey is designed for cases where the structure must be inferred
  from the time series
- The dataset is large enough to hit the scalability limits of the SMC algorithm,
  which the paper identifies as future work needed for missions such as PLATO
- The analysis goal is exoplanet transit parameter inference or background/transit
  separation rather than evaluating interpolation/extrapolation of variability
edam_topics:
- http://edamontology.org/topic_3473
tools:
- name: gallifrey
  role: Python/JAX package performing Bayesian GP structure learning with SMC, involutive
    MCMC rejuvenation, and predictive mixture distributions for interpolation/extrapolation
  repo: github.com/ChrisBoettner/gallifrey
- name: JAX
  role: Numerical backend enabling efficient GP computation and parallelized SMC posterior
    approximation
- name: Python
  role: Implementation language (requires Python 3.10+) in which the evaluation workflow
    and metric export are scripted
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202554518
    title: 'gallifrey: JAX-based Gaussian process structure learning for astronomical
      time series'
  source_content_hash: sha256:1db758270ee11033657e3cd94579339071f7a47ce10ecc611c2ed226841da56b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - JAX
  - Python
  - gallifrey
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# gp-interpolation-extrapolation-evaluation

## Summary

Evaluates whether a learned-kernel GP (via gallifrey's SMC structure learning) accurately interpolates within an observed time-series window and extrapolates beyond it, quantifying predictive RMSE and credible-interval coverage — as demonstrated on stellar variability modelling.

## When to use

Use when a time series (e.g. stellar variability) must be modelled with a GP whose covariance structure is learned rather than fixed, and you must verify predictive accuracy and uncertainty coverage for both held-out interpolation points and times beyond the observed window.

## When NOT to use

- The covariance function is already known and an explicitly specified fixed kernel is sufficient — gallifrey is designed for cases where the structure must be inferred from the time series
- The dataset is large enough to hit the scalability limits of the SMC algorithm, which the paper identifies as future work needed for missions such as PLATO
- The analysis goal is exoplanet transit parameter inference or background/transit separation rather than evaluating interpolation/extrapolation of variability

## Inputs

- Stellar-variability time series (synthetic quasi-periodic/stochastic data, e.g. cond_stellar_variability example) as x/y arrays
- Observed training window (xtrain, ytrain) and held-out test points for interpolation
- Times beyond the observed window for extrapolation (e.g. gpmodel.x_transform(jnp.linspace(0, 18, 60)))
- GPConfig kernel prior over kernel structures and hyperparameters (periodic and stochastic components)

## Outputs

- Predictive mixture distribution at held-out and extrapolation times (predictive mean and standard deviation with uncertainties)
- RMSE against held-out truth for interpolation and extrapolation regimes
- Credible-interval coverage for interpolation and extrapolation regimes
- stellar_variability_metrics.csv with error and coverage fields populated for both regimes

## How to apply

Load the stellar-variability time series (e.g. the synthetic quasi-periodic/stochastic data from the cond_stellar_variability example) and split it into an observed training window and held-out test points. In gallifrey, configure a kernel prior over kernel structures and hyperparameters including periodic and stochastic components suited to stellar variability, then run the SMC structure-learning loop: build an annealing schedule (e.g. LinearSchedule().generate(len(xtrain), 10)) and call fit_smc with n_mcmc and n_hmc steps per SMC iteration, which performs particle resampling and involutive MCMC rejuvenation moves. Update the model with the final SMC state and obtain the predictive mixture distribution via get_mixture_distribution at both interpolation points and times beyond the observed window, extracting predictive_mean and predictive_std. Compute RMSE against the held-out truth and coverage of the credible intervals for both regimes, and compare against a fixed-kernel baseline — the paper shows structure learning has advantages in accuracy and uncertainty estimation over traditional fixed-kernel approaches. Save error and coverage metrics for interpolation and extrapolation to stellar_variability_metrics.csv.

## Related tools

- **gallifrey** (Python/JAX package performing Bayesian GP structure learning with SMC, involutive MCMC rejuvenation, and predictive mixture distributions for interpolation/extrapolation) — github.com/ChrisBoettner/gallifrey
- **JAX** (Numerical backend enabling efficient GP computation and parallelized SMC posterior approximation)
- **Python** (Implementation language (requires Python 3.10+) in which the evaluation workflow and metric export are scripted)

## Examples

```
from gallifrey.model import GPConfig, GPModel; gpmodel = GPModel(model_key, x=xtrain, y=ytrain, num_particles=8, config=GPConfig())
```

## Evaluation signals

- stellar_variability_metrics.csv exists and contains populated error and coverage fields for both interpolation and extrapolation regimes
- Learned-kernel RMSE against the held-out truth is lower than the fixed-kernel baseline (EnrichedIndex finding on structure-learning advantages)
- Credible-interval coverage is close to the nominal level, reflecting the claimed advantage in uncertainty estimation over fixed kernels
- The learned kernel mixture reproduces the paper's qualitative finding of accurate interpolation and extrapolation of stellar variability
- The SMC fit completes with an annealing schedule and the final state is applied via gpmodel.update_state(final_smc_state) before prediction

## Limitations

- Scalability to larger datasets is an open limitation; the paper discusses a road map for algorithmic improvements needed for future missions such as PLATO
- Requires Python 3.10+ and pinned dependency versions (jax >=0.5.0,<0.6.0; blackjax >=1.2.5,<2.0.0), so environment drift can break the workflow
- SMC results depend on particle count and the annealing schedule; an inappropriate LinearSchedule or too few particles/MCMC steps can degrade posterior approximation and predictive coverage

## Evidence

- [abstract] stellar variability interpolation/extrapolation finding: "We applied gallifrey to common astronomical time series tasks, including stellar variability modelling, exoplanet transit modelling, and transmission spectroscopy"
- [abstract] structure learning vs fixed-kernel advantage: "When compared with traditional fixed-kernel approaches, we show that structure learning has advantages in terms of accuracy and uncertainty estimation"
- [abstract] involutive MCMC within SMC posterior sampling: "defines a prior distribution over kernel structures and hyperparameters, and efficiently samples the GP posterior distribution using a novel involutive Markov chain Monte Carlo procedure"
- [readme] SMC + JAX computational basis (README): "utilizes JAX for efficient numerical computation and Sequential Monte Carlo (SMC) methods for robust posterior approximation."
- [readme] kernel inferred rather than specified (README): [evidence span withheld — anchor status: external_doc]
