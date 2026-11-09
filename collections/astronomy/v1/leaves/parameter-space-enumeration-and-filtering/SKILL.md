---
name: parameter-space-enumeration-and-filtering
description: Use when you have time-resolved transit photometry attributed to a circumsecondary
  disc (e.g. J1407, PDS 110) and need to constrain disc size/orientation by rejecting
  geometrically impossible configurations via light-curve gradient comparison.
when_to_use_negative:
- The transit is not attributable to a circumsecondary disc or azimuthally symmetric
  disc structure (BeyonCE's rejection relies on the azimuthal symmetry assumption)
- The photometry is too sparse or noisy for a reliable measured light-curve gradient
  to be extracted, since the rejection hinges on comparing observed vs theoretical
  gradients
edam_operation: http://edamontology.org/operation_2426
tools:
- name: BeyonCE
  role: Maps all possible disc geometries, calculates theoretical gradients for rings
    crossing the star per grid point, and performs the rejection of configurations
    whose measured light-curve gradient exceeds the theoretical gradient
  repo: github.com/dmvandamt/beyonce
- name: Sphinx
  role: Generates the BeyonCE documentation used to understand and apply the gradient-based
    rejection method
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202245620
    title: Beyond Circular Eclipsers (BeyonCE) light curve modelling
  source_content_hash: sha256:65d529e2caeb7a6f5ac19cf19e0abb544e7ce15ddbc63550dd6d12f91bfddc41
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BeyonCE
  - Sphinx
  edam_operation: http://edamontology.org/operation_2426
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# parameter-space-enumeration-and-filtering

## Summary

Enumerate all possible circumsecondary disc (CSD) transit geometries, compute theoretical light-curve gradients for rings crossing the star at each configuration with BeyonCE, and reject configurations whose measured light-curve gradient exceeds the theoretical maximum — reducing the parameter space to azimuthally symmetric disc structures.

## When to use

Use when you have time-resolved transit photometry attributed to a circumsecondary disc (e.g. J1407, PDS 110) and need to constrain disc size/orientation by rejecting geometrically impossible configurations via light-curve gradient comparison.

## When NOT to use

- The transit is not attributable to a circumsecondary disc or azimuthally symmetric disc structure (BeyonCE's rejection relies on the azimuthal symmetry assumption)
- The photometry is too sparse or noisy for a reliable measured light-curve gradient to be extracted, since the rejection hinges on comparing observed vs theoretical gradients

## Inputs

- Time-series transit photometry of the target system (e.g. SuperWASP photometry of the J1407 circumsecondary disc transit)
- Grid of candidate disc geometries (size and orientation parameter space) to enumerate

## Outputs

- Parameter-space table of allowed vs rejected disc configurations (e.g. j1407_disc_rejection_grid.csv)
- Reduced set of disc geometries consistent with azimuthally symmetric, non-uniform optical-depth profiles

## How to apply

Load the transit photometry (e.g. the J1407 SuperWASP light curve) and use BeyonCE to map out all possible disc geometries, calculating the theoretical gradients for rings crossing the star at each disc size/orientation in the grid. Measure the observed gradient of the light curve directly from the photometry. Reject those configurations where the measured gradient of the light curve is greater than the theoretical gradient from the given disc orientation, since azimuthally symmetric, non-uniform optical-depth profiles cannot produce steeper gradients than the geometry allows. Assemble surviving (allowed) and rejected configurations into a parameter-space table (e.g. j1407_disc_rejection_grid.csv) and validate that the allowed region is a strict subset of the enumerated grid, confirming the rejection considerably reduces the parameter space.

## Related tools

- **BeyonCE** (Maps all possible disc geometries, calculates theoretical gradients for rings crossing the star per grid point, and performs the rejection of configurations whose measured light-curve gradient exceeds the theoretical gradient) — github.com/dmvandamt/beyonce
- **Sphinx** (Generates the BeyonCE documentation used to understand and apply the gradient-based rejection method)

## Examples

```
from beyonce import BeyonCE; grid = BeyonCE.map_disc_geometries(j1407_photometry); allowed = grid.reject(measured_gradient > grid.theoretical_gradient); allowed.to_csv('j1407_disc_rejection_grid.csv')
```

## Evaluation signals

- The rejection step considerably reduces the parameter space: the allowed region is a strict subset of the enumerated disc grid
- Every rejected configuration has a measured light-curve gradient strictly greater than the theoretical gradient for that disc orientation
- The output table (e.g. j1407_disc_rejection_grid.csv) contains both surviving (allowed) and rejected configurations covering the full enumerated grid
- Applied to J1407, the surviving configurations correspond to discs containing an azimuthally symmetric structure

## Limitations

- The method only constrains azimuthally symmetric, non-uniform optical-depth profiles; other disc structures are not handled
- It is a rejection (parameter-space reduction) technique, not a unique disc solution — multiple configurations may survive
- Requires the measured light-curve gradient to be reliably measurable from the photometry for the rejection criterion to be valid

## Evidence

- [abstract] We mapped out all the possible geometries of a disc and calculated the gradients for rings crossing the star: "We mapped out all the possible geometries of a disc and calculated the gradients for rings crossing the star"
- [abstract] rejection criterion: "We then rejected those configurations where the measured gradient of the light curve is greater than the theoretical gradient from the given disc orientation"
- [abstract] reduction of parameter space: "demonstrate its effectiveness in considerably reducing the parameter space of discs that contain an azimuthally symmetric structure"
- [abstract] target systems: "We used the code to analyse the light curves seen towards J1407 and PDS 110, attributed to CSD transits"
