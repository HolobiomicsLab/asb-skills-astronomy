---
name: hmc-posterior-sampling-for-cluster-profiles
description: Use when reconstructing hydrostatic mass profiles and deprojecting gas
  thermodynamic profiles from X-ray (surface brightness, spectroscopic temperature)
  and/or Sunyaev-Zeldovich pressure data for galaxy clusters.
when_to_use_negative:
- The data lack both X-ray thermodynamic profiles and SZ pressure measurements, so
  hydrostatic equilibrium cannot be constrained jointly
- The goal is a modified-gravity / radial-acceleration-relation test that requires
  baryonic acceleration decomposition beyond standard hydrostatic reconstruction
- A quick parametric fit is needed without deprojection or PSF deconvolution of the
  gas profiles
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: hydromass
  role: Primary package providing the global Bayesian framework for deprojection and
    hydrostatic mass profile reconstruction; load X-COP data products in a Jupyter
    notebook and run HMC-based posterior sampling
  repo: https://github.com/domeckert/hydromass
- name: pymc3
  role: Backend for efficient Bayesian optimization based on Hamiltonian Monte Carlo
- name: pyproffit
  role: Dependency of hydromass for surface brightness/profile analysis
- name: numpy
  role: Dependency of hydromass
- name: scipy
  role: Dependency of hydromass
- name: astropy
  role: Dependency of hydromass
- name: matplotlib
  role: Dependency of hydromass; visualization of output mass and thermodynamic profiles
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202142507
    title: The gravitational field of X-COP galaxy clusters
  source_content_hash: sha256:cbca74b3bb8359248513bb0c5c671eab513191f326e2ceee98924e00ee6018b2
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
yes_rate: 0.4166666666666667
n_claims: 12
---

# hmc-posterior-sampling-for-cluster-profiles

## Summary

Bayesian hydrostatic mass reconstruction for galaxy clusters using Hamiltonian Monte Carlo (PyMC3) via the hydromass package, jointly fitting X-ray surface brightness, spectroscopic temperature, and SZ pressure profiles to recover mass and thermodynamic profiles with posteriors.

## When to use

Use when reconstructing hydrostatic mass profiles and deprojecting gas thermodynamic profiles from X-ray (surface brightness, spectroscopic temperature) and/or Sunyaev-Zeldovich pressure data for galaxy clusters.

## When NOT to use

- The data lack both X-ray thermodynamic profiles and SZ pressure measurements, so hydrostatic equilibrium cannot be constrained jointly
- The goal is a modified-gravity / radial-acceleration-relation test that requires baryonic acceleration decomposition beyond standard hydrostatic reconstruction
- A quick parametric fit is needed without deprojection or PSF deconvolution of the gas profiles

## Inputs

- X-ray surface brightness profiles (e.g. X-COP public data products)
- X-ray spectroscopic temperature profiles
- Sunyaev-Zeldovich pressure profiles
- Chosen mass model specification (NFW, Einasto, custom priors)

## Outputs

- Hydrostatic mass profiles with posterior distributions
- Deprojected thermodynamic gas profiles (PSF-deconvolved)
- Baryonic vs dark matter mass decomposition profiles
- Diagnostic outputs (posterior predictive checks, WAIC)
- Plot-ready mass and thermodynamic profile figures

## How to apply

Load public X-COP data products (X-ray surface brightness, spectroscopic temperature, SZ pressure profiles) within a Jupyter notebook using hydromass. Run the global Bayesian framework for mass modeling, deprojection and PSF deconvolution of thermodynamic gas profiles, with efficient Bayesian optimization based on Hamiltonian Monte Carlo through PyMC3. Choose between parametric mass models (Navarro-Frenk-White, Einasto, others with automatic or custom priors) and non-parametric log-normal mixture temperature deprojection; include non-thermal pressure modeling and marginalization when assessing baryon fractions in cluster outskirts. Assess goodness-of-fit with posterior predictive checks and WAIC, then visualize output mass and thermodynamic profiles with hydromass's plotting utilities.

## Related tools

- **hydromass** (Primary package providing the global Bayesian framework for deprojection and hydrostatic mass profile reconstruction; load X-COP data products in a Jupyter notebook and run HMC-based posterior sampling) — https://github.com/domeckert/hydromass
- **pymc3** (Backend for efficient Bayesian optimization based on Hamiltonian Monte Carlo)
- **pyproffit** (Dependency of hydromass for surface brightness/profile analysis)
- **numpy** (Dependency of hydromass)
- **scipy** (Dependency of hydromass)
- **astropy** (Dependency of hydromass)
- **matplotlib** (Dependency of hydromass; visualization of output mass and thermodynamic profiles)

## Examples

```
pip install hydromass; then within a Jupyter notebook: import hydromass; m = hydromass.MCMCModel(...); m.run()
```

## Evaluation signals

- Recovered mass profiles should be consistent with an NFW description to within ~10% deviations over a wide radial range (article abstract result)
- Average NFW concentration and its scatter should agree with ΛCDM predictions
- Posterior predictive checks and WAIC diagnostics should indicate acceptable goodness-of-fit
- Baryon fraction profile should reach the cosmic value at R200; values beyond R200 exceeding cosmic baryon fraction should correspond to ~10-20% nonthermal pressure support
- For decomposed systems, stellar content should dominate the gravitational field inside ~0.02 R500 and contribute only 1-2% of the total inside R200

## Limitations

- PyPI may not host the latest hydromass version, so some features may be missing
- Tested on Python 3.6+ under Linux and Mac OS; other environments untested
- There is more diversity in the shape of individual cluster mass profiles than the NFW model can capture, so parametric model choice matters
- Hydrostatic equilibrium assumption underpins the reconstruction; excess baryon fraction beyond R200 is attributed to nonthermal pressure support rather than measured directly

## Evidence

- [readme] Efficient Bayesian optimization based on Hamiltonian Monte Carlo using PyMC3: "Efficient Bayesian optimization based on Hamiltonian Monte Carlo using PyMC3"
- [readme] Joint modeling of X-ray surface brightness, X-ray spectroscopic temperature, and SZ pressure: "Joint modeling of X-ray surface brightness, X-ray spectroscopic temperature, and SZ pressure"
- [abstract] The NFW profile provides an excellent description of the recovered mass profiles, with deviations of less than 10% over a wide radial range.: "the NFW profile provides an excellent description of the recovered mass profiles, with deviations of less than 10% over a wide radial range"
- [abstract] The average NFW concentration and its scatter agree very well with the prediction of the ΛCDM framework.: "The average NFW concentration and its scatter agree very well with the prediction of the ΛCDM framework"
- [methods] The PyPI repository should contain the latest stable release (as judged by the developer), it may not be the latest version thus some features may be missing.: "The PyPI repository should contain the latest stable release (as judged by the developer), it may not be the latest version thus some features may be missing."
- [methods] hydromass depends on numpy, scipy, astropy, matplotlib, pymc3, and pyproffit.: "``hydromass`` depends on numpy, scipy, astropy, matplotlib, pymc3, and pyproffit."
