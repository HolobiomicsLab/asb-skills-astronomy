---
name: light-curve-gradient-measurement
description: Use when fitting a transit light curve attributed to a circumsecondary
  disc (e.g. the J1407 SuperWASP photometry or the PDS 110 eclipse photometry) and
  you need to reject disc size/orientation configurations whose theoretical ring gradient
  cannot exceed the measured light-curve gradient.
when_to_use_negative:
- The disc model does not assume azimuthally symmetric structure with a non-uniform
  optical-depth profile — the gradient-rejection criterion is derived for azimuthally
  symmetric CSD profiles
- The light curve lacks resolvable ingress/egress structure, so a measured light-curve
  gradient cannot be reliably determined from the photometry
- A full forward-model posterior fit (e.g. MCMC over all disc parameters) is required
  rather than a fast rejection of geometrically impossible configurations
edam_operation: http://edamontology.org/operation_2422
tools:
- name: BeyonCE
  role: Package that maps all possible disc geometries, calculates theoretical gradients
    for rings crossing the star, and performs the measured-vs-theoretical gradient
    rejection of configurations
  repo: https://github.com/dmvandamt/beyonce
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202245620
    title: Beyond Circular Eclipsers (BeyonCE) light curve modelling
  source_content_hash: sha256:23bc50e6ec07bd8e78d756f6e0e6e926ed338b3afbf3fb20579706cec1cdebc4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dmvandamt/beyonce
  tools:
  - BeyonCE
  edam_operation: http://edamontology.org/operation_2422
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# light-curve-gradient-measurement

## Summary

Measures the observed gradient of a circumsecondary disc (CSD) transit light curve and compares it against pre-computed theoretical gradients for rings of every disc size/orientation, so that geometries unable to reproduce the measured gradient can be rejected. This is the core rejection step of the BeyonCE method for shrinking the parameter space of discs with azimuthally symmetric, non-uniform optical-depth profiles.

## When to use

Use when fitting a transit light curve attributed to a circumsecondary disc (e.g. the J1407 SuperWASP photometry or the PDS 110 eclipse photometry) and you need to reject disc size/orientation configurations whose theoretical ring gradient cannot exceed the measured light-curve gradient.

## When NOT to use

- The disc model does not assume azimuthally symmetric structure with a non-uniform optical-depth profile — the gradient-rejection criterion is derived for azimuthally symmetric CSD profiles
- The light curve lacks resolvable ingress/egress structure, so a measured light-curve gradient cannot be reliably determined from the photometry
- A full forward-model posterior fit (e.g. MCMC over all disc parameters) is required rather than a fast rejection of geometrically impossible configurations

## Inputs

- Eclipse/transit photometry of a CSD host star (e.g. J1407 SuperWASP transit photometry, PDS 110 eclipse photometry)
- Pre-computed theoretical ring gradient map covering all disc size/orientation geometries in the enumerated grid
- Grid definition of disc sizes and orientations to evaluate

## Outputs

- Per-star rejection table over disc size and orientation with allowed vs. rejected labels (e.g. j1407_disc_rejection_grid.csv, pds110_disc_rejection_table.csv)
- Per-star counts of retained and rejected configurations, and rejected/retained fractions (e.g. parameter_space_reduction_summary.csv)

## How to apply

Load the eclipse/transit photometry into the BeyonCE package together with a pre-computed theoretical ring-gradient map that spans the full grid of disc sizes and orientations. Measure the observed gradient of the light curve directly from the photometry (the gradient of the flux drop during ingress/egress as rings cross the stellar disc). For each disc size/orientation configuration, retrieve the maximum theoretical gradient achievable by rings of that orientation crossing the star, and flag the configuration as rejected whenever the measured light-curve gradient is greater than the theoretical gradient for that orientation. The rationale is that a real ring can only produce flux changes up to its geometrically allowed maximum gradient, so any configuration whose ceiling falls below the measured gradient is physically inconsistent. Assemble the surviving allowed configurations and the rejected configurations into a single table over disc size and orientation (e.g. j1407_disc_rejection_grid.csv, pds110_disc_rejection_table.csv) and validate that the allowed region is a strict subset of the enumerated grid.

## Related tools

- **BeyonCE** (Package that maps all possible disc geometries, calculates theoretical gradients for rings crossing the star, and performs the measured-vs-theoretical gradient rejection of configurations) — https://github.com/dmvandamt/beyonce

## Evaluation signals

- The rejection step considerably reduces the parameter space of discs that contain an azimuthally symmetric structure — the allowed region is a strict subset of the enumerated size/orientation grid
- Every row labelled rejected corresponds to a configuration whose measured light-curve gradient exceeds the theoretical gradient for that orientation, per the BeyonCE rejection criterion
- The rejection table contains an entry for every evaluated size/orientation configuration (complete grid coverage)
- Retained and rejected fractions per star sum to 1, and the summary table has one row per star with columns: star, total configurations, retained fraction, rejected fraction

## Limitations

- The method constrains only configurations consistent with azimuthally symmetric, non-uniform optical-depth profiles; asymmetric disc structures are outside the rejection criterion
- Rejection is an inequality test (measured gradient > theoretical gradient), so configurations whose theoretical gradient merely exceeds the measured value are retained, not confirmed

## Evidence

- [abstract] Gradient-based rejection criterion: "We then rejected those configurations where the measured gradient of the light curve is greater than the theoretical gradient from the given disc orientation"
- [abstract] Theoretical gradient computation: "We mapped out all the possible geometries of a disc and calculated the gradients for rings crossing the star"
- [abstract] Parameter-space reduction effect: "demonstrate its effectiveness in considerably reducing the parameter space of discs that contain an azimuthally symmetric structure"
- [abstract] Scope of applicable systems: "which reduces the parameter space encompassed by the transit of circumsec-ondary disc (CSD) systems with azimuthally symmetric, non-uniform optical-depth profiles"
