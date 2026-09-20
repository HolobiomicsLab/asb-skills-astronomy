---
name: bbn-monte-carlo-parameter-inference
description: Use when the analysis goal is marginalizing BBN abundance posteriors
  over neutron lifetime, baryon density, and nuclear-rate uncertainties — e.g. comparing
  PRIMAT vs NACRE II rate compilations for the deuterium or lithium problems.
when_to_use_negative:
- The goal is a single deterministic BBN abundance computation at fixed (tau_n, Omega_B
  h^2) rather than uncertainty propagation — run PRyMordial's demo notebooks directly
  instead of the Monte Carlo
- The analysis targets BSM particle physics active during BBN (e.g. extra relativistic
  degrees of freedom) rather than SM nuclear-rate and cosmological-parameter uncertainty
  marginalization
- Only one rate compilation (PRIMAT or NACRE II) is of interest and no cross-compilation
  comparison of the deuterium anomaly is needed
edam_operation: http://edamontology.org/operation_3659
tools:
- name: PRyMordial
  role: Core BBN code computing light-element abundances and N_eff; provides the nacreii_flag
    switch between PRIMAT and NACRE II rates and the likelihood interface for the
    Monte Carlo
  repo: github.com/vallima/PRyMordial
- name: emcee
  role: Ensemble Monte Carlo sampler used to explore the posterior over tau_n, Omega_B
    h^2, and the 63 nuclear-rate nuisance parameters
- name: PRIMAT
  role: Source of the ab-initio thermonuclear nuclear-rate set used as the baseline
    (nacreii_flag off)
- name: Python 3
  role: Runtime language in which PRyMordial is written and runs entirely, interfacing
    with the Monte Carlo sampling libraries
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1140/epjc/s10052-024-12442-0
    title: 'PRyMordial: the first three minutes, within and beyond the standard model'
  source_content_hash: sha256:86ff0af816ce3701084676a1096738883e3de4c47ca04939475bc4c1b8b8671d
schema_version: 0.2.0
metadata:
  license_tier: open
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
yes_rate: 0.9
n_claims: 10
---

# bbn-monte-carlo-parameter-inference

## Summary

Runs ensemble Monte Carlo posterior inference over Big Bang Nucleosynthesis (BBN) light-element abundances using PRyMordial: Gaussian priors on tau_n and Omega_B h^2 are combined with 63 log-normal nuclear-rate nuisance parameters to propagate thermonuclear-rate uncertainties into primordial abundance predictions.

## When to use

Use when the analysis goal is marginalizing BBN abundance posteriors over neutron lifetime, baryon density, and nuclear-rate uncertainties — e.g. comparing PRIMAT vs NACRE II rate compilations for the deuterium or lithium problems.

## When NOT to use

- The goal is a single deterministic BBN abundance computation at fixed (tau_n, Omega_B h^2) rather than uncertainty propagation — run PRyMordial's demo notebooks directly instead of the Monte Carlo
- The analysis targets BSM particle physics active during BBN (e.g. extra relativistic degrees of freedom) rather than SM nuclear-rate and cosmological-parameter uncertainty marginalization
- Only one rate compilation (PRIMAT or NACRE II) is of interest and no cross-compilation comparison of the deuterium anomaly is needed

## Inputs

- Gaussian priors on tau_n = 878.4 +/- 0.5 s and Omega_B h^2 = 0.02230 +/- 0.00020
- 63 log-normal nuclear-rate nuisance parameters (PRIMAT or NACRE II rate compilation selected via nacreii_flag)
- Observed primordial deuterium abundance value for posterior comparison

## Outputs

- Marginalized 1D and 2D posterior distributions of key parameters for each nacreii_flag setting
- Posterior-predicted primordial light-element abundances (deuterium, helium-4, lithium) per rate compilation
- Verdict on the deuterium anomaly: present with PRIMAT rates, absent with NACRE II rates

## How to apply

Configure the PRyMordial BBN computation with PRIMAT thermonuclear rates and define Gaussian priors on tau_n = 878.4 +/- 0.5 s and Omega_B h^2 = 0.02230 +/- 0.00020. Implement the likelihood so the 63 nuclear-rate nuisance parameters are varied log-normally, propagating thermonuclear-rate uncertainties into the light-element abundance predictions. Set up the emcee ensemble sampler and run it twice — once with nacreii_flag off (PRIMAT rates) and once with nacreii_flag on (NACRE II rates) — sampling the posterior over the two Gaussian-constrained cosmological/nuclear parameters plus the nuisance parameters. Marginalize the sampled chains to obtain 1D and 2D posterior distributions for each nacreii_flag setting, then compare the posterior-predicted deuterium abundance against the observed primordial deuterium value.

## Related tools

- **PRyMordial** (Core BBN code computing light-element abundances and N_eff; provides the nacreii_flag switch between PRIMAT and NACRE II rates and the likelihood interface for the Monte Carlo) — github.com/vallima/PRyMordial
- **emcee** (Ensemble Monte Carlo sampler used to explore the posterior over tau_n, Omega_B h^2, and the 63 nuclear-rate nuisance parameters)
- **PRIMAT** (Source of the ab-initio thermonuclear nuclear-rate set used as the baseline (nacreii_flag off))
- **Python 3** (Runtime language in which PRyMordial is written and runs entirely, interfacing with the Monte Carlo sampling libraries)

## Examples

```
python runPRyM_julia.py
```

## Evaluation signals

- The marginalized posteriors reproduce the reported pattern: a deuterium anomaly arises with PRIMAT rates and disappears when NACRE II rates are used via nacreii_flag
- The lithium problem remains statistically significant under both the PRIMAT and NACRE II rate prescriptions
- Both chains are sampled over 2 Gaussian-constrained cosmological/nuclear parameters plus 63 log-normal nuisance parameters, and 1D/2D marginal posteriors are extractable from each
- N_eff is computed including non-instantaneous decoupling effects, and predicted Y_p / D/H are consistent with PRyMordial's precision-BBN expectations

## Limitations

- The deuterium anomaly found with PRIMAT rates is completely washed out under the NACRE II database, so conclusions are rate-compilation dependent
- PRyMordial runs in Python 3 but optional Julia/SciML routines (PyJulia, diffeqpy) are needed for the most advanced usage; Numba, Numdifftools, and Vegas are recommended but the code can avoid the latter dependencies
- Existing public BBN codes (PArthENoPE, PRIMAT, AlterBBN) have known limitations in accuracy, language, or BSM-capable precision treatment — verify PRyMordial's settings rather than mixing codebases

## Evidence

- [results] The SM Monte Carlo analysis with PRyMordial (63 nuclear-rate nuisance parameters varied log-normally) displays the deuterium anomaly for the PRIMAT key nuclear rates: [evidence span withheld — anchor status: not_found]
- [methods] Set up the emcee ensemble Monte Carlo sampler and run it twice, once with nacreii_flag off (PRIMAT rates) and once with nacreii_flag on (NACRE II rates): [evidence span withheld — anchor status: not_found]
- [abstract] The package is written in Python, but more advanced users can optionally take advantage of the open-source community for Julia.: "The package is written in Python, but more advanced users can optionally take advantage of the open-source community for Julia."
- [readme] Examples on the usage of PRyMordial are provided in the Jupyter notebooks PRyMdemoSM and PRyMdemoNP.: "Examples on the usage of PRyMordial are provided in the Jupyter notebooks PRyMdemoSM and PRyMdemoNP."
