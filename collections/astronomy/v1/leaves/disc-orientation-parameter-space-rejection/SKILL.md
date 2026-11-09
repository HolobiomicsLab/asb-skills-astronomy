---
name: disc-orientation-parameter-space-rejection
description: Use when modelling a transit light curve attributed to a circumsecondary
  disc and you need to constrain disc size and orientation by rejecting azimuthally
  symmetric disc geometries that cannot reproduce the observed light-curve gradient.
when_to_use_negative:
- The transit is better explained by a companion body (e.g. planet or stellar companion)
  rather than an azimuthally symmetric circumsecondary disc, since the rejection criterion
  is defined only for discs with azimuthally symmetric, non-uniform optical-depth
  profiles
- No measured light-curve gradient can be reliably extracted from the photometry,
  making the measured-vs-theoretical gradient comparison impossible
- A full forward-model fit of the disc light curve is required rather than rejection/exclusion
  of geometries — BeyonCE reduces parameter space but does not itself fit a unique
  best-fit disc model
tools:
- name: BeyonCE
  role: 'Implements the gradient-rejection method: maps possible disc geometries,
    calculates theoretical gradients for rings crossing the star, and rejects configurations
    where the measured light-curve gradient exceeds the theoretical gradient for the
    given disc orientation'
  repo: https://github.com/dmvandamt/beyonce
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202245620
    title: Beyond Circular Eclipsers (BeyonCE) light curve modelling
  source_content_hash: sha256:f8e59a2d7cc90a660f5d1c995dae5852fadc79c91814dfb1166be5323fc5ddd1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dmvandamt/beyonce
  tools:
  - BeyonCE
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# disc-orientation-parameter-space-rejection

## Summary

Gradient-based rejection of circumsecondary disc (CSD) geometries using the BeyonCE package: enumerate disc size/orientation configurations, compute theoretical ring-crossing gradients, and reject configurations whose theoretical gradient cannot reproduce the measured light-curve gradient, as demonstrated on the J1407 and PDS 110 transits.

## When to use

Use when modelling a transit light curve attributed to a circumsecondary disc and you need to constrain disc size and orientation by rejecting azimuthally symmetric disc geometries that cannot reproduce the observed light-curve gradient.

## When NOT to use

- The transit is better explained by a companion body (e.g. planet or stellar companion) rather than an azimuthally symmetric circumsecondary disc, since the rejection criterion is defined only for discs with azimuthally symmetric, non-uniform optical-depth profiles
- No measured light-curve gradient can be reliably extracted from the photometry, making the measured-vs-theoretical gradient comparison impossible
- A full forward-model fit of the disc light curve is required rather than rejection/exclusion of geometries — BeyonCE reduces parameter space but does not itself fit a unique best-fit disc model

## Inputs

- Transit/eclipse photometry of the target star (e.g. J1407 SuperWASP light curve, PDS 110 eclipse photometry)
- Pre-computed theoretical ring gradient map covering all disc size/orientation configurations (from BeyonCE)
- Grid of enumerated disc geometries (disc size and orientation configurations)

## Outputs

- Per-star parameter-space rejection table with allowed and rejected configurations spanning disc size and orientation (e.g. j1407_disc_rejection_grid.csv, pds110_disc_rejection_table.csv)
- Per-star rejected and retained fractions summarizing the parameter-space reduction (e.g. parameter_space_reduction_summary.csv with columns: star, total configurations, retained fraction, rejected fraction)

## How to apply

Load the target transit photometry (e.g. SuperWASP photometry for J1407 or eclipse photometry for PDS 110) and, using BeyonCE, map out all possible disc geometries, calculating the theoretical gradients for rings of each size/orientation crossing the stellar disc. Measure the observed gradient of the light curve directly from the photometry. For each configuration in the grid, apply the rejection criterion: flag a configuration as rejected whenever the measured light-curve gradient is greater than the maximum theoretical gradient achievable by rings of that orientation. Assemble surviving (allowed) and rejected configurations into a per-star parameter-space table (e.g. j1407_disc_rejection_grid.csv, pds110_disc_rejection_table.csv), with entries for each evaluated size/orientation configuration. The rationale is that a disc containing an azimuthally symmetric structure cannot produce light-curve gradients steeper than the theoretical maximum for its orientation, so exceeding it excludes the geometry outright.

## Related tools

- **BeyonCE** (Implements the gradient-rejection method: maps possible disc geometries, calculates theoretical gradients for rings crossing the star, and rejects configurations where the measured light-curve gradient exceeds the theoretical gradient for the given disc orientation) — https://github.com/dmvandamt/beyonce

## Evaluation signals

- The rejection step considerably reduces the parameter space: the set of allowed configurations is a strict subset of the enumerated size/orientation grid
- Every row labelled rejected corresponds to a configuration whose measured gradient exceeds the theoretical gradient for that orientation, per the BeyonCE rejection criterion
- The output table contains an entry for every evaluated size/orientation configuration (allowed plus rejected counts equal the total grid size)
- Summary fractions are consistent: retained fraction + rejected fraction = 1 per star
- Application to J1407 and PDS 110 reproduces the reported considerable reduction in the parameter space of discs containing an azimuthally symmetric structure

## Limitations

- The rejection criterion applies only to discs with azimuthally symmetric, non-uniform optical-depth profiles; other disc structures are outside its scope
- The method reduces (rejects parts of) the parameter space but does not by itself determine a unique disc size and orientation
- Rejection depends on the maximum theoretical gradient achievable by rings of a given orientation; errors in the measured light-curve gradient propagate directly into the allowed/rejected classification

## Evidence

- [abstract] We mapped out all the possible geometries of a disc and calculated the gradients for rings crossing the star: "We mapped out all the possible geometries of a disc and calculated the gradients for rings crossing the star"
- [abstract] We then rejected those configurations where the measured gradient of the light curve is greater than the theoretical gradient from the given disc orientation: "We then rejected those configurations where the measured gradient of the light curve is greater than the theoretical gradient from the given disc orientation"
- [abstract] reduces the parameter space encompassed by the transit of circumsecondary disc (CSD) systems: "which reduces the parameter space encompassed by the transit of circumsec-ondary disc (CSD) systems with azimuthally symmetric, non-uniform optical-depth profiles"
- [abstract] We used the code to analyse the light curves seen towards J1407 and PDS 110, attributed to CSD transits: "We used the code to analyse the light curves seen towards J1407 and PDS 110, attributed to CSD transits"
