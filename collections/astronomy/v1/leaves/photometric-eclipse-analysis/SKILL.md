---
name: photometric-eclipse-analysis
description: Use when a transit/eclipse light curve is attributed to a circumsecondary
  disc (e.g. J1407 SuperWASP photometry or PDS 110 eclipse photometry) and you need
  to constrain the disc's size/orientation parameter space via gradient-based rejection
  of azimuthally symmetric disc geometries.
when_to_use_negative:
- The light curve is not attributed to a circumsecondary disc transit (the method
  was demonstrated specifically on the J1407 and PDS 110 CSD transits)
- The hypothesised disc structure is not azimuthally symmetric — BeyonCE constrains
  discs with azimuthally symmetric, non-uniform optical-depth profiles
- No theoretical gradient map for the disc geometry grid is available, so configurations
  cannot be compared against theoretical ring gradients
tools:
- name: BeyonCE
  role: 'Core package: maps all possible disc geometries, computes theoretical gradients
    for rings crossing the star, and performs the gradient-based rejection of configurations'
  repo: github.com/dmvandamt/beyonce
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202245620
    title: Beyond Circular Eclipsers (BeyonCE) light curve modelling
  source_content_hash: sha256:abe177b8e1947b3d061b65b027d74a7d81d9ab2c38479585e289a10a838fb491
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BeyonCE
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# photometric-eclipse-analysis

## Summary

A gradient-based rejection method, implemented in the BeyonCE package, that constrains the size and orientation of circumsecondary discs (CSDs) from eclipse photometry by rejecting disc geometries whose theoretical ring gradients cannot reproduce the measured light-curve gradient. It was demonstrated on the J1407 and PDS 110 transits.

## When to use

Use when a transit/eclipse light curve is attributed to a circumsecondary disc (e.g. J1407 SuperWASP photometry or PDS 110 eclipse photometry) and you need to constrain the disc's size/orientation parameter space via gradient-based rejection of azimuthally symmetric disc geometries.

## When NOT to use

- The light curve is not attributed to a circumsecondary disc transit (the method was demonstrated specifically on the J1407 and PDS 110 CSD transits)
- The hypothesised disc structure is not azimuthally symmetric — BeyonCE constrains discs with azimuthally symmetric, non-uniform optical-depth profiles
- No theoretical gradient map for the disc geometry grid is available, so configurations cannot be compared against theoretical ring gradients

## Inputs

- Eclipse/transit photometry light curve attributed to a circumsecondary disc transit (e.g. J1407 SuperWASP transit photometry, PDS 110 eclipse photometry)
- Pre-computed theoretical ring gradient map covering the grid of disc sizes and orientations

## Outputs

- Parameter-space table of allowed (surviving) and rejected disc configurations spanning disc size and orientation, e.g. j1407_disc_rejection_grid.csv
- Disc rejection table per system, e.g. pds110_disc_rejection_table.csv, with each row labelled allowed or rejected

## How to apply

Load the eclipse photometry (e.g. the J1407 SuperWASP light curve or PDS 110 eclipse photometry) into BeyonCE together with the pre-computed theoretical gradient map covering all disc geometries. Map out all possible disc geometries and calculate the theoretical gradients for rings of each size/orientation crossing the stellar disc. Measure the observed gradient of the light curve from the photometry, then reject every configuration where the measured gradient of the light curve is greater than the theoretical gradient for that disc orientation. Assemble the surviving (allowed) and rejected configurations into a parameter-space table spanning disc size and orientation and write it to CSV (e.g. j1407_disc_rejection_grid.csv, pds110_disc_rejection_table.csv). The rationale is that azimuthally symmetric, non-uniform optical-depth discs impose a maximum achievable gradient, so observed gradients steeper than this maximum physically rule out that geometry.

## Related tools

- **BeyonCE** (Core package: maps all possible disc geometries, computes theoretical gradients for rings crossing the star, and performs the gradient-based rejection of configurations) — github.com/dmvandamt/beyonce

## Evaluation signals

- The output table (e.g. pds110_disc_rejection_table.csv or j1407_disc_rejection_grid.csv) exists and contains an entry for every evaluated size/orientation configuration in the grid
- Every row labelled rejected corresponds to a configuration whose measured light-curve gradient exceeds the theoretical gradient for that disc orientation (the BeyonCE rejection criterion)
- The rejection step considerably reduces the parameter space of discs containing an azimuthally symmetric structure, i.e. the allowed region is a strict subset of the enumerated grid
- Both allowed and rejected configurations are present in the assembled table, spanning the disc size and orientation dimensions

## Limitations

- The method only constrains discs with azimuthally symmetric, non-uniform optical-depth profiles; asymmetric structures are not captured by the gradient-rejection criterion
- Rejection depends on a pre-computed theoretical gradient map covering all disc geometries; without it no configuration can be evaluated
- The criterion is one-sided (reject when measured gradient > theoretical gradient), so configurations whose theoretical gradients exceed the measured gradient remain allowed and the parameter space is reduced, not uniquely determined

## Evidence

- [abstract] BeyonCE reduces parameter space via gradient rejection: [evidence span withheld — anchor status: not_found]
- [abstract] Workflow: map geometries and compute ring gradients: "We mapped out all the possible geometries of a disc and calculated the gradients for rings crossing the star"
- [abstract] Rejection criterion: measured vs theoretical gradient: "We then rejected those configurations where the measured gradient of the light curve is greater than the theoretical gradient from the given disc orientation"
- [abstract] Demonstrated on J1407 and PDS 110 CSD transits: "We used the code to analyse the light curves seen towards J1407 and PDS 110, attributed to CSD transits"
- [abstract] Effectiveness: considerable parameter-space reduction: "demonstrate its effectiveness in considerably reducing the parameter space of discs that contain an azimuthally symmetric structure"
