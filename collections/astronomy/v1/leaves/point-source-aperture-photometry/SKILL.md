---
name: point-source-aperture-photometry
description: Use when an integrated flux density for a compact, bright point source
  must be measured from HEALPix I/Q/U map products (e.g., the B-Machine 37.5 GHz destriped
  maps) and compared to a reference instrument such as WMAP Q-band.
when_to_use_negative:
- The target's polarized emission in U is too faint to be detected (e.g., Tau A),
  so aperture photometry on the U map would not yield a usable flux
- The data are still raw timestreams rather than destriped maps — destriping must
  be done first (dst pipeline), since correlated-noise baselines bias map-domain fluxes
- The source is extended diffuse emission rather than a compact point source suitable
  for aperture photometry on a 5-degree patch
tools:
- name: Python
  role: Driving language for downloading the Figshare deposit, running aperture photometry,
    and writing the comparison CSV
  repo: http://github.com/zonca/dst
- name: HEALPix
  role: Map scheme used to load the destriped I, Q, U FITS maps and cut 5-degree gnomonic
    patches centered on Tau A
- name: B-Machine
  role: UCSB ground-based 37.5 GHz polarimeter whose destriped maps provide the I/Q
    data being photometered
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.ascom.2013.10.002
    title: Destriping cosmic microwave background polarimeter data
  source_content_hash: sha256:aeaf5ef9641b3e39fb13378d633c8ba4a9cb7f845e5df1257764bf32b1c75ea2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - B-Machine
  - HEALPix
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# point-source-aperture-photometry

## Summary

Measures the integrated flux density of a compact source (here Tau A / Crab Nebula) from destriped B-Machine Nside 512 HEALPix I, Q, U maps via aperture photometry on gnomonic patches, and validates the result against the WMAP 7-year Q-band reference flux in intensity and polarization.

## When to use

Use when an integrated flux density for a compact, bright point source must be measured from HEALPix I/Q/U map products (e.g., the B-Machine 37.5 GHz destriped maps) and compared to a reference instrument such as WMAP Q-band.

## When NOT to use

- The target's polarized emission in U is too faint to be detected (e.g., Tau A), so aperture photometry on the U map would not yield a usable flux
- The data are still raw timestreams rather than destriped maps — destriping must be done first (dst pipeline), since correlated-noise baselines bias map-domain fluxes
- The source is extended diffuse emission rather than a compact point source suitable for aperture photometry on a 5-degree patch

## Inputs

- Destriped B-Machine channel 6 maps in HEALPix FITS format (Nside 512, Gaussian-smoothed) from figshare 10.6084/m9.figshare.644507
- Known sky position of Tau A (Crab Nebula)
- WMAP 7-year Q-band reference flux for Tau A (Weiland et al. 2011, LAMBDA)

## Outputs

- Integrated flux density of Tau A in I and Q from the B-Machine maps
- B-Machine/WMAP flux ratios in I and Q
- CSV table tau_a_flux_comparison.csv with columns for B-Machine flux, WMAP flux, and ratio in I and Q

## How to apply

Download the destriped B-Machine channel 6 maps in HEALPix FITS format from the Figshare deposit (doi 10.6084/m9.figshare.644507) and load the destriped I, Q (and U) maps with healpy. Cut out 5-degree gnomonic projection patches centered on the known Tau A (Crab Nebula) position, matching the Nside 512, 40-arcmin Gaussian-smoothed point-source maps shown in Fig. 10 of the paper. Perform aperture photometry on the I and Q patches to obtain integrated flux densities; skip U because Tau A emission in U is too faint to be detected. Retrieve the WMAP 7-year Q-band reference flux for Tau A (Weiland et al. 2011, via LAMBDA) and compute B-Machine/WMAP flux ratios in I and Q, writing the results to a comparison CSV. The decision point is the validation step: the I flux ratio must lie within 20% of unity and the Q flux ratio within 6% of unity, consistent with the paper's reported agreement of '20% in I and 6% in Q'.

## Related tools

- **Python** (Driving language for downloading the Figshare deposit, running aperture photometry, and writing the comparison CSV) — http://github.com/zonca/dst
- **HEALPix** (Map scheme used to load the destriped I, Q, U FITS maps and cut 5-degree gnomonic patches centered on Tau A)
- **B-Machine** (UCSB ground-based 37.5 GHz polarimeter whose destriped maps provide the I/Q data being photometered)

## Examples

```
import healpy as hp; I, Q, U = hp.read_map('bmachine_ch6_destriped.fits', field=(0,1,2)); hp.gnomview(I, rot=tau_a_radec, reso=40*360/5/60)
```

## Evaluation signals

- The I flux ratio (B-Machine / WMAP 7-year Q-band) lies within 20% of unity
- The Q flux ratio lies within 6% of unity, consistent with the reported '20% in I and 6% in Q' agreement
- All flux fields (B-Machine I, B-Machine Q, WMAP I, WMAP Q, and both ratios) are present in tau_a_flux_comparison.csv
- The extracted patches are 5-degree gnomonic projections centered on the known Tau A position at Nside 512, matching the 40-arcmin Gaussian-smoothed point-source maps in Fig. 10
- No U-band flux value is claimed, since U emission from Tau A is expected to be undetectable

## Limitations

- Tau A emission in U is too faint to be detected, so polarization position-angle estimates cannot be derived from this measurement
- Aperture photometry assumes the maps are destriped; residual correlated noise or unmasked systematics (e.g., thermally induced gain variations near sunrise/sunset) would bias the fluxes
- The validation thresholds (20% in I, 6% in Q) are specific to the B-Machine vs WMAP Q-band comparison and may not transfer to other instruments or bands

## Evidence

- [other] Aperture photometry on I and Q patches: [evidence span withheld — anchor status: not_found]
- [other] U not measured: [evidence span withheld — anchor status: not_found]
- [other] Validation thresholds: [evidence span withheld — anchor status: not_found]
