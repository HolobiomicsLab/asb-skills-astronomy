---
name: binary-population-synthesis-simulation
description: Use when you need to simulate large populations of massive binary stars
  (Kroupa IMF, m1 up to 100 Msun) and predict compact-remnant merger outcomes as a
  function of metallicity (e.g. Z=0.002 and Z=0.001).
when_to_use_negative:
- 'You need a fully convolved cosmic merger rate: the paper notes an accurate rate
  requires convolution of metallicity-specific redshift-dependent star formation history
  with the delay-time distribution, which the fiducial workflow does not perform'
- 'You need stellar rotation, tides, or wind accretion modeled: these are excluded
  from the Fiducial model and would require reconfiguring COMPAS beyond the fiducial
  setup'
- 'You require full multi-dimensional parameter-space exploration: the paper only
  varies individual parameters independently relative to the Fiducial model'
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_0203
tools:
- name: COMPAS
  role: Rapid binary population synthesis code that draws binaries from initial distributions
    and evolves them from ZAMS to double compact remnants; prescriptions and parameters
    are easily adjustable
  repo: http://github.com/TeamCOMPAS/COMPAS
- name: STROOPWAFEL
  role: Adaptive importance-sampling algorithm used with COMPAS to efficiently sample
    the rare merging-BBH subpopulation
- name: BSE
  role: Reference rapid population-synthesis approach on which COMPAS's methodology
    is broadly similar
provenance:
  source_task_ids:
  - task_001
  - task_005
  - task_007
  - task_003
  - task_004
  - task_002
  source_papers:
  - doi: 10.1038/ncomms14906
    title: Formation of the first three gravitational-wave observations through isolated
      binary evolution
  source_content_hash: sha256:78cfdf99b76462fe8b3828e53d47bede3a9d5a1e1722affebe8e870305dce5cb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BSE
  - COMPAS
  - STROOPWAFEL
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# binary-population-synthesis-simulation

## Summary

Run rapid binary population synthesis with COMPAS to evolve zero-age-main-sequence binary populations into double compact objects, quantify which systems merge within a Hubble time, and yield metallicity-specific binary black hole (BBH) merger rates per 1e9 solar masses of star formation.

## When to use

Use when you need to simulate large populations of massive binary stars (Kroupa IMF, m1 up to 100 Msun) and predict compact-remnant merger outcomes as a function of metallicity (e.g. Z=0.002 and Z=0.001).

## When NOT to use

- You need a fully convolved cosmic merger rate: the paper notes an accurate rate requires convolution of metallicity-specific redshift-dependent star formation history with the delay-time distribution, which the fiducial workflow does not perform
- You need stellar rotation, tides, or wind accretion modeled: these are excluded from the Fiducial model and would require reconfiguring COMPAS beyond the fiducial setup
- You require full multi-dimensional parameter-space exploration: the paper only varies individual parameters independently relative to the Fiducial model

## Inputs

- COMPAS configuration file specifying Fiducial parameters (Fryer delayed SN alpha=1, alpha_lambda=0.1, f_LBV=1.5, Vink winds, no rotation/tides/wind accretion)
- Initial distribution parameters: Kroupa IMF (m1 <= 100 Msun), flat mass ratio, flat-in-log semimajor axis, zero eccentricity
- Target metallicity values (Z=0.002, Z=0.001, solar)
- Number of simulated binaries / STROOPWAFEL importance-sampling settings
- Reference fiducial outputs from the COMPAS Zenodo community for validation

## Outputs

- DoubleCompactObjects tables with component masses, chirp mass, mass ratio, delay (merger) time per system
- Counts of BBHs merging within a Hubble time per 1e9 Msun of star formation per metallicity (e.g. ~2e4 at 10% Zsun, ~3e4 at 5% Zsun)
- Rescaled volumetric BBH merger rate at z=0 (e.g. ~300 Gpc^-3 yr^-1)
- CSV comparison tables (e.g. bbh_merger_rates.csv, per-event selection matches, parameter-variation sensitivity tables)

## How to apply

Configure COMPAS with the Fiducial physics: delayed Fryer supernova model (alpha=1), common-envelope efficiency alpha_lambda=0.1, LBV wind rate 1.5e-4 Msun/yr (f_LBV=1.5), Vink winds, with stellar rotation, tides and wind accretion switched off. Draw initial conditions as all stars in binaries: primary mass from a Kroupa IMF up to 100 Msun, a uniform (flat) mass-ratio distribution, a flat-in-the-log semimajor-axis distribution, and zero initial eccentricity. Evolve the population at fixed metallicities (e.g. 10% and 5% solar) through mass transfer and common envelope to the double compact object stage, then compute GW inspiral times with the Peters prescription and retain only BBHs merging within a Hubble time (~13.8 Gyr). Use STROOPWAFEL adaptive importance sampling to efficiently target the rare merging-BBH subpopulation. Normalize merging-BBH counts per 1e9 solar masses of star formation at each metallicity and rescale to a volumetric BBH merger rate at z=0; vary single parameters (e.g. alpha_lambda=0.01, LBV rate 1e-3 Msun/yr) one at a time against the Fiducial baseline to quantify model uncertainty.

## Related tools

- **COMPAS** (Rapid binary population synthesis code that draws binaries from initial distributions and evolves them from ZAMS to double compact remnants; prescriptions and parameters are easily adjustable) — http://github.com/TeamCOMPAS/COMPAS
- **STROOPWAFEL** (Adaptive importance-sampling algorithm used with COMPAS to efficiently sample the rare merging-BBH subpopulation)
- **BSE** (Reference rapid population-synthesis approach on which COMPAS's methodology is broadly similar)

## Examples

```
python compas.py --initial-mass-function kroupa --maximum-initial-mass 100 --metallicity 0.001 --fryer-supernova-engine delayed --common-envelope-alpha 1.0 --common-envelope-lambda 0.1 --lbv-mass-loss-multiplier 1.5
```

## Evaluation signals

- Fiducial run at 10% solar metallicity yields ~2e4 merging BBHs per 1e9 Msun of star formation, rising to ~3e4 at 5% solar metallicity
- Rescaled BBH formation rate ~300 Gpc^-3 per year falls within (or near) the LIGO empirical estimate of 9-240 Gpc^-3 per year
- Every one of GW150914, GW151226 and LVT151012 has at least one matching simulated system under 90% credible-interval cuts on total/chirp mass and mass ratio
- Minimum progenitor ZAMS total masses at Z=0.001 reach at least ~160, ~60 and ~90 Msun for GW150914-, GW151226- and LVT151012-like systems
- Single-parameter sensitivity checks reproduce reported drop factors (alpha_lambda 0.1->0.01: ~2x rate drop at 10% Zsun, ~4x at 5% Zsun; LBV rate 1.5e-4->1e-3: total rate unchanged but LVT151012-like mergers drop ~10x)

## Limitations

- The single-star (SSE) grid underlying stellar evolution extends only to 50 solar masses and is extrapolated above that limit
- Stellar rotation, tides, and wind accretion by the companion are not included in the Fiducial model
- Merger-rate comparison to LIGO is approximate; an accurate rate requires convolution of metallicity-specific redshift-dependent SFR with the delay-time distribution, which was not done
- Full exploration of the multidimensional parameter space (e.g. alpha, f_LBV) is deferred; only individual parameters are varied independently
- The ability of a single formation channel to explain all observed events remained untested pending future GW observations

## Evidence

- [intro] COMPAS is a publicly available rapid binary population synthesis code (https://compas.science/) that is designed so that evolution prescriptions and model parameters are easily adjustable.: "COMPAS is a publicly available rapid binary population synthesis code (https://compas.science/) that is designed so that evolution prescriptions and parameters are easily adjustable."
- [discussion] We form ∼2 × 104 BBHs that merge in a Hubble time per 1 × 109 solar masses of star formation at 10% solar metallicity: [evidence span withheld — anchor status: not_found]
- [discussion] the primary mass m1 drawn from the Kroupa IMF31 up to m1≤100M⊙ ... a flat-in-the-log distribution6162 ... an initial eccentricity of zero: [evidence span withheld — anchor status: not_found]
- [readme] COMPAS draws properties for a binary star system from a set of initial distributions, and evolves it from zero-age main sequence to the end of its life as two compact remnants.: "COMPAS draws properties for a binary star system from a set of initial distributions, and evolves it from zero-age main sequence to the end of its life as two compact remnants."
- [readme] If you use COMPAS's importance sampling algorithm STROOPWAFEL, please cite: [evidence span withheld — anchor status: not_found]
