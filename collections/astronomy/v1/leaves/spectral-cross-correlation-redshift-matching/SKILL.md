---
name: spectral-cross-correlation-redshift-matching
description: Use when you need automatic redshifts and correlation peaks for generic
  FITS spectra (e.g. from the AAOmega spectrograph), covering stellar, galaxy, and
  quasar object types, especially at low signal-to-noise.
when_to_use_negative:
- The spectrum requires human-verified classification (QOP merging) or manual feature
  marking rather than automatic matching — use Marz's manual redshifting interface
  instead
- Input files are not in a well-defined FITS spectral format that fitsjs/Marz can
  parse
- The goal is template catalogue construction rather than redshift measurement of
  input spectra
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Marz
  role: Implements the dual general/quasar preprocessing paths, autoz cross-correlation
    against twelve templates, quadratic sub-pixel peak refinement, and CSV redshift
    output; runnable via web app or CLI
  repo: https://github.com/samreay/Marz
- name: autoz
  role: The underlying cross-correlation algorithm (modified) that matches preprocessed
    input spectra against stellar and galaxy templates in Fourier space
- name: node.js / io.js
  role: Runtime required to execute Marz's command-line matching pipeline via marz.sh
  repo: https://nodejs.org
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2016.03.001
    title: 'Marz: Manual and automatic redshifting software'
  source_content_hash: sha256:8abe6b7e43e1105d4439f96fd0958c9afa8b6a4d56e7418e2b6e45dec54b390b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/samreay/Marz
  tools:
  - Marz
  - autoz
  - node.js / io.js
  edam_operation: http://edamontology.org/operation_0338
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# spectral-cross-correlation-redshift-matching

## Summary

A method for automatically determining redshifts of FITS spectra by cross-correlating continuum-subtracted, RMS-normalised spectra against stellar and galaxy templates in Fourier space using a modified autoz algorithm, as implemented in Marz for the OzDES and 2dFLenS surveys.

## When to use

Use when you need automatic redshifts and correlation peaks for generic FITS spectra (e.g. from the AAOmega spectrograph), covering stellar, galaxy, and quasar object types, especially at low signal-to-noise.

## When NOT to use

- The spectrum requires human-verified classification (QOP merging) or manual feature marking rather than automatic matching — use Marz's manual redshifting interface instead
- Input files are not in a well-defined FITS spectral format that fitsjs/Marz can parse
- The goal is template catalogue construction rather than redshift measurement of input spectra

## Inputs

- FITS spectra files (*.fits) from any spectrograph with a well-defined input format (originally AAOmega spectrograph data)
- Twelve Marz stellar and galaxy templates drawn from the WiggleZ and SDSS template samples

## Outputs

- Per-spectrum CSV output file listing the highest correlation peaks and their heliocentric- and CMB-corrected redshifts
- Top-10 ranked cross-correlation peaks with quadratic sub-pixel refined redshifts
- stdout output when run via the command line interface

## How to apply

Load the input FITS spectra (Marz runs on all *.fits files when a folder is specified) and preprocess each along dual general/quasar paths: apply a cosine taper, subtract the continuum with a boxcar-median filter, widen the variance array, and RMS-normalise the spectrum. Cross-correlate the preprocessed spectrum in Fourier space against the twelve Marz stellar and galaxy templates using the modified autoz algorithm, applying the corrected peak normalisation (factor of 1 for single-peak results instead of NaN). Sort correlation peaks from all templates together, retain the top 10, and refine each peak's redshift via a quadratic fit around the peak for sub-pixel determination of local maxima. Apply heliocentric and CMB corrections to the refined redshifts, then write the highest correlation peaks and their corrected redshifts to a per-spectrum CSV output file. The rationale is that automatic matching replaces runz and raised OzDES automatic matching performance from 54% to 91%.

## Related tools

- **Marz** (Implements the dual general/quasar preprocessing paths, autoz cross-correlation against twelve templates, quadratic sub-pixel peak refinement, and CSV redshift output; runnable via web app or CLI) — https://github.com/samreay/Marz
- **autoz** (The underlying cross-correlation algorithm (modified) that matches preprocessed input spectra against stellar and galaxy templates in Fourier space)
- **node.js / io.js** (Runtime required to execute Marz's command-line matching pipeline via marz.sh) — https://nodejs.org

## Examples

```
./marz.sh spectra_folder/ --output results.csv
```

## Evaluation signals

- Each output spectrum yields a CSV with correlation peaks and corrected redshifts rather than NaN — single-peak results must have a normalisation factor of 1 after the xcor normalisation fix
- Ten highest correlation peaks are retained per spectrum, each refined by quadratic sub-pixel interpolation around the peak
- Automatic matching performance should approach the reported 91% on OzDES spectra (up from 54% with runz)
- Redshifts include heliocentric and CMB corrections; check the corrected values are written to the CSV output
- Command-line run on a folder processes all *.fits files in that folder

## Limitations

- Marz was developed as a client-based JavaScript web-application without significant supporting scientific libraries, as mathematical library support for Javascript was still in its infancy
- Spectra not matched correctly by the automatic algorithm require manual redshifting by cycling automatic results, manual template comparison, or marking spectral features
- No version numbers are stated for external libraries (AngularJS, Bootstrap, AngularUI, fitsjs, DSP package), which complicates reproducibility
- Command-line use requires installing node.js/io.js and npm dependencies

## Evidence

- [abstract] Automatic matching performance improved from 54% (runz) to 91% (Marz): [evidence span withheld — anchor status: not_found]
