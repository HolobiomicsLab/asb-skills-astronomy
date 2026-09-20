---
name: neutron-to-proton-born-rate-constraint-analysis
description: Use when the goal is to constrain fractional NP modifications of BBN-era
  weak n↔p conversion rates from percent-level PDG measurements of primordial helium-4
  and deuterium, using PRyMordial predictions sampled with emcee.
when_to_use_negative:
- When the target is NP affecting thermonuclear reaction rates or the thermal history
  (e.g. N_eff, decoupling) rather than the weak n↔p Born conversion rates — the skill's
  parameter space and likelihood are specific to NP_delta_nTOp.
- When the analysis goal is a forward prediction of BBN abundances within the Standard
  Model (e.g. via the PRyMdemoSM notebook) rather than a Bayesian constraint on a
  BSM parameter.
- When the observational inputs are not percent-level primordial D/H and Y_p measurements
  with CMB-informed Omega_B h^2 priors — the few-percent bound is only meaningful
  against PDG-level precision data.
edam_operation: http://edamontology.org/operation_0306
tools:
- name: PRyMordial
  role: Computes BBN light-element abundances (Y_p, D/H) and the effective number
    of relativistic degrees of freedom for given NP-shifted n↔p Born rates; provides
    the n_TOp prediction pipeline and cosmoMC-style likelihood table (PRyM_Yp_DH_cosmoMC_2023.dat)
  repo: github.com/vallima/PRyMordial
- name: emcee
  role: Affine-invariant ensemble MCMC sampler used to explore the NP_delta_nTOp posterior
    against the Y_p / D/H likelihood
- name: Python 3
  role: Implementation language of PRyMordial and of the likelihood/sampling/corner-plot
    workflow
- name: Julia
  role: Optional accelerated backend via the SciML kit / diffeqpy for PRyMordial's
    differential-equation routines
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1140/epjc/s10052-024-12442-0
    title: 'PRyMordial: the first three minutes, within and beyond the standard model'
  source_content_hash: sha256:84bdb303a6b51107b4daf074a947202ea8e602c9b2ea1fcbb947a32e71525180
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Julia
  - PRyMordial
  - Python 3
  - emcee
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# neutron-to-proton-born-rate-constraint-analysis

## Summary

A Bayesian workflow that uses PRyMordial's n↔p weak-conversion pipeline to place credible upper bounds on New Physics (NP) shifts of the Born neutron-to-proton rates by fitting primordial helium-4 (Y_p) and deuterium (D/H) abundances with emcee, reproducing the few-percent bound reported in the PRyMordial paper.

## When to use

Use when the goal is to constrain fractional NP modifications of BBN-era weak n↔p conversion rates from percent-level PDG measurements of primordial helium-4 and deuterium, using PRyMordial predictions sampled with emcee.

## When NOT to use

- When the target is NP affecting thermonuclear reaction rates or the thermal history (e.g. N_eff, decoupling) rather than the weak n↔p Born conversion rates — the skill's parameter space and likelihood are specific to NP_delta_nTOp.
- When the analysis goal is a forward prediction of BBN abundances within the Standard Model (e.g. via the PRyMdemoSM notebook) rather than a Bayesian constraint on a BSM parameter.
- When the observational inputs are not percent-level primordial D/H and Y_p measurements with CMB-informed Omega_B h^2 priors — the few-percent bound is only meaningful against PDG-level precision data.

## Inputs

- PRyMordial n↔p conversion-rate prediction pipeline (github.com/vallima/PRyMordial) with smallnet_flag and nacreii_flags enabled
- PDG percent-level measurements of primordial helium-4 (Y_p) and deuterium (D/H) abundances
- Gaussian priors on neutron lifetime tau_n and baryon density Omega_B h^2
- emcee sampler configuration (number of walkers, burn-in steps) for the NP_delta_nTOp parameter

## Outputs

- Posterior chain (MCMC samples) for NP_delta_nTOp saved to disk
- Corner plot with joint 68% and 95% credible contours (matplotlib)
- Upper credible bound on NP_delta_nTOp expressed as a few-percent limit on the NP modification of the n↔p Born rates

## How to apply

Load the PRyMordial prediction pipeline from github.com/vallima/PRyMordial and enable the smallnet_flag and nacreii_flags so that the n↔p Born rates run over the small nuclear networks with the NACRE II reaction library. Build a Gaussian likelihood comparing PRyMordial-predicted Y_p and D/H against the PDG percent-level measurements, with Gaussian priors on tau_n and Omega_B h^2. Sample the single NP_delta_nTOp parameter with the emcee affine-invariant ensemble sampler, running enough walkers and burn-in steps to converge the 1-2 parameter posterior. Compute joint 68% and 95% credible contours, render a corner plot with matplotlib, extract the upper credible bound on NP_delta_nTOp, and convert it into the few-percent upper limit on the NP modification of the n↔p Born rates. Validation requires the reproduced contours to match the paper's Fig. 5 (NP_delta_nTOp) regions and the Born-rate bound to remain at the few-percent level.

## Related tools

- **PRyMordial** (Computes BBN light-element abundances (Y_p, D/H) and the effective number of relativistic degrees of freedom for given NP-shifted n↔p Born rates; provides the n_TOp prediction pipeline and cosmoMC-style likelihood table (PRyM_Yp_DH_cosmoMC_2023.dat)) — github.com/vallima/PRyMordial
- **emcee** (Affine-invariant ensemble MCMC sampler used to explore the NP_delta_nTOp posterior against the Y_p / D/H likelihood)
- **Python 3** (Implementation language of PRyMordial and of the likelihood/sampling/corner-plot workflow)
- **Julia** (Optional accelerated backend via the SciML kit / diffeqpy for PRyMordial's differential-equation routines)

## Examples

```
python runPRyM_julia.py
```

## Evaluation signals

- The reproduced 68% and 95% joint credible contours for NP_delta_nTOp match the paper's reported contours (Fig. 5, NP_delta_nTOp).
- The extracted upper bound on the NP modification of the n↔p Born rates remains at the few-percent level relative to the standard Born rates.
- The emcee chain shows convergence: enough walkers and burn-in steps that the posterior is stable across extended runs.
- PRyMordial runs with smallnet_flag and nacreii_flags enabled, so Born rates reflect the small networks and NACRE II reaction library.
- Posterior chain, corner plot, and upper-bound value are all saved to disk as deliverables.

## Limitations

- The constraint is limited to the weak n↔p conversion rates; NP acting through other channels (e.g. thermonuclear rates or thermal-bath parameters) is outside this parameter space.
- The bound scales with the precision of the abundance measurements — with only percent-level PDG data, only few-percent NP deviations can be probed; much larger rates are trivially excluded, much smaller ones are unconstrained.
- Existing public BBN codes such as PArthENoPE, PRIMAT, and AlterBBN have limitations and missing features, so results may differ from other BBN codes in accuracy or interconnection with first-principle thermal background computations.
- Optional dependencies (Numba, Numdifftools, Vegas, PyJulia, diffeqpy) affect performance; the code avoids Numba/Numdifftools dependence by default and does not use Vegas by default.

## Evidence

- [abstract] The code offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom: "The code offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom"
- [intro] percent-level measurements of primordial deuterium and helium-4, and CMB constraints on thermal history: [evidence span withheld — anchor status: not_found]
- [readme] A Python package for precise computations of BBN within and beyond the Standard Model.: "A Python package for precise computations of BBN within and beyond the Standard Model."
- [readme] Examples on the usage of PRyMordial are provided in the Jupyter notebooks PRyMdemoSM and PRyMdemoNP.: "Examples on the usage of PRyMordial are provided in the Jupyter notebooks PRyMdemoSM and PRyMdemoNP."
