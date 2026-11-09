---
name: beam-smearing-psf-convolution
description: Use when fitting 1D rotation curves of IFU-observed high-z disk galaxies
  where beam smearing degrades the observed velocity field, or when generating mock-observed
  rotation/dispersion curves from a parametric mass model at a given sky orientation
  and PSF.
when_to_use_negative:
- Galaxies that are dispersion-dominated or not rotationally supported (V/sigma_0
  <~ 2), or with strong deviations from circular motions — outside the tool's stated
  axisymmetric, rotationally supported regime
- Inclinations outside 10–80 degrees, where RotCurves is stated to be unsuitable
- Marginally resolved systems requiring <~5 per cent accuracy — beam smearing drives
  discrepancies up to ~15 per cent when PSF FWHM >= 1.5 R_eff
edam_operation: http://edamontology.org/operation_0244
tools:
- name: RotCurves
  role: 'Performs the forward-modelling beam projection: galaxy_model/mass_model build
    the mass components, rotation_curve mock-observes the equilibrium rotation velocity
    with the PSF to reconstruct beam-smeared rotation and dispersion curves'
  repo: github.com/nestoramit/RotCurves
- name: dysmalpy
  role: Established reference code used to benchmark the beam-smeared rotation and
    dispersion curves produced by the convolution
- name: emcee
  role: MCMC sampler used to fit the reconstructed beam-smeared rotation curve directly
    to observational data via Bayesian inference
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_001
  source_papers:
  - doi: 10.1093/mnras/stag134
    title: '<tt>RotCurves</tt> : a <tt>PYTHON</tt> package for efficient modelling
      and fitting of galactic rotation curves at high- <i>z</i>'
  source_content_hash: sha256:7c1f54018cbb522cb2c3cafd7c852b3a82b0699c6d1d96bff66513a405f71540
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - RotCurves
  - dysmalpy
  - emcee
  edam_operation: http://edamontology.org/operation_0244
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# beam-smearing-psf-convolution

## Summary

Forward-modelling procedure from the RotCurves package that corrects for beam smearing by projecting and convolving the instrumental beam PSF in the plane of the galaxy, reconstructing a beam-smeared rotation curve from the equilibrium rotation velocity of a disk+bulge+dark-matter-halo mass model.

## When to use

Use when fitting 1D rotation curves of IFU-observed high-z disk galaxies where beam smearing degrades the observed velocity field, or when generating mock-observed rotation/dispersion curves from a parametric mass model at a given sky orientation and PSF.

## When NOT to use

- Galaxies that are dispersion-dominated or not rotationally supported (V/sigma_0 <~ 2), or with strong deviations from circular motions — outside the tool's stated axisymmetric, rotationally supported regime
- Inclinations outside 10–80 degrees, where RotCurves is stated to be unsuitable
- Marginally resolved systems requiring <~5 per cent accuracy — beam smearing drives discrepancies up to ~15 per cent when PSF FWHM >= 1.5 R_eff

## Inputs

- Axisymmetric mass-model configuration with disk, bulge, and dark-matter halo components and analytical density profiles (RotCurves.mass_model / RotCurves.dm_halos)
- Sky orientation parameters (inclination 10–80 degrees, kinematic position angle)
- Instrumental PSF definition (e.g. FWHM relative to disk effective radius R_eff)

## Outputs

- Beam-smeared rotation curve of the fiducial galaxy
- Beam-smeared dispersion curve per galaxy
- Mock-observed curves validated against dysmalpy reference (benchmark_results.csv / beam_smeared_curves.csv)

## How to apply

Assemble an axisymmetric, rotationally supported (V/sigma_0 >~ 2) disk+bulge+dark-matter-halo mass model with selectable analytical density profiles using RotCurves.mass_model, RotCurves.baryons, RotCurves.dm_halos, and RotCurves.galaxy_model, restricting inclination to 10–80 degrees. Compute the equilibrium rotation velocity on the galaxy plane, then mock-observe it with RotCurves.rotation_curve given the sky orientation and instrumental PSF: the projected beam on the plane of the galaxy is simulated and convolved to reconstruct the beam-smeared rotation and dispersion curves, each realization completing in ~10 ms. Keep PSF FWHM <= R_eff for well-resolved systems, where the corrected curves match dysmalpy to within 5 per cent up to 3 R_eff; for marginally resolved systems (PSF FWHM >= 1.5 R_eff) expect discrepancies up to ~15 per cent. The reconstructed beam-smeared curve is then fit directly to observational data via Bayesian inference (MCMC with emcee), which is why beam projection is applied inside the forward model rather than as a data-side correction.

## Related tools

- **RotCurves** (Performs the forward-modelling beam projection: galaxy_model/mass_model build the mass components, rotation_curve mock-observes the equilibrium rotation velocity with the PSF to reconstruct beam-smeared rotation and dispersion curves) — github.com/nestoramit/RotCurves
- **dysmalpy** (Established reference code used to benchmark the beam-smeared rotation and dispersion curves produced by the convolution)
- **emcee** (MCMC sampler used to fit the reconstructed beam-smeared rotation curve directly to observational data via Bayesian inference)

## Examples

```
from rotcurves import galaxy_model, rotation_curve; gm = galaxy_model(mass_model); rot, disp = rotation_curve(gm, sky_orientation, psf)
```

## Evaluation signals

- For well-resolved systems (PSF FWHM < R_eff), the mock-observed rotation and dispersion curves agree with dysmalpy to within 5 per cent up to 3 R_eff
- Marginally resolved configurations (PSF FWHM >= 1.5 R_eff) show fractional differences up to about 15 per cent versus dysmalpy
- A single realization completes in ~10 ms, a factor ~250 faster than dysmalpy (check runtime ratio ~250)
- Output beam_smeared_curves.csv exists and contains rotation and dispersion curves per galaxy
- Recovered model parameters at S/N >~ 25 are consistent with intrinsic values within posterior uncertainties, confirming the forward model is correctly applied inside the fit

## Limitations

- Fitting procedure performs best for resolved systems with beam size the same size or smaller than the disk effective radius (PSF <= R_eff,disk)
- Requires axisymmetric disk galaxies with a well-defined kinematic axis and no strong deviations from circular motions
- Marginally resolved systems show curve discrepancies up to ~15 per cent relative to dysmalpy up to 3 R_eff
- Systematic biases emerge in recovered parameters for the effective radius and for low disc masses (M_disc <~ 3x10^9 M_sun)

## Evidence

- [abstract] Beam projection convolution step: "correcting for ‘beam smearing’ by projecting and convolving the beam PSF in the plane of the galaxy"
- [intro] Reconstruction of the beam-smeared curve: "We simulate the projected beam on the plane of the galaxy (i.e., "beam projection"), reconstructing the beam-smeared rotation curve"
- [readme] Mock observation given sky orientation and PSF: "The equilibrium rotation velocity is mock-observed given the sky-orientation and the instrumental point-spread-function (PSF)."
- [readme] Resolution regime requirement: "The fitting procedure peforms best for resolved systems, with a beam size that is of the same size or smaller than the effective radius of the disk"
- [abstract] Benchmark agreement threshold: "For well-resolved systems (PSF FWHM $< R_{\rm eff}$), the mock observed rotation and dispersion curves agree to within 5 per cent up to $3R_{\rm eff}$"
