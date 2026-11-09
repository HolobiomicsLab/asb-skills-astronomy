---
name: bbh-merger-rate-normalization-per-star-formation
description: Use when a COMPAS/STROOPWAFEL double-compact-object output must be expressed
  as BBHs merging in a Hubble time per 1e9 solar masses of star formation at a given
  metallicity, or when quantifying rate sensitivity to one varied parameter (e.g.
  alphalambda 0.1 vs 0.01).
when_to_use_negative:
- 'An accurate volumetric merger rate is required: this skill yields a metallicity-specific
  rate per formed stellar mass and does not convolve the metallicity-specific redshift-dependent
  star formation rate with the time-delay distribution.'
- A full multidimensional parameter-space exploration (e.g. joint alpha and fLBV)
  is the goal; the paper's approach varies individual parameters independently only.
- The population model must include stellar rotation, tides, or wind accretion by
  the companion — these are not accounted for in the Fiducial model this normalization
  assumes.
edam_operation: http://edamontology.org/operation_2426
tools:
- name: COMPAS
  role: Rapid binary population synthesis code used to simulate the binary populations
    and produce the merging-BBH counts that are normalized per 1e9 solar masses of
    star formation
  repo: http://github.com/TeamCOMPAS/COMPAS
- name: STROOPWAFEL
  role: Adaptive importance-sampling algorithm (shipped with COMPAS) used to efficiently
    sample the rare merging-BBH subpopulation at each metallicity
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/ncomms14906
    title: Formation of the first three gravitational-wave observations through isolated
      binary evolution
  source_content_hash: sha256:ea1e6418e4b3984f42698a5b91a26370494543c9aa8bb3f44907da484ed7347e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COMPAS
  - STROOPWAFEL
  edam_operation: http://edamontology.org/operation_2426
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# bbh-merger-rate-normalization-per-star-formation

## Summary

Normalizes COMPAS binary-black-hole population-synthesis output into merger counts per 1e9 solar masses of star formation at fixed metallicity, and tabulates how total and per-event (GW150914/GW151226/LVT151012-analog) rates and maximum BBH masses shift when a single model parameter such as the common-envelope efficiency alpha_lambda is varied.

## When to use

Use when a COMPAS/STROOPWAFEL double-compact-object output must be expressed as BBHs merging in a Hubble time per 1e9 solar masses of star formation at a given metallicity, or when quantifying rate sensitivity to one varied parameter (e.g. alphalambda 0.1 vs 0.01).

## When NOT to use

- An accurate volumetric merger rate is required: this skill yields a metallicity-specific rate per formed stellar mass and does not convolve the metallicity-specific redshift-dependent star formation rate with the time-delay distribution.
- A full multidimensional parameter-space exploration (e.g. joint alpha and fLBV) is the goal; the paper's approach varies individual parameters independently only.
- The population model must include stellar rotation, tides, or wind accretion by the companion — these are not accounted for in the Fiducial model this normalization assumes.

## Inputs

- COMPAS population-synthesis run outputs (double compact object catalogs with merger times) at Z=0.002 and Z=0.001
- STROOPWAFEL importance-sampling configuration for the rare merging-BBH subpopulation
- Fiducial model parameter settings (Kroupa IMF up to 100 Msun, uniform mass ratio, flat-in-the-log semi-major axis, zero eccentricity, all stars in binaries)
- Varied-parameter setting (alphalambda = 0.01 replacing fiducial 0.1)

## Outputs

- CSV table of total BBH merger rate per 1e9 solar masses of star formation per metallicity and per model variant
- Per-event analog merger counts (GW150914-like, GW151226-like, LVT151012-like) with drop factors relative to Fiducial
- Maximum mass of merging BBHs per metallicity and model (e.g. ~50 to ~60 Msun at Z=0.002; ~75 to ~65 Msun at Z=0.001)

## How to apply

Reconstruct the Fiducial COMPAS configuration — Kroupa IMF with primary mass up to 100 solar masses, uniform mass-ratio distribution, flat-in-the-log semi-major-axis distribution, zero initial eccentricity, all stars in binaries — then run the population at Z=0.002 (10% solar) and Z=0.001 (5% solar) using STROOPWAFEL adaptive importance sampling to efficiently sample the rare merging-BBH subpopulation. From each run, count only systems whose merger time falls within a Hubble time, normalize that count per 1e9 solar masses of star formation, and additionally tally per-event analog counts (systems matching GW150914, GW151226, LVT151012) and the maximum mass of the merging BBHs. To measure parameter sensitivity, re-run with the single varied parameter — here alphalambda set to 0.01 instead of the fiducial 0.1 — and compute drop factors against the fiducial baseline at each metallicity. Record everything in a CSV table with total rate, per-event factors, and maximum-BBH-mass entries for all three analogs at both metallicities. The fiducial baseline against which the drop is judged is ~2e4 merging BBHs per 1e9 solar masses at 10% solar metallicity and ~3e4 at 5% solar metallicity.

## Related tools

- **COMPAS** (Rapid binary population synthesis code used to simulate the binary populations and produce the merging-BBH counts that are normalized per 1e9 solar masses of star formation) — http://github.com/TeamCOMPAS/COMPAS
- **STROOPWAFEL** (Adaptive importance-sampling algorithm (shipped with COMPAS) used to efficiently sample the rare merging-BBH subpopulation at each metallicity)

## Evaluation signals

- Total-rate drop factor at 10% solar metallicity reproduces the reported factor of ~2 when alphalambda is changed from 0.1 to 0.01
- Total-rate drop factor at 5% solar metallicity reproduces the reported factor of ~4
- CSV table contains rate and maximum-mass entries for all three event analogs (GW150914-like, GW151226-like, LVT151012-like) at both metallicities
- Per-event drop factors fall in the reported ranges: ~25 for GW151226-like, ~4 for LVT151012-like, ~50 for GW150914-like systems
- Fiducial baseline rate matches ~2e4 merging BBHs per 1e9 solar masses at Z=0.002 and ~3e4 at Z=0.001, and rescaled z=0 formation rate lands near ~300 Gpc^-3 per year vs the LIGO empirical estimate of 9-240 Gpc^-3 per year

## Limitations

- The rate comparison to LIGO is approximate; an accurate rate requires convolving the metallicity-specific redshift-dependent star formation rate with the time-delay distribution, which was not done.
- The single-star model grid underlying SSE extends only to 50 solar masses and is extrapolated above that limit.
- Stellar rotation and tides are not included in the Fiducial model, and wind accretion by a companion is not accounted for.
- Only individual parameters were varied independently relative to the Fiducial model; a full exploration of the multidimensional parameter space (e.g. alpha, fLBV) is deferred to future studies.

## Evidence

- [discussion] Fiducial baseline rate per formed stellar mass: "We form ∼2 × 104 BBHs that merge in a Hubble time per 1 × 109 solar masses of star formation at 10% solar metallicity"
- [discussion] alpha_lambda sensitivity result being tabulated: "changing the value of αλ from the fiducial 0.1 to 0.01. At 10% solar metallicity we find the total BBHs merger rate drops by a factor of ∼2"
- [discussion] Fiducial initial distributions underlying the normalized population: "the primary mass m1 drawn from the Kroupa IMF31 up to m1≤100M⊙"
- [methods] Tool producing the rate counts: "Simulations in this paper made use of the COMPAS rapid binary population synthesis code (version X.X.X), which is freely available"
- [readme] README role of COMPAS in drawing and evolving binaries: "COMPAS draws properties for a binary star system from a set of initial distributions, and evolves it from zero-age main sequence to the end of its life as two compact"
