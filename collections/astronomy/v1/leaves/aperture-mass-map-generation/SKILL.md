---
name: aperture-mass-map-generation
description: Use when given a weak-lensing galaxy shear catalog (positions and elliptities)
  and the goal is to produce an aperture mass map over the surveyed field via the
  Schirmer et al (2007) filter.
when_to_use_negative:
- The analysis goal is the 2-pt angular correlation function rather than an aperture
  mass map — use the ccogs angular_correlation package instead, which uses data plus
  a flat distribution of randomly generated galaxies and a chosen estimator.
- The input is already a computed aperture mass map (FITS image) rather than a raw
  shear catalog; this skill reconstructs the map from galaxy positions and elliptities.
- No GPU execution environment is available, since ccogs packages are designed for
  computing cosmological quantities on the GPU.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3336
- http://edamontology.org/topic_3397
tools:
- name: aperture_mass
  role: Core package that calculates the aperture mass map for a given dataset using
    the Schirmer et al (2007) NFW-profile filter with exponential cut-offs at zero
    and large radii.
  repo: https://github.com/djbard/ccogs
- name: ccogs
  role: GPU suite containing aperture_mass; the convolution kernel runs on the GPU
    as designed for ccogs.
  repo: https://github.com/djbard/ccogs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2012.11.001
    title: Cosmological calculations on the GPU
  source_content_hash: sha256:127603d1567074df48318423850f9b5acc9ff70f540d84fb20c53b1b0424f578
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/djbard/ccogs
  tools:
  - aperture_mass
  - ccogs
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3336
  - http://edamontology.org/topic_3397
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# aperture-mass-map-generation

## Summary

Computes a 2-D aperture mass map from a weak-lensing galaxy shear catalog using the aperture_mass package of the ccogs suite, applying the Schirmer et al (2007) filter — an NFW profile with exponential cut-offs at zero and large radii — with GPU-accelerated convolution.

## When to use

Use when given a weak-lensing galaxy shear catalog (positions and elliptities) and the goal is to produce an aperture mass map over the surveyed field via the Schirmer et al (2007) filter.

## When NOT to use

- The analysis goal is the 2-pt angular correlation function rather than an aperture mass map — use the ccogs angular_correlation package instead, which uses data plus a flat distribution of randomly generated galaxies and a chosen estimator.
- The input is already a computed aperture mass map (FITS image) rather than a raw shear catalog; this skill reconstructs the map from galaxy positions and elliptities.
- No GPU execution environment is available, since ccogs packages are designed for computing cosmological quantities on the GPU.

## Inputs

- Weak-lensing galaxy shear catalog with galaxy positions and elliptities
- Grid of radii centered at each map pixel (constructed by the workflow)
- Dataset parameters for the surveyed field (map pixel grid dimensions)

## Outputs

- 2-D aperture mass map over the surveyed field
- aperture_mass_map.fits (readable FITS image with expected map dimensions)

## How to apply

Load the galaxy shear catalog (sky positions and elliptities) into the aperture_mass package from the ccogs suite. Construct the Schirmer et al (2007) filter — an NFW profile with exponential cut-offs at zero and large radii — on a grid of radii centered at each map pixel. Convolve the filter with the tangential shear field of the dataset to evaluate the aperture mass at each grid position, running the kernel on the GPU as designed for ccogs. Assemble the per-position aperture mass values into a 2-D mass map and write it to aperture_mass_map.fits. Validate that the output FITS image exists, is readable, has the expected map dimensions, and that its values agree with the aperture-mass calculation reported by the aperture_mass package.

## Related tools

- **aperture_mass** (Core package that calculates the aperture mass map for a given dataset using the Schirmer et al (2007) NFW-profile filter with exponential cut-offs at zero and large radii.) — https://github.com/djbard/ccogs
- **ccogs** (GPU suite containing aperture_mass; the convolution kernel runs on the GPU as designed for ccogs.) — https://github.com/djbard/ccogs

## Evaluation signals

- aperture_mass_map.fits exists and is a readable FITS image.
- The output FITS image has the expected map dimensions for the surveyed field.
- Computed map values are consistent with the aperture-mass calculation reported by the aperture_mass package for the given dataset.
- The Schirmer et al (2007) filter construction shows an NFW profile with exponential cut-offs at both zero and large radii.
- Per-position aperture mass values assemble into a coherent 2-D map over the full pixel grid.

## Limitations

- The Schirmer et al (2007) filter applies exponential cut-offs at zero and large radii, constraining its response near the map center and at the field edges.
- The implementation is designed to run on the GPU; performance and availability depend on a suitable GPU environment (ccogs suite design).
- No changelog found, so version-level provenance of the code cannot be traced.

## Evidence

- [intro] filter definition: "This code calculates the aperture mass map for a given dataset. In this code, we use the filter proposed by Schirmer et al (2007), which is an NFW profile with exponential cut-offs at zero and large"
- [readme] GPU design: "This suite of tools contains packages for computing cosmological quantities on the GPU."
- [readme] package role: "This code calculates the aperture mass map for a given dataset."
