---
name: iterative-polynomial-continuum-subtraction
description: Use when preparing AAOmega-format FITS spectra (flux + variance + wavelength
  extensions) for redshift cross-correlation, and the continuum must be removed so
  matching operates on emission/absorption features rather than the continuum shape.
when_to_use_negative:
- The input is not a 1D FITS spectrum needing redshift estimation — e.g. it is already
  continuum-normalised or is an image cube
- The spectrum will be matched by methods that require the absolute flux level or
  continuum shape preserved (e.g. photometric calibration checks)
- Spectral features of interest are very broad (> polynomial curvature scale), where
  continuum subtraction would absorb real signal
tools:
- name: Marz
  role: Provides the preprocessing pipeline (bad-pixel flagging, cosmic-ray removal,
    iterative polynomial continuum fitting and subtraction) applied before its modified
    autoz cross-correlation matching; usable interactively or via the command line
    on FITS files
  repo: https://github.com/samreay/Marz
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2016.03.001
    title: 'Marz: Manual and automatic redshifting software'
  source_content_hash: sha256:b28dfdd494d7e979003d3b9cec6e1783e7ec098756e180bc581b90dd5134fc8e
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
yes_rate: 1.0
n_claims: 7
---

# iterative-polynomial-continuum-subtraction

## Summary

Preprocessing step from the Marz redshifting pipeline for AAOmega spectra: after bad-pixel flagging and cosmic-ray replacement, an iterative 6-degree rejected polynomial (3.5 sigma rejection, up to 15 iterations) is fitted to the flux and subtracted to flatten the continuum before automatic template matching runs.

## When to use

Use when preparing AAOmega-format FITS spectra (flux + variance + wavelength extensions) for redshift cross-correlation, and the continuum must be removed so matching operates on emission/absorption features rather than the continuum shape.

## When NOT to use

- The input is not a 1D FITS spectrum needing redshift estimation — e.g. it is already continuum-normalised or is an image cube
- The spectrum will be matched by methods that require the absolute flux level or continuum shape preserved (e.g. photometric calibration checks)
- Spectral features of interest are very broad (> polynomial curvature scale), where continuum subtraction would absorb real signal

## Inputs

- AAOmega-format FITS spectrum with flux, variance, and wavelength extensions
- Configurable bad-pixel intensity threshold
- Polynomial degree (6), rejection sigma (3.5), and maximum iterations (15)

## Outputs

- Continuum-subtracted (flattened) flux array
- Cleaned flux and variance arrays written to a preprocessed FITS file (e.g. preprocessed_spectrum.fits)

## How to apply

Load the AAOmega FITS spectrum (flux, variance, wavelength extensions) and first flag bad pixels where intensity is NaN, negative, or over a configurable threshold, or where variance is negative. Detect cosmic rays with the second-neighbour test at >30 sigma and replace flagged pixels using the 9-pixel and 19-pixel replacement windows. Then fit a 6-degree polynomial to the cleaned flux with iterative 3.5 sigma rejection, running up to 15 iterations, so that emission/absorption lines and residual outliers do not bias the continuum estimate. Subtract the fitted continuum from the flux so the input to the modified autoz cross-correlation matching is continuum-flattened. The rationale is that cross-correlation against stellar and galaxy templates is sensitive to continuum shape, and OzDES/2dFLenS spectra can be low signal-to-noise, so robust rejection-based fitting is required.

## Related tools

- **Marz** (Provides the preprocessing pipeline (bad-pixel flagging, cosmic-ray removal, iterative polynomial continuum fitting and subtraction) applied before its modified autoz cross-correlation matching; usable interactively or via the command line on FITS files) — https://github.com/samreay/Marz

## Examples

```
from astropy.io import fits; hdul = fits.open('example_AAOmega.fits'); flux, variance = hdul[1].data, hdul[2].data  # then fit 6-deg poly with 3.5 sigma rejection, <=15 iters, and subtract
```

## Evaluation signals

- Residual continuum of the subtracted spectrum is flat, with no remaining large-scale trend across the wavelength range
- No >30 sigma cosmic-ray spikes remain in the cleaned flux after replacement and subtraction
- All pixels with NaN, negative, or over-threshold intensity, and all negative-variance pixels, are masked or excluded from the polynomial fit
- The output FITS file exists, is readable with astropy.io.fits, and contains the cleaned flux and variance extensions
- The polynomial fit converges within the 15-iteration limit and the 3.5 sigma rejection set stabilises

## Limitations

- Bad-pixel intensity threshold is configurable, so behaviour depends on chosen settings and must be checked per dataset
- Marz was built for generic FITS input in a specific AAOmega format; spectra not following the defined input format are not supported
- Rejection-based polynomial fitting can misbehave at very low signal-to-noise; Marz's matching is reported to perform well at both high and low S/N but continuum estimates may still be noisy
- No version numbers stated for external libraries used in the Marz implementation, so reproducing exact preprocessing behaviour across library versions is not guaranteed

## Evidence

- [other] Marz defines a pixel as bad when the intensity spectrum is NaN, negative, or exceeds a certain configurable threshold, or if the variance spectrum for that pixel is negative.: [evidence span withheld — anchor status: not_found]
- [other] Fit the continuum with an iterative 6-degree rejected polynomial using 3.5 sigma rejection and up to 15 iterations, then subtract the fitted continuum from the flux.: [evidence span withheld — anchor status: not_found]
- [other] Detect cosmic rays with the second-neighbour test at >30 sigma and replace flagged pixels using the 9-pixel and 19-pixel replacement windows.: [evidence span withheld — anchor status: not_found]
