---
name: hydrostatic-mass-profile-reconstruction
description: Use when you have joint X-ray (surface brightness + spectroscopic temperature)
  and SZ pressure profiles for galaxy clusters and need hydrostatic mass profiles
  with posterior uncertainties, via nonparametric or parametric reconstruction.
when_to_use_negative:
- The input is only a single radial observable (e.g. SZ pressure alone) where the
  joint X-ray + SZ Bayesian framework and its validation diagnostics cannot be applied
  as described
- The goal is testing the radial acceleration relation against baryonic-only data
  without a total gravitational-field reconstruction, since this skill's outputs (recovered
  total mass profiles) are prerequisites rather than the end product
- The system violates the hydrostatic-equilibrium assumption being relied on, e.g.
  strongly disturbed mergers where nonthermal pressure marginalization (10–20% in
  outskirts) is insufficient
edam_operation: http://edamontology.org/operation_0279
tools:
- name: hydromass
  role: 'Primary package: global Bayesian framework for joint deprojection, PSF deconvolution,
    and hydrostatic mass reconstruction (nonparametric log-normal mixture and parametric
    NFW/Einasto models)'
  repo: https://github.com/domeckert/hydromass
- name: pymc3
  role: Hamiltonian Monte Carlo backend for the efficient Bayesian optimization of
    the joint X-ray/SZ fit
- name: pyproffit
  role: Declared hydromass dependency used for X-ray/SZ profile analysis underlying
    the reconstruction
- name: numpy
  role: Declared hydromass dependency for numerical handling of radial profile grids
    and posterior samples
- name: scipy
  role: Declared hydromass dependency for scientific computation in the reconstruction
- name: astropy
  role: Declared hydromass dependency for astronomical units/cosmology handling in
    mass-profile computations
- name: matplotlib
  role: Declared hydromass dependency for visualization of output mass and thermodynamic
    profiles
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_005
  - task_006
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202142507
    title: The gravitational field of X-COP galaxy clusters
  source_content_hash: sha256:5f6f355a79ae11576cc1bea6bfcf99737bf58d7b4dcd762e2eeba083808980f7
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
  edam_operation: http://edamontology.org/operation_0279
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# hydrostatic-mass-profile-reconstruction

## Summary

Reconstructs gravitational (hydrostatic) mass profiles of galaxy clusters from X-ray surface brightness, X-ray spectroscopic temperature, and Sunyaev–Zel'dovich pressure profiles using the hydromass package's global Bayesian framework, with nonparametric log-normal mixture deprojection and Hamiltonian Monte Carlo sampling. This skill reproduces the X-COP approach of constraining cluster gravitational fields over two decades in radius under the hydrostatic-equilibrium assumption.

## When to use

Use when you have joint X-ray (surface brightness + spectroscopic temperature) and SZ pressure profiles for galaxy clusters and need hydrostatic mass profiles with posterior uncertainties, via nonparametric or parametric reconstruction.

## When NOT to use

- The input is only a single radial observable (e.g. SZ pressure alone) where the joint X-ray + SZ Bayesian framework and its validation diagnostics cannot be applied as described
- The goal is testing the radial acceleration relation against baryonic-only data without a total gravitational-field reconstruction, since this skill's outputs (recovered total mass profiles) are prerequisites rather than the end product
- The system violates the hydrostatic-equilibrium assumption being relied on, e.g. strongly disturbed mergers where nonthermal pressure marginalization (10–20% in outskirts) is insufficient

## Inputs

- X-ray surface brightness profiles of the cluster
- X-ray spectroscopic temperature profiles
- Sunyaev–Zel'dovich (SZ) pressure profiles
- PSF characterization for deconvolution of thermodynamic profiles
- Public X-COP data products for the 12-cluster sample (loaded within a Jupyter notebook)

## Outputs

- Nonparametrically reconstructed hydrostatic (gravitational) mass profiles with posterior uncertainties over two decades in radius
- Deprojected thermodynamic gas profiles (e.g. temperature, gas density) with PSF deconvolution
- Parametric mass-model fits (NFW with c200/M200; Einasto with shape parameter alpha) and fractional-deviation summaries
- Baryonic vs dark matter decomposition profiles including non-thermal pressure support estimates
- Output mass and thermodynamic profile plots and goodness-of-fit diagnostics (posterior predictive checks, WAIC)

## How to apply

Install hydromass (from PyPI or github.com/domeckert/hydromass) with its declared dependencies (numpy, scipy, astropy, matplotlib, pymc3, pyproffit) under Python 3.6+, and load the public X-COP data products — X-ray surface brightness, X-ray spectroscopic temperature, and SZ pressure profiles — within a Jupyter notebook. For the nonparametric route, configure the log-normal mixture model in the hydromass.nonparametric module for temperature deprojection and PSF deconvolution of thermodynamic gas profiles, then run the joint Bayesian fit of all three observables under hydrostatic equilibrium using Hamiltonian Monte Carlo sampling via PyMC3. The decision point between nonparametric and parametric reconstruction is the goal: log-normal mixture reconstruction for flexible shape recovery (e.g. testing against NFW/Einasto), or hydromass.mhyd parametric fitting (NFW, Einasto, other models with automatic or custom priors) when specific mass-model parameters such as concentration c200 and M200 are needed. Assess goodness-of-fit with posterior predictive checks and WAIC diagnostics before accepting the reconstruction, then extract mass profiles with posterior uncertainties over the full radial range. For baryonic decomposition, apply hydromass's separation into gas, brightest cluster galaxy, satellite galaxies, and dark matter, using its non-thermal pressure modeling and marginalization for cluster outskirts.

## Related tools

- **hydromass** (Primary package: global Bayesian framework for joint deprojection, PSF deconvolution, and hydrostatic mass reconstruction (nonparametric log-normal mixture and parametric NFW/Einasto models)) — https://github.com/domeckert/hydromass
- **pymc3** (Hamiltonian Monte Carlo backend for the efficient Bayesian optimization of the joint X-ray/SZ fit)
- **pyproffit** (Declared hydromass dependency used for X-ray/SZ profile analysis underlying the reconstruction)
- **numpy** (Declared hydromass dependency for numerical handling of radial profile grids and posterior samples)
- **scipy** (Declared hydromass dependency for scientific computation in the reconstruction)
- **astropy** (Declared hydromass dependency for astronomical units/cosmology handling in mass-profile computations)
- **matplotlib** (Declared hydromass dependency for visualization of output mass and thermodynamic profiles)

## Examples

```
pip install hydromass && python -c "import hydromass; from hydromass import nonparametric"
```

## Evaluation signals

- Posterior predictive checks indicate an adequate joint fit to the observed X-ray surface brightness, spectroscopic temperature, and SZ pressure profiles
- WAIC diagnostics indicate an adequate model before accepting the reconstruction
- Recovered mass profiles are constrained with high precision over two decades in radius for each of the 12 clusters
- Cross-check against NFW: average deviations of the recovered profiles from NFW fits are less than 10% over a wide radial range
- For baryonic decomposition: stellar component dominates inside ~0.02 R500 but accounts for only 1–2% of the total gravitational field inside R200, and the baryon fraction reaches the cosmic value at R200

## Limitations

- The reconstruction assumes hydrostatic equilibrium; the reported excess baryon fraction beyond R200 may indicate mild nonthermal pressure support (10–20%) in cluster outskirts, which must be marginalized via hydromass's non-thermal pressure modeling
- The PyPI release may not be the latest version, so some features may be missing relative to the latest stable state
- The implementation has only been tested on Python 3.6+ under Linux and Mac OS
- The NFW parametric model captures the average shape well but individual cluster profiles show more shape diversity than NFW can capture, so per-cluster residual analysis is warranted

## Evidence

- [readme] Joint X-ray/SZ Bayesian reconstruction: "Joint modeling of X-ray surface brightness, X-ray spectroscopic temperature, and SZ pressure"
- [methods] HMC via PyMC3: "Efficient Bayesian optimization based on Hamiltonian Monte Carlo using PyMC3"
- [methods] Nonparametric log-normal mixture: "Non-parametric temperature deprojection and hydrostatic mass reconstruction using a log-normal mixture model"
- [readme] GoF diagnostics: "Diagnostic tools to investigate goodness-of-fit through posterior predictive checks and WAIC"
- [abstract] NFW deviation threshold: "the NFW profile provides an excellent description of the recovered mass profiles, with deviations of less than 10% over a wide radial range"
