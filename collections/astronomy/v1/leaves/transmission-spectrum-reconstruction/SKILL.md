---
name: transmission-spectrum-reconstruction
description: Use when multi-wavelength transit light curves contain a slowly varying
  stellar background that must be marginalized out before per-wavelength transit depths
  can be assembled into a transmission spectrum.
when_to_use_negative:
- The covariance function is already known and can be specified explicitly — gallifrey's
  structure learning is for inferring covariance structure from the time series itself
- 'The dataset is very large: the paper flags scalability to larger datasets (e.g.
  PLATO-scale) as future work'
- The data are not time series (gallifrey is focused on time series modelling, analysis,
  and forecasting)
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: gallifrey
  role: JAX-based GP Bayesian kernel structure learning package; runs SMC with involutive
    MCMC to infer background covariance structure and produce the particle posterior
    used for background subtraction and depth uncertainty estimation
  repo: https://github.com/ChrisBoettner/gallifrey
- name: JAX
  role: Numerical backend used by gallifrey for efficient GP computation; configure
    device count for parallel SMC particles
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202554518
    title: 'gallifrey: JAX-based Gaussian process structure learning for astronomical
      time series'
  source_content_hash: sha256:bbf8d8ce4842c138614ec255b33b682d6448d76bc4fc64cf62d75f1be294cc42
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChrisBoettner/gallifrey
  tools:
  - JAX
  - gallifrey
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# transmission-spectrum-reconstruction

## Summary

Reconstructs a wavelength-dependent transmission spectrum from transit spectroscopy light curves by using gallifrey's Bayesian GP structure learning (SMC with involutive MCMC) to separate the slowly varying stellar background from the transit signal at each wavelength, then extracting transit depths against injected ground truth.

## When to use

Use when multi-wavelength transit light curves contain a slowly varying stellar background that must be marginalized out before per-wavelength transit depths can be assembled into a transmission spectrum.

## When NOT to use

- The covariance function is already known and can be specified explicitly — gallifrey's structure learning is for inferring covariance structure from the time series itself
- The dataset is very large: the paper flags scalability to larger datasets (e.g. PLATO-scale) as future work
- The data are not time series (gallifrey is focused on time series modelling, analysis, and forecasting)

## Inputs

- Multi-wavelength transit light curve dataset (cond_transmission_spectroscopy example data shipped with gallifrey) containing transit signal plus slowly varying stellar background
- Injected ground-truth wavelength-dependent transit depths (for validation)

## Outputs

- Per-wavelength background-subtracted (transit-only) light curves
- Transmission spectrum: wavelength-dependent transit depths with posterior uncertainties from the gallifrey particle posterior
- transmission_spectrum.csv containing recovered depths alongside injected ground truth

## How to apply

Load the synthetic transmission spectroscopy dataset (the cond_transmission_spectroscopy example data shipped with gallifrey) containing the transit signal plus slowly varying stellar background at each wavelength. Initialize a GPModel with an SMC particle ensemble and run gallifrey's Bayesian kernel structure learning using a LinearSchedule annealing schedule with fit_smc, which samples the GP posterior with involutive MCMC moves (n_mcmc steps with n_hmc HMC steps each) to infer the covariance structure and a particle posterior over the background component. Use the learned GP mixture mean/posterior to subtract the stellar background model from the observed light curve at each wavelength, isolating the transit signal. Extract the wavelength-dependent transit depth from each residual light curve, recovering transit parameters with robust uncertainties from the particle posterior. Assemble the depths into a transmission spectrum and compare against injected ground truth.

## Related tools

- **gallifrey** (JAX-based GP Bayesian kernel structure learning package; runs SMC with involutive MCMC to infer background covariance structure and produce the particle posterior used for background subtraction and depth uncertainty estimation) — https://github.com/ChrisBoettner/gallifrey
- **JAX** (Numerical backend used by gallifrey for efficient GP computation; configure device count for parallel SMC particles)

## Examples

```
pip install gallifrey; from gallifrey.model import GPConfig, GPModel; from gallifrey.schedule import LinearSchedule; gpmodel = GPModel(key, x=xtrain, y=ytrain, num_particles=8, config=GPConfig()); final_smc_state, history = gpmodel.fit_smc(smc_key, annealing_schedule=LinearSchedule().generate(len(xtrain), 10), n_mcmc=50, n_hmc=10)
```

## Evaluation signals

- Recovered wavelength-dependent transit depths agree with the injected ground truth within the posterior uncertainties reported by gallifrey's particle posterior
- Residual light curves after background subtraction show the transit signal isolated (no remaining slowly varying stellar trend)
- SMC run completes with a valid LinearSchedule annealing schedule over the training data length and converges to a final_smc_state
- transmission_spectrum.csv is produced with both recovered depths and injected ground-truth columns
- Transit parameters are recovered with robust (particle-posterior) uncertainties rather than point estimates only

## Limitations

- Scalability to larger datasets is identified as future work needed for missions such as PLATO
- Validated on the synthetic cond_transmission_spectroscopy example data; application to real multi-wavelength data is not demonstrated in the provided context
- Requires Python >= 3.10 and pinned JAX ecosystem versions (jax >=0.5.0,<0.6.0, blackjax >=1.2.5,<2.0.0, etc.)

## Evidence

- [abstract] applied to transmission spectroscopy for background/transit separation: "We applied gallifrey to common astronomical time series tasks, including stellar variability modelling, exoplanet transit modelling, and transmission spectroscopy"
- [intro] gallifrey infers covariance structure from the time series: [evidence span withheld — anchor status: not_found]
- [abstract] prior over kernel structures and involutive MCMC sampling: "defines a prior distribution over kernel structures and hyperparameters, and efficiently samples the GP posterior distribution using a novel involutive Markov chain Monte Carlo procedure"
- [intro] gallifrey was built for exoplanet transit light curves: "gallifrey was created with exoplanet transit light curves in mind, but is applicable to a wide variety of time series modelling, analysis, and forecasting tasks."
- [readme] README quick-start fit_smc invocation with LinearSchedule: "final_smc_state, history = gpmodel.fit_smc(
    smc_key,
    annealing_schedule=annealing_schedule,
    n_mcmc=50,      # Number of MCMC steps per SMC iteration
    n_hmc=10"
- [abstract] scalability limitation: "We discuss a road map for algorithmic improvements in terms of scalability to larger datasets"
