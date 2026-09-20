---
name: gaussian-prior-specification-cosmological-parameters
description: Use when running an MCMC posterior analysis of Big Bang Nucleosynthesis
  abundances where neutron lifetime and baryon density must be Gaussian-constrained
  and nuclear-rate uncertainties propagated log-normally.
when_to_use_negative:
- The analysis is not a Monte Carlo posterior study (e.g. a single fixed-parameter
  BBN abundance evaluation, for which runPRyM_julia.py suffices).
- Uncertainties on tau_n, Omega_B h^2, or nuclear rates follow non-Gaussian/non-log-normal
  shapes that would require custom likelihood definitions rather than the reported
  prior choices.
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: PRyMordial
  role: 'BBN code whose likelihood is sampled: evaluates light-element abundances
    given tau_n, Omega_B h^2, and the 63 log-normal nuclear-rate nuisance parameters,
    with nacreii_flag switching rate databases'
  repo: github.com/vallima/PRyMordial
- name: emcee
  role: Ensemble Monte Carlo sampler used to run the posterior twice, once per nacreii_flag
    setting
- name: PRIMAT
  role: Source of the key thermonuclear nuclear rates in the default (nacreii_flag
    off) configuration
- name: Python 3
  role: Language in which PRyMordial and the Monte Carlo workflow are written and
    run
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1140/epjc/s10052-024-12442-0
    title: 'PRyMordial: the first three minutes, within and beyond the standard model'
  source_content_hash: sha256:5268990fa0e410c5e0e4944b31a8ad5c60af6ecd204078c50cef15da596761dd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PRIMAT
  - PRyMordial
  - Python 3
  - emcee
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# Gaussian-prior specification of cosmological/nuisance parameters for BBN Monte Carlo

## Summary

Specifies the Gaussian priors on tau_n and Omega_B h^2 and log-normal priors on the 63 nuclear-rate nuisance parameters used to sample the BBN light-element abundance posterior in PRyMordial, enabling PRIMAT-vs-NACRE II uncertainty propagation.

## When to use

Use when running an MCMC posterior analysis of Big Bang Nucleosynthesis abundances where neutron lifetime and baryon density must be Gaussian-constrained and nuclear-rate uncertainties propagated log-normally.

## When NOT to use

- The analysis is not a Monte Carlo posterior study (e.g. a single fixed-parameter BBN abundance evaluation, for which runPRyM_julia.py suffices).
- Uncertainties on tau_n, Omega_B h^2, or nuclear rates follow non-Gaussian/non-log-normal shapes that would require custom likelihood definitions rather than the reported prior choices.

## Inputs

- Gaussian prior definitions for tau_n (878.4 +/- 0.5 s) and Omega_B h^2 (0.02230 +/- 0.00020)
- Log-normal nuisance-parameter specification for 63 thermonuclear nuclear rates
- Nuclear-rate database choice: PRIMAT rates or NACRE II rates (nacreii_flag on/off)
- PRyMordial BBN abundance computation (Python 3 package)

## Outputs

- emcee posterior sample chains over cosmological and nuclear-rate nuisance parameters for each nacreii_flag setting
- Marginalized 1D and 2D posterior distributions of key parameters and light-element abundances (Fig. 3 reproduction)
- Posterior-predicted deuterium abundance compared against the observed primordial deuterium value

## How to apply

Configure PRyMordial's BBN computation with PRIMAT thermonuclear rates and define Gaussian priors tau_n = 878.4 +/- 0.5 s and Omega_B h^2 = 0.02230 +/- 0.00020. Implement the likelihood so that the 63 nuclear-rate nuisance parameters are varied log-normally, propagating thermonuclear-rate uncertainties into the light-element abundance predictions. Run the emcee ensemble sampler twice, toggling nacreii_flag off (PRIMAT rates) and on (NACRE II rates), sampling over the two Gaussian-constrained parameters plus the nuisance parameters. Marginalize the chains to obtain 1D and 2D posteriors for each rate prescription, then compare the posterior-predicted deuterium abundance against the observed primordial value to test for the deuterium anomaly. The rationale is that PRyMordial is designed to 'render easy exploration of input parameter and thermonuclear rate uncertainties' while interfacing with Monte Carlo analysis libraries.

## Related tools

- **PRyMordial** (BBN code whose likelihood is sampled: evaluates light-element abundances given tau_n, Omega_B h^2, and the 63 log-normal nuclear-rate nuisance parameters, with nacreii_flag switching rate databases) — github.com/vallima/PRyMordial
- **emcee** (Ensemble Monte Carlo sampler used to run the posterior twice, once per nacreii_flag setting)
- **PRIMAT** (Source of the key thermonuclear nuclear rates in the default (nacreii_flag off) configuration)
- **Python 3** (Language in which PRyMordial and the Monte Carlo workflow are written and run)

## Examples

```
python runPRyM_julia.py
```

## Evaluation signals

- Prior centers/widths match the reported values exactly: tau_n = 878.4 +/- 0.5 s and Omega_B h^2 = 0.02230 +/- 0.00020, with 63 nuisance parameters varied log-normally.
- Marginalized posteriors reproduce the reported pattern: the deuterium anomaly appears with PRIMAT rates and is completely washed out when the NACRE II database is employed.
- The lithium problem remains statistically significant under both PRIMAT and NACRE II rate prescriptions.
- Posterior-predicted deuterium abundance is checked against the observed primordial deuterium value for each nacreii_flag setting.
- 1D and 2D marginalized distributions reproduce Fig. 3 of the paper for both rate-compilation runs.

## Limitations

- The deuterium anomaly is rate-compilation dependent: it appears for PRIMAT key nuclear rates but is completely washed out with NACRE II, so conclusions from the posterior are sensitive to the nacreii_flag setting.
- The lithium problem is statistically significant regardless of the nuclear-rate approach, so this prior/sampler setup cannot resolve it.
- PRyMordial's optional dependencies (Numba, Numdifftools, Vegas, PyJulia/diffeqpy with Julia and SciML) are recommended for the best possible usage; the code can avoid Numba/Numdifftools and does not use Vegas by default.

## Evidence

- [abstract] The code offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom, including non-instantaneous decoupling effects.: "The code offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom, including non-instantaneous decoupling effects."
- [intro] render easy exploration of input parameter and thermonuclear rate uncertainties, and adopt a user-friendly modern programming language interfacing with Monte Carlo analysis libraries.: [evidence span withheld — anchor status: not_found]
- [readme] A dedicated numerical table for standard cosmological analyses involving BBN likelihoods is also in the repo.: "A dedicated numerical table for standard cosmological analyses involving BBN likelihoods is also in the repo."
- [readme] The code can easily avoid dependencies on Numba and Numdifftools and it does not make use of Vegas by default.: "The code can easily avoid dependencies on Numba and Numdifftools and it does not make use of Vegas by default."
