---
name: transit-dilution-correction-analysis
description: Use when a TESS transit or activity signal has a measured depth (in ppm/ppt/per/frac)
  and you must decide whether nearby Gaia contaminant stars inside the SPOC aperture
  could be the signal's origin from TESS photometry alone.
when_to_use_negative:
- The goal is to confirm/deny contamination with high spatial resolution — this skill
  explicitly cannot discard contaminants with implied depths below 100%; higher-resolution
  ground-based photometry or spectroscopy is needed instead
- 'Analyzing SPOC PDCSAP photometry without setting gaia_catalog: DR2 — using the
  default DR3 catalogue for TIC/SPOC-stacked data risks mismatched contamination metrics'
- The target has no measured transit/activity depth — the DILUTION analysis requires
  td; for pure contamination quantification use the standard TESS-cont output instead
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3322
tools:
- name: TESS-cont
  role: Python tool that computes flux contamination from nearby stars in the TESS
    aperture and runs the DILUTION analysis to attribute transit/activity signals
    to contaminants
  repo: https://github.com/castro-gzlz/TESS-cont
- name: ExoFOP
  role: Source of the measured transit depth (3471 ppm) for the target's aperture
- name: TESS-PRF
  role: PRF models used to build nearby-source PRFs and compute per-pixel flux distributions
    across the TESS TPFs/FFIs
- name: tpfplotter
  role: Companion plotting tool cited alongside TESS-cont in the methods for TPF visualisation
provenance:
  source_task_ids:
  - task_003
  - task_008
  source_papers:
  - doi: 10.1051/0004-6361/202451656
    title: 'TOI-5005 b: A super-Neptune in the savanna near the ridge'
  source_content_hash: sha256:23d0e2deae2b6700e820a3bcff63048b56dcd734b7b085e2ed69c67315eb17bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/castro-gzlz/TESS-cont
  tools:
  - ExoFOP
  - Python
  - TESS-PRF
  - TESS-cont
  - tpfplotter
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3322
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# transit-dilution-correction-analysis

## Summary

Uses the TESS-cont DILUTION feature to rescale a measured TESS transit depth (e.g. TOI-4479.01 at 3471 ppm) by each contaminant star's flux fraction inside the photometric aperture, computing the depth each contaminant would need and filtering sources against the 100% unphysical-depth threshold.

## When to use

Use when a TESS transit or activity signal has a measured depth (in ppm/ppt/per/frac) and you must decide whether nearby Gaia contaminant stars inside the SPOC aperture could be the signal's origin from TESS photometry alone.

## When NOT to use

- The goal is to confirm/deny contamination with high spatial resolution — this skill explicitly cannot discard contaminants with implied depths below 100%; higher-resolution ground-based photometry or spectroscopy is needed instead
- Analyzing SPOC PDCSAP photometry without setting gaia_catalog: DR2 — using the default DR3 catalogue for TIC/SPOC-stacked data risks mismatched contamination metrics
- The target has no measured transit/activity depth — the DILUTION analysis requires td; for pure contamination quantification use the standard TESS-cont output instead

## Inputs

- Measured transit depth in the target's aperture (e.g. 3471 ppm from ExoFOP for TOI-4479.01)
- TESS-cont configuration .ini file with target, sector, DILUTION block (td, td_unit, dilution_corr)
- Per-source contaminant flux fractions inside the TESS photometric aperture from upstream TESS-cont PRF analysis
- Gaia DR2/DR3 source catalogue positions and fluxes of nearby stars (default search radius 200 arcsec)

## Outputs

- Structured attribution table (Star, Gaia_ID, transit_depth(%)) listing implied depths per contaminant, e.g. dilution_report.csv
- Implied transit depths for the most contaminant stars (TOI-4479 S41: 1.74, 6.10, 8.256, 11.30, 24.35, 27.69%) with accept/reject verdicts vs the 100% threshold
- Output figure in pdfpng format showing flux-fraction heatmaps and contaminant pie charts
- CROWDSAP and FLFRCSAP metrics as a by-product, saved in metrics.dat

## How to apply

Run TESS-cont with a configuration file for the target and sector (e.g. TOI-4479_S41.ini), keeping the default accurate PRF interpolation (method_prf: accurate) so PRFs are interpolated to all pixels with Gaia sources. Populate the DILUTION block with the measured depth (td = 3471, td_unit = ppm from ExoFOP) and set dilution_corr: True because SPOC transit depths are already corrected for dilution; the measured depth is rescaled by each contaminant's flux fraction inside the aperture. The tool computes the transit depth each of the N most contaminant stars (default n_sources: 5) would need to produce the observed signal, then filters sources against the unphysical 100% threshold: sources implying depths at or above 100% are discarded as possible origins. For the TOI-4479 Sector 41 benchmark, all six implied depths (1.74%, 6.10%, 8.256%, 11.30%, 24.35%, 27.69%) fall below 100%, so a contaminant origin cannot be discarded from TESS data alone. Set gaia_catalog: DR2 when analyzing SPOC PDCSAP photometry, since TIC/SPOC is stacked to Gaia DR2, and save the report and figure (img_fmt: pdfpng).

## Related tools

- **TESS-cont** (Python tool that computes flux contamination from nearby stars in the TESS aperture and runs the DILUTION analysis to attribute transit/activity signals to contaminants) — https://github.com/castro-gzlz/TESS-cont
- **ExoFOP** (Source of the measured transit depth (3471 ppm) for the target's aperture)
- **TESS-PRF** (PRF models used to build nearby-source PRFs and compute per-pixel flux distributions across the TESS TPFs/FFIs)
- **tpfplotter** (Companion plotting tool cited alongside TESS-cont in the methods for TPF visualisation)

## Examples

```
python TESS-cont.py TOI-4479_S41.ini
```

## Evaluation signals

- For the TOI-4479 Sector 41 benchmark, the report lists implied depths of 1.74%, 6.10%, 8.256%, 11.30%, 24.35% and 27.69% for Gaia IDs 1841177816084707584, 1841177816084683648, 1841178129618984960, 1841177717302124288, 1841178232698201216, 1841177682942388608
- All implied contaminant depths fall in the reported ~2-28% range and remain below the 100% unphysical threshold
- Total contaminant fraction inside the aperture matches ~30.4% for the TOI-4479 S41 case (84.4% target contribution for the TOI-5005 S65 case)
- dilution_corr is True when depths come from SPOC (already dilution-corrected); td_unit matches the depth source's unit (ppm/ppt/per/frac)
- CROWDSAP and FLFRCSAP in metrics.dat show no major differences with the official TESS metrics (discrepancies may indicate DR3-vs-DR2 Gaia catalogue mismatch)

## Limitations

- Sources with implied depths below 100% cannot be discarded from TESS photometry alone; higher-spatial-resolution observations (ground-based photometry or spectroscopy) are required to confidently reject them
- The default Gaia DR3 catalogue is stacked differently from the TIC/SPOC PDCSAP (DR2), so DR3-vs-DR2 discrepancies can shift the contamination metrics; running both catalogues is encouraged
- The default accurate PRF interpolation over all pixels with Gaia sources can take several minutes; the approximate method is faster but should be reserved for first hints, not final analyses

## Evidence

- [other] DILUTION block with td = 3471, td_unit = ppm (value from ExoFOP), and dilution_corr = True since the SPOC-reported depth is already corrected for dilution.: [evidence span withheld — anchor status: not_found]
- [readme] Implied depths listing from README: "Since those values (**2-28%**) are lower than the unphysical threshold of 100%, we cannot discard that they could be the origin of the transit signal found based on the TESS data alone"
- [other] Filter logic from task card: [evidence span withheld — anchor status: not_found]
- [readme] DR2/DR3 precaution from README: "By default, *TESS-cont* uses the *Gaia* DR3 catalogue. However, the TIC catalogue (and SPOC PDCSAP) is stacked to *Gaia* DR2. Therefore, to analyze the TESS PDCSAP photometry the **DR2 catalog should"
- [intro] Tool description from article intro: "*TESS-cont* is a Python tool to **compute the flux contamination** from **nearby stars** in the **TESS aperture** of **any observed target**."
