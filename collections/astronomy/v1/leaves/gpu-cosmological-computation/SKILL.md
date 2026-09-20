---
name: gpu-cosmological-computation
description: Use when you need GPU-accelerated cosmological quantities — an aperture
  mass map from a weak-lensing galaxy shear catalog, or the 2-pt angular correlation
  function w(theta) of a galaxy catalog — via the ccogs packages.
when_to_use_negative:
- You need correlation functions computed from a real observed random catalogue matched
  to survey depth/geometry — this skill uses a flat distribution of randomly generated
  galaxies as the random catalogue.
- You need a cosmological quantity outside the two ccogs packages (aperture mass maps
  and 2-pt angular correlation functions).
- You need an aperture mass filter other than the Schirmer et al (2007) NFW-profile-with-exponential-cut-offs
  form; aperture_mass uses that specific filter.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: ccogs
  role: GPU suite hosting the aperture_mass and angular_correlation packages used
    to compute cosmological quantities
  repo: github.com/djbard/ccogs
- name: aperture_mass
  role: ccogs package that convolves the Schirmer et al (2007) filter with the tangential
    shear field to produce aperture_mass_map.fits
  repo: github.com/djbard/ccogs
- name: angular_correlation
  role: ccogs package that computes w(theta) from data and a flat random catalogue
    with a user-selected estimator, producing angular_correlation_results.csv
  repo: github.com/djbard/ccogs
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2012.11.001
    title: Cosmological calculations on the GPU
  source_content_hash: sha256:6b8c2ac36f7562dcfb36e111e073c25545ce3b940d9767cde1f5cacd3dd628e1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - angular_correlation
  - aperture_mass
  - ccogs
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# gpu-cosmological-computation

## Summary

A GPU-accelerated cosmological analysis skill from the ccogs suite: compute aperture mass maps from weak-lensing shear catalogs using the Schirmer et al (2007) filter, and 2-pt angular correlation functions w(theta) from data plus flat random catalogs with a user-selectable estimator.

## When to use

Use when you need GPU-accelerated cosmological quantities — an aperture mass map from a weak-lensing galaxy shear catalog, or the 2-pt angular correlation function w(theta) of a galaxy catalog — via the ccogs packages.

## When NOT to use

- You need correlation functions computed from a real observed random catalogue matched to survey depth/geometry — this skill uses a flat distribution of randomly generated galaxies as the random catalogue.
- You need a cosmological quantity outside the two ccogs packages (aperture mass maps and 2-pt angular correlation functions).
- You need an aperture mass filter other than the Schirmer et al (2007) NFW-profile-with-exponential-cut-offs form; aperture_mass uses that specific filter.

## Inputs

- Weak-lensing galaxy shear catalog (sky positions and elliptities) for aperture_mass
- Galaxy catalog of sky positions for angular_correlation
- User-selected correlation estimator choice for angular_correlation

## Outputs

- aperture_mass_map.fits — 2-D aperture mass map over the surveyed field
- angular_correlation_results.csv — tabulated w(theta) versus angular separation bins
- Flat random galaxy catalogue generated internally for the estimator pair counts

## How to apply

For the aperture mass map, load the weak-lensing galaxy catalog (positions and elliptities) into the aperture_mass package, construct the Schirmer et al (2007) filter — an NFW profile with exponential cut-offs at zero and large radii — on a grid of radii centered at each map pixel, and convolve it with the tangential shear field on the GPU. Assemble per-position aperture mass values into a 2-D map and write it to aperture_mass_map.fits. For the angular correlation, load the galaxy positions into the angular_correlation package, generate a flat distribution of randomly generated galaxies matching the survey geometry as the random catalogue, build data, random, and cross pair-count histograms with the user-selected estimator, evaluate w(theta) over angular separation bins, and write angular_correlation_results.csv. Each ccogs package carries its own README and a CITING_THIS_CODE file with calculation and implementation details.

## Related tools

- **ccogs** (GPU suite hosting the aperture_mass and angular_correlation packages used to compute cosmological quantities) — github.com/djbard/ccogs
- **aperture_mass** (ccogs package that convolves the Schirmer et al (2007) filter with the tangential shear field to produce aperture_mass_map.fits) — github.com/djbard/ccogs
- **angular_correlation** (ccogs package that computes w(theta) from data and a flat random catalogue with a user-selected estimator, producing angular_correlation_results.csv) — github.com/djbard/ccogs

## Evaluation signals

- aperture_mass_map.fits exists, is a readable FITS image, and has the expected map dimensions for the surveyed field
- Map values are consistent with the aperture-mass calculation reported by the aperture_mass package for the given dataset
- angular_correlation_results.csv exists and parses as a table of angular separations with corresponding w(theta) values
- Computed w(theta) is qualitatively consistent with the estimator chosen, with user flexibility to compare against other estimators
- Computation runs on the GPU as designed for the ccogs suite

## Limitations

- The angular correlation random catalogue is a flat distribution of randomly generated galaxies, not an observed depth-matched random sample
- A single correlation estimator is chosen by default; exploring others is left to the user
- The aperture_mass code is fixed to the Schirmer et al (2007) filter (NFW profile with exponential cut-offs at zero and large radii)
- No changelog found for the ccogs repository

## Evidence

- [intro] This suite of tools contains packages for computing cosmological quantities on the GPU.: "This suite of tools contains packages for computing cosmological quantities on the GPU."
