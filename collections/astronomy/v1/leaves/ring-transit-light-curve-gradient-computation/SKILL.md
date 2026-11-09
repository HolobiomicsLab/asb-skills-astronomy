---
name: ring-transit-light-curve-gradient-computation
description: Use when modelling transits of circumsecondary disc systems (e.g. J1407,
  PDS 110) where all possible disc geometries must be enumerated and a theoretical
  light-curve gradient per ring crossing the star is needed prior to rejecting configurations.
when_to_use_negative:
- The system under study is a circular, opaque eclipsing body rather than a circumsecondary
  disc with an azimuthally symmetric, non-uniform optical-depth profile
- You only need a single best-fit disc geometry rather than an exhaustive enumeration
  of all possible geometries with per-ring gradients
tools:
- name: BeyonCE
  role: Provides the disc-geometry parameterisation and the light-curve gradient routines
    used to compute theoretical gradients for rings crossing the star
  repo: github.com/dmvandamt/beyonce
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202245620
    title: Beyond Circular Eclipsers (BeyonCE) light curve modelling
  source_content_hash: sha256:42b86b744505abd2335f9f835cba1c9699ec73359437bdf264cd101a712a1d83
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
yes_rate: 0.8571428571428571
n_claims: 7
---

# ring-transit-light-curve-gradient-computation

## Summary

Computes theoretical light-curve gradients for rings of an azimuthally symmetric circumsecondary disc (CSD) crossing the stellar disc, using the BeyonCE package's disc-geometry parameterisation. This pre-computation of gradients across all disc configurations is what enables BeyonCE to reduce the CSD transit parameter space before any rejection step.

## When to use

Use when modelling transits of circumsecondary disc systems (e.g. J1407, PDS 110) where all possible disc geometries must be enumerated and a theoretical light-curve gradient per ring crossing the star is needed prior to rejecting configurations.

## When NOT to use

- The system under study is a circular, opaque eclipsing body rather than a circumsecondary disc with an azimuthally symmetric, non-uniform optical-depth profile
- You only need a single best-fit disc geometry rather than an exhaustive enumeration of all possible geometries with per-ring gradients

## Inputs

- Parameter grid of azimuthally symmetric disc geometries (viewing orientations: inclination and projected tilt; ring radii spanning the disc)
- Light curve of a suspected CSD transit event (e.g. J1407, PDS 110) for which measured ring-crossing gradients are to be compared

## Outputs

- Table of theoretical light-curve gradients keyed by disc geometry parameters (orientation and ring radius), e.g. theoretical_gradient_table.csv

## How to apply

Install the BeyonCE package from github.com/dmvandamt/beyonce, which ships the disc-geometry parameterisation and light-curve gradient routines for azimuthally symmetric, non-uniform optical-depth profiles. Build a parameter grid enumerating all possible disc geometries: viewing orientations (inclination and projected tilt) and ring radii spanning the disc. For every geometry on the grid, run the BeyonCE gradient routine to compute the theoretical light-curve gradient for each ring crossing the stellar disc. Assemble the gradients into a table keyed by geometry parameters (orientation, ring radius), e.g. theoretical_gradient_table.csv. These theoretical gradients are subsequently used as the acceptance bound: configurations where the measured light-curve gradient exceeds the theoretical gradient for a given disc orientation are rejected, which is how BeyonCE reduces the parameter space.

## Related tools

- **BeyonCE** (Provides the disc-geometry parameterisation and the light-curve gradient routines used to compute theoretical gradients for rings crossing the star) — github.com/dmvandamt/beyonce

## Examples

```
pip install git+https://github.com/dmvandamt/beyonce
```

## Evaluation signals

- The gradient table covers every point on the geometry grid (every orientation and ring radius combination) with finite gradient values
- Computed gradients are consistent with reference values produced by the BeyonCE package's own gradient routines (e.g. its example outputs); the paper reports no standalone quantitative acceptance criterion
- Downstream rejection behaves as described: configurations where the measured gradient of the light curve exceeds the theoretical gradient for the given disc orientation are rejected

## Limitations

- Restricted to azimuthally symmetric, non-uniform optical-depth disc profiles; discs lacking an azimuthally symmetric structure are outside the method's demonstrated scope
- No standalone quantitative acceptance criterion is reported in the paper, so validation relies on consistency with the BeyonCE package's own reference outputs

## Evidence

- [abstract] We mapped out all the possible geometries of a disc and calculated the gradients for rings crossing the star: "We mapped out all the possible geometries of a disc and calculated the gradients for rings crossing the star"
- [abstract] We then rejected those configurations where the measured gradient of the light curve is greater than the theoretical gradient from the given disc orientation: "We then rejected those configurations where the measured gradient of the light curve is greater than the theoretical gradient from the given disc orientation"
- [abstract] reduces the parameter space encompassed by the transit of circumsecondary disc (CSD) systems with azimuthally symmetric, non-uniform optical-depth profiles: "reduces the parameter space encompassed by the transit of circumsec-ondary disc (CSD) systems with azimuthally symmetric, non-uniform optical-depth profiles"
