---
name: weak-lensing-shear-catalog-processing
description: Use when you have a weak-lensing galaxy catalog of positions and ellipticities
  and need to produce a 2-D aperture mass map over the surveyed field to locate mass
  concentrations.
when_to_use_negative:
- Input is a catalog for computing the 2-pt angular correlation function (use the
  angular_correlation package of ccogs instead)
- You need a generic matched filter other than the Schirmer et al (2007) NFW-with-exponential-cut-offs
  profile; the package hard-codes that filter choice
- The task requires CPU-only execution with no GPU available, since ccogs packages
  are designed to compute cosmological quantities on the GPU
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: aperture_mass
  role: 'Primary package: constructs the Schirmer et al (2007) filter, convolves it
    with the tangential shear field on the GPU, and writes the aperture mass map to
    aperture_mass_map.fits'
  repo: https://github.com/djbard/ccogs
- name: ccogs
  role: Suite providing the aperture_mass package and GPU infrastructure for the cosmological
    calculations
  repo: https://github.com/djbard/ccogs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2012.11.001
    title: Cosmological calculations on the GPU
  source_content_hash: sha256:c24aa12a2da2107d1d794fc6cc8e6d2fe857e37a94aa8ff7c61a8dc8f95c547a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/djbard/ccogs
  tools:
  - aperture_mass
  - ccogs
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# weak-lensing-shear-catalog-processing

## Summary

Computes an aperture mass map from a weak-lensing galaxy shear catalog (positions and ellipticities) using the aperture_mass package of the ccogs GPU suite, applying the Schirmer et al (2007) filter — an NFW profile with exponential cut-offs at zero and large radii.

## When to use

Use when you have a weak-lensing galaxy catalog of positions and ellipticities and need to produce a 2-D aperture mass map over the surveyed field to locate mass concentrations.

## When NOT to use

- Input is a catalog for computing the 2-pt angular correlation function (use the angular_correlation package of ccogs instead)
- You need a generic matched filter other than the Schirmer et al (2007) NFW-with-exponential-cut-offs profile; the package hard-codes that filter choice
- The task requires CPU-only execution with no GPU available, since ccogs packages are designed to compute cosmological quantities on the GPU

## Inputs

- Weak-lensing galaxy shear catalog with galaxy positions and elliptities (shear estimates)
- Grid of map pixel positions defining the aperture mass map over the surveyed field

## Outputs

- 2-D aperture mass map written to aperture_mass_map.fits (readable FITS image with expected map dimensions)

## How to apply

Load the weak-lensing galaxy shear catalog (positions and elliptities) into the aperture_mass package from the ccogs suite. Construct the Schirmer et al (2007) filter — an NFW profile with exponential cut-offs at zero and large radii — on a grid of radii centered at each map pixel. Convolve the filter with the tangential shear field of the dataset to evaluate the aperture mass at each grid position, running the kernel on the GPU as designed for ccogs. Assemble the per-position aperture mass values into a 2-D mass map over the surveyed field and write it to aperture_mass_map.fits (FITS format).

## Related tools

- **aperture_mass** (Primary package: constructs the Schirmer et al (2007) filter, convolves it with the tangential shear field on the GPU, and writes the aperture mass map to aperture_mass_map.fits) — https://github.com/djbard/ccogs
- **ccogs** (Suite providing the aperture_mass package and GPU infrastructure for the cosmological calculations) — https://github.com/djbard/ccogs

## Evaluation signals

- aperture_mass_map.fits exists in the output directory
- aperture_mass_map.fits is a readable FITS image with the expected map dimensions
- Computed map values are consistent with the aperture-mass calculation reported by the aperture_mass package for the given dataset
- Per-position aperture mass values assemble into a coherent 2-D map covering the surveyed field without gaps or misaligned pixels

## Limitations

- The filter is fixed to the Schirmer et al (2007) form (NFW profile with exponential cut-offs at zero and large radii); other filter choices are not provided in this code
- The kernel is designed to run on the GPU as part of the ccogs suite, so a GPU environment is expected
- No changelog was found in the repository documentation

## Evidence
