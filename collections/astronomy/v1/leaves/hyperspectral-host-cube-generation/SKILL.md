---
name: hyperspectral-host-cube-generation
description: Use when you need a spectrally-resolved host galaxy cube from archival
  broadband photometry (e.g. PS1 grizy cutouts) to serve as a fixed background template
  for IFS scene modeling or transient subtraction.
when_to_use_negative:
- No external host-galaxy redshift is available — HyperGal requires an external redshift
  measurement as a priori input and cannot fit redshift reliably from the photometry
  alone.
- The target is an a posteriori host measurement (e.g. redshift or local Halpha flux
  from residuals) — the residual host component from scene modeling precludes such
  use.
- Host pixels lack sufficient SNR (>3 in all bands) or the galaxy is too faint for
  per-pixel SED fitting to converge meaningfully.
edam_operation: http://edamontology.org/operation_2426
edam_topics: []
tools:
- name: CIGALE
  role: SED fitter run per pixel on multi-band photometry to produce the best-fit
    SED used as a physically-motivated spectral interpolator
- name: HyperGal
  role: Pipeline that consumes the hyperspectral host cube as a fixed chromatic background
    component projected onto the SEDm observation space
  repo: https://github.com/JeremyLezmy/hypergal
- name: PS1 images (Pan-STARRS)
  role: Pre-transient broadband grizy images from which the host galaxy model and
    B_gal background level are estimated
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202244740
    title: 'HyperGal: Hyperspectral scene modeling for supernova typing with the SED
      Machine integral field spectrograph'
  source_content_hash: sha256:f8c9e441805ea3aded839c05c2c0d28f6812c2d4f093f3ff22d268d7a9d7b9af
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JeremyLezmy/hypergal
  tools:
  - CIGALE
  - HyperGal
  - PS1 images (Pan-STARRS)
  - PanSTARRS-1
  edam_operation: http://edamontology.org/operation_2426
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# hyperspectral-host-cube-generation

## Summary

Generates a hyperspectral model cube of a SN host galaxy from pre-transient broadband photometric images (e.g. PS1 grizy) by running CIGALE per pixel as a physically-motivated spectral interpolator, as done in the HyperGal scene-modeling pipeline for SEDm data.

## When to use

Use when you need a spectrally-resolved host galaxy cube from archival broadband photometry (e.g. PS1 grizy cutouts) to serve as a fixed background template for IFS scene modeling or transient subtraction.

## When NOT to use

- No external host-galaxy redshift is available — HyperGal requires an external redshift measurement as a priori input and cannot fit redshift reliably from the photometry alone.
- The target is an a posteriori host measurement (e.g. redshift or local Halpha flux from residuals) — the residual host component from scene modeling precludes such use.
- Host pixels lack sufficient SNR (>3 in all bands) or the galaxy is too faint for per-pixel SED fitting to converge meaningfully.

## Inputs

- Pre-transient multi-band broadband stack cutouts (e.g. PS1 grizy FITS images)
- External host-galaxy redshift measurement (fixed input)
- CIGALE module configuration and parameter grid (sfhdelayed, bc03, nebular, dustatt_modified_CF00, dale2014)

## Outputs

- hostmodel_cube.fits — hyperspectral host model cube rebinned to ~26 A/px over [3700, 9300] A
- hostmodel_quality.fits — per-pixel pull and spectral relative RMS (Eq. 14) quality map

## How to apply

Retrieve pre-transient multi-band stack cutouts (e.g. from the PS1 Image Cutout Server at the transient position) and build per-pixel multi-band photometry, keeping only pixels with SNR>3 in all bands. Run CIGALE per pixel using a physically-motivated module set (sfhdelayed SFH, bc03 stellar populations, nebular emission, dustatt_modified_CF00 attenuation, dale2014 dust emission) over the paper's parameter grid, fixing the host redshift from an external measurement. Rebin the best-fit per-pixel SEDs to the target IFS sampling (~26 A/px for SEDm) truncated to the [3700, 9300] A domain, and save as hostmodel_cube.fits. Compute per-pixel pulls and the spectral relative RMS (Eq. 14) into a quality map (hostmodel_quality.fits) to verify the fit: median relative RMS should be below 3% in the host core, with up to ~10% tolerated in outer regions.

## Related tools

- **CIGALE** (SED fitter run per pixel on multi-band photometry to produce the best-fit SED used as a physically-motivated spectral interpolator)
- **HyperGal** (Pipeline that consumes the hyperspectral host cube as a fixed chromatic background component projected onto the SEDm observation space) — https://github.com/JeremyLezmy/hypergal
- **PS1 images (Pan-STARRS)** (Pre-transient broadband grizy images from which the host galaxy model and B_gal background level are estimated)

## Examples

```
python run_hypergal.py -t ZTF20aamifit
```

## Evaluation signals

- Median spectral relative RMS is below 3% in the host core of the quality map.
- Spectral relative RMS stays within ~10% in the outer host regions.
- hostmodel_cube.fits wavelength axis matches ~26 A/px sampling truncated to [3700, 9300] A.
- Only pixels with SNR>3 in all input bands are fitted; excluded pixels are flagged/absent in the cube.
- Best-fit per-pixel SEDs reproduce the input PS1 grizy photometry within per-pixel pull consistency.

## Limitations

- Requires an external host-galaxy redshift measurement as a priori input — the most limiting constraint of the approach.
- Fit quality degrades from <3% RMS in the core to ~10% in the outer galaxy parts.
- The correction of redshift offsets and emission-line fluxes by monochromatic galaxy amplitudes G(lambda) during the 3D fit depends on the relative distribution of stellar and gaseous components and has not been studied extensively.

## Evidence

- [abstract] uses pre-transient photometric images to generate a hyperspectral model of the host galaxy; it is based on the CIGALE SED fitter used as a physically-motivated spectral interpolator: "uses pre-transient photometric images to generate a hyperspectral model of the host galaxy; it is based on the CIGALE SED fitter used as a physically-motivated spectral interpolator"
- [methods] SNR>3 per-pixel selection and CIGALE module set over Table 1 grid with fixed external redshift: "Since only B gal is estimated from PS1 images, an additional B sky is estimated using two di ff erent sky levels"
- [results] The CIGALE SED fit of the PS1 grizy images yields an averaged spectral RMS generally lower than 3% in the core of the galaxy, but reaching about 10% in the outer parts.: [evidence span withheld — anchor status: not_found]
- [discussion] HyperGal requires external host redshift; G(lambda) interaction with emission lines understudied: "the most limiting constraint from Hyper Gal is the need for an external redshift measurement of the host galaxy"
- [readme] README describes HyperGal as chromatic scene modeler using CIGALE-based host cube: "HyperGal is a fully chromatic scene modeler, which uses pre-transient photometric images to generate a hyperspectral model of the host galaxy; it is based on the CIGALE SED fitter used as a"
