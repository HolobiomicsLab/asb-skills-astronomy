---
name: kroupa-imf-binary-sampling
description: Use when setting up a binary population synthesis run (e.g. COMPAS) that
  needs initial conditions for massive-star binaries to predict double compact object
  or BBH merger yields.
when_to_use_negative:
- The study target requires stellar rotation, tides, or wind accretion — the Fiducial
  model these distributions feed into explicitly excludes them ('Stellar rotation
  and tides are not included in the Fiducial model')
- High-metallicity or chemically homogeneous evolution channels are of interest —
  this sampling underpins the classical isolated binary evolution via common envelope
  channel at low metallicity (Z=0.001)
- A full multidimensional parameter-space exploration (e.g. over alpha, fLBV) is required
  — the paper only varied individual parameters independently and deferred joint exploration
  to future studies
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_3452
tools:
- name: COMPAS
  role: Rapid binary population synthesis code that draws binaries from these initial
    distributions and evolves them from zero-age main sequence to compact remnants
  repo: http://github.com/TeamCOMPAS/COMPAS
- name: STROOPWAFEL
  role: Importance-sampling algorithm within COMPAS that can be used with this initial-distribution
    setup to improve sampling of rare merging BBHs
  repo: https://compas.science/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/ncomms14906
    title: Formation of the first three gravitational-wave observations through isolated
      binary evolution
  source_content_hash: sha256:d52867cd935a406fc81183ad36510f97ed13d33ae0e7ae3ca7e7c46eaf816197
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COMPAS
  - STROOPWAFEL
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# Kroupa IMF binary population sampling

## Summary

Draws the initial binary population for stellar population synthesis using a Kroupa IMF for primary masses (up to 100 Msun), a uniform mass-ratio distribution, a flat-in-the-log semi-major-axis distribution, and zero initial eccentricity, assuming all stars are in binaries. This distribution choice defines the progenitor pool from which COMPAS evolves double compact objects and BBH merger rates.

## When to use

Use when setting up a binary population synthesis run (e.g. COMPAS) that needs initial conditions for massive-star binaries to predict double compact object or BBH merger yields.

## When NOT to use

- The study target requires stellar rotation, tides, or wind accretion — the Fiducial model these distributions feed into explicitly excludes them ('Stellar rotation and tides are not included in the Fiducial model')
- High-metallicity or chemically homogeneous evolution channels are of interest — this sampling underpins the classical isolated binary evolution via common envelope channel at low metallicity (Z=0.001)
- A full multidimensional parameter-space exploration (e.g. over alpha, fLBV) is required — the paper only varied individual parameters independently and deferred joint exploration to future studies

## Inputs

- Population size (number of binaries to draw) for a COMPAS run
- Metallicity setting for evolution (e.g. 10% and 5% solar, Z=0.002 and Z=0.001)
- Fiducial physics configuration: delayed Fryer SN model (alpha=1), alpha_lambda=0.1 common envelope, fLBV=1.5, Vink winds, rotation/tides/wind accretion off

## Outputs

- Sampled initial binary parameters (m1, m2, semi-major axis, eccentricity) per system
- Evolved double compact object population with gravitational-wave merger times
- Merging-BBH counts per 1e9 solar masses of star formation per metallicity (e.g. written to bbh_merger_rates.csv)

## How to apply

Draw the primary mass m1 from the Kroupa IMF with an upper limit of m1 <= 100 Msun, and assume all stars are in binaries so no single-star fraction is sampled. Assign secondary masses from a uniform (flat) mass-ratio distribution q = m2/m1, and orbital separations from a flat-in-the-log semi-major-axis distribution. Set initial eccentricity to zero (circular binaries). In the Stevenson et al. 2017 workflow these sampled binaries are then evolved with COMPAS at metallicities of 10% and 5% solar (Z=0.002, Z=0.001) through mass transfer and common envelope, and only those forming double compact objects that merge within a Hubble time (via gravitational-wave merger-time calculation) are retained. Rationale: these choices reproduce the fiducial initial distributions used to show that GW150914, GW151226 and LVT151012 can arise from progenitors with total masses of at least ~160, ~60 and ~90 solar masses respectively at Z=0.001.

## Related tools

- **COMPAS** (Rapid binary population synthesis code that draws binaries from these initial distributions and evolves them from zero-age main sequence to compact remnants) — http://github.com/TeamCOMPAS/COMPAS
- **STROOPWAFEL** (Importance-sampling algorithm within COMPAS that can be used with this initial-distribution setup to improve sampling of rare merging BBHs) — https://compas.science/

## Examples

```
./COMPAS --random-seed 0 --IMF Kroupa --maximum-notification-mass 100 --mass-ratio-distribution flat --semi-major-axis-distribution flat-in-log --eccentricity-distribution zero --metallicity 0.001
```

## Evaluation signals

- Tabulated merging-BBH yields should reproduce the fiducial values of ~2e4 per 1e9 Msun of star formation at 10% solar metallicity and ~3e4 at 5% solar metallicity
- Rescaled volumetric rate should be consistent with ~300 Gpc^-3 per year at z=0, within the LIGO empirical estimate of 9-240 Gpc^-3 per year
- All three events (GW150914, GW151226, LVT151012) should be reproducible from progenitors with total masses of at least ~160, ~60 and ~90 solar masses at Z=0.001
- Sampled primary-mass distribution must follow the Kroupa IMF with no primaries above 100 Msun; mass-ratio and log-separation histograms must be flat; eccentricities all zero

## Limitations

- The single-star model grid underlying SSE extends only to 50 solar masses and is extrapolated above that limit, affecting the most massive drawn primaries
- The merger-rate comparison to LIGO is approximate; an accurate rate requires convolving metallicity-specific redshift-dependent star formation rate with the time delay distribution, which was not done
- Only individual parameters were varied relative to the Fiducial model; a full exploration of the multidimensional parameter space (e.g. alpha, fLBV) was deferred to future studies

## Evidence

- [discussion] Kroupa IMF sampling of primary mass: "the primary mass m1 drawn from the Kroupa IMF31 up to m1≤100M⊙"
- [discussion] Binary fraction and secondary/orbital distributions: "using the Kroupa31 initial mass function (IMF), a uniform mass ratio distribution and assuming that all stars are in binaries"
- [abstract] Resulting low-metallicity progenitor masses: "We show all three events could have formed in low-metallicity environments ( Z =0.001) from progenitor binaries with typical total masses ≳160 M ⊙"
- [intro] COMPAS as the evolution engine for the sampled population: "COMPAS is a publicly available rapid binary population synthesis code (https://compas.science/) that is designed so that evolution prescriptions and model parameters are easily adjustable."
- [discussion] Fiducial yield benchmark per metallicity: "We form ∼2 × 104 BBHs that merge in a Hubble time per 1 × 109 solar masses of star formation at 10% solar metallicity"
