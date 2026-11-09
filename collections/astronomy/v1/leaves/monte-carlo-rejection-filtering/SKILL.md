---
name: monte-carlo-rejection-filtering
description: Use when simulating X-ray polarimetric observations where an instrumental
  effect (vignetting vs. off-axis angle, chip fiducial area, readout dead-time) must
  be realized by stochastic event rejection rather than deterministic reweighting.
when_to_use_negative:
- Analyzing real, already-corrected IXPE level-2 event lists where vignetting and
  dead-time corrections are applied analytically rather than by Monte Carlo rejection
- Bin-level analysis (e.g. PHA1/PCUBE products via xpbin) — rejection filtering operates
  on unbinned event lists during simulation, not on binned data products
- Detectors whose instrumental effects (e.g. non-paralyzable dead-time, chip gap geometry)
  are not representable as independent per-event rejection probabilities
edam_operation: http://edamontology.org/operation_0244
tools:
- name: ixpeobssim
  role: Simulation framework implementing the ppf-spline sampling and the vignetting,
    fiducial-area, and dead-time rejection filter stages for IXPE event lists
  repo: https://github.com/lucabaldini/ixpeobssim
- name: Python
  role: Implementation language for the interpolated-spline ppf sampling and the filtering
    stages, with FITS output readable by astropy
- name: CALDB
  role: Source of the instrument response functions selected via the IRFNAME/CALDB
    configuration keywords
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.softx.2022.101194
    title: 'ixpeobssim: A simulation and analysis framework for the imaging X-ray
      polarimetry explorer'
  source_content_hash: sha256:e9491d40c877ed908dc61835c350f2e99b577c97120c353b29797eed71f77783
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lucabaldini/ixpeobssim
  tools:
  - CALDB
  - Python
  - ixpeobssim
  edam_operation: http://edamontology.org/operation_0244
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# monte-carlo-rejection-filtering

## Summary

Monte Carlo rejection sampling as used in ixpeobssim to realize instrumental effects: one-dimensional pdfs are sampled via interpolated splines built from the cumulative function (ppf), and corrections such as vignetting, fiducial-area clipping, and dead-time are applied by randomly discarding events with the proper statistical dependence.

## When to use

Use when simulating X-ray polarimetric observations where an instrumental effect (vignetting vs. off-axis angle, chip fiducial area, readout dead-time) must be realized by stochastic event rejection rather than deterministic reweighting.

## When NOT to use

- Analyzing real, already-corrected IXPE level-2 event lists where vignetting and dead-time corrections are applied analytically rather than by Monte Carlo rejection
- Bin-level analysis (e.g. PHA1/PCUBE products via xpbin) — rejection filtering operates on unbinned event lists during simulation, not on binned data products
- Detectors whose instrumental effects (e.g. non-paralyzable dead-time, chip gap geometry) are not representable as independent per-event rejection probabilities

## Inputs

- ixpeobssim configuration file (e.g. ixpeobssim/config/toy_point_source.py: point source with power-law spectrum, constant polarization degree and angle)
- Instrument response functions (IRFs) identified by the IRFNAME/CALDB keywords
- Photon source model (spatial, spectral, and polarization)
- Partial per-component event lists from the ROI

## Outputs

- Consolidated, time-sorted FITS event list with EVENTS, GTI and MONTE_CARLO extensions (columns TIME, ENERGY, X/Y, Q, U, W_MOM; TSTART/TSTOP in MET, MJDREF 01 Jan 2017)
- Filtered event sample statistically consistent with the photon spectrum folded through the IRFs

## How to apply

Build the count spectrum as a function of energy and time (or phase) from the photon spectrum convolved with the instrument response, then sample one-dimensional pdfs by calculating the cumulative function on a regular grid and building an interpolated spline representing the percent point function (ppf). Extract the photoelectron emission angle from the proper azimuthal distribution to populate DETPHI/PHI, Q, U and W_MOM columns. Apply each instrumental correction as a dedicated filtering stage: (1) vignetting by randomly discarding a fraction of events with the proper dependence on the off-axis angle; (2) fiducial-area clipping by discarding events whose projected focal-plane position falls outside the fiducial area of the readout chip; (3) dead-time by discarding events occurring during the readout of an earlier event. Merge and sort partial per-component event lists in time before the filtering stages, then write the consolidated event list as a FITS file with EVENTS, GTI and MONTE_CARLO extensions.

## Related tools

- **ixpeobssim** (Simulation framework implementing the ppf-spline sampling and the vignetting, fiducial-area, and dead-time rejection filter stages for IXPE event lists) — https://github.com/lucabaldini/ixpeobssim
- **Python** (Implementation language for the interpolated-spline ppf sampling and the filtering stages, with FITS output readable by astropy)
- **CALDB** (Source of the instrument response functions selected via the IRFNAME/CALDB configuration keywords)

## Evaluation signals

- The output FITS file opens with astropy and contains an EVENTS extension with the documented columns (TIME, ENERGY, X/Y, Q, U, W_MOM) and time keywords (TSTART, TSTOP, TELAPSE = TSTOP - TSTART)
- The simulated count spectrum is statistically consistent with the input photon spectrum folded through the IRFs
- Events with projected focal-plane positions outside the fiducial area of the readout chip are absent from the final list
- No events occur within the dead-time window of a previously read event
- The effective vignetting behavior matches the proper dependence on the off-axis angle in expectation

## Limitations

- Rejection-based stages discard events, so the realized effective area emerges statistically from the sample rather than analytically; a sufficiently large event sample is needed to match the folded spectrum within statistical errors
- The framework is IXPE-specific in much of its detail, though its modular implementation makes it potentially adaptable to other missions with polarization capabilities
- The predecessor ximpol is no longer developed nor maintained; ixpeobssim is the supported framework

## Evidence

- [abstract] Vignetting rejection stage: "the correct vignetting function is obtained by randomly discarding a fraction of the events with the proper dependence on the off-axis angle"
- [abstract] Fiducial-area and dead-time stages: "events with a projected position on the focal plane lying outside the fiducial area of the readout chip are discarded"
- [abstract] Dead-time rejection: "events occurring during the readout of an earlier event are discarded, as well."
- [abstract] ppf spline sampling: "we calculate the cumulative function on a suitable, regular grid and use the values to build an interpolated spline representing the percent point function (ppf)"
- [readme] Framework context for simulated event lists: "it is designed to produce realistic simulated observations, in the form of event lists in FITS format, containing a strict superset of the information included in the publicly released IXPE data"
