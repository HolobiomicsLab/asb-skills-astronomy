---
name: low-metallicity-progenitor-analysis
description: Use when a task asks to reproduce or perturb the paper's COMPAS BBH progenitor
  analysis at Z=0.001 (5% solar metallicity) — e.g. changing the LBV mass-loss rate
  — to compare total merger rates and LVT151012-like (progenitor total mass ≳90 Msun)
  yields against the Fiducial model.
when_to_use_negative:
- The analysis needs an accurate cosmological merger rate — the paper only rescales
  to an approximate ~300 Gpc^-3/yr and notes that an accurate rate requires convolving
  metallicity-specific, redshift-dependent star-formation rate with the delay-time
  distribution, which it does not do.
- The goal is a full multidimensional parameter-space exploration (e.g. varying common-envelope
  physics and the LBV rate jointly) — the paper explicitly varies individual parameters
  independently and defers joint exploration to future studies.
- The study requires stellar rotation, tides, or wind accretion by companions — these
  physical processes are not included in the Fiducial model, so COMPAS results under
  this skill would not capture them.
edam_operation: http://edamontology.org/operation_2945
tools:
- name: COMPAS
  role: Rapid binary population synthesis code used to evolve the low-metallicity
    progenitor binaries from zero-age main sequence to compact remnants and to produce
    merger statistics when the LBV mass-loss parameter is varied
  repo: http://github.com/TeamCOMPAS/COMPAS
- name: STROOPWAFEL
  role: COMPAS's importance sampling algorithm used for efficient sampling of the
    rare BBH mergers and LVT151012-like channels
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/ncomms14906
    title: Formation of the first three gravitational-wave observations through isolated
      binary evolution
  source_content_hash: sha256:064b9a0ee4327797db0d297159198bdc7289e2479eb03006643d5f133c0a14c2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COMPAS
  - STROOPWAFEL
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# low-metallicity-progenitor-analysis

## Summary

Reproduces the paper's low-metallicity (Z=0.001, 5% solar) BBH progenitor analysis in COMPAS: run the Fiducial population synthesis and single-parameter variants (e.g. LBV wind 1.5e-4 → 1e-3 Msun/yr) with STROOPWAFEL sampling, then compare total Hubble-time BBH merger yields and LVT151012-like (progenitor total mass ≳90 Msun) counts against Fiducial.

## When to use

Use when a task asks to reproduce or perturb the paper's COMPAS BBH progenitor analysis at Z=0.001 (5% solar metallicity) — e.g. changing the LBV mass-loss rate — to compare total merger rates and LVT151012-like (progenitor total mass ≳90 Msun) yields against the Fiducial model.

## When NOT to use

- The analysis needs an accurate cosmological merger rate — the paper only rescales to an approximate ~300 Gpc^-3/yr and notes that an accurate rate requires convolving metallicity-specific, redshift-dependent star-formation rate with the delay-time distribution, which it does not do.
- The goal is a full multidimensional parameter-space exploration (e.g. varying common-envelope physics and the LBV rate jointly) — the paper explicitly varies individual parameters independently and defers joint exploration to future studies.
- The study requires stellar rotation, tides, or wind accretion by companions — these physical processes are not included in the Fiducial model, so COMPAS results under this skill would not capture them.

## Inputs

- Fiducial COMPAS input settings from the COMPAS Zenodo community (Z=0.001, LBV mass-loss rate 1.5e-4 Msun/yr)
- Fiducial merger statistics from the COMPAS results site (www.sr.bham.ac.uk/compas/)
- Initial distribution specification: Kroupa IMF (m1 ≤ 100 Msun), uniform mass ratio, flat-in-the-log semimajor axis, zero eccentricity, all stars in binaries
- Modified COMPAS configuration with the LBV mass-loss rate set to 1e-3 Msun/yr (only this parameter changed)

## Outputs

- Total BBH merger rate for the modified run, expressed as mergers within a Hubble time per 1e9 solar masses of star formation
- Count of LVT151012-like BBH mergers (progenitor total mass ≳90 Msun) in the modified run vs the Fiducial count
- Comparison table written to lbv_variation_results.csv

## How to apply

Load the Fiducial COMPAS input settings and reference merger statistics (from the COMPAS Zenodo community and the results site) and confirm the run configuration: Z=0.001, LBV mass-loss rate 1.5e-4 Msun/yr. Vary only the target parameter (e.g. set the LBV wind to 1e-3 Msun/yr) relative to Fiducial, drawing primary masses from a Kroupa IMF with m1 ≤ 100 Msun, a flat mass-ratio distribution, flat-in-the-log semimajor axes, zero initial eccentricity, and assuming all stars are in binaries, with STROOPWAFEL importance sampling. Post-process the output with COMPAS scripts to compute the total BBH merger rate as mergers within a Hubble time per 1e9 solar masses of star formation (Fiducial forms ~3e4 such BBHs at 5% solar metallicity). Separately count mergers similar to LVT151012 (progenitor total mass ≳90 Msun) in both runs. Write the comparison to lbv_variation_results.csv. Accept the reproduction only if the total BBH merger rate is not significantly changed while the LVT151012-like count drops by a factor of ~10, as reported in the paper's discussion.

## Related tools

- **COMPAS** (Rapid binary population synthesis code used to evolve the low-metallicity progenitor binaries from zero-age main sequence to compact remnants and to produce merger statistics when the LBV mass-loss parameter is varied) — http://github.com/TeamCOMPAS/COMPAS
- **STROOPWAFEL** (COMPAS's importance sampling algorithm used for efficient sampling of the rare BBH mergers and LVT151012-like channels)

## Examples

```
python runCOMPAS.py --metallicity 0.001 --LBV_MassLoss_Rate 1e-3  # vary only this vs Fiducial (1.5e-4); post-process counts into lbv_variation_results.csv
```

## Evaluation signals

- Total BBH merger rate in the modified run is not significantly changed relative to Fiducial (Fiducial forms ~2e4 BBHs merging in a Hubble time per 1e9 Msun of star formation at 10% solar metallicity, rising to ~3e4 at 5%).
- Number of LVT151012-like mergers (progenitor total mass ~90 Msun regime) is reduced by a factor of ~10 relative to the Fiducial count at Z=0.001.
- Diff between Fiducial and modified run configurations confirms only the LBV mass-loss parameter changed.
- Initial distributions in the run match the paper: Kroupa IMF up to 100 Msun, flat mass-ratio, flat-in-the-log semimajor axis, eccentricity zero, all stars in binaries.
- lbv_variation_results.csv exists and contains the Fiducial vs modified comparison of total rate and LVT151012-like counts.

## Limitations

- Wind accretion by the companion is not accounted for in the model.
- Stellar rotation and tides are not included in the Fiducial model.
- The single-star model grid underlying SSE extends only to 50 solar masses and is extrapolated above that limit — relevant here because LVT151012-like progenitors have total masses ≳90 Msun.
- The merger rate comparison to LIGO's empirical estimate (9–240 Gpc^-3/yr) is approximate; no convolution with cosmic star-formation history is performed.

## Evidence

- [discussion] LBV variation validation: "does not significantly change the total BBHs merger rate; nevertheless, the number of BBHs mergers similar to LVT151012 was reduced by a factor of ∼10"
- [discussion] Initial distributions: "the primary mass m1 drawn from the Kroupa IMF31 up to m1≤100M⊙"
- [abstract] Low-metallicity progenitors: "all three events could have formed in low-metallicity environments ( Z =0.001) from progenitor binaries with typical total masses"
- [readme] COMPAS purpose: "COMPAS draws properties for a binary star system from a set of initial distributions, and evolves it from zero-age main sequence to the end of its life as two compact remnants"
- [discussion] Fiducial model physics exclusions: "Stellar rotation and tides are not included in the Fiducial model."
