---
name: tess-target-pixel-file-handling
description: Use when a TESS TIC/TOI target needs pixel-level analysis — resolving
  the identifier to sky coordinates, obtaining the SPOC TPF (or an 11x11 FFI tesscut),
  and computing per-pixel target-flux percentages inside the photometric aperture.
when_to_use_negative:
- When the analysis only needs the calibrated PDCSAP/SAP time series and no pixel-level
  aperture or crowding information
- When contaminant sources must be spatially resolved or discarded with confidence
  — TESS pixel scales cannot separate them, and higher-spatial-resolution observations
  (ground-based photometry or spectroscopy) are required
- When no TESS data exist for the target (no sector with observations and no FFI coverage)
tools:
- name: TESS-cont
  role: Python tool that loads the TPF/FFI, queries Gaia, and computes pixel-level
    flux contamination; run via a config .ini file
  repo: github.com/castro-gzlz/TESS-cont
- name: tpfplotter
  role: Used to work with the TESS TPF and its photometric aperture in the contamination
    workflow
- name: TESS-PRF
  role: Builds the PRFs of nearby sources that are interpolated across the TPF/FFI
    pixels
- name: TESS
  role: Source of the SPOC Target Pixel Files and Full Frame Images analyzed by the
    workflow
provenance:
  source_task_ids:
  - task_006
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202451656
    title: 'TOI-5005 b: A super-Neptune in the savanna near the ridge'
  source_content_hash: sha256:7933d52cb06ef8cbe59e656e6cf163e2a48dc83c5c852ed256dfeb26e50cf4b8
schema_version: 0.2.0
metadata:
  license_tier: open
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
yes_rate: 0.6363636363636364
n_claims: 11
---

# tess-target-pixel-file-handling

## Summary

Loads and works with TESS Target Pixel Files (or tesscut FFI cutouts) for a resolved target so that pixel-level flux fractions, apertures, and PRF models can be computed with TESS-cont and its dependencies.

## When to use

Use when a TESS TIC/TOI target needs pixel-level analysis — resolving the identifier to sky coordinates, obtaining the SPOC TPF (or an 11x11 FFI tesscut), and computing per-pixel target-flux percentages inside the photometric aperture.

## When NOT to use

- When the analysis only needs the calibrated PDCSAP/SAP time series and no pixel-level aperture or crowding information
- When contaminant sources must be spatially resolved or discarded with confidence — TESS pixel scales cannot separate them, and higher-spatial-resolution observations (ground-based photometry or spectroscopy) are required
- When no TESS data exist for the target (no sector with observations and no FFI coverage)

## Inputs

- Target identifier: TIC number, TOI number, or Simbad-resolvable star name (e.g. TIC 282485660 / TOI-5005)
- Configuration file (.ini) with mandatory 'target' parameter and optional sector, gaia_catalog, search_radius, n_sources, tpf_or_tesscut, cutout_size
- TESS SPOC Target Pixel File (TPF) for the sector, or TESS FFI data accessed via tesscut
- Gaia DR2 or DR3 catalogue queried within a 200 arcsec search radius

## Outputs

- Pixel-by-pixel heatmap of the target star's flux percentage (60–90% per aperture pixel for TOI-5005 Sector 65)
- Ranked contaminant-source table contaminant_sources.csv (Star index, Gaia_ID, inside/outside aperture flag)
- metrics.dat file containing the CROWDSAP and FLFRCSAP by-products
- Pie chart of flux shares inside the photometric aperture (target 84.4% vs 15.6% nearby sources for TOI-5005 S65), rendered in pdf/png/pdfpng format

## How to apply

Resolve the target identifier (any TIC, TOI, or Simbad-resolvable name, e.g. TOI-5005 = TIC 282485660) from the .ini configuration file to obtain sky coordinates, then load the TESS Target Pixel File via tpf_or_tesscut: tpf (default) or an FFI tesscut with cutout_size 11,11. Query the Gaia catalogue around the target within search_radius = 200 arcsec — DR3 by default, but DR2 must be selected when analyzing SPOC PDCSAP photometry because the TIC is stacked to Gaia DR2. Build the PRFs of nearby sources (tpfplotter + TESS-PRF) and interpolate them across the TPF/FFI pixels using method_prf: accurate (default; the approximate method interpolates only once and is only for first hints). Compute pixel-by-pixel target flux percentages and the aggregated target-vs-nearby-star fractions inside the SPOC aperture, rank contaminants (n_sources = 5) by flux contribution, and write the CROWDSAP/FLFRCSAP by-products to metrics.dat. Validate that these metrics show no major differences with the official TESS values, since discrepancies typically indicate a DR3 vs DR2 catalogue mismatch.

## Related tools

- **TESS-cont** (Python tool that loads the TPF/FFI, queries Gaia, and computes pixel-level flux contamination; run via a config .ini file) — github.com/castro-gzlz/TESS-cont
- **tpfplotter** (Used to work with the TESS TPF and its photometric aperture in the contamination workflow)
- **TESS-PRF** (Builds the PRFs of nearby sources that are interpolated across the TPF/FFI pixels)
- **TESS** (Source of the SPOC Target Pixel Files and Full Frame Images analyzed by the workflow)

## Examples

```
python TESS-cont.py TOI-5005_S65.ini
```

## Evaluation signals

- CROWDSAP and FLFRCSAP written to metrics.dat show no major differences with the official TESS metrics; discrepancies signal a DR3 vs DR2 Gaia catalogue mismatch
- For TOI-5005 in Sector 65, the pie-chart target contribution reproduces the reported 84.4% (remaining 15.6% from nearby sources)
- Heatmap shows target flux percentage between 60% and 90% inside the aperture pixels for the TOI-5005 S65 example
- contaminant_sources.csv contains the top n_sources = 5 Gaia source_ids with an inside/outside aperture flag for each
- Default parameters hold: search_radius 200 arcsec, n_sources 5, gaia_catalog DR3, method_prf accurate, cutout_size 11,11 for tesscut

## Limitations

- The TIC catalogue and SPOC PDCSAP are stacked to Gaia DR2, so the DR3 default must be overridden with gaia_catalog: DR2 for PDCSAP analyses; running both DR2 and DR3 is encouraged to catch mismatches
- The approximate PRF method interpolates the PRF only once (mid-TPF) and should not be used for final analyses or publications, even though accurate runs should stay under ~5 minutes
- Bright contaminants can lie outside the TPF (e.g. Star 2 for TOI-5005), so contaminant identification is not limited to pixels actually downloaded
- The custom aperture generation feature (e.g. 80% target-flux threshold) is not documented and requires contacting the author

## Evidence

- [intro] The algorithm is based on building the PRFs of nearby sources and computing their flux distributions across the TESS TPFs/FFIs.: "The algorithm is based on building the PRFs of nearby sources and computing their flux distributions across the TESS TPFs/FFIs."
- [methods] A by-product of the TESS-cont operation is the computation of the CROWDSAP and FLFRCSAP metrics.: "A **by-product** of the *TESS-cont* operation is the computation of the *CROWDSAP* and *FLFRCSAP* metrics"
- [readme] By default, TESS-cont uses the Gaia DR3 catalogue. However, the TIC catalogue (and SPOC PDCSAP) is stacked to Gaia DR2.: "By default, *TESS-cont* uses the *Gaia* DR3 catalogue. However, the TIC catalogue (and SPOC PDCSAP) is stacked to *Gaia* DR2."
- [readme] This method interpolates the PRF only once (in the middle of the TPF/FFI), before locating it in its corresponding position.: "This method **interpolates the PRF only once** (in the middle of the TPF/FFI), before locating it in its corresponding position"
- [readme] Overall, TOI-5005 contributes 84.4% to the measured photometry, while the remaining 15.6% comes from different nearby sources.: "Overall, **TOI-5005 contributes 84.4%** to the measured photometry, while **the remaining 15.6% comes from different nearby sources**"
- [readme] The second most contaminant star (Star 2) is a very bright source located outside the TESS Target Pixel File (TPF).: "the second most contaminant star (**Star 2**) is a very bright source located **outside the TESS Target Pixel File (TPF)**"
