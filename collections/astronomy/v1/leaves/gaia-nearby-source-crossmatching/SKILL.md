---
name: gaia-nearby-source-crossmatching
description: Use when analyzing TESS TPF/FFI photometry of a target whose SPOC aperture
  may include flux from neighboring Gaia sources, and you need to rank contaminants
  or test whether any could produce the observed transit depth.
when_to_use_negative:
- When the scientific goal is planet parameter derivation (radius, mass, density)
  from RVs and photometry — TESS-cont only quantifies contamination, not planetary
  properties
- 'When you need to confidently rule out a contaminant origin for a transit: TESS
  data alone cannot discard sources whose implied depths fall below 100% — higher-spatial-resolution
  follow-up (ground-based photometry/spectroscopy) is required'
- 'When analyzing SPOC PDCSAP photometry with the default gaia_catalog: DR3 setting
  — the DR2 catalogue must be selected instead to avoid stacking mismatches'
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: TESS-cont
  role: 'Main tool: identifies Gaia DR2/DR3 contaminants, computes their aperture
    contributions, and runs the DILUTION analysis'
  repo: github.com/castro-gzlz/TESS-cont
- name: TESS-PRF
  role: Provides the PRF models TESS-cont uses to build and distribute nearby-source
    flux across TPF/FFI pixels
- name: tpfplotter
  role: Prior plotting/visualization tool for TESS target pixel files cited alongside
    TESS-cont
- name: Python
  role: 'Execution environment: TESS-cont is run via `python TESS-cont.py <config>.ini`
    or a Jupyter notebook'
  repo: github.com/castro-gzlz/TESS-cont
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202451656
    title: 'TOI-5005 b: A super-Neptune in the savanna near the ridge'
  source_content_hash: sha256:ea08c7343bf5ceb6d1fc0a58f73f2e6d44f3ade87d6b5da6080925b8dba26658
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - TESS-PRF
  - TESS-cont
  - tpfplotter
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# gaia-nearby-source-crossmatching

## Summary

Identifies and crossmatches the main contaminant Gaia DR2/DR3 sources around a TESS target so their per-pixel flux contributions to a SPOC photometric aperture can be computed by TESS-cont, revealing which nearby stars could be responsible for transit or activity signals.

## When to use

Use when analyzing TESS TPF/FFI photometry of a target whose SPOC aperture may include flux from neighboring Gaia sources, and you need to rank contaminants or test whether any could produce the observed transit depth.

## When NOT to use

- When the scientific goal is planet parameter derivation (radius, mass, density) from RVs and photometry — TESS-cont only quantifies contamination, not planetary properties
- When you need to confidently rule out a contaminant origin for a transit: TESS data alone cannot discard sources whose implied depths fall below 100% — higher-spatial-resolution follow-up (ground-based photometry/spectroscopy) is required
- When analyzing SPOC PDCSAP photometry with the default gaia_catalog: DR3 setting — the DR2 catalogue must be selected instead to avoid stacking mismatches

## Inputs

- Target star identifier: any TIC, TOI, or Simbad-resolvable name (e.g. TOI-5005, TIC 282485660)
- TESS sector number (default: first with observations)
- Configuration .ini file with MANDATORY target and OPTIONAL/DILUTION blocks (td, td_unit, dilution_corr)
- Gaia DR2 or DR3 catalogue (default DR3; DR2 required for SPOC PDCSAP analysis)
- TESS TPFs or FFI tesscut cutouts (default tpf, cutout_size 11,11)
- Measured transit depth from ExoFOP or SPOC for DILUTION analysis

## Outputs

- Heatmap of per-pixel flux percentage from the target star
- Pie chart of target vs. nearby-star flux contributions inside (and outside) the aperture
- Ranked list of contaminant sources with Gaia IDs and implied transit depths in % (DILUTION mode)
- CROWDSAP and FLFRCSAP metrics saved to metrics.dat
- Optional custom aperture meeting a target-flux-percentage threshold (e.g. 80%)
- Output figures in img_fmt format (pdf, png, or pdfpng)

## How to apply

Give TESS-cont a target identifier (TIC, TOI, or Simbad-resolvable name) plus optional sector; by default it queries the Gaia DR3 catalogue within a 200 arcsec search radius and studies the 5 most contaminant sources individually. Because the TIC catalogue and SPOC PDCSAP photometry are stacked to Gaia DR2, set gaia_catalog: DR2 when analyzing PDCSAP data, and run both DR2 and DR3 to ensure no major catalogue discrepancies exist. The tool builds the PRFs of nearby sources and computes their flux distributions across all TPF/FFI pixels (default method_prf: accurate interpolates PRFs to every pixel with Gaia sources; the approximate method interpolates once at the TPF center). It then ranks contaminants, reports per-pixel and in-aperture flux fractions, and in DILUTION mode converts the measured transit depth (e.g. 3471 ppm from ExoFOP for TOI-4479, with dilution_corr: True since SPOC depths are already corrected) into implied per-contaminant depths that are compared against the 100% physical threshold.

## Related tools

- **TESS-cont** (Main tool: identifies Gaia DR2/DR3 contaminants, computes their aperture contributions, and runs the DILUTION analysis) — github.com/castro-gzlz/TESS-cont
- **TESS-PRF** (Provides the PRF models TESS-cont uses to build and distribute nearby-source flux across TPF/FFI pixels)
- **tpfplotter** (Prior plotting/visualization tool for TESS target pixel files cited alongside TESS-cont)
- **Python** (Execution environment: TESS-cont is run via `python TESS-cont.py <config>.ini` or a Jupyter notebook) — github.com/castro-gzlz/TESS-cont

## Examples

```
python TESS-cont.py TOI-4479_S41.ini
```

## Evaluation signals

- All implied contaminant transit depths for TOI-4479 Sector 41 fall in the reported ~2–28% range (1.74, 6.10, 8.256, 11.30, 24.35, 27.69% for Gaia IDs 1841177816084707584 … 1841177682942388608) and stay below the 100% unphysical threshold
- The in-aperture contaminant flux fraction matches published values (30.4% for TOI-4479 S41; TOI-5005 contributes 84.4% with 15.6% from nearby sources in S65)
- The computed CROWDSAP/FLFRCSAP metrics in metrics.dat show no major differences from the official TESS metrics — large discrepancies suggest Gaia DR2/DR3 catalogue mismatches
- DR2 and DR3 catalogue runs yield consistent contamination metrics; if not, use DR3 metrics to crowding-correct the PDCSAP/SAP photometry
- Figures are produced in the requested img_fmt (pdf, png, or pdfpng) with heatmaps and pie charts highlighting contaminant stars inside and outside the aperture

## Limitations

- The default approximate PRF method (method_prf: approximate) gives only a first hint; the accurate method is recommended for final analyses/publications
- Even the accurate method in crowded fields should not surpass ~5 minutes because PRFs of stars in common pixels are computed once, but it still interpolates PRFs to all Gaia pixels and takes several minutes
- Discrepancies between Gaia DR2 and DR3 catalogues can produce CROWDSAP/FLFRCSAP values that differ from official TESS metrics
- A contaminant origin for a transit signal cannot be discarded from TESS data alone whenever implied depths fall below 100% — higher-spatial-resolution observations are needed
- The custom aperture generation feature (target-flux-percentage threshold, e.g. 80%) is currently undocumented

## Evidence

- [intro] The package **(1)** identifies the main contaminant *Gaia* DR2 / DR3 sources, **(2)** computes their contributions to the aperture: "The package **(1)** identifies the main contaminant *Gaia* DR2 / DR3 sources, **(2)** computes their contributions to the aperture"
- [readme] By default, *TESS-cont* uses the *Gaia* DR3 catalogue. However, the TIC catalogue (and SPOC PDCSAP) is stacked to *Gaia* DR2.: "By default, *TESS-cont* uses the *Gaia* DR3 catalogue. However, the TIC catalogue (and SPOC PDCSAP) is stacked to *Gaia* DR2."
- [readme] the TESS-cont **DILUTION** feature **to know whether we can discard the nearby contaminant sources as being the origin of the signal found** based on the TESS photometry alone: "We here use the *TESS-cont* **DILUTION** feature **to know whether we can discard the nearby contaminant sources as being the origin of the signal found** based on the TESS photometry alone"
- [readme] This example reveals the necessity of **higher-spatial resolution observations** (e.g. ground-based photometry or spectroscopy) to confidently **discard those contaminant sources**: "This example reveals the necessity of **higher-spatial resolution observations** (e.g. ground-based photometry or spectroscopy) to confidently **discard those contaminant sources**"
- [methods] **TOI-5005 contributes 84.4%** to the measured photometry, while **the remaining 15.6% comes from different nearby sources**: "**TOI-5005 contributes 84.4%** to the measured photometry, while **the remaining 15.6% comes from different nearby sources**"
