---
name: sigma-clipping-rejection
description: Use when preprocessing AAOmega-format FITS spectra (flux, variance, wavelength
  extensions) before the automatic matching algorithm runs, and pixel-level outliers
  such as cosmic rays, NaNs, negative flux/variance values, or a strong continuum
  need masking, replacement, or clipping.
when_to_use_negative:
- The input spectrum does not follow the well-defined generic FITS format Marz expects
  (Marz was generalised beyond AAOmega 'given it follows a well defined input format'),
  e.g. raw 2D detector frames or non-FITS spectral tables
- The goal is the redshift matching/classification step itself (autoz cross-correlation,
  QOP merging) rather than the pixel-level preprocessing that precedes it
- The spectrum already has cosmic rays removed and continuum subtracted, so applying
  the 3.5 sigma / 15-iteration polynomial clipping would over-subtract real spectral
  features
tools:
- name: Marz
  role: Redshifting application whose preprocessing stage implements the bad-pixel
    flagging, >30 sigma cosmic-ray detection with 9/19-pixel replacement windows,
    and 3.5 sigma iterative continuum subtraction before automatic matching
  repo: https://github.com/samreay/Marz
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2016.03.001
    title: 'Marz: Manual and automatic redshifting software'
  source_content_hash: sha256:c370664916fa1b966b3c58ec053c6bdd345c057e55e7f4704b147eeba0a7360a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/samreay/Marz
  tools:
  - Marz
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# sigma-clipping-rejection

## Summary

Preprocessing skill from the Marz redshifting pipeline (Hinton et al. 2016) that iteratively rejects outlier pixels in AAOmega spectra: bad-pixel flagging of NaN/negative/over-threshold intensity and negative variance, >30 sigma second-neighbour cosmic-ray detection, and iterative sigma-clipped polynomial continuum subtraction.

## When to use

Use when preprocessing AAOmega-format FITS spectra (flux, variance, wavelength extensions) before the automatic matching algorithm runs, and pixel-level outliers such as cosmic rays, NaNs, negative flux/variance values, or a strong continuum need masking, replacement, or clipping.

## When NOT to use

- The input spectrum does not follow the well-defined generic FITS format Marz expects (Marz was generalised beyond AAOmega 'given it follows a well defined input format'), e.g. raw 2D detector frames or non-FITS spectral tables
- The goal is the redshift matching/classification step itself (autoz cross-correlation, QOP merging) rather than the pixel-level preprocessing that precedes it
- The spectrum already has cosmic rays removed and continuum subtracted, so applying the 3.5 sigma / 15-iteration polynomial clipping would over-subtract real spectral features

## Inputs

- AAOmega-format FITS spectrum with flux, variance, and wavelength extensions (drag-and-drop FITS file as consumed by Marz)
- Configurable bad-pixel intensity threshold and sigma-clipping parameters (>30 sigma cosmic-ray cut, 3.5 sigma continuum rejection, 15 iterations, 6-degree polynomial)

## Outputs

- Cleaned flux array with bad pixels masked, cosmic rays replaced, and the fitted continuum subtracted
- Cleaned variance array with negative-variance pixels flagged
- preprocessed_spectrum.fits containing the preprocessed flux and variance

## How to apply

Load the AAOmega-format FITS spectrum (flux, variance, wavelength extensions) with astropy.io.fits and first apply the bad-pixel rule: flag pixels as bad where the intensity is NaN, negative, or exceeds a configurable threshold, or where the variance for that pixel is negative. Next, detect cosmic rays with a second-neighbour test at >30 sigma and replace flagged pixels using the 9-pixel and 19-pixel replacement windows. Then fit the continuum with an iterative 6-degree 'rejected polynomial' using 3.5 sigma rejection and up to 15 iterations, and subtract the fitted continuum from the flux. The rationale is that the modified autoz cross-correlation matcher in Marz assumes cleaned, continuum-subtracted spectra, so unmasked outliers or residual continuum degrade automatic redshift matching.

## Related tools

- **Marz** (Redshifting application whose preprocessing stage implements the bad-pixel flagging, >30 sigma cosmic-ray detection with 9/19-pixel replacement windows, and 3.5 sigma iterative continuum subtraction before automatic matching) — https://github.com/samreay/Marz

## Examples

```
from astropy.io import fits; from astropy.stats import sigma_clip; flux = fits.getdata('spectrum.fits', 'flux'); cont = sigma_clip(flux, sigma=3.5, maxiters=15)
```

## Evaluation signals

- The output preprocessed_spectrum.fits exists and is readable with astropy.io.fits
- No NaN, negative, or over-threshold intensity pixels, and no negative-variance pixels, remain unmasked in the cleaned flux array
- No residual >30 sigma cosmic-ray spikes remain in the continuum-subtracted flux
- Masked pixels, replaced cosmic-ray spikes, and the flattened continuum can be verified visually and numerically on the example AAOmega file

## Limitations

- The bad-pixel intensity threshold is configurable rather than fixed, so results depend on the chosen cut-off
- The cosmic-ray replacement is heuristic (9-pixel and 19-pixel windows after a second-neighbour >30 sigma test) and may mis-handle broad or weak spikes
- The 3.5 sigma, up-to-15-iteration rejected-polynomial continuum fit assumes a smooth continuum; strong broad features may be clipped

## Evidence

- [full_text] Bad-pixel definition: [evidence span withheld — anchor status: not_found]
- [full_text] Cosmic-ray removal parameters: [evidence span withheld — anchor status: not_found]
- [full_text] Continuum subtraction parameters: [evidence span withheld — anchor status: not_found]
