---
name: transit-light-curve-dilution-correction
description: Use when a TESS target's photometric aperture contains flux from nearby
  Gaia sources and you need to correct or bound the measured transit depth, or to
  test whether contaminants could produce the observed transit.
when_to_use_negative:
- When the goal is the orbital/physical parameter fit itself (joint Bayesian RV +
  photometry) — contamination analysis is a precursor/validation step, not the Keplerian
  fit
- When the field is effectively uncontaminated and the aperture contains only the
  target star (near-zero contaminant flux) — dilution correction is then a no-op
- When only the approximate PRF method has been run — the README states the accurate
  method should be used for final analyses/publications
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3303
tools:
- name: TESS-cont
  role: Computes pixel-by-pixel flux contamination from nearby Gaia sources in the
    TESS aperture, performs the DILUTION analysis to test contaminant-origin hypotheses,
    and yields CROWDSAP/FLFRCSAP metrics
  repo: https://github.com/castro-gzlz/TESS-cont
- name: Python
  role: Runtime environment; TESS-cont is invoked via CLI with a .ini config file
    or through the provided Jupyter notebook
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202451656
    title: 'TOI-5005 b: A super-Neptune in the savanna near the ridge'
  source_content_hash: sha256:ccc1c7b12b308c31f5d7ba6019ed26ceb15922b63f7b729ae81b9c2bc6d373c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/castro-gzlz/TESS-cont
  tools:
  - HARPS
  - PEST
  - Python
  - TESS-cont
  - TRAPPIST-South
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3303
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# transit-light-curve-dilution-correction

## Summary

Quantifies how much of a TESS photometric aperture's flux comes from nearby stars (via TESS-cont's PRF-based contamination modelling and DILUTION feature) and uses that to assess or correct transit depth dilution, deciding whether a nearby Gaia source could be the true origin of the transit signal.

## When to use

Use when a TESS target's photometric aperture contains flux from nearby Gaia sources and you need to correct or bound the measured transit depth, or to test whether contaminants could produce the observed transit.

## When NOT to use

- When the goal is the orbital/physical parameter fit itself (joint Bayesian RV + photometry) — contamination analysis is a precursor/validation step, not the Keplerian fit
- When the field is effectively uncontaminated and the aperture contains only the target star (near-zero contaminant flux) — dilution correction is then a no-op
- When only the approximate PRF method has been run — the README states the accurate method should be used for final analyses/publications

## Inputs

- Target identifier: any TIC, TOI, or Simbad-resolvable star name (e.g. TIC 282485660 / TOI-5005)
- TESS sector number (optional; defaults to first with observations)
- TESS Target Pixel Files (TPFs) or FFI tesscut cutouts (default cutout_size 11,11)
- Gaia DR2 and/or DR3 source catalogues for nearby sources (default search_radius 200 arcsec, n_sources 5)
- Measured transit depth in the target's aperture (td, with td_unit in ppm/ppt/per/frac) and a flag for whether the depth is dilution-corrected (e.g. SPOC PDCSAP)

## Outputs

- Heatmap of per-pixel flux percentage from the target star (e.g. 60–90% inside the TOI-5005 aperture)
- Pie chart of target vs. nearby-star flux inside (and outside) the photometric aperture
- Per-contaminant transit depths implied by DILUTION analysis (e.g. Star,Gaia_ID,transit_depth(%) rows for TOI-4479.01)
- CROWDSAP and FLFRCSAP contamination metrics saved to metrics.dat
- Optional custom aperture at a chosen target-flux threshold (e.g. 80%)

## How to apply

Run TESS-cont with the target's TIC/TOI identifier (e.g. TIC 282485660, TOI-5005, Sector 65) to build PRFs of nearby Gaia DR2/DR3 sources and compute their pixel-by-pixel flux contributions to the SPOC aperture; the tool reports the target's total flux fraction (e.g. TOI-5005 contributes 84.4%, leaving 15.6% contamination) and the CROWDSAP/FLFRCSAP metrics. For the dilution analysis, supply the measured transit depth (with td_unit set to ppm, ppt, per, or frac) via the [DILUTION] section; set dilution_corr: True when the depth comes from SPOC, since SPOC depths are already corrected for dilution. TESS-cont then computes, for each of the top contaminant stars, the transit depth the signal would imply in that star; if any implied depth exceeds the unphysical 100% threshold, that contaminant is ruled out as the signal's origin — as in the TOI-4479.01 example where implied depths of 2–28% could not rule out six contaminant stars. When analyzing PDCSAP photometry, select gaia_catalog: DR2 because the TIC/SPOC stack is tied to Gaia DR2, and cross-check against DR3; use the accurate PRF method for final analyses rather than the approximate method.

## Related tools

- **TESS-cont** (Computes pixel-by-pixel flux contamination from nearby Gaia sources in the TESS aperture, performs the DILUTION analysis to test contaminant-origin hypotheses, and yields CROWDSAP/FLFRCSAP metrics) — https://github.com/castro-gzlz/TESS-cont
- **Python** (Runtime environment; TESS-cont is invoked via CLI with a .ini config file or through the provided Jupyter notebook)

## Examples

```
python TESS-cont.py TOI-4479_S41.ini  # with [DILUTION] td=3471, td_unit=ppm, dilution_corr=True
```

## Evaluation signals

- Compare the tool's CROWDSAP/FLFRCSAP output in metrics.dat against the official TESS metrics; major discrepancies likely indicate a Gaia DR2 vs DR3 catalogue mismatch
- Target flux fraction should be reported for the aperture (e.g. TOI-5005 at 84.4% in Sector 65, per-pixel 60–90%) — verify the sum over contaminants plus target equals ~100%
- DILUTION output: any contaminant with an implied transit depth above the unphysical 100% threshold can be discarded as the signal's origin; depths below 100% mean the contaminant cannot be ruled out from TESS photometry alone
- Ensure gaia_catalog matches the photometry pipeline (DR2 for SPOC PDCSAP); run with both DR2 and DR3 and check for major differences
- Confirm the accurate PRF method was used (not method_prf: approximate) for publication-grade results

## Limitations

- If implied contaminant depths fall below 100%, TESS photometry alone cannot rule out contaminants; higher-spatial-resolution observations (ground-based photometry or spectroscopy) are required
- Discrepancies between the DR3 and DR2 Gaia catalogues can shift contamination metrics; the tool defaults to DR3 while SPOC PDCSAP is stacked to DR2
- The custom-aperture feature is currently not documented in the README
- The approximate PRF method gives similar but not identical results to the accurate method; use accurate for final analyses

## Evidence

- [methods] DILUTION feature to discard contaminants: "We here use the *TESS-cont* **DILUTION** feature **to know whether we can discard the nearby contaminant sources as being the origin of the signal found**"
- [readme] SPOC depths already dilution-corrected: "dilution_corr: True``` (since the SPOC transit depths are already corrected for dilution)"
- [readme] TOI-5005 aperture contamination fraction: "**TOI-5005 contributes 84.4%** to the measured photometry, while **the remaining 15.6% comes from different nearby sources**"
- [readme] unphysical 100% depth threshold: "Since those values (**2-28%**) are lower than the unphysical threshold of 100%, we cannot discard that they could be the origin of the transit signal"
- [methods] DR2 required for PDCSAP: "to analyze the TESS PDCSAP photometry the **DR2 catalog should be selected** as an [OPTIONAL](#optional--optional-parameters) parameter: ```gaia_catalog: DR2```"
