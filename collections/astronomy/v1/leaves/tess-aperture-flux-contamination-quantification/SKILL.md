---
name: tess-aperture-flux-contamination-quantification
description: Use when analyzing TESS SPOC PDCSAP or TPF/FFI photometry of a target
  where nearby Gaia sources may dilute the aperture, or when judging whether a transit/activity
  signal could originate from a contaminant rather than the target.
when_to_use_negative:
- When the goal is exoplanet parameter inference from radial velocities or light-curve
  fitting (e.g. Bayesian analysis of HARPS RVs) rather than aperture contamination
  — use dedicated transit/RV pipelines instead
- When the required spatial resolution to exclude contaminants exceeds what TESS photometry
  alone can provide; TESS-cont can rank implied depths but cannot discard contaminants
  below the 100% depth threshold
- 'When a custom aperture is needed on a tight deadline without author support: the
  custom-aperture feature is explicitly undocumented'
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: TESS-cont
  role: 'Core tool: identifies contaminant Gaia DR2/DR3 sources, computes their per-pixel
    and aperture-aggregated flux contributions, and runs the DILUTION analysis'
  repo: https://github.com/castro-gzlz/TESS-cont
- name: tpfplotter
  role: Used to access and display TESS TPF pixels and Gaia overlays within the TESS-cont
    workflow
- name: TESS-PRF
  role: Builds the PRFs of nearby sources that TESS-cont interpolates across the pixels
    of the TPF/FFI
- name: Python
  role: Runtime environment; executed as 'python TESS-cont.py <config>.ini' from the
    terminal or via the provided Jupyter notebook
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_008
  source_papers:
  - doi: 10.1051/0004-6361/202451656
    title: 'TOI-5005 b: A super-Neptune in the savanna near the ridge'
  source_content_hash: sha256:d2c145bf6e78d956d40d9a05ccccc14ec414ec3eeae149eaf65a26eecbe78157
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# tess-aperture-flux-contamination-quantification

## Summary

Quantify the fraction of flux in a TESS photometric aperture contributed by the target star versus nearby Gaia DR2/DR3 contaminants, using TESS-cont's PRF-based modelling of the target's TPF/FFI pixels. This distinguishes dilution-driven transit-depth biases and identifies contaminant sources that could mimic transit or activity signals.

## When to use

Use when analyzing TESS SPOC PDCSAP or TPF/FFI photometry of a target where nearby Gaia sources may dilute the aperture, or when judging whether a transit/activity signal could originate from a contaminant rather than the target.

## When NOT to use

- When the goal is exoplanet parameter inference from radial velocities or light-curve fitting (e.g. Bayesian analysis of HARPS RVs) rather than aperture contamination — use dedicated transit/RV pipelines instead
- When the required spatial resolution to exclude contaminants exceeds what TESS photometry alone can provide; TESS-cont can rank implied depths but cannot discard contaminants below the 100% depth threshold
- When a custom aperture is needed on a tight deadline without author support: the custom-aperture feature is explicitly undocumented

## Inputs

- TESS-cont configuration file (.ini) with mandatory 'target' parameter and optional sector/method_prf/search_radius/n_sources/gaia_catalog
- TESS Target Pixel Files (TPFs) or FFI tesscut cutouts for the target sector
- Gaia DR2 and/or DR3 source catalogues around the target
- Measured transit depth with unit (ppm, ppt, per, frac) for DILUTION analysis, e.g. from ExoFOP
- TIC catalogue / SPOC PDCSAP photometry

## Outputs

- Heatmap figure of the target's flux percentage per pixel (disk sizes scaled to emitted fluxes, pdfpng format)
- Pie chart of flux shares from target vs nearby stars inside (and outside) the photometric aperture
- metrics.dat file with by-product CROWDSAP and FLFRCSAP values
- DILUTION attribution table (Star, Gaia_ID, transit_depth(%)) of implied transit depths per contaminant
- Optional custom aperture at a chosen target-flux threshold (e.g. 80%)

## How to apply

Write a TESS-cont configuration .ini file with the mandatory 'target' identifier (any TIC, TOI, or Simbad-resolvable name) plus optional parameters such as 'sector', 'search_radius' (default 200 arcsec), 'n_sources' (default 5), and 'method_prf' (accurate by default; 'approximate' interpolates the PRF once at the TPF/FFI centre for a quick first hint, but accurate is encouraged for final analyses). Run the tool, which builds the PRFs of nearby Gaia sources (via tpfplotter and TESS-PRF), computes per-pixel and aggregated flux fractions inside the SPOC aperture, and renders a heatmap of target flux percentage per pixel plus a pie chart of target-vs-nearby flux shares. Decide the Gaia catalogue carefully: default is DR3, but to analyze PDCSAP photometry select 'gaia_catalog: DR2' since TIC/SPOC are stacked to DR2, and run both DR2 and DR3 to check for discrepancies. Optionally feed the measured transit depth (e.g. 3471 ppm from ExoFOP, 'td_unit: ppm') with 'dilution_corr: True' so implied contaminant depths are rescaled and filtered against the unphysical 100% threshold.

## Related tools

- **TESS-cont** (Core tool: identifies contaminant Gaia DR2/DR3 sources, computes their per-pixel and aperture-aggregated flux contributions, and runs the DILUTION analysis) — https://github.com/castro-gzlz/TESS-cont
- **tpfplotter** (Used to access and display TESS TPF pixels and Gaia overlays within the TESS-cont workflow)
- **TESS-PRF** (Builds the PRFs of nearby sources that TESS-cont interpolates across the pixels of the TPF/FFI)
- **Python** (Runtime environment; executed as 'python TESS-cont.py <config>.ini' from the terminal or via the provided Jupyter notebook)

## Examples

```
python TESS-cont.py TOI-5005_S65.ini
```

## Evaluation signals

- Benchmark TOI-5005 Sector 65 (TIC 282485660): the pie chart must show the target contributing 84.4% of aperture photometry, with the remaining 15.6% from nearby sources
- Benchmark TOI-4479 Sector 41: total contaminant fraction must equal 30.4%, and DILUTION depths for the six most contaminant stars must be 1.74%, 6.10%, 8.256%, 11.30%, 24.35%, 27.69% (all < 100%)
- metrics.dat CROWDSAP/FLFRCSAP values must not differ substantially from official TESS metrics; large discrepancies may indicate a Gaia DR3 vs DR2 mismatch
- PRF interpolation with method_prf: accurate should complete in under ~5 minutes even in crowded fields; approximate and accurate results should be very similar
- All contaminant-derived implied transit depths should fall below the unphysical 100% threshold to remain candidates for the signal origin

## Limitations

- The default accurate PRF interpolation can take several minutes; the approximate method (method_prf: approximate) is faster but the accurate method is encouraged for final analyses/publications
- Gaia DR3 is the default catalogue but TIC/SPOC PDCSAP is stacked to DR2; DR3-vs-DR2 discrepancies can propagate into CROWDSAP/FLFRCSAP, so both catalogues should be run and DR3 metrics used for crowding corrections of PDCSAP/SAP when needed
- The custom aperture generation feature (e.g. 80% target-flux threshold) is currently undocumented
- If implied contaminant depths are below 100%, TESS data alone cannot discard them as the signal origin; higher-spatial-resolution observations (ground-based photometry or spectroscopy) are required

## Evidence

- [intro] TESS-cont is a Python tool to compute the flux contamination from nearby stars in the TESS aperture of any observed target.: "*TESS-cont* is a Python tool to **compute the flux contamination** from **nearby stars** in the **TESS aperture** of **any observed target**."
- [readme] TOI-5005 contributes 84.4% to the measured photometry, while the remaining 15.6% comes from different nearby sources.: "Overall, **TOI-5005 contributes 84.4%** to the measured photometry, while **the remaining 15.6% comes from different nearby sources**."
- [readme] By default, TESS-cont uses the Gaia DR3 catalogue. However, the TIC catalogue (and SPOC PDCSAP) is stacked to Gaia DR2.: "By default, *TESS-cont* uses the *Gaia* DR3 catalogue. However, the TIC catalogue (and SPOC PDCSAP) is stacked to *Gaia* DR2."
- [readme] A by-product of the TESS-cont operation is the computation of the CROWDSAP and FLFRCSAP metrics.: "A **by-product** of the *TESS-cont* operation is the computation of the *CROWDSAP* and *FLFRCSAP* metrics."
- [readme] This approximate method typically provides very similar results to the default method_prf: accurate method, so it can be useful to have a first hint of the contamination level.: "This approximate method **typically provides very similar results** to the default ```method_prf: accurate``` method, so it can be **useful to have a first hint** of the contamination level affecting"
- [readme] Since those values (2-28%) are lower than the unphysical threshold of 100%, we cannot discard that they could be the origin of the transit signal found based on the TESS data alone.: "Since those values (**2-28%**) are lower than the unphysical threshold of 100%, we cannot discard that they could be the origin of the transit signal found based on the TESS data alone"
