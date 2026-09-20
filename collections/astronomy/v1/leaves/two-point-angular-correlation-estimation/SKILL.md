---
name: two-point-angular-correlation-estimation
description: Use when you have sky positions of a galaxy dataset and need the angular
  correlation function w(theta) tabulated over angular separation bins, especially
  when GPU-accelerated pair counting via the ccogs suite is available.
when_to_use_negative:
- The target quantity is a shear-based statistic such as the aperture mass map — use
  the ccogs aperture_mass package (Schirmer et al 2007 filter) instead.
- You need a 3D spatial correlation function rather than the projected 2-pt angular
  correlation function w(theta); this package works only with sky positions.
- You cannot construct a flat random catalogue matching the survey geometry, since
  the estimator requires data-random cross-counts.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: angular_correlation
  role: Core package that builds pair-count histograms and evaluates the 2-pt angular
    correlation function w(theta) from data and random catalogues with a chosen estimator
- name: ccogs
  role: GPU suite that hosts the angular_correlation package for computing cosmological
    quantities on the GPU
  repo: github.com__djbard__ccogs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2012.11.001
    title: Cosmological calculations on the GPU
  source_content_hash: sha256:55396bbddae8b8295aaeae4003e34d5e44f7c9c2ee83be988fe563fba23d504e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - angular_correlation
  - ccogs
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# two-point-angular-correlation-estimation

## Summary

Computes the 2-pt angular correlation function w(theta) from a galaxy dataset using the angular_correlation package of the ccogs GPU suite, pairing the data catalogue against a flat distribution of randomly generated galaxies with a user-selected estimator.

## When to use

Use when you have sky positions of a galaxy dataset and need the angular correlation function w(theta) tabulated over angular separation bins, especially when GPU-accelerated pair counting via the ccogs suite is available.

## When NOT to use

- The target quantity is a shear-based statistic such as the aperture mass map — use the ccogs aperture_mass package (Schirmer et al 2007 filter) instead.
- You need a 3D spatial correlation function rather than the projected 2-pt angular correlation function w(theta); this package works only with sky positions.
- You cannot construct a flat random catalogue matching the survey geometry, since the estimator requires data-random cross-counts.

## Inputs

- Galaxy dataset with positions on the sky (data catalogue)
- Flat distribution of randomly generated galaxies matching the survey geometry (random catalogue)
- User-selected two-point correlation estimator choice

## Outputs

- Tabulated correlation function w(theta) over angular separation bins
- angular_correlation_results.csv (angular separations with corresponding w(theta) values)

## How to apply

Load the input galaxy dataset (positions on the sky) with the angular_correlation package from the ccogs suite. Generate a flat distribution of randomly generated galaxies matching the survey geometry to serve as the random catalogue. Build pair-count histograms for the data catalogue, the random catalogue, and their cross-counts, using the user-selected estimator (the package ships with one default estimator but the user has flexibility to explore others). Evaluate w(theta) from the pair counts with the chosen estimator and tabulate it over angular separation bins. Write the tabulated values to angular_correlation_results.csv. Rationale: the data-vs-random comparison removes survey geometry imprint, which is why a flat random catalogue matched to the data is required rather than raw data-only pair counts.

## Related tools

- **angular_correlation** (Core package that builds pair-count histograms and evaluates the 2-pt angular correlation function w(theta) from data and random catalogues with a chosen estimator)
- **ccogs** (GPU suite that hosts the angular_correlation package for computing cosmological quantities on the GPU) — github.com__djbard__ccogs

## Evaluation signals

- Confirm angular_correlation_results.csv exists and parses as a table of angular separations with corresponding w(theta) values
- Check the computed w(theta) is qualitatively consistent with the estimator chosen
- Verify the random catalogue is a flat distribution of randomly generated galaxies consistent with the survey geometry
- Optionally re-run with a different user-selected estimator and compare w(theta) estimates across estimators

## Limitations

- No changelog found for the repository, so package revisions cannot be tracked from documentation alone
- Only a particular default estimator is shipped; other estimators require user modification, and per the README the user 'has the flexibility to explore other estimators' rather than selecting among pre-built ones
- Requires GPU support since it is part of a suite of tools for computing cosmological quantities on the GPU

## Evidence
