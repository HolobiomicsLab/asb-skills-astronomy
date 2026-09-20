---
name: prf-flux-distribution-modelling
description: Use when a TESS light curve (SPOC PDCSAP or SAP from TPFs/FFIs) may be
  diluted by nearby stars, and you need pixel-by-pixel flux fractions, contaminant
  ranking, CROWDSAP/FLFRCSAP estimates, or to test whether a contaminant could explain
  a transit/activity signal.
when_to_use_negative:
- When high-spatial-resolution follow-up (ground-based photometry/spectroscopy) is
  available, since TESS-cont alone cannot conclusively discard contaminant sources
  for depths below 100%
- When a dilution-corrected contamination metric from the official SPOC pipeline is
  already trusted and no independent crowding check is needed
- When the science question does not involve spatial flux dilution in TESS apertures
  (e.g. purely spectroscopic RV analyses)
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3390
tools:
- name: TESS-cont
  role: 'Main tool: builds PRFs of nearby Gaia sources, computes per-pixel flux contributions
    in the aperture, ranks contaminants, and runs the DILUTION analysis'
  repo: https://github.com/castro-gzlz/TESS-cont
- name: TESS-PRF
  role: Underlying PRF interpolation library used by TESS-cont to model PRF flux distributions
    across TPF/FFI pixels
- name: tpfplotter
  role: Complementary plotting of Gaia sources over TESS TPFs used in the paper's
    contamination workflow
- name: Python
  role: 'Execution environment: TESS-cont is run as ''python TESS-cont.py <config>.ini''
    with dependencies installed via ''pip install -r requirements.txt'''
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202451656
    title: 'TOI-5005 b: A super-Neptune in the savanna near the ridge'
  source_content_hash: sha256:c4c285b7e8cf57c61608135adfbc95e3911712190689de8ac76577368bd30187
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/castro-gzlz/TESS-cont
  tools:
  - Python
  - TESS-PRF
  - TESS-cont
  - tpfplotter
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3390
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# prf-flux-distribution-modelling

## Summary

Models how the PRFs (point response functions) of the target and nearby Gaia DR2/DR3 sources distribute their flux across TESS TPF/FFI pixels, quantifying per-pixel and aperture-integrated contamination with TESS-cont.

## When to use

Use when a TESS light curve (SPOC PDCSAP or SAP from TPFs/FFIs) may be diluted by nearby stars, and you need pixel-by-pixel flux fractions, contaminant ranking, CROWDSAP/FLFRCSAP estimates, or to test whether a contaminant could explain a transit/activity signal.

## When NOT to use

- When high-spatial-resolution follow-up (ground-based photometry/spectroscopy) is available, since TESS-cont alone cannot conclusively discard contaminant sources for depths below 100%
- When a dilution-corrected contamination metric from the official SPOC pipeline is already trusted and no independent crowding check is needed
- When the science question does not involve spatial flux dilution in TESS apertures (e.g. purely spectroscopic RV analyses)

## Inputs

- TESS Target Pixel Files (TPFs) or FFI tesscut cutouts (default cutout size 11,11)
- Gaia DR2 or DR3 source catalogue (default DR3; use DR2 for SPOC PDCSAP analysis)
- Configuration .ini file with target identifier (TIC, TOI, or Simbad name) and TESS sector
- Measured transit depth in the aperture (ppm, ppt, per, or frac) for the DILUTION analysis

## Outputs

- Heatmap of the flux percentage from the target star per pixel
- Pie charts of flux fractions from target and nearby stars inside (and outside) the aperture
- Ranked DILUTION table of implied transit depths for the most contaminant Gaia sources
- CROWDSAP and FLFRCSAP metrics saved in metrics.dat
- Output figures in pdf, png, or pdfpng format

## How to apply

Write an .ini configuration file naming the target (TIC, TOI, or Simbad-resolvable) and the TESS sector, then run 'python TESS-cont.py <config>.ini'. By default TESS-cont uses the Gaia DR3 catalogue with an 'accurate' PRF interpolation to all pixels with Gaia sources (method_prf: accurate), building the PRFs of nearby sources within a 200 arcsec search radius and computing their per-pixel flux distributions across the TPF/FFI; a faster 'approximate' method interpolates the PRF only once at the TPF centre. If analysing SPOC PDCSAP photometry, set gaia_catalog: DR2 because the TIC/SPOC is stacked to Gaia DR2, and ideally run both DR2 and DR3 to check for discrepancies. To test transit-signal origins, populate the DILUTION block with the measured depth (e.g. td = 3471, td_unit = ppm) and dilution_corr: True when the depth is already dilution-corrected by SPOC, then check each contaminant's implied transit depth against the unphysical 100% threshold — depths below 100% mean a contaminant origin cannot be discarded from TESS data alone.

## Related tools

- **TESS-cont** (Main tool: builds PRFs of nearby Gaia sources, computes per-pixel flux contributions in the aperture, ranks contaminants, and runs the DILUTION analysis) — https://github.com/castro-gzlz/TESS-cont
- **TESS-PRF** (Underlying PRF interpolation library used by TESS-cont to model PRF flux distributions across TPF/FFI pixels)
- **tpfplotter** (Complementary plotting of Gaia sources over TESS TPFs used in the paper's contamination workflow)
- **Python** (Execution environment: TESS-cont is run as 'python TESS-cont.py <config>.ini' with dependencies installed via 'pip install -r requirements.txt')

## Examples

```
python TESS-cont.py TOI-4479_S41.ini
```

## Evaluation signals

- For TOI-4479 Sector 41, the DILUTION output must give six implied contaminant depths of 1.74%, 6.10%, 8.256%, 11.30%, 24.35%, and 27.69% for Gaia IDs 1841177816084707584 through 1841177682942388608
- Total aperture contaminant fraction for TOI-4479 S41 should be 30.4%, and for TOI-5005 S65 the target should contribute 84.4% of the aperture photometry
- The CROWDSAP/FLFRCSAP by-products in metrics.dat should show no major differences with the official TESS metrics
- The accurate (default) and approximate PRF methods should give very similar contamination levels; use the accurate method for final analyses
- No implied contaminant transit depth should exceed the unphysical 100% threshold; values in the ~2-28% range mean a contaminant origin cannot be discarded

## Limitations

- The approximate method (method_prf: approximate) interpolates the PRF only once at the TPF centre, assuming the shape does not vary across nearby pixels; use accurate for publications
- Default uses Gaia DR3 while TIC/SPOC PDCSAP is stacked to Gaia DR2 — discrepancies between CROWDSAP/FLFRCSAP and official metrics may arise from DR2/DR3 catalogue differences
- Depths below the 100% threshold mean a contaminant origin cannot be discarded from TESS photometry alone; higher-spatial-resolution observations are needed
- The custom aperture generation feature (e.g. 80% target-flux threshold) is currently undocumented

## Evidence

- [readme] The algorithm is based on building the PRFs of nearby sources and computing their flux distributions across the TESS TPFs/FFIs.: "The algorithm is based on building the PRFs of nearby sources and computing their flux distributions across the TESS TPFs/FFIs."
- [methods] SPOC depth corrected for dilution; DILUTION setup: "a transit depth of 3471 ppm (parts per million) according to [ExoFOP](https://exofop.ipac.caltech.edu/tess/target.php?id=126606859)"
- [readme] By default, *TESS-cont* uses the *Gaia* DR3 catalogue. However, the TIC catalogue (and SPOC PDCSAP) is stacked to *Gaia* DR2.: "By default, *TESS-cont* uses the *Gaia* DR3 catalogue. However, the TIC catalogue (and SPOC PDCSAP) is stacked to *Gaia* DR2."
- [readme] TOI-5005 contributes 84.4% to the measured photometry, while the remaining 15.6% comes from different nearby sources: "**TOI-5005 contributes 84.4%** to the measured photometry, while **the remaining 15.6% comes from different nearby sources**"
- [methods] A **by-product** of the *TESS-cont* operation is the computation of the *CROWDSAP* and *FLFRCSAP* metrics: "A **by-product** of the *TESS-cont* operation is the computation of the *CROWDSAP* and *FLFRCSAP* metrics"
