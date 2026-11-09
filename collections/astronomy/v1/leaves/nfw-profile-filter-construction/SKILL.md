---
name: nfw-profile-filter-construction
description: Use when computing an aperture mass map from a weak-lensing galaxy shear
  catalog (positions and elliptities) and a radially defined mass filter matching
  an NFW profile is required.
when_to_use_negative:
- The goal is computing the 2-pt angular correlation function — the ccogs angular_correlation
  package is the correct tool, not aperture_mass
- No GPU execution environment is available, since the aperture_mass kernel is designed
  to run cosmological calculations on the GPU
- A different filter profile than the Schirmer et al (2007) NFW-with-exponential-cut-offs
  filter is required
tools:
- name: aperture_mass
  role: Calculates the aperture mass map for a given dataset using the Schirmer et
    al (2007) filter; run on the GPU
  repo: github.com/djbard/ccogs
- name: ccogs
  role: Suite of tools containing the aperture_mass package for computing cosmological
    quantities on the GPU
  repo: github.com/djbard/ccogs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2012.11.001
    title: Cosmological calculations on the GPU
  source_content_hash: sha256:3f8b9db7724c9f84efd32aa68b0b3fe4ccf54be004357e7315ab1017588af7f3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - aperture_mass
  - ccogs
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# nfw-profile-filter-construction

## Summary

Constructs the aperture mass filter proposed by Schirmer et al (2007) — an NFW profile with exponential cut-offs at zero and large radii — for use in weak-lensing aperture mass mapping. This is the filter implemented in the aperture_mass package of the ccogs GPU cosmology suite.

## When to use

Use when computing an aperture mass map from a weak-lensing galaxy shear catalog (positions and elliptities) and a radially defined mass filter matching an NFW profile is required.

## When NOT to use

- The goal is computing the 2-pt angular correlation function — the ccogs angular_correlation package is the correct tool, not aperture_mass
- No GPU execution environment is available, since the aperture_mass kernel is designed to run cosmological calculations on the GPU
- A different filter profile than the Schirmer et al (2007) NFW-with-exponential-cut-offs filter is required

## Inputs

- Weak-lensing galaxy shear catalog with positions and elliptities
- Grid of radii centered at each map pixel

## Outputs

- 2-D aperture mass map over the surveyed field (aperture_mass_map.fits)

## How to apply

Load the weak-lensing galaxy shear catalog (positions and elliptities) into the aperture_mass package from the ccogs suite. Construct the Schirmer et al (2007) filter — an NFW profile with exponential cut-offs at zero and large radii — on a grid of radii centered at each map pixel. Convolve the filter with the tangential shear field of the dataset to evaluate the aperture mass at each grid position, running the kernel on the GPU as designed for ccogs. Assemble the per-position aperture mass values into a 2-D mass map over the surveyed field and write it to aperture_mass_map.fits. The key design choice is the filter shape: NFW profile truncated exponentially at both zero and large radii, which regularizes the profile at the center and at the aperture boundary.

## Related tools

- **aperture_mass** (Calculates the aperture mass map for a given dataset using the Schirmer et al (2007) filter; run on the GPU) — github.com/djbard/ccogs
- **ccogs** (Suite of tools containing the aperture_mass package for computing cosmological quantities on the GPU) — github.com/djbard/ccogs

## Evaluation signals

- aperture_mass_map.fits exists and is a readable FITS image
- The FITS image has the expected map dimensions for the surveyed field
- The computed map values are consistent with the aperture-mass calculation reported by the aperture_mass package for the given dataset
- Per-position aperture mass values can be reassembled into the 2-D map without gaps or reordering errors

## Limitations

- No changelog found for the ccogs suite, so version-specific behavior of the filter implementation is not documented
- The filter form is fixed in the code to the Schirmer et al (2007) profile; no alternative estimators or filter shapes are offered for aperture mass
- No changelog or documentation beyond per-package READMEs is available for verifying parameter details

## Evidence
