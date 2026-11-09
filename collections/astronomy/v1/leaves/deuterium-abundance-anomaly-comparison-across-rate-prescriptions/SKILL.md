---
name: deuterium-abundance-anomaly-comparison-across-rate-prescriptions
description: Use when primordial deuterium (or lithium) abundance predictions from
  a BBN code must be compared under different nuclear-rate compilations (PRIMAT vs
  NACRE II), propagating rate uncertainties via Monte Carlo posterior sampling.
when_to_use_negative:
- When a single fixed-rate BBN point prediction suffices (no rate-uncertainty marginalization
  needed) — a direct PRyMordial run is cheaper than the full emcee double-run.
- When the question is Standard-Model thermal-background physics (N_eff, neutron-to-proton
  conversion) rather than nuclear-rate-systematics comparison.
- When only one rate compilation is of interest — the skill's value comes from the
  PRIMAT-vs-NACRE II contrast, not from either run alone.
edam_operation: http://edamontology.org/operation_0438
tools:
- name: PRyMordial
  role: BBN computation engine producing light-element abundances and handling the
    nacreii_flag switch between PRIMAT and NACRE II rates
  repo: https://github.com/vallima/PRyMordial
- name: emcee
  role: Ensemble Monte Carlo sampler used to run the posterior twice, once per rate
    prescription
- name: PRIMAT
  role: Source of the default thermonuclear key-rate compilation
- name: Python 3
  role: Language in which PRyMordial runs entirely, interfacing with Monte Carlo analysis
    libraries
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1140/epjc/s10052-024-12442-0
    title: 'PRyMordial: the first three minutes, within and beyond the standard model'
  source_content_hash: sha256:5e77686c58b033ba91afc4b0ccab519b0b227b33bf4b6371b3992411c6a8669c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/vallima/PRyMordial
  tools:
  - PRIMAT
  - PRyMordial
  - Python 3
  - emcee
  edam_operation: http://edamontology.org/operation_0438
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# deuterium-abundance-anomaly-comparison-across-rate-prescriptions

## Summary

Runs PRyMordial's BBN Monte Carlo twice — once with PRIMAT thermonuclear rates and once with NACRE II rates (nacreii_flag on) — sampling 63 log-normal nuclear-rate nuisance parameters plus Gaussian-prior-constrained tau_n and Omega_B h^2, then compares marginalized posterior-predicted deuterium abundances against observation to test whether the deuterium anomaly persists or washes out under each rate prescription.

## When to use

Use when primordial deuterium (or lithium) abundance predictions from a BBN code must be compared under different nuclear-rate compilations (PRIMAT vs NACRE II), propagating rate uncertainties via Monte Carlo posterior sampling.

## When NOT to use

- When a single fixed-rate BBN point prediction suffices (no rate-uncertainty marginalization needed) — a direct PRyMordial run is cheaper than the full emcee double-run.
- When the question is Standard-Model thermal-background physics (N_eff, neutron-to-proton conversion) rather than nuclear-rate-systematics comparison.
- When only one rate compilation is of interest — the skill's value comes from the PRIMAT-vs-NACRE II contrast, not from either run alone.

## Inputs

- Gaussian priors on neutron lifetime tau_n = 878.4 +/- 0.5 s and baryon density Omega_B h^2 = 0.02230 +/- 0.00020
- PRIMAT thermonuclear key-rate compilation (nacreii_flag off)
- NACRE II thermonuclear key-rate compilation (nacreii_flag on)
- 63 log-normal nuclear-rate nuisance parameters with their uncertainty priors
- Observed primordial deuterium abundance value for anomaly comparison

## Outputs

- emcee posterior chains for each nacreii_flag setting (PRIMAT and NACRE II runs)
- Marginalized 1D and 2D posterior distributions of light-element abundances (deuterium, helium-4, lithium) and key parameters
- Binary verdict on deuterium anomaly presence under each rate prescription

## How to apply

Configure PRyMordial for BBN with PRIMAT thermonuclear rates and set Gaussian priors tau_n = 878.4 +/- 0.5 s and Omega_B h^2 = 0.02230 +/- 0.00020. Implement the likelihood so the 63 nuclear-rate nuisance parameters vary log-normally, propagating thermonuclear-rate uncertainties into light-element abundance predictions. Run the emcee ensemble sampler twice: once with nacreii_flag off (PRIMAT rates) and once with nacreii_flag on (NACRE II rates), sampling over the two cosmological parameters plus the nuisance parameters. Marginalize the chains to obtain 1D and 2D posterior distributions of the key parameters for each flag setting, then compare the posterior-predicted deuterium abundance against the observed primordial value under each prescription. The rationale is that the deuterium anomaly arises with PRIMAT rates but is completely washed out with NACRE II, isolating nuclear-rate uncertainties as the driver, while the lithium problem remains statistically significant under both.

## Related tools

- **PRyMordial** (BBN computation engine producing light-element abundances and handling the nacreii_flag switch between PRIMAT and NACRE II rates) — https://github.com/vallima/PRyMordial
- **emcee** (Ensemble Monte Carlo sampler used to run the posterior twice, once per rate prescription)
- **PRIMAT** (Source of the default thermonuclear key-rate compilation)
- **Python 3** (Language in which PRyMordial runs entirely, interfacing with Monte Carlo analysis libraries)

## Examples

```
python runPRyM_julia.py
```

## Evaluation signals

- The marginalized posteriors reproduce the reported pattern: a deuterium anomaly with PRIMAT rates that disappears when nacreii_flag enables NACRE II rates.
- The lithium problem remains statistically significant in both PRIMAT and NACRE II runs — a failure to see this indicates a sampling or likelihood error.
- Both emcee runs converge over the full parameter space: 2 Gaussian-constrained cosmological/nuclear parameters plus 63 log-normal nuisance parameters.
- Marginalized 1D and 2D posterior distributions are produced for both nacreii_flag settings and can be overlaid for comparison.
- Posterior-predicted deuterium abundance is explicitly compared against the observed primordial deuterium value in each run.

## Limitations

- The deuterium-anomaly conclusion is rate-compilation-dependent: it is an artifact of the PRIMAT key-rate prescription and is washed out with NACRE II, so it should not be reported as a physical anomaly.
- The comparison rests on log-normal treatment of 63 nuisance parameters and the specific Gaussian priors on tau_n and Omega_B h^2; different prior choices could alter the posterior pattern.
- PRyMordial's optional Julia/SciML routines require PyJulia and diffeqpy; the README notes Numba, Numdifftools, and Vegas are recommended (not mandatory) for best usage, so missing optional dependencies can affect performance or accuracy.

## Evidence

- [abstract] The code offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom: "The code offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom, including non-instantaneous decoupling effects."
- [results] The SM Monte Carlo analysis with PRyMordial (63 nuclear-rate nuisance parameters varied log-normally) displays the deuterium anomaly for the PRIMAT key nuclear rates, which is completely washed out when the NACRE II database is employed, while the lithium problem is statistically significant regardless of the nuclear-rate approach.: [evidence span withheld — anchor status: not_found]
- [intro] PRIMAT [47, 48] is an user-friendly Mathematica package containing all the inputs and ingredients for an ab-initio computation: "PRIMAT [47, 48] is an user-friendly Mathematica package containing all the inputs and ingredients for an ab-initio computation"
- [readme] Examples on the usage of PRyMordial are provided in the Jupyter notebooks PRyMdemoSM and PRyMdemoNP.: "Examples on the usage of PRyMordial are provided in the Jupyter notebooks PRyMdemoSM and PRyMdemoNP."
