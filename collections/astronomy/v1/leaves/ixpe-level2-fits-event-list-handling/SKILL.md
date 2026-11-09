---
name: ixpe-level2-fits-event-list-handling
description: Use when the analysis starts from public IXPE level-2 filtered event
  lists downloaded from HEASARC and needs event positions (DETX/DETY) or per-event
  attributes extracted in Python/ixpeobssim for spatial, spectral, or temporal (polarization-aware)
  analysis.
when_to_use_negative:
- Input is not an IXPE FITS event list but already-binned products (e.g., PHA1/PHA1Q/PHA1U
  or PCUBE files) — go straight to binning/fitting tools instead
- 'The goal is raw track-image processing: level-1 FITS files containing individual
  photoelectron track images require a different pipeline than filtered level-2 event
  lists'
- 'The mission is not IXPE and adaptation is required: the README notes the framework
  is only ''potentially straightforward to adapt'' to other missions, so level-2 IXPE
  file handling is not directly applicable'
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: ixpeobssim
  role: Simulation and analysis framework used to read and post-process IXPE level-2
    event FITS lists (DETX/DETY positions) in Python
  repo: https://github.com/lucabaldini/ixpeobssim
- name: IXPE (Imaging X-ray Polarimetry Explorer)
  role: Mission instrument whose HEASARC-released level-2 filtered event lists are
    the input data handled by this skill
- name: HEASOFT
  role: Standard X-ray software ecosystem with which ixpeobssim and the HEASARC IXPE
    data products interoperate
- name: Python
  role: Implementation language for reading the EVENTS extension of the level-2 FITS
    files via ixpeobssim
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.softx.2022.101194
    title: 'ixpeobssim: A simulation and analysis framework for the imaging X-ray
      polarimetry explorer'
  source_content_hash: sha256:9e97adc76a48309e78e87c44d3ec4da5f57e4d9923aa9bab8fc649ca8d959f34
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lucabaldini/ixpeobssim
  tools:
  - HEASOFT
  - IXPE (Imaging X-ray Polarimetry Explorer)
  - Python
  - ixpeobssim
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# ixpe-level2-fits-event-list-handling

## Summary

Handling and ingesting standard publicly released IXPE level-2 filtered event FITS files from HEASARC with ixpeobssim, reading DETX/DETY event positions in the GPD instrument frame for downstream analyses such as the instrumental-background off-axis radial-slope (alpha) fit.

## When to use

Use when the analysis starts from public IXPE level-2 filtered event lists downloaded from HEASARC and needs event positions (DETX/DETY) or per-event attributes extracted in Python/ixpeobssim for spatial, spectral, or temporal (polarization-aware) analysis.

## When NOT to use

- Input is not an IXPE FITS event list but already-binned products (e.g., PHA1/PHA1Q/PHA1U or PCUBE files) — go straight to binning/fitting tools instead
- The goal is raw track-image processing: level-1 FITS files containing individual photoelectron track images require a different pipeline than filtered level-2 event lists
- The mission is not IXPE and adaptation is required: the README notes the framework is only 'potentially straightforward to adapt' to other missions, so level-2 IXPE file handling is not directly applicable

## Inputs

- Public IXPE level-2 filtered event FITS files from the HEASARC archive (one or more celestial observations)
- IXPE EVENTS extension tables with DETX/DETY positions in the GPD instrument frame

## Outputs

- Per-event arrays of DETX/DETY detector-frame positions extracted from the EVENTS extension
- Off-axis radial distributions per observation (e.g., for background-dominated regions)
- CSV of per-observation fitted quantities, e.g., alpha_values.csv with best-fit alpha and uncertainty

## How to apply

Retrieve the level-2 filtered event FITS files for the celestial observations of interest from the HEASARC archive, which distributes the standard IXPE high-level data products (level-1 files contain photoelectron track images, level-2 contain filtered event lists). In Python with ixpeobssim, open the EVENTS extension and read the DETX/DETY event positions in the GPD instrument frame; ixpeobssim event lists are a strict superset of the publicly released data, so all per-event columns are accessible. For background studies, select background-dominated field regions and compute each event's radial distance from the detector center to build the off-axis radial distribution. Keep in mind that instrumental corrections (vignetting, fiducial-area and readout-window event rejection) have already been applied in dedicated filtering stages before the final consolidated event list was written, so no re-filtering is needed. Write per-observation derived quantities (e.g., best-fit alpha with uncertainty) to a CSV such as alpha_values.csv.

## Related tools

- **ixpeobssim** (Simulation and analysis framework used to read and post-process IXPE level-2 event FITS lists (DETX/DETY positions) in Python) — https://github.com/lucabaldini/ixpeobssim
- **IXPE (Imaging X-ray Polarimetry Explorer)** (Mission instrument whose HEASARC-released level-2 filtered event lists are the input data handled by this skill)
- **HEASOFT** (Standard X-ray software ecosystem with which ixpeobssim and the HEASARC IXPE data products interoperate)
- **Python** (Implementation language for reading the EVENTS extension of the level-2 FITS files via ixpeobssim)

## Examples

```
pip install ixpeobssim; then in Python read the EVENTS extension DETX/DETY columns from the HEASARC level-2 event FITS file of each observation with ixpeobssim utilities
```

## Evaluation signals

- The FITS file's EVENTS extension parses correctly in ixpeobssim, yielding DETX/DETY columns in the GPD instrument frame with the expected number of rows
- Events lying outside the fiducial area of the readout chip are already absent (they were discarded during the level-2 filtering stage), so the position distribution stays within the fiducial rectangle
- For background radial analysis, fitted alpha values vary from observation to observation consistent with radiation-environment variations in low-Earth orbit and are typically lower than 0.2
- Per-observation results are recorded row-by-row in the output CSV (e.g., alpha with uncertainty for each observation), and the file schema is consistent across observations

## Limitations

- Instrumental background behavior differs by component: the standard instrumental background is stable over time and shows no significant polarization, whereas solar-flaring background is often sub-threshold and can be persistently strongly polarized — level-2 event handling alone does not disentangle these
- The framework is largely IXPE-specific; adapting it to other missions 'is more than welcome' but is not supported out of the box
- ximpol, the predecessor framework, is no longer developed nor maintained; use ixpeobssim instead

## Evidence

- [abstract] The standard IXPE high-level data products distributed by the HEASARC include: level-1 FITS files containing individual photoelectron track images: "The standard IXPE high-level data products distributed by the HEASARC include: level-1 FITS files containing individual photoelectron track images"
- [abstract] Fiducial-area filtering at level 2: "events with a projected position on the focal plane lying outside the fiducial area of the readout chip are discarded"
- [abstract] Filtering stages before final event list: "All the additional instrumental corrections are applied in dedicated filtering stages before the final, consolidated event list is written"
- [methods] Alpha values typically lower than 0.2: [evidence span withheld — anchor status: not_found]
- [methods] Background stability note: "To our knowledge, this background component is stable over time and shows no significant polarization"
- [readme] ximpol deprecation: [evidence span withheld — anchor status: not_found]
