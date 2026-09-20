---
name: gravitational-wave-event-progenitor-matching
description: Use when you must match specific observed BBH events (90% credible intervals
  on chirp/component masses and mass ratio) to progenitor binaries in COMPAS double-compact-object
  populations at particular metallicities (solar, Z=0.002, Z=0.001).
when_to_use_negative:
- The goal is total BBH merger-rate estimation or parameter-uncertainty exploration
  rather than progenitor matching for specific observed events
- The population of interest is non-merging systems or objects other than double compact
  objects produced by COMPAS
- Input observations are not gravitational-wave event posteriors (e.g. no credible
  intervals on chirp/component masses exist to filter against)
edam_operation: http://edamontology.org/operation_2945
tools:
- name: COMPAS
  role: Rapid binary population synthesis code used to generate the fiducial double
    compact object populations (Kroupa IMF to 100 solar masses, flat mass ratio, flat-in-log
    semimajor axis) whose merging BBHs are matched to the observed events
  repo: http://github.com/TeamCOMPAS/COMPAS
- name: STROOPWAFEL
  role: COMPAS importance sampling algorithm used to efficiently sample the rare merging-BBH
    progenitor subpopulations being matched to the events
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/ncomms14906
    title: Formation of the first three gravitational-wave observations through isolated
      binary evolution
  source_content_hash: sha256:9cee886f512e169f03020b4dbaf708276bba73d2723b763909fab381b767e9fb
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
yes_rate: 0.7777777777777778
n_claims: 9
---

# gravitational-wave-event-progenitor-matching

## Summary

Matches observed binary-black-hole gravitational-wave events (GW150914, GW151226, LVT151012) to progenitor binaries in fiducial COMPAS population-synthesis outputs, by filtering merging BBHs within the events' 90% credible mass intervals and tabulating zero-age-main-sequence total masses at Z=0.002 and Z=0.001.

## When to use

Use when you must match specific observed BBH events (90% credible intervals on chirp/component masses and mass ratio) to progenitor binaries in COMPAS double-compact-object populations at particular metallicities (solar, Z=0.002, Z=0.001).

## When NOT to use

- The goal is total BBH merger-rate estimation or parameter-uncertainty exploration rather than progenitor matching for specific observed events
- The population of interest is non-merging systems or objects other than double compact objects produced by COMPAS
- Input observations are not gravitational-wave event posteriors (e.g. no credible intervals on chirp/component masses exist to filter against)

## Inputs

- Fiducial-model COMPAS double compact object population outputs at solar, Z=0.002 and Z=0.001 metallicity (from the COMPAS Zenodo community deposit)
- 90% credible intervals on chirp mass, component masses and mass ratio for GW150914, GW151226 and LVT151012
- COMPAS merger-time prescription (Peters equations) for filtering BBHs that merge within a Hubble time

## Outputs

- pandas DataFrame / CSV (e.g. progenitor_initial_masses.csv) of minimum and typical zero-age-main-sequence total masses per event per metallicity
- Solar-metallicity formation verdicts per event (infeasible / at high-mass boundary / feasible) under classical isolated binary evolution with a common envelope phase
- Median mass ratio of the merging BBH population per metallicity (~0.5 at 10% solar)

## How to apply

Load the fiducial-model COMPAS double compact object outputs at solar, Z=0.002 (10% solar) and Z=0.001 (5% solar) metallicity from the COMPAS Zenodo community deposit. Filter to binary black holes that merge within a Hubble time using COMPAS's merger-time prescription (Peters equations) to build the event-selection sample. For each event, select progenitors whose chirp mass, component masses and mass ratio are consistent with the 90% credible intervals, then compute the zero-age-main-sequence total mass (m1+m2) for each selected progenitor at each metallicity. Tabulate minimum and typical initial total masses per event per metallicity (expected: ≥160, ≥60, ≥90 solar masses at Z=0.001 for GW150914, GW151226, LVT151012; 65–100 and 95–125 solar masses at Z=0.002) and write to progenitor_initial_masses.csv. As a cross-check, verify the median mass ratio of the merging BBH population is ~0.5 at 10% solar metallicity and classify solar-metallicity formation as infeasible (GW150914, LVT151012) or at the high-mass boundary (GW151226). Rationale: the Fiducial COMPAS model — Kroupa IMF to 100 solar masses, flat mass ratio, flat-in-log semimajor axis, circular binaries, common-envelope channel — is the single channel shown to explain all three events.

## Related tools

- **COMPAS** (Rapid binary population synthesis code used to generate the fiducial double compact object populations (Kroupa IMF to 100 solar masses, flat mass ratio, flat-in-log semimajor axis) whose merging BBHs are matched to the observed events) — http://github.com/TeamCOMPAS/COMPAS
- **STROOPWAFEL** (COMPAS importance sampling algorithm used to efficiently sample the rare merging-BBH progenitor subpopulations being matched to the events)

## Examples

```
import pandas as pd; d=pd.read_hdf('COMPAS_Output.h5','DoubleCompactObjects'); bbh=d[(d['Merges_Hubble_Time']==1)&(d['Metallicity@ZAMS']==0.001)]
```

## Evaluation signals

- Minimum initial total masses at Z=0.001 are at least approximately 160, 60 and 90 solar masses for GW150914, GW151226 and LVT151012 respectively
- Z=0.002 initial total masses fall within the reported 65–100 and 95–125 solar mass ranges
- Computed median mass ratio of merging BBHs at 10% solar metallicity is approximately 0.5
- Solar-metallicity verdicts reproduce: GW150914 and LVT151012 infeasible, GW151226 at the high-mass boundary of BBHs formable at solar metallicity
- Selection sample contains only BBHs that merge within a Hubble time per the COMPAS/Peters prescription

## Limitations

- Single-star model grid underlying SSE extends only to 50 solar masses and is extrapolated above that limit
- Stellar rotation, tides, and wind accretion by a companion are not included in the Fiducial model; individual parameters were varied only independently, with full multidimensional parameter-space exploration deferred
- Only merging BBHs under the Fiducial model are relevant; conclusions are channel-specific (classical isolated binary evolution via mass transfer including a common envelope phase)

## Evidence

- [abstract] typical total masses ≥160, ≥60 and ≥90 solar masses for GW150914, GW151226 and LVT151012 at Z=0.001: "We show all three events could have formed in low-metallicity environments ( Z =0.001) from progenitor binaries with typical total masses ≳160 M ⊙ , ≳60 M ⊙ and ≳90 M ⊙ , for GW150914, GW151226 and"
- [discussion] All three observed systems can be explained through the common-envelope channel under the Fiducial model: "All three observed systems can be explained through this channel under our Fiducial model assumptions."
- [readme] COMPAS evolves binaries from zero-age main sequence to two compact remnants: "COMPAS draws properties for a binary star system from a set of initial distributions, and evolves it from zero-age main sequence to the end of its life as two compact remnants."
- [intro] COMPAS is publicly available and adjustable: "COMPAS is a publicly available rapid binary population synthesis code (https://compas.science/) that is designed so that evolution prescriptions and model parameters are easily adjustable."
