---
name: tess-contaminant-flux-ranking
description: Use when you have a TIC/TOI or Simbad-resolvable target with TESS photometry
  and need to determine which nearby Gaia stars contaminate the SPOC aperture, whether
  inside or outside the TPF.
when_to_use_negative:
- When the goal is to confidently discard contaminant sources as the origin of a transit
  signal and TESS-cont DILUTION shows the contaminant depths (e.g. 2–28%) cannot be
  excluded from the TESS photometry alone — higher-spatial resolution observations
  (ground-based photometry/spectroscopy) are needed instead
- When generating custom apertures at a target-flux-percentage threshold — this feature
  is currently not documented in TESS-cont and requires contacting the author
- When you need a single authoritative catalogue run only — the README encourages
  running with BOTH DR2 and DR3 catalogues to ensure no major differences
edam_operation: http://edamontology.org/operation_3695
tools:
- name: TESS-cont
  role: Python tool that identifies the main contaminant Gaia DR2/DR3 sources, computes
    their flux contributions to the TESS aperture, and flags each source as inside/outside
    the SPOC photometric aperture
  repo: https://github.com/castro-gzlz/TESS-cont
- name: Python
  role: Execution environment; TESS-cont is run from the terminal or Jupyter Notebook
    via a configuration .ini file in the config folder
  repo: https://github.com/castro-gzlz/TESS-cont
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1051/0004-6361/202451656
    title: 'TOI-5005 b: A super-Neptune in the savanna near the ridge'
  source_content_hash: sha256:9ba99e73a9d34f60a3beb215c9f6de26a0499c9c2751f09aa70fe158b9e69579
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/castro-gzlz/TESS-cont
  tools:
  - Python
  - TESS-cont
  edam_operation: http://edamontology.org/operation_3695
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# tess-contaminant-flux-ranking

## Summary

Identifies and ranks the main Gaia DR2/DR3 contaminant sources contributing flux to a TESS target's photometric aperture using TESS-cont, which builds PRFs of nearby sources and computes their flux distributions across TPFs/FFIs.

## When to use

Use when you have a TIC/TOI or Simbad-resolvable target with TESS photometry and need to determine which nearby Gaia stars contaminate the SPOC aperture, whether inside or outside the TPF.

## When NOT to use

- When the goal is to confidently discard contaminant sources as the origin of a transit signal and TESS-cont DILUTION shows the contaminant depths (e.g. 2–28%) cannot be excluded from the TESS photometry alone — higher-spatial resolution observations (ground-based photometry/spectroscopy) are needed instead
- When generating custom apertures at a target-flux-percentage threshold — this feature is currently not documented in TESS-cont and requires contacting the author
- When you need a single authoritative catalogue run only — the README encourages running with BOTH DR2 and DR3 catalogues to ensure no major differences

## Inputs

- Target identifier resolvable as a TIC, TOI, or Simbad name (e.g. TIC 282485660 / TOI-5005)
- Configuration .ini file (e.g. TOI-5005_S65.ini) with mandatory 'target' parameter and optional parameters (sector, search_radius, n_sources, gaia_catalog, tpf_or_tesscut, cutout_size)
- TESS Target Pixel File (default) or FFI tesscut with cutout_size 11,11
- Gaia DR2 or DR3 catalogue query (DR3 default; DR2 for SPOC PDCSAP analysis)

## Outputs

- contaminant_sources.csv: ranked table with Star index, Gaia source_id, and inside/outside aperture flag for the top n_sources = 5 contaminants
- CROWDSAP and FLFRCSAP contamination metrics saved to metrics.dat
- Heatmap of flux percentage from the target star per pixel and pie chart of flux from target vs nearby stars inside (and outside) the aperture (img_fmt: pdfpng by default)

## How to apply

Resolve the target identifier (any TIC, TOI, or Simbad name, e.g. TOI-5005 = TIC 282485660) from a configuration .ini file to obtain sky coordinates. Load the TESS Target Pixel File (tpf_or_tesscut: tpf by default; use tesscut with cutout_size 11,11 for FFI targets) and query the Gaia catalogue around the target within search_radius = 200 arcsec, using gaia_catalog: DR3 by default but switching to DR2 when analyzing SPOC PDCSAP photometry since the TIC/SPOC products are stacked to Gaia DR2. Rank the retrieved sources by their flux contribution to the aperture and select the top n_sources = 5 contaminants to study individually. For each source, determine whether it falls inside or outside the SPOC photometric aperture and record its Gaia source_id. Save the ranked table (Star index, Gaia_ID, inside/outside aperture flag) as contaminant_sources.csv, then check that the by-product CROWDSAP and FLFRCSAP metrics written to metrics.dat show no major differences with the official TESS metrics — discrepancies indicate a DR3 vs DR2 catalogue mismatch.

## Related tools

- **TESS-cont** (Python tool that identifies the main contaminant Gaia DR2/DR3 sources, computes their flux contributions to the TESS aperture, and flags each source as inside/outside the SPOC photometric aperture) — https://github.com/castro-gzlz/TESS-cont
- **Python** (Execution environment; TESS-cont is run from the terminal or Jupyter Notebook via a configuration .ini file in the config folder) — https://github.com/castro-gzlz/TESS-cont

## Examples

```
python TESS-cont.py TOI-5005_S65.ini
```

## Evaluation signals

- contaminant_sources.csv contains one row per studied source with Star index, Gaia source_id, and an inside/outside aperture flag
- metrics.dat CROWDSAP/FLFRCSAP by-products show no major differences with the official TESS metrics — large discrepancies signal a DR3 vs DR2 catalogue mismatch
- Independent DR2 and DR3 runs produce consistent contamination results (if not, use the DR3 contamination metrics to correct PDCSAP/SAP photometry from crowding)
- Ranked list is limited to n_sources = 5 contaminants by default, retrieved within the search_radius = 200 arcsec Gaia query
- For the grounded example (TOI-5005, Sector 65), the target contributes 84.4% of in-aperture photometry with the remaining 15.6% from nearby sources

## Limitations

- Gaia DR3 vs DR2 discrepancies: TESS-cont defaults to DR3 but the TIC catalogue and SPOC PDCSAP are stacked to Gaia DR2, so catalog mismatch can distort CROWDSAP/FLFRCSAP validation
- The approximate PRF method (method_prf: approximate) interpolates the PRF only once at the middle of the TPF/FFI; it is a fast first hint, but the accurate method should be used for final analyses/publications
- Major contaminants can lie outside the TESS Target Pixel File entirely (e.g. Star 2 for TOI-5005), so contaminants are not always evident from the TPF alone
- Default PRF interpolation to all pixels with Gaia sources can take several minutes, though even crowded fields should not surpass ~5 minutes because shared-pixel PRFs are computed once

## Evidence

- [intro] TESS-cont is a Python tool to compute the flux contamination from nearby stars in the TESS aperture of any observed target.: "*TESS-cont* is a Python tool to **compute the flux contamination** from **nearby stars** in the **TESS aperture** of **any observed target**."
- [readme] The package identifies the main contaminant Gaia DR2/DR3 sources and computes their contributions to the aperture.: "The package **(1)** identifies the main contaminant *Gaia* DR2 / DR3 sources, **(2)** computes their contributions to the aperture"
- [readme] Default Gaia search radius is 200 arcsec.: "search_radius | Any number | Search radius of *Gaia* sources (in arcsec). **Default**: 200"
- [readme] Default Gaia DR3 must be swapped for DR2 when analyzing SPOC PDCSAP photometry because the TIC is stacked to DR2.: "By default, *TESS-cont* uses the *Gaia* DR3 catalogue. However, the TIC catalogue (and SPOC PDCSAP) is stacked to *Gaia* DR2."
- [readme] CROWDSAP and FLFRCSAP metrics are a by-product of TESS-cont and discrepancies with official TESS metrics point to DR3/DR2 mismatches.: "A **by-product** of the *TESS-cont* operation is the computation of the *CROWDSAP* and *FLFRCSAP* metrics. These are automatically saved in the metrics.dat"
- [methods] DR2 catalog should be selected to analyze TESS PDCSAP photometry.: "to analyze the TESS PDCSAP photometry the **DR2 catalog should be selected**"
