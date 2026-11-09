---
name: sedm-spectral-rebinning
description: 'Use when a CIGALE-based hyperspectral host model must be matched to
  SEDm low-resolution IFS data: the model cube''s native spectral grid differs from
  the SEDm ~26 A/px sampling and [3700, 9300] A domain and must be rebinned before
  scene-model fitting.'
when_to_use_negative:
- When the target instrument is not SEDm — the [3700, 9300] A domain and ~26 A/px
  binning are specific to the SEDm spectro-spatial observation space
- When no external host redshift is available — HyperGal/CIGALE fitting requires an
  external redshift measurement as input, and the rebinning step depends on the fitted
  model
- When the goal is a posteriori host measurements (e.g. redshift or local Halpha flux)
  from SEDm residuals — the residual host component cannot support such measurements
edam_operation: http://edamontology.org/operation_0337
edam_topics: []
tools:
- name: CIGALE
  role: Produces the per-pixel best-fit SEDs that are rebinned; used as a physically-motivated
    spectral interpolator
- name: HyperGal
  role: Scene modeler that consumes the rebinned hostmodel cube and projects it onto
    the SEDm observation space
  repo: https://github.com/JeremyLezmy/hypergal
- name: PS1 images (Pan-STARRS)
  role: Pre-transient broadband photometric input from which the host model is built
    before rebinning
- name: pysedm
  role: Standard SEDm data-reduction pipeline whose cubes provide the SEDm observation
    space the rebinned model is compared against
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202244740
    title: 'HyperGal: Hyperspectral scene modeling for supernova typing with the SED
      Machine integral field spectrograph'
  source_content_hash: sha256:ee93d1296d2ee4ccccad8f09916d27a078c9a24328957c9b84d30a1f33d07fee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JeremyLezmy/hypergal
  tools:
  - CIGALE
  - HyperGal
  - PS1 images (Pan-STARRS)
  - PanSTARRS-1
  - pysedm
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# sedm-spectral-rebinning

## Summary

Rebin a hyperspectral host-galaxy model cube (derived from per-pixel CIGALE SED fits of pre-transient broadband images) onto the SEDm spectro-spatial observation space, i.e. the ~26 A/px SEDm spectral sampling over the [3700, 9300] A domain, so the host model can be projected and fit against SEDm IFU cubes in HyperGal.

## When to use

Use when a CIGALE-based hyperspectral host model must be matched to SEDm low-resolution IFS data: the model cube's native spectral grid differs from the SEDm ~26 A/px sampling and [3700, 9300] A domain and must be rebinned before scene-model fitting.

## When NOT to use

- When the target instrument is not SEDm — the [3700, 9300] A domain and ~26 A/px binning are specific to the SEDm spectro-spatial observation space
- When no external host redshift is available — HyperGal/CIGALE fitting requires an external redshift measurement as input, and the rebinning step depends on the fitted model
- When the goal is a posteriori host measurements (e.g. redshift or local Halpha flux) from SEDm residuals — the residual host component cannot support such measurements

## Inputs

- Per-pixel CIGALE best-fit SEDs from archival PS1 grizy cutouts (pixels retained only where SNR>3 in all bands)
- Fixed external host-galaxy redshift (e.g. from SDSS, sigma_z ~ 1e-5 for z<0.1)
- Host model cube with native CIGALE spectral sampling

## Outputs

- hostmodel_cube.fits — hyperspectral host model cube rebinned to the ~26 A/px SEDm sampling over [3700, 9300] A
- hostmodel_quality.fits — per-pixel pull and spectral relative RMS quality map

## How to apply

First build the hyperspectral host cube from per-pixel CIGALE best-fit SEDs (sfhdelayed SFH, bc03 populations, nebular emission, dustatt_modified_CF00 attenuation, dale2014 dust emission) fitted to PS1 grizy cutouts with a fixed external host redshift. Then resample the resulting model spectra onto the SEDm instrument grid: truncate to the [3700, 9300] A domain and rebin to the ~26 A/px SEDm sampling, storing the result as hostmodel_cube.fits. This chromatic conversion is what makes the galaxy model a 'physically-motivated spectral interpolator' that can subsequently be projected, together with a point source and diffuse background, onto the SEDm spectro-spatial observation space and adjusted to observations. Quality is tracked per pixel via the spectral relative RMS (Eq. 14) saved as hostmodel_quality.fits.

## Related tools

- **CIGALE** (Produces the per-pixel best-fit SEDs that are rebinned; used as a physically-motivated spectral interpolator)
- **HyperGal** (Scene modeler that consumes the rebinned hostmodel cube and projects it onto the SEDm observation space) — https://github.com/JeremyLezmy/hypergal
- **PS1 images (Pan-STARRS)** (Pre-transient broadband photometric input from which the host model is built before rebinning)
- **pysedm** (Standard SEDm data-reduction pipeline whose cubes provide the SEDm observation space the rebinned model is compared against)

## Examples

```
git clone https://github.com/JeremyLezmy/hypergal.git && cd hypergal && python setup.py install
```

## Evaluation signals

- Output cube wavelength grid matches the ~26 A/px SEDm sampling truncated to [3700, 9300] A
- Median spectral relative RMS (Eq. 14) of the host model is below 3% in the host core
- Spectral relative RMS up to ~10% is tolerated in the outer host regions; larger values indicate a bad per-pixel fit
- All retained input pixels had SNR>3 in every PS1 band (grizy)
- Rebinned cube, plus point-source and diffuse-background components, must reduce residuals when fit to the SEDm cube relative to an isolated point-source extraction

## Limitations

- The most limiting constraint is the need for an external redshift measurement of the host galaxy (~50% of ZTF SN hosts lack spectral redshifts and fall back to SNID estimates with sigma_z ~ 5e-3)
- Residual flux error grows with radius: ~10% in the outer parts vs <3% in the core, reflecting PS1 image depth
- Per-pixel SNR>3 filtering across all bands can drop the outer galaxy regions entirely
- Correction of redshift offset and emission-line fluxes by monochromatic galaxy amplitudes G(lambda) during the 3D fit is only first-order

## Evidence

- [full_text] Convert the best-fit per-pixel SEDs into a hyperspectral host model cube rebinned to the ~26 A/px SEDm sampling truncated to the [3700, 9300] A domain, saved as hostmodel_cube.fits.: [evidence span withheld — anchor status: not_found]
- [abstract] uses pre-transient photometric images to generate a hyperspectral model of the host galaxy; it is based on the CIGALE SED fitter used as a physically-motivated spectral interpolator: "it is based on the CIGALE SED fitter used as a physically-motivated spectral interpolator"
- [abstract] The galaxy model, complemented by a point source and a diffuse background component, is projected onto the SEDm spectro-spatial observation space and adjusted to observations: "The galaxy model, complemented by a point source and a diffuse background component, is projected onto the SEDm spectro-spatial observation space and adjusted to observations"
- [results] The CIGALE SED fit of the PS1 grizy images yields an averaged spectral RMS generally lower than 3% in the core of the galaxy, but reaching about 10% in the outer parts.: [evidence span withheld — anchor status: not_found]
- [readme] HyperGal is a fully chromatic scene modeler, which uses pre-transient photometric images to generate a hyperspectral model of the host galaxy: "HyperGal is a fully chromatic scene modeler, which uses pre-transient photometric images to generate a hyperspectral model of the host galaxy"
