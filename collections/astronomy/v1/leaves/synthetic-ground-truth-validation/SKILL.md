---
name: synthetic-ground-truth-validation
description: Use when a fitted model (e.g. a GP background model or transit parameter
  fit) must be validated against injected ground truth in a synthetic dataset such
  as the cond_transmission_spectroscopy example shipped with gallifrey.
when_to_use_negative:
- Real observational data with no injected ground truth — comparison against truth
  is impossible, so use cross-validation or physical consistency checks instead
- Datasets where the model's assumptions (e.g. a slowly varying background separable
  from the signal) do not hold, so posterior coverage of the injected values would
  not be meaningful
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: gallifrey
  role: Fits the GP model via Bayesian kernel structure learning with SMC (involutive
    MCMC) and supplies the particle posterior used to subtract the background and
    report posterior uncertainties for validation
  repo: github.com/ChrisBoettner/gallifrey
- name: JAX
  role: Backend used by gallifrey for efficient numerical computation of the GP posterior
    during the validation run
  repo: github.com/ChrisBoettner/gallifrey
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202554518
    title: 'gallifrey: JAX-based Gaussian process structure learning for astronomical
      time series'
  source_content_hash: sha256:26f1f14619af545898f54baf1bfb31fa7d18dc56ded31a30c8c7129002763af2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - JAX
  - gallifrey
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# synthetic-ground-truth-validation

## Summary

Validates a statistical inference pipeline (here, gallifrey's GP structure-learning-based background/transit separation) by running it on synthetic data with known injected parameters and checking that recovered quantities match ground truth within posterior uncertainties.

## When to use

Use when a fitted model (e.g. a GP background model or transit parameter fit) must be validated against injected ground truth in a synthetic dataset such as the cond_transmission_spectroscopy example shipped with gallifrey.

## When NOT to use

- Real observational data with no injected ground truth — comparison against truth is impossible, so use cross-validation or physical consistency checks instead
- Datasets where the model's assumptions (e.g. a slowly varying background separable from the signal) do not hold, so posterior coverage of the injected values would not be meaningful

## Inputs

- Synthetic dataset with injected ground-truth parameters (e.g. gallifrey's cond_transmission_spectroscopy example data: transit signal plus slowly varying stellar background per wavelength)
- Model posterior output (gallifrey GP particle posterior / learned GP mean over the background component)

## Outputs

- Comparison table of recovered vs. injected values per wavelength (e.g. transmission_spectrum.csv with recovered depth, injected ground truth, posterior uncertainty)
- Accept/reject validation decision based on posterior-uncertainty coverage of ground truth

## How to apply

Load the synthetic dataset containing a signal of interest (transit) plus a confounding component (slowly varying stellar background) with known injected parameters. Fit the model — for gallifrey, run Bayesian kernel structure learning with SMC and involutive MCMC moves to infer the GP covariance structure and a particle posterior over the background. Subtract the learned GP mean/particle-posterior background model from the observed light curve at each wavelength and extract the wavelength-dependent transit depth from each residual light curve to assemble the transmission spectrum. Write recovered and injected values to a common output (e.g. transmission_spectrum.csv) and accept the result only when recovered depths agree with the injected ground truth within the posterior uncertainties reported by gallifrey's particle posterior, alongside robust transit-parameter recovery.

## Related tools

- **gallifrey** (Fits the GP model via Bayesian kernel structure learning with SMC (involutive MCMC) and supplies the particle posterior used to subtract the background and report posterior uncertainties for validation) — github.com/ChrisBoettner/gallifrey
- **JAX** (Backend used by gallifrey for efficient numerical computation of the GP posterior during the validation run) — github.com/ChrisBoettner/gallifrey

## Examples

```
final_smc_state, history = gpmodel.fit_smc(smc_key, annealing_schedule=LinearSchedule().generate(len(xtrain), 10), n_mcmc=50, n_hmc=10)
```

## Evaluation signals

- Recovered wavelength-dependent transit depths agree with the injected ground truth within the posterior uncertainties reported by gallifrey's particle posterior
- Transit parameters are recovered with robust uncertainties, not just point estimates
- The separated background and transit components reproduce the known synthetic decomposition (effective background/transit separation)
- Recovered and injected values are written together (e.g. to transmission_spectrum.csv) so differences can be checked numerically
- SMC run is properly annealed (an annealing schedule, e.g. LinearSchedule, is used before extracting the final posterior)

## Limitations

- Validation is only as strong as the synthetic data generator — success on cond_transmission_spectroscopy-style data does not guarantee performance on real stellar light curves
- Scalability to larger datasets is an open algorithmic problem; the authors flag it as needed for future missions such as PLATO

## Evidence

- [other] gallifrey derives transmission spectra by effectively separating the background from the transit signal, alongside recovering transit parameters with robust uncertainties.: [evidence span withheld — anchor status: not_found]
- [abstract] defines a prior distribution over kernel structures and hyperparameters, and efficiently samples the GP posterior distribution using a novel involutive Markov chain Monte Carlo procedure: "defines a prior distribution over kernel structures and hyperparameters, and efficiently samples the GP posterior distribution using a novel involutive Markov chain Monte Carlo procedure"
- [abstract] We applied gallifrey to common astronomical time series tasks, including stellar variability modelling, exoplanet transit modelling, and transmission spectroscopy: "We applied gallifrey to common astronomical time series tasks, including stellar variability modelling, exoplanet transit modelling, and transmission spectroscopy"
- [readme] Generate an annealing schedule (important for SMC): "# Generate an annealing schedule (important for SMC)
annealing_schedule = LinearSchedule().generate(len(xtrain), 10)"
