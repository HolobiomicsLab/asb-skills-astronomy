---
name: lognormal-mixture-temperature-deprojection
description: Use when deep X-ray surface brightness, spectroscopic temperature, and
  Sunyaev-Zel'dovich pressure profiles are available for galaxy clusters and a nonparametric
  hydrostatic mass profile is needed to test parametric forms such as NFW or Einasto.
when_to_use_negative:
- When the goal is to fit a specific parametric mass model (NFW, Einasto, polytropic)
  rather than a shape-free reconstruction — hydromass's parametric mass model fitting
  or forward fitting modes are more appropriate
- When no SZ pressure data are available and only X-ray constraints exist at the cluster
  outskirts, since the joint X-ray+SZ fit is what extends the reconstruction over
  two decades in radius
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: hydromass
  role: Provides the nonparametric log-normal mixture model (hydromass.nonparametric
    module) for temperature deprojection, PSF deconvolution, and joint Bayesian hydrostatic
    mass reconstruction
  repo: https://github.com/domeckert/hydromass
- name: pymc3
  role: Backend for Hamiltonian Monte Carlo sampling of the joint Bayesian posterior
    over deprojected profiles and mass
- name: pyproffit
  role: hydromass dependency used for handling the X-ray surface brightness and spectroscopic
    profile data products
- name: astropy
  role: hydromass dependency for astronomical units, coordinates, and cosmology used
    in profile reconstruction
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202142507
    title: The gravitational field of X-COP galaxy clusters
  source_content_hash: sha256:1dc8de37e53643247668b7e2d71ef4a9884e25c9db78a6a19b3b401db98356d7
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# lognormal-mixture-temperature-deprojection

## Summary

Nonparametric reconstruction of cluster thermodynamic and hydrostatic mass profiles using hydromass's log-normal mixture model, which deprojects X-ray spectroscopic temperatures with PSF deconvolution and jointly fits X-ray surface brightness and SZ pressure under hydrostatic equilibrium. It avoids imposing a parametric (e.g., NFW/Einasto) shape on the recovered gravitational field.

## When to use

Use when deep X-ray surface brightness, spectroscopic temperature, and Sunyaev-Zel'dovich pressure profiles are available for galaxy clusters and a nonparametric hydrostatic mass profile is needed to test parametric forms such as NFW or Einasto.

## When NOT to use

- When the goal is to fit a specific parametric mass model (NFW, Einasto, polytropic) rather than a shape-free reconstruction — hydromass's parametric mass model fitting or forward fitting modes are more appropriate
- When no SZ pressure data are available and only X-ray constraints exist at the cluster outskirts, since the joint X-ray+SZ fit is what extends the reconstruction over two decades in radius

## Inputs

- X-ray surface brightness profiles for the cluster sample
- X-ray spectroscopic temperature profiles
- Sunyaev-Zel'dovich pressure profiles
- Public X-COP data products for the 12-cluster sample

## Outputs

- Nonparametric hydrostatic mass (gravitational-field) profiles with posterior uncertainties over the full radial range (saved to nonparametric_mass_profiles.csv)
- Deprojected thermodynamic gas profiles with PSF deconvolution
- Output mass and thermodynamic profile plots
- Goodness-of-fit diagnostics (posterior predictive checks and WAIC)

## How to apply

Load the public X-COP data products (X-ray surface brightness, spectroscopic temperature, and SZ pressure profiles) into hydromass within a Jupyter notebook. Configure the nonparametric log-normal mixture model in the hydromass.nonparametric module to deproject the temperature and deconvolve the PSF of the thermodynamic gas profiles. Run the joint Bayesian fit of the three observables under the hydrostatic-equilibrium assumption using Hamiltonian Monte Carlo sampling through PyMC3. Extract the recovered gravitational-field (hydrostatic mass) profiles with posterior uncertainties over the full radial range for each cluster, and assess goodness-of-fit with posterior predictive checks and WAIC before accepting the reconstruction.

## Related tools

- **hydromass** (Provides the nonparametric log-normal mixture model (hydromass.nonparametric module) for temperature deprojection, PSF deconvolution, and joint Bayesian hydrostatic mass reconstruction) — https://github.com/domeckert/hydromass
- **pymc3** (Backend for Hamiltonian Monte Carlo sampling of the joint Bayesian posterior over deprojected profiles and mass)
- **pyproffit** (hydromass dependency used for handling the X-ray surface brightness and spectroscopic profile data products)
- **astropy** (hydromass dependency for astronomical units, coordinates, and cosmology used in profile reconstruction)

## Examples

```
from hydromass import nonparametric; nonparametric.fit(sbm=sb_prof, tmk=t_prof, psz=p_sz, nmca=nmix, mcord=True)
```

## Evaluation signals

- Posterior predictive checks indicate an adequate joint fit to the observed X-ray and SZ profiles
- WAIC diagnostics support the reconstruction; accept only when both PPC and WAIC are adequate
- Recovered mass profiles extend over roughly two decades in radius for each cluster
- Comparison against parametric fits: on average the NFW profile describes the recovered mass profiles with deviations of less than 10% over a wide radial range
- Posterior uncertainties on the mass profiles are quantified over the full radial range for each of the 12 clusters

## Limitations

- The PyPI release may not be the latest version, so some features may be missing; the hydromass version is not stated in the article
- Reconstruction relies on the hydrostatic-equilibrium assumption; the baryon fraction slightly exceeding the cosmic value beyond R200 suggests mild nonthermal pressure support (10-20%) in cluster outskirts that this model does not capture by default
- There is more diversity in the shape of individual mass profiles than can be captured by the NFW model, so parametric comparisons must be interpreted per-cluster rather than only on average

## Evidence

- [readme] Non-parametric temperature deprojection and hydrostatic mass reconstruction using a log-normal mixture model: "Non-parametric temperature deprojection and hydrostatic mass reconstruction using a log-normal mixture model"
- [methods] Joint X-ray/SZ modeling with HMC: "Joint modeling of X-ray surface brightness, X-ray spectroscopic temperature, and SZ pressure"
- [readme] Diagnostics via PPC and WAIC: "Diagnostic tools to investigate goodness-of-fit through posterior predictive checks and WAIC"
- [abstract] NFW agreement threshold: "the NFW profile provides an excellent description of the recovered mass profiles, with deviations of less than 10% over a wide radial range"
- [methods] Public X-COP data products loaded in notebook: "the user to easily load the public X-COP data products and apply our reconstruction tools directly within a Jupyter notebook"
