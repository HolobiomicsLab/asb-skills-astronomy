---
name: psf-deconvolution-of-gas-profiles
description: Use when you have annular X-ray surface-brightness profiles (optionally
  spectroscopic temperature and SZ pressure) from XMM-Newton/NASA-style observations
  and need PSF-deconvolved thermodynamic profiles as input to hydrostatic mass modeling.
when_to_use_negative:
- Your data are already deprojected thermodynamic profiles — this skill targets raw
  annular surface-brightness/temperature inputs
- Your goal is parametric mass-model fitting only with no thermodynamic-profile deprojection
  step
- You have no PSF information for the instrument, since the deprojection explicitly
  requires PSF deconvolution of thermodynamic gas profiles
tools:
- name: hydromass
  role: Provides the deproject module and global Bayesian framework for deprojection
    and PSF deconvolution of thermodynamic gas profiles
  repo: https://github.com/domeckert/hydromass
- name: pyproffit
  role: Extracts the surface-brightness profile in radial annuli feeding the deprojection
- name: numpy
  role: Dependency for array handling in the deprojection pipeline
- name: scipy
  role: Dependency for numerical routines in the deprojection pipeline
- name: astropy
  role: Dependency for loading/inspecting cluster data products in a Jupyter environment
- name: matplotlib
  role: Visualization of the deprojected thermodynamic profiles
- name: pymc3
  role: Hamiltonian Monte Carlo sampling backend for the Bayesian deprojection
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202142507
    title: The gravitational field of X-COP galaxy clusters
  source_content_hash: sha256:54804be817af5535f34102e700332e9e21ca9c40f2b63024a1617c0a886aa2b3
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
yes_rate: 0.4166666666666667
n_claims: 12
---

# psf-deconvolution-of-gas-profiles

## Summary

Deprojects annular X-ray surface-brightness (and SZ pressure) profiles into thermodynamic gas profiles (electron density, temperature) with PSF deconvolution, within hydromass's global Bayesian framework. This corrects instrument blurring so downstream hydrostatic mass reconstruction uses unbiased profile shapes.

## When to use

Use when you have annular X-ray surface-brightness profiles (optionally spectroscopic temperature and SZ pressure) from XMM-Newton/NASA-style observations and need PSF-deconvolved thermodynamic profiles as input to hydrostatic mass modeling.

## When NOT to use

- Your data are already deprojected thermodynamic profiles — this skill targets raw annular surface-brightness/temperature inputs
- Your goal is parametric mass-model fitting only with no thermodynamic-profile deprojection step
- You have no PSF information for the instrument, since the deprojection explicitly requires PSF deconvolution of thermodynamic gas profiles

## Inputs

- Annular X-ray surface-brightness profiles per cluster (e.g., from pyproffit)
- X-ray spectroscopic temperature measurements in radial annuli
- SZ pressure profiles (Sunyaev-Zel'dovich), optional but used for joint modeling
- PSF characterization of the X-ray instrument (for deconvolution)

## Outputs

- Electron-density radial profiles per cluster
- Deconvolved temperature radial profiles per cluster
- HDF5 files with density and temperature arrays on a common radial grid
- Forward-model predictions of the input surface-brightness data for goodness-of-fit checks

## How to apply

Load the public X-COP data products (annular surface-brightness profiles and spectroscopic temperature measurements) into a Jupyter environment using pyproffit, and extract the surface-brightness profile in radial annuli for each cluster. Run the hydromass deproject module to convert the surface-brightness profile into an electron-density profile, applying PSF deconvolution of the thermodynamic gas profiles as implemented in the package's global Bayesian framework. Combine surface brightness, X-ray spectroscopic temperature, and SZ pressure in the joint modeling setup so the deprojected temperature is consistent with the gas-density solution; for the nonparametric route, the hydromass.nonparametric module performs log-normal mixture temperature deprojection with PSF deconvolution. Save per-cluster density and temperature profiles on a common radial grid via the hydromass save module and inspect them with the package's visualization tools.

## Related tools

- **hydromass** (Provides the deproject module and global Bayesian framework for deprojection and PSF deconvolution of thermodynamic gas profiles) — https://github.com/domeckert/hydromass
- **pyproffit** (Extracts the surface-brightness profile in radial annuli feeding the deprojection)
- **numpy** (Dependency for array handling in the deprojection pipeline)
- **scipy** (Dependency for numerical routines in the deprojection pipeline)
- **astropy** (Dependency for loading/inspecting cluster data products in a Jupyter environment)
- **matplotlib** (Visualization of the deprojected thermodynamic profiles)
- **pymc3** (Hamiltonian Monte Carlo sampling backend for the Bayesian deprojection)

## Examples

```
from hydromass import deproject; deproject(sx=sx_profile, tspec=tspec_data, psf=psf_model)  # annular surface brightness -> PSF-deconvolved electron-density profile
```

## Evaluation signals

- Output HDF5 file exists and contains density and temperature arrays defined on a common radial grid for each cluster
- The deprojected profiles forward-predict the input surface-brightness data (package goodness-of-fit diagnostic)
- Joint fit to surface brightness, spectroscopic temperature, and SZ pressure passes posterior predictive checks and WAIC diagnostics
- PSF-convolved model profiles reproduce the observed (blurred) surface-brightness annuli, not the raw data alone

## Limitations

- The PyPI release may not be the latest version of hydromass, so some features may be missing
- PSF deconvolution quality depends on the accuracy of the instrument PSF model; no independent validation of the PSF kernel is described in the source
- No discussion-section text was available in the source content, so additional limitations or reproducibility caveats could not be extracted

## Evidence

- [readme] global framework for deprojection and mass reconstruction with PSF handling: "A global framework for mass modeling, deprojection and PSF deconvolution of thermodynamic gas profiles"
- [intro] package purpose statement: "A Python package for the reconstruction of hydrostatic mass profiles and deprojection of thermodynamic profile from X-ray and/or Sunyaev-Zeldovich data"
- [methods] nonparametric deprojection route: "Non-parametric temperature deprojection and hydrostatic mass reconstruction using a log-normal mixture model"
- [methods] sampling backend: "Efficient Bayesian optimization based on Hamiltonian Monte Carlo using PyMC3"
- [methods] data loading route: "the user to easily load the public X-COP data products and apply our reconstruction tools directly within a Jupyter notebook"
