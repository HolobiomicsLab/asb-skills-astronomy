---
name: schirmer-aperture-mass-filter-convolution
description: Use when a galaxy positions+ellipticities shear catalog must be turned
  into an aperture mass map for cluster detection, and an NFW-motivated radial filter
  (Schirmer 2007) with exponential cut-offs is appropriate.
when_to_use_negative:
- The analysis goal is the 2-pt angular correlation function — the ccogs angular_correlation
  package (with a chosen estimator and a flat random galaxy distribution) is the appropriate
  tool instead.
- No shear catalog (positions and elliptities of lensed galaxies) is available; the
  aperture mass map cannot be computed from random or non-lensing point data.
- The desired filter profile is not NFW-like; the aperture_mass package hard-codes
  the Schirmer et al (2007) filter rather than offering arbitrary filter choices.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: aperture_mass
  role: Computes the aperture mass map for the given dataset by convolving the Schirmer
    (2007) NFW filter with tangential shear, on the GPU
  repo: https://github.com/djbard/ccogs
- name: ccogs
  role: Suite of GPU cosmology packages in which the aperture_mass package is distributed
  repo: https://github.com/djbard/ccogs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2012.11.001
    title: Cosmological calculations on the GPU
  source_content_hash: sha256:f19b2dfe1de05d1468c5c205ef03e049805bab510d81ee992b376c311ddde1a4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/djbard/ccogs
  tools:
  - aperture_mass
  - ccogs
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# schirmer-aperture-mass-filter-convolution

## Summary

Computes a 2-D aperture mass map from a weak-lensing shear catalog by convolving a Schirmer et al (2007) filter — an NFW profile with exponential cut-offs at zero and large radii — with the tangential shear field, using the GPU-based aperture_mass package of the ccogs suite.

## When to use

Use when a galaxy positions+ellipticities shear catalog must be turned into an aperture mass map for cluster detection, and an NFW-motivated radial filter (Schirmer 2007) with exponential cut-offs is appropriate.

## When NOT to use

- The analysis goal is the 2-pt angular correlation function — the ccogs angular_correlation package (with a chosen estimator and a flat random galaxy distribution) is the appropriate tool instead.
- No shear catalog (positions and elliptities of lensed galaxies) is available; the aperture mass map cannot be computed from random or non-lensing point data.
- The desired filter profile is not NFW-like; the aperture_mass package hard-codes the Schirmer et al (2007) filter rather than offering arbitrary filter choices.

## Inputs

- Weak-lensing galaxy shear catalog with positions and ellipticities
- Grid of radii / map pixel grid defining the aperture positions

## Outputs

- 2-D aperture mass map (aperture_mass_map.fits, a readable FITS image with expected map dimensions)

## How to apply

Load the weak-lensing galaxy catalog (sky positions and ellipticities) into the aperture_mass package from the ccogs suite. Build the Schirmer et al (2007) filter — an NFW profile with exponential cut-offs at zero and large radii — on a grid of radii centered at each map pixel. Convolve this filter with the tangential shear field to evaluate the aperture mass at each grid position; the convolution kernel is designed to run on the GPU. Assemble the per-position values into a 2-D mass map and write it to aperture_mass_map.fits. Rationale: the NFW shape with cut-offs matches the expected shear profile of dark matter halos, so peaks in the map indicate mass concentrations.

## Related tools

- **aperture_mass** (Computes the aperture mass map for the given dataset by convolving the Schirmer (2007) NFW filter with tangential shear, on the GPU) — https://github.com/djbard/ccogs
- **ccogs** (Suite of GPU cosmology packages in which the aperture_mass package is distributed) — https://github.com/djbard/ccogs

## Evaluation signals

- aperture_mass_map.fits exists in the output directory
- aperture_mass_map.fits is a readable FITS image with the expected map dimensions for the surveyed field
- Computed map values are consistent with the aperture-mass calculation reported by the aperture_mass package for the same dataset
- Map exhibits per-pixel aperture mass values obtained by GPU convolution of the tangential shear field with the Schirmer filter

## Limitations

- The filter choice is fixed: the aperture_mass code uses only the Schirmer et al (2007) NFW filter with exponential cut-offs at zero and large radii; other filter profiles require code modification.
- No changelog is available for the package, so version-specific behavioral changes cannot be tracked.
- The implementation is GPU-based (designed for the ccogs suite), so execution depends on GPU availability.

## Evidence
