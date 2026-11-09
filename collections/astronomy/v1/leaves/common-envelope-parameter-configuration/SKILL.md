---
name: common-envelope-parameter-configuration
description: Use when running COMPAS (or similar BSE-derived) binary population synthesis
  at fixed metallicity (e.g. Z=0.002 and Z=0.001) and you must set or vary the common-envelope
  prescription (alpha_lambda) to test its impact on double-compact-object yields and
  BBH merger rates.
when_to_use_negative:
- When the goal is a full multidimensional exploration of the CE/model parameter space
  (e.g. joint alpha and fLBV sampling) rather than one-at-a-time variations relative
  to Fiducial — the paper explicitly defers this to future studies
- When the research question concerns wind-driven mass loss (LBV wind multiplier fLBV)
  rather than common-envelope ejection efficiency; fLBV is a separate parameter with
  a separate sensitivity analysis
edam_operation: http://edamontology.org/operation_0244
tools:
- name: COMPAS
  role: Rapid binary population synthesis code used to evolve binaries from zero-age
    main sequence through common envelope to the double-compact-object stage under
    the chosen CE parameterization
  repo: http://github.com/TeamCOMPAS/COMPAS_
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/ncomms14906
    title: Formation of the first three gravitational-wave observations through isolated
      binary evolution
  source_content_hash: sha256:c2ae0e22f28c4fd93b7268b67130c3bde467fd8630d54438377b505402e8e741
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COMPAS
  edam_operation: http://edamontology.org/operation_0244
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# common-envelope-parameter-configuration

## Summary

Configure and perturb the common-envelope (CE) physics parameters of the COMPAS binary population synthesis code — most importantly the common-envelope efficiency parameter alpha_lambda (fiducial 0.1, versus 0.01 and an 'optimistic' CE treatment) — to quantify how CE assumptions propagate into merging-BBH yields and volumetric merger rates.

## When to use

Use when running COMPAS (or similar BSE-derived) binary population synthesis at fixed metallicity (e.g. Z=0.002 and Z=0.001) and you must set or vary the common-envelope prescription (alpha_lambda) to test its impact on double-compact-object yields and BBH merger rates.

## When NOT to use

- When the goal is a full multidimensional exploration of the CE/model parameter space (e.g. joint alpha and fLBV sampling) rather than one-at-a-time variations relative to Fiducial — the paper explicitly defers this to future studies
- When the research question concerns wind-driven mass loss (LBV wind multiplier fLBV) rather than common-envelope ejection efficiency; fLBV is a separate parameter with a separate sensitivity analysis

## Inputs

- COMPAS input parameter set (Fiducial: delayed Fryer SN model alpha=1, alpha_lambda=0.1, fLBV=1.5, Vink winds, rotation/tides/wind accretion off)
- Initial binary population draws: Kroupa IMF primaries up to 100 Msun, uniform mass ratio, flat-in-the-log semimajor axis, e=0
- Metallicity values Z=0.002 (10% solar) and Z=0.001 (5% solar)

## Outputs

- Per-metallicity counts of merging BBHs (merging within a Hubble time) per 1e9 Msun of star formation
- Rescaled volumetric BBH merger rate at z=0 (Gpc^-3 per year)
- Per-parameter-variant rate ratios relative to the Fiducial model (e.g. alpha_lambda=0.01, optimistic CE)
- Per-metallicity yields table (e.g. bbh_merger_rates.csv)

## How to apply

Set the Fiducial CE configuration: COMPAS with the delayed Fryer supernova model (alpha=1), common-envelope parameter alpha_lambda=0.1, with stellar rotation, tides, and wind accretion switched off (LBV wind multiplier fLBV=1.5, Vink winds). Evolve the initial population — all stars in binaries, Kroupa IMF primary masses up to 100 Msun, flat mass-ratio, flat-in-the-log semimajor axis, zero eccentricity — at 10% and 5% solar metallicity, selecting only double compact objects whose gravitational-wave merger time is less than a Hubble time. Then perform one-at-a-time sensitivity tests relative to Fiducial: lower alpha_lambda from 0.1 to 0.01, and separately adopt an 'optimistic' CE treatment, recomputing merging-BBH counts per 1e9 Msun of star formation and the rescaled volumetric rate at z=0. The rationale is that CE physics is poorly constrained by observations, so bracketing alpha_lambda (tighter envelope ejection lowers rates; optimistic survival of tighter post-CE systems raises them) bounds the uncertainty on the predicted rate, which under Fiducial is ~2e4–3e4 merging BBHs per 1e9 Msun and ~300 Gpc^-3 per year at z=0.

## Related tools

- **COMPAS** (Rapid binary population synthesis code used to evolve binaries from zero-age main sequence through common envelope to the double-compact-object stage under the chosen CE parameterization) — http://github.com/TeamCOMPAS/COMPAS_

## Evaluation signals

- Fiducial yields reproduce ~2e4 merging BBHs per 1e9 Msun of star formation at 10% solar metallicity and ~3e4 at 5% solar metallicity
- Rescaled rate is consistent with ~300 Gpc^-3 per year at z=0, within the LIGO empirical estimate of 9–240 Gpc^-3 per year
- alpha_lambda=0.01 variant drops the total BBH merger rate by ~2x at 10% solar metallicity and ~4x at 5% solar metallicity relative to Fiducial
- Optimistic CE variant raises total BBH merger rates by ~3x at 10% Zsun and ~2x at 5% Zsun relative to Fiducial
- All three events (GW150914, GW151226, LVT151012) remain reproducible via classical isolated binary evolution with a CE phase under the Fiducial assumptions

## Limitations

- Common-envelope physics is parametrized into one- or two-parameter models in rapid population synthesis; the evolution of massive progenitor binaries is poorly constrained by observations
- Only individual parameters were varied independently; a full exploration of the multidimensional parameter space (e.g. alpha, fLBV) is deferred to future COMPAS studies
- Merger-rate comparison to LIGO is approximate: an accurate rate requires convolution of the metallicity-specific redshift-dependent star formation rate with the time-delay distribution, which was not performed

## Evidence

- [discussion] alphalambda sensitivity: "changing the value of αλ from the fiducial 0.1 to 0.01. At 10% solar metallicity we find the total BBHs merger rate drops by a factor of ∼2"
- [discussion] optimistic CE: "the optimistic CE treatment predicts total BBHs merger rates, which are ∼3 times higher than the Fiducial model at Z=10%Z⊙ and ∼2 times higher at Z=5%Z⊙"
- [discussion] fiducial yield: "We form ∼2 × 104 BBHs that merge in a Hubble time per 1 × 109 solar masses of star formation at 10% solar metallicity"
- [discussion] parametrized CE models: "In rapid population synthesis codes such as COMPAS, these uncertainties are treated by parametrizing complex physical processes into simple one or two parameter models"
- [readme] COMPAS evolves binaries to compact remnants: "COMPAS draws properties for a binary star system from a set of initial distributions, and evolves it from zero-age main sequence to the end of its life as two compact"
