---
name: fits-spectrum-loading
description: Use when the input is a generic FITS file containing 1D spectra with
  flux, variance and wavelength extensions (e.g. AAOmega/OzDES data) that must be
  ingested for redshifting or visual inspection.
when_to_use_negative:
- The input is not a FITS file or lacks the expected flux/variance/wavelength extensions
  — Marz requires generic FITS files following a well-defined input format
- The spectra need processing Marz does not provide (e.g. custom instrument-specific
  preprocessing beyond its standard pipeline)
edam_operation: http://edamontology.org/operation_0335
tools:
- name: Marz
  role: Web-application and CLI that ingests FITS spectrum files via drag-and-drop
    or ./marz.sh for redshifting and FITS inspection
  repo: https://github.com/samreay/Marz
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2016.03.001
    title: 'Marz: Manual and automatic redshifting software'
  source_content_hash: sha256:b13784960a608e5033b8d037e332de801b285ee3bc8f190cec94f8e70139d34c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/samreay/Marz
  tools:
  - Marz
  edam_operation: http://edamontology.org/operation_0335
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# fits-spectrum-loading

## Summary

Loads spectroscopic FITS files (flux/intensity, variance, and wavelength extensions, e.g. from the AAOmega spectrograph) into a form suitable for redshifting or inspection with Marz, either via drag-and-drop in the web application or through its command-line interface.

## When to use

Use when the input is a generic FITS file containing 1D spectra with flux, variance and wavelength extensions (e.g. AAOmega/OzDES data) that must be ingested for redshifting or visual inspection.

## When NOT to use

- The input is not a FITS file or lacks the expected flux/variance/wavelength extensions — Marz requires generic FITS files following a well-defined input format
- The spectra need processing Marz does not provide (e.g. custom instrument-specific preprocessing beyond its standard pipeline)

## Inputs

- FITS file with flux/intensity, variance, and wavelength extensions (AAOmega-format or equivalent generic FITS spectra)
- Directory of *.fits files (batch mode)

## Outputs

- Loaded spectrum object ready for automatic template matching in Marz
- Interactive FITS visualization in the Marz web application
- stdout match results (or file output per autoConfig.js parameters) when run via the CLI

## How to apply

Confirm the FITS file follows Marz's well-defined input format with extensions for flux/intensity, variance, and wavelength. For interactive use, drag and drop the FITS file onto the Marz web-application (served via fitsjs), which doubles as a generic FITS viewer; for batch or scripted use, install node.js, run install.sh to fetch npm dependencies, and invoke ./marz.sh on one or many files — a folder argument runs Marz on all *.fits files in it. Marz was built to accept generic FITS files, so files from instruments other than AAOmega are acceptable if they follow the same extension convention; loaded spectra then feed directly into the autoz-based cross-correlation matching against stellar and galaxy templates.

## Related tools

- **Marz** (Web-application and CLI that ingests FITS spectrum files via drag-and-drop or ./marz.sh for redshifting and FITS inspection) — https://github.com/samreay/Marz

## Examples

```
./marz.sh <FITSFilePath>
```

## Evaluation signals

- The FITS file opens without error in Marz (drag-and-drop in browser, or ./marz.sh writes match results to stdout)
- Flux, variance, and wavelength extensions are all read — the loaded spectrum plots with variance overlay rather than flux alone
- In batch mode, every *.fits file in the specified folder is processed rather than only the first
- Output redshifts/templates are produced by the autoz cross-correlation against stellar and galaxy templates, indicating successful ingestion
- CLI output matches the parameters documented in autoConfig.js when run with option flags

## Limitations

- Mathematical library support for Javascript is still in its infancy, so no version numbers are stated for external libraries (AngularJS, Bootstrap, AngularUI, fitsjs); file-size/card limits were relaxed by updating the fitsjs dependency (v1.3.4)
- Non-AAOmega spectra are only supported if they follow Marz's well-defined input format — arbitrary instrument FITS layouts are not auto-detected
- The CLI requires node.js or io.js and npm dependencies installed via install.sh

## Evidence

- [intro] drag and drop a FITS file: [evidence span withheld — anchor status: not_found]
- [readme] If a folder is specified, Marz will run on all *.fits files in that folder.: [evidence span withheld — anchor status: external_doc]
