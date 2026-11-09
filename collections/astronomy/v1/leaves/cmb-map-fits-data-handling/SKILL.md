---
name: cmb-map-fits-data-handling
description: Use when the input is a HEALPix FITS sky map from a CMB polarimeter (I,
  Q, U at Nside 512) and the goal is to extract, inspect, or cut out regions (e.g.
  5-degree patches around Tau A) for photometry or comparison.
when_to_use_negative:
- Input is still raw polarimeter timestream data (HDF5 timelines) needing destriping
  rather than already-mapped FITS products — use the dst destriper pipeline instead
- The target source has significant U emission that must be measured — the Tau A workflow
  here explicitly skips U because it is too faint to detect
- Systematic effects such as gain variations or spin-synchronous effects dominate,
  since destriping/mapping alone cannot correct them and they must be handled before
  map-making
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Python
  role: Driver language for downloading the Figshare deposit and loading/manipulating
    the HEALPix FITS maps
  repo: https://github.com/zonca/dst
- name: HEALPix
  role: Pixelization scheme of the maps; healpy routines read the FITS I/Q/U fields
    and support gnomonic patch extraction and partial-sky analysis
- name: dst
  role: Produces the destriped maps (parallel Python destriper) whose FITS outputs
    this skill then handles
  repo: https://github.com/zonca/dst
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.ascom.2013.10.002
    title: Destriping cosmic microwave background polarimeter data
  source_content_hash: sha256:8a52ee5aeec615fc203fc3826963a38270142494f06f0b755588d34868a6181a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/zonca/dst
  tools:
  - B-Machine
  - HEALPix
  - Python
  - dst
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# cmb-map-fits-data-handling

## Summary

Handles CMB polarimeter sky maps stored in HEALPix FITS format (e.g. the B-Machine 40 GHz destriped maps on Figshare), loading I/Q/U fields with healpy and preparing gnomonic patches for follow-on measurements such as Tau A aperture photometry.

## When to use

Use when the input is a HEALPix FITS sky map from a CMB polarimeter (I, Q, U at Nside 512) and the goal is to extract, inspect, or cut out regions (e.g. 5-degree patches around Tau A) for photometry or comparison.

## When NOT to use

- Input is still raw polarimeter timestream data (HDF5 timelines) needing destriping rather than already-mapped FITS products — use the dst destriper pipeline instead
- The target source has significant U emission that must be measured — the Tau A workflow here explicitly skips U because it is too faint to detect
- Systematic effects such as gain variations or spin-synchronous effects dominate, since destriping/mapping alone cannot correct them and they must be handled before map-making

## Inputs

- HEALPix FITS sky maps (destriped I, Q, U fields, Nside 512) from a CMB polarimeter, e.g. B-Machine channel 6
- Source catalog position (e.g. Tau A / Crab Nebula RA-Dec) for patch centering
- HDF5 timestream files with TIME, PHI, THETA, PSI, FLAG, TEMP, Q, U columns (raw data stage, per README)

## Outputs

- 5-degree gnomonic projection patches (I, Q) centered on the target source
- Integrated flux density measurements in I and Q (e.g. tau_a_flux_comparison.csv with B-Machine flux, WMAP flux, and ratio columns)

## How to apply

Download the destriped channel-6 maps in HEALPix FITS format from the Figshare deposit (DOI 10.6084/m9.figshare.644507) and load the I, Q, and U fields with healpy, confirming the Nside 512 pixelization. Cut out 5-degree gnomonic projection patches centered on known source positions (e.g. the Crab Nebula / Tau A), reproducing the 40-arcmin Gaussian-smoothed point-source maps of the paper's Fig. 10. For point-source flux work, apply aperture photometry to the I and Q patches; leave U unmeasured when the source's U emission is too faint to detect (as for Tau A). Validate against an independent reference (e.g. WMAP 7-year Q-band fluxes) by computing flux ratios in each Stokes parameter.

## Related tools

- **Python** (Driver language for downloading the Figshare deposit and loading/manipulating the HEALPix FITS maps) — https://github.com/zonca/dst
- **HEALPix** (Pixelization scheme of the maps; healpy routines read the FITS I/Q/U fields and support gnomonic patch extraction and partial-sky analysis)
- **dst** (Produces the destriped maps (parallel Python destriper) whose FITS outputs this skill then handles) — https://github.com/zonca/dst

## Examples

```
import healpy as hp; maps = hp.read_map('destriped_ch6.fits', field=(0,1), nest=False); patch = hp.gnomview(maps[1], rot=tau_a_rad, reso=1.0)
```

## Evaluation signals

- The FITS file loads correctly with healpy as I, Q, U fields at Nside 512, matching the deposited B-Machine map format
- Gnomonic patches are exactly 5 degrees across and centered on the known Tau A position
- I-band integrated flux agrees with WMAP 7-year Q-band reference within 20% of unity
- Q-band integrated flux agrees with WMAP within 6% of unity, consistent with the paper's reported agreement
- All flux fields (B-Machine flux, WMAP flux, ratio in I and Q) are present and non-empty in the output CSV table

## Limitations

- Destriping acts as a high-pass filter on the timestream only, so systematic effects (gain variations, spin-synchronous effects) must be corrected separately before map-making
- Data sections near sunrise and sunset are contaminated by thermally induced gain variations and must be masked out
- For Tau A, the U emission is too faint to detect, so U-based flux measurements are not meaningful with these maps

## Evidence

- [methods] Maps in HEALPix FITS format are publicly available on the Internet via Figshare: "Maps in HEALPix FITS format are publicly available on the Internet via Figshare"
- [readme] figshare deposit of destriped maps: "Destriped maps from the B-Machine 40GHz polarimeter are available on
figshare at: http://dx.doi.org/10.6084/m9.figshare.644507"
- [intro] HEALPix pixelization: [evidence span withheld — anchor status: not_found]
- [intro] destriping as a timestream high-pass filter: [evidence span withheld — anchor status: not_found]
- [methods] sunrise/sunset masking: [evidence span withheld — anchor status: not_found]
