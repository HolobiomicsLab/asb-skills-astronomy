---
name: hamiltonian-monte-carlo-bayesian-inference
description: Use when reconstructing hydrostatic mass profiles or fitting parametric
  mass models (NFW, Einasto) to X-ray surface brightness, spectroscopic temperature,
  and SZ pressure profiles under a global Bayesian framework with posterior uncertainties.
when_to_use_negative:
- Input data are not X-ray and/or Sunyaev–Zel'dovich cluster profiles, since hydromass
  is designed specifically for deprojection and mass reconstruction from such observations
- A quick deterministic fit suffices and the overhead of HMC-based Bayesian posterior
  sampling and convergence diagnostics is unnecessary
- The system violates the hydrostatic-equilibrium assumption so strongly that hydrostatic
  mass reconstruction is not meaningful
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3452
tools:
- name: hydromass
  role: Python package providing the global Bayesian framework for joint deprojection,
    PSF deconvolution, and hydrostatic mass profile reconstruction fitted via HMC
  repo: https://github.com/domeckert/hydromass
- name: pymc3
  role: Backend performing the Hamiltonian Monte Carlo sampling of the joint Bayesian
    model
- name: pyproffit
  role: Declared hydromass dependency supporting X-ray/SZ profile analysis
- name: numpy
  role: Declared hydromass dependency for numerical array handling
- name: scipy
  role: Declared hydromass dependency for scientific computing routines
- name: astropy
  role: Declared hydromass dependency for astronomical units and cosmology handling
- name: matplotlib
  role: Declared hydromass dependency for visualizing output mass and thermodynamic
    profiles
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202142507
    title: The gravitational field of X-COP galaxy clusters
  source_content_hash: sha256:6c8a9c50f196880466fe736f1a16d22ce3ca8b1925267800a13094f9b193793e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/domeckert/hydromass
  tools:
  - Python
  - astropy
  - hydromass
  - matplotlib
  - numpy
  - pymc3
  - pyproffit
  - scipy
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# hamiltonian-monte-carlo-bayesian-inference

## Summary

Efficient Bayesian inference of hydrostatic mass profiles and cluster thermodynamics using Hamiltonian Monte Carlo sampling implemented in PyMC3 within the hydromass package, applied to joint X-ray and SZ data of the 12 X-COP galaxy clusters.

## When to use

Use when reconstructing hydrostatic mass profiles or fitting parametric mass models (NFW, Einasto) to X-ray surface brightness, spectroscopic temperature, and SZ pressure profiles under a global Bayesian framework with posterior uncertainties.

## When NOT to use

- Input data are not X-ray and/or Sunyaev–Zel'dovich cluster profiles, since hydromass is designed specifically for deprojection and mass reconstruction from such observations
- A quick deterministic fit suffices and the overhead of HMC-based Bayesian posterior sampling and convergence diagnostics is unnecessary
- The system violates the hydrostatic-equilibrium assumption so strongly that hydrostatic mass reconstruction is not meaningful

## Inputs

- X-ray surface brightness profiles of galaxy clusters
- X-ray spectroscopic temperature profiles
- Sunyaev–Zel'dovich pressure profiles
- Mass model specification (NFW, Einasto, or nonparametric log-normal mixture) with automatic or custom priors

## Outputs

- Posterior samples of hydrostatic mass profile parameters (c200, M200, Einasto alpha)
- Recovered nonparametric gravitational-field (mass) profiles with posterior uncertainties
- Goodness-of-fit diagnostics (posterior predictive checks, WAIC)
- Output mass and thermodynamic profile plots and saved/reloadable fit objects

## How to apply

Load the public X-COP data products (X-ray surface brightness, X-ray spectroscopic temperature, and SZ pressure profiles for the 12-cluster sample) into hydromass within a Jupyter notebook. Configure either the parametric mass model reconstruction (NFW, Einasto, with automatic or custom priors) or the nonparametric log-normal mixture deprojection in hydromass.nonparametric, including PSF deconvolution of thermodynamic gas profiles. Run the joint Bayesian fit of X-ray surface brightness, spectroscopic temperature, and SZ pressure under the hydrostatic-equilibrium assumption using Hamiltonian Monte Carlo sampling via PyMC3. Extract posterior samples for mass profile parameters (e.g., c200, M200, and the Einasto shape parameter alpha) and evaluate goodness-of-fit with posterior predictive checks and WAIC before accepting the reconstruction.

## Related tools

- **hydromass** (Python package providing the global Bayesian framework for joint deprojection, PSF deconvolution, and hydrostatic mass profile reconstruction fitted via HMC) — https://github.com/domeckert/hydromass
- **pymc3** (Backend performing the Hamiltonian Monte Carlo sampling of the joint Bayesian model)
- **pyproffit** (Declared hydromass dependency supporting X-ray/SZ profile analysis)
- **numpy** (Declared hydromass dependency for numerical array handling)
- **scipy** (Declared hydromass dependency for scientific computing routines)
- **astropy** (Declared hydromass dependency for astronomical units and cosmology handling)
- **matplotlib** (Declared hydromass dependency for visualizing output mass and thermodynamic profiles)

## Examples

```
import hydromass; hydromass.mhyd(survey_data, model='NFW')  # HMC sampling via PyMC3 within a Jupyter notebook on X-COP data products
```

## Evaluation signals

- Posterior predictive checks indicate an adequate joint fit to the observed X-ray and SZ profiles
- WAIC diagnostics indicate an adequate joint fit and support model comparison
- Recovered NFW fits deviate from the nonparametric hydrostatic mass profiles by less than 10% over a wide radial range
- Mean NFW concentration and its scatter agree with the ΛCDM concentration–mass prediction
- Posterior uncertainties cover the full radial range out to R200 for each of the 12 clusters

## Limitations

- The version of hydromass is not stated; the PyPI release may not be the latest version so some features may be missing
- No discussion-section limitations or reproducibility details were available in the extracted source content
- Reconstruction rests on the hydrostatic-equilibrium assumption; excess baryon fraction beyond R200 possibly indicates mild nonthermal pressure support (10–20%) in cluster outskirts

## Evidence

- [readme] Efficient Bayesian optimization based on Hamiltonian Monte Carlo using PyMC3: "Efficient Bayesian optimization based on Hamiltonian Monte Carlo using PyMC3"
- [methods] Joint Bayesian fit of X-ray and SZ data: "Joint modeling of X-ray surface brightness, X-ray spectroscopic temperature, and SZ pressure"
- [methods] Parametric mass models with automatic or custom priors: "Parametric mass model reconstruction including Navarro-Frenk-White, Einasto and several other popular mass models, with automatic or custom priors"
- [abstract] Nonparametric log-normal mixture reconstruction: "We introduce a nonparametric framework to reconstruct the shape of the gravitational field under the assumption of hydrostatic equilibrium"
- [abstract] NFW describes recovered profiles to <10%: "the NFW profile provides an excellent description of the recovered mass profiles, with deviations of less than 10% over a wide radial range"
- [readme] Goodness-of-fit via posterior predictive checks and WAIC: "Diagnostic tools to investigate goodness-of-fit through posterior predictive checks and WAIC"
