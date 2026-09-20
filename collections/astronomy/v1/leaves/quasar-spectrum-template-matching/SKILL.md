---
name: quasar-spectrum-template-matching
description: Use when input is generic FITS spectra (e.g. from AAOmega or any spectrograph
  following the Marz format) that need automatic redshifts and correlation peaks,
  particularly for quasars or mixed stellar/galaxy target types, including low signal-to-noise
  data.
when_to_use_negative:
- Input spectra are not in the well-defined Marz FITS input format and cannot be converted
  — Marz 'has since been generalised to take spectra from any spectrograph, given
  it follows a well defined input format'
- Redshifts must be derived from emission-line catalogues alone rather than full-spectrum
  template cross-correlation (use feature-marking/manual approaches instead)
- The task requires nonlinear spectral fitting or PCA-based redshifting of photometric
  data — this skill is strictly a Fourier-space template cross-correlation method
edam_operation: http://edamontology.org/operation_3240
tools:
- name: Marz
  role: Web application and CLI that runs the preprocessed dual-path matching algorithm
    against templates and writes redshift results; invoked via marz.sh for command-line/batch
    use
  repo: https://github.com/samreay/Marz
- name: autoz
  role: Modified autoz cross-correlation algorithm used behind the scenes to match
    input spectra against stellar and galaxy templates in Fourier space
- name: runz
  role: Prior redshifting package used as the performance baseline (54% automatic
    matching on OzDES) that this skill supersedes
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2016.03.001
    title: 'Marz: Manual and automatic redshifting software'
  source_content_hash: sha256:d51899ef4c14809aeb5eb6d15f01b4a48fb298108e262d68c9370ba805bf0458
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/samreay/Marz
  tools:
  - Marz
  - autoz
  - runz
  edam_operation: http://edamontology.org/operation_3240
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# quasar-spectrum-template-matching

## Summary

An automatic spectroscopic redshift pipeline (Marz) that preprocesses FITS spectra along dual general/quasar paths and matches them against twelve stellar and galaxy templates using a modified autoz cross-correlation algorithm, ranking peaks and refining redshifts. It replaced runz, raising automatic matching performance on OzDES spectra from 54% to 91%.

## When to use

Use when input is generic FITS spectra (e.g. from AAOmega or any spectrograph following the Marz format) that need automatic redshifts and correlation peaks, particularly for quasars or mixed stellar/galaxy target types, including low signal-to-noise data.

## When NOT to use

- Input spectra are not in the well-defined Marz FITS input format and cannot be converted — Marz 'has since been generalised to take spectra from any spectrograph, given it follows a well defined input format'
- Redshifts must be derived from emission-line catalogues alone rather than full-spectrum template cross-correlation (use feature-marking/manual approaches instead)
- The task requires nonlinear spectral fitting or PCA-based redshifting of photometric data — this skill is strictly a Fourier-space template cross-correlation method

## Inputs

- Generic FITS spectra files (*.fits), e.g. from the AAOmega spectrograph on the 2dF fibre positioner at the AAT
- Twelve Marz stellar and galaxy templates (from WiggleZ and SDSS template samples)

## Outputs

- Automatic redshift candidates per spectrum, with the ten highest cross-correlation peaks refined by quadratic sub-pixel interpolation
- Heliocentric- and CMB-corrected redshifts written to a per-spectrum CSV file (or saved as .mz files via the web interface)
- Classification/quality-of-fit (QOP) results for merging when QOP 3 and QOP 4 disagree

## How to apply

Load the input FITS spectra (Marz runs on all *.fits files in a specified folder) and preprocess each spectrum along dual general/quasar paths: apply a cosine taper, subtract the continuum via a boxcar-median filter, widen the variance array, and RMS-normalise the result. Cross-correlate each preprocessed spectrum in Fourier space against the twelve Marz stellar and galaxy templates (drawn from the WiggleZ and SDSS template samples) using the modified autoz algorithm, with the corrected normalisation factor of 1 for single-peak results instead of NaN. Sort correlation peaks from all templates together, retain the ten highest correlation value peaks, and apply a quadratic fit around each peak for sub-pixel determination of the local maximum. Apply heliocentric and CMB corrections to the refined redshifts, then write the highest correlation peaks and corrected redshifts per spectrum to a CSV output file (or interactively via the web application where QOP 3/4 disagreements need merging).

## Related tools

- **Marz** (Web application and CLI that runs the preprocessed dual-path matching algorithm against templates and writes redshift results; invoked via marz.sh for command-line/batch use) — https://github.com/samreay/Marz
- **autoz** (Modified autoz cross-correlation algorithm used behind the scenes to match input spectra against stellar and galaxy templates in Fourier space)
- **runz** (Prior redshifting package used as the performance baseline (54% automatic matching on OzDES) that this skill supersedes)

## Examples

```
./marz.sh <FITSFilePath>  # or: ./marz.sh <folder> to batch-analyse all *.fits files in the folder
```

## Evaluation signals

- Automatic matching performance should approach the reported 91% on OzDES spectra (up from runz's 54%)
- Single-peak cross-correlation results must have a normalisation factor of 1, never NaN (the documented xcor normalisation fix)
- Exactly the top 10 correlation peaks are retained and each is locally refined by quadratic sub-pixel interpolation
- Output redshifts must include heliocentric and CMB corrections
- When a folder is given, every *.fits file in it yields a result in the per-spectrum CSV output

## Limitations

- Spectra not matched correctly by the automatic algorithm must be redshifted manually by cycling automatic results, manual template comparison, or marking spectral features
- Redshift extraction must remain robust at low signal-to-noise, which stresses the automatic matcher for faint quasars
- Command-line use requires node.js/io.js installed and npm dependencies installed via install.sh; no version numbers are stated for the external Javascript libraries (AngularJS, Bootstrap, AngularUI, fitsjs) on which Marz depends

## Evidence
