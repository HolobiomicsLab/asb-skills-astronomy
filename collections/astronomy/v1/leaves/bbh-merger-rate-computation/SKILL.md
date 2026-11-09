---
name: bbh-merger-rate-computation
description: Use when quantifying how a COMPAS model-parameter change (e.g. LBV mass-loss
  rate 1.5e-4 → 1e-3 Msun/yr, common-envelope alpha, or alpha_lambda) alters the total
  BBH merger rate or the count of specific sub-populations (e.g. LVT151012-like ~90
  Msun total-mass mergers) at a given metallicity.
when_to_use_negative:
- When an accurate volumetric merger rate is required — that needs convolution of
  the metallicity-specific redshift-dependent star formation rate with the time delay
  distribution, which this per-formed-population counting approach does not perform
- When the question involves stellar rotation, tides, or wind accretion onto a companion
  — these are not modelled in the COMPAS Fiducial framework
- When probing coupled parameter effects — the method varies individual parameters
  independently and explicitly defers full multidimensional exploration (e.g. over
  alpha, fLBV) to future studies
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: COMPAS
  role: Rapid binary population synthesis code that draws binary initial conditions
    (Kroupa IMF m1 <= 100 Msun, flat mass ratio, flat-in-log semimajor axes, circular)
    and evolves systems to compact remnants; post-processing scripts yield merger
    counts per formed population
  repo: http://github.com/TeamCOMPAS/COMPAS
- name: STROOPWAFEL
  role: COMPAS importance-sampling algorithm used to efficiently sample the rare BBH-merger
    channel and reduce uncertainty on merger yields
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/ncomms14906
    title: Formation of the first three gravitational-wave observations through isolated
      binary evolution
  source_content_hash: sha256:10789447b29b0ddb547e0b92deeb3f882729b1c8345e845482730bd4a59a27dc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COMPAS
  - STROOPWAFEL
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# bbh-merger-rate-computation

## Summary

Compute binary black hole (BBH) merger yields and rates from COMPAS binary population synthesis runs, expressed as mergers within a Hubble time per formed population and rescaled per 1e9 solar masses of star formation, enabling comparison of model-parameter variations against LIGO rate estimates.

## When to use

Use when quantifying how a COMPAS model-parameter change (e.g. LBV mass-loss rate 1.5e-4 → 1e-3 Msun/yr, common-envelope alpha, or alpha_lambda) alters the total BBH merger rate or the count of specific sub-populations (e.g. LVT151012-like ~90 Msun total-mass mergers) at a given metallicity.

## When NOT to use

- When an accurate volumetric merger rate is required — that needs convolution of the metallicity-specific redshift-dependent star formation rate with the time delay distribution, which this per-formed-population counting approach does not perform
- When the question involves stellar rotation, tides, or wind accretion onto a companion — these are not modelled in the COMPAS Fiducial framework
- When probing coupled parameter effects — the method varies individual parameters independently and explicitly defers full multidimensional exploration (e.g. over alpha, fLBV) to future studies

## Inputs

- COMPAS input settings file for the Fiducial model (from the COMPAS Zenodo community)
- COMPAS reference/post-processed results for the Fiducial run (from the COMPAS results site)
- Modified COMPAS run output (COMPASOutput/post-processing files) with the varied parameter (e.g. LBV mass-loss rate 1e-3 Msun/yr at Z=0.001)
- Metallicity grid values at which rates are quoted (10% and 5% solar metallicity)

## Outputs

- Total BBH merger rate: number of BBHs merging in a Hubble time per 1e9 solar masses of star formation
- Sub-population merger counts (e.g. LVT151012-like mergers, progenitor total mass ~90 Msun regime)
- Comparison table (lbv_variation_results.csv) contrasting varied-run vs Fiducial-run yields
- Optionally, rescaled volumetric BBH formation rate (Gpc−3 per year at z=0)

## How to apply

Run COMPAS with the Fiducial configuration at the target metallicity (e.g. Z=0.001, 5% solar): primary masses drawn from the Kroupa IMF up to m1 ≤ 100 Msun, flat-in-log semimajor axes, flat mass ratio, circular binaries, all stars in binaries, using STROOPWAFEL importance sampling. Vary exactly one parameter relative to Fiducial per run (e.g. fLBV from 1.5e-4 to 1e-3 Msun/yr) so the rate change is attributable to that parameter. Use COMPAS post-processing scripts to count BBHs that merge in a Hubble time per formed population, then normalize per 1e9 solar masses of star formation (Fiducial gives ~2e4 at 10% solar and ~3e4 at 5% solar metallicity). Optionally rescale to a volumetric formation rate (~300 Gpc−3 per year at z=0) for comparison with the LIGO empirical estimate of 9–240 Gpc−3 per year, noting this is approximate. Tabulate per-variant comparisons (e.g. to lbv_variation_results.csv), validating that only the targeted sub-population count changed as reported.

## Related tools

- **COMPAS** (Rapid binary population synthesis code that draws binary initial conditions (Kroupa IMF m1 <= 100 Msun, flat mass ratio, flat-in-log semimajor axes, circular) and evolves systems to compact remnants; post-processing scripts yield merger counts per formed population) — http://github.com/TeamCOMPAS/COMPAS
- **STROOPWAFEL** (COMPAS importance-sampling algorithm used to efficiently sample the rare BBH-merger channel and reduce uncertainty on merger yields)

## Evaluation signals

- Fiducial normalization check: ~2e4 merging BBHs per 1e9 solar masses of star formation at 10% solar metallicity, rising to ~3e4 at 5% solar metallicity
- Single-parameter invariant: total BBH merger rate should not significantly change when only the LBV mass-loss rate is raised from 1.5e-4 to 1e-3 Msun/yr at Z=0.001, while LVT151012-like mergers drop by a factor of ~10
- Rescaled rate plausibility: BBH formation rate ~300 Gpc−3 per year at z=0 should be compared against the LIGO empirical estimate of 9–240 Gpc−3 per year
- Only one parameter differs from Fiducial between runs (e.g. alpha_lambda 0.1 → 0.01 should drop the total rate by ~2x at 10% and ~4x at 5% solar metallicity)
- Output comparison table (lbv_variation_results.csv) is written and matches fiducial reference counts from the COMPAS results site

## Limitations

- An accurate merger-rate calculation requires convolution of the metallicity-specific redshift-dependent star formation rate with the time delay distribution; this was not done, so the Gpc−3 comparison is approximate
- The SSE single-star model grid extends only to 50 solar masses and is extrapolated above that limit for massive primaries
- Stellar rotation, tides, and wind accretion by a companion are not included in the Fiducial model
- Massive-binary evolution is poorly observationally constrained; only individual parameters were varied, with full parameter-space exploration deferred

## Evidence

- [discussion] Fiducial rate normalization: "We form ∼2 × 104 BBHs that merge in a Hubble time per 1 × 109 solar masses of star formation at 10% solar metallicity"
- [discussion] Rescaled rate vs LIGO estimate: [evidence span withheld — anchor status: not_found]
- [discussion] LBV variation result and validation: "does not significantly change the total BBHs merger rate; nevertheless, the number of BBHs mergers similar to LVT151012 was reduced by a factor of ∼10"
- [discussion] Initial condition specification: [evidence span withheld — anchor status: not_found]
- [discussion] Rate convolution caveat: "An accurate calculation of the merger rate requires the convolution of the metallicity-specific redshift-dependent star formation rate with the time delay distribution"
