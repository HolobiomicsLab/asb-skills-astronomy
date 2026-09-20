---
name: mcmc-posterior-marginalization
description: Use when you need to marginalize over nuisance parameters (e.g., 63 log-normal
  nuclear-rate uncertainties) to obtain light-element abundance posteriors from a
  BBN forward model like PRyMordial, comparing rate prescriptions such as PRIMAT vs
  NACRE II.
when_to_use_negative:
- Single-point BBN abundance predictions with fixed input parameters — no Monte Carlo
  sampling is needed; run the PRyMordial forward model directly
- BSM analyses where nuclear-rate uncertainties are not the dominant nuisance (e.g.,
  varying N_eff or new-particle lifetimes without rate systematics) — the 63-parameter
  log-normal nuisance treatment is specific to rate-uncertainty propagation
- Datasets requiring a non-ensemble, non-MCMC inference method (e.g., direct likelihood
  scans) where emcee chain statistics are inappropriate
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: PRyMordial
  role: Fast and precise BBN forward model computing light-element abundances; configured
    with PRIMAT or NACRE II rates and evaluated inside the Monte Carlo likelihood
  repo: https://github.com/vallima/PRyMordial
- name: emcee
  role: Ensemble Monte Carlo sampler used to sample the posterior over the Gaussian-constrained
    parameters plus the 63 log-normal nuisance parameters, run once per rate prescription
- name: Python 3
  role: Implementation language for the likelihood, sampler wiring, and chain marginalization;
    PRyMordial runs entirely with Python 3
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1140/epjc/s10052-024-12442-0
    title: 'PRyMordial: the first three minutes, within and beyond the standard model'
  source_content_hash: sha256:15f29ccce1767088f74685a80e33c8f33e7a0e620baa37acd1965caf9f50fb79
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/vallima/PRyMordial
  tools:
  - PRyMordial
  - Python 3
  - emcee
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# mcmc-posterior-marginalization

## Summary

Ensemble Monte Carlo posterior sampling and marginalization for Big Bang Nucleosynthesis abundance analyses, using PRyMordial as the fast BBN forward model and emcee as the sampler, with log-normal nuclear-rate nuisance parameters propagated into 1D/2D abundance posteriors.

## When to use

Use when you need to marginalize over nuisance parameters (e.g., 63 log-normal nuclear-rate uncertainties) to obtain light-element abundance posteriors from a BBN forward model like PRyMordial, comparing rate prescriptions such as PRIMAT vs NACRE II.

## When NOT to use

- Single-point BBN abundance predictions with fixed input parameters — no Monte Carlo sampling is needed; run the PRyMordial forward model directly
- BSM analyses where nuclear-rate uncertainties are not the dominant nuisance (e.g., varying N_eff or new-particle lifetimes without rate systematics) — the 63-parameter log-normal nuisance treatment is specific to rate-uncertainty propagation
- Datasets requiring a non-ensemble, non-MCMC inference method (e.g., direct likelihood scans) where emcee chain statistics are inappropriate

## Inputs

- PRyMordial BBN configuration with PRIMAT or NACRE II thermonuclear rates (nacreii_flag off/on)
- Gaussian priors: tau_n = 878.4 +/- 0.5 s and Omega_B h^2 = 0.02230 +/- 0.00020
- 63 nuclear-rate nuisance parameters varied log-normally
- Observed primordial light-element abundance measurements (deuterium, helium-4, lithium) for likelihood comparison

## Outputs

- Marginalized 1D posterior distributions of key parameters for each nacreii_flag setting
- 2D posterior distributions of key parameters for each nacreii_flag setting
- Posterior-predicted light-element abundance distributions (deuterium, helium-4, lithium) per rate prescription
- emcee MCMC chains over cosmological, nuclear, and nuisance parameters

## How to apply

Configure the PRyMordial BBN computation with a chosen thermonuclear-rate database (PRIMAT rates with nacreii_flag off, NACRE II rates with nacreii_flag on) and define Gaussian priors on the two cosmological/nuclear parameters: tau_n = 878.4 +/- 0.5 s and Omega_B h^2 = 0.02230 +/- 0.00020. Implement the likelihood so the 63 nuclear-rate nuisance parameters are varied log-normally, propagating thermonuclear-rate uncertainties into predicted light-element abundances. Run the emcee ensemble sampler twice — once per rate prescription — sampling the posterior over the Gaussian-constrained parameters plus the nuisance parameters. Marginalize the resulting chains to 1D and 2D posterior distributions, then compare the posterior-predicted deuterium abundance against the observed primordial value. Rationale: PRyMordial was designed to be a user-friendly Python 3 code interfacing with Monte Carlo analysis libraries, filling gaps of existing codes (PArthENoPE, PRIMAT, AlterBBN) in rendering easy exploration of thermonuclear-rate uncertainties.

## Related tools

- **PRyMordial** (Fast and precise BBN forward model computing light-element abundances; configured with PRIMAT or NACRE II rates and evaluated inside the Monte Carlo likelihood) — https://github.com/vallima/PRyMordial
- **emcee** (Ensemble Monte Carlo sampler used to sample the posterior over the Gaussian-constrained parameters plus the 63 log-normal nuisance parameters, run once per rate prescription)
- **Python 3** (Implementation language for the likelihood, sampler wiring, and chain marginalization; PRyMordial runs entirely with Python 3)

## Examples

```
python runPRyM_julia.py
```

## Evaluation signals

- The marginalized posterior under PRIMAT rates (nacreii_flag off) shows a statistically significant deuterium anomaly relative to the observed primordial deuterium value
- Switching to NACRE II rates (nacreii_flag on) completely washes out the deuterium anomaly while the lithium problem remains statistically significant
- Chains include the specified Gaussian priors (tau_n = 878.4 +/- 0.5 s, Omega_B h^2 = 0.02230 +/- 0.00020) and exactly 63 log-normal nuisance parameters
- 1D and 2D marginal distributions are recoverable from the chains for both nacreii_flag settings, matching the reported PRIMAT-vs-NACRE II pattern

## Limitations

- Optional Julia/SciML acceleration (PyJulia, diffeqpy) must be installed separately; the package otherwise relies on NumPy, SciPy, and recommended Numba, Numdifftools, and Vegas
- The deuterium-anomaly conclusion is conditional on the nuclear-rate database chosen; switching rates (PRIMAT vs NACRE II) changes the statistical significance of anomalies
- PRyMordial's paper notes its discussion/reproducibility section was not extractable ('_No changelog found_'), so no artifact-level limitations are documented in the source context

## Evidence

- [abstract] The package is written in Python, but more advanced users can optionally take advantage of the open-source community for Julia.: "The package is written in Python, but more advanced users can optionally take advantage of the open-source community for Julia."
- [abstract] PRyMordial offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom.: "The code offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom, including non-instantaneous decoupling effects."
- [intro] A precision tool for Cosmology... should render easy exploration of input parameter and thermonuclear rate uncertainties and interface with Monte Carlo analysis libraries.: [evidence span withheld — anchor status: not_found]
- [readme] Examples on the usage of PRyMordial are provided in the Jupyter notebooks PRyMdemoSM and PRyMdemoNP.: "Examples on the usage of PRyMordial are provided in the Jupyter notebooks PRyMdemoSM and PRyMdemoNP."
