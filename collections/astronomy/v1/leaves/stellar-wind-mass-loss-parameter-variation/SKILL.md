---
name: stellar-wind-mass-loss-parameter-variation
description: Use when assessing how uncertainty in stellar wind mass loss (e.g. LBV
  winds at low metallicity, Z=0.001) propagates into binary black hole merger populations,
  or when reproducing parameter-sensitivity checks like the LVT151012 yield test.
when_to_use_negative:
- When the goal is a full exploration of the multidimensional model parameter space
  (e.g. varying alpha and fLBV jointly) — the paper explicitly defers this to future
  studies and only varies individual parameters independently
- When the parameter of interest is not a wind/mass-loss prescription (e.g. common-envelope
  treatment alpha_lambda or the Fryer supernova model) — different variation workflows
  apply
- When an accurate observational merger rate is required via convolution of metallicity-specific
  redshift-dependent star formation rate with the time delay distribution — this skill
  yields per-formed-population yields only
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: COMPAS
  role: Rapid binary population synthesis code used to run the Fiducial and modified
    (LBV 1e-3 Msun/yr) populations and to post-process merger statistics
  repo: http://github.com/TeamCOMPAS/COMPAS_
- name: STROOPWAFEL
  role: COMPAS importance sampling algorithm used to efficiently sample rare BBH merger
    channels in both fiducial and modified runs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/ncomms14906
    title: Formation of the first three gravitational-wave observations through isolated
      binary evolution
  source_content_hash: sha256:622a981e0abbee23a3eb2d05575a3af920aab8b97e0cf60e020c8f02d1da5890
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
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# stellar-wind-mass-loss-parameter-variation

## Summary

A one-at-a-time parameter-variation method for binary population synthesis in which a single wind prescription parameter (the LBV mass-loss rate) is changed relative to a Fiducial COMPAS model to quantify its impact on total BBH merger rates and on yields of specific events such as LVT151012-like mergers.

## When to use

Use when assessing how uncertainty in stellar wind mass loss (e.g. LBV winds at low metallicity, Z=0.001) propagates into binary black hole merger populations, or when reproducing parameter-sensitivity checks like the LVT151012 yield test.

## When NOT to use

- When the goal is a full exploration of the multidimensional model parameter space (e.g. varying alpha and fLBV jointly) — the paper explicitly defers this to future studies and only varies individual parameters independently
- When the parameter of interest is not a wind/mass-loss prescription (e.g. common-envelope treatment alpha_lambda or the Fryer supernova model) — different variation workflows apply
- When an accurate observational merger rate is required via convolution of metallicity-specific redshift-dependent star formation rate with the time delay distribution — this skill yields per-formed-population yields only

## Inputs

- COMPAS Fiducial model input settings and reference results (COMPAS Zenodo community; COMPAS results site)
- Fiducial run configuration at Z=0.001 with LBV mass-loss rate 1.5e-4 Msun/yr
- Modified COMPAS run output with LBV mass-loss rate 1e-3 Msun/yr at Z=0.001
- Initial distributions: Kroupa IMF (m1 <= 100 Msun), flat mass ratio, flat-in-log semimajor axis, zero eccentricity, all stars in binaries

## Outputs

- Total BBH merger rate (mergers within a Hubble time per formed population) for the modified run
- Count of LVT151012-like BBH mergers (progenitor total mass ~90 Msun regime) in the modified run
- Comparison table lbv_variation_results.csv (fiducial vs modified LBV mass-loss rate)

## How to apply

Start from the COMPAS Fiducial model configuration at Z=0.001 (5% solar metallicity) with the fiducial LBV mass-loss rate of 1.5e-4 Msun/yr, loading reference inputs from the COMPAS Zenodo community and merger statistics from the COMPAS results site. Draw primary masses from a Kroupa IMF (m1 <= 100 Msun), a flat mass-ratio distribution, and flat-in-log semimajor axes for circular binaries, evolving all stars in binaries with STROOPWAFEL importance sampling. Change ONLY the LBV mass-loss parameter to 1e-3 Msun/yr, keeping every other Fiducial setting fixed, following the paper's approach of varying individual parameters independently. Run the modified population with COMPAS, then compute the total BBH merger rate (mergers within a Hubble time per formed population) with COMPAS post-processing scripts and count LVT151012-like mergers (progenitor total mass ~90 Msun regime). Write the fiducial-vs-modified comparison to a results table (e.g. lbv_variation_results.csv) and validate against the published expectation.

## Related tools

- **COMPAS** (Rapid binary population synthesis code used to run the Fiducial and modified (LBV 1e-3 Msun/yr) populations and to post-process merger statistics) — http://github.com/TeamCOMPAS/COMPAS_
- **STROOPWAFEL** (COMPAS importance sampling algorithm used to efficiently sample rare BBH merger channels in both fiducial and modified runs)

## Examples

```
./compas --mass-loss-LBV 1e-3 --metallicity 0.001 --initial-mass-max 100 --sampling-type STROOPWAFEL
```

## Evaluation signals

- Total BBH merger rate at Z=0.001 must not be significantly changed relative to the Fiducial model when the LBV rate is raised to 1e-3 Msun/yr
- Number of LVT151012-like mergers (progenitor total mass ~90 Msun regime) must drop by a factor of ~10 versus the Fiducial count
- Only the LBV mass-loss parameter differs between the Fiducial and modified runs — all other settings (Kroupa IMF to 100 Msun, flat q, flat-in-log a, circular binaries, Z=0.001) must be identical
- lbv_variation_results.csv exists and contains the fiducial vs modified rate and LVT151012-like counts
- STROOPWAFEL importance sampling is active in both runs so that rare-merger yield comparisons are statistically meaningful

## Limitations

- The paper varies individual parameters independently; a full exploration of the multidimensional parameter space (e.g. alpha, fLBV) is deferred to future COMPAS studies
- Stellar rotation and tides are not included in the Fiducial model, and wind accretion by a companion is not accounted for
- The SSE single-star grid extends only to 50 solar masses and is extrapolated above that limit
- Merger rates are per formed population; conversion to an observable rate requires convolving metallicity-specific redshift-dependent star formation rate with the time delay distribution

## Evidence

- [discussion] LBV rate increase does not change total rate but reduces LVT151012-like mergers: "does not significantly change the total BBHs merger rate; nevertheless, the number of BBHs mergers similar to LVT151012 was reduced by a factor of ∼10"
- [discussion] One-at-a-time parameter variation relative to Fiducial: "here we follow the common approach273637 of varying individual parameters independently and assessing their impact relative to the Fiducial model."
- [discussion] Initial distributions used in the population synthesis: [evidence span withheld — anchor status: not_found]
- [discussion] Full parameter-space exploration deferred to future studies: "We leave a full exploration of this parameter space for future studies with COMPAS; here we follow the common approach273637 of varying individual parameters independently"
- [methods] STROOPWAFEL is COMPAS's importance sampling algorithm: "If you use COMPAS's importance sampling algorithm STROOPWAFEL, please cite"
