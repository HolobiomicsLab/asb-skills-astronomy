---
name: parameter-grid-enumeration
description: Use when modelling a transit light curve attributed to a circumsecondary
  disc (e.g. J1407 or PDS 110) and you need the full set of disc geometries with their
  theoretical light-curve gradients before applying any rejection step.
when_to_use_negative:
- The occulter is not an azimuthally symmetric disc — BeyonCE's parameterisation and
  gradient reduction assume azimuthally symmetric, non-uniform optical-depth profiles
- The analysis goal is to fit measured light curves directly rather than to pre-compute
  theoretical gradients for a configuration rejection step
edam_operation: http://edamontology.org/operation_2428
tools:
- name: BeyonCE
  role: Provides the disc-geometry parameterisation and the light-curve gradient routines
    evaluated at every point of the geometry grid
  repo: github.com/dmvandamt/beyonce
- name: Sphinx
  role: Generates the BeyonCE documentation describing the gradient routines
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202245620
    title: Beyond Circular Eclipsers (BeyonCE) light curve modelling
  source_content_hash: sha256:29b254e5b4915c744c331b4d8b585ac10521c802ce442017fa80fa7af9314a12
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BeyonCE
  - Sphinx
  edam_operation: http://edamontology.org/operation_2428
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# parameter-grid-enumeration

## Summary

Enumerates every allowed configuration of an azimuthally symmetric circumsecondary disc (viewing orientations and ring radii) and computes the theoretical light-curve gradient for each ring crossing the stellar disc using the BeyonCE package, so that physically impossible configurations can be rejected prior to fitting.

## When to use

Use when modelling a transit light curve attributed to a circumsecondary disc (e.g. J1407 or PDS 110) and you need the full set of disc geometries with their theoretical light-curve gradients before applying any rejection step.

## When NOT to use

- The occulter is not an azimuthally symmetric disc — BeyonCE's parameterisation and gradient reduction assume azimuthally symmetric, non-uniform optical-depth profiles
- The analysis goal is to fit measured light curves directly rather than to pre-compute theoretical gradients for a configuration rejection step

## Inputs

- Disc geometry parameter grid: viewing orientations (inclination and projected tilt) and ring radii spanning the azimuthally symmetric disc
- Stellar and transit parameters needed by the BeyonCE light-curve gradient routines

## Outputs

- Table of theoretical light-curve gradients keyed by geometry parameters (orientation and ring radius), e.g. theoretical_gradient_table.csv
- Complete enumeration of possible disc geometries for downstream rejection of configurations

## How to apply

Install and import the BeyonCE package from github.com/dmvandamt/beyonce, which ships the disc-geometry parameterisation and light-curve gradient routines. Build a parameter grid enumerating all possible geometries of the azimuthally symmetric disc: viewing orientations (inclination and projected tilt) and ring radii spanning the disc. For every geometry on the grid, run the BeyonCE gradient routine to compute the theoretical light-curve gradient for each ring crossing the stellar disc. Assemble the computed gradients into a table keyed by geometry parameters (orientation and ring radius), e.g. saved as theoretical_gradient_table.csv. Rationale: the paper computes this theoretical gradient for each disc configuration 'prior to any rejection step', so the grid must be complete before downstream filtering of configurations where the measured gradient exceeds the theoretical gradient for a given disc orientation.

## Related tools

- **BeyonCE** (Provides the disc-geometry parameterisation and the light-curve gradient routines evaluated at every point of the geometry grid) — github.com/dmvandamt/beyonce
- **Sphinx** (Generates the BeyonCE documentation describing the gradient routines)

## Examples

```
pip install git+https://github.com/dmvandamt/beyonce.git
```

## Evaluation signals

- The gradient table covers every point on the geometry grid (all orientation and ring-radius combinations) with finite gradient values
- Computed gradients are consistent with the reference values produced by the BeyonCE package's own gradient routines (e.g. its example outputs)
- The table is keyed by both geometry parameters: viewing orientation and ring radius
- Every disc configuration on the grid has a theoretical gradient recorded before any rejection step is applied

## Limitations

- The paper reports no standalone quantitative acceptance criterion for validating the gradient table; consistency with the package's own example outputs is the only check described
- Restricted to discs with azimuthally symmetric, non-uniform optical-depth profiles — non-symmetric disc structures fall outside the parameterisation
- The gradient enumeration is only the first stage; it does not itself reject configurations — that requires comparing against the measured light-curve gradient

## Evidence

- [abstract] BeyonCE works by mapping out all the possible geometries of a disc and calculating the light-curve gradients for rings crossing the star, providing a theoretical gradient for each disc configuration prior to any rejection step.: "We mapped out all the possible geometries of a disc and calculated the gradients for rings crossing the star"
- [abstract] BeyonCE targets the reduced parameter space of circumsecondary disc transits.: "which reduces the parameter space encompassed by the transit of circumsec-ondary disc (CSD) systems with azimuthally symmetric, non-uniform optical-depth profiles"
- [abstract] Downstream rejection uses the enumerated theoretical gradients.: "We then rejected those configurations where the measured gradient of the light curve is greater than the theoretical gradient from the given disc orientation"
