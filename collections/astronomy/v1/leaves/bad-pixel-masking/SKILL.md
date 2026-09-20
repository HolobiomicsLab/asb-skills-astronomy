---
name: bad-pixel-masking
description: Use when preprocessing raw AAOmega-format (or generic FITS) spectra with
  flux, variance, and wavelength extensions before redshift matching, and pixels need
  to be flagged as unreliable.
when_to_use_negative:
- Input is already a cleaned/preprocessed spectrum with no negative or NaN intensity
  values and validated positive variances — re-masking would be a no-op and risks
  discarding valid pixels.
- The goal is the redshift matching or manual classification step itself (cross-correlation,
  template comparison, QOP merging) rather than input-spectrum quality control.
- The input is not a FITS spectrum in the Marz-defined format (e.g. a catalog, image,
  or non-spectral data product).
edam_operation: http://edamontology.org/operation_2928
tools:
- name: Marz
  role: Implements the bad-pixel definition (NaN/negative/over-threshold intensity,
    negative variance) during preprocessing before the modified autoz cross-correlation
    matcher runs; runnable via web app or ./marz.sh CLI on AAOmega FITS files
  repo: https://github.com/samreay/Marz
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2016.03.001
    title: 'Marz: Manual and automatic redshifting software'
  source_content_hash: sha256:d2ae3a4bd3c3da879c020b0c9fb02c3dc6d1cec149055b3c6e86bcbfdc0a7807
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/samreay/Marz
  tools:
  - Marz
  edam_operation: http://edamontology.org/operation_2928
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# bad-pixel-masking

## Summary

Preprocessing step in the Marz AAOmega redshifting pipeline that flags bad pixels in input intensity and variance spectra before the automatic cross-correlation matching algorithm runs, so that NaN, negative, over-threshold, or high-variance-noise pixels cannot corrupt matching.

## When to use

Use when preprocessing raw AAOmega-format (or generic FITS) spectra with flux, variance, and wavelength extensions before redshift matching, and pixels need to be flagged as unreliable.

## When NOT to use

- Input is already a cleaned/preprocessed spectrum with no negative or NaN intensity values and validated positive variances — re-masking would be a no-op and risks discarding valid pixels.
- The goal is the redshift matching or manual classification step itself (cross-correlation, template comparison, QOP merging) rather than input-spectrum quality control.
- The input is not a FITS spectrum in the Marz-defined format (e.g. a catalog, image, or non-spectral data product).

## Inputs

- AAOmega-format FITS spectrum with intensity (flux), variance, and wavelength extensions
- Configurable intensity threshold parameter for over-threshold flagging

## Outputs

- Bad-pixel mask over the spectrum (boolean flag per pixel)
- Cleaned flux and variance arrays carried into cosmic-ray removal and continuum subtraction, written to preprocessed_spectrum.fits

## How to apply

Load the FITS spectrum (flux, variance, wavelength extensions) with astropy.io.fits and apply Marz's bad-pixel rule: a pixel is bad if the intensity is NaN, negative, or exceeds a configurable threshold, or if the variance for that pixel is negative. Flag these pixels in a mask before any continuum fitting or cosmic-ray handling, since the downstream autoz-based cross-correlation matcher assumes only trustworthy pixels contribute. The threshold is configurable rather than fixed, so it should be set per instrument/pipeline. After masking, continue with cosmic-ray detection (second-neighbour test at >30 sigma with 9- and 19-pixel replacement windows) and iterative 6-degree polynomial continuum subtraction (3.5 sigma rejection, up to 15 iterations). Rationale: NaN/negative fluxes and negative variances carry no usable spectral information and can bias peak normalisation in cross-correlation (Marz's xcor normalisation already guards against single-peak NaN results).

## Related tools

- **Marz** (Implements the bad-pixel definition (NaN/negative/over-threshold intensity, negative variance) during preprocessing before the modified autoz cross-correlation matcher runs; runnable via web app or ./marz.sh CLI on AAOmega FITS files) — https://github.com/samreay/Marz

## Examples

```
from astropy.io import fits; hdus = fits.open('example_aatmega.fits'); flux, var = hdus[1].data, hdus[2].data; bad = (~__import__('numpy').isfinite(flux)) | (flux < 0) | (flux > THRESHOLD) | (var < 0)
```

## Evaluation signals

- No unmasked pixels remain in the cleaned flux array that are NaN, negative, or over-threshold, and no negative-variance pixels remain unflagged.
- The preprocessed output FITS file exists, is readable, and contains cleaned flux and variance arrays.
- No residual >30 sigma cosmic-ray spikes survive after masking plus cosmic-ray replacement.
- The residual continuum after 6-degree polynomial subtraction (3.5 sigma rejection, <=15 iterations) is flattened with no systematic trend.
- The automatic matching stage runs without NaN normalisation failures, consistent with Marz's xcor normalisation fix for single-peak results.

## Limitations

- The intensity over-threshold criterion is configurable, so mask completeness depends on choosing an appropriate threshold for the instrument and data quality.
- Masking handles NaN/negative/over-threshold intensity and negative variance only; cosmic rays require the separate second-neighbour test at >30 sigma and are not caught by the basic bad-pixel rule.
- Marz requires FITS input in a well-defined format; spectra deviating from that format are not covered by this rule as stated.
- No version numbers are stated for the external libraries Marz depends on (AngularJS, Bootstrap, AngularUI, fitsjs), so reproduction of the exact preprocessing behaviour across library versions is not guaranteed by the article.

## Evidence

- [other] Marz defines a pixel as bad when the intensity spectrum is NaN, negative, or exceeds a certain configurable threshold, or if the variance spectrum for that pixel is negative.: [evidence span withheld — anchor status: not_found]
- [other] Workflow: load FITS spectrum, apply the bad-pixel rule before cosmic-ray detection and continuum subtraction.: [evidence span withheld — anchor status: not_found]
- [discussion] Marz accepts generic FITS files and can operate on spectra from instruments beyond AAOmega.: "Marz simply requires generic FITS files to work, it can be used for simple FITS file inspection and visualisa- tion, or as a redshifting tool for other surveys"
- [readme] README: Marz takes spectra from the AAOmega spectrograph and has been generalised to any spectrograph following a well-defined input format.: "This project aims to take spectra from the AAOmega spectrograph and redshift them via an online application, for application in cosmology."
- [readme] README: FITS files can be analysed via the command line with node.js using marz.sh.: "you can analyse a FITS file by running `./marz.sh <FITSFilePath>`, which will direct output to `stdout`."
