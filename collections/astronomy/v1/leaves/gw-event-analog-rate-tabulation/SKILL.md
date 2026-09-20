---
name: gw-event-analog-rate-tabulation
description: Use when a single binary-evolution parameter (e.g. common-envelope efficiency
  alphalambda) is varied against a Fiducial COMPAS model and you need per-metallicity
  drop factors for total and event-analog BBH merger rates plus max-BBH-mass shifts.
when_to_use_negative:
- The goal is an absolute cosmic merger rate comparable to LIGO's 9-240 Gpc^-3 per
  year estimate — that requires convolving metallicity-specific redshift-dependent
  star formation rate with the time-delay distribution, which this tabulation does
  not do.
- Multiple model parameters are varied simultaneously — the article explicitly varies
  individual parameters independently relative to the Fiducial model and defers full
  parameter-space exploration to future work.
- Stellar rotation, tides, wind accretion, or full multidimensional parameter-space
  exploration are required — these are outside the Fiducial model's scope.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_0102
tools:
- name: COMPAS
  role: Rapid binary population synthesis code used to evolve progenitor binaries
    and produce merging-BBH populations whose rates are tabulated
  repo: http://github.com/TeamCOMPAS/COMPAS
- name: STROOPWAFEL
  role: COMPAS's adaptive importance-sampling algorithm used to efficiently sample
    the rare merging-BBH subpopulation at each metallicity
  repo: http://github.com/TeamCOMPAS/COMPAS
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/ncomms14906
    title: Formation of the first three gravitational-wave observations through isolated
      binary evolution
  source_content_hash: sha256:e722df13ef96f878bdaf1cbcf3a1941fe88cac661c25c82aa1754c67a7d62e45
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COMPAS
  - STROOPWAFEL
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# gw-event-analog-rate-tabulation

## Summary

Tabulates COMPAS-predicted binary black hole merger rates (total per 1e9 solar masses of star formation and per-event counts for GW150914-, GW151226- and LVT151012-like systems) and maximum merging-BBH masses under single-parameter variations (e.g. alphalambda 0.1 -> 0.01) at Z=0.002 and Z=0.001, quantifying sensitivity relative to the Fiducial model.

## When to use

Use when a single binary-evolution parameter (e.g. common-envelope efficiency alphalambda) is varied against a Fiducial COMPAS model and you need per-metallicity drop factors for total and event-analog BBH merger rates plus max-BBH-mass shifts.

## When NOT to use

- The goal is an absolute cosmic merger rate comparable to LIGO's 9-240 Gpc^-3 per year estimate — that requires convolving metallicity-specific redshift-dependent star formation rate with the time-delay distribution, which this tabulation does not do.
- Multiple model parameters are varied simultaneously — the article explicitly varies individual parameters independently relative to the Fiducial model and defers full parameter-space exploration to future work.
- Stellar rotation, tides, wind accretion, or full multidimensional parameter-space exploration are required — these are outside the Fiducial model's scope.

## Inputs

- COMPAS Fiducial-model run outputs at Z=0.002 (10% solar metallicity) and Z=0.001 (5% solar metallicity) using STROOPWAFEL sampling
- COMPAS varied-parameter run outputs (alphalambda=0.01) at the same two metallicities
- Event-analog definitions (component masses / chirp mass) for GW150914, GW151226 and LVT151012

## Outputs

- CSV table with total BBH merger rate per 1e9 Msun of star formation, per-event-analog merger counts (GW150914-, GW151226-, LVT151012-like), and maximum merging-BBH mass, at both metallicities, with drop factors vs the Fiducial model
- Reported sensitivity factors (e.g. total rate ~2x drop at Z=0.002, ~4x at Z=0.001)

## How to apply

Run COMPAS with the Fiducial initial distributions (Kroupa IMF to 100 Msun, uniform mass ratio, flat-in-the-log semi-major axis, zero eccentricity, all stars in binaries) at Z=0.002 and Z=0.001 using STROOPWAFEL adaptive importance sampling, then repeat with the single varied parameter (alphalambda=0.01 instead of 0.1). From each run compute the number of BBHs merging in a Hubble time per 1e9 solar masses of star formation, the counts of systems analogous to GW150914, GW151226 and LVT151012, and the maximum mass of the merging BBHs. Divide each varied-model quantity by the Fiducial baseline to obtain drop factors and record them, together with absolute maximum-BBH masses, in a CSV table with one row per metallicity and per-event-analog entry. Expect roughly a 2x total-rate drop at 10% solar metallicity and ~4x at 5% solar metallicity, with per-event factors of ~25 (GW151226-like), ~4 (LVT151012-like) and ~50 (GW150914-like), and max-BBH-mass shifts of ~50 to ~60 Msun at Z=0.002 and ~75 to ~65 Msun at Z=0.001.

## Related tools

- **COMPAS** (Rapid binary population synthesis code used to evolve progenitor binaries and produce merging-BBH populations whose rates are tabulated) — http://github.com/TeamCOMPAS/COMPAS
- **STROOPWAFEL** (COMPAS's adaptive importance-sampling algorithm used to efficiently sample the rare merging-BBH subpopulation at each metallicity) — http://github.com/TeamCOMPAS/COMPAS

## Examples

```
./COMPAS --initial-mass-max 100 --metallicity 0.002 --common-envelope-alpha 0.01 --modes SA --number-of-systems 1000000
```

## Evaluation signals

- Total BBH merger-rate drop factor at 10% solar metallicity must reproduce ~2 relative to the Fiducial model when alphalambda is changed from 0.1 to 0.01.
- Table contains rate and maximum-mass entries for all three event analogs (GW150914-, GW151226-, LVT151012-like) at both Z=0.002 and Z=0.001.
- Total rates are normalized per 1e9 solar masses of star formation and are consistent in order of magnitude with the Fiducial values (~2e4 at Z=0.002 rising to ~3e4 at Z=0.001).
- Maximum merging-BBH masses fall in the reported ranges (~50-60 Msun at Z=0.002; ~65-75 Msun at Z=0.001) with the expected direction of shift.
- Only alphalambda differs between the compared runs; all other initial distributions and physics settings match the Fiducial configuration.

## Limitations

- An accurate merger-rate comparison to LIGO requires convolving metallicity-specific redshift-dependent star formation rate with the time-delay distribution; this tabulation is approximate and does not perform that convolution.
- The SSE single-star model grid extends only to 50 solar masses and is extrapolated above that limit.
- Stellar rotation and tides are not included, wind accretion by a companion is not accounted for, and only individual parameters are varied independently rather than exploring the full parameter space.

## Evidence

- [discussion] Rate sensitivity to alphalambda: "changing the value of αλ from the fiducial 0.1 to 0.01. At 10% solar metallicity we find the total BBHs merger rate drops by a factor of ∼2"
- [discussion] Fiducial formation rate normalization: "We form ∼2 × 104 BBHs that merge in a Hubble time per 1 × 109 solar masses of star formation at 10% solar metallicity"
- [discussion] Parameter-varying methodology: "here we follow the common approach273637 of varying individual parameters independently and assessing their impact relative to the Fiducial model."
- [discussion] Initial distributions used for the population: [evidence span withheld — anchor status: not_found]
- [readme] Code and sampler availability: "If you use COMPAS's importance sampling algorithm STROOPWAFEL, please cite"
