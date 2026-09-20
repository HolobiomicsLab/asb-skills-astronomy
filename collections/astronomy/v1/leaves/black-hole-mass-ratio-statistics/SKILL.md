---
name: black-hole-mass-ratio-statistics
description: Use when you have COMPAS population-synthesis DCO output at multiple
  metallicities plus observed BBH component masses, and you need median mass-ratio
  statistics or a formation-feasibility verdict per event.
when_to_use_negative:
- Input data are not COMPAS double-compact-object outputs and/or observed GW component
  masses — the feasibility boundary is defined by the solar-metallicity fiducial population
  and cannot be inferred from other data types
- 'The question is an accurate merger-rate estimate: the paper notes an accurate rate
  requires convolution of metallicity-specific redshift-dependent star formation rate
  with the time-delay distribution, which this skill does not perform'
- The formation channel of interest is a dynamical or stellar-merger channel rather
  than classical isolated binary evolution via mass transfer including a common envelope
  phase
tools:
- name: COMPAS
  role: Rapid binary population synthesis code used to generate the DCO populations
    whose mass ratios and solar-metallicity boundary are analyzed
  repo: http://github.com/TeamCOMPAS/COMPAS_
- name: STROOPWAFEL
  role: COMPAS importance-sampling algorithm used to obtain the population statistics
    efficiently
- name: pandas/numpy
  role: 'Post-processing of COMPAS DCO output tables: filtering to Hubble-time mergers
    and computing median mass ratios'
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/ncomms14906
    title: Formation of the first three gravitational-wave observations through isolated
      binary evolution
  source_content_hash: sha256:e1203241210fd57f7ec888d928c90a4fad61f4628140d4891a1a8379fa21c8fc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COMPAS
  - STROOPWAFEL
  - pandas/numpy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# black-hole-mass-ratio-statistics

## Summary

Computes mass-ratio statistics of merging BBH populations from COMPAS double-compact-object output and tests whether observed gravitational-wave systems (GW150914, GW151226, LVT151012) can form through classical isolated binary evolution at a given metallicity, using solar metallicity as the formation boundary.

## When to use

Use when you have COMPAS population-synthesis DCO output at multiple metallicities plus observed BBH component masses, and you need median mass-ratio statistics or a formation-feasibility verdict per event.

## When NOT to use

- Input data are not COMPAS double-compact-object outputs and/or observed GW component masses — the feasibility boundary is defined by the solar-metallicity fiducial population and cannot be inferred from other data types
- The question is an accurate merger-rate estimate: the paper notes an accurate rate requires convolution of metallicity-specific redshift-dependent star formation rate with the time-delay distribution, which this skill does not perform
- The formation channel of interest is a dynamical or stellar-merger channel rather than classical isolated binary evolution via mass transfer including a common envelope phase

## Inputs

- COMPAS fiducial-model double compact object output files for solar metallicity, Z=0.002 and Z=0.001 (from the COMPAS Zenodo community)
- Observed BBH component masses for GW150914, GW151226 and LVT151012

## Outputs

- Median mass ratio of the merging BBH population at each metallicity (expected ~0.5 at 10% solar metallicity)
- Formation-feasibility verdict per gravitational-wave event: infeasible (GW150914, LVT151012), boundary (GW151226), or feasible at solar metallicity

## How to apply

Load the fiducial-model COMPAS double-compact-object outputs for the three metallicities (solar, Z=0.002, Z=0.001) from the COMPAS Zenodo community. Filter to binary black holes that merge within a Hubble time, then compute the median mass ratio (secondary/primary) of the merging population per metallicity with pandas/numpy; at 10% solar metallicity the median q should be ~0.5, consistent with COMPAS's flat (uniform) mass-ratio initial distribution drawn alongside a Kroupa IMF to 100 Msun. Next, overlay the component masses of GW150914, GW151226 and LVT151012 on the solar-metallicity fiducial population to test formation via mass transfer including a common envelope phase: record 'infeasible' for events beyond the maximum solar-metallicity masses (GW150914, LVT151012), 'boundary' for events at the high-mass edge (GW151226), and 'feasible' for events inside the population. The rationale is that metallicity strongly limits remnant masses, so the solar-metallicity population defines a hard formation boundary for the isolated-evolution channel.

## Related tools

- **COMPAS** (Rapid binary population synthesis code used to generate the DCO populations whose mass ratios and solar-metallicity boundary are analyzed) — http://github.com/TeamCOMPAS/COMPAS_
- **STROOPWAFEL** (COMPAS importance-sampling algorithm used to obtain the population statistics efficiently)
- **pandas/numpy** (Post-processing of COMPAS DCO output tables: filtering to Hubble-time mergers and computing median mass ratios)

## Examples

```
merging = dco[(dco.TC==0) & (dco.Time < 13.8e9)]; print(np.median(merging.Mass1/merging.Mass2))
```

## Evaluation signals

- Median mass ratio of the merging BBH population at 10% solar metallicity is approximately 0.5
- GW150914 and LVT151012 are judged infeasible, and GW151226 is judged at the high-mass boundary, for solar-metallicity formation under the Fiducial model
- Only BBHs merging within a Hubble time are included in the statistics (time-delay filter applied before computing q)
- Rate sanity check: the Fiducial model yields ~2e4 merging BBHs per 1e9 solar masses of star formation at 10% solar metallicity, rising to ~3e4 at 5% solar metallicity
- Verdicts hold across the paper's metallicity set (solar, Z=0.002, Z=0.001), with low-Z (Z=0.001) progenitor total masses of at least ~160, ~60 and ~90 solar masses for the three events

## Limitations

- Only individual model parameters were varied independently relative to the Fiducial model; a full exploration of the parameter space (e.g. common-envelope alpha, LBV mass-loss rate) was deferred to future COMPAS studies
- Stellar rotation, tides, and wind accretion by a companion are not included in the Fiducial model
- The single-star (SSE) model grid extends only to 50 solar masses and is extrapolated above that limit; the merger-rate comparison to LIGO is approximate (no SFR/time-delay convolution)

## Evidence

- [abstract] COMPAS as rapid BPS engine used to explain all three events: "Here we use the rapid binary population synthesis code COMPAS to show that all three events can be explained by a single evolutionary channel"
- [discussion] Initial distributions: uniform mass ratio, all stars in binaries: "a uniform mass ratio distribution and assuming that all stars are in binaries"
- [discussion] Expected merging-BBH yield at 10% solar metallicity: "We form ∼2 × 104 BBHs that merge in a Hubble time per 1 × 109 solar masses of star formation at 10% solar metallicity"
- [readme] COMPAS evolves binaries from ZAMS to compact remnants: "COMPAS draws properties for a binary star system from a set of initial distributions, and evolves it from zero-age main sequence to the end of its life as two compact remnants"
- [intro] COMPAS is publicly available and adjustable: "COMPAS is a publicly available rapid binary population synthesis code (https://compas.science/)"
