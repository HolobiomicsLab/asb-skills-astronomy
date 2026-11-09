---
name: xray-sz-joint-bayesian-profile-modeling
description: Use when you have X-ray surface brightness, spectroscopic temperature,
  and/or SZ pressure profiles for a galaxy cluster and need hydrostatic mass or thermodynamic
  profile reconstruction with posterior uncertainties.
when_to_use_negative:
- When only one data set (e.g. X-ray only) is available with no SZ pressure constraint,
  since the skill is designed around joint X-ray/SZ modeling — hydromass does support
  single-probe deprojection, but the two-decade-radius precision claimed here relies
  on deep X-ray plus SZ data
- When hydrostatic equilibrium is known to be strongly violated (e.g. major mergers),
  as the reconstruction assumes hydrostatic equilibrium; mild nonthermal pressure
  support of 10–20% in cluster outskirts was flagged as a caveat
- When the goal is measuring the radial acceleration relation, since this paper shows
  the observed-vs-baryonic acceleration relation strongly departs from the RAR of
  spiral galaxies at cluster scale
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3452
tools:
- name: hydromass
  role: Python package providing the global Bayesian framework for joint X-ray/SZ
    profile modeling, deprojection, PSF deconvolution, and hydrostatic mass reconstruction
  repo: https://github.com/domeckert/hydromass
- name: pymc3
  role: Hamiltonian Monte Carlo backend for the Bayesian posterior sampling of the
    joint profile model
- name: pyproffit
  role: hydromass dependency used for X-ray profile analysis
- name: numpy
  role: declared hydromass dependency for numerical computation
- name: scipy
  role: declared hydromass dependency for scientific computation
- name: astropy
  role: declared hydromass dependency for astronomical units/coordinates
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202142507
    title: The gravitational field of X-COP galaxy clusters
  source_content_hash: sha256:38c184a4c2f50d88f4571b880b876b72764b7432340d6a1d52796ff698329d25
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
yes_rate: 0.5
n_claims: 12
---

# xray-sz-joint-bayesian-profile-modeling

## Summary

Jointly fits X-ray surface brightness, X-ray spectroscopic temperature, and Sunyaev–Zel'dovich pressure profiles in a global Bayesian framework (hydromass) to reconstruct hydrostatic mass and thermodynamic profiles of galaxy clusters, including nonparametric log-normal mixture temperature deprojection with PSF deconvolution.

## When to use

Use when you have X-ray surface brightness, spectroscopic temperature, and/or SZ pressure profiles for a galaxy cluster and need hydrostatic mass or thermodynamic profile reconstruction with posterior uncertainties.

## When NOT to use

- When only one data set (e.g. X-ray only) is available with no SZ pressure constraint, since the skill is designed around joint X-ray/SZ modeling — hydromass does support single-probe deprojection, but the two-decade-radius precision claimed here relies on deep X-ray plus SZ data
- When hydrostatic equilibrium is known to be strongly violated (e.g. major mergers), as the reconstruction assumes hydrostatic equilibrium; mild nonthermal pressure support of 10–20% in cluster outskirts was flagged as a caveat
- When the goal is measuring the radial acceleration relation, since this paper shows the observed-vs-baryonic acceleration relation strongly departs from the RAR of spiral galaxies at cluster scale

## Inputs

- X-ray surface brightness profile
- X-ray spectroscopic temperature profile
- Sunyaev–Zel'dovich pressure profile
- Public X-COP data products for the 12-cluster sample

## Outputs

- Hydrostatic gravitational-field (mass) profiles with posterior uncertainties over the full radial range
- Deprojected, PSF-deconvolved thermodynamic gas profiles
- Posterior predictive checks and WAIC goodness-of-fit diagnostics
- Output mass and thermodynamic profile plots

## How to apply

Install hydromass (Python 3.6+, dependencies numpy, scipy, astropy, matplotlib, pymc3, pyproffit) and load the public X-COP data products — X-ray surface brightness, X-ray spectroscopic temperature, and SZ pressure profiles — into a Jupyter notebook. Configure the joint Bayesian model of the three profiles under the hydrostatic-equilibrium assumption; for nonparametric reconstruction use the hydromass.nonparametric log-normal mixture model for temperature deprojection with PSF deconvolution of the thermodynamic gas profiles. Sample the posterior with Hamiltonian Monte Carlo via PyMC3, and optionally decompose the hydrostatic mass profile into baryonic (gas, BCG, satellite galaxies) and dark-matter components or apply parametric NFW/Einasto mass models with automatic or custom priors. Assess goodness-of-fit with posterior predictive checks and WAIC before accepting the recovered mass and thermodynamic profiles with posterior uncertainties over the full radial range.

## Related tools

- **hydromass** (Python package providing the global Bayesian framework for joint X-ray/SZ profile modeling, deprojection, PSF deconvolution, and hydrostatic mass reconstruction) — https://github.com/domeckert/hydromass
- **pymc3** (Hamiltonian Monte Carlo backend for the Bayesian posterior sampling of the joint profile model)
- **pyproffit** (hydromass dependency used for X-ray profile analysis)
- **numpy** (declared hydromass dependency for numerical computation)
- **scipy** (declared hydromass dependency for scientific computation)
- **astropy** (declared hydromass dependency for astronomical units/coordinates)

## Examples

```
pip install hydromass; from hydromass import nonparametric; # load X-COP X-ray SB, spectroscopic temperature, SZ pressure profiles and run the joint Bayesian HMC fit via PyMC3
```

## Evaluation signals

- Posterior predictive checks indicate an adequate joint fit to the observed X-ray and SZ profiles
- WAIC diagnostics indicate an adequate joint fit before accepting the reconstruction
- Recovered mass profile uncertainties are reported over the full radial range (two decades in radius for X-COP)
- Nonparametric mass profiles deviate from the NFW form by less than 10% over a wide radial range, on average
- Average NFW concentration and its scatter agree with ΛCDM predictions

## Limitations

- The PyPI release may not be the latest version, so some features may be missing; the exact version of hydromass used is not stated
- Reconstruction assumes hydrostatic equilibrium; the baryon fraction slightly exceeding the cosmic value beyond R200 possibly indicates mild nonthermal pressure support (10–20%) in cluster outskirts
- Individual mass profiles show more shape diversity than the NFW model can capture
- Tested on Python 3.6+ under Linux and Mac OS only

## Evidence

- [intro] A Python package for the reconstruction of hydrostatic mass profiles and deprojection of thermodynamic profile from X-ray and/or Sunyaev-Zeldovich data: "A Python package for the reconstruction of hydrostatic mass profiles and deprojection of thermodynamic profile from X-ray and/or Sunyaev-Zeldovich data"
- [methods] Non-parametric temperature deprojection and hydrostatic mass reconstruction using a log-normal mixture model: "Non-parametric temperature deprojection and hydrostatic mass reconstruction using a log-normal mixture model"
- [readme] Joint modeling of X-ray surface brightness, X-ray spectroscopic temperature, and SZ pressure: "Joint modeling of X-ray surface brightness, X-ray spectroscopic temperature, and SZ pressure"
- [abstract] The NFW profile provides an excellent description of the recovered mass profiles, with deviations of less than 10% over a wide radial range.: "the NFW profile provides an excellent description of the recovered mass profiles, with deviations of less than 10% over a wide radial range"
- [readme] The current implementation has been developed in Python 3 and tested on Python 3.6+ under Linux and Mac OS.: "The current implementation has been developed in Python 3 and tested on Python 3.6+ under Linux and Mac OS."
