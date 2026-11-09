---
name: fourier-domain-signal-processing
description: Use when automatically redshifting one-dimensional FITS spectra (e.g.
  from the AAOmega spectrograph) by cross-correlating continuum-subtracted, RMS-normalised
  spectra against stellar/galaxy templates in Fourier space.
when_to_use_negative:
- The spectrum cannot be matched automatically with sufficient quality and must be
  redshifted manually by cycling automatic results, manual template comparison, or
  marking spectral features
- The input is not a generic FITS spectral file following Marz's well-defined input
  format, so preprocessing (cosine taper, boxcar-median continuum subtraction) cannot
  be applied
- Only a single peak exists and downstream tooling cannot tolerate the xcor-normalisation
  change from NaN to a factor of 1
edam_operation: http://edamontology.org/operation_0438
tools:
- name: Marz
  role: Runs the modified autoz cross-correlation matching pipeline over FITS spectra,
    interactively via web app or via the ./marz.sh command-line interface
  repo: https://github.com/samreay/Marz
- name: autoz
  role: Modified version of the autoz cross-correlation algorithm performing Fourier-space
    template matching inside Marz
- name: kriskowal/q
  role: JavaScript promise library used as an npm dependency for the Marz command-line
    pipeline
  repo: https://github.com/kriskowal/q
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2016.03.001
    title: 'Marz: Manual and automatic redshifting software'
  source_content_hash: sha256:52e34195208ba87daa84a469cca436649dd4346eeb51eca503a71b509a81fb0e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/samreay/Marz
  tools:
  - Marz
  - autoz
  - kriskowal/q
  edam_operation: http://edamontology.org/operation_0438
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# fourier-domain-signal-processing

## Summary

Fourier-space cross-correlation signal processing as implemented in Marz's modified autoz algorithm, which converts preprocessed FITS spectra into ranked redshift candidates against stellar and galaxy templates.

## When to use

Use when automatically redshifting one-dimensional FITS spectra (e.g. from the AAOmega spectrograph) by cross-correlating continuum-subtracted, RMS-normalised spectra against stellar/galaxy templates in Fourier space.

## When NOT to use

- The spectrum cannot be matched automatically with sufficient quality and must be redshifted manually by cycling automatic results, manual template comparison, or marking spectral features
- The input is not a generic FITS spectral file following Marz's well-defined input format, so preprocessing (cosine taper, boxcar-median continuum subtraction) cannot be applied
- Only a single peak exists and downstream tooling cannot tolerate the xcor-normalisation change from NaN to a factor of 1

## Inputs

- Generic FITS spectral files (*.fits) from any spectrograph with a well-defined input format, e.g. AAOmega spectra
- Marz stellar and galaxy template catalogue (derived from WiggleZ and SDSS template samples)
- Variance array accompanying each spectrum

## Outputs

- Ranked cross-correlation peaks (top 10) with quadratic-sub-pixel-refined redshifts per spectrum
- Per-spectrum CSV output file of highest correlation peaks and heliocentric/CMB-corrected redshifts (command-line mode directs output to stdout or file per autoConfig.js)

## How to apply

Load each input FITS spectrum and preprocess it along dual general/quasar paths: apply a cosine taper, subtract the continuum via a boxcar-median filter, widen the variance array, and RMS-normalise the spectrum. Cross-correlate the preprocessed spectrum in Fourier space against the twelve Marz stellar and galaxy templates using the modified autoz algorithm. Apply the corrected cross-correlation peak normalisation so results with only one peak get a normalisation factor of 1 instead of returning NaN. Sort all correlation peaks together, retain the ten highest, and refine each peak's redshift with a quadratic fit around the peak for sub-pixel determination of local maxima, then apply heliocentric and CMB corrections before writing the top peaks and corrected redshifts to per-spectrum output.

## Related tools

- **Marz** (Runs the modified autoz cross-correlation matching pipeline over FITS spectra, interactively via web app or via the ./marz.sh command-line interface) — https://github.com/samreay/Marz
- **autoz** (Modified version of the autoz cross-correlation algorithm performing Fourier-space template matching inside Marz)
- **kriskowal/q** (JavaScript promise library used as an npm dependency for the Marz command-line pipeline) — https://github.com/kriskowal/q

## Examples

```
./marz.sh <FITSFilePath>  # or specify a folder to run on all *.fits files; options via autoConfig.js
```

## Evaluation signals

- Each preprocessed spectrum shows a cosine-tapered, continuum-subtracted (boxcar-median), RMS-normalised profile before correlation, with a widened variance array
- Exactly the ten highest cross-correlation peaks (from all templates, sorted together) are retained per spectrum, each refined by a quadratic sub-pixel fit
- Single-peak results carry a normalisation factor of 1 rather than NaN, confirming the corrected xcor normalisation is in effect
- Output redshifts reflect applied heliocentric and CMB corrections and are written per spectrum to the CSV output file
- Running ./marz.sh on a folder processes all *.fits files in that folder, confirming batch invocation behaves as documented

## Limitations

- The xcor normalisation fix (factor of 1 instead of NaN for single peaks) means downstream code that previously expected NaN handling must be rechecked
- Automatic matching can fail on some spectra; those cases fall back to manual redshifting workflows
- Mathematical library support for Javascript (the platform Marz is built on) is limited, as the paper notes it was developed without significant supporting scientific libraries
- No version numbers are stated for the external JavaScript libraries (AngularJS, Bootstrap, AngularUI, fitsjs, DSP package), complicating reproducibility

## Evidence

- [methods] Peak ranking and sub-pixel refinement: [evidence span withheld — anchor status: not_found]
