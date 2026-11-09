---
name: gaia-source-identification
description: Use when quantifying which nearby Gaia DR2/DR3 stars contaminate a TESS
  target's aperture — e.g. a TOI/TIC where flux dilution or false-positive checks
  require per-source Gaia IDs and inside/outside-aperture flags.
when_to_use_negative:
- The downstream steps are already done — i.e. you need per-pixel PRF flux fractions
  (step 2) or transit-depth attribution against the 100% unphysical threshold (step
  3), not source identification.
- 'Analyzing SPOC PDCSAP photometry while keeping the default Gaia DR3 catalogue;
  the TIC is stacked to Gaia DR2, so this skill requires gaia_catalog: DR2 in that
  case.'
- The target cannot be resolved to sky coordinates (not TIC/TOI/Simbad-resolvable)
  or the region of interest extends beyond the 200 arcsec default search radius without
  adjusting it.
edam_operation: http://edamontology.org/operation_0306
tools:
- name: TESS-cont
  role: 'Runs the identification step: resolves the target, queries Gaia DR2/DR3 within
    200 arcsec, ranks contaminants, and flags inside/outside-aperture status'
  repo: github.com/castro-gzlz/TESS-cont
- name: Python
  role: Execution environment; TESS-cont is invoked via 'python TESS-cont.py <config>.ini'
    or through the bundled Jupyter notebook
  repo: github.com/castro-gzlz/TESS-cont
provenance:
  source_task_ids:
  - task_008
  - task_007
  - task_002
  - task_006
  source_papers:
  - doi: 10.1051/0004-6361/202451656
    title: 'TOI-5005 b: A super-Neptune in the savanna near the ridge'
  source_content_hash: sha256:d75534148868ef0d8e0963742effad045028b87743e99b13ddc6dab7103e00d2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - TESS-cont
  - tpfplotter
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# gaia-source-identification

## Summary

TESS-cont step (1): resolve any TIC/TOI/Simbad target, query Gaia DR2/DR3 sources within a 200 arcsec search radius, and rank nearby stars by flux contribution, flagging each as inside or outside the SPOC photometric aperture to select the top n_sources=5 contaminants.

## When to use

Use when quantifying which nearby Gaia DR2/DR3 stars contaminate a TESS target's aperture — e.g. a TOI/TIC where flux dilution or false-positive checks require per-source Gaia IDs and inside/outside-aperture flags.

## When NOT to use

- The downstream steps are already done — i.e. you need per-pixel PRF flux fractions (step 2) or transit-depth attribution against the 100% unphysical threshold (step 3), not source identification.
- Analyzing SPOC PDCSAP photometry while keeping the default Gaia DR3 catalogue; the TIC is stacked to Gaia DR2, so this skill requires gaia_catalog: DR2 in that case.
- The target cannot be resolved to sky coordinates (not TIC/TOI/Simbad-resolvable) or the region of interest extends beyond the 200 arcsec default search radius without adjusting it.

## Inputs

- Configuration .ini file with target identifier (TIC/TOI/Simbad name), e.g. TOI-5005_S65.ini specifying TIC 282485660 and sector 65
- TESS Target Pixel File (tpf) or FFI tesscut (cutout_size 11,11)
- Gaia DR3 (default) or Gaia DR2 catalogue query within search_radius=200 arcsec

## Outputs

- contaminant_sources.csv: ranked table with Star index, Gaia_ID, and inside/outside-aperture flag
- metrics.dat with CROWDSAP and FLFRCSAP contamination metrics
- Heatmap of per-pixel target flux percentage and pie chart of target vs. contaminant flux inside the aperture (img_fmt pdfpng)

## How to apply

Start from a configuration file (e.g. TOI-5005_S65.ini) whose only mandatory parameter is `target` (any TIC, TOI, or Simbad-resolvable name), plus optional parameters sector, gaia_catalog, search_radius, and n_sources. Load the SPOC Target Pixel File (or an FFI tesscut of 11,11 pixels) and query the Gaia catalogue around the target within search_radius=200 arcsec — DR3 by default, but DR2 must be selected when analyzing SPOC PDCSAP photometry since the TIC is stacked to Gaia DR2. Rank the retrieved sources by flux contribution and keep the top n_sources=5 contaminants for individual study, recording each Gaia source_id and whether it falls inside or outside the photometric aperture. Note that bright contaminants can sit outside the TPF entirely (e.g. Star 2 of TOI-5005 in Sector 65), which is why the search radius, not the TPF footprint, defines the candidate set. Validate the run by cross-checking the CROWDSAP/FLFRCSAP by-products in metrics.dat against the official TESS metrics; major differences indicate a Gaia DR2 vs DR3 catalogue mismatch.

## Related tools

- **TESS-cont** (Runs the identification step: resolves the target, queries Gaia DR2/DR3 within 200 arcsec, ranks contaminants, and flags inside/outside-aperture status) — github.com/castro-gzlz/TESS-cont
- **Python** (Execution environment; TESS-cont is invoked via 'python TESS-cont.py <config>.ini' or through the bundled Jupyter notebook) — github.com/castro-gzlz/TESS-cont

## Examples

```
python TESS-cont.py TOI-5005_S65.ini
```

## Evaluation signals

- contaminant_sources.csv lists the top n_sources=5 contaminants with Star index, Gaia_ID, and an inside/outside-aperture flag for each.
- metrics.dat CROWDSAP/FLFRCSAP values show no major differences with the official TESS metrics; discrepancies flag a Gaia DR2 vs DR3 catalogue mismatch.
- For the TOI-5005 Sector 65 benchmark, the pie-chart target contribution reproduces 84.4% (15.6% from nearby sources).
- Bright contaminants located outside the TPF (e.g. TOI-5005's Star 2) are still captured, confirming the 200 arcsec Gaia search radius was applied rather than limiting to the TPF footprint.
- Running both DR2 and DR3 catalogues yields no major differences; otherwise the DR3 metrics should be used to correct PDCSAP/SAP crowding.

## Limitations

- Default Gaia DR3 vs. TIC/SPOC stacking to DR2: run both catalogues to ensure no major differences; if they disagree, use the DR3 contamination metrics to correct PDCSAP/SAP photometry for crowding.
- The default accurate PRF method interpolates to all pixels with Gaia sources and may take several minutes; the approximate method interpolates once at the TPF/FFI centre and is only a first hint — use accurate for final analyses/publications.
- The custom-aperture feature (threshold of target flux percentage, e.g. 80%) is currently not documented.

## Evidence

- [intro] TESS-cont computes the flux contamination from nearby stars in the TESS aperture of any observed target: "*TESS-cont* is a Python tool to **compute the flux contamination** from **nearby stars** in the **TESS aperture** of **any observed target**."
- [readme] DR2 vs DR3 catalogue stacking for PDCSAP photometry: "the TIC catalogue (and SPOC PDCSAP) is stacked to *Gaia* DR2. Therefore, to analyze the TESS PDCSAP photometry the **DR2 catalog should be selected**"
- [readme] Cross-check contamination metrics against official TESS values: "We encourage ensuring that there are no major differences with the official TESS metrics."
- [methods] Default Gaia catalogue and search parameters: "By default, *TESS-cont* uses the *Gaia* DR3 catalogue"
