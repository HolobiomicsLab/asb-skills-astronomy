---
name: chirp-mass-and-mass-ratio-computation
description: Use when a binary-coalescence dataset has component masses (m1, m2) but
  lacks the LIGO-comparable derived quantities chirp mass and mass ratio needed for
  event-selection filtering.
when_to_use_negative:
- When the input already contains chirp mass and mass ratio columns, or when the dataset
  is not a coalescing compact-object population (e.g. it lacks two component masses).
- When the goal is to predict new LIGO events or rates rather than retro-match a simulated
  population to the three specific O1 events — the article defers full parameter-space
  exploration and rate convolution to future studies.
- 'When observed data is at issue rather than simulation: credible-interval cuts replicate
  the paper''s Fiducial-model selection and should not be imposed on the LIGO posterior
  samples themselves.'
edam_operation: http://edamontology.org/operation_3695
tools:
- name: COMPAS
  role: Rapid binary population synthesis code that produces the DoubleCompactObjects
    BBH population whose component masses feed the chirp-mass/mass-ratio computation
    and event-selection filter
  repo: http://github.com/TeamCOMPAS/COMPAS_
- name: pandas
  role: Used to load the DoubleCompactObjects table and apply the Hubble-time and
    per-event 90% credible-interval cuts
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/ncomms14906
    title: Formation of the first three gravitational-wave observations through isolated
      binary evolution
  source_content_hash: sha256:775b934790bd4d17c2c396c2ee0bab87d38a6ded810166c48330dad1c98a6b2e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COMPAS
  - pandas
  edam_operation: http://edamontology.org/operation_3695
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# chirp-mass-and-mass-ratio-computation

## Summary

Computes the chirp mass M_c = (m1*m2)^(3/5)/(m1+m2)^(1/5) and mass ratio q for compact-object binaries (here BBHs from COMPAS DoubleCompactObjects output) so that simulated systems can be matched against the 90% credible-interval mass and q constraints of observed LIGO events like GW150914, GW151226 and LVT151012.

## When to use

Use when a binary-coalescence dataset has component masses (m1, m2) but lacks the LIGO-comparable derived quantities chirp mass and mass ratio needed for event-selection filtering.

## When NOT to use

- When the input already contains chirp mass and mass ratio columns, or when the dataset is not a coalescing compact-object population (e.g. it lacks two component masses).
- When the goal is to predict new LIGO events or rates rather than retro-match a simulated population to the three specific O1 events — the article defers full parameter-space exploration and rate convolution to future studies.
- When observed data is at issue rather than simulation: credible-interval cuts replicate the paper's Fiducial-model selection and should not be imposed on the LIGO posterior samples themselves.

## Inputs

- COMPAS fiducial-population DoubleCompactObjects table (e.g. CSV/H5) with component masses m1, m2 and merger (inspiral) time per binary
- 90% credible-interval bounds on total (chirp) mass and mass ratio for GW150914, GW151226 and LVT151012 from the LIGO O1 discovery papers

## Outputs

- Per-binary chirp mass M_c and mass ratio q appended to the filtered BBH table
- event_selection_matches.csv: per-event matched systems with component masses, chirp mass, mass ratio and delay time

## How to apply

Load the COMPAS DoubleCompactObjects table (pandas) and select only BBHs whose gravitational-wave inspiral time is shorter than a Hubble time (~13.8 Gyr), since the Fiducial model yields ~2e4 merging BBHs per 1e9 solar masses of star formation at 10% solar metallicity. For each surviving system compute the chirp mass as M_c = (m1*m2)^(3/5)/(m1+m2)^(1/5) and the mass ratio q from the two component masses. Then, for each target event (GW150914, GW151226, LVT151012), retain systems whose total (or chirp) mass falls inside that event's 90% credible interval and whose q exceeds the event's 90% credible-interval lower bound, using values from the LIGO O1 discovery papers. Tabulate the matched systems with component masses, chirp mass, mass ratio and delay time, and save the per-event selection as event_selection_matches.csv. The rationale is that chirp mass is the best-measured LIGO parameter, making it the natural matching quantity between simulation and observation.

## Related tools

- **COMPAS** (Rapid binary population synthesis code that produces the DoubleCompactObjects BBH population whose component masses feed the chirp-mass/mass-ratio computation and event-selection filter) — http://github.com/TeamCOMPAS/COMPAS_
- **pandas** (Used to load the DoubleCompactObjects table and apply the Hubble-time and per-event 90% credible-interval cuts)

## Examples

```
df = pd.read_h5('DoubleCompactObjects.h5'); df = df[df.tinsp < 13.8e9]; df['Mchirp'] = (df.m1*df.m2)**0.6/(df.m1+df.m2)**0.2; df['q'] = df.m2/df.m1
```

## Evaluation signals

- Every computed M_c must satisfy M_c < (m1+m2)/2 and lie between the component masses' symmetric combination bounds; spot-check against hand-recalculation from (m1, m2).
- All retained binaries must have inspiral time shorter than ~13.8 Gyr (a Hubble time) before event cuts are applied.
- Under the Fiducial model, each of GW150914, GW151226 and LVT151012 must have at least one matched system in event_selection_matches.csv, consistent with the paper's conclusion that all three observed systems are explained through this channel.
- Matched systems' total (chirp) masses must fall within the corresponding event's 90% credible interval, and their q must exceed the event's 90% credible-interval lower bound.
- The output schema must contain component masses, chirp mass, mass ratio and delay time for every matched system.

## Limitations

- The Fiducial model excludes stellar rotation and tides, wind accretion by the companion, and extrapolates the single-star grid above 50 solar masses, so matched progenitors carry those model assumptions.
- Only GW150914, GW151226 and LVT151012 (O1 events) are used as selection targets; the channel explanation for future events was untested at publication time.
- Mass-ratio credible-interval lower bounds are treated as one-sided cuts (q must exceed the lower bound), not a full posterior match.

## Evidence

- [discussion] Fiducial Hubble-time yield: "We form ∼2 × 104 BBHs that merge in a Hubble time per 1 × 109 solar masses of star formation at 10% solar metallicity"
- [discussion] All three events matched: "All three observed systems can be explained through this channel under our Fiducial model assumptions."
- [methods] COMPAS provenance: "Simulations in this paper made use of the COMPAS rapid binary population synthesis code (version X.X.X), which is freely available at http://github.com/TeamCOMPAS/COMPAS_"
- [readme] COMPAS output description: "COMPAS draws properties for a binary star system from a set of initial distributions, and evolves it from zero-age main sequence to the end of its life as two compact"
- [discussion] Unconstrained massive binary physics: "The evolution of massive progenitor binaries is poorly constrained by observations, although there has been recent progress"
