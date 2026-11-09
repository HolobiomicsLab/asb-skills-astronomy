---
name: fits-spectra-ingestion
description: Use when input data are FITS-format spectra (e.g. from the AAOmega spectrograph
  or any instrument matching Marz's well-defined input format) that need to be loaded
  for automatic redshift matching or simple FITS inspection.
when_to_use_negative:
- The spectra are not in Marz's well-defined generic FITS input format — Marz was
  generalised to 'take spectra from any spectrograph, given it follows a well defined
  input format', so non-conforming FITS layouts will not ingest correctly.
- The goal is interactive display or redshifting of a single spectrum with no node.js/Python
  environment available — use the hosted server rather than the CLI path.
edam_operation: http://edamontology.org/operation_2409
tools:
- name: Marz
  role: Web-application and command-line tool that ingests the FITS spectra, preprocesses
    them (cosine taper, boxcar-median continuum subtraction, RMS normalisation) and
    runs the autoz matching
  repo: https://github.com/samreay/Marz
- name: fitsjs
  role: Client-side library used by Marz to read FITS files dragged and dropped into
    the browser interface
- name: node.js
  role: Runtime required to execute the ./marz.sh command-line entry point after install.sh
    installs npm dependencies
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2016.03.001
    title: 'Marz: Manual and automatic redshifting software'
  source_content_hash: sha256:7dcaf4665f72bb0005924724b89a317c7479e8fcac833be9199da03c87dc461e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/samreay/Marz
  tools:
  - Marz
  - autoz
  - fitsjs
  - node.js
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# fits-spectra-ingestion

## Summary

Load generic FITS spectra into the Marz redshifting application so they can be preprocessed and automatically redshifted; Marz accepts any spectrograph's spectra provided they follow a well-defined FITS input format, and supports drag-and-drop, a local Python-served instance, or a node.js command line.

## When to use

Use when input data are FITS-format spectra (e.g. from the AAOmega spectrograph or any instrument matching Marz's well-defined input format) that need to be loaded for automatic redshift matching or simple FITS inspection.

## When NOT to use

- The spectra are not in Marz's well-defined generic FITS input format — Marz was generalised to 'take spectra from any spectrograph, given it follows a well defined input format', so non-conforming FITS layouts will not ingest correctly.
- The goal is interactive display or redshifting of a single spectrum with no node.js/Python environment available — use the hosted server rather than the CLI path.

## Inputs

- FITS-format spectra (*.fits files, e.g. from the AAOmega spectrograph)
- A directory of FITS spectra to batch-process
- Marz repository with npm dependencies installed (node.js/io.js) or a running local Python server

## Outputs

- Loaded, preprocessed spectra available for autoz cross-correlation matching in the Marz web interface or stdout
- Per-spectrum correlation peaks and redshift results (optionally saved to file via marz.sh parameters, e.g. .mz outputs)

## How to apply

Acquire the Samreay/Marz repository and either use the public server at http://samreay.github.io/Marz, start a local server via runServer.bat/runServer.sh (requires Python on PATH), or install node.js and run install.sh to install npm dependencies. For command-line execution, invoke ./marz.sh with one or more FITS file paths, directing output to stdout; when a folder is specified, Marz processes all *.fits files in that folder. Input spectra must follow Marz's well-defined FITS format (e.g. flux, variance and wavelength arrays suitable for continuum subtraction and RMS normalisation). Once loaded, each spectrum is preprocessed with a cosine taper, boxcar-median continuum subtraction, variance widening and RMS normalisation before cross-correlation against the twelve stellar and galaxy templates.

## Related tools

- **Marz** (Web-application and command-line tool that ingests the FITS spectra, preprocesses them (cosine taper, boxcar-median continuum subtraction, RMS normalisation) and runs the autoz matching) — https://github.com/samreay/Marz
- **fitsjs** (Client-side library used by Marz to read FITS files dragged and dropped into the browser interface)
- **node.js** (Runtime required to execute the ./marz.sh command-line entry point after install.sh installs npm dependencies)

## Examples

```
./install.sh && ./marz.sh path/to/spectra_folder/
```

## Evaluation signals

- Marz reports no load errors and displays each FITS file in the interface (drag-and-drop of a FITS file should render it as a viewer of spectroscopic FITS files).
- When a folder is passed, all *.fits files in that folder appear as processed cards, confirming batch ingestion.
- Each ingested spectrum yields correlation peaks after matching, showing preprocessing (taper, continuum subtraction, RMS normalisation) ran without NaN results.
- stdout from ./marz.sh contains the highest correlation peaks and corrected redshifts for each spectrum.
- Peak normalisation behaves as expected: results with only one peak carry a normalisation factor of 1 instead of NaN.

## Limitations

- Marz requires spectra to follow its well-defined input format; FITS files from arbitrary pipelines may need restructuring before ingestion.
- Command-line use requires node.js (or io.js) plus npm dependencies installed via install.sh; local servers require a Python installation on PATH.
- Marz was developed without significant supporting scientific libraries — mathematical library support for Javascript was still in its infancy — which constrained ingestion/processing robustness.

## Evidence

- [intro] The resulting Marz software is a web-application that, thanks to its ease of use (drag and drop a FITS file), may be adaptable as a generic viewer of spectroscopic fits files.: [evidence span withheld — anchor status: not_found]
