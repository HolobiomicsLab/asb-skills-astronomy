---
name: tess-pixel-response-function-modelling
description: Use when quantifying what fraction of flux in a TESS SPOC aperture (or
  FFI tesscut) comes from the target star versus nearby Gaia contaminants, or when
  judging whether a transit/activity signal could originate in a contaminant.
when_to_use_negative:
- The photometry is not from TESS (no TPF/FFI data available), since the PRF modelling
  is specific to TESS pixels
- 'You need a definitive origin attribution for a transit signal: dilution depths
  below the 100% unphysical threshold only rule contaminants in, not out — higher-spatial-resolution
  ground-based photometry/spectroscopy is required'
- Only a rough first-look contamination estimate is acceptable and runtime is critical
  — even then, approximate method_prf results should not be used for final analyses
  or publications
tools:
- name: TESS-cont
  role: Python driver that identifies Gaia contaminant sources, builds PRFs, computes
    aperture flux fractions, and renders the heatmap/pie-chart outputs
  repo: https://github.com/castro-gzlz/TESS-cont
- name: tpfplotter
  role: Used by TESS-cont to access TESS TPFs and overlay/identify Gaia sources around
    the target
- name: TESS-PRF
  role: Builds the PRFs of nearby sources which TESS-cont interpolates across the
    pixels of the TPF/FFI
- name: Python
  role: Execution environment; TESS-cont is invoked as 'python TESS-cont.py <config>.ini'
    with dependencies installed via pip install -r requirements.txt
  repo: https://github.com/castro-gzlz/TESS-cont
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202451656
    title: 'TOI-5005 b: A super-Neptune in the savanna near the ridge'
  source_content_hash: sha256:5fe3fe624bebae5c852cb161e02c66b33537577dc5067ec5d0514eb111822328
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/castro-gzlz/TESS-cont
  tools:
  - Python
  - TESS
  - TESS-PRF
  - TESS-cont
  - tpfplotter
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# tess-pixel-response-function-modelling

## Summary

Modelling of TESS Pixel Response Functions (PRFs) of nearby Gaia sources to quantify flux contamination in a target's photometric aperture, as implemented by TESS-cont: contaminant Gaia DR2/DR3 sources are identified, their PRFs are interpolated across TESS TPF/FFI pixels, and target-vs-nearby flux fractions (CROWDSAP, FLFRCSAP) are computed.

## When to use

Use when quantifying what fraction of flux in a TESS SPOC aperture (or FFI tesscut) comes from the target star versus nearby Gaia contaminants, or when judging whether a transit/activity signal could originate in a contaminant.

## When NOT to use

- The photometry is not from TESS (no TPF/FFI data available), since the PRF modelling is specific to TESS pixels
- You need a definitive origin attribution for a transit signal: dilution depths below the 100% unphysical threshold only rule contaminants in, not out — higher-spatial-resolution ground-based photometry/spectroscopy is required
- Only a rough first-look contamination estimate is acceptable and runtime is critical — even then, approximate method_prf results should not be used for final analyses or publications

## Inputs

- Configuration .ini file (e.g. TOI-5005_S65.ini) with mandatory 'target' parameter (TIC, TOI, or Simbad name)
- TESS Target Pixel Files (TPFs) or Full-Frame-Image tesscut cutouts for the specified sector
- Gaia DR2 or DR3 source catalogue within the search radius (default 200 arcsec)
- Optional: measured transit depth (td, with td_unit ppm/ppt/per/frac and dilution_corr flag) for the DILUTION analysis

## Outputs

- Two-panel figure (pdf/png/pdfpng): pixel-by-pixel heatmap of target flux percentage and pie chart of target-vs-nearby flux fractions inside the aperture
- metrics.dat file containing the CROWDSAP and FLFRCSAP contamination metrics
- Per-contaminant dilution table (Star, Gaia_ID, implied transit depth) when DILUTION is enabled
- Optionally, a custom aperture saved at a chosen target-flux-percentage threshold (e.g. 80%)

## How to apply

Prepare a configuration file (e.g. TOI-5005_S65.ini) specifying the target as a TIC/TOI number or Simbad-resolvable name plus optional sector, method_prf, gaia_catalog, search_radius (default 200 arcsec), and n_sources (default 5); run 'python TESS-cont.py <config>.ini' from the terminal or via TESS-cont.ipynb. The tool downloads the Sector TPF (or an FFI tesscut with cutout_size 11,11), identifies the main Gaia DR2/DR3 contaminants within the search radius, and — using tpfplotter and TESS-PRF — builds the PRFs of nearby sources and interpolates them across the TPF/FFI pixels to obtain pixel-by-pixel flux percentages. By default method_prf is 'accurate' (PRFs interpolated to every pixel; ~5 minutes even in crowded fields), while 'approximate' interpolates the PRF only once at the TPF center for a fast first look — use accurate for final analyses. If analyzing SPOC PDCSAP photometry, set gaia_catalog: DR2 because the TIC/SPOC stack is tied to Gaia DR2, and preferentially run both DR2 and DR3 to check for catalogue discrepancies. Outputs are a two-panel figure (pixel-wise target-flux heatmap plus pie chart of aperture flux shares, disk sizes scaled to emitted fluxes, img_fmt pdfpng) and a metrics.dat file with the CROWDSAP and FLFRCSAP by-products.

## Related tools

- **TESS-cont** (Python driver that identifies Gaia contaminant sources, builds PRFs, computes aperture flux fractions, and renders the heatmap/pie-chart outputs) — https://github.com/castro-gzlz/TESS-cont
- **tpfplotter** (Used by TESS-cont to access TESS TPFs and overlay/identify Gaia sources around the target)
- **TESS-PRF** (Builds the PRFs of nearby sources which TESS-cont interpolates across the pixels of the TPF/FFI)
- **Python** (Execution environment; TESS-cont is invoked as 'python TESS-cont.py <config>.ini' with dependencies installed via pip install -r requirements.txt) — https://github.com/castro-gzlz/TESS-cont

## Examples

```
python TESS-cont.py TOI-5005_S65.ini
```

## Evaluation signals

- For TOI-5005 (TIC 282485660) in Sector 65, the pie-chart target contribution must reproduce 84.4% target flux with 15.6% from nearby sources
- The example-1 heatmap should show the target contributing between 60% and 90% of flux per pixel inside the SPOC aperture, with disk sizes scaled to emitted fluxes
- metrics.dat is produced and its CROWDSAP/FLFRCSAP values show no major differences from the official TESS metrics
- The approximate and accurate method_prf modes give very similar contamination results; large divergence signals a problem
- DR2 and DR3 runs agree on contaminant identification; major differences indicate a Gaia catalogue discrepancy to resolve

## Limitations

- Default Gaia DR3 catalogue mismatches the Gaia DR2 stacking of the TIC/SPOC PDCSAP photometry; using the wrong catalogue can skew contamination metrics
- The approximate method_prf assumes the PRF shape does not vary much across nearby pixels and is only a fast hint, not publication-grade
- DILUTION analysis cannot discard contaminants whose implied transit depths (2-28% in the TOI-4479 example) fall below the 100% unphysical threshold
- The custom aperture generation feature (threshold on target flux percentage) is currently not documented
- The second most contaminant star for TOI-5005 lies outside the TPF, so contaminants outside the downloaded cutout can be missed if the search radius/cutout is too small

## Evidence

- [intro] TESS-cont is a Python tool to compute the flux contamination from nearby stars in the TESS aperture of any observed target.: "*TESS-cont* is a Python tool to **compute the flux contamination** from **nearby stars** in the **TESS aperture** of **any observed target**."
- [readme] Algorithm basis: PRFs of nearby sources across TPFs/FFIs.: "The algorithm is based on building the PRFs of nearby sources and computing their flux distributions across the TESS TPFs/FFIs."
- [methods] TOI-5005 84.4% target contribution in Sector 65.: "**TOI-5005 contributes 84.4%** to the measured photometry, while **the remaining 15.6% comes from different nearby sources**"
- [readme] DR2 vs DR3 catalogue precaution for PDCSAP photometry.: "By default, *TESS-cont* uses the *Gaia* DR3 catalogue. However, the TIC catalogue (and SPOC PDCSAP) is stacked to *Gaia* DR2."
- [readme] CROWDSAP and FLFRCSAP are by-products saved in metrics.dat.: "A **by-product** of the *TESS-cont* operation is the computation of the *CROWDSAP* and *FLFRCSAP* metrics"
- [readme] Accurate method recommended for final analyses.: "we encourage to use the accurate method for final analyses/publications"
