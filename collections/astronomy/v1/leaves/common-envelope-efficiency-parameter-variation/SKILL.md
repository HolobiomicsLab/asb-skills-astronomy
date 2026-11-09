---
name: common-envelope-efficiency-parameter-variation
description: Use when assessing sensitivity of COMPAS-predicted BBH merger rates (total
  and per-event analogs like GW150914, GW151226, LVT151012) to the common-envelope
  parameter alpha_lambda at Z=0.002 and Z=0.001.
when_to_use_negative:
- The goal is a full multidimensional exploration of the model parameter space (e.g.
  joint variation of alpha_lambda and f_LBV) rather than an independent one-parameter
  sensitivity check — the paper explicitly deferred this.
- An accurate cosmological merger rate is required via convolution of metallicity-specific
  redshift-dependent star formation rate with the time-delay distribution — this skill
  yields per-1e9-Msun formation rates, not convolved cosmic rates.
- The question concerns LBV wind mass-loss rate or supernova engine (rapid Fryer model)
  sensitivity rather than common-envelope efficiency.
edam_operation: http://edamontology.org/operation_2426
edam_topics: []
tools:
- name: COMPAS
  role: Rapid binary population synthesis code used to evolve binaries from zero-age
    main sequence to compact remnants under fiducial and varied alpha_lambda settings
  repo: https://github.com/TeamCOMPAS/COMPAS
- name: STROOPWAFEL
  role: COMPAS adaptive importance sampling algorithm used to efficiently sample the
    rare merging-BBH subpopulation at each metallicity
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/ncomms14906
    title: Formation of the first three gravitational-wave observations through isolated
      binary evolution
  source_content_hash: sha256:f2ffc8993d6787227550b9165b1f4ff30d3ab8272b9d35e059a47ba891511c29
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TeamCOMPAS/COMPAS
  tools:
  - COMPAS
  - STROOPWAFEL
  edam_operation: http://edamontology.org/operation_2426
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# common-envelope-efficiency-parameter-variation

## Summary

Quantifies how the common-envelope ejection efficiency parameter alpha_lambda controls predicted binary black hole merger rates and mass scales, by rerunning COMPAS population synthesis with alpha_lambda=0.01 against the fiducial 0.1 at low metallicity.

## When to use

Use when assessing sensitivity of COMPAS-predicted BBH merger rates (total and per-event analogs like GW150914, GW151226, LVT151012) to the common-envelope parameter alpha_lambda at Z=0.002 and Z=0.001.

## When NOT to use

- The goal is a full multidimensional exploration of the model parameter space (e.g. joint variation of alpha_lambda and f_LBV) rather than an independent one-parameter sensitivity check — the paper explicitly deferred this.
- An accurate cosmological merger rate is required via convolution of metallicity-specific redshift-dependent star formation rate with the time-delay distribution — this skill yields per-1e9-Msun formation rates, not convolved cosmic rates.
- The question concerns LBV wind mass-loss rate or supernova engine (rapid Fryer model) sensitivity rather than common-envelope efficiency.

## Inputs

- COMPAS Fiducial model configuration (Kroupa IMF up to 100 Msun, uniform mass ratio, flat-in-log semi-major axis, zero eccentricity, 100% binaries)
- Metallicity grid values Z=0.002 (10% solar) and Z=0.001 (5% solar)
- Common-envelope ejection efficiency values: fiducial alpha_lambda=0.1 and varied alpha_lambda=0.01

## Outputs

- CSV table of total BBH merger rate (mergers in a Hubble time per 1e9 Msun star formation) at each (metallicity, alpha_lambda) combination
- Per-event merger counts for GW150914-, GW151226- and LVT151012-like systems at both metallicities
- Maximum mass of merging BBHs at both metallicities under fiducial and varied alpha_lambda

## How to apply

Install COMPAS from github.com/TeamCOMPAS/COMPAS and reconstruct the Fiducial model: Kroupa IMF with primary mass up to 100 solar masses, uniform mass-ratio distribution, flat-in-the-log semi-major-axis distribution, zero initial eccentricity, all stars in binaries. Run two populations at Z=0.002 (10% solar) and Z=0.001 (5% solar) with STROOPWAFEL adaptive importance sampling to efficiently sample the rare merging-BBH subpopulation, then repeat with the single varied parameter alphalambda=0.01 replacing the fiducial 0.1. For each run compute the number of BBHs merging within a Hubble time per 1e9 solar masses of star formation, per-event counts for systems analogous to GW150914, GW151226 and LVT151012, and the maximum mass of the merging BBHs. Record drop factors relative to the fiducial baseline (total rate ~2x at Z=0.002, ~4x at Z=0.001; per-event ~25x for GW151226-like, ~4x for LVT151012-like, ~50x for GW150914-like) and maximum-BBH-mass shifts (~50 to ~60 Msun at Z=0.002; ~75 to ~65 Msun at Z=0.001) in a CSV table. Vary alpha_lambda independently of other parameters, per the paper's one-parameter-at-a-time uncertainty approach; a full multidimensional exploration of alpha and f_LBV was explicitly deferred.

## Related tools

- **COMPAS** (Rapid binary population synthesis code used to evolve binaries from zero-age main sequence to compact remnants under fiducial and varied alpha_lambda settings) — https://github.com/TeamCOMPAS/COMPAS
- **STROOPWAFEL** (COMPAS adaptive importance sampling algorithm used to efficiently sample the rare merging-BBH subpopulation at each metallicity)

## Evaluation signals

- Tabulated total-rate drop factor at Z=0.002 must reproduce the reported factor of ~2 relative to the Fiducial model
- Total-rate drop factor at Z=0.001 should be ~4
- CSV table must contain rate and maximum-mass entries for all three event analogs (GW150914-, GW151226-, LVT151012-like) at both metallicities
- Maximum-BBH-mass values should shift from ~50 to ~60 Msun at Z=0.002 and from ~75 to ~65 Msun at Z=0.001 when alpha_lambda goes 0.1 -> 0.01
- Per-event drop factors should be approximately 25 (GW151226-like), 4 (LVT151012-like) and 50 (GW150914-like)

## Limitations

- Only individual parameters are varied independently; a full exploration of the multidimensional parameter space (e.g. alpha and f_LBV jointly) was left to future studies with COMPAS.
- The merger-rate comparison to LIGO is approximate: an accurate rate requires convolution of the metallicity-specific redshift-dependent star formation rate with the time-delay distribution, which was not performed.
- The single-star model grid underlying SSE extends only to 50 solar masses and is extrapolated above that limit.
- Stellar rotation, tides, and wind accretion by a companion are not included in the Fiducial model.
- The ability of a single formation channel to explain all observed events remains untested pending future GW observations.

## Evidence

- [discussion] Total rate drop at 10% solar metallicity: "changing the value of αλ from the fiducial 0.1 to 0.01. At 10% solar metallicity we find the total BBHs merger rate drops by a factor of ∼2"
- [discussion] Per-parameter uncertainty approach: "here we follow the common approach273637 of varying individual parameters independently and assessing their impact relative to the Fiducial model."
- [discussion] Initial distributions for the population: [evidence span withheld — anchor status: not_found]
- [methods] STROOPWAFEL citation instruction: "If you use COMPAS's importance sampling algorithm STROOPWAFEL, please cite"
- [discussion] Fiducial rate normalization: "We form ∼2 × 104 BBHs that merge in a Hubble time per 1 × 109 solar masses of star formation at 10% solar metallicity"
- [discussion] Rate normalization caveat: "An accurate calculation of the merger rate requires the convolution of the metallicity-specific redshift-dependent star formation rate with the time delay distribution"
- [readme] COMPAS availability: "COMPAS is a publicly available rapid binary population synthesis code (https://compas.science/) that is designed so that evolution prescriptions and model parameters are easily adjustable."
