---
name: bbn-abundance-likelihood-evaluation
description: Use when reproducing or extending Bayesian BBN constraints — e.g. placing
  credible bounds on NP modifications of weak n↔p conversion rates from primordial
  Y_p and D/H measurements via emcee sampling of a PRyMordial prediction pipeline.
when_to_use_negative:
- The goal is a forward-model BBN abundance prediction under the Standard Model only
  (e.g. the PRyMdemoSM notebook quick run) rather than a Bayesian parameter constraint
  — a single PRyMordial evaluation suffices.
- The analysis targets full BSM thermal-history scenarios (e.g. extra relativistic
  degrees of freedom with modified expansion) rather than a low-dimensional rescaling
  of the weak n↔p conversion rates — a more general BSM code or the NUDEC_BSM-style
  approach is more appropriate.
- Inputs are measured light-element abundances without Gaussian uncertainties and
  priors on tau_n and Omega_B h^2 — the likelihood construction assumes PDG-style
  percent-level Gaussian measurements.
edam_operation: http://edamontology.org/operation_3656
tools:
- name: PRyMordial
  role: Computes Y_p and D/H abundances for each proposed parameter point, with small
    networks and NACRE II reaction rates enabled via flags
  repo: https://github.com/vallima/PRyMordial
- name: emcee
  role: Affine-invariant ensemble sampler used to draw the NP_delta_nTOp posterior
    with multiple walkers and burn-in
- name: Python 3
  role: Runtime environment for the likelihood, sampler driver, and corner-plot rendering
- name: NumPy
  role: Mandatory dependency for array handling in the PRyMordial pipeline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1140/epjc/s10052-024-12442-0
    title: 'PRyMordial: the first three minutes, within and beyond the standard model'
  source_content_hash: sha256:6c8d16bc25bcde81c14e3c688e1377cfd821c8c128e4184ec241718591e5e0e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/vallima/PRyMordial
  tools:
  - NumPy
  - PRyMordial
  - Python 3
  - emcee
  edam_operation: http://edamontology.org/operation_3656
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# bbn-abundance-likelihood-evaluation

## Summary

Builds and evaluates a Bayesian BBN likelihood comparing PRyMordial-predicted primordial helium-4 (Y_p) and deuterium (D/H) abundances against PDG percent-level measurements, with Gaussian priors on tau_n and Omega_B h^2, to constrain New Physics parameters such as NP_delta_nTOp. It packages the parameter-inference loop (prediction -> likelihood -> MCMC sampling -> credible contours) around the PRyMordial abundance engine.

## When to use

Use when reproducing or extending Bayesian BBN constraints — e.g. placing credible bounds on NP modifications of weak n↔p conversion rates from primordial Y_p and D/H measurements via emcee sampling of a PRyMordial prediction pipeline.

## When NOT to use

- The goal is a forward-model BBN abundance prediction under the Standard Model only (e.g. the PRyMdemoSM notebook quick run) rather than a Bayesian parameter constraint — a single PRyMordial evaluation suffices.
- The analysis targets full BSM thermal-history scenarios (e.g. extra relativistic degrees of freedom with modified expansion) rather than a low-dimensional rescaling of the weak n↔p conversion rates — a more general BSM code or the NUDEC_BSM-style approach is more appropriate.
- Inputs are measured light-element abundances without Gaussian uncertainties and priors on tau_n and Omega_B h^2 — the likelihood construction assumes PDG-style percent-level Gaussian measurements.

## Inputs

- PRyMordial n↔p prediction pipeline (github.com/vallima/PRyMordial) with smallnet_flag and nacreii_flags enabled
- PDG percent-level measurements of primordial helium-4 (Y_p) and deuterium (D/H) abundances
- Gaussian priors on neutron lifetime tau_n and baryon density Omega_B h^2
- NP parameter to constrain (e.g. NP_delta_nTOp scaling the n↔p Born rates)

## Outputs

- Posterior samples from the emcee affine-invariant ensemble sampler
- Joint 68% and 95% credible contours rendered as a matplotlib corner plot
- Upper credible bound on NP_delta_nTOp expressed as a few-percent limit on the n↔p Born-rate modification
- Saved posterior chain, corner plot, and upper-bound value

## How to apply

Load the PRyMordial n↔p prediction pipeline from github.com/vallima/PRyMordial and enable the smallnet_flag and nacreii_flags so Born weak rates use the small networks and NACRE II reaction library. Construct a Gaussian likelihood comparing PRyMordial-predicted Y_p and D/H against the PDG percent-level measurements, imposing Gaussian priors on tau_n and Omega_B h^2. Sample the NP parameter (e.g. NP_delta_nTOp) with the emcee affine-invariant ensemble sampler, running enough walkers and burn-in steps to converge the 2-parameter posterior, then compute joint 68%/95% credible contours and render a corner plot. Extract the upper credible bound and convert it into a few-percent limit on the NP modification of the n↔p Born rates; PRyMordial is suited to this because it offers 'fast and precise evaluation of BBN light-element abundances' and can be driven from Python with optional Julia/SciML routines.

## Related tools

- **PRyMordial** (Computes Y_p and D/H abundances for each proposed parameter point, with small networks and NACRE II reaction rates enabled via flags) — https://github.com/vallima/PRyMordial
- **emcee** (Affine-invariant ensemble sampler used to draw the NP_delta_nTOp posterior with multiple walkers and burn-in)
- **Python 3** (Runtime environment for the likelihood, sampler driver, and corner-plot rendering)
- **NumPy** (Mandatory dependency for array handling in the PRyMordial pipeline)

## Examples

```
python runPRyM_julia.py
```

## Evaluation signals

- Reproduced 68% and 95% joint credible regions on (Y_p, D/H)-constrained NP_delta_nTOp match the paper's reported corner-plot contours.
- The extracted upper bound on the NP modification of the n↔p Born rates remains at the few-percent level relative to the standard Born rates.
- The emcee chain passes convergence checks: enough walkers and burn-in steps were run for a 2-parameter posterior (visually stationary traces, stable contours across longer runs).
- Predicted Y_p and D/H at the posterior maximum are consistent with the PDG percent-level measurements within the Gaussian likelihood.
- Saved artifacts exist on disk: posterior chain file, corner plot image, and the numeric upper-bound value.

## Limitations

- The constraint strength is limited by the percent-level precision of the PDG helium-4 and deuterium measurements; NP in the weak rates is only consistent up to a few percent.
- PRyMordial's advanced routines (SciML/Julia via PyJulia and diffeqpy) are optional; without Numba, Numdifftools, or Vegas the default code path is slower, though it 'can easily avoid dependencies on Numba and Numdifftools'.
- Repository discussion/documentation sections were empty ('No changelog found.'), so reproducibility details beyond the paper and README are limited.

## Evidence

- [abstract] fast and precise evaluation of BBN light-element abundances: "The code offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom, including non-instantaneous decoupling effects."
- [abstract] PRyMordial suitable for NP investigations during BBN: "PRyMordial is suitable for state-of-the-art analyses in the Standard Model as well as for general investigations into New Physics active during BBN."
- [intro] percent-level primordial measurements motivating BBN constraints: "the study of the Early Universe around the BBN epoch offers unique insight on New Physics (NP)"
- [intro] Python 3 runtime with optional Julia routines: "The package is written and runs entirely with Python 3"
- [readme] README quick example invocation and demo notebooks: "To run a quick example, open the terminal and once in PRyMordial folder just type: 

python runPRyM_julia.py"
- [readme] README dependency list for the pipeline: "- NumPy (mandatory) – pip install numpy
- SciPy (mandatory) – pip install scipy
- Numba (recommended) – pip install numba"
