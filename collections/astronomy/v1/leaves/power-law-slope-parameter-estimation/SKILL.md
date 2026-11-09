---
name: power-law-slope-parameter-estimation
description: Use when analyzing background-dominated regions of public IXPE level-2
  event lists (DETX/DETY GPD positions) to characterize the off-axis spatial distribution
  of the instrumental background via the fitted slope alpha.
when_to_use_negative:
- The events are not background-dominated (e.g. source photons dominate the field),
  since the fit is meant to characterize the instrumental background component, which
  is reported as stable over time and showing no significant polarization
- You need a physically calibrated background model rather than an empirical radial-distribution
  characterization of the off-axis background
edam_operation: http://edamontology.org/operation_3799
tools:
- name: ixpeobssim
  role: Python framework used to read IXPE level-2 event FITS files and support the
    radial-distribution fitting analysis
  repo: https://github.com/lucabaldini/ixpeobssim
- name: IXPE (Imaging X-ray Polarimetry Explorer)
  role: Instrument whose GPD DETX/DETY event positions and HEASARC-released level-2
    products provide the input data
- name: Python
  role: Language in which the event reading, radial-distance computation, and model
    fitting are implemented
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.softx.2022.101194
    title: 'ixpeobssim: A simulation and analysis framework for the imaging X-ray
      polarimetry explorer'
  source_content_hash: sha256:14e0ccac701dab6fa3c241fa1e364bdb10d823edec010f4afd9340accd953aa9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lucabaldini/ixpeobssim
  tools:
  - IXPE (Imaging X-ray Polarimetry Explorer)
  - Python
  - ixpeobssim
  edam_operation: http://edamontology.org/operation_3799
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# power-law-slope-parameter-estimation

## Summary

Estimates the radial power-law slope alpha of the instrumental-background off-axis distribution in IXPE GPD event data by fitting the radial event-distance distribution with the linear radial-dependence model, per observation. The slope quantifies how far the background deviates from uniformity across the detector (alpha=0 is uniform, alpha=2 is maximal dependence).

## When to use

Use when analyzing background-dominated regions of public IXPE level-2 event lists (DETX/DETY GPD positions) to characterize the off-axis spatial distribution of the instrumental background via the fitted slope alpha.

## When NOT to use

- The events are not background-dominated (e.g. source photons dominate the field), since the fit is meant to characterize the instrumental background component, which is reported as stable over time and showing no significant polarization
- You need a physically calibrated background model rather than an empirical radial-distribution characterization of the off-axis background

## Inputs

- IXPE level-2 filtered event FITS files from the HEASARC archive (multiple celestial observations)
- DETX/DETY event positions from the EVENTS extension of each event file, restricted to background-dominated field regions

## Outputs

- Per-observation best-fit radial slope alpha with its uncertainty
- alpha_values.csv — a table with one row per observation containing the fitted alpha value and uncertainty

## How to apply

Retrieve public IXPE level-2 filtered event FITS files from HEASARC for multiple celestial observations, and read the DETX/DETY event positions (GPD instrument frame) from the EVENTS extension for background-dominated field regions of each observation. For each observation, compute the radial distance of each background event from the detector center and build the off-axis radial distribution. Fit each radial distribution with the supported linear radial-dependence model parameterized by the slope alpha — defined as the fractional half-excursion of the variation across the fiducial rectangle size h, where alpha=0 corresponds to a uniform distribution and alpha=2 to the maximal dependence with zero density at the detector center. Record the best-fit alpha value with its uncertainty for each observation and write all rows to alpha_values.csv.

## Related tools

- **ixpeobssim** (Python framework used to read IXPE level-2 event FITS files and support the radial-distribution fitting analysis) — https://github.com/lucabaldini/ixpeobssim
- **IXPE (Imaging X-ray Polarimetry Explorer)** (Instrument whose GPD DETX/DETY event positions and HEASARC-released level-2 products provide the input data)
- **Python** (Language in which the event reading, radial-distance computation, and model fitting are implemented)

## Examples

```
pip install ixpeobssim
```

## Evaluation signals

- Fitted alpha values fall below 0.2 for the observations analyzed, matching the reported range for IXPE celestial observations
- Fitted alpha values vary from observation to observation at a level consistent with radiation-environment variations in low Earth orbit
- alpha_values.csv contains one row with an alpha value and uncertainty per processed observation
- alpha=0 is recovered for a spatially uniform test distribution and alpha=2 corresponds to maximal radial dependence with zero density at the detector center

## Limitations

- The instrumental background is described as stable over time and showing no significant polarization, but the fitted alpha is expected to vary across observations due to radiation-environment variations in low Earth orbit
- The fit must be performed on background-dominated regions; source-contaminated fields will bias the radial distribution
- Solar flaring background can be sub-threshold and persistently strongly polarized, so it is a distinct background component from the one characterized here

## Evidence

- [methods] Experimental radial slope alpha values are typically lower than 0.2: "and are typically lower than 0.2"
- [methods] Background component stability and polarization: "To our knowledge, this background component is stable over time and shows no significant polarization"
- [abstract] ixpeobssim handles IXPE event FITS data: "it is designed to produce realistic simulated observations, in the form of event lists in FITS format"
- [readme] ixpeobssim is the analysis framework for IXPE: "ixpeobssim is a simulation and analysis framework specifically developed for the Imaging X-ray Polarimetry Explorer (IXPE)."
