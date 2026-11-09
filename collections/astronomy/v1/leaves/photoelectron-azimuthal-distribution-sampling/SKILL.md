---
name: photoelectron-azimuthal-distribution-sampling
description: Use when simulating IXPE (or generic X-ray polarimetry) observations
  and photon events must be assigned photoelectron emission angles consistent with
  the source polarization degree/angle, so Stokes columns Q and U can be populated
  in a FITS event list.
when_to_use_negative:
- You are analyzing existing IXPE level-1/level-2 FITS event files rather than generating
  simulated observations — the azimuthal sampling applies only to the Monte Carlo
  simulation stage
- The quantity to sample is multi-dimensional (e.g. energy-dependent joint distributions),
  since this method specifically covers one-dimensional pdfs sampled via a ppf spline
- The downstream framework or mission has no photoelectron-track polarization model
  (the article notes adaptation is possible but the azimuthal distribution is tied
  to polarization-sensitive detectors)
edam_operation: http://edamontology.org/operation_2928
tools:
- name: ixpeobssim
  role: Framework that convolves the photon spectrum with the IRFs and samples the
    azimuthal distribution to populate DETPHI/PHI, Q, U and W_MOM event columns
  repo: https://github.com/lucabaldini/ixpeobssim
- name: Python
  role: Implementation language in which the cumulative-function splines (ppf) are
    built for sampling one-dimensional pdfs
  repo: https://github.com/lucabaldini/ixpeobssim
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.softx.2022.101194
    title: 'ixpeobssim: A simulation and analysis framework for the imaging X-ray
      polarimetry explorer'
  source_content_hash: sha256:904156dc7ba7a9ebd3c7036ce27f672df9d1ae5bb14cd2517552d0cf65044b6a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lucabaldini/ixpeobssim
  tools:
  - Python
  - ixpeobssim
  edam_operation: http://edamontology.org/operation_2928
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# photoelectron-azimuthal-distribution-sampling

## Summary

Sampling of the photoelectron emission angle from the proper azimuthal distribution during ixpeobssim event simulation, which populates the polarization-sensitive event columns (PHI/DETPHI, Q, U, W_MOM) in the per-component event list.

## When to use

Use when simulating IXPE (or generic X-ray polarimetry) observations and photon events must be assigned photoelectron emission angles consistent with the source polarization degree/angle, so Stokes columns Q and U can be populated in a FITS event list.

## When NOT to use

- You are analyzing existing IXPE level-1/level-2 FITS event files rather than generating simulated observations — the azimuthal sampling applies only to the Monte Carlo simulation stage
- The quantity to sample is multi-dimensional (e.g. energy-dependent joint distributions), since this method specifically covers one-dimensional pdfs sampled via a ppf spline
- The downstream framework or mission has no photoelectron-track polarization model (the article notes adaptation is possible but the azimuthal distribution is tied to polarization-sensitive detectors)

## Inputs

- Photon spectrum of a single on-axis model component (e.g. power-law source from a configuration file such as ixpeobssim/config/toy_point_source.py with constant polarization degree and angle)
- Instrument response functions (IRFs) selected via the IRFNAME/CALDB keywords
- Count spectrum as a function of energy and time (or phase) convolved with the IRFs
- Azimuthal distribution of the photoelectron emission angle for the given polarization state

## Outputs

- Per-event photoelectron emission angles populating the DETPHI/PHI columns
- Stokes-parameter event columns Q and U and the W_MOM (weighted moments) column for each simulated event
- Partial (per-component) event lists that feed the vignetting, fiducial-area and dead-time filtering stages

## How to apply

Once the count spectrum has been calculated as a function of energy and time (or phase) from the photon spectrum convolved with the instrument response functions, extract the photoelectron emission angle for each event by sampling the proper azimuthal distribution. The sampling is done in ixpeobssim's standard way: build the cumulative function of the azimuthal pdf on a suitable regular grid and use interpolated splines through those values as the percent point function (ppf) for one-dimensional inverse-transform sampling. The sampled angles populate the DETPHI/PHI, Q, U, and W_MOM event columns, which later drive vignetting, fiducial-area and dead-time filtering stages and the final merged FITS event list (EVENTS/GTI/MONTE_CARLO extensions, MJDREF 01 Jan 2017).

## Related tools

- **ixpeobssim** (Framework that convolves the photon spectrum with the IRFs and samples the azimuthal distribution to populate DETPHI/PHI, Q, U and W_MOM event columns) — https://github.com/lucabaldini/ixpeobssim
- **Python** (Implementation language in which the cumulative-function splines (ppf) are built for sampling one-dimensional pdfs) — https://github.com/lucabaldini/ixpeobssim

## Evaluation signals

- The sampled azimuthal angles reproduce the input polarization degree and angle statistically (Stokes Q and U consistent with the model's polconst parameters)
- The ppf spline is built from the cumulative function evaluated on a suitable, regular grid before sampling
- Output FITS event list opens with astropy and contains an EVENTS extension with the documented PHI/DETPHI, Q, U and W_MOM columns
- The simulated count spectrum folded through the IRFs is statistically consistent with the input photon spectrum

## Limitations

- The azimuthal sampling is part of the simulation chain for a single model component; multi-component observations require merging partial event lists and sorting in time before instrumental corrections
- This sampling happens before vignetting, fiducial-area and dead-time filtering, so sampled events can be later discarded and the angles must be applied before those stages, not after
- ximpol, the predecessor framework, is no longer developed nor maintained; use ixpeobssim

## Evidence

- [abstract] Photoelectron angle extraction from azimuthal distribution: "we also have all the necessary ingredients to extract the photo-electron emission angle, according to the proper azimuthal distribution"
- [abstract] ppf spline sampling of one-dimensional pdfs: "we calculate the cumulative function on a suitable, regular grid and use the values to build an interpolated spline representing the percent point function (ppf)"
- [abstract] Downstream filtering stages operate after angle sampling: "the partial event lists are then merged and sorted in time. All the additional instrumental corrections are applied in dedicated filtering stages before the final, consolidated event list is written"
- [readme] Framework purpose per README: "Given a source model and the response functions of the telescopes, it is designed to produce realistic simulated observations, in the form of event lists in FITS format"
