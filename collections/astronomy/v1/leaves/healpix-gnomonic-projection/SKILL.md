---
name: healpix-gnomonic-projection
description: Use when a HEALPix full-sky map (e.g. B-Machine destriped I/Q/U FITS
  maps) must be converted to small projected patches centered on a known point source
  for flux measurement or visual inspection.
when_to_use_negative:
- The target source's polarized U emission is detectable and needs measuring — the
  paper notes Tau A emission in U is too faint to be detected, so U patches were not
  measured
- The analysis requires full-sky or large-area statistics (e.g. angular power spectra),
  where gnomonic patches discard sky coverage
- The map is not in HEALPix format or the source position is unknown and must first
  be detected blind
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: HEALPix
  role: Pixelization scheme of the destriped maps; provides the gnomonic patch extraction
    of the Nside 512 maps
- name: Python
  role: Used with healpy to download the Figshare FITS maps, load I/Q/U, cut out patches,
    and run aperture photometry
  repo: http://github.com/zonca/dst
- name: B-Machine
  role: 'Source instrument: 37.5 GHz ground-based CMB polarimeter whose destriped
    maps are projected to patches'
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.ascom.2013.10.002
    title: Destriping cosmic microwave background polarimeter data
  source_content_hash: sha256:9883d3b400f28b8adc12a235b1a3d3e8e7c91a96a41136faaa358202bcf2e8b1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - B-Machine
  - HEALPix
  - Python
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# healpix-gnomonic-projection

## Summary

Extract small flat-sky patches from HEALPix maps via gnomonic (gnomview) projection so point sources like Tau A can be measured with aperture photometry. Applied to the Nside 512, 40-arcmin gaussian-smoothed B-Machine destriped I/Q/U maps from the 37.5 GHz polarimeter.

## When to use

Use when a HEALPix full-sky map (e.g. B-Machine destriped I/Q/U FITS maps) must be converted to small projected patches centered on a known point source for flux measurement or visual inspection.

## When NOT to use

- The target source's polarized U emission is detectable and needs measuring — the paper notes Tau A emission in U is too faint to be detected, so U patches were not measured
- The analysis requires full-sky or large-area statistics (e.g. angular power spectra), where gnomonic patches discard sky coverage
- The map is not in HEALPix format or the source position is unknown and must first be detected blind

## Inputs

- HEALPix FITS destriped I, Q, U maps (B-Machine channel 6, Nside 512, 40-arcmin gaussian smoothing, from figshare 10.6084/m9.figshare.644507)
- Known sky coordinates of the target point source (Tau A / Crab Nebula position)
- Reference integrated flux density (WMAP 7-year Q-band, Weiland et al. 2011, LAMBDA)

## Outputs

- 5-degree gnomonic projection patches of I and Q centered on Tau A
- Integrated flux density of Tau A in I and Q from aperture photometry
- CSV table (e.g. tau_a_flux_comparison.csv) with B-Machine flux, WMAP flux, and ratios in I and Q

## How to apply

Download the B-Machine destriped maps in HEALPix FITS format from the Figshare deposit (10.6084/m9.figshare.644507) and load the I, Q (and U) maps with healpy. Identify the known sky position of the target (Tau A / Crab Nebula) and cut out a 5-degree gnomonic projection patch centered on that position, matching the Nside 512, 40-arcmin gaussian-smoothed point-source maps shown in Fig. 10 of the paper. Run aperture photometry on the I and Q patches to get integrated flux densities; U is skipped because Tau A emission in U is too faint to be detected. Compare the resulting fluxes to a reference (e.g. WMAP 7-year Q-band flux from Weiland et al. 2011, LAMBDA) to validate the measurement.

## Related tools

- **HEALPix** (Pixelization scheme of the destriped maps; provides the gnomonic patch extraction of the Nside 512 maps)
- **Python** (Used with healpy to download the Figshare FITS maps, load I/Q/U, cut out patches, and run aperture photometry) — http://github.com/zonca/dst
- **B-Machine** (Source instrument: 37.5 GHz ground-based CMB polarimeter whose destriped maps are projected to patches)

## Examples

```
import healpy as hp; mI, mQ, mU = hp.read_map('destriped.fits', (0,1,2)); hp.gnomview(mI, rot=tau_a_rad, xsize=512, reso=2.34)
```

## Evaluation signals

- The I flux ratio B-Machine/WMAP Q-band must lie within 20% of unity
- The Q flux ratio must lie within 6% of unity, consistent with the reported '20% in I and 6% in Q' agreement
- All flux fields (B-Machine flux, WMAP flux, ratios in I and Q) must be present and non-empty in tau_a_flux_comparison.csv
- Patch size and centering match the paper's presentation: 5-degree gnomonic patches centered on the known Tau A position at Nside 512 with 40-arcmin gaussian smoothing
- No flux is reported for U, since Tau A U emission is expected to be below detection

## Limitations

- U flux cannot be measured for faint polarization sources: Tau A emission in U is too faint to be detected in the B-Machine maps
- Destriped map quality depends on prior masking of sunrise/sunset sections contaminated by thermally induced gain variations before map-making
- The destriped maps assume destriping limitations were handled: systematic effects like gain variations or spin-synchronous effects must be dealt with before the map-making stage

## Evidence

- [methods] The input signal was based on WMAP 7 years Q band, smoothed by 1 degree in order to reduce the noise in map domain: [evidence span withheld — anchor status: not_found]
