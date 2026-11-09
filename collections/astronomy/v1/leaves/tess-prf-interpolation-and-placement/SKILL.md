---
name: tess-prf-interpolation-and-placement
description: Use when you need to quantify how much flux Gaia neighbours contribute
  inside a TESS SPOC aperture or FFI cutout — e.g. to compute CROWDSAP/FLFRCSAP, correct
  dilution, or attribute a transit/activity signal to contaminant stars.
when_to_use_negative:
- The analysis goal is to definitively confirm which star hosts a transit/activity
  signal — the README notes higher-spatial resolution observations (ground-based photometry/spectroscopy)
  are needed to confidently discard contaminants when the DILUTION depth thresholds
  are below the unphysical 100% limit
- Working with non-TESS data (no TPF or FFI pixels onto which Gaia-source PRFs can
  be placed) — the skill is specific to TESS TPFs/FFIs
- Only the PRF shape itself is needed for one source at a fixed position without aperture
  flux accounting — the surrounding aperture-summing machinery of TESS-cont is unnecessary
  overhead
tools:
- name: TESS-cont
  role: Python tool that orchestrates PRF interpolation, placement at Gaia sky positions,
    aperture flux accounting, heatmap/pie-chart rendering, and CROWDSAP/FLFRCSAP computation
  repo: https://github.com/castro-gzlz/TESS-cont
- name: tpfplotter
  role: Machinery used by TESS-cont to build the PRFs of nearby sources
- name: TESS-PRF
  role: Library used by TESS-cont to build and interpolate the TESS PRF at each source
    position
- name: TESS
  role: Provides the TPF/FFI target pixel data and the SPOC photometric aperture on
    which PRFs are placed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1051/0004-6361/202451656
    title: 'TOI-5005 b: A super-Neptune in the savanna near the ridge'
  source_content_hash: sha256:3142066c12c98af6157c38a661e39d8134c08f44ae3237918f3831c3570496a5
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
yes_rate: 0.6666666666666666
n_claims: 9
---

# tess-prf-interpolation-and-placement

## Summary

Builds the TESS PRF of each nearby Gaia DR2/DR3 source, interpolates it onto the pixels of the TESS TPF/FFI cutout, and places it at each source's sky position so per-source flux contributions inside the SPOC photometric aperture can be computed pixel-by-pixel.

## When to use

Use when you need to quantify how much flux Gaia neighbours contribute inside a TESS SPOC aperture or FFI cutout — e.g. to compute CROWDSAP/FLFRCSAP, correct dilution, or attribute a transit/activity signal to contaminant stars.

## When NOT to use

- The analysis goal is to definitively confirm which star hosts a transit/activity signal — the README notes higher-spatial resolution observations (ground-based photometry/spectroscopy) are needed to confidently discard contaminants when the DILUTION depth thresholds are below the unphysical 100% limit
- Working with non-TESS data (no TPF or FFI pixels onto which Gaia-source PRFs can be placed) — the skill is specific to TESS TPFs/FFIs
- Only the PRF shape itself is needed for one source at a fixed position without aperture flux accounting — the surrounding aperture-summing machinery of TESS-cont is unnecessary overhead

## Inputs

- TESS Target Pixel File (TPF) for the target/sector, or an FFI tesscut (default cutout_size 11,11)
- Configuration .ini file with at least a mandatory target identifier (TIC, TOI, or Simbad-resolvable name), e.g. TOI-5005_S65.ini
- Gaia DR3 (default) or DR2 positions and fluxes of nearby sources within search_radius=200 arcsec, limited to n_sources=5 individually studied contaminants
- Optional DILUTION parameters: measured transit depth td with td_unit (ppm, ppt, per, frac) and dilution_corr flag

## Outputs

- Per-source, pixel-by-pixel PRF flux-distribution maps across the TPF/FFI
- Per-contaminant flux percentages inside the SPOC photometric aperture (pie chart of target vs. nearby-source flux)
- Heatmap of the target's flux percentage per pixel (viridis colormap, natural scale, scale_factor 4000) and pie chart, saved as pdf/png (img_fmt=pdfpng)
- CROWDSAP and FLFRCSAP metrics written to metrics.dat
- Optional custom aperture meeting a chosen target-flux threshold (e.g. 80%)
- DILUTION analysis: per-contaminant transit depths (e.g. 1.74–27.69% for the six most contaminant stars of TOI-4479)

## How to apply

Retrieve the TESS Target Pixel File (or an FFI tesscut of size 11,11) for the target/sector together with positions and fluxes of Gaia DR3 (default) sources within search_radius=200 arcsec. Build the PRF of each nearby source using the tpfplotter/TESS-PRF machinery, interpolating the PRF to all pixels containing Gaia sources (method_prf: accurate) or only once at the TPF/FFI centre (method_prf: approximate). Locate each source PRF at its sky position and compute its flux distribution pixel-by-pixel, coding shared-pixel PRFs only once so that runtime stays within ~5 minutes even in highly crowded fields. Sum the per-pixel contributions inside the SPOC photometric aperture to obtain per-contaminant flux percentages (e.g. TOI-5005 contributing 84.4% in Sector 65 with 15.6% from nearby sources). Render the heatmap of target flux percentages (colormap viridis, natural scale, disk sizes scaled with default scale_factor 4000) and the pie chart of target vs. contaminant flux inside the aperture, saving images as pdfpng. Finally, cross-check the derived CROWDSAP and FLFRCSAP written to metrics.dat against the official TESS metrics, and run with both gaia_catalog: DR2 and DR3 since the TIC catalogue and SPOC PDCSAP are stacked to Gaia DR2.

## Related tools

- **TESS-cont** (Python tool that orchestrates PRF interpolation, placement at Gaia sky positions, aperture flux accounting, heatmap/pie-chart rendering, and CROWDSAP/FLFRCSAP computation) — https://github.com/castro-gzlz/TESS-cont
- **tpfplotter** (Machinery used by TESS-cont to build the PRFs of nearby sources)
- **TESS-PRF** (Library used by TESS-cont to build and interpolate the TESS PRF at each source position)
- **TESS** (Provides the TPF/FFI target pixel data and the SPOC photometric aperture on which PRFs are placed)

## Examples

```
python TESS-cont.py TOI-5005_S65.ini
```

## Evaluation signals

- CROWDSAP and FLFRCSAP in metrics.dat match the official TESS metrics; major differences flag a Gaia DR2 vs DR3 catalogue discrepancy
- Target plus contaminant fractions in the aperture pie chart sum to 100% (e.g. TOI-5005: 84.4% target + 15.6% nearby sources in Sector 65)
- Heatmap flux percentages inside the aperture fall in a physically plausible per-pixel range (Example 1: TOI-5005 contributes between 60% and 90% per pixel)
- Runtime does not surpass ~5 minutes even in highly crowded fields, confirming shared-pixel PRFs were computed only once
- DILUTION per-star transit depths are physical (below the 100% unphysical threshold), e.g. the 2–28% range reported for TOI-4479.01

## Limitations

- The approximate method interpolates the PRF only once at the middle of the TPF/FFI, assuming its shape does not vary much across nearby pixels; the accurate method (interpolating to all Gaia pixels) is encouraged for final analyses/publications
- Discrepancies between Gaia DR3 and DR2 can cause CROWDSAP/FLFRCSAP to differ from the official TESS metrics; DR2 should be selected to analyze SPOC PDCSAP photometry
- The custom aperture generation feature (e.g. an 80% target-flux threshold) is currently undocumented
- Based on the TESS photometry alone, contaminant stars cannot be ruled out as the origin of a transit signal whenever the derived depths fall below the unphysical 100% threshold

## Evidence

- [intro] intro: "*TESS-cont* is a Python tool to **compute the flux contamination** from **nearby stars** in the **TESS aperture** of **any observed target**."
- [methods] methods: "By default, *TESS-cont* uses the *Gaia* DR3 catalogue. However, the TIC catalogue (and SPOC PDCSAP) is stacked to *Gaia* DR2."
- [readme] readme: "The algorithm is based on building the PRFs of nearby sources and computing their flux distributions across the TESS TPFs/FFIs."
