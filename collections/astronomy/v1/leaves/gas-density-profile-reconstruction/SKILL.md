---
name: gas-density-profile-reconstruction
description: Use when you have radial/annular X-ray surface-brightness profiles (e.g.
  extracted with pyproffit) and need to deproject them into electron-density and thermodynamic
  profiles as input to hydrostatic mass reconstruction from X-ray and/or SZ data.
when_to_use_negative:
- Input is already a deprojected 3D density/temperature profile — deprojection would
  be redundant; use hydromass mass-model fitting directly instead.
- No X-ray surface-brightness or SZ data are available (e.g. only weak-lensing shear
  catalogs) — the deprojection is defined for X-ray and/or Sunyaev-Zeldovich observations.
- You need purely 2D image analysis without annular binning — the skill requires radial
  annular profiles as extracted by pyproffit.
tools:
- name: hydromass
  role: Provides the deproject module that converts the surface-brightness profile
    into an electron-density profile with PSF deconvolution, plus save and plotting
    modules for the output profiles
  repo: https://github.com/domeckert/hydromass
- name: pyproffit
  role: Extracts the surface-brightness profile in radial annuli for each cluster,
    the direct input to the hydromass deprojection
- name: pymc3
  role: Backend for the efficient Bayesian optimization based on Hamiltonian Monte
    Carlo used in the global deprojection framework
- name: astropy
  role: Used alongside pyproffit to load the public X-COP data products into the Jupyter
    environment
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202142507
    title: The gravitational field of X-COP galaxy clusters
  source_content_hash: sha256:7a8f32d84104a0265241fb214c81626742787c985c32fa618a0a7e6ab3ca851a
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# gas-density-profile-reconstruction

## Summary

Deprojects annular X-ray surface-brightness profiles (and optionally SZ pressure and spectroscopic temperature) into electron-density and thermodynamic gas profiles using the hydromass package's global Bayesian framework, including PSF deconvolution. This deprojection step supplies the density input needed for hydrostatic mass-profile reconstruction of galaxy clusters.

## When to use

Use when you have radial/annular X-ray surface-brightness profiles (e.g. extracted with pyproffit) and need to deproject them into electron-density and thermodynamic profiles as input to hydrostatic mass reconstruction from X-ray and/or SZ data.

## When NOT to use

- Input is already a deprojected 3D density/temperature profile — deprojection would be redundant; use hydromass mass-model fitting directly instead.
- No X-ray surface-brightness or SZ data are available (e.g. only weak-lensing shear catalogs) — the deprojection is defined for X-ray and/or Sunyaev-Zeldovich observations.
- You need purely 2D image analysis without annular binning — the skill requires radial annular profiles as extracted by pyproffit.

## Inputs

- Annular X-ray surface-brightness profiles in radial annuli (extracted per cluster with pyproffit)
- X-ray spectroscopic temperature measurements per annulus
- Sunyaev-Zeldovich pressure profiles (optional joint input)
- Public X-COP data products (annular surface-brightness profiles and spectroscopic temperature measurements)

## Outputs

- Per-cluster deprojected electron-density radial profiles
- Deconvolved (PSF-corrected) thermodynamic temperature profiles consistent with the density solution
- HDF5 file containing density and temperature arrays defined on a common radial grid for each cluster
- Goodness-of-fit diagnostics (forward prediction of the input surface-brightness data, posterior predictive checks, WAIC)

## How to apply

First use pyproffit to extract the surface-brightness profile in radial annuli for each cluster, loading the public X-COP data products (annular surface-brightness profiles and spectroscopic temperature measurements) into a Jupyter environment with pyproffit and astropy. Then run the hydromass deproject module to convert the surface-brightness profile into an electron-density profile, applying PSF deconvolution of the thermodynamic gas profiles within hydromass's global Bayesian framework (Hamiltonian Monte Carlo via PyMC3). Combine the surface-brightness, X-ray spectroscopic temperature, and SZ pressure inputs in the joint modeling setup so the deprojected temperature profile is consistent with the gas density solution. Save the per-cluster density and temperature radial profiles to disk with the hydromass save module (HDF5 output) and inspect them with the package's visualization tools. Decision point: choose between parametric forward/polytropic fitting and the non-parametric log-normal mixture reconstruction depending on whether you want a smooth parametric profile or a flexible deprojection.

## Related tools

- **hydromass** (Provides the deproject module that converts the surface-brightness profile into an electron-density profile with PSF deconvolution, plus save and plotting modules for the output profiles) — https://github.com/domeckert/hydromass
- **pyproffit** (Extracts the surface-brightness profile in radial annuli for each cluster, the direct input to the hydromass deprojection)
- **pymc3** (Backend for the efficient Bayesian optimization based on Hamiltonian Monte Carlo used in the global deprojection framework)
- **astropy** (Used alongside pyproffit to load the public X-COP data products into the Jupyter environment)

## Evaluation signals

- Verify the output HDF5 file exists and contains density and temperature arrays defined on a common radial grid for each cluster.
- Check that the deprojected profiles forward-predict the input surface-brightness data (package goodness-of-fit diagnostic).
- Run the package's posterior predictive checks and WAIC diagnostics to confirm goodness-of-fit.
- Confirm joint consistency: the deprojected temperature profile must be consistent with the gas density solution in the joint SB + spectroscopic temperature + SZ pressure model.
- Confirm the hydromass dependency stack (numpy, scipy, astropy, matplotlib, pymc3, pyproffit) is installed and Python 3.6+ is used.

## Limitations

- The PyPI release may not be the latest version, so some features may be missing.
- The implementation has been tested only on Python 3.6+ under Linux and Mac OS.
- No discussion-section limitations or changelog were available in the source content ('_No changelog found._'), so further reproducibility caveats could not be extracted.

## Evidence

- [methods] A global framework for mass modeling, deprojection and PSF deconvolution of thermodynamic gas profiles: "A global framework for mass modeling, deprojection and PSF deconvolution of thermodynamic gas profiles"
- [methods] ``hydromass`` depends on numpy, scipy, astropy, matplotlib, pymc3, and pyproffit.: "``hydromass`` depends on numpy, scipy, astropy, matplotlib, pymc3, and pyproffit."
- [methods] the user to easily load the public X-COP data products and apply our reconstruction tools directly within a Jupyter notebook: "the user to easily load the public X-COP data products and apply our reconstruction tools directly within a Jupyter notebook"
- [readme] deprojection of thermodynamic profile from X-ray and/or Sunyaev-Zeldovich data: "A Python package for the reconstruction of hydrostatic mass profiles and deprojection of thermodynamic profile from X-ray and/or Sunyaev-Zeldovich data."
- [readme] Diagnostic tools to investigate goodness-of-fit through posterior predictive checks and WAIC: "Diagnostic tools to investigate goodness-of-fit through posterior predictive checks and WAIC"
