---
name: correlation-function-tabulation
description: Use when you have an input galaxy dataset (positions on the sky) and
  need to produce the 2-pt angular correlation function w(theta) as a tabulated table
  of angular separations vs. correlation values.
when_to_use_negative:
- The goal is weak-lensing aperture mass statistics, not galaxy clustering — use the
  ccogs aperture_mass package (Schirmer et al 2007 filter on an NFW profile) instead
- You need a 3-D spatial correlation function rather than the 2-pt angular correlation
  function on the sky
edam_operation: http://edamontology.org/operation_2426
tools:
- name: angular_correlation
  role: Core package that calculates the 2-pt angular correlation function from data
    and a flat random catalogue with a chosen estimator
- name: ccogs
  role: Parent GPU suite containing the angular_correlation package for computing
    cosmological quantities
  repo: github.com__djbard__ccogs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2012.11.001
    title: Cosmological calculations on the GPU
  source_content_hash: sha256:50261e0fb8b0f9469dd591c7d2d3ce84ccc01c623b826299631472d93f290924
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - angular_correlation
  - ccogs
  edam_operation: http://edamontology.org/operation_2426
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# correlation-function-tabulation

## Summary

Computes the 2-pt angular correlation function w(theta) for a galaxy dataset using the angular_correlation package from the ccogs GPU suite, pairing the data catalogue against a flat distribution of randomly generated galaxies with a user-selected estimator, and tabulating w(theta) over angular separation bins.

## When to use

Use when you have an input galaxy dataset (positions on the sky) and need to produce the 2-pt angular correlation function w(theta) as a tabulated table of angular separations vs. correlation values.

## When NOT to use

- The goal is weak-lensing aperture mass statistics, not galaxy clustering — use the ccogs aperture_mass package (Schirmer et al 2007 filter on an NFW profile) instead
- You need a 3-D spatial correlation function rather than the 2-pt angular correlation function on the sky

## Inputs

- Galaxy dataset (positions on the sky) as the data catalogue
- Flat distribution of randomly generated galaxies matching the survey geometry (random catalogue)

## Outputs

- Tabulated 2-pt angular correlation function w(theta) over angular separation bins, written to angular_correlation_results.csv

## How to apply

Load the sky-position galaxy dataset with the angular_correlation package from the ccogs suite and generate a flat distribution of randomly generated galaxies matching the survey geometry as the random catalogue. Build pair-count histograms for the data catalogue, the random catalogue, and their cross-counts, using the user-selected correlation estimator. Evaluate w(theta) from the pair counts with the chosen estimator over the angular separation bins. The estimator is a decision point: the package ships with a particular default estimator but the user has the flexibility to explore other estimators, so results can be compared across estimator choices. Calculations run on the GPU as part of the ccogs suite of packages.

## Related tools

- **angular_correlation** (Core package that calculates the 2-pt angular correlation function from data and a flat random catalogue with a chosen estimator)
- **ccogs** (Parent GPU suite containing the angular_correlation package for computing cosmological quantities) — github.com__djbard__ccogs

## Evaluation signals

- Confirm the output file angular_correlation_results.csv exists and parses as a table of angular separations with corresponding w(theta) values
- Check that the computed w(theta) is qualitatively consistent with the estimator chosen
- Exploit estimator flexibility: recompute with a different estimator and compare tabulated w(theta) values for qualitative agreement
- Verify the random catalogue is a flat distribution matching the survey geometry, so pair-count histograms for data, randoms, and cross-counts are built over comparable solid angle

## Limitations

- The estimator choice is user-selectable, so results depend on which estimator was chosen; comparisons across estimators are qualitative
- No changelog found for the ccogs package, so version provenance of the code cannot be verified from the repository documentation
- The random catalogue is a flat distribution of randomly generated galaxies, which assumes the survey geometry can be adequately represented by flat randoms

## Evidence

- [readme] README package description: "This code calculates the 2-pt angular correlation function using data and a flat distribution of randomly generated galaxies. A particular estimator is chosen, but the user has the flexibility to"
- [intro] Article intro on ccogs suite: "This suite of tools contains packages for computing cosmological quantities on the GPU."
- [methods] Task validation workflow: [evidence span withheld — anchor status: not_found]
