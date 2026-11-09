---
name: stellar-background-transit-separation
description: Use when input is transmission-spectroscopy light curve data containing
  a transit signal superposed on a slowly varying stellar background, and the goal
  is to isolate the transit at each wavelength and derive a transmission spectrum.
when_to_use_negative:
- The covariance structure of the GP is already known and can be specified explicitly
  — gallifrey exists to infer covariance structure from the time series itself
- The task is plain GP regression with a fixed kernel rather than structure discovery
  / background-transit separation (structure learning may be unnecessary overhead)
- The dataset is very large, since scalability to larger datasets is identified as
  future work needed for missions such as PLATO
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: gallifrey
  role: JAX-based Bayesian GP structure-learning package; its GPModel + SMC sampler
    (involutive MCMC) infers the background GP posterior that is subtracted to isolate
    the transit signal
  repo: github.com/ChrisBoettner/gallifrey
- name: JAX
  role: numerical backend gallifrey uses for efficient GP computation and SMC posterior
    approximation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202554518
    title: 'gallifrey: JAX-based Gaussian process structure learning for astronomical
      time series'
  source_content_hash: sha256:d346032eadf24e5fe12707941efbca1a3f8ed899adbd3f5f49efb8f1d6217efc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - JAX
  - gallifrey
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# stellar-background-transit-separation

## Summary

Separates a slowly varying stellar background from the transit signal in wavelength-dependent light curves using gallifrey's Bayesian GP kernel structure learning with SMC, then extracts the wavelength-dependent transit depths to assemble a transmission spectrum.

## When to use

Use when input is transmission-spectroscopy light curve data containing a transit signal superposed on a slowly varying stellar background, and the goal is to isolate the transit at each wavelength and derive a transmission spectrum.

## When NOT to use

- The covariance structure of the GP is already known and can be specified explicitly — gallifrey exists to infer covariance structure from the time series itself
- The task is plain GP regression with a fixed kernel rather than structure discovery / background-transit separation (structure learning may be unnecessary overhead)
- The dataset is very large, since scalability to larger datasets is identified as future work needed for missions such as PLATO

## Inputs

- wavelength-dependent transit light curves (transit signal + slowly varying stellar background), e.g. the cond_transmission_spectroscopy example dataset shipped with gallifrey
- GPConfig specifying kernel/prior configuration for gallifrey's GPModel

## Outputs

- background-subtracted (transit-isolated) light curve at each wavelength
- wavelength-dependent transit depths assembled into a transmission spectrum (e.g. transmission_spectrum.csv)
- posterior uncertainties on recovered transit parameters from the SMC particle posterior

## How to apply

Load the synthetic transmission spectroscopy dataset (the cond_transmission_spectroscopy example data shipped with gallifrey) containing the transit signal plus slowly varying stellar background. Initialize a GPModel with SMC particles and run gallifrey's Bayesian kernel structure learning using the sequential Monte Carlo sampler with involutive MCMC moves, driven by an annealing schedule (e.g. LinearSchedule). Use the learned GP mixture posterior (predictive mean and per-particle uncertainty) to subtract the stellar background model from the observed light curve at each wavelength, isolating the transit signal. Extract the wavelength-dependent transit depth from each residual light curve and assemble the depths into a transmission spectrum (e.g. transmission_spectrum.csv). Validate by checking that recovered depths agree with injected ground truth within the posterior uncertainties from gallifrey's particle posterior.

## Related tools

- **gallifrey** (JAX-based Bayesian GP structure-learning package; its GPModel + SMC sampler (involutive MCMC) infers the background GP posterior that is subtracted to isolate the transit signal) — github.com/ChrisBoettner/gallifrey
- **JAX** (numerical backend gallifrey uses for efficient GP computation and SMC posterior approximation)

## Examples

```
gpmodel = GPModel(model_key, x=xtrain, y=ytrain, num_particles=8, config=GPConfig()); final_smc_state, history = gpmodel.fit_smc(smc_key, annealing_schedule=LinearSchedule().generate(len(xtrain), 10), n_mcmc=50, n_hmc=10)
```

## Evaluation signals

- Recovered wavelength-dependent transit depths agree with the injected ground truth within the posterior uncertainties reported by gallifrey's particle posterior
- The learned GP mixture posterior reproduces the slowly varying stellar background while the residual light curve retains the transit signal
- SMC run uses a valid annealing schedule (e.g. LinearSchedule generated over the training set) and converges with the configured number of particles and MCMC/HMC steps
- Predictive distribution (gpmodel.get_mixture_distribution) yields finite mean and stddev over the test wavelength grid
- Output transmission_spectrum.csv contains depth values at every input wavelength with associated uncertainties

## Limitations

- Scalability to larger datasets is identified as future work needed for missions such as PLATO
- Posterior quality depends on SMC settings (num_particles, n_mcmc, n_hmc, annealing schedule); the README notes the annealing schedule is 'important for SMC'
- Validation in the source task is against synthetic/injected ground truth (cond_transmission_spectroscopy example data), so results are only as good as the learned background model on real data

## Evidence

- [abstract] background/transit separation via SMC structure learning: "We applied gallifrey to common astronomical time series tasks, including stellar variability modelling, exoplanet transit modelling, and transmission spectroscopy"
- [abstract] SMC posterior with involutive MCMC: "defines a prior distribution over kernel structures and hyperparameters, and efficiently samples the GP posterior distribution using a novel involutive Markov chain Monte Carlo procedure"
- [intro] gallifrey infers covariance structure rather than requiring explicit specification: [evidence span withheld — anchor status: not_found]
- [readme] annealing schedule is required for SMC fitting: "# Generate an annealing schedule (important for SMC)
annealing_schedule = LinearSchedule().generate(len(xtrain), 10)"
- [abstract] scalability limitation: "We discuss a road map for algorithmic improvements in terms of scalability to larger datasets"
