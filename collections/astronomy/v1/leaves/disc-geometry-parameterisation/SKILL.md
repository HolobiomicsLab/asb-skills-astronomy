---
name: disc-geometry-parameterisation
description: Use when modelling transits of circumsecondary disc (CSD) systems with
  azimuthally symmetric, non-uniform optical-depth profiles, and you need theoretical
  light-curve gradients for every possible disc geometry before rejecting configurations.
when_to_use_negative:
- The disc system is not azimuthally symmetric or has no non-uniform optical-depth
  profile — the parameterisation assumes azimuthally symmetric CSD structure
- You already have a single fixed disc configuration and only need a forward light-curve
  model, not an exhaustive geometry grid with per-configuration gradients
tools:
- name: BeyonCE
  role: Provides the disc-geometry parameterisation and light-curve gradient routines
    used to compute theoretical gradients for every ring crossing the star
  repo: github.com/dmvandamt/beyonce
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202245620
    title: Beyond Circular Eclipsers (BeyonCE) light curve modelling
  source_content_hash: sha256:5b01dd4ce39a9feea62078ad47a50975fa7da56c6ea340025723194e5e5d67e1
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
yes_rate: 0.8333333333333334
n_claims: 6
---

# disc-geometry-parameterisation

## Summary

Enumerates all geometries of an azimuthally symmetric circumsecondary disc (viewing orientation and ring radii) and computes the theoretical light-curve gradient for every ring crossing the stellar disc, using the BeyonCE package. This gives a per-configuration theoretical gradient that later rejection steps compare against the measured light-curve gradient.

## When to use

Use when modelling transits of circumsecondary disc (CSD) systems with azimuthally symmetric, non-uniform optical-depth profiles, and you need theoretical light-curve gradients for every possible disc geometry before rejecting configurations.

## When NOT to use

- The disc system is not azimuthally symmetric or has no non-uniform optical-depth profile — the parameterisation assumes azimuthally symmetric CSD structure
- You already have a single fixed disc configuration and only need a forward light-curve model, not an exhaustive geometry grid with per-configuration gradients

## Inputs

- Parameter grid of disc geometries: viewing orientations (inclination, projected tilt) and ring radii for an azimuthally symmetric disc
- Light-curve observation of a CSD transit event (e.g. J1407 or PDS 110) for which gradients will later be compared

## Outputs

- Table of theoretical light-curve gradients keyed by geometry parameters (orientation and ring radius), e.g. theoretical_gradient_table.csv
- Per-geometry gradient values covering every ring crossing the stellar disc

## How to apply

Install BeyonCE from github.com/dmvandamt/beyonce, which ships the disc-geometry parameterisation and light-curve gradient routines. Build a parameter grid enumerating all possible geometries of the azimuthally symmetric disc: viewing orientations (inclination and projected tilt) and ring radii spanning the disc. For every geometry on the grid, run the BeyonCE gradient routine to compute the theoretical light-curve gradient for each ring crossing the stellar disc. Assemble the gradients into a table keyed by orientation and ring radius (e.g. theoretical_gradient_table.csv). The rationale is that mapping all geometries up front allows a subsequent rejection step that discards configurations where the measured gradient exceeds the theoretical gradient for a given disc orientation.

## Related tools

- **BeyonCE** (Provides the disc-geometry parameterisation and light-curve gradient routines used to compute theoretical gradients for every ring crossing the star) — github.com/dmvandamt/beyonce

## Evaluation signals

- The gradient table covers every point on the geometry grid (all orientation × ring-radius combinations) with finite gradient values
- Computed gradients are consistent with the reference values produced by BeyonCE's own gradient routines (e.g. its example outputs)
- Every ring crossing the stellar disc has an associated theoretical gradient for each enumerated disc orientation
- No standalone quantitative acceptance criterion is reported in the paper, so consistency against BeyonCE's own example outputs is the practical check

## Limitations

- The paper reports no standalone quantitative acceptance criterion for this gradient-mapping step; validation relies on consistency with the package's own example outputs
- Applies to azimuthally symmetric, non-uniform optical-depth profiles; configurations outside this symmetry assumption are not covered
- This step only produces theoretical gradients — it does not itself reject configurations; rejection happens when the measured gradient exceeds the theoretical gradient for a given disc orientation

## Evidence

- [abstract] We mapped out all the possible geometries of a disc and calculated the gradients for rings crossing the star: "We mapped out all the possible geometries of a disc and calculated the gradients for rings crossing the star"
- [abstract] We then rejected those configurations where the measured gradient of the light curve is greater than the theoretical gradient from the given disc orientation: "We then rejected those configurations where the measured gradient of the light curve is greater than the theoretical gradient from the given disc orientation"
- [abstract] which reduces the parameter space encompassed by the transit of circumsec-ondary disc (CSD) systems with azimuthally symmetric, non-uniform optical-depth profiles: "which reduces the parameter space encompassed by the transit of circumsec-ondary disc (CSD) systems with azimuthally symmetric, non-uniform optical-depth profiles"
- [methods] BeyonCE documentation master file, created by sphinx-quickstart on Thu Apr 25 19:54:23 2024: "BeyonCE documentation master file, created by
   sphinx-quickstart on Thu Apr 25 19:54:23 2024"
