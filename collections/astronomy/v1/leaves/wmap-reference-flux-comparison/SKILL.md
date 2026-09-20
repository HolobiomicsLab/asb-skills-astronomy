---
name: wmap-reference-flux-comparison
description: Use when validating destriped 30–40 GHz polarized microwave maps (e.g.
  B-Machine 37.5 GHz HEALPix FITS maps) against an independent reference instrument's
  flux for a bright, well-characterized point source such as Tau A.
when_to_use_negative:
- The target source's U-component emission is expected to be detectable — the article
  notes Tau A emission in U is too faint to be measured, so U photometry is excluded
  here
- The maps have not been destriped — the validation is defined on destriped B-Machine
  maps, not raw timelines
- The reference instrument band differs substantially from 37.5–40 GHz, where spectral-index
  extrapolation would dominate the comparison
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3366
tools:
- name: B-Machine
  role: UCSB ground-based 37.5 GHz CMB polarimeter whose destriped maps provide the
    measured fluxes
- name: HEALPix
  role: Pixelization scheme of the destriped maps; patches are cut and photometry
    is performed on HEALPix FITS data
- name: Python
  role: Used to download the Figshare deposit and perform the flux measurement and
    comparison workflow
- name: dst
  role: Python parallel destriper that produced the destriped maps being validated
  repo: http://github.com/zonca/dst
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.ascom.2013.10.002
    title: Destriping cosmic microwave background polarimeter data
  source_content_hash: sha256:fc922de292a0e5dbc6d06544b620c1307552e8b6258cd7824d97479667c46878
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - B-Machine
  - HEALPix
  - Python
  - dst
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# wmap-reference-flux-comparison

## Summary

Validates a low-frequency polarimeter's destriped maps by measuring the integrated flux density of Tau A (Crab Nebula) via aperture photometry on I and Q patches and comparing the resulting fluxes to the WMAP 7-year Q-band reference, expecting agreement within 20% in I and 6% in Q.

## When to use

Use when validating destriped 30–40 GHz polarized microwave maps (e.g. B-Machine 37.5 GHz HEALPix FITS maps) against an independent reference instrument's flux for a bright, well-characterized point source such as Tau A.

## When NOT to use

- The target source's U-component emission is expected to be detectable — the article notes Tau A emission in U is too faint to be measured, so U photometry is excluded here
- The maps have not been destriped — the validation is defined on destriped B-Machine maps, not raw timelines
- The reference instrument band differs substantially from 37.5–40 GHz, where spectral-index extrapolation would dominate the comparison

## Inputs

- Destriped B-Machine channel 6 maps in HEALPix FITS format (Nside 512, 40-arcmin gaussian smoothing) from Figshare DOI 10.6084/m9.figshare.644507
- WMAP 7-year Q-band Tau A reference flux (Weiland et al. 2011, LAMBDA)
- Known Tau A (Crab Nebula) sky position for patch centering

## Outputs

- tau_a_flux_comparison.csv with columns for B-Machine flux, WMAP flux, and ratio in I and Q
- 5-degree gnomonic projection I and Q patches centered on Tau A
- Integrated flux density measurements of Tau A in I and Q

## How to apply

Download the publicly deposited destriped B-Machine channel 6 maps in HEALPix FITS format from the Figshare deposit (DOI 10.6084/m9.figshare.644507) and load the I, Q (and U) maps with healpy at Nside 512, matching the 40-arcmin gaussian-smoothed point-source maps. Cut out 5-degree gnomonic projection patches centered on the known Tau A (Crab Nebula) position. Measure the integrated flux density by aperture photometry on the I and Q patches; skip U since Tau A emission there is too faint to detect. Retrieve the WMAP 7-year Q-band reference flux for Tau A from Weiland et al. (2011, LAMBDA) and compute B-Machine/WMAP flux ratios in I and Q, writing all fluxes and ratios to a CSV table. Correct application is judged by the I ratio lying within 20% of unity and the Q ratio within 6% of unity, consistent with the reported agreement.

## Related tools

- **B-Machine** (UCSB ground-based 37.5 GHz CMB polarimeter whose destriped maps provide the measured fluxes)
- **HEALPix** (Pixelization scheme of the destriped maps; patches are cut and photometry is performed on HEALPix FITS data)
- **Python** (Used to download the Figshare deposit and perform the flux measurement and comparison workflow)
- **dst** (Python parallel destriper that produced the destriped maps being validated) — http://github.com/zonca/dst

## Examples

```
import healpy as hp; I, Q, U = hp.read_map('bmachine_ch6_destriped.fits', (0,1,2)); patch = hp.gnomview(I, rot=tau_a_pos, reso=5*60)
```

## Evaluation signals

- B-Machine/WMAP I flux ratio lies within 20% of unity
- B-Machine/WMAP Q flux ratio lies within 6% of unity
- All flux fields (B-Machine flux, WMAP flux, ratio) are present and populated in tau_a_flux_comparison.csv for both I and Q
- Patches are 5-degree gnomonic projections centered on the known Tau A position at Nside 512
- U is correctly excluded from the flux comparison because Tau A emission in U is too faint to detect

## Limitations

- The comparison validates only I and Q; no U-flux measurement is performed since Tau A emission in U is too faint to be detected
- The skill is tied to the WMAP 7-year Q-band reference (Weiland et al. 2011); other reference bands or epochs are not covered
- Aperture photometry assumes the destriped maps are point-source-preserving; destriping acts as a high-pass filter that only affects the noise in the timestream, and other systematic effects must be handled separately before map-making

## Evidence

- [methods] WMAP reference input: [evidence span withheld — anchor status: not_found]
- [intro] destriping as high-pass filter: [evidence span withheld — anchor status: not_found]
