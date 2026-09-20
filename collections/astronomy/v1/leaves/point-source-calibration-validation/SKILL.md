---
name: point-source-calibration-validation
description: Use when a newly destriped low-frequency microwave sky map needs absolute
  calibration validation against a known bright point source, or when reproducing
  the B-Machine '20% in I, 6% in Q' Tau A agreement check.
when_to_use_negative:
- Tau A emission in the U Stokes parameter is too faint to be detected, so this skill
  must not be used to calibrate U
- The maps have not been destriped (low-frequency correlated noise not removed), since
  correlated-noise residuals would bias the aperture photometry
- The target source is extended diffuse foreground emission rather than a compact
  point source matching the 40-arcmin gaussian-smoothed point-source maps
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3366
tools:
- name: Python
  role: Downloading the Figshare deposit, running aperture photometry on the map patches,
    and writing the flux comparison CSV
  repo: http://github.com/zonca/dst
- name: HEALPix
  role: Map pixelization scheme of the destriped FITS maps; used to load I, Q, U maps
    and cut gnomonic projection patches
- name: B-Machine
  role: The 37.5 GHz polarimeter whose destriped channel 6 maps provide the flux being
    validated
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.ascom.2013.10.002
    title: Destriping cosmic microwave background polarimeter data
  source_content_hash: sha256:ee3c02c3da717a6e8b261c2880475cc20b64c103a7a61c777f2ea182c0f85250
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - B-Machine
  - HEALPix
  - Python
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# point-source-calibration-validation

## Summary

Validates the absolute polarization/temperature calibration of a ground-based CMB polarimeter (B-Machine, 37.5 GHz) by measuring the integrated flux density of the bright point source Tau A (Crab Nebula) on destriped HEALPix maps and comparing it to the WMAP 7-year Q-band reference flux.

## When to use

Use when a newly destriped low-frequency microwave sky map needs absolute calibration validation against a known bright point source, or when reproducing the B-Machine '20% in I, 6% in Q' Tau A agreement check.

## When NOT to use

- Tau A emission in the U Stokes parameter is too faint to be detected, so this skill must not be used to calibrate U
- The maps have not been destriped (low-frequency correlated noise not removed), since correlated-noise residuals would bias the aperture photometry
- The target source is extended diffuse foreground emission rather than a compact point source matching the 40-arcmin gaussian-smoothed point-source maps

## Inputs

- Destriped B-Machine channel 6 maps in HEALPix FITS format (from Figshare DOI 10.6084/m9.figshare.644507)
- Known Tau A (Crab Nebula) sky position
- WMAP 7-year Q-band reference flux for Tau A (Weiland et al. 2011, LAMBDA)

## Outputs

- tau_a_flux_comparison.csv with columns for B-Machine flux, WMAP flux, and ratio in I and Q
- Integrated flux density measurements of Tau A in I and Q from aperture photometry on 5-degree gnomonic patches

## How to apply

Download the destriped B-Machine channel 6 maps (Nside 512, 40-arcmin gaussian-smoothed point-source maps) in HEALPix FITS format from the Figshare deposit (10.6084/m9.figshare.644507). Load the destriped I, Q, and U maps with healpy and cut out 5-degree gnomonic projection patches centered on the known Tau A (Crab Nebula) position. Measure the integrated flux density by aperture photometry on the I and Q patches; skip U because Tau A emission in U is too faint to be detected. Retrieve the WMAP 7-year Q-band reference flux (Weiland et al. 2011, via LAMBDA) and compute B-Machine/WMAP flux ratios, writing the results to a CSV table with B-Machine flux, WMAP flux, and ratio columns for both I and Q.

## Related tools

- **Python** (Downloading the Figshare deposit, running aperture photometry on the map patches, and writing the flux comparison CSV) — http://github.com/zonca/dst
- **HEALPix** (Map pixelization scheme of the destriped FITS maps; used to load I, Q, U maps and cut gnomonic projection patches)
- **B-Machine** (The 37.5 GHz polarimeter whose destriped channel 6 maps provide the flux being validated)

## Examples

```
import healpy as hp; mI, mQ, mU = hp.read_map('bmachine_ch6_destriped.fits', (0,1,2)); patch = hp.gnomview(mI, rot=tau_a_pos, reso=arcmin, xsize=patch_size)
```

## Evaluation signals

- The I flux ratio (B-Machine/WMAP) must lie within 20% of unity, consistent with the reported '20% in I' agreement
- The Q flux ratio must lie within 6% of unity, consistent with the reported '6% in Q' agreement
- All flux fields (B-Machine flux, WMAP flux, ratio in I and Q) must be present in tau_a_flux_comparison.csv
- Map patches must be 5-degree gnomonic projections centered on the known Tau A position at Nside 512, matching the 40-arcmin gaussian-smoothed point-source maps of Fig. 10

## Limitations

- Tau A emission in the U Stokes parameter is too faint to be detected, so no U flux measurement is possible
- Destriping acts as a high-pass filter on the timestream noise; systematic effects like thermally induced gain variations near sunrise/sunset must be masked out before map-making, otherwise flux measurements can be biased
- Validation is limited to the Q-band (20-40 GHz regime) comparison with WMAP; spectral extrapolation to other bands is not covered

## Evidence

- [methods] sections near sunrise and sunset contaminated: [evidence span withheld — anchor status: not_found]
