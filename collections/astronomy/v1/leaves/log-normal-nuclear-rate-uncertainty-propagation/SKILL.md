---
name: log-normal-nuclear-rate-uncertainty-propagation
description: Use when running BBN Monte Carlo analyses with PRyMordial and you need
  to propagate nuclear thermonuclear-rate uncertainties into light-element abundance
  posteriors, or compare PRIMAT vs NACRE II rate compilations.
when_to_use_negative:
- When nuclear-rate uncertainties are not the dominant systematic — e.g. you only
  need a single fixed-rate BBN abundance prediction, not an uncertainty propagation
  analysis
- When the analysis targets BSM physics during BBN rather than SM nuclear-rate systematics
  (PRyMordial supports both, but this skill addresses the SM Monte Carlo workflow)
- When no rate-compilation comparison is needed and the default rate database is already
  validated for the question at hand
edam_operation: http://edamontology.org/operation_3659
edam_topics: []
tools:
- name: PRyMordial
  role: Core BBN computation engine providing fast, precise light-element abundance
    evaluations and the nacreii_flag switch for rate compilations
  repo: https://github.com/vallima/PRyMordial
- name: emcee
  role: Ensemble Monte Carlo sampler used to explore the posterior over tau_n, Omega_B
    h^2, and the 63 log-normal nuclear-rate nuisance parameters
- name: PRIMAT
  role: Source of the default thermonuclear-rate compilation used when nacreii_flag
    is off
- name: Python 3
  role: Language in which PRyMordial is written and in which the Monte Carlo analysis
    is implemented
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1140/epjc/s10052-024-12442-0
    title: 'PRyMordial: the first three minutes, within and beyond the standard model'
  source_content_hash: sha256:b1a9692de9f266acd8a2cfda2792280527e81b6ef2bdc27de38fedcfd026599e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/vallima/PRyMordial
  tools:
  - PRIMAT
  - PRyMordial
  - Python 3
  - emcee
  edam_operation: http://edamontology.org/operation_3659
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# log-normal-nuclear-rate-uncertainty-propagation

## Summary

Propagate thermonuclear-rate uncertainties in BBN abundance predictions by modeling 63 key nuclear-rate nuisance parameters as log-normal variables within PRyMordial's Monte Carlo framework, testing sensitivity to rate-compilation choices (PRIMAT vs NACRE II).

## When to use

Use when running BBN Monte Carlo analyses with PRyMordial and you need to propagate nuclear thermonuclear-rate uncertainties into light-element abundance posteriors, or compare PRIMAT vs NACRE II rate compilations.

## When NOT to use

- When nuclear-rate uncertainties are not the dominant systematic — e.g. you only need a single fixed-rate BBN abundance prediction, not an uncertainty propagation analysis
- When the analysis targets BSM physics during BBN rather than SM nuclear-rate systematics (PRyMordial supports both, but this skill addresses the SM Monte Carlo workflow)
- When no rate-compilation comparison is needed and the default rate database is already validated for the question at hand

## Inputs

- PRyMordial BBN configuration with PRIMAT thermonuclear rates
- Gaussian priors: tau_n = 878.4 +/- 0.5 s and Omega_B h^2 = 0.02230 +/- 0.00020
- 63 log-normal nuclear-rate nuisance parameter definitions
- nacreii_flag setting (off = PRIMAT rates, on = NACRE II rates)
- Observed primordial light-element abundance values (e.g. deuterium)

## Outputs

- emcee MCMC chains sampled over cosmological, nuclear, and nuisance parameters
- Marginalized 1D and 2D posterior distributions of light-element abundances (Fig. 3-style)
- Posterior-predicted deuterium abundance for comparison against observation
- Diagnosis of deuterium anomaly presence/absence per rate-compilation choice

## How to apply

Configure the PRyMordial BBN computation with PRIMAT thermonuclear rates and define Gaussian priors on tau_n = 878.4 +/- 0.5 s and Omega_B h^2 = 0.02230 +/- 0.00020. Implement the likelihood so that the 63 key nuclear-rate nuisance parameters are varied log-normally, propagating rate uncertainties into the predicted light-element abundances. Set up the emcee ensemble sampler and run it twice — once with nacreii_flag off (PRIMAT rates) and once with nacreii_flag on (NACRE II rates) — sampling the posterior over the two Gaussian-constrained cosmological/nuclear parameters plus the nuisance parameters. Marginalize the chains to obtain 1D and 2D posterior distributions of the abundance predictions, then compare the posterior-predicted deuterium abundance against the observed primordial value. The rationale is that log-normal parameterization respects the positivity of reaction rates while Gaussian priors cover the well-measured neutron lifetime and baryon density; flipping the nacreii_flag isolates the impact of the rate-compilation choice on abundance anomalies.

## Related tools

- **PRyMordial** (Core BBN computation engine providing fast, precise light-element abundance evaluations and the nacreii_flag switch for rate compilations) — https://github.com/vallima/PRyMordial
- **emcee** (Ensemble Monte Carlo sampler used to explore the posterior over tau_n, Omega_B h^2, and the 63 log-normal nuclear-rate nuisance parameters)
- **PRIMAT** (Source of the default thermonuclear-rate compilation used when nacreii_flag is off)
- **Python 3** (Language in which PRyMordial is written and in which the Monte Carlo analysis is implemented)

## Examples

```
python runPRyM_julia.py
```

## Evaluation signals

- The marginalized posteriors reproduce the reported pattern: a deuterium anomaly arises with PRIMAT rates and disappears when NACRE II rates are used via nacreii_flag
- The lithium problem remains statistically significant under both the PRIMAT and NACRE II rate prescriptions
- All 63 nuclear-rate nuisance parameters are sampled log-normally alongside tau_n and Omega_B h^2 with the specified Gaussian priors
- Two independent emcee runs are completed (nacreii_flag off and on) and chains are marginalized to 1D and 2D posterior distributions
- Posterior-predicted deuterium abundance is compared against the observed primordial deuterium value for each rate prescription

## Limitations

- The result is rate-compilation dependent: the deuterium anomaly appears with PRIMAT rates but is completely washed out with NACRE II rates, so conclusions are sensitive to the database choice
- The lithium problem is statistically significant regardless of nuclear-rate approach, so this skill does not resolve that discrepancy
- Optional performance dependencies (Numba, Numdifftools, Vegas, PyJulia/diffeqpy) may need installation for best package usage; the code can avoid Numba and Numdifftools but does not use Vegas by default

## Evidence

- [results] SM Monte Carlo analysis with PRyMordial and 63 log-normal nuisance parameters: [evidence span withheld — anchor status: not_found]
- [results] deuterium anomaly washed out with NACRE II: [evidence span withheld — anchor status: not_found]
- [methods] Gaussian priors on tau_n and Omega_B h^2: [evidence span withheld — anchor status: not_found]
- [abstract] PRyMordial fast and precise BBN abundance evaluation: "The code offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom"
- [intro] PRyMordial written in Python 3: "The package is written and runs entirely with Python 3"
