---
name: galaxy-catalogue-pair-counting
description: Use when the analysis goal is to measure the angular clustering of a
  galaxy dataset, i.e. produce tabulated w(theta) values over angular separation bins
  from data and random catalogues.
when_to_use_negative:
- The target quantity is the aperture mass map for a dataset (use the aperture_mass
  package with the Schirmer et al (2007) filter instead)
- The catalogue lacks sky positions, so data–random pair counts on angular separations
  cannot be formed
edam_operation: http://edamontology.org/operation_0314
tools:
- name: ccogs
  role: GPU suite that hosts the angular_correlation package used to compute the correlation
    function
  repo: github.com__djbard__ccogs
- name: angular_correlation
  role: Calculates the 2-pt angular correlation function from data plus a flat random
    galaxy distribution using a chosen estimator
  repo: github.com__djbard__ccogs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2012.11.001
    title: Cosmological calculations on the GPU
  source_content_hash: sha256:ac9568749ca5766803cf5a246c5e7921299f4cc5cea54ae5820b379a29d20500
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - angular_correlation
  - ccogs
  edam_operation: http://edamontology.org/operation_0314
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# galaxy-catalogue-pair-counting

## Summary

Computes the 2-pt angular correlation function w(theta) from a galaxy data catalogue paired against a flat distribution of randomly generated galaxies, using a user-selected estimator from the angular_correlation package in the ccogs GPU suite.

## When to use

Use when the analysis goal is to measure the angular clustering of a galaxy dataset, i.e. produce tabulated w(theta) values over angular separation bins from data and random catalogues.

## When NOT to use

- The target quantity is the aperture mass map for a dataset (use the aperture_mass package with the Schirmer et al (2007) filter instead)
- The catalogue lacks sky positions, so data–random pair counts on angular separations cannot be formed

## Inputs

- Galaxy dataset with positions on the sky (data catalogue)
- Flat distribution of randomly generated galaxies matching the survey geometry (random catalogue)
- Choice of correlation estimator (user-selectable)

## Outputs

- Tabulated 2-pt angular correlation function w(theta) over angular separation bins
- angular_correlation_results.csv (angular separations with corresponding w(theta) values)

## How to apply

Load the input galaxy dataset (positions on the sky) with the angular_correlation package from the ccogs suite, then generate a flat distribution of randomly generated galaxies matching the survey geometry as the random catalogue. Build pair-count histograms for the data catalogue, the random catalogue, and their cross-counts, using the estimator chosen for the run; the user retains flexibility to switch and explore other estimators. Evaluate w(theta) from the pair counts with the chosen estimator and tabulate it over the angular separation bins, writing the result to angular_correlation_results.csv. The estimator choice is the key decision point, so qualitatively compare the computed w(theta) against expectations for the estimator selected.

## Related tools

- **ccogs** (GPU suite that hosts the angular_correlation package used to compute the correlation function) — github.com__djbard__ccogs
- **angular_correlation** (Calculates the 2-pt angular correlation function from data plus a flat random galaxy distribution using a chosen estimator) — github.com__djbard__ccogs

## Evaluation signals

- angular_correlation_results.csv exists and parses as a table of angular separations with corresponding w(theta) values
- The tabulated w(theta) is qualitatively consistent with the estimator chosen for the run
- The random catalogue is a flat distribution matching the survey geometry of the data catalogue
- w(theta) is tabulated over the defined angular separation bins rather than as a single scalar

## Limitations

- A particular estimator is chosen by default, but results depend on that choice; users must explore alternative estimators to check robustness
- No changelog found for the repository, so version-to-version behavior of the package is unverified

## Evidence

- [intro] angular_correlation computes w(theta) using data and randoms: "This code calculates the 2-pt angular correlation function using data and a flat distribution of randomly generated galaxies. A particular estimator is chosen, but the user has the flexibility to"
- [intro] ccogs is a GPU suite of cosmological tools: "This suite of tools contains packages for computing cosmological quantities on the GPU."
- [readme] README describes the angular_correlation package and estimator flexibility: "This code calculates the 2-pt angular correlation function using data and
a flat distribution of randomly generated galaxies. A particular estimator 
is chosen, but the user has the flexibility to"
- [readme] README states each package has its own README in its subdirectory: "Each package has its own descriptive "README" which can be found in its
subdirectory. In there, you can find the details of each package."
