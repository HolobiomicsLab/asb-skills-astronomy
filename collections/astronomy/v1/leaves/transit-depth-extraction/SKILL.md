---
name: transit-depth-extraction
description: Use when you have transit light curves (e.g. synthetic transmission-spectroscopy
  data like the cond_transmission_spectroscopy example) where a slowly varying stellar
  background contaminates the transit signal and you need per-wavelength transit depths
  with posterior uncertainties.
when_to_use_negative:
- The task is plain GP regression where the covariance function is already known and
  can be specified explicitly — structure learning is aimed at inferring covariance
  structure from the series itself
- The dataset is large enough that SMC scalability becomes a bottleneck (scalability
  to larger datasets, e.g. for PLATO, is explicitly flagged as future work)
- The input has no transit component or no wavelength dependence, so no transmission
  spectrum can be assembled from per-wavelength residuals
tools:
- name: gallifrey
  role: 'JAX-based GP structure-learning package: SMC sampling of the GP background
    posterior with involutive MCMC moves, background subtraction, and predictive distribution
    for transit-depth extraction'
  repo: https://github.com/ChrisBoettner/gallifrey
- name: JAX
  role: Numerical backend gallifrey uses for efficient GP computation and parallel
    SMC particle evaluation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202554518
    title: 'gallifrey: JAX-based Gaussian process structure learning for astronomical
      time series'
  source_content_hash: sha256:fa613e356a4a70602764dac4202981efeb694e138a978ad298d77ba6e5e090f7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChrisBoettner/gallifrey
  tools:
  - JAX
  - gallifrey
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# transit-depth-extraction

## Summary

Extracts a wavelength-dependent transmission spectrum by using gallifrey's Bayesian GP structure learning (SMC with involutive MCMC) to model and subtract the slowly varying stellar background from transit light curves, then reading off the transit depth per wavelength.

## When to use

Use when you have transit light curves (e.g. synthetic transmission-spectroscopy data like the cond_transmission_spectroscopy example) where a slowly varying stellar background contaminates the transit signal and you need per-wavelength transit depths with posterior uncertainties.

## When NOT to use

- The task is plain GP regression where the covariance function is already known and can be specified explicitly — structure learning is aimed at inferring covariance structure from the series itself
- The dataset is large enough that SMC scalability becomes a bottleneck (scalability to larger datasets, e.g. for PLATO, is explicitly flagged as future work)
- The input has no transit component or no wavelength dependence, so no transmission spectrum can be assembled from per-wavelength residuals

## Inputs

- Wavelength-resolved transit light curves (e.g. the cond_transmission_spectroscopy example data shipped with gallifrey) containing transit signal plus slowly varying stellar background
- Time/flux arrays (x, y) suitable for GPModel initialisation in gallifrey

## Outputs

- Wavelength-dependent transit depths with posterior uncertainties from gallifrey's particle posterior
- transmission_spectrum.csv containing recovered depths alongside injected ground-truth values

## How to apply

Load the transmission-spectroscopy light curve (transit signal plus slowly varying stellar background) and initialise a gallifrey GPModel over it, choosing num_particles for the SMC posterior. Run Bayesian kernel structure learning with gpmodel.fit_smc, supplying an annealing schedule (e.g. LinearSchedule().generate(len(xtrain), 10)) and setting n_mcmc and n_hmc steps; gallifrey places a prior over kernel structures and hyperparameters and samples the GP posterior via involutive MCMC rather than requiring a fixed kernel. Update the model with the final SMC state (gpmodel.update_state(final_smc_state)) and obtain the background posterior via get_mixture_distribution, whose predictive mean/std carry the particle-posterior uncertainties. Subtract the learned GP background from the observed light curve at each wavelength to isolate the transit, then extract the wavelength-dependent transit depth from each residual light curve to assemble the transmission spectrum. Validate by checking that recovered depths agree with the injected/known ground truth within the posterior uncertainties reported by the particle posterior; write depths and ground truth to transmission_spectrum.csv.

## Related tools

- **gallifrey** (JAX-based GP structure-learning package: SMC sampling of the GP background posterior with involutive MCMC moves, background subtraction, and predictive distribution for transit-depth extraction) — https://github.com/ChrisBoettner/gallifrey
- **JAX** (Numerical backend gallifrey uses for efficient GP computation and parallel SMC particle evaluation)

## Examples

```
from gallifrey.model import GPConfig, GPModel; from gallifrey.schedule import LinearSchedule; gpmodel = GPModel(key, x=xtrain, y=ytrain, num_particles=8, config=GPConfig()); final_smc_state, history = gpmodel.fit_smc(smc_key, annealing_schedule=LinearSchedule().generate(len(xtrain), 10), n_mcmc=50, n_hmc=10)
```

## Evaluation signals

- Recovered wavelength-dependent transit depths agree with the injected ground truth within the posterior uncertainties from gallifrey's particle posterior
- SMC run terminates with a completed annealing schedule and a final_smc_state that is passed through gpmodel.update_state before prediction
- Predictive distribution from get_mixture_distribution yields finite predictive mean and stddev over the background model (no NaNs from the GP fit)
- transmission_spectrum.csv contains both recovered depths and ground-truth values for every wavelength
- Structure-learning result should outperform a fixed-kernel GP baseline in accuracy and uncertainty estimation, as the paper reports for structure learning

## Limitations

- Scalability to larger datasets is identified as future work; the paper discusses 'a road map for algorithmic improvements in terms of scalability to larger datasets' for missions such as PLATO
- gallifrey requires Python 3.10+ with pinned JAX 0.5.x, blackjax 1.2.x, flax, equinox, and tensorflow-probability dependencies
- Validation relies on synthetic/known ground truth; for real data there is no injected truth to compare depths against

## Evidence

- [abstract] applied to transmission spectroscopy: "We applied gallifrey to common astronomical time series tasks, including stellar variability modelling, exoplanet transit modelling, and transmission spectroscopy"
- [abstract] structure learning outperforms fixed kernels: "When compared with traditional fixed-kernel approaches, we show that structure learning has advantages in terms of accuracy and uncertainty estimation"
- [abstract] SMC with involutive MCMC: "defines a prior distribution over kernel structures and hyperparameters, and efficiently samples the GP posterior distribution using a novel involutive Markov chain Monte Carlo procedure"
- [intro] no explicit kernel needed: [evidence span withheld — anchor status: not_found]
- [readme] fit_smc usage: "final_smc_state, history = gpmodel.fit_smc(
    smc_key,
    annealing_schedule=annealing_schedule,
    n_mcmc=50,      # Number of MCMC steps per SMC iteration
    n_hmc=10,       # Number of HMC"
