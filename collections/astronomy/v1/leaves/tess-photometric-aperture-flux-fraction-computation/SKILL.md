---
name: tess-photometric-aperture-flux-fraction-computation
description: Use when a TESS TPF/FFI photometric aperture contains flux from Gaia
  nearby sources and you need to quantify each contaminant's contribution to the aperture
  or check whether a transit/activity signal could originate there.
when_to_use_negative:
- The goal is exoplanet parameter inference (radial velocities, transit fitting, internal
  structure modelling) rather than aperture flux-contamination quantification
- Only higher-spatial-resolution data (ground-based photometry/spectroscopy) can confidently
  discard contaminants — TESS photometry alone cannot discard contaminants whose DILUTION
  depths fall below 100%
- No Gaia sources fall within the aperture/search radius, so no contamination computation
  is meaningful
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: TESS-cont
  role: Main Python tool that identifies contaminant Gaia DR2/DR3 sources, builds
    their PRFs, computes per-pixel and per-aperture flux contributions, and renders
    heatmaps and pie charts
  repo: github.com/castro-gzlz/TESS-cont
- name: tpfplotter
  role: Machinery used to build the PRFs of nearby sources on the TESS TPF pixels
- name: TESS-PRF
  role: PRF-interpolation machinery underlying the flux distribution computation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1051/0004-6361/202451656
    title: 'TOI-5005 b: A super-Neptune in the savanna near the ridge'
  source_content_hash: sha256:11e158606ca2d55af7da07b90955bca765ebead095c1e81ab53b499ec3fede8a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - TESS
  - TESS-PRF
  - TESS-cont
  - tpfplotter
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# tess-photometric-aperture-flux-fraction-computation

## Summary

Computes the per-pixel and per-source flux contributions of nearby stars to a TESS photometric aperture by building PRFs of Gaia DR2/DR3 contaminants and locating them on the TESS TPF/FFI pixels, yielding the target's flux fraction (e.g. TOI-5005 contributes 84.4% in Sector 65) and CROWDSAP/FLFRCSAP metrics.

## When to use

Use when a TESS TPF/FFI photometric aperture contains flux from Gaia nearby sources and you need to quantify each contaminant's contribution to the aperture or check whether a transit/activity signal could originate there.

## When NOT to use

- The goal is exoplanet parameter inference (radial velocities, transit fitting, internal structure modelling) rather than aperture flux-contamination quantification
- Only higher-spatial-resolution data (ground-based photometry/spectroscopy) can confidently discard contaminants — TESS photometry alone cannot discard contaminants whose DILUTION depths fall below 100%
- No Gaia sources fall within the aperture/search radius, so no contamination computation is meaningful

## Inputs

- Configuration file (e.g. TOI-5005_S65.ini) with target identifier (TIC/TOI/Simbad name) and sector
- TESS Target Pixel File or FFI tesscut (default cutout_size 11,11)
- Gaia DR2 or DR3 source positions and fluxes within a search radius (default 200 arcsec)
- Optional measured transit depth (td with td_unit ppm/ppt/per/frac and dilution_corr flag) for the DILUTION analysis

## Outputs

- Heatmap of the target star's per-pixel flux percentage (viridis colormap, natural scale, disk sizes scaled by emitted flux)
- Pie chart of target vs. contaminant flux fractions inside (and outside) the aperture
- Per-contaminant flux percentages (e.g. TOI-5005 84.4%, nearby sources 15.6% in Sector 65)
- CROWDSAP and FLFRCSAP metrics written to metrics.dat
- Output images in pdf, png, or pdfpng format

## How to apply

Read a configuration .ini file specifying the TIC/TOI (or Simbad-resolvable name) and sector, with defaults gaia_catalog=DR3, search_radius=200 arcsec, n_sources=5, method_prf=accurate. Retrieve the SPOC Target Pixel File (or an 11,11-pixel FFI tesscut via tpf_or_tesscut) together with positions and fluxes of Gaia sources within the search radius. Build each source's PRF with the tpfplotter/TESS-PRF machinery, interpolating to all pixels containing Gaia sources (accurate mode) or once at the TPF/FFI centre (approximate mode), then locate each PRF at its sky position and compute its pixel-by-pixel flux distribution, computing shared-pixel PRFs only once so runtime stays within ~5 minutes. Sum per-pixel contributions inside the SPOC aperture to obtain per-contaminant flux percentages, and optionally run the DILUTION analysis by supplying the measured transit depth (td, td_unit in ppm/ppt/per/frac, dilution_corr=True for SPOC depths) to test whether contaminants could produce the observed signal. Render the viridis heatmap of target flux percentage per pixel (scale_factor=4000, natural scale) and the pie chart of target vs contaminant flux inside the aperture, saving images with img_fmt=pdfpng.

## Related tools

- **TESS-cont** (Main Python tool that identifies contaminant Gaia DR2/DR3 sources, builds their PRFs, computes per-pixel and per-aperture flux contributions, and renders heatmaps and pie charts) — github.com/castro-gzlz/TESS-cont
- **tpfplotter** (Machinery used to build the PRFs of nearby sources on the TESS TPF pixels)
- **TESS-PRF** (PRF-interpolation machinery underlying the flux distribution computation)

## Examples

```
python TESS-cont.py TOI-5005_S65.ini
```

## Evaluation signals

- Derived CROWDSAP and FLFRCSAP values in metrics.dat match the official TESS metrics; major discrepancies likely indicate a Gaia DR2 vs DR3 catalogue mismatch
- Heatmap shows the target's flux percentage per pixel (e.g. 60–90% inside the aperture for TOI-5005) and the pie chart sums to 100% of aperture flux
- Running with both gaia_catalog=DR2 and DR3 produces consistent contamination metrics; differences should be resolved with DR3 values for crowding correction
- Approximate and accurate method_prf modes give very similar flux fractions; accurate mode is required for final publications
- DILUTION output transit depths for the n_sources contaminants (e.g. 2–28% for TOI-4479.01) are physically plausible (< 100%)

## Limitations

- The approximate PRF method interpolates once at the TPF/FFI centre and is only a first hint; the accurate method should be used for final analyses/publications
- The TIC catalogue and SPOC PDCSAP are stacked to Gaia DR2 while TESS-cont defaults to DR3, so DR2 must be selected for PDCSAP analysis and both catalogues should be run to check for discrepancies
- The custom aperture generation feature at a target-flux-percentage threshold is currently not documented
- DILUTION analysis based on TESS photometry alone often cannot discard contaminants as transit origins (depths below the 100% threshold), requiring higher-spatial-resolution follow-up

## Evidence

- [methods] TOI-5005 contributes 84.4% to the measured photometry, while the remaining 15.6% comes from different nearby sources: "**TOI-5005 contributes 84.4%** to the measured photometry, while **the remaining 15.6% comes from different nearby sources**"
- [readme] The standard plotting output includes heatmaps and pie charts: "The standard plotting output includes **heatmaps showing the flux fraction coming from the target star** and **pie charts highlighting the most contaminant stars** inside (and outside!) the aperture."
- [readme] By default, TESS-cont uses the Gaia DR3 catalogue but the TIC catalogue (and SPOC PDCSAP) is stacked to Gaia DR2: "By default, *TESS-cont* uses the *Gaia* DR3 catalogue. However, the TIC catalogue (and SPOC PDCSAP) is stacked to *Gaia* DR2."
- [readme] A by-product of the TESS-cont operation is the computation of the CROWDSAP and FLFRCSAP metrics: "A **by-product** of the *TESS-cont* operation is the computation of the *CROWDSAP* and *FLFRCSAP* metrics."
- [readme] We encourage to use the accurate method for final analyses/publications: "However, **we encourage to use the accurate method for final analyses/publications**. We have coded *TESS-cont* so that the PRFs of stars in common pixels are only computed once"
