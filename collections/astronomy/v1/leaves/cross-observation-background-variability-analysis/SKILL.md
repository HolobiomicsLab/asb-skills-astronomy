---
name: cross-observation-background-variability-analysis
description: Use when analyzing multiple public IXPE level-2 event files and you need
  per-observation characterization of the instrumental background's off-axis radial
  distribution (slope alpha) for background modeling or subtraction.
when_to_use_negative:
- The analysis region is source-dominated rather than background-dominated — the radial
  alpha fit requires background-only event samples.
- The goal is forward simulation of IXPE observations rather than characterizing the
  background in real archived observations.
- The data are from a different mission — ixpeobssim is IXPE-specific, and adapting
  it to other missions requires modifying the underlying implementation.
edam_operation: http://edamontology.org/operation_3659
tools:
- name: ixpeobssim
  role: Framework used in Python to read DETX/DETY positions from level-2 EVENTS extensions
    and fit the linear radial-dependence model with slope alpha to the off-axis background
    distribution
  repo: https://github.com/lucabaldini/ixpeobssim
- name: Python
  role: Implementation environment for reading event FITS files, computing radial
    distances, and fitting the radial distributions
- name: IXPE (Imaging X-ray Polarimetry Explorer)
  role: Mission whose HEASARC-released level-2 event lists provide the background-dominated
    event samples analyzed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.softx.2022.101194
    title: 'ixpeobssim: A simulation and analysis framework for the imaging X-ray
      polarimetry explorer'
  source_content_hash: sha256:d31b15b219e3602b25ec73c372b16db2d75d33f2d3ea44ada5cc84c9b226f022
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lucabaldini/ixpeobssim
  tools:
  - IXPE (Imaging X-ray Polarimetry Explorer)
  - Python
  - ixpeobssim
  edam_operation: http://edamontology.org/operation_3659
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# cross-observation-background-variability-analysis

## Summary

Quantifies how the IXPE instrumental background varies across observations by fitting the off-axis radial distribution of background-dominated events with a linear radial-dependence model and extracting the fitted slope alpha per observation. It grounds background modeling in the actual radiation-environment variability of low-Earth orbit rather than assuming a uniform background.

## When to use

Use when analyzing multiple public IXPE level-2 event files and you need per-observation characterization of the instrumental background's off-axis radial distribution (slope alpha) for background modeling or subtraction.

## When NOT to use

- The analysis region is source-dominated rather than background-dominated — the radial alpha fit requires background-only event samples.
- The goal is forward simulation of IXPE observations rather than characterizing the background in real archived observations.
- The data are from a different mission — ixpeobssim is IXPE-specific, and adapting it to other missions requires modifying the underlying implementation.

## Inputs

- Public IXPE level-2 filtered event FITS files from the HEASARC archive (multiple celestial observations)
- DETX/DETY event positions read from the EVENTS extension of each level-2 file
- Selection of background-dominated field regions within each observation

## Outputs

- Best-fit radial slope alpha with uncertainty for each observation
- alpha_values.csv with one row per observation (alpha value and uncertainty)
- Per-observation off-axis radial distributions of background events

## How to apply

Retrieve public IXPE level-2 filtered event FITS files for multiple celestial observations from the HEASARC archive. In Python with ixpeobssim, read the DETX/DETY event positions (GPD instrument frame) from the EVENTS extension, selecting only background-dominated field regions of each observation. Compute the radial distance from the detector center for each background event and build the off-axis radial distribution per observation, then fit it with the supported linear radial-dependence model parameterized by the slope alpha — defined as the fractional half-excursion of the variation across the fiducial rectangle size h, where alpha=0 is a uniform distribution and alpha=2 the maximal dependence with zero density at the detector center. Record the best-fit alpha with its uncertainty for each observation into alpha_values.csv. Judgment: the alpha values should vary from observation to observation consistently with radiation-environment variations in low-Earth orbit, and should be typically lower than 0.2 as reported for IXPE celestial observations.

## Related tools

- **ixpeobssim** (Framework used in Python to read DETX/DETY positions from level-2 EVENTS extensions and fit the linear radial-dependence model with slope alpha to the off-axis background distribution) — https://github.com/lucabaldini/ixpeobssim
- **Python** (Implementation environment for reading event FITS files, computing radial distances, and fitting the radial distributions)
- **IXPE (Imaging X-ray Polarimetry Explorer)** (Mission whose HEASARC-released level-2 event lists provide the background-dominated event samples analyzed)

## Examples

```
from astropy.io import fits; ev = fits.open('ixpe_level2_events.fits')['EVENTS'].data; detx, dety = ev['DETX'], ev['DETY']
```

## Evaluation signals

- Fitted alpha values are typically lower than 0.2 across celestial observations, matching the reported IXPE experimental values.
- Fitted alpha varies from observation to observation at a level consistent with radiation-environment variations in low-Earth orbit.
- alpha_values.csv contains one best-fit alpha with uncertainty per observation, with a defined convention check (alpha=0 means uniform distribution, alpha=2 the maximal dependence with zero density at the detector center).
- Radial distributions are built only from background-dominated field regions, verified by the source-free character of the selected DETX/DETY regions.
- Each level-2 input file is a standard HEASARC IXPE event FITS file readable via the EVENTS extension in ixpeobssim.

## Limitations

- The instrumental background component is, to the authors' knowledge, stable over time and shows no significant polarization — a variation large enough to break this assumption would complicate interpretation.
- Solar activity is often sub-threshold but carries counts that are sometimes persistently strongly polarized; such flaring intervals can contaminate the background sample and may need deflaring (e.g., insun/ineclipse subtraction) before fitting.
- The alpha parameterization is tied to the fiducial rectangle size h and the GPD instrument frame, so results are not directly transferable to other detector geometries or missions.

## Evidence

- [methods] Experimental radial slope alpha values are typically lower than 0.2: [evidence span withheld — anchor status: not_found]
- [methods] Background stability: "To our knowledge, this background component is stable over time and shows no significant polarization"
- [abstract] Public data products as inputs: "event lists in FITS format, containing a strict superset of the information included in the publicly released IXPE data"
- [readme] Tool scope: "ixpeobssim is a simulation and analysis framework specifically developed for the Imaging X-ray Polarimetry Explorer (IXPE)"
- [methods] Deflaring caveat: "The two datasets can be used to create a "deflared" dataset by subtraction"
