---
name: gravitational-wave-event-mass-selection
description: Use when you have a COMPAS DoubleCompactObjects table of merging BBHs
  and need to identify which simulated systems match the 90% credible intervals of
  specific observed gravitational-wave events.
when_to_use_negative:
- When the goal is population-level BBH merger-rate inference or parameter-uncertainty
  exploration (e.g. varying alpha or f_LBV) rather than matching individual observed
  events.
- When you need the full simulated population including systems that do not merge
  within a Hubble time — the Hubble-time cut discards those systems by design.
- For gravitational-wave events that lack published 90% credible-interval mass and
  mass-ratio bounds to cut against.
edam_operation: http://edamontology.org/operation_3695
tools:
- name: COMPAS
  role: Rapid binary population synthesis code whose fiducial-model DoubleCompactObjects
    output supplies the simulated BBH population that the event-selection filter is
    applied to
  repo: https://github.com/TeamCOMPAS/COMPAS
- name: pandas
  role: Used to load and filter the DoubleCompactObjects table and to compute derived
    chirp mass and mass-ratio columns
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/ncomms14906
    title: Formation of the first three gravitational-wave observations through isolated
      binary evolution
  source_content_hash: sha256:0a4fb29eff8325cb472e2e6aa72011a31cb9bcdd5469c0a09814f52c8111b712
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TeamCOMPAS/COMPAS
  tools:
  - COMPAS
  - pandas
  edam_operation: http://edamontology.org/operation_3695
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# gravitational-wave-event-mass-selection

## Summary

Filters a COMPAS-simulated fiducial binary black hole population down to the progenitor candidates of specific LIGO detections (GW150914, GW151226, LVT151012) by applying each event's published 90% credible-interval cuts on total/chirp mass and mass ratio, retaining only systems that merge within a Hubble time.

## When to use

Use when you have a COMPAS DoubleCompactObjects table of merging BBHs and need to identify which simulated systems match the 90% credible intervals of specific observed gravitational-wave events.

## When NOT to use

- When the goal is population-level BBH merger-rate inference or parameter-uncertainty exploration (e.g. varying alpha or f_LBV) rather than matching individual observed events.
- When you need the full simulated population including systems that do not merge within a Hubble time — the Hubble-time cut discards those systems by design.
- For gravitational-wave events that lack published 90% credible-interval mass and mass-ratio bounds to cut against.

## Inputs

- COMPAS fiducial-population DoubleCompactObjects table (simulated BBH population, loaded with pandas)
- 90% credible-interval values for total/chirp mass and mass ratio of GW150914, GW151226 and LVT151012 from the LIGO O1 discovery papers

## Outputs

- event_selection_matches.csv — per-event table of matched systems with component masses, chirp mass, mass ratio q and delay time

## How to apply

Load the COMPAS fiducial-population DoubleCompactObjects table with pandas, and first keep only systems whose gravitational-wave inspiral (merger) time is shorter than a Hubble time (~13.8 Gyr), since the fiducial model forms ~2e4 merging BBHs per 1e9 solar masses of star formation at 10% solar metallicity. For each of GW150914, GW151226 and LVT151012, impose the 90% credible-interval cuts on total (or chirp) mass and on mass ratio q taken from the LIGO O1 discovery papers, requiring q to exceed the event's 90% CI lower bound. Where needed, compute the chirp mass M_c = (m1*m2)^(3/5)/(m1+m2)^(1/5) and mass ratio q from the component masses of each surviving system. Tabulate the matched systems per event with component masses, chirp mass, mass ratio and delay time, and save the per-event results as event_selection_matches.csv. Under the Fiducial model, validation requires that every one of the three events has at least one matching system, consistent with the paper's conclusion that classical isolated binary evolution via a common envelope phase explains all three events.

## Related tools

- **COMPAS** (Rapid binary population synthesis code whose fiducial-model DoubleCompactObjects output supplies the simulated BBH population that the event-selection filter is applied to) — https://github.com/TeamCOMPAS/COMPAS
- **pandas** (Used to load and filter the DoubleCompactObjects table and to compute derived chirp mass and mass-ratio columns)

## Examples

```
import pandas as pd; dco = pd.read_hdf('DoubleCompactObjects.h5', 'DoubleCompactObjects'); merging = dco[dco['mergerTime'] < 13.8e3]  # keep BBHs merging within Hubble time before 90% CI mass/q cuts
```

## Evaluation signals

- Under the Fiducial model, each of GW150914, GW151226 and LVT151012 has at least one matching system in the filtered population.
- Every retained system has a gravitational-wave merger time shorter than a Hubble time (~13.8 Gyr).
- All survivors satisfy the event-specific 90% credible-interval cuts: total (chirp) mass inside the interval and mass ratio q above the interval's lower bound.
- Recomputed chirp mass M_c = (m1*m2)^(3/5)/(m1+m2)^(1/5) is consistent with the component masses listed per matched system.
- Output event_selection_matches.csv exists and contains component masses, chirp mass, mass ratio and delay time for each matched system.

## Limitations

- Credible-interval bounds come from the LIGO O1 discovery papers, and LVT151012 is a statistically less significant candidate, so its cuts are correspondingly less robust.
- The Fiducial COMPAS model excludes stellar rotation, tides, and wind accretion by a companion, so matched progenitors reflect those simplifying assumptions.
- The single-star (SSE) grid extends only to 50 solar masses and is extrapolated above that limit, affecting the highest-mass candidates such as GW150914 progenitors.
- Match systems are only as good as the fiducial model prescriptions (e.g. common envelope treatment); varying parameters like alpha_lambda or the LBV mass-loss rate changes which systems survive.

## Evidence

- [discussion] All three observed systems explained under Fiducial model: "All three observed systems can be explained through this channel under our Fiducial model assumptions."
- [discussion] Fiducial BBH formation rate: "We form ∼2 × 104 BBHs that merge in a Hubble time per 1 × 109 solar masses of star formation at 10% solar metallicity"
- [intro] COMPAS designed for adjustable evolution prescriptions: "COMPAS is a publicly available rapid binary population synthesis code (https://compas.science/) that is designed so that evolution prescriptions and model parameters are easily adjustable."
- [readme] COMPAS evolves binaries to compact remnants used for GW inference: "COMPAS draws properties for a binary star system from a set of initial distributions, and evolves it from zero-age main sequence to the end of its life as two compact remnants."
