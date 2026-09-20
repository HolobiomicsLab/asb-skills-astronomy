---
name: black-hole-progenitor-mass-tabulation
description: Use when you have COMPAS double compact object output (e.g. from the
  COMPAS Zenodo community) for a fiducial model and need to identify which progenitor
  binaries match observed GW event parameters and tabulate their initial total masses
  per metallicity.
when_to_use_negative:
- You need merger-rate densities (e.g. Gpc^-3 per year) rather than progenitor mass
  ranges — an accurate rate requires convolving metallicity-specific redshift-dependent
  star formation rate with the time delay distribution, which this tabulation does
  not perform
- Your population was simulated with non-fiducial parameter variations (e.g. optimistic
  common envelope, alpha_lambda=0.01, or LBV mass-loss rate 1e-3 Msun/yr) — the reported
  mass ranges are specific to the Fiducial model
- The target systems involve neutron stars, X-ray binaries, or luminous red novae
  rather than merging binary black holes within a Hubble time
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: COMPAS
  role: Rapid binary population synthesis code whose fiducial-model double compact
    object outputs supply the progenitor binaries; used to filter merging BBHs and
    compute ZAMS initial total masses
  repo: https://github.com/TeamCOMPAS/COMPAS
- name: pandas
  role: Used to tabulate minimum and typical initial total masses per event per metallicity
    and write progenitor_initial_masses.csv
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/ncomms14906
    title: Formation of the first three gravitational-wave observations through isolated
      binary evolution
  source_content_hash: sha256:112baf62efea9b3ee88d50fcf3dcda698642960a89a3c61c2edc2387e8a8b01a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TeamCOMPAS/COMPAS
  tools:
  - COMPAS
  - pandas
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# black-hole-progenitor-mass-tabulation

## Summary

Tabulates the zero-age main-sequence initial total masses (m1+m2) of binary black hole progenitors that reproduce specific gravitational-wave events (GW150914, GW151226, LVT151012) in COMPAS populations at Z=0.002 and Z=0.001, quantifying how massive the stellar ancestors must be at each metallicity.

## When to use

Use when you have COMPAS double compact object output (e.g. from the COMPAS Zenodo community) for a fiducial model and need to identify which progenitor binaries match observed GW event parameters and tabulate their initial total masses per metallicity.

## When NOT to use

- You need merger-rate densities (e.g. Gpc^-3 per year) rather than progenitor mass ranges — an accurate rate requires convolving metallicity-specific redshift-dependent star formation rate with the time delay distribution, which this tabulation does not perform
- Your population was simulated with non-fiducial parameter variations (e.g. optimistic common envelope, alpha_lambda=0.01, or LBV mass-loss rate 1e-3 Msun/yr) — the reported mass ranges are specific to the Fiducial model
- The target systems involve neutron stars, X-ray binaries, or luminous red novae rather than merging binary black holes within a Hubble time

## Inputs

- Fiducial-model COMPAS output data for double compact object populations at Z=0.002 and Z=0.001 from the COMPAS Zenodo community deposit
- 90% credible intervals on total mass, chirp mass, mass ratio and component masses for GW150914, GW151226 and LVT151012

## Outputs

- progenitor_initial_masses.csv: a pandas DataFrame of minimum and typical initial ZAMS total masses (m1+m2, in solar masses) per event (GW150914, GW151226, LVT151012) per metallicity (Z=0.002, Z=0.001)

## How to apply

Load the fiducial-model COMPAS output for double compact objects at Z=0.002 and Z=0.001 from the COMPAS Zenodo community deposit. Filter to binary black holes that merge within a Hubble time using the COMPAS merger-time prescription (Peters equations) to build the event-selection sample. For each of GW150914, GW151226 and LVT151012, select progenitor systems whose chirp mass and component masses fall within the 90% credible intervals on total/chirp mass and mass ratio of the observed event. Compute the initial total mass as the sum of the two zero-age main-sequence masses (m1+m2) for each selected progenitor at each metallicity, tabulate the minimum and typical values per event per metallicity in a pandas DataFrame, and write to progenitor_initial_masses.csv. The rationale, from Stevenson et al. (2017), is that a single classical isolated binary evolution channel with a common envelope phase can explain all three events, and the progenitor total-mass floor is a metallicity-dependent diagnostic of that channel.

## Related tools

- **COMPAS** (Rapid binary population synthesis code whose fiducial-model double compact object outputs supply the progenitor binaries; used to filter merging BBHs and compute ZAMS initial total masses) — https://github.com/TeamCOMPAS/COMPAS
- **pandas** (Used to tabulate minimum and typical initial total masses per event per metallicity and write progenitor_initial_masses.csv)

## Examples

```
df = pd.read_csv('COMPAS_Z0.001_DCO.h5'); bbh = df[(df.Merger_Time_in_Hubble_Time) & (df.Stellar_Type1==14) & (df.Stellar_Type2==14)]; sel = bbh[(bbh.Mchirp>=mchirp_lo)&(bbh.Mchirp<=mchirp_hi)]; sel['M_ZAMS_total'] = sel.Mass_ZAMS_1+sel.Mass_ZAMS_2; sel[['M_ZAMS_total']].describe().to_csv('progenitor_initial_masses.csv')
```

## Evaluation signals

- The minimum initial total masses at Z=0.001 must be at least approximately 160, 60 and 90 solar masses for GW150914, GW151226 and LVT151012 respectively
- The Z=0.002 initial total masses must fall within the reported 65-100 and 95-125 solar mass ranges
- Every selected progenitor's chirp mass and component masses must lie inside the 90% credible intervals of the corresponding observed event
- All selected systems must be binary black holes that merge within a Hubble time under the COMPAS Peters-equation merger-time prescription
- progenitor_initial_masses.csv must contain one row per event per metallicity with both minimum and typical mass columns

## Limitations

- The single-star grid underlying SSE extends only to 50 solar masses and is extrapolated above that limit, affecting the most massive progenitors
- Stellar rotation and tides are not included in the Fiducial model, and wind accretion by a companion is not accounted for
- The Fiducial-model assumptions (e.g. common envelope treatment) are one point in an uncertain parameter space; only individual parameters were varied, not a full multidimensional exploration
- Only the fiducial COMPAS model at Z=0.002 and Z=0.001 is characterized; results may shift under the optimistic CE or other parameter variations

## Evidence

- [abstract] All three events could have formed in low-metallicity environments (Z=0.001) from progenitor binaries with typical total masses of at least ~160, ~60 and ~90 solar masses: "We show all three events could have formed in low-metallicity environments ( Z =0.001) from progenitor binaries with typical total masses ≳160 M ⊙ , ≳60 M ⊙ and ≳90 M ⊙ , for GW150914, GW151226 and"
- [discussion] All three events explained by classical isolated binary evolution via a common envelope phase under the Fiducial model: "All three observed systems can be explained through this channel under our Fiducial model assumptions."
- [intro] COMPAS is publicly available and designed so evolution prescriptions are easily adjustable: "COMPAS is a publicly available rapid binary population synthesis code (https://compas.science/) that is designed so that evolution prescriptions and model parameters are easily adjustable."
- [methods] COMPAS data and settings are shared via the Zenodo community: "make your input settings and output data publicly available by uploading it to the [COMPAS Zenodo community](https://zenodo.org/communities/compas/)"
- [discussion] SSE single-star grid extends only to 50 solar masses and is extrapolated above that: "the original grid of single star models extends only to 50 solar masses. We extrapolate above this limit, as described in Hurley et al.39"
