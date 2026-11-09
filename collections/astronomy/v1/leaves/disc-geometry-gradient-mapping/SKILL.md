---
name: disc-geometry-gradient-mapping
description: Use when fitting a disc transit light curve attributed to a circumsecondary
  disc and you need to reduce the parameter space of disc size/orientation configurations
  by gradient-based rejection.
when_to_use_negative:
- The disc system does not have an azimuthally symmetric, non-uniform optical-depth
  profile — BeyonCE's rejection logic is defined for azimuthally symmetric disc structures
- The transit is caused by a bound circular companion or another eclipsing body rather
  than a circumsecondary disc transit (the method targets CSD transits like J1407
  and PDS 110)
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: BeyonCE
  role: Package that maps out all possible disc geometries, calculates gradients for
    rings crossing the star, and rejects configurations where the measured light-curve
    gradient exceeds the theoretical gradient
  repo: github.com/dmvandamt/beyonce
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202245620
    title: Beyond Circular Eclipsers (BeyonCE) light curve modelling
  source_content_hash: sha256:bb55647feae93c9c337dd038fb87fd92dc0aaa2795a7f8161b2ac4264985b2f5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BeyonCE
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# disc-geometry-gradient-mapping

## Summary

Maps all possible circumsecondary disc geometries and computes the theoretical light-curve gradients for rings crossing the stellar disc, then rejects configurations whose measured light-curve gradient exceeds the theoretical gradient for that disc orientation. Implemented in the BeyonCE package and demonstrated on the J1407 and PDS 110 CSD transit light curves.

## When to use

Use when fitting a disc transit light curve attributed to a circumsecondary disc and you need to reduce the parameter space of disc size/orientation configurations by gradient-based rejection.

## When NOT to use

- The disc system does not have an azimuthally symmetric, non-uniform optical-depth profile — BeyonCE's rejection logic is defined for azimuthally symmetric disc structures
- The transit is caused by a bound circular companion or another eclipsing body rather than a circumsecondary disc transit (the method targets CSD transits like J1407 and PDS 110)

## Inputs

- Eclipse/transit photometry light curve of a CSD system (e.g. PDS 110 eclipse photometry)
- Pre-computed theoretical ring gradient map covering all disc geometries (size and orientation configurations)

## Outputs

- Disc rejection table (e.g. pds110_disc_rejection_table.csv) with rows for each evaluated size/orientation configuration labelled allowed or rejected
- Reduced parameter space of surviving disc geometries

## How to apply

Load the eclipse photometry (e.g. the PDS 110 light curve) into the BeyonCE package together with a pre-computed theoretical gradient map covering all disc geometries. For each disc size/orientation configuration, retrieve the maximum theoretical gradient achievable by rings of that orientation crossing the stellar disc. Flag a configuration as rejected whenever the measured gradient of the light curve is greater than the theoretical gradient from the given disc orientation, following the BeyonCE rejection criterion. Assemble surviving allowed configurations and rejected configurations into a single table spanning disc size and orientation (e.g. pds110_disc_rejection_table.csv). This considerably reduces the parameter space of discs with azimuthally symmetric, non-uniform optical-depth profiles.

## Related tools

- **BeyonCE** (Package that maps out all possible disc geometries, calculates gradients for rings crossing the star, and rejects configurations where the measured light-curve gradient exceeds the theoretical gradient) — github.com/dmvandamt/beyonce

## Evaluation signals

- The rejection table (e.g. pds110_disc_rejection_table.csv) exists and contains an entry for each evaluated size/orientation configuration
- Every row labelled rejected corresponds to a configuration whose measured gradient exceeds the theoretical gradient for that orientation, per the BeyonCE rejection criterion
- The number of surviving allowed configurations is smaller than the full input geometry set, demonstrating parameter-space reduction
- The theoretical gradients used per orientation match the pre-computed gradient map for rings crossing the stellar disc

## Limitations

- The method assumes azimuthally symmetric, non-uniform optical-depth profiles; discs lacking an azimuthally symmetric structure fall outside its scope
- Rejection is one-sided: a configuration is rejected only when the measured gradient exceeds the theoretical gradient, so configurations with compatible gradients remain unconstrained by this test
- Requires a pre-computed theoretical gradient map covering all disc geometries; results are limited by the geometry grid sampled in that map

## Evidence

- [abstract] Map geometries and calculate gradients: "We mapped out all the possible geometries of a disc and calculated the gradients for rings crossing the star"
- [abstract] Rejection criterion: "We then rejected those configurations where the measured gradient of the light curve is greater than the theoretical gradient from the given disc orientation"
- [abstract] PDS 110 and J1407 application: "We used the code to analyse the light curves seen towards J1407 and PDS 110, attributed to CSD transits"
- [abstract] Scope assumption: "which reduces the parameter space encompassed by the transit of circumsec-ondary disc (CSD) systems with azimuthally symmetric, non-uniform optical-depth profiles"
